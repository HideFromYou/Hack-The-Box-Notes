# SMTP

## Overview

This lesson covers SMTP protocol fundamentals, the mail flow pipeline (MUA → MSA → MTA → MDA), Postfix default configuration, manual protocol interaction via `telnet`, user enumeration with `VRFY`, dangerous open-relay settings, and Nmap-based footprinting.

## Learning Objectives

- Understand SMTP's role, ports, and the MUA/MSA/MTA/MDA mail flow
- Read a Postfix `main.cf` and know the core SMTP commands
- Manually interact with an SMTP server via `telnet` (`HELO`/`EHLO`, send an email)
- Enumerate valid usernames via `VRFY`
- Identify the open relay misconfiguration and footprint it with Nmap

## Protocol Fundamentals

SMTP (Simple Mail Transfer Protocol) sends/relays email — it does not handle retrieval (that's IMAP/POP3). Client-server based, but a server relaying to another server acts as a client itself.

**Ports:**

| Port | Use |
|---|---|
| 25 | Default, plaintext |
| 587 | Authenticated client submission, upgraded via `STARTTLS` |
| 465 | Implicit TLS (SMTPS) |

SMTP is plaintext by default — commands, data, and auth are all sent in the clear unless SSL/TLS is layered on. **ESMTP** (what people usually mean by "SMTP" today) extends the protocol with `STARTTLS` (sent after `EHLO` to upgrade the connection to TLS) and `AUTH PLAIN` for safe authentication once encrypted.

**Mail flow pipeline:**

```
Client (MUA) → Submission Agent (MSA) → Relay (MTA) → Mail Delivery Agent (MDA) → Mailbox (POP3/IMAP)
```

- **MUA** (Mail User Agent) — the client (Thunderbird, Outlook, or a raw `telnet` session); converts the message into header + body and uploads it
- **MTA** (Mail Transfer Agent) — the server software that checks size/spam, stores, and looks up the recipient's mail server in DNS
- **MSA** (Mail Submission Agent) — sits in front of the MTA to check the email's origin/validity, relieving the MTA; also called a **Relay server** — misconfiguration here enables the **Open Relay Attack**
- **MDA** (Mail Delivery Agent) — reassembles the email at the destination server and delivers it to the recipient's mailbox

**Inherent protocol weaknesses:**

- No usable delivery confirmation — only an unstructured plaintext error on failure
- No sender authentication at connection time — the sender is unreliable, enabling **mail spoofing** and open-relay spam abuse. Modern mitigations: DKIM (sender identification) and SPF (Sender Policy Framework)

## Postfix Default Configuration

```shellsession
cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"
```

Key fields to know:

| Field | Meaning |
|---|---|
| `smtpd_banner` | Banner string shown on connect |
| `myhostname` | This server's hostname |
| `mydestination` | Domains this server accepts mail for |
| `mynetworks` | **Trusted networks allowed to relay** — the open relay risk lives here |
| `home_mailbox` | Mailbox storage path |

## SMTP Commands

| Command | Description |
|---|---|
| `AUTH PLAIN` | Authenticate the client |
| `HELO`/`EHLO` | Client identifies itself and starts the session (`EHLO` also requests extended/ESMTP capabilities) |
| `MAIL FROM` | Names the sender |
| `RCPT TO` | Names the recipient |
| `DATA` | Begin the email body, terminated by a single `.` on its own line |
| `RSET` | Abort the transmission, keep the connection open |
| `VRFY` | Check if a mailbox exists |
| `EXPN` | Check if a mailbox/mailing-list exists |
| `NOOP` | Keepalive, prevents timeout disconnect |
| `QUIT` | End the session |

## Manual Interaction — HELO/EHLO

```shellsession
telnet <TARGET> 25
HELO mail1.inlanefreight.htb
EHLO mail1
```

`EHLO` returns the extended capability list (`PIPELINING`, `SIZE`, `8BITMIME`, etc.) — useful for fingerprinting and confirming what the server supports.

## Username Enumeration — VRFY

```shellsession
telnet <TARGET> 25
VRFY root
VRFY cry0l1t3
VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

Response code **252** means "cannot verify, but will accept and attempt delivery" — depending on configuration, the server may return 252 for *every* name, including nonexistent ones, so `VRFY` results aren't always reliable on their own. Never trust a single automated tool's output blindly here — check manually, since no tool knows exactly how a given admin configured the target.

(Through a web proxy, the same connection can be tunneled with `CONNECT <TARGET>:25 HTTP/1.0`.)

## Sending an Email Manually

```shellsession
telnet <TARGET> 25
EHLO inlanefreight.htb
MAIL FROM: <cry0l1t3@inlanefreight.htb>
RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure
DATA
From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200

Hey man, I am trying to access our XY-DB but the creds don't work.
Did you make any changes there?
.
QUIT
```

The mail **header** carries sender/recipient, timestamps, and the relay hops the email passed through (structure defined by RFC 5322) — valuable OSINT if you can read a target's sent/received mail.

## Dangerous Settings — Open Relay

Admins often don't have a clear picture of which IP ranges they truly need to allow, so to avoid breaking legitimate mail flow they over-permit:

```
mynetworks = 0.0.0.0/0
```

This lets the server relay mail from **anyone** — enabling spoofed-sender spam and email reading/interception via the relay. This is a common finding on both internal and external pentests.

## Footprinting with Nmap

```shellsession
sudo nmap <TARGET> -sC -sV -p25
sudo nmap <TARGET> -p25 --script smtp-open-relay -v
```

- `smtp-commands` (part of default `-sC`) — sends `EHLO` and lists every command the target SMTP server supports
- `smtp-open-relay` — runs 16 different relay tests (varied `MAIL FROM`/`RCPT TO` formats) and reports how many succeed — `16/16 tests` passing confirms a fully open relay

## Skills Practiced

- Mapping SMTP's MUA/MSA/MTA/MDA mail flow and where the Open Relay Attack fits in
- Reading Postfix `main.cf` for relevant/dangerous settings (`mynetworks` especially)
- Manual SMTP interaction via `telnet` — session start, sending a full email, `VRFY` enumeration
- Nmap-based SMTP footprinting (`smtp-commands`, `smtp-open-relay`)

## Key Takeaways

- `VRFY`'s 252 response code is ambiguous by design ("cannot verify, but will try") — some servers return it for nonexistent users too, so automated enumeration results must be sanity-checked manually rather than trusted outright.
- Open relay (`mynetworks = 0.0.0.0/0`) is usually a convenience misconfiguration, not an intentional feature — admins over-permit to avoid blocking legitimate mail and end up allowing anyone to relay/spoof through the server.
- A raw `telnet` session can fully substitute for a mail client — `EHLO` → `MAIL FROM` → `RCPT TO` → `DATA` → `.` → `QUIT` is the entire protocol needed to send mail by hand.
- Mail headers (RFC 5322) are a secondary OSINT source beyond the message body — hop history and timestamps can reveal internal infrastructure.
