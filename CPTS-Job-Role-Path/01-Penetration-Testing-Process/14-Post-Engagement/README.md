# Post-Engagement

## Overview

Covers everything that happens after active testing concludes: cleanup, documentation and reporting, the report review meeting, deliverable acceptance, post-remediation retesting, the pentester's limited role in remediation, data retention, and close-out activities.

## Learning Objectives

- Understand cleanup obligations and what to do when an artifact can't be removed
- Know what a report deliverable should contain
- Understand the report review meeting's purpose and format
- Understand the DRAFT → FINAL deliverable acceptance process
- Understand why the pentester stays an impartial third party during remediation
- Understand data retention expectations and close-out activities

## Cleanup

Delete any uploaded tools/scripts, and revert any (minor) configuration changes made during testing. If a change or artifact can't be removed (e.g., lost access to a system), it must be disclosed to the client and documented in report appendices — so the client can confirm any alerts they see were caused by sanctioned testing activity, not a real attacker.

## Documentation and Reporting

Before formally ending testing/disconnecting, ensure all evidence (command output, screenshots, affected-host lists, scan/log data) needed for the report is collected. No PII, incriminating information, or other sensitive data should be retained beyond what's needed.

The report deliverable should include:

- An attack chain walkthrough (for full/partial compromise)
- A non-technical-friendly executive summary
- Detailed per-finding risk rating, impact, remediation, and references
- Reproduction steps for each finding
- Near/medium/long-term recommendations
- Appendices (scope, OSINT data, password-cracking analysis, discovered ports/services, compromised hosts/accounts, transferred files, account/system modifications, AD security analysis, supplementary scan data, etc.)

The draft report is the first deliverable sent.

## Report Review Meeting

A walkthrough of the draft report with the client (same attendees as the kick-off meeting, plus relevant SMEs as needed) — typically a brief explanation per finding rather than reading the whole report aloud, with room for client questions, clarifications, and corrections.

## Deliverable Acceptance

The report is first issued marked **DRAFT**, then **FINAL** after client feedback is incorporated. Some audit frameworks require a FINAL designation specifically.

## Post-Remediation Testing

Usually included in the project cost — re-testing each finding after the client claims remediation, producing a before/after status report.

Example status table:

| Finding | Severity | Remediated |
|---|---|---|
| Weak domain password policy | High | Not Remediated |
| Outdated plugin RCE | Critical | Remediated |

## Role of the Pentester in Remediation

Must stay an impartial third party — doesn't apply fixes, patches, or config changes directly, and gives only general remediation guidance (e.g., "sanitize user input" for SQLi, not rewritten code) to preserve independence and avoid a conflict of interest.

## Data Retention

Varies by country/firm and should be defined in the SoW/RoE. PCI DSS guidance: no hard requirement, but best practice is to retain evidence for some period (useful for post-remediation retesting or later questions), store it securely encrypted at rest, owned/controlled by the firm. All data is wiped from tester machines at engagement end; a fresh client-specific VM is used for any later post-remediation work.

## Close Out

- Wipe/destroy any systems used to connect to or process client data
- Securely archive remaining artifacts per policy/contract
- Invoice and collect payment
- Send a post-assessment client satisfaction survey

Client relationships and future work often hinge more on communication quality and professionalism than technical "wow factor."

## Skills Practiced

- Structuring a complete report deliverable (executive summary, per-finding detail, appendices)
- Running a report review meeting focused on findings rather than a page-by-page read-through
- Applying a DRAFT → FINAL acceptance workflow
- Writing remediation guidance that stays general rather than prescribing a specific fix

## Key Takeaways

- An artifact that can't be fully cleaned up isn't something to hide — disclosing it in the report appendix is what lets the client distinguish sanctioned test activity from a real intrusion later.
- Staying an impartial third party during remediation (general guidance only, never applying the fix) is what preserves the value of a later post-remediation retest — the tester can't grade their own work.
- Long-term client relationships are won more by communication quality and professionalism through close-out than by the technical impressiveness of the compromise itself.
