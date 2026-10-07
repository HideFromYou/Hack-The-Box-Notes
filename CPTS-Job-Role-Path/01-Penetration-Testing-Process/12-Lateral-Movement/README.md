# Lateral Movement

## Overview

Covers testing what an attacker could do across the entire network from an initial foothold, not just on the first compromised host — including pivoting/tunneling, and the repeated per-host cycle of evasive testing, information gathering, vulnerability assessment, exploitation, and post-exploitation.

## Learning Objectives

- Understand the goal of lateral movement testing and why it matters (e.g., ransomware spread scenarios)
- Understand pivoting/tunneling and when it's needed
- Understand why internal networks tend to have more exploitable misconfigurations than internet-facing systems
- Understand common internal credential-reuse and hash-based exploitation techniques

## Goal of Lateral Movement

Entered after successful exploitation and post-exploitation. The goal is to test what an attacker could do across the **entire** network, not just the initial foothold — ransomware spreading network-wide is a key real-world motivator for this type of testing.

## Stage Components (Repeated per New Host)

Pivoting, Evasive Testing, Information Gathering, Vulnerability Assessment, (Privilege) Exploitation, Post-Exploitation.

## Pivoting/Tunneling

Using an exploited host as a proxy to route scans/attacks from the attack machine into otherwise-unreachable, non-routable internal network segments — analogous to sending a print job through a home network to reach a printer that isn't directly reachable from the internet.

## Evasive Testing

Same considerations as in Post-Exploitation: understanding network segmentation, threat monitoring, IPS/IDS, and EDR defenses to disguise lateral requests from blue-team detection.

## Information Gathering (Repeated, Network-Wide)

Builds on what was learned in the prior post-exploitation stage, now mapping which other systems are reachable from the current foothold.

## Vulnerability Assessment (Repeated, Internal-Network-Focused)

Internal networks tend to have far more misconfigurations than internet-facing systems. Group memberships and shared resources matter heavily here — for example, compromising a developer-group account can open access to most development resources.

## (Privilege) Exploitation

Commonly involves cracking passwords/hashes, reusing existing credentials across systems, or using captured hashes directly — for example, intercepting NTLMv2 hashes with Responder, then using pass-the-hash to authenticate as that user/admin on multiple hosts without ever cracking the hash.

## Post-Exploitation (Repeated per Newly Reached Host)

The same information-gathering and evidence-collection loop as before, always mindful of the sensitive-data handling rules defined in the contract.

## Skills Practiced

- Using an exploited host as a pivot point to reach non-routable internal segments
- Weighing group membership and shared resources as lateral movement targets, not just individual hosts
- Recognizing pass-the-hash as a valid exploitation path that doesn't require cracking a hash first

## Key Takeaways

- Lateral movement exists as a testing stage because real attackers (and ransomware) don't stop at the first compromised host — demonstrating network-wide reach is often the actual point a client cares about.
- Internal vulnerability assessment should weight group memberships and shared resources heavily — compromising one group-level account can be a bigger lateral movement win than compromising several individual hosts.
- Credential reuse and hash-based authentication (pass-the-hash) are often more productive lateral movement paths than finding a fresh exploit on each new host.
