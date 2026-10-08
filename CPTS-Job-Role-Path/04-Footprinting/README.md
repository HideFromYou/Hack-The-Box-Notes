# Footprinting

## Overview

Notes from the Hack The Box Academy **Footprinting** module. This module is a checkpoint — **sections 1-9 of 21 are documented so far**. Completed sections cover the enumeration mindset and methodology, passive OSINT techniques (domain information, cloud resources, staff), and active service footprinting for FTP, SMB, NFS, and DNS. Sections 10-21 (additional services covered by the module) are still pending.

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
| 10-21 | Remaining Footprinting sections | ⬜ pending |

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

## Tools Used

- dig
- crt.sh / curl / jq
- Shodan
- FTP / TFTP clients
- smbclient / rpcclient / enum4linux-ng / smbmap / crackmapexec / samrdump.py
- NFS mount / showmount
- dnsenum

## Key Takeaways

- Enumeration's value comes from breadth and iteration, not speed — the methodology (6 layers, 3 levels) exists precisely to keep findings organized instead of jumping straight to credential attacks.
- Fully passive OSINT (certificate transparency, Shodan, Google dorking, employee research) can surface a surprising amount — subdomains, exposed cloud storage, even leaked credentials — before a single active scan is run.
- Anonymous/null-session access is a recurring theme across FTP, SMB, and NFS: each protocol has a "no real auth" mode that, left misconfigured, leaks far more than intended.
- DNS zone transfer (AXFR) misconfiguration remains one of the highest-impact single findings possible in footprinting — a successful pull can hand over an entire internal network map.
- Cross-checking results across multiple tools (e.g. SMB's `smbclient`/`rpcclient`/enum4linux-ng/CrackMapExec) consistently surfaces information that any single tool missed.
- This checkpoint covers only Layer 3 (Accessible Services) territory for FTP/SMB/NFS/DNS — the remaining 12 sections of this module cover additional services and are not yet documented.
