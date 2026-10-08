# Host and Port Scanning

## Overview

Once a host is confirmed alive, the next step is determining its open ports and services, service versions, the information those services expose, and the operating system. This lesson covers the six Nmap port states, TCP SYN/Connect scanning, distinguishing dropped from rejected filtered ports, and UDP scanning.

## Learning Objectives

- Know all six Nmap port states and what each one means
- Compare SYN scans and Connect scans at the packet level
- Distinguish a dropped filtered port from a rejected filtered port using `--packet-trace`
- Perform UDP port scanning and interpret its ambiguous results

## Port States

| State | Meaning |
|---|---|
| open | Connection established (TCP/UDP/SCTP) |
| closed | TCP RST flag received — can also confirm host is alive |
| filtered | No response or error code — Nmap can't tell open vs. closed |
| unfiltered | Only occurs with TCP-ACK scan — port reachable but open/closed undetermined |
| open\|filtered | No response at all — may be a firewall-protected open port |
| closed\|filtered | Only occurs with IP ID idle scans — can't distinguish closed from filtered |

## Discovering Open TCP Ports

By default, Nmap scans the top 1000 TCP ports, using a SYN scan if run as root, or a Connect scan otherwise. Ports can be defined:

- One-by-one: `-p 22,25,80,139,445`
- By range: `-p 22-445`
- Top-N most common: `--top-ports=10`
- All 65535: `-p-`
- Fast top-100: `-F`

```bash
sudo nmap 10.129.2.28 --top-ports=10
```

## Tracing SYN Scan Packets

For a clean diagnostic view, disable ICMP ping (`-Pn`), DNS resolution (`-n`), and ARP ping (`--disable-arp-ping`):

```bash
sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping
```

The `SENT` line is our SYN packet (flag `S`). An `RCVD` line with `RA` flags (RST+ACK) means the port is closed and the session was terminated. Each SENT/RCVD line carries a timestamp, protocol, `src:port > dst:port`, flags, and header fields (ttl, id, iplen, seq, win, mss).

## Connect Scan (-sT)

A Connect scan completes the full TCP three-way handshake (SYN → SYN-ACK → ACK), establishing a full connection. It's highly accurate — it reports the exact open/closed/filtered state — but it is the **least stealthy** option: it creates logs and is easily detected by an IDS/IPS. It's sometimes called more "polite" since it behaves like a normal client and is less likely to cause service instability. A SYN/half-open scan is more stealthy since it never completes the handshake, but an advanced IDS/IPS can still detect it.

```bash
sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT
```

This shows `CONN` lines instead of raw `SENT`/`RCVD` lines, since it's a full socket connect rather than raw packet crafting. With `--reason`, it shows e.g. `REASON: syn-ack`.

## Filtered Ports — Dropped vs. Rejected

`--max-retries` (default 10) controls the retry count when no response comes back.

- **Dropped** (silent firewall rule): only `SENT` lines appear, no `RCVD` at all. Nmap retries roughly 1 second apart before giving up, reporting `filtered` with no explicit reason.
- **Rejected** (active firewall rule): a `SENT` line is followed by a `RCVD` line carrying an ICMP error (`type=3/code=3` = Port unreachable). State is still reported as `filtered`, but `--reason` shows e.g. `port-unreach`.

```bash
# Dropped example
sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn

# Rejected example (ICMP unreachable)
sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn
```

## Discovering Open UDP Ports (-sU)

UDP is stateless — there's no handshake or acknowledgment — so timeouts are much longer and UDP scans are significantly slower than TCP scans.

```bash
sudo nmap 10.129.2.28 -F -sU
```

Open UDP ports often only respond if the application itself is configured to respond to empty datagrams; many UDP ports show as `open|filtered` simply because there's no response at all.

**With `--reason` on UDP:**

- A response received (`udp-response`) → open.
- ICMP error code 3, port unreachable (`port-unreach`) → closed.
- No response at all (`no-response`) → `open|filtered` (ambiguous).

## Version Scan (-sV)

The `-sV` version scan gives the deepest information — service name, exact version, and extra details (e.g. OS hints via a Service Info line). One example showed a full NSOCK probe trace (a NULL probe, then an SMBProgNeg probe) leading to the identification `Samba smbd 3.X - 4.X`.

More info: https://nmap.org/book/man-port-scanning-techniques.html

## Skills Practiced

- Classifying scan results into the six Nmap port states
- Diagnosing filtered ports as dropped vs. rejected using packet traces
- Weighing SYN scan stealth against Connect scan accuracy
- Running and interpreting UDP scans

## Key Takeaways

- "Filtered" is not one outcome — dropped (silence) and rejected (ICMP error) imply different firewall configurations, and only `--packet-trace`/`--reason` reveal which one actually happened.
- SYN scans trade some detectability resistance for speed; Connect scans trade stealth for full accuracy — the right choice depends on how much the engagement cares about being noticed.
- UDP's `open|filtered` ambiguity is a structural limitation of the protocol, not a scanning mistake — many UDP services simply don't answer empty probes.
- `-sV`'s version detection can reveal far more than a port's base identity, including OS hints buried in a Service Info line.
