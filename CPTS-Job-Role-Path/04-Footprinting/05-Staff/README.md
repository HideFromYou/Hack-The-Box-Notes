# Staff

## Overview

This lesson covers OSINT on employees — job postings, LinkedIn/Xing profiles, and personal GitHub repos — as a way to infer a company's tech stack and security posture without touching its infrastructure.

## Learning Objectives

- Extract tech-stack information from job postings
- Recognize how employee GitHub repos can leak hardcoded secrets
- Use LinkedIn's advanced search to target technical staff specifically
- Connect a named framework/technology to publicly documented misconfiguration checklists

## Job Postings and Tech Stack

OSINT on employees (LinkedIn, Xing) and job postings reveals the technologies, languages, frameworks, and databases a company uses. One example job posting listed:

- OOP languages: Java, C#, C++
- Scripting: Python, Ruby, PHP, Perl
- Databases: PostgreSQL, MySQL, SQL Server, Oracle
- Web frameworks: Flask, Django, Spring, ASP.NET MVC
- Tooling: Atlassian suite, Git/SVN/Perforce

## Employee GitHub and Personal Projects

Employee "About" sections or personal project pages can reveal personal GitHub repos. These may contain hardcoded secrets — for example, a personal email address, or a hardcoded JWT token found in example code.

Researching a named framework (e.g. "Django security misconfigurations") can lead to public OWASP-style checklists that describe exactly what a pentester should look for, and what filenames/conventions are commonly trusted blindly by that framework's developers.

## LinkedIn Advanced Search

LinkedIn's advanced search supports filtering by connections, location, company, school, industry, and title — more specific filters yield fewer but more targeted results. The best approach is to search for technical employees (developers and security staff specifically), since they reveal both the tech stack **and** what security measures/posture the company likely has in place.

## Skills Practiced

- Extracting technology stack details from job postings
- Finding and reviewing employee personal GitHub repos for leaked secrets
- Using LinkedIn's advanced search filters to target technical staff
- Connecting a discovered framework to its publicly known misconfiguration patterns

## Key Takeaways

- Job postings are an underused source of precise tech-stack detail — companies often list exact languages, frameworks, and databases to attract qualified candidates.
- Personal employee repositories can leak secrets the company itself never exposed — the risk surface extends beyond official company infrastructure.
- Targeting technical staff specifically in OSINT (rather than staff broadly) yields insight into both the tech stack and the company's actual security posture.
