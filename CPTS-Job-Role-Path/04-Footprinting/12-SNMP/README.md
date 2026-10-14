# SNMP

## Overview

This lesson covers SNMP protocol fundamentals (versions, traps, MIB/OID structure), community strings as SNMPv1/v2c's only "security", dangerous daemon settings, and a full footprinting toolkit — `snmpwalk` for querying, `onesixtyone` for community string brute-forcing, and `braa` for bulk OID enumeration.

## Learning Objectives

- Understand SNMP's monitoring/management role and its two UDP ports (queries vs. traps)
- Know the MIB/OID addressing model and why it matters for cross-vendor compatibility
- Explain why SNMPv1/v2c's community strings are not real authentication
- Footprint SNMP with `snmpwalk`, `onesixtyone`, and `braa`

## Protocol Fundamentals

SNMP (Simple Network Management Protocol) monitors and manages network devices (routers, switches, servers, IoT) and can also perform remote configuration changes.

**Two UDP ports, two directions:**

| Port | Direction | Purpose |
|---|---|---|
| 161/UDP | Client → Server | Queries/control commands |
| 162/UDP | Server → Client | **Traps** — unsolicited notifications sent when a configured event occurs, without being requested |

**Versions:**

| Version | Security |
|---|---|
| SNMPv1 | No authentication at all — anyone on the network can read/modify data; no encryption, fully plaintext |
| SNMPv2c | Same security level as v1 (community-based); community string is the only gate, and it's sent in **plaintext** |
| SNMPv3 | Real improvement — username/password authentication + encryption via pre-shared key, at the cost of much higher configuration complexity |

Because migrating to v3 is complex, **v2c remains widely deployed** — a recurring real-world finding.

## MIB and OID

**MIB (Management Information Base)** — a vendor-independent text format (ASN.1-based ASCII) listing every queryable SNMP object on a device, in a standardized tree hierarchy. MIBs don't contain data themselves — they describe *where* to find information and *what type* it is.

**OID (Object Identifier)** — a dot-notation sequence of integers identifying one node's exact position in that tree (e.g. `.1.3.6.1.2.1.1.1.0`). Longer chain = more specific object. Many nodes exist purely as pointers to deeper child nodes.

## Community Strings

Community strings function as the "password" gating SNMPv1/v2c access — but they're just a shared string, transmitted in plaintext, interceptable on the wire, and can be named arbitrarily by the administrator (meaning they sometimes need to be brute-forced rather than guessed outright).

## Default Configuration (`/etc/snmp/snmpd.conf`)

Key default fields: `sysLocation`, `sysContact` (often leaks an admin email directly), `rocommunity`/`rocommunity6` (read-only community string + scope), `rouser` (SNMPv3 read-only user + security level).

**Dangerous settings:**

| Setting | Risk |
|---|---|
| `rwuser noauth` | Full read-**write** access to the entire OID tree, with **no authentication required** |
| `rwcommunity <string> <IPv4>` | Read-write access to the entire OID tree regardless of request source |
| `rwcommunity6 <string> <IPv6>` | Same as above, IPv6 variant |

## Footprinting Tools

**`snmpwalk`** — query OIDs and their values once the community string is known:
```bash
snmpwalk -v2c -c public <TARGET>
```
A misconfigured/default `public` community string on v1/v2c hands over detailed system info directly — OS/kernel version, `sysContact` (admin email), installed packages, uptime, and more, all unauthenticated.

**`onesixtyone`** — brute-force community string names when unknown, using a wordlist:
```bash
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt <TARGET>
```
Community strings bound to specific IPs are often named after the hostname (sometimes obfuscated) — in large environments with a consistent naming pattern, a custom wordlist (e.g. via `crunch`) can target that pattern directly.

**`braa`** — bulk-enumerate OIDs once the community string is known, faster/more thorough than manual `snmpwalk` for scripting across OID ranges:
```bash
braa <community>@<TARGET>:.1.3.6.*
```

## Skills Practiced

- Reading SNMP daemon configuration for dangerous read-write/no-auth settings
- Querying SNMP with `snmpwalk` to pull system info, admin contact, and installed software from an unauthenticated v1/v2c service
- Brute-forcing unknown community strings with `onesixtyone` and SecLists wordlists
- Bulk OID enumeration with `braa`

## Key Takeaways

- SNMPv1/v2c's community string is not authentication in any meaningful sense — it's a shared, plaintext, often-default (`public`) string standing in for a password, and a `public`-readable v2c service routinely leaks admin contact info and full package/OS fingerprinting without any real barrier.
- The gap between SNMPv3's real security and SNMPv1/v2c's near-none explains why so many networks still run the old versions — migration complexity directly trades off against security, and administrators often choose to keep a known-insecure-but-working setup rather than risk breaking monitoring during a v3 migration.
- `sysContact`/`sysLocation` fields are an easy, direct source of OSINT (admin email, site info) once any readable community string is found — no further digging required.
- Community string brute-forcing (`onesixtyone`) matters because strings are admin-arbitrary, not protocol-fixed — `public`/`private` are common defaults, but a competent admin may have renamed them, requiring a wordlist approach.
