# Firewall Evasion - Medium Lab

## Overview

The second of three hands-on labs applying the evasion techniques from the Firewall and IDS/IPS Evasion lesson. After the Easy lab, the client's admin hardens the firewall further. This lab's goal is to discover the target's DNS server version.

## Learning Objectives

- Recognize when a service is UDP-based rather than TCP-based, and adjust scan technique accordingly
- Use `-sU` to reach and fingerprint a UDP service
- Understand why a plain TCP scan can completely miss a UDP-only service

## Scenario

After the admin hardens the firewall further, the exercise requires discovering the target's DNS server version.

## Practical Notes (Exercise)

Target: `<TARGET_IP>`.

**Methodology:**

1. Initial scanning with a standard TCP approach (`-sS`, `-sA`, and similar TCP scan types without `-sU`) showed the DNS-related port as filtered or simply irrelevant — because DNS version detection is a **UDP**-based service (port 53/UDP), not TCP.
2. Recognizing this required stepping back from TCP-only assumptions: the relevant service wasn't going to show up correctly under any TCP scan type, no matter how the TCP flags or evasion options were tuned.
3. Switching to a UDP scan combined with version detection against the specific port resolved this:

```bash
sudo nmap -sU -sV -p53 <TARGET_IP>
```

4. This correctly reached the UDP-based DNS service and returned its version information, which a TCP-only scan could never have surfaced regardless of evasion technique.

**Key technique:** `-sU -sV -p53` — UDP scan with version detection against port 53, required because the target service only responds over UDP; no amount of TCP scan tuning (SYN, ACK, source-port tricks, etc.) can substitute for scanning the correct transport protocol.

## Skills Practiced

- Diagnosing a falsely "filtered"/missing result as a transport-protocol mismatch rather than a firewall block
- Running targeted UDP version detection against a specific port
- Avoiding wasted effort tuning TCP scan options against a UDP-only service

## Key Takeaways

- Before escalating to more exotic evasion techniques, confirm the transport protocol is even correct — a UDP-only service will never respond to a TCP scan, however it's evaded or tuned.
- `-sU -sV` on a specific, known-relevant port is far more efficient than a full UDP sweep, since UDP scanning is inherently slow.
- "Filtered" or "no response" on a TCP scan isn't always a firewall doing the filtering — sometimes it just means the service was never listening on TCP in the first place.
