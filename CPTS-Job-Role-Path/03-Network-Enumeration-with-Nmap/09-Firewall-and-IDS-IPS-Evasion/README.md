# Firewall and IDS/IPS Evasion

## Overview

Firewalls, IDS, and IPS systems actively shape what a scan can see and how much attention it draws. This lesson covers determining firewall rules, using ACK scans to probe filtering, decoy scanning, custom source IP/interface tricks, DNS proxying, and source-port manipulation to bypass loosely configured firewall rules.

## Learning Objectives

- Distinguish a firewall, an IDS, and an IPS by role
- Determine whether a filtered port is dropped or rejected, and what that implies about the firewall rule
- Use an ACK scan alongside a SYN scan to confirm filtering state
- Use decoy scanning, custom source IP/interface, DNS proxying, and source-port manipulation to evade firewall rules

## Firewall, IDS, and IPS

- **Firewall** — software/hardware that monitors traffic and decides pass/drop/block per rule as traffic crosses it.
- **IDS** — passive monitoring; analyzes traffic via pattern-matching/signatures and alerts an admin.
- **IPS** — complements an IDS by actively taking defensive action (e.g. blocking) when a potential attack pattern is detected, such as a service-detection scan.

## Determining Firewall Rules

Filtered ports can be **dropped** (silently ignored, no response) or **rejected** (an explicit response — a TCP RST, or an ICMP error). ICMP error types seen for rejected packets include: Net Unreachable, Net Prohibited, Host Unreachable, Host Prohibited, Port Unreachable, and Proto Unreachable.

## ACK Scan (-sA) vs. SYN Scan (-sS) for Firewall Detection

An ACK scan is much harder for firewalls/IDS to filter than SYN or Connect scans, because firewalls typically block incoming SYN packets (new connection attempts from outside) but often pass ACK packets through — it's hard to tell whether a connection was already established from the inside vs. the outside. An ACK scan can **only** tell filtered vs. unfiltered — it cannot determine open vs. closed on its own, since a closed-but-reachable port also returns an RST.

```bash
sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace
sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace
```

Comparing SYN vs. ACK results side by side on the same ports distinguishes three cases:

- A port returning SYN-ACK (open) on the SYN scan, and RST on the ACK scan (unfiltered) — confirmed open **and** confirmed passing the firewall.
- A port returning ICMP-unreachable on both scans — confirmed filtered either way.
- A port with no response on either scan — confirmed dropped either way.

## Detecting IDS/IPS

Detecting an IDS/IPS is much harder since they're passive by nature. An IDS alerts the admin on a matching pattern; an IPS blocks automatically. It's recommended to test from a VPS with a disposable IP — if the admin blocks that IP after a detected "attack" pattern (e.g. an aggressive single-port scan), that confirms active monitoring is present, and signals a need to be quieter, disguise further activity, and switch source IPs.

## Decoy Scanning (-D)

Decoy scanning generates several random spoofed IPs inserted into the IP header alongside the real IP, placed randomly among them, to disguise which one is the real attacker.

```bash
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

`-D RND:5` generates 5 random decoy IPs plus the real one, mixed in.

Decoys should be alive — otherwise SYN-flood protections on the target may block the scan. Spoofed packets are often filtered by ISPs/routers regardless, even from within the same subnet range, so it's sometimes better to specify real VPS IPs under your control as decoys, combined with IP ID manipulation.

## Custom Source IP (-S) + Interface (-e)

Useful when only specific subnets lack access, to test whether another source IP gets better results. Decoys and custom-source both work with SYN, ACK, and ICMP scans, as well as OS detection scans.

```bash
sudo nmap 10.129.2.28 -n -Pn -p445 -O                              # default source — filtered
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0     # spoofed source — open!
```

## DNS Proxying (--dns-server)

By default Nmap does reverse DNS resolution over UDP port 53 — TCP 53 is historically used only for zone transfers or responses over 512 bytes, though IPv6/DNSSEC increasingly use TCP too. A trusted DNS server can be specified explicitly, useful in a DMZ scenario where the company's internal DNS is more trusted than external resolvers.

```bash
sudo nmap <target> --dns-server <ns1>,<ns2>
```

## Source Port Manipulation (--source-port)

If a firewall trusts traffic from a specific source port — port 53/DNS is commonly treated as "trusted" by loosely configured firewalls, since admins assume it's a legitimate DNS response — scan traffic can be made to appear to originate from that port.

```bash
# filtered, no response
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace

# open! SYN-ACK received
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

Once confirmed working, a manual connection can be made through this "trusted" port using `ncat`:

```bash
ncat -nv --source-port 53 10.129.2.28 50000
```

This returned a banner (`220 ProFTPd`) that the firewall would otherwise have blocked a normal connection attempt from reaching.

## Labs

This module includes 3 practical labs (Easy / Medium / Hard) that apply these evasion techniques against progressively stricter IDS/IPS configurations.

## Skills Practiced

- Classifying firewall rules as dropping vs. rejecting traffic
- Using paired SYN/ACK scans to confirm filtering state
- Applying decoy scanning, custom source IP/interface, DNS proxying, and source-port manipulation as evasion techniques
- Recognizing IDS/IPS presence through behavioral testing from a disposable source

## Key Takeaways

- A SYN scan alone can't confirm whether a firewall is actively filtering a port versus the port simply being closed — pairing it with an ACK scan on the same ports resolves that ambiguity.
- Loosely configured firewalls often trust traffic by superficial signals — a "well-known" source port like 53 — rather than verifying the traffic actually belongs to that service, which is exactly the misconfiguration `--source-port` exploits.
- Decoy scanning only helps if the decoys are plausible and alive; dead or obviously spoofed decoys can trigger the very SYN-flood protections they're meant to avoid.
- Testing from a disposable VPS IP is a safe way to probe for active IPS blocking without burning a primary operating address.
