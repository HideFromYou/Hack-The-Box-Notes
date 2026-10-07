# Pre-Engagement

## Overview

Covers everything that must happen before active testing begins: confirming who at the client organization can legally authorize the test, and producing the full set of documents — NDA, scoping questionnaire, scoping document, contract, Rules of Engagement, and (for physical assessments) a Contractors Agreement.

## Learning Objectives

- Identify the three core components of pre-engagement and the meetings that tie them together
- Understand the different NDA types and when each applies
- Know the required documents, their order, and what each one covers
- Understand what a Rules of Engagement document must contain
- Understand the purpose of a Contractors Agreement for physical assessments
- Know what happens in a kick-off meeting

## Core Components

Three core components make up pre-engagement: the **Scoping Questionnaire**, the **Pre-Engagement Meeting**, and the **Kick-off Meeting** — all preceded by a signed **NDA**.

NDA types:

| Type | Description |
|---|---|
| Unilateral | One party is bound |
| Bilateral | Both parties are bound — most common for pentest work |
| Multilateral | Three or more parties — e.g., cooperative networks |

## Authorization Chain

It's critical to confirm who at the client organization actually has the authority to hire/authorize the pentest — this can be a CEO, CTO, CISO, CSO, CRO, CIO, VP/Director of IT or InfoSec, Audit Manager, etc., depending on company size. Hiring without proper authorization (e.g., a rogue employee trying to "test" their own employer without real authority) creates serious legal exposure for the tester.

## Required Documents (in creation order)

1. **NDA** — after initial contact
2. **Scoping Questionnaire** — before the pre-engagement meeting
3. **Scoping Document** — during the pre-engagement meeting
4. **Penetration Testing Proposal** (Contract/SoW) — during the pre-engagement meeting
5. **Rules of Engagement (RoE)** — before the kick-off meeting
6. **Contractors Agreement** (physical assessments only) — before the kick-off meeting
7. **Reports** — during/after testing

All documents should be lawyer-reviewed.

## Scoping Questionnaire

Asks which assessment type(s) are needed (Internal/External VA or Pentest, Wireless, Application, Physical, Social Engineering, Red Team, Web App), plus scale details and the required evasion/information-disclosure level:

- Scale details: live hosts, IP/CIDR ranges, domains/subdomains, wireless SSIDs, web/mobile apps + auth roles, phishing targets, physical locations, whether an AD assessment is needed, anonymous vs. domain-user network perspective, NAC bypass needed
- Evasion/disclosure level: black/grey/white box; non-evasive, hybrid-evasive, or fully evasive

## Pre-Engagement Meeting Contract Checklist

- NDA
- Goals
- Scope
- Penetration Testing Type
- Methodologies (OSSTMM, OWASP, etc.)
- Testing Locations
- Time Estimation (including per-phase time windows, business-hours vs. after-hours)
- Third Parties (written consent required from any third-party-hosted infrastructure)
- Evasive Testing level
- Risks
- Scope Limitations & Restrictions (critical systems to avoid)
- Information Handling (HIPAA/PCI/etc.)
- Contact Information + escalation order
- Lines of Communication
- Reporting requirements
- Payment Terms

## Rules of Engagement (RoE) Checklist

- Introduction
- Contractor
- Penetration Testers
- Contact Info
- Purpose
- Goals
- Scope
- Lines of Communication
- Time Estimation
- Time of Day to Test
- Pentest Type
- Pentest Locations
- Methodologies
- Objectives/Flags
- Evidence Handling
- System Backups
- Information Handling
- Incident Handling and Reporting
- Status Meetings
- Reporting
- Retesting
- Disclaimers and Limitation of Liability
- Permission to Test

## Kick-off Meeting

Happens after all contracts are signed — in-person or a scheduled call, including client POCs, technical staff, and the pentest team (practice lead, testers, PM). Covers:

- No DoS testing by default
- The critical-finding pause procedure (pause testing, generate a vulnerability notification report, alert emergency contacts — typically only for External tests with critical flaws like unauthenticated RCE, SQLi, or sensitive data exposure)
- Risk warnings to the client (log/alarm noise, possible account lockouts from brute forcing)
- Explaining the process clearly for both technical and non-technical attendees — target the explanation at the least technical person in the room

## Contractors Agreement (Physical Assessments Only)

A "get out of jail free card" in case testers are caught during physical/social engineering attempts and employees call the police — kept separate from the main contract since physical intrusion involves different laws.

Checklist: Introduction, Contractor, Purpose, Goal, Penetration Testers, Contact Info, Physical Addresses, Building Name, Floors, Physical Room IDs, Physical Components, Timeline, Notarization, Permission to Test.

## Setting Up

The final step before testing: preparing VMs/VPS/tools for all scenarios ahead of actual testing (covered in depth in a separate "Setting Up" module).

## Skills Practiced

- Sequencing pre-engagement documents correctly
- Verifying an authorization chain before accepting an engagement
- Building a complete RoE from a checklist rather than from memory

## Key Takeaways

- Confirming who actually has authority to hire a pentest isn't a formality — testing at the request of someone without real authority creates real legal exposure for the tester, not just the client.
- The Contractors Agreement exists specifically because physical/social-engineering testing intersects with different laws than remote/network testing — it's not redundant with the main contract.
- The RoE checklist is long because it has to answer, in writing, every question that could otherwise cause a dispute mid-engagement (what's in scope, what happens if something critical is found, who gets called, what "done" looks like).
