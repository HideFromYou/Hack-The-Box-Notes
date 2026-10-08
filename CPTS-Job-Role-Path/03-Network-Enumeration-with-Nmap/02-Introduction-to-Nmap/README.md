# Introduction to Nmap

## Overview

Nmap (Network Mapper) is an open-source network analysis and security auditing tool. It scans networks to find live hosts via raw packets, identifies services and application versions, can detect the operating system, and can determine firewall/IDS configuration.

## Learning Objectives

- Understand Nmap's core architecture and scan technique groups
- Learn the basic Nmap syntax and available scan types
- Understand how a TCP SYN scan works at the packet level

## What Nmap Does

Nmap is written in C, C++, Python, and Lua. Typical use cases include:

- Security auditing
- Penetration test simulation
- Firewall/IDS configuration checking
- Connection-type testing
- Network mapping
- Response analysis
- Open-port identification
- Vulnerability assessment

**Architecture / scan technique groups:** host discovery, port scanning, service enumeration/detection, OS detection, and scriptable interaction (the Nmap Scripting Engine).

## Basic Syntax

```bash
nmap <scan types> <options> <target>
```

## Scan Techniques

Scan types available, per `nmap --help`:

```
-sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
-sU: UDP Scan
-sN/sF/sX: TCP Null, FIN, and Xmas scans
--scanflags <flags>: Customize TCP scan flags
-sI <zombie host[:probeport]>: Idle scan
-sY/sZ: SCTP INIT/COOKIE-ECHO scans
-sO: IP protocol scan
-b <FTP relay host>: FTP bounce scan
```

## TCP-SYN Scan (-sS)

The SYN scan is Nmap's default scan type when run as root, since it requires raw-socket permissions. If not run as root, Nmap falls back to a Connect scan (`-sT`).

A SYN scan sends only a SYN packet and never completes the full TCP three-way handshake:

- **SYN-ACK** received back → port is **open**.
- **RST** received back → port is **closed**.
- **No response** → port is **filtered** (a firewall may be dropping or ignoring the packet).

```bash
sudo nmap -sS localhost
```

This shows the open ports with PORT / STATE / SERVICE columns.

## Skills Practiced

- Reading and constructing basic Nmap command syntax
- Identifying which scan type is active by default based on privilege level
- Interpreting SYN/RST/no-response outcomes of a raw SYN scan

## Key Takeaways

- Nmap's default behavior silently changes based on privilege level (`-sS` as root, `-sT` otherwise) — always confirm which scan type actually ran.
- A SYN scan's three possible outcomes (SYN-ACK, RST, silence) map directly to open, closed, and filtered — understanding this at the packet level is the foundation for every later scan type in this module.
- Nmap's five technique groups (host discovery, port scanning, service enumeration, OS detection, NSE) are the mental model for organizing everything else in this module.
