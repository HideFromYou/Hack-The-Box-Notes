# Nibbles: Enumeration

## Overview

Starting a full walkthrough of **Nibbles**, a retired easy-rated Linux box, used to practice common enumeration tactics, basic web app exploitation, and a file-related privilege escalation misconfiguration. This lesson covers the initial reconnaissance phase: engagement type, port scanning, and banner grabbing.

## Learning Objectives

- Understand the difference between black-box, grey-box, and white-box engagements
- Run an initial Nmap service/version scan and interpret the results
- Run a full TCP port scan in the background while continuing enumeration
- Cross-check Nmap results with manual banner grabbing
- Use Nmap script scans and `http-enum` for extra web enumeration

## Engagement Type

Before scanning, it's worth noting the engagement type. This walkthrough is a **grey-box** engagement — the IP address and OS type are already known in advance, as opposed to black-box (nothing known beforehand) or white-box (full internal knowledge, including source code/credentials).

## Initial Nmap Scan

```bash
nmap -sV --open -oA nibbles_initial_scan <ip>
```

- `-sV` — service version detection
- `--open` — only show open ports
- `-oA` — output in all three formats (`.gnmap`, `.nmap`, `.xml`)

Result: port 22 (OpenSSH 7.2p2 Ubuntu) and port 80 (Apache, Ubuntu) open.

To see exactly what ports Nmap scans by default without even specifying a target:

```bash
nmap -v -oG - 
```

This prints the full port list Nmap would scan, in verbose/greppable stdout format (the scan itself fails since no target was given — it's purely informational).

## Full Port Scan

A default scan only checks the top 1,000 ports. A full TCP sweep catches anything running on a non-standard port:

```bash
nmap -p- --open -oA nibbles_full_tcp_scan <ip>
```

This is slow — kick it off in the background and keep enumerating with the results already in hand.

## Banner Grab Cross-Check

Manually cross-checking service banners is good practice and doesn't rely solely on Nmap's fingerprinting:

```bash
nc -nv <ip> 22
```

This shows the SSH banner directly. HTTP (port 80) doesn't expose anything useful this way — it needs an actual HTTP request, not a raw banner grab.

## Script Scan

```bash
nmap -sC -p 22,80 -oA nibbles_script_scan <ip>
```

Runs Nmap's default NSE scripts. In this case it didn't reveal much beyond the SSH host key and an empty HTTP title.

## HTTP Enumeration Script

```bash
nmap -sV --script=http-enum -oA nibbles_nmap_http_enum <ip>
```

`http-enum` checks for common web directories automatically. It didn't find anything extra here either — a reminder that NSE scripts are a convenient first pass, not a substitute for manual web enumeration.

## Skills Practiced

- Grey-box engagement scoping
- Nmap version scanning, full-port scanning, and NSE script scanning
- Manual banner grabbing with Netcat as a cross-check against automated tools

## Key Takeaways

- Running the slow full-port scan in the background, rather than waiting on it, keeps the overall enumeration pace up.
- Automated NSE scripts like `http-enum` are a quick first pass but came up empty here — that's a cue to move to manual web enumeration next, not a sign the target is clean.
- Cross-checking a service banner manually (via netcat) is a cheap sanity check against what `-sV` reports.
