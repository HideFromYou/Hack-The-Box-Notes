# Penetration Testing Overview

## Overview

Defines what a penetration test is, how it fits into a company's broader risk management program, how it differs from a pure vulnerability assessment and from a Red Team assessment, and surveys the testing methods, information levels, and environment types a pentest can target.

## Learning Objectives

- Define a penetration test and contrast it with a Red Team assessment and a Vulnerability Assessment
- Understand how a pentest fits into a company's risk management process
- Understand employee notification and personal-data handling expectations
- Distinguish External vs. Internal testing, and Blackbox/Greybox/Whitebox/Red-Team/Purple-Team testing types
- Identify the range of environments a pentest can target

## Definition

A Penetration Test is an organized, targeted, authorized attack attempt using real-attacker methods and techniques to test IT infrastructure and defenders, aiming to uncover **all** vulnerabilities and improve security. This is in contrast to Red Team assessments, which are scenario/goal-based and only cover the vulnerabilities relevant to reaching a specific objective.

## Risk Management Context

Pentesting feeds into a company's broader IT security risk management process: identify, evaluate, and mitigate risk to confidentiality, integrity, and availability. Inherent risk always remains even with proper controls in place. Companies can respond to risk in one of four ways — accept it, transfer it (insurance/contracts), avoid it, or mitigate it.

A pentest produces a point-in-time snapshot, not continuous monitoring. The client/system operator is responsible for actually fixing the issues found — the tester's role is trusted advisor providing findings and remediation recommendations, not the party applying the fixes.

## Vulnerability Assessments vs. Penetration Tests

| | Vulnerability Assessment | Penetration Test |
|---|---|---|
| Method | Purely automated tool-based scanning (Nessus, Qualys, OpenVAS) against known-issue databases | Mix of automated + manual testing, individually tailored |
| Adaptability | Can't adapt to specific target configurations | Preceded by extensive manual information gathering; much more complex planning, execution, and tool selection |
| Authorization | Requires mutual written agreement | Requires mutual written agreement |

Testing without explicit authorization can be a criminal offense in either case. Third-party-hosted infrastructure (e.g., some cloud providers) may need separate written authorization from that third party — though some providers, like AWS, have blanket policies covering certain types of testing.

## Employee Notification and Data Handling

Employees are generally not informed in advance, at management's discretion — but they have a right to know eventually, since they have no expectation of privacy at work. Testers must handle any personal or sensitive data found (names, salaries, credit card numbers, etc.) per Data Protection Act principles: keep it private, and recommend remediation (e.g., password changes, encryption) rather than exposing it further.

## Testing Methods: External vs. Internal

- **External:** performed as an anonymous internet user, often via VPN or VPS. The client may want full stealth, a "hybrid" gradually-noisier approach, or no stealth at all. The goal is to access external-facing hosts, sensitive data, or a path into the internal network.
- **Internal:** performed from within the corporate network, either after a successful external breach or from an assumed-breach starting point. May require physical on-site presence for fully isolated/air-gapped systems.

## Types of Penetration Testing (by information provided)

| Type | Information Provided |
|---|---|
| Blackbox | Minimal — just IPs/domains |
| Greybox | Extended — URLs, hostnames, subnets, etc. |
| Whitebox | Maximum — configs, admin credentials, source code |
| Red-Teaming | May include physical/social engineering; combinable with any of the above |
| Purple-Teaming | Combinable with any of the above; focused on working closely with defenders |

Less information provided generally means a longer, more complex approach, since more reconnaissance is needed upfront.

## Types of Testing Environments

What's being tested is often mixed across several categories: Network, Web App, Mobile, API, Thick Clients, IoT, Cloud, Source Code, Physical Security, Employees, Hosts, Server, Security Policies, Firewalls, IDS/IPS.

## Skills Practiced

- Distinguishing pentest scope/depth models (Blackbox/Greybox/Whitebox) and selecting the right one for a stated goal
- Recognizing when a task is a Vulnerability Assessment versus a full Penetration Test

## Key Takeaways

- A pentest is a point-in-time snapshot, not continuous monitoring — the tester's deliverable is informed recommendations, not a guarantee of ongoing security.
- The core difference between a VA and a pentest isn't tooling, it's depth: VA checks for known issues, a pentest adapts to the specific target through manual analysis.
- A Red Team assessment and a full pentest are not the same goal — Red Team is scenario-driven and narrower by design, while a pentest aims for comprehensive coverage.
- Less information given up front (blackbox) doesn't mean less thorough testing — it means more time has to go into reconnaissance before exploitation can even start.
