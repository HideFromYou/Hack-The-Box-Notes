# Saving the Results

## Overview

Nmap can save scan results in three formats, which matters for comparison, documentation, and reporting value later in an engagement. This lesson covers Normal, Grepable, and XML output, plus converting XML into an HTML report.

## Learning Objectives

- Save Nmap scan output in Normal, Grepable, and XML formats
- Use `-oA` to save all three formats at once
- Convert an XML scan result into a readable HTML report with `xsltproc`

## The Three Output Formats

- **Normal** (`-oN`, `.nmap`) — a human-readable summary, the same as what prints to the terminal.
- **Grepable** (`-oG`, `.gnmap`) — single-line-per-host format, easy to parse with `grep`/`awk`.
- **XML** (`-oX`, `.xml`) — structured, machine-parseable output (hostnames, ports, service/version attributes, timing stats).

**`-oA`** saves all three formats at once under a shared base filename:

```bash
sudo nmap 10.129.2.28 -p- -oA target
```

This creates `target.gnmap`, `target.xml`, and `target.nmap` in the current directory (or the given path).

## Converting XML to an HTML Report

The XML output can be transformed into a clean, readable HTML report using `xsltproc` — useful for documentation or for sharing results with non-technical stakeholders:

```bash
xsltproc target.xml -o target.html
```

Open `target.html` in a browser for a structured visual presentation of the scan.

More info: https://nmap.org/book/output.html

## Skills Practiced

- Saving scan output in Normal, Grepable, and XML formats
- Using `-oA` to generate all three formats in a single run
- Converting XML scan results into an HTML report with `xsltproc`

## Key Takeaways

- Always save scans, even quick ones — a saved Grepable or XML file is instantly searchable later, while terminal scrollback is not.
- XML output isn't just for machine parsing — it's also the input format for generating a polished HTML report for reporting/documentation purposes.
- Defaulting to `-oA` means never having to decide in advance which single format will be needed later.
