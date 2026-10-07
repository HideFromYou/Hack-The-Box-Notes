# Penetration Testing Process

## Overview

Notes from the Hack The Box Academy **Penetration Testing Process** module — the conceptual foundation for the whole CPTS Job Role Path. Covers the Academy learning philosophy and platform layout, the legal/ethical framework governing pentesting, and the full 8-stage penetration testing process from Pre-Engagement through Post-Engagement, plus how to structure ongoing practice.

## Lessons

| # | Topic |
|---|---|
| 01 | Introduction to the Pentester Path |
| 02 | Academy Modules Layout |
| 03 | Academy Exercises and Questions |
| 04 | Penetration Testing Overview |
| 05 | Laws and Regulations |
| 06 | Penetration Testing Process |
| 07 | Pre-Engagement |
| 08 | Information Gathering |
| 09 | Vulnerability Assessment |
| 10 | Exploitation |
| 11 | Post-Exploitation |
| 12 | Lateral Movement |
| 13 | Proof-of-Concept |
| 14 | Post-Engagement |
| 15 | Practice |

## Skills Practiced

- Mapping the HTB Academy catalog and the CPTS path onto the stages of a real penetration test
- Applying the legal/ethical framework (consent, scope, regional law) that gates every engagement
- Structuring an engagement around the 8-stage process: Pre-Engagement, Information Gathering, Vulnerability Assessment, Exploitation, Post-Exploitation, Lateral Movement, Proof-of-Concept, Post-Engagement
- Building and sequencing pre-engagement documents (NDA, Scoping Questionnaire, RoE, Contractors Agreement)
- Distinguishing root-cause remediation advice from symptom-level fixes
- Structuring a practice regimen around modules, retired/active machines, and Pro Labs

## Key Takeaways

- The entire field rests on one hard gate: written authorization and defined scope. Every later technical skill in this path is only legal to use within that boundary.
- The 8-stage process (Pre-Engagement → Information Gathering → Vulnerability Assessment → Exploitation → Post-Exploitation → Lateral Movement → Proof-of-Concept → Post-Engagement) is a flexible framework, not a rigid script — Information Gathering and Vulnerability Assessment routinely loop back on each other, and Post-Exploitation/Lateral Movement repeat per newly reached host.
- A real pentest is not a CTF: thoroughness and quality of analysis are the actual deliverable, not speed to a single flag — missing a simple vector that leads to a real breach reflects poorly on the whole engagement.
- Lateral Movement and Pillaging deliberately have no standalone modules — they're iterative practices woven through Information Gathering, Post-Exploitation, and many later technical modules rather than one-time checklist steps.
- Good remediation advice targets root causes (e.g., a weak password policy), not just the specific instance found (e.g., one weak account) — and the pentester stays an impartial third party, giving guidance rather than applying fixes directly.
- Communication quality — clear reports, well-run kickoff and review meetings, honest documentation of what couldn't be cleaned up — matters as much to client trust as the technical impressiveness of the compromise itself.
