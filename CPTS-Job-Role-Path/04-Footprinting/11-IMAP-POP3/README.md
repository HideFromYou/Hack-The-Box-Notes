# IMAP / POP3

## Overview

This lesson covers IMAP vs. POP3 fundamentals, Dovecot default/dangerous settings, and footprinting both services via Nmap, `curl`, and `openssl s_client` — including reading the TLS certificate for organization/FQDN disclosure and authenticating once valid credentials are known.

## Learning Objectives

- Distinguish IMAP (online, server-side mailbox management) from POP3 (download/delete only)
- Identify dangerous Dovecot auth-logging settings
- Footprint IMAP/POP3 with Nmap, extracting TLS cert info (org, FQDN)
- Authenticate and interact with IMAP/POP3 via `curl` and `openssl s_client`

## Protocol Fundamentals

**IMAP (Internet Message Access Protocol)** — online mailbox management directly on the server: folder structures, multiple clients staying in sync, emails stay server-side until explicitly deleted. Supports simultaneous multi-user/multi-client access. Text-based, ASCII commands, pipelined (can send several commands without waiting for each response — matched back via command identifiers).

**POP3 (Post Office Protocol)** — much simpler: list, retrieve, delete. No server-side folder structure, no online management — built for "download then disconnect" workflows.

**Ports:**

| Service | Plaintext | Encrypted (TLS/SSL) |
|---|---|---|
| POP3 | 110 | 995 |
| IMAP | 143 | 993 |

Both are plaintext by default (commands, credentials, email content) unless upgraded to TLS/SSL — same theme as SMTP.

## IMAP Commands

| Command | Description |
|---|---|
| `1 LOGIN username password` | Authenticate |
| `1 LIST "" *` | List all directories/mailboxes |
| `1 CREATE "INBOX"` | Create a mailbox |
| `1 DELETE "INBOX"` | Delete a mailbox |
| `1 RENAME "ToRead" "Important"` | Rename a mailbox |
| `1 SELECT INBOX` | Select a mailbox to access its messages |
| `1 FETCH <ID> all` | Retrieve a message's data |
| `1 CLOSE` | Permanently remove Deleted-flagged messages |
| `1 LOGOUT` | Close the connection |

(The leading `1` is a client-chosen command identifier/tag — needed since IMAP pipelines commands and matches responses back to tags.)

## POP3 Commands

| Command | Description |
|---|---|
| `USER username` | Identify the user |
| `PASS password` | Authenticate |
| `STAT` | Number of saved emails |
| `LIST` | Number + size of all emails |
| `RETR id` | Retrieve email by ID |
| `DELE id` | Delete email by ID |
| `CAPA` | Show server capabilities |
| `QUIT` | Close the connection |

## Dangerous Dovecot Settings

| Setting | Risk |
|---|---|
| `auth_debug` | Logs all authentication debug info |
| `auth_debug_passwords` | Logs submitted passwords + scheme |
| `auth_verbose` | Logs failed auth attempts + reasons |
| `auth_verbose_passwords` | Logs passwords used in auth attempts |
| `auth_anonymous_username` | Defines the username used for ANONYMOUS SASL login |

## Footprinting with Nmap

```bash
sudo nmap <TARGET> -sV -p110,143,993,995 -sC
```

On a TLS-enabled port, Nmap's `ssl-cert` script dumps the server's embedded certificate — a direct source of **organization name, location, and FQDN** (`commonName=mail1.inlanefreight.htb`, `organizationName=Inlanefreight`, etc.), plus `pop3-capabilities`/`imap-capabilities` listing supported extensions (`STARTTLS`, `SASL`, `PIPELINING`, ...).

## Authenticated Interaction — `curl`

```bash
curl -k 'imaps://<TARGET>' --user <user>:<password>
```
`-k` ignores a self-signed/untrusted cert. Lists mailbox folders directly. Add `-v` for verbose mode — shows the full TLS handshake, the negotiated TLS version, the server's certificate details, **and the server banner** (often reveals the mail server's version, e.g. `HTB-Academy IMAP4 v.0.21.4`).

## Manual TLS Interaction — `openssl s_client`

```bash
openssl s_client -connect <TARGET>:pop3s
openssl s_client -connect <TARGET>:imaps
```
Opens a raw TLS session you can then drive manually with the IMAP/POP3 commands above (e.g. `A001 LOGIN user pass`) — same idea as `telnet`/`nc` for plaintext protocols, but TLS-wrapped. The handshake output alone reveals the full certificate chain (org, FQDN, validity dates) before you even authenticate.

## Credential Reuse

As with FTP/SMB/SMTP earlier in this module, a username discovered via one service (e.g. SMTP `VRFY` enumeration) is always worth testing against every other exposed auth-gated service — password reuse (including trivial cases like username-as-password) is a recurring, high-yield finding across protocols.

## Skills Practiced

- Distinguishing IMAP's server-side mailbox model from POP3's simpler download-and-delete model
- Identifying dangerous Dovecot auth-logging settings
- Nmap-based IMAP/POP3 footprinting, extracting org/FQDN from TLS certificates
- Authenticated IMAP/POP3 interaction via `curl` (`-k`, `-v`) and manual TLS sessions via `openssl s_client`
- Applying credentials recovered from one service (SMTP) against another (IMAP/POP3) — credential reuse testing

## Key Takeaways

- A TLS certificate on a mail service is itself a footprinting goldmine — `commonName`/`organizationName`/`emailAddress` fields routinely leak the real FQDN, company name, and even an employee email address, all before authentication.
- `curl -v` against an IMAPS/POP3S endpoint is a fast way to get both the TLS/cert details *and* the server banner/version in one shot, without needing a manual `openssl s_client` session.
- IMAP's richer command set (folders, multi-client sync) versus POP3's minimal one reflects two different usage models (server-hosted mailbox vs. local download-and-delete) — this affects what's worth enumerating once authenticated (IMAP has actual folder structure to explore; POP3 is just a flat list).
- Credential reuse keeps being the thread connecting sections in this module — a username found via SMTP enumeration becomes the first thing worth testing against IMAP/POP3, SSH, or any other exposed auth.
