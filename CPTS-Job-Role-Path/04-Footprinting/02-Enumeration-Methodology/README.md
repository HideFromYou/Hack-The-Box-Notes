# Enumeration Methodology

## Overview

This lesson introduces a static, 6-layer enumeration methodology spanning Infrastructure-based, Host-based, and OS-based enumeration, and scopes this Footprinting module to the Accessible Services layer. It also frames why no time-boxed pentest can claim complete coverage.

## Learning Objectives

- Understand the 3 levels and 6 layers of the enumeration methodology
- Recognize that a methodology is a systematic approach, not a rigid step-by-step guide
- Understand why a time-boxed pentest can never prove the absence of further vulnerabilities

## The Methodology

A static, 6-layer enumeration methodology applies to external and internal pentests (internal/AD-specific layers are covered in other modules). It is divided into 3 levels: Infrastructure-based, Host-based, and OS-based enumeration.

**Analogy:** walls or obstacles to find gaps through, or a labyrinth with multiple possible entry points — not all gaps actually lead inside. A time-boxed pentest can never claim with 100% certainty that no more vulnerabilities exist. The SolarWinds supply-chain attack is a real-world example: attackers with months of persistent access found issues that a pentest spanning only a few weeks likely would not have surfaced.

## The 6 Layers

| Layer | Description | Info Categories |
|---|---|---|
| 1. Internet Presence | Identify internet-facing infrastructure | Domains, Subdomains, vHosts, ASN, Netblocks, IPs, Cloud Instances, Security Measures |
| 2. Gateway | Identify security measures protecting infra | Firewalls, DMZ, IPS/IDS, EDR, Proxies, NAC, Network Segmentation, VPN, Cloudflare |
| 3. Accessible Services | Identify hosted interfaces/services | Service Type, Functionality, Config, Port, Version, Interface |
| 4. Processes | Identify internal processes/data flow | PID, Processed Data, Tasks, Source, Destination |
| 5. Privileges | Identify permissions on accessible services | Groups, Users, Permissions, Restrictions, Environment |
| 6. OS Setup | Identify internal OS components/setup | OS Type, Patch Level, Network config, Config files, sensitive files |

This Footprinting module focuses mainly on **Layer 3 (Accessible Services)**.

## Methodology vs. Tooling

A methodology is a systematic approach, not a rigid step-by-step guide. The specific tools and commands used to execute it are a dynamic "cheat sheet" reference — they change over time and by target — but they are not part of the methodology itself.

## Skills Practiced

- Mapping discovered information to the correct layer of the 6-layer methodology
- Scoping an engagement's current focus (Layer 3) within the larger methodology
- Separating a durable methodology from the tools used to execute it at any given time

## Key Takeaways

- The 6 layers give enumeration findings a place to live — a domain name is Layer 1, a share permission is Layer 5, a config file is Layer 6 — which keeps sprawling findings organized.
- A methodology outlives any specific tool; tools get replaced, the systematic approach behind them does not.
- No pentest, however thorough, can prove a negative — the SolarWinds example is a reminder that time-boxed engagements are inherently a sample, not a guarantee.
