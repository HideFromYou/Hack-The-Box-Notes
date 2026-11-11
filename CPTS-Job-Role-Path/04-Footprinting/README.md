# Footprinting

## Overview

Notes from the Hack The Box Academy **Footprinting** module. **All 21 sections are completed.** The module covered enumeration principles and methodology, passive OSINT, infrastructure- and host-based enumeration, manual interaction with common network services, and practical Easy, Medium, and Hard labs.

## Lessons

| # | Topic | Status |
|---|-------|--------|
| 01 | Enumeration Principles | ✅ |
| 02 | Enumeration Methodology | ✅ |
| 03 | Domain Information | ✅ |
| 04 | Cloud Resources | ✅ |
| 05 | Staff | ✅ |
| 06 | FTP | ✅ |
| 07 | SMB | ✅ |
| 08 | NFS | ✅ |
| 09 | DNS | ✅ |
| 10 | SMTP | ✅ |
| 11 | IMAP / POP3 | ✅ |
| 12 | SNMP | ✅ |
| 13 | MySQL | ✅ |
| 14 | MSSQL | ✅ |
| 15 | Oracle TNS | ✅ |
| 16 | IPMI | ✅ |
| 17 | [Linux Remote Management Protocols](17-Linux-Remote-Management-Protocols/) | ✅ |
| 18 | [Windows Remote Management Protocols](18-Windows-Remote-Management-Protocols/) | ✅ |
| 19 | [Footprinting Lab — Easy](19-Footprinting-Lab-Easy/) | ✅ |
| 20 | [Footprinting Lab — Medium](20-Footprinting-Lab-Medium/) | ✅ |
| 21 | [Footprinting Lab — Hard](21-Footprinting-Lab-Hard/) | ✅ |

## Skills Practiced

- Applying the enumeration mindset (active + passive, iterative, "what can/can't we see") before touching exploitation
- Mapping findings to the 6-layer enumeration methodology (Infrastructure/Host/OS-based levels)
- Passive domain reconnaissance via certificate transparency logs (crt.sh), Shodan, and DNS TXT records
- Passive cloud storage discovery via Google dorking, source code inspection, domain.glass, and GrayHatWarfare
- Staff/OSINT reconnaissance via job postings, employee GitHub repos, and LinkedIn advanced search
- Anonymous FTP enumeration, manual protocol interaction, and Nmap NSE footprinting
- SMB share/user/group enumeration via `smbclient`, `rpcclient`, RID brute-forcing, and automated tools
- NFS export enumeration and mounting, with attention to UID/GID trust weaknesses
- DNS footprinting via SOA/ANY queries, zone transfer (AXFR) attempts, and subdomain brute-forcing
- SMTP mail-flow mapping (MUA/MSA/MTA/MDA), manual protocol interaction, `VRFY` user enumeration, and open relay identification
- IMAP/POP3 footprinting via Nmap TLS cert extraction, authenticated interaction via `curl`/`openssl s_client`, and credential reuse testing across services
- SNMP footprinting via community strings, `snmpwalk` OID queries, and `onesixtyone` community string brute-forcing
- MySQL footprinting via Nmap `mysql*` NSE scripts, manual verification of NSE false positives, and direct `mysql` CLI database/table enumeration
- MSSQL footprinting via Nmap `ms-sql-*` NSE scripts, Metasploit's `mssql_ping`, and authenticated T-SQL interaction via Impacket's `mssqlclient.py`
- Oracle TNS footprinting via Nmap SID brute-forcing, ODAT credential brute-forcing, and authenticated `sqlplus` interaction including `sysdba` access
- IPMI footprinting via Nmap's `ipmi-version` NSE script and Metasploit IPMI scanners; RAKP hash retrieval and offline cracking with Hashcat mode 7300
- Chained enumeration across SNMP, IMAPS, SSH, Linux user artifacts, and MySQL in the Hard lab
- Manual IMAP interaction over TLS using `openssl s_client`, mailbox/folder enumeration, and retrieval of an SSH key from a message
- Linux host enumeration through `.bash_history` and `.mysql_history` to identify previous user activity and database usage
- MySQL database, table, and column enumeration, followed by a targeted SQL query to identify a specific account record

## Tools Used

- dig
- crt.sh / curl / jq
- Shodan
- FTP / TFTP clients
- smbclient / rpcclient / enum4linux-ng / smbmap / crackmapexec / samrdump.py
- NFS mount / showmount
- dnsenum
- telnet / nc (manual SMTP/IMAP/POP3 interaction)
- smtp-user-enum
- openssl s_client
- snmpwalk / onesixtyone / braa
- mysql client
- Nmap ms-sql-* NSE scripts / Metasploit mssql_ping / Impacket mssqlclient.py
- ODAT / sqlplus
- Nmap `ipmi-version` NSE script
- Metasploit `auxiliary/scanner/ipmi/ipmi_version` and `auxiliary/scanner/ipmi/ipmi_dumphashes`
- Hashcat (mode 7300 — IPMI2 RAKP HMAC-SHA1)
- SSH client / ssh-keygen

## Practical Lab Notes

### Easy

Completed the Easy practical lab and applied the module's service-enumeration workflow to identify and investigate exposed services.

### Medium

Completed the Medium practical lab. The exercise reinforced enumeration of NFS and SMB, inspection of exposed files, credential discovery, and testing whether discovered credentials were reused across services. Database access and SQL queries were used to locate the target account record.

### Hard

Completed the Hard practical lab against a Linux host:

1. TCP and UDP Nmap scans identified SSH, POP3, IMAP/IMAPS, POP3S, and SNMP.
2. `onesixtyone` identified an SNMP community string; `snmpwalk` exposed host information and credentials for a local user.
3. Password-based SSH authentication was not accepted, so the investigation continued through IMAPS.
4. `openssl s_client` established a TLS connection to IMAPS. Manual IMAP commands enumerated mail folders and retrieved a message containing an SSH private key.
5. The key was saved with restrictive permissions (`chmod 600`) and used for SSH authentication.
6. After obtaining a shell, `.bash_history` and `.mysql_history` revealed prior MySQL activity and the database/table names.
7. The MySQL client was used to inspect the `users` database and query the target account.

**Security note:** Lab credentials, recovered passwords, private keys, flags, and target-specific secrets are intentionally excluded from this repository. Keep them only in the authorized lab environment.

## Key Takeaways

- Enumeration's value comes from breadth and iteration, not speed — the methodology (6 layers, 3 levels) exists precisely to keep findings organized instead of jumping straight to credential attacks.
- Fully passive OSINT (certificate transparency, Shodan, Google dorking, employee research) can surface a surprising amount — subdomains, exposed cloud storage, even leaked credentials — before a single active scan is run.
- Anonymous/null-session access is a recurring theme across FTP, SMB, and NFS: each protocol has a "no real auth" mode that, left misconfigured, leaks far more than intended.
- DNS zone transfer (AXFR) misconfiguration remains one of the highest-impact single findings possible in footprinting — a successful pull can hand over an entire internal network map.
- Cross-checking results across multiple tools (e.g. SMB's `smbclient`/`rpcclient`/enum4linux-ng/CrackMapExec) consistently surfaces information that any single tool missed.
- SMTP's `VRFY`/open-relay weaknesses mirror the same theme as FTP/SMB/NFS — protocol-level trust (in a claimed username, in a claimed relay network) left unchecked becomes a direct enumeration or abuse path.
- TLS certificates on mail services (IMAP/POP3) are an easy, pre-auth source of org name/FQDN/employee email — footprinting value hiding in plain sight in the handshake.
- Credential reuse across services (a username/password found via one protocol working on a completely different one) has been the single most recurring theme across this module.
- SNMP's community string is the same "shared secret pretending to be authentication" pattern seen in SMTP/mail services — v1/v2c has essentially no real access control, just an often-default (`public`) plaintext string.
- Automated NSE findings (MySQL's empty-password false positive) are a lead to verify manually, never a fact to trust outright — a theme that's now shown up with SMTP (VRFY false positives) and MySQL both.
- Default/well-known credentials left enabled (`sa` on MSSQL, `public`/`private` SNMP community strings, MySQL `root`, Oracle's `scott:tiger`/`dbsnmp:dbsnmp`) is the single most recurring root cause across this module's database/management-protocol sections.
- Low-privilege DB access is rarely the end state — re-authenticating with an elevated role (Oracle's `as sysdba`) when the account happens to be authorized is a direct escalation path worth checking after a successful DB login.
- IPMI expands footprinting beyond application and database services into hardware management. UDP/623 exposure can reveal BMC interfaces; IPMI 2.0 RAKP may expose password-derived authentication data that can be cracked offline. A cracked BMC credential can have significant impact because BMC access can enable remote hardware-level management.
- A low-privileged shell can expose valuable operational artifacts: shell history, application history, local mail stores, configuration files, and SSH material.
- A failed authentication method on one service does not invalidate the credential elsewhere. Pivot to other exposed services and test hypotheses based on evidence.
- The strongest practical lesson is to connect findings across services: SNMP can expose credentials, mail can expose a key, SSH can expose user history, and user history can reveal a path to a database.

---

**Status: Footprinting completed — 21/21.**
