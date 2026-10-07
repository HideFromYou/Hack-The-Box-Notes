# Academy Modules Layout

## Overview

Maps the full HTB Academy module catalog onto each stage of the penetration testing process, and explains the history and learning philosophy behind how Academy content is structured.

## Learning Objectives

- Understand the history of HTB Academy and how it complements the competitive HTB platform
- Understand why Academy modules are deliberately structured to build deep competence rather than surface-level familiarity
- Map Academy modules (36 total) to the stages of the penetration testing process
- Distinguish the 28-module CPTS Job Role Path from the broader Academy catalog

## History and Philosophy

HTB started as a purely competitive CTF platform — black-box, no guidance, not beginner friendly. HTB Academy was created to add guided learning alongside the competitive side, with Starting Point acting as a bridge from guided learning into independent play.

Cybersecurity requires broad IT fundamentals (networking, Linux/Windows administration, scripting, databases) — it's not possible to be a confident pentester without understanding the technologies being assessed. Academy modules are deliberately structured to feel hard at first, because that structure builds the deepest, most efficient path to competence — similar to needing extensive hands-on practice to play an instrument well, not just knowing facts about it.

## Academy Module Catalog Mapped to the Pentest Process

The full Academy catalog spans 36 modules, broader than just the 28-module CPTS Job Role Path:

| Pentest Stage | Academy Modules |
|---|---|
| Information Gathering — foundations | Learning Process, Linux Fundamentals, Windows Fundamentals, Introduction to Networking, Introduction to Web Applications, Web Requests, JavaScript Deobfuscation, Introduction to Active Directory, Getting Started |
| Information Gathering — stage modules | Network Enumeration with Nmap, Footprinting, Information Gathering - Web Edition, OSINT: Corporate Recon |
| Vulnerability Assessment | Vulnerability Assessment, File Transfers, Shells & Payloads, Using the Metasploit Framework |
| Exploitation | Password Attacks, Attacking Common Services, Pivoting/Tunneling/Port Forwarding, Active Directory Enumeration & Attacks |
| Web Exploitation | Using Web Proxies, Attacking Web Applications with Ffuf, Login Brute Forcing, SQL Injection Fundamentals, SQLMap Essentials, XSS, File Inclusion, Command Injections, Web Attacks, Attacking Common Applications |
| Post-Exploitation | Linux Privilege Escalation, Windows Privilege Escalation |
| Lateral Movement | No dedicated module — folded into Getting Started, Linux/Windows PrivEsc, and others, since it's iterative rather than a one-time phase |
| Proof-of-Concept | Introduction to Python 3 (for automating PoC steps) |
| Post-Engagement | Documentation & Reporting, Attacking Enterprise Networks (capstone) |

## CPTS Path vs. the Broader Catalog

The 28-module CPTS Job Role Path checklist is a curated subset of this broader 36-module catalog. The extra ~8 modules listed above (Learning Process, Linux/Windows Fundamentals, Intro to Networking, Intro to Web Apps, Web Requests, JS Deobfuscation, Intro to AD, OSINT: Corporate Recon, Intro to Python 3) are prerequisite/supporting modules from other paths (e.g., Information Security Foundations). They're referenced here for context but aren't part of the official 28-module path tracker.

## Skills Practiced

- Mapping unfamiliar course content to a known process framework
- Distinguishing core-path modules from prerequisite/supporting modules

## Key Takeaways

- Academy's deliberately steep early learning curve is a design choice, not a flaw — depth over quick wins builds more durable competence.
- Lateral Movement and Pillaging have no standalone module on purpose: treating them as skills practiced repeatedly across many modules more closely matches how they're actually used in a real engagement.
- The 28-module CPTS path sits inside a larger 36-module catalog — the extra modules are prerequisites worth knowing about even though they aren't tracked in the CPTS checklist.
