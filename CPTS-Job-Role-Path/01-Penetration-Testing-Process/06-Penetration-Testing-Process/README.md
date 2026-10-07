# Penetration Testing Process

## Overview

Introduces the 8-stage penetration testing process model used throughout the rest of the CPTS path, and walks through a worked example showing how each stage's output feeds the next.

## Learning Objectives

- Name and describe all 8 stages of the penetration testing process
- Understand that the process is a flexible framework, not a rigid step-by-step recipe
- Trace how a single target's assessment flows through all 8 stages in practice

## The 8-Stage Process Model

The pentest process is a deterministic — not purely stochastic — sequence of stages, flexible rather than a rigid recipe, since every client environment is different:

**Pre-Engagement → Information Gathering → Vulnerability Assessment → Exploitation → Post-Exploitation → Lateral Movement → Proof-of-Concept → Post-Engagement**

| Stage | Description |
|---|---|
| Pre-Engagement | Educating the client, NDA, scope, time estimation, rules of engagement |
| Information Gathering | Obtaining information about the target company, software, and hardware to find potential security gaps for a foothold |
| Vulnerability Assessment | Analyzing information-gathering results for known vulnerabilities (manual + automated) to find attack vectors |
| Exploitation | Testing/executing attacks against potential vectors to gain initial access |
| Post-Exploitation | Maintaining access, escalating privileges, hunting sensitive data (pillaging) — either to demonstrate impact or feed into lateral movement |
| Lateral Movement | Moving within the internal network to additional hosts at the same or higher privilege — iterative, combined with post-exploitation |
| Proof-of-Concept | Documenting step-by-step how compromise was achieved, chaining vulnerabilities together, optionally with automation scripts |
| Post-Engagement | Documentation for admins/management, cleanup of all traces, deliverables, report walkthrough, executive presentation, data archival per contract/policy, eventual retest |

## Worked Example (Website Target)

A walkthrough of a single target moving through all 8 stages, showing how each stage's output becomes the next stage's input:

1. Pre-engagement documents are signed, defining scope and rules.
2. Information gathering identifies the target's technology stack.
3. Vulnerability assessment finds known vulnerabilities or unusual/weird features in that stack.
4. Exploitation prepares and tests exploit code against the identified weakness.
5. Post-exploitation performs internal information gathering and privilege escalation after initial access is gained.
6. Lateral movement uses the information gathered to reach other in-scope hosts.
7. Proof-of-concept builds a reproducible write-up or automation of the full attack chain.
8. Post-engagement produces the final report and a walkthrough meeting with the client.

## Skills Practiced

- Mapping a real engagement's activities onto the 8-stage process model
- Recognizing stage outputs as the next stage's required inputs

## Key Takeaways

- The 8 stages aren't a strict linear checklist — stages like Information Gathering and Vulnerability Assessment routinely loop back on each other, and Post-Exploitation/Lateral Movement repeat per newly reached host.
- Every stage's value depends on the quality of the stage before it — weak information gathering produces a weak vulnerability assessment, which produces weak exploitation targeting.
- This model is the organizing skeleton for the rest of the CPTS path: later modules map directly onto one or more of these 8 stages.
