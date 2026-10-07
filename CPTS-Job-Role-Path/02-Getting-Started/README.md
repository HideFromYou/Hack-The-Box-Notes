# Getting Started

## Overview

Notes from the Hack The Box Academy **Getting Started** module — the fundamentals of penetration testing and an introduction to the Hack The Box platform. Covers infosec basics, pentest distro setup, enumeration, public exploits, shells, privilege escalation, file transfers, platform navigation, a full guided box walkthrough (Nibbles), common pitfalls, and a final independent skills assessment.

## Lessons

| # | Topic |
|---|---|
| 01 | Infosec Overview |
| 02 | Getting Started with a Pentest Distro |
| 03 | Staying Organized |
| 04 | Connecting Using VPN |
| 05 | Common Terms |
| 06 | Basic Tools |
| 07 | Service Scanning |
| 08 | Web Enumeration |
| 09 | Public Exploits |
| 10 | Types of Shells |
| 11 | Privilege Escalation |
| 12 | Transferring Files |
| 13 | Starting Out |
| 14 | Navigating HTB |
| 15 | Nibbles: Enumeration |
| 16 | Nibbles: Web Footprinting |
| 17 | Nibbles: Initial Foothold |
| 18 | Nibbles: Privilege Escalation |
| 19 | Nibbles: Alternate User Method - Metasploit |
| 20 | Common Pitfalls |
| 21 | Getting Help |
| 22 | Next Steps |
| 23 | Knowledge Check |

## Skills Practiced

- Nmap scanning (default, version, script, full-port)
- Web enumeration (Gobuster, DNS brute force, fingerprinting)
- Public exploit research (searchsploit, Metasploit, Exploit-DB)
- Reverse shells, bind shells, web shells, TTY upgrading
- SMB, FTP, SNMP manual enumeration
- Linux privilege escalation (sudo misconfigurations, cron/scheduled task abuse, SSH key abuse, kernel/software exploits)
- File transfer techniques (HTTP server + wget/curl, scp, base64 encode/decode)
- CMS exploitation (Nibbleblog file upload, GetSimple theme-editor RCE)
- Metasploit Framework module search, configuration, and execution against a real target
- GTFOBins-driven exploitation of sudo-permitted binaries and interpreters
- HTB platform navigation (Tracks, Machines, Challenges, Fortress, Endgame, Pro Labs) and community help-seeking practices

## Tools Used

- Nmap, Gobuster, Subfinder, Whatweb, cURL
- searchsploit, Metasploit Framework, wpscan
- Netcat, SSH, Tmux, Vim, smbclient, snmpwalk
- LinEnum, PEASS/linpeas, GTFOBins, LOLBAS
- Python3 (HTTP server, pty TTY upgrade), scp, base64, xmllint

## Key Takeaways

- Enumeration depth matters more than any single tool — passive and active scans can surface different results for the same target.
- Cross-reference exploit advisories against the exact installed version before assuming a PoC will work as-is.
- A reliable shell (and knowing how to upgrade/stabilize it) is the foundation everything else in a pentest builds on.
- A sudo rule or writable file found during enumeration is only a lead until it's checked against GTFOBins/LOLBAS or the referencing cron job/sudoers entry — that's what turns it into an actual root shell.
- When one attack surface is broken or filtered, a different feature in the same application can often reach the same underlying goal (arbitrary code write) through an entirely different, unfiltered path.
- Manual exploitation and Metasploit's automated modules are worth knowing side by side — they often land on the same user, but only the manual path builds a real understanding of *why* it works.
- Completing this module end-to-end, down to an independent knowledge-check target with no walkthrough, is the actual marker of readiness for the next stage of the CPTS path — not just reading through the lessons.
