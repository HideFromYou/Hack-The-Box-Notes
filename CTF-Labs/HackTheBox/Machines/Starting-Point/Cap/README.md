# Cap

## Overview

**Cap** — HTB Starting Point (Tier 2), Easy-rated Linux box. Chains an IDOR on a web app's file-download endpoint to pull a downloadable `.pcap` capture, plaintext FTP credentials recovered from that capture, SSH access with the reused credentials, and a `python3.8` binary with the `cap_setuid` Linux capability set, abused directly for root.

## Learning Objectives

- Identify an IDOR on a resource-download endpoint by manipulating a URL path parameter
- Extract plaintext credentials from a `.pcap` file using Wireshark's **Follow TCP Stream**
- Recognize credential reuse across services (FTP creds reused for SSH)
- Enumerate Linux capabilities on binaries (`getcap`) and abuse `cap_setuid` for privilege escalation

## Enumeration

```bash
nmap -sV -sC <target-IP>
```
Open ports: 21 (FTP), 22 (SSH, needs credentials), 80 (HTTP).

Browsing to `http://<target-IP>` lands directly on a SOC-themed dashboard, auto-logged-in as a user `nathan` — no login screen, no credentials required for this initial access level.

## Broken Access Control (IDOR via URL Path)

The dashboard lists a captured `.pcap` file, downloadable for analysis. The download URL encodes an index as part of the path (e.g. `/data/<n>`). Changing that index directly in the browser (e.g. `/data/0`) is enough to access a different, non-owned capture file — a path-parameter **IDOR** (OWASP A01:2021 — Broken Access Control), same underlying flaw class as Oopsie's cookie-based IDOR but on a path parameter instead of a cookie value: the server trusts the client-supplied reference with no ownership check.

## Credential Disclosure — pcap Analysis

Downloaded the `.pcap` and opened it in Wireshark:

```
File → Open <capture>.pcap
```
Filtered for HTTP traffic and used **Follow → TCP Stream** on the login POST request, which revealed the full HTTP login exchange in plaintext — including a username/password pair sent with no encryption at all.

## Foothold — SSH with Reused Credentials

The open FTP port and the recovered credentials pointed at the same reuse pattern as Oopsie's DB-config bug: creds captured from one plaintext channel (here, an HTTP login in a pcap) turned out valid for a completely different service.

```bash
ssh nathan@<target-IP>
```
Logged in directly as `nathan` with the recovered password — no exploitation needed beyond the credential itself. `user.txt` readable from `nathan`'s home directory.

## Privilege Escalation — Linux Capabilities Abuse

Enumerated binaries with elevated Linux capabilities:

```bash
getcap -r / 2>/dev/null
```
Found `/usr/bin/python3.8` with `cap_setuid+ep` set. Linux capabilities let a binary hold a specific subset of root's privileges (here, the power to change the process's UID) without needing the full SUID-root bit — but a capability like `cap_setuid` on an interpreter as flexible as Python is just as dangerous as SUID-root, since the interpreter can execute arbitrary code under that privilege.

```bash
python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
`os.setuid(0)` succeeds because the capability grants exactly that permission, and the subsequent `/bin/bash` spawn inherits UID 0. Result: root shell, `root.txt` readable.

See `07-Privilege-Escalation/Linux Capabilities (getcap) Privilege Escalation.txt` in the cheatsheet for the technique generalized beyond this specific box.

## Skills Practiced

- Path-parameter IDOR identification and exploitation
- Wireshark traffic analysis — Follow TCP Stream for plaintext credential recovery
- Recognizing credential reuse across unrelated services (HTTP login → SSH)
- Linux capability enumeration (`getcap -r /`) and `cap_setuid` abuse for root

## Key Takeaways

- IDOR isn't limited to cookies or query parameters — any client-controlled reference in a URL (including a path segment) needs a server-side ownership check, or it's exploitable the same way.
- A `.pcap` handed to you as "evidence" or a "dashboard feature" is itself an enumeration target — plaintext protocols inside it (HTTP, FTP, Telnet) routinely leak live credentials.
- Credential reuse is the recurring theme across both Starting Point boxes solved so far (Oopsie's DB-config password, Cap's sniffed HTTP login) — always test recovered credentials against every other open service, not just the one they came from.
- Linux capabilities are a finer-grained alternative to full root/SUID, but a powerful capability (`cap_setuid`, `cap_setgid`, `cap_dac_override`, etc.) on a general-purpose interpreter (Python, Perl, Ruby) is functionally equivalent to SUID-root — always check `getcap -r /` alongside the usual SUID search.
