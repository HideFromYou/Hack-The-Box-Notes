# Host Discovery

## Overview

For internal pentests, the first step is getting an overview of which systems are online, using Nmap's host discovery options. The most effective default method is ICMP echo requests, but on a local subnet Nmap actually prefers ARP. Scans should always be saved for later comparison, documentation, and reporting.

## Learning Objectives

- Perform host discovery across a network range, an IP list, or specific IPs
- Understand the difference between ARP-based and ICMP-based host discovery on a local subnet
- Use `--packet-trace` and `--reason` to see exactly why Nmap classified a host as up or down

## Scanning a Network Range

```bash
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```

- `-sn` disables port scanning — host discovery only.
- `-oA tnet` saves all output formats under the base filename `tnet`.

This only works if target firewalls allow it; otherwise other evasion techniques are needed (covered later in this module).

## Scanning From an IP List File

```bash
sudo nmap -sn -oA tnet -iL hosts.lst
```

`-iL` reads targets from the given file, e.g. `cat hosts.lst`. In one example only 3 of 7 listed hosts responded — the others may simply ignore the default ICMP echo due to firewall configuration, which doesn't necessarily mean they're dead.

## Scanning Multiple IPs on One Line

```bash
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20
```

## Scanning a Contiguous Range in One Octet

```bash
sudo nmap -sn -oA tnet 10.129.2.18-20
```

## Scanning a Single IP

```bash
sudo nmap 10.129.2.18 -sn -oA host
```

This shows whether the host is up, its latency, and its MAC address.

## ARP vs. ICMP on Local Subnets

When disabling port scanning with `-sn`, Nmap automatically pings via ICMP Echo Requests (`-PE`). However, on the **same local subnet**, Nmap actually sends an **ARP ping first** — it's faster and more reliable on a LAN — even if `-PE` is explicitly added. This can be confirmed with `--packet-trace`:

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace
```

The trace shows `SENT ARP who-has...` / `RCVD ARP reply...` instead of ICMP packets.

**`--reason`** shows why Nmap decided a host is up, e.g. "received arp-response":

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --reason
```

**`--disable-arp-ping`** forces Nmap to actually use ICMP instead of ARP — useful to verify real ICMP connectivity, or for remote/non-LAN targets where ARP doesn't apply anyway:

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```

This shows real `ICMP Echo request` / `Echo reply` packets in the trace.

More host discovery strategies: https://nmap.org/book/host-discovery-strategies.html

## Skills Practiced

- Host discovery across ranges, file-based target lists, and individual IPs
- Differentiating ARP-based vs. ICMP-based discovery behavior on a LAN
- Using `--packet-trace` and `--reason` to verify the actual mechanism behind a scan result

## Key Takeaways

- On a local subnet, Nmap silently prefers ARP over ICMP even when `-PE` is explicitly requested — `--disable-arp-ping` is required to force a true ICMP test.
- A host that doesn't respond to a default host-discovery scan isn't necessarily down — it may simply be configured to ignore ICMP.
- Always save host discovery results (`-oA`) — different tools and different scans against the same range can disagree, and having a saved baseline matters for later comparison and reporting.
