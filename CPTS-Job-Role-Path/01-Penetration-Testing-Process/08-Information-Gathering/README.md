# Information Gathering

## Overview

Covers the four categories of information gathering performed on every engagement — OSINT, Infrastructure Enumeration, Service Enumeration, and Host Enumeration — and introduces Pillaging as a concept that threads through this stage and Post-Exploitation.

## Learning Objectives

- Understand why information gathering is the cornerstone phase that all later exploitation depends on
- Identify the four categories of information gathering and what each one covers
- Understand how OSINT can surface critical sensitive data, and what to do when it does
- Understand why internal/non-internet-facing services and hosts are often more vulnerable, not less
- Understand what Pillaging is and where it fits in the overall process

## The Cornerstone Phase

Information gathering begins after pre-engagement contracts are signed. It's the cornerstone phase — all later exploitation steps depend on the information gathered here. Four categories are performed on every engagement.

## 1. Open-Source Intelligence (OSINT)

Finding publicly available information (events, external/internal dependencies and connections) using open sources only. OSINT can surface highly sensitive leaked data within minutes — passwords, hashes, keys, tokens — often via misconfigured public repositories (GitHub, etc.) or code pasted on sites like StackOverflow.

If critical sensitive information is found this way (e.g., a live SSH key), the RoE's Incident Handling section should define how to report it immediately to the client before proceeding. Covered in depth in the OSINT: Corporate Recon module.

## 2. Infrastructure Enumeration

Mapping the company's internet/intranet footprint via OSINT plus initial active scans (DNS mapping of name/mail/web servers, cloud instances, etc.), cross-referenced against the agreed scope. Also used to identify security measures in place (firewalls, WAFs) to inform evasive testing decisions.

Done from an external or internal perspective. Internal enumeration is especially useful for planning Password Spraying targets (one password tried against many usernames, hoping for one hit).

## 3. Service Enumeration

Identifying what services are reachable, their versions, and what they're used for. Version history often reveals outdated or vulnerable software — administrators often accept known risk rather than risk breaking functionality by upgrading.

## 4. Host Enumeration

Detailed per-host examination (OS, services, versions) combining active scanning with OSINT. Internal-only services are often misconfigured or neglected because administrators assume "not internet-facing = safe." Determining a host's role and what it communicates with is key here.

Internal host enumeration, performed post-exploitation, extends to local sensitive files, services, scripts, and apps — feeding directly into Pillaging and Post-Exploitation.

## Pillaging (Introduced Here)

Collecting sensitive information locally on an already-exploited host (employee names, customer data, etc.) — only possible after a foothold has been gained. HTB Academy deliberately has no standalone "Pillaging" module: it's treated as integral to Information Gathering and Privilege Escalation, woven throughout many modules (Network Enumeration with Nmap, Getting Started, Password Attacks, AD Enumeration & Attacks, Linux/Windows PrivEsc, Attacking Common Services/Applications, Attacking Enterprise Networks) rather than isolated into one place — practiced across 150+ targets and 9 simulated mini-pentests throughout the path. Covered in more depth in Post-Exploitation.

## Skills Practiced

- Structuring reconnaissance into the four information-gathering categories rather than ad-hoc scanning
- Recognizing when a sensitive-data find (e.g., a leaked credential) requires immediate client notification per the RoE
- Identifying internal/non-internet-facing assets as high-value targets rather than deprioritizing them

## Key Takeaways

- "Not internet-facing" is a common but false assumption of safety — internal hosts and services are frequently more vulnerable because administrators neglect them on that exact assumption.
- OSINT alone can end an engagement's information-gathering phase early if it surfaces a critical live credential — knowing the RoE's incident-handling process in advance avoids hesitation in that moment.
- Pillaging has no dedicated module by design: it's a mindset applied continuously during information gathering and post-exploitation, not a one-time checklist step.
