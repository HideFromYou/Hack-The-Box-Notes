# Nmap Scripting Engine (NSE)

## Overview

The Nmap Scripting Engine lets you write and run Lua scripts to interact with specific services, extending Nmap far beyond basic port/version detection. This lesson covers the 14 NSE categories, how to invoke scripts, and worked examples against SMTP and a WordPress target.

## Learning Objectives

- Know all 14 NSE script categories and what each is for
- Invoke default scripts, whole categories, and individually named scripts
- Use the aggressive scan (`-A`) shortcut and understand what it combines
- Use the `vuln` category to cross-reference a target against known CVEs

## NSE Categories

| Category | Description |
|---|---|
| auth | Determine authentication credentials |
| broadcast | Host discovery via broadcasting; discovered hosts can auto-join remaining scans |
| brute | Brute-force login attempts |
| default | What `-sC` runs |
| discovery | Evaluate accessible services |
| dos | Check for DoS vulnerabilities (rarely used — harms services) |
| exploit | Try to exploit known vulnerabilities on the scanned port |
| external | Uses external services for further processing |
| fuzzer | Send varied/unexpected fields to find vulnerabilities/unexpected handling (slow) |
| intrusive | Could negatively affect the target |
| malware | Check if target is infected with malware |
| safe | Defensive, non-intrusive/non-destructive |
| version | Extension for service detection |
| vuln | Identify specific vulnerabilities |

## Ways to Specify Scripts

```bash
sudo nmap <target> -sC                                # default scripts
sudo nmap <target> --script <category>                # whole category
sudo nmap <target> --script <script1>,<script2>,...    # specific named scripts
```

## Specific Scripts Example (SMTP)

```bash
sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands
```

The `banner` script revealed the Ubuntu distro, and `smtp-commands` listed the accepted SMTP commands (VRFY, ETRN, etc.) — useful for user enumeration.

## Aggressive Scan (-A)

`-A` is a shortcut that combines `-sV` (version detection), `-O` (OS detection), `--traceroute`, and `-sC` (default scripts):

```bash
sudo nmap 10.129.2.28 -p 80 -A
```

In one example this revealed the web server version, the installed web application and its version (WordPress 5.3.4) via `http-generator`, the page title, OS guesses with confidence percentages, and a full traceroute.

## Vuln Category Example (HTTP/WordPress)

```bash
sudo nmap 10.129.2.28 -p 80 -sV --script vuln
```

- `http-enum` found WordPress admin paths and version fingerprints via static asset files.
- `http-wordpress-users` found a valid username (`admin`).
- `vulners` cross-referenced the Apache version against a live CVE database and printed matching CVEs with CVSS scores and links.

More: https://nmap.org/nsedoc/index.html

## Skills Practiced

- Selecting NSE scripts by category vs. by specific script name
- Reading `-A`'s combined output (version, OS, traceroute, default scripts) as a single aggressive pass
- Using the `vuln` category to automatically cross-reference a service version against known CVEs

## Key Takeaways

- The 14 NSE categories map directly to engagement phases — `safe`/`discovery` for early recon, `vuln`/`exploit` for later, more intrusive stages.
- `-A` is convenient but loud — it runs OS detection, traceroute, and default scripts all at once, which is not a stealthy choice.
- NSE's `vuln` scripts (like `vulners`) can do CVE cross-referencing automatically, but the underlying version info still needs to be confirmed accurate first — garbage version data in means garbage CVE matches out.
- Script categories like `brute`, `dos`, `exploit`, and `intrusive` can actively harm or destabilize a target — category choice should match what the engagement's rules of engagement actually allow.
