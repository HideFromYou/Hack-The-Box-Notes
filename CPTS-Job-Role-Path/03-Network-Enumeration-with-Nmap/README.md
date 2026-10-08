# Network Enumeration with Nmap

## Overview

Notes from the Hack The Box Academy **Network Enumeration with Nmap** module. Covers the enumeration mindset, Nmap fundamentals, host discovery, TCP/UDP port scanning, saving and reporting scan results, service enumeration and manual banner verification, the Nmap Scripting Engine, performance tuning, and firewall/IDS/IPS evasion techniques — closing with three hands-on labs that apply the evasion toolkit against progressively hardened defenses.

## Lessons

| # | Topic |
|---|-------|
| 01 | Enumeration |
| 02 | Introduction to Nmap |
| 03 | Host Discovery |
| 04 | Host and Port Scanning |
| 05 | Saving the Results |
| 06 | Service Enumeration |
| 07 | Nmap Scripting Engine |
| 08 | Performance |
| 09 | Firewall and IDS/IPS Evasion |
| 10 | Firewall Evasion - Easy Lab |
| 11 | Firewall Evasion - Medium Lab |
| 12 | Firewall Evasion - Hard Lab |

## Skills Practiced

- Host discovery via ICMP and ARP, across ranges, file-based lists, and individual IPs
- TCP SYN, Connect, and ACK scanning, plus UDP scanning
- Diagnosing filtered ports as dropped vs. rejected using `--packet-trace` and `--reason`
- Service and version enumeration, cross-checked against raw banners via `tcpdump`/`nc`
- Saving and converting scan output (Normal, Grepable, XML, HTML via `xsltproc`)
- NSE scripting by category and by specific script name, including `vuln`-category CVE cross-referencing
- Performance tuning (RTT timeout, retries, rate, timing templates) with measured speed/accuracy tradeoffs
- Firewall/IDS/IPS evasion: decoy scanning, custom source IP/interface, DNS proxying, source-port manipulation
- Manual quiet-connection techniques (`ncat`/`nc`) as a substitute for automated tools that get blocked post-handshake

## Tools Used

- Nmap
- nc / ncat
- tcpdump
- xsltproc

## Key Takeaways

- **Scan type is a tradeoff, not a default to accept blindly.** SYN scans are faster and stealthier but never complete a real connection; Connect scans are fully accurate but loud and logged; ACK scans can't determine open/closed at all but are far harder for firewalls to block — picking the right one depends on what the engagement actually needs to know.
- **"Filtered" hides two different firewall behaviors.** A silently dropped port and an actively rejected port (ICMP unreachable, or an RST) imply different firewall configurations — `--packet-trace` and `--reason` are what actually distinguish them, not the reported state alone.
- **NSE's category system turns script selection into a deliberate choice, not noise.** Categories range from `safe`/`discovery` (low-risk recon) to `brute`/`dos`/`exploit`/`intrusive` (actively risky) — the right category depends on what the rules of engagement allow at that stage.
- **Automated tools can get blocked even after a connection succeeds.** The Hard lab's central lesson: getting a SYN-ACK past a firewall (even with `--source-port 53`) doesn't guarantee an automated version-detection probe (`-sV`) will get a response — the probe itself can look "noisy" enough at the application layer to get reset and reported as `tcpwrapped`. A manual, quiet `ncat`/`nc` connection that just opens and waits can succeed where the automated tool fails.
- **Evasion techniques stack.** No single technique (quiet SYN scan, UDP-awareness, source-port trust, manual quiet connections) was sufficient on its own by the Hard lab — each builds on lessons from the previous one, and real-world evasion against a hardened target usually requires combining several at once.
- **Manual verification remains the ground truth.** Nmap's own parsed/summarized output (service names, `-sV` version strings) can omit information a service actually sends — a raw `tcpdump` capture or a direct `nc`/`ncat` connection is what confirms exactly what's on the wire.
