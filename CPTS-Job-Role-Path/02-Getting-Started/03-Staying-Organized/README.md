# Staying Organized

## Overview

Whether performing client assessments, playing CTFs, taking a course, or working through HTB boxes/labs, organization is crucial. Clear and accurate documentation from the very beginning is a skill that pays off in any infosec career path.

## Learning Objectives

- Build a consistent folder structure for engagements
- Choose a note-taking tool and workflow
- Understand the role of a personal knowledge base and findings database

## Folder Structure

Example structure for a client engagement with two assessment types (External and Internal Penetration Test):

```
Projects/
└── Acme Company
    ├── EPT (External Penetration Test)
    │   ├── evidence
    │   │   ├── credentials
    │   │   ├── data
    │   │   └── screenshots
    │   ├── logs
    │   ├── scans
    │   ├── scope
    │   └── tools
    └── IPT (Internal Penetration Test)
        └── (same layout)
```

- `scans` — scan data; `tools` — relevant tools; `logs` — logging output
- `scope` — scoping info (e.g., IP/network lists fed to scanners)
- `evidence` — credentials, retrieved data, screenshots

This is a personal preference — some testers create a folder per target host, others organize by host/network and save screenshots directly into the note tool. Experiment to find what works.

## Note-Taking Tools

Options: Cherrytree, Visual Studio Code, Evernote, Notion, GitBook, Sublime Text, Notepad++. Notion and GitBook offer richer wiki-style features. **Client data must be stored locally only, never synced to the cloud, on real assessments.**

Tip: learning Markdown is easy and very useful for organized, visually appealing notes.

## Knowledge Base and Findings Database

- Maintain a knowledge base with quick-reference guides for routine setup tasks and cheat sheets per assessment phase.
- Aggregate every useful payload, command, and tip from boxes, labs, assessments, and courses — each HTB Academy module provides a downloadable cheat sheet.
- Maintain checklists, report templates per assessment type, and a findings/vulnerability database (spreadsheet or more complex) with title, description, impact, remediation advice, and references. Pre-written findings save significant time during reporting.

## Skills Practiced

- Engagement folder structuring
- Note-taking and documentation habits
- Building a reusable knowledge base

## Key Takeaways

- Start building documentation habits early — they compound over time.
- Never sync client data to the cloud.
- A pre-built findings database saves major time during report writing.
- Markdown is a fast, portable format for technical notes.
