# Proof-of-Concept

## Overview

Covers what a Proof-of-Concept (PoC) is meant to demonstrate in a pentest, the forms it can take, a key caveat about scripted PoCs, and why the final report should frame findings as root causes rather than isolated instances.

## Learning Objectives

- Understand what a PoC proves in a pentesting context
- Identify the two forms a PoC can take
- Understand the risk of administrators "fixing the script" instead of the underlying vulnerability
- Distinguish a root cause from a symptom when writing remediation advice

## What a PoC Proves

PoC is a project-management term proving a project or finding is feasible. In pentesting, it's proof that a found vulnerability is real, reproducible, and that its impact can be assessed — so developers/admins can validate the issue and test their remediation. A classic simple example is popping `calc.exe` to prove code execution on Windows.

## Forms a PoC Can Take

- Written documentation of the vulnerability chain
- A script or code that automatically exploits the finding, demonstrating flawless exploitation — often more practical

## Caveat with Scripted PoCs

Administrators sometimes focus on breaking the specific script rather than fixing the underlying root cause. It's important to clarify that the script is just one way to exploit the issue — the underlying vulnerability remains even if that specific script stops working. This should be explicitly discussed in the report and in the report review meeting.

## Report Framing

The report should show the big picture, not just isolated exploit mechanics. An attack-chain walkthrough (for full domain compromise, etc.) shows how multiple flaws combine — and that fixing just one flaw in the chain only breaks that link, since the other flaws still need remediation, or another path to the same compromise may exist.

## Example: Root Cause vs. Symptom

If a Domain Admin account is found using a weak, guessable password, the real vulnerability is the weak **password policy**, not that one specific password. Fixing just that one account's password leaves the systemic weak-password problem — and likely many other weak accounts — unaddressed. Remediation advice should target root causes and standards, not just the specific instance found.

## Skills Practiced

- Writing PoC documentation and/or exploit scripts that demonstrate a finding's real-world impact
- Framing a report around attack chains rather than isolated vulnerabilities
- Distinguishing a root-cause recommendation from a symptom-level fix

## Key Takeaways

- A scripted PoC is a demonstration vehicle, not the vulnerability itself — explicitly saying so in the report prevents administrators from treating "the script no longer works" as "the vulnerability is fixed."
- Reporting an attack chain (not just a single flaw) matters because remediating one link only breaks that specific path — the other flaws in the chain remain exploitable on their own or via another route.
- Remediation advice is most valuable when it targets the systemic root cause (e.g., password policy) rather than the specific finding (e.g., one weak account), since the specific instance is rarely the only one.
