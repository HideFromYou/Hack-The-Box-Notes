# Privilege Escalation

## Overview

Initial access to a remote server is usually in the context of a low-privileged user. To gain full control, we need to find a local vulnerability or misconfiguration to escalate privileges to root (Linux) or Administrator/SYSTEM (Windows).

## Learning Objectives

- Use privilege escalation checklists and automated enumeration scripts
- Identify kernel exploits and vulnerable installed software
- Spot and abuse sudo misconfigurations with GTFOBins/LOLBAS
- Abuse writable cron jobs and scheduled tasks
- Find exposed credentials and SSH keys for lateral movement or persistence

## Privilege Escalation Checklists

HackTricks and PayloadsAllTheThings both maintain thorough Linux/Windows local privilege escalation checklists. Running through the various checks manually builds familiarity with what each one is actually looking for, rather than just running a script and reading colored output.

## Enumeration Scripts

Enumeration scripts automate many of the manual checklist items. Common options:

- **Linux:** LinEnum, linuxprivchecker
- **Windows:** Seatbelt, JAWS
- **Cross-platform:** PEASS (Privilege Escalation Awesome Scripts SUITE) — well maintained and frequently updated

```bash
./linpeas.sh
```

linpeas.sh collects system info and outputs a color-coded report:

| Color | Meaning |
|---|---|
| RED/YELLOW | Likely PE vector |
| RED | Must look |
| LightCyan | Users with console |
| Blue | Users without console & mounted devices |
| Green | Common items — users/groups/SUID/SGID/mounts/`.sh` scripts/cronjobs |
| LightMagenta | Your current username |

**Note:** these scripts run many checks very quickly, which creates "noise" that can trigger AV/EDR or security monitoring. Sometimes manual enumeration is the safer choice, especially on a monitored or production target.

## Kernel Exploits

An unpatched or old OS is often vulnerable to a specific kernel exploit — e.g., Linux `3.9.0-73-generic` is vulnerable to CVE-2016-5195 ("DirtyCow"), found via a quick Google search or `searchsploit`.

Kernel exploits can cause system instability. Be careful on production systems — prefer testing in a lab first, and only run one against production with explicit client approval.

## Vulnerable Software

Check installed software for public exploits, especially older or unpatched versions:

```bash
dpkg -l                 # Linux — installed packages
```

On Windows, check `C:\Program Files` and `C:\Program Files (x86)` the same way.

## User Privileges — Sudo

```bash
sudo -l
```

- `(ALL : ALL) ALL` → full sudo access: `sudo su -` to switch straight to root.
- `(user : user) NOPASSWD: /bin/echo` → run a specific command as a specific user without a password: `sudo -u user /bin/echo Hello World!`

Once an app we have sudo rights over is identified, check **GTFOBins** for the exact command needed to turn that access into a root shell. **LOLBAS** is the Windows equivalent for living-off-the-land binaries.

## Scheduled Tasks and Cron Jobs

Two main abuse paths: add a brand-new scheduled task/cron job, or trick an existing one into executing malicious code. Common writable Linux cron locations:

```
/etc/crontab
/etc/cron.d
/var/spool/cron/crontabs/root
```

If a directory called by a cron job is writable, dropping a reverse-shell bash script there and waiting for the next run is enough.

## Exposed Credentials

Check readable config files, logs, and history files for cleartext credentials:

```bash
cat ~/.bash_history        # Linux
```

On Windows, check PSReadLine history instead. Enumeration scripts usually surface these automatically. Also watch for password reuse — the same password turning up elsewhere, such as in a database connection string.

## SSH Keys

If we have **read** access to a user's `.ssh` directory, their private key can be copied and used to log in directly:

```bash
chmod 600 id_rsa
ssh root@<TARGET> -i id_rsa
```

If instead we have **write** access to a user's `.ssh` directory (after already gaining some access as that user), we can plant our own key for persistence:

```bash
ssh-keygen -f key
echo "ssh-rsa AAAA...=" >> /root/.ssh/authorized_keys
ssh root@<TARGET> -i key
```

No password needed going forward.

## Skills Practiced

- Manual and automated Linux/Windows privilege escalation enumeration
- Sudo misconfiguration abuse via GTFOBins
- Cron job and scheduled task abuse
- SSH key extraction and persistence

## Key Takeaways

- Automated enumeration scripts are fast but noisy — manual checks are sometimes the better call on a monitored host.
- `sudo -l` output is only half the picture; GTFOBins (or LOLBAS on Windows) turns "I can run this binary as root" into an actual shell.
- A writable file referenced by a privileged cron job or sudo rule is effectively the same as having sudo on `/bin/sh` — find out what calls it before assuming it's a dead end.
- Read access to `.ssh` grants immediate access; write access grants durable persistence — know which one you have.
