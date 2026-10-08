# FTP

## Overview

This lesson covers FTP/TFTP protocol fundamentals, dangerous vsFTPd configuration settings, and hands-on anonymous-login enumeration using the native client, manual protocol interaction, and Nmap NSE scripts.

## Learning Objectives

- Distinguish FTP's control/data channels and active vs. passive modes
- Identify dangerous vsFTPd configuration settings
- Enumerate an FTP server via anonymous login, `wget` mirroring, and manual protocol interaction
- Use Nmap's `ftp-anon` and `ftp-syst` NSE scripts to footprint an FTP service

## Protocol Fundamentals

FTP (File Transfer Protocol) is an application-layer protocol, one of the oldest on the internet. It uses **two** channels: control (TCP 21, commands/status codes) and data (TCP 20, actual file transfer, error-checked/resumable).

**Active vs. Passive FTP:**
- **Active** — the server initiates the data connection back to the client. Fails if the client is firewalled.
- **Passive** — the server announces a port and the client initiates the connection. Works through client-side firewalls, which is why it's widely preferred.

**Anonymous FTP** — the server allows login with no real credentials; the options available to the anonymous user are usually limited for security.

**TFTP (Trivial FTP)** — simpler, UDP-based (unreliable, needs application-layer recovery), with **no authentication at all** — access is controlled purely by OS-level file read/write permissions. Commands: `connect`, `get`, `put`, `quit`, `status`, `verbose`. TFTP has no directory listing capability (no `ls`).

## vsFTPd Configuration

Config file: `/etc/vsftpd.conf`.

| Setting | Default/Note |
|---|---|
| `listen` | `NO` |
| `anonymous_enable` | `NO` (default off) |
| `local_enable` | `YES` |
| `write_enable` | `YES` |
| `ssl_enable` | `NO` |

`/etc/ftpusers` lists users explicitly **denied** FTP login even if they exist on the system.

**Dangerous anonymous settings:**

| Setting | Risk |
|---|---|
| `anonymous_enable=YES` | Anonymous login allowed |
| `anon_upload_enable=YES` | Anonymous users can upload files |
| `anon_mkdir_write_enable=YES` | Anonymous users can create directories |
| `no_anon_password=YES` | No password required for anonymous login |
| `anon_root=<path>` | Dedicated root directory for anonymous users |
| `write_enable=YES` | Write access enabled globally |

## Anonymous Login and Enumeration

```bash
ftp <TARGET>
Name: anonymous
```

Inside the session:

- `ls` — list directory contents
- `status` — show connection settings
- `debug` + `trace` — verbose raw command tracing, shows the PORT/LIST commands sent
- `ls -R` — recursive listing (needs `ls_recurse_enable=YES` server-side)
- `get <file>` — download a file
- `put <file>` — test upload if write access is suspected

`hide_ids=YES` masks the real UID/GID as "ftp" in listings — an anti-recon measure, though fail2ban-style lockouts are now the standard defense against brute-forcing discovered usernames anyway.

## Mass Download

```bash
wget -m --no-passive ftp://anonymous:anonymous@<TARGET>
```

Creates a folder named after the target IP with everything mirrored locally — but this is noisy/noticeable due to unusual download volume.

## Manual Protocol Interaction

```bash
nc -nv <TARGET> 21
telnet <TARGET> 21
```

If the service is TLS/SSL-wrapped, use `openssl` instead (this also reveals the SSL certificate's hostname/email/org):

```bash
openssl s_client -connect <TARGET>:21 -starttls ftp
```

## Nmap FTP Footprinting

```bash
sudo nmap --script-updatedb
find / -type f -name ftp* 2>/dev/null | grep scripts
sudo nmap -sV -p21 -sC -A <TARGET>
sudo nmap -sV -p21 -sC -A <TARGET> --script-trace
```

- `ftp-anon` — checks and lists anonymous access.
- `ftp-syst` — runs the STAT command, showing server status/version.
- `--script-trace` — shows the raw commands/responses NSE scripts exchange with the server.

Upload capability on an FTP server tied to a web server significantly raises risk (a potential direct webshell/RCE path) — a common finding because admins neglect hardening "internal-only" components they assume are unreachable externally.

## Skills Practiced

- Identifying active vs. passive FTP and reasoning about firewall implications
- Spotting dangerous vsFTPd settings from a config file
- Anonymous FTP login, interactive enumeration, and mass mirroring with `wget`
- Manual protocol interaction via `nc`/`telnet`/`openssl`
- Nmap NSE-based FTP footprinting (`ftp-anon`, `ftp-syst`, `--script-trace`)

## Key Takeaways

- Passive FTP's popularity is a direct consequence of client-side firewalls — active mode simply fails against them, so passive became the practical default.
- Anonymous access alone isn't necessarily a vulnerability, but combined with upload capability on a server tied to a web root, it becomes a direct RCE path — a common oversight on "internal-only" components.
- TFTP's complete lack of authentication means its access control lives entirely at the OS file-permission layer — there's no protocol-level gate at all.
- `--script-trace` is valuable beyond FTP specifically: it shows exactly what an NSE script sends/receives, useful for verifying automated findings manually.
