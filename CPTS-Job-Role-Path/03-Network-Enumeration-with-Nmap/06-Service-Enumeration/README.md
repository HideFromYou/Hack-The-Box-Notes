# Service Enumeration

## Overview

Determining the exact application and version running on a port lets you search for matching known vulnerabilities and exploit code. This lesson covers the recommended scan workflow, tracking scan progress, and verifying Nmap's version detection manually with `tcpdump` and `nc`.

## Learning Objectives

- Follow an efficient scan workflow that balances speed and thoroughness
- Track the progress of a long-running scan
- Understand that Nmap can miss or summarize information a service actually sends
- Manually verify a service banner with `tcpdump` and `nc`

## Recommended Workflow

1. Run a quick port scan first — less traffic, less detectable.
2. Run the full `-p-` scan in the background while working with the initial results.
3. Run `-sV` against the specific ports found, for version detection.

## Checking Scan Progress

- Press **Space Bar** during a running scan to show live status immediately.
- **`--stats-every=5s`** shows status automatically every N seconds/minutes.
- **`-v` / `-vv`** (verbosity) shows each open port as soon as it's discovered, rather than only at the end.

```bash
sudo nmap 10.129.2.28 -p- -sV --stats-every=5s
sudo nmap 10.129.2.28 -p- -sV -v
```

## Banner Grabbing / Full Version Scan

```bash
sudo nmap 10.129.2.28 -p- -sV
```

This shows PORT / STATE / SERVICE / VERSION for every open port, plus a `Service Info:` line with host/OS/CPE hints.

## What Nmap Can Miss

Nmap primarily reads service banners; if that fails, it falls back to slower signature-based matching. **Nmap can still miss information a service actually sends.** For example, `-sV` reported:

```
25/tcp open smtp Postfix smtpd
```

but the raw NSOCK trace revealed the full real banner:

```
220 inlane ESMTP Postfix (Ubuntu)
```

That full banner also disclosed the OS distro (Ubuntu) — information that Nmap's summary line didn't surface.

```bash
sudo nmap 10.129.2.28 -p- -sV -Pn -n --disable-arp-ping --packet-trace
```

Look at the raw `NSOCK READ SUCCESS` line in the trace to see the full banner Nmap actually received.

## Manual Verification with tcpdump + nc

Manual verification confirms what Nmap's automated parsing might smooth over:

```bash
sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28
nc -nv 10.129.2.28 25
```

The `tcpdump` capture shows the full TCP three-way handshake (SYN / SYN-ACK / ACK), then a PSH-ACK packet from the server — PSH means "sending data now," combined with ACK meaning "confirming receipt too" — carrying the actual banner, followed by a final ACK from the client confirming receipt.

## Skills Practiced

- Structuring a scan workflow for speed without sacrificing thoroughness
- Monitoring long-running scans in real time
- Comparing Nmap's summarized service detection against the raw banner
- Manual banner grabbing with `nc`, verified against a `tcpdump` packet capture

## Key Takeaways

- Nmap's `-sV` summary line is a convenience, not the full picture — the raw banner it reads can contain more (like OS distro) than what gets printed.
- Running a quick scan first, then a full `-p-` scan in the background, then a targeted `-sV` pass is far more time-efficient than running `-p- -sV` as the first command.
- A manual `nc` connection paired with a `tcpdump` capture is the ground-truth way to confirm exactly what a service sends, independent of how Nmap parses it.
