# Web Enumeration

## Overview

Web servers on ports 80/443 often host applications that provide a considerable attack surface and are high-value targets. Proper web enumeration is critical, especially when an organization exposes few other services, or those services are well patched.

## Learning Objectives

- Perform directory/file brute-forcing with Gobuster
- Perform DNS subdomain enumeration
- Use header grabbing, fingerprinting, certificates, robots.txt, and page source for recon

## Directory/File Enumeration

```bash
gobuster dir -u http://10.10.10.121/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

Status codes: `200` success, `403` forbidden, `301` redirect (not a failure — worth following up). A discovered `/wordpress` path, for example, could reveal a CMS installation still in setup mode, potentially leading to RCE.

## DNS Subdomain Enumeration

Important resources (admin panels, extra functionality) can live on subdomains.

```bash
git clone https://github.com/danielmiessler/SecLists
# or: sudo apt install seclists -y

gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

If DNS resolution fails, add a public resolver (e.g., `1.1.1.1`) to `/etc/resolv.conf`.

**Gobuster `dns` mode (active brute force)** vs. **Subfinder (passive OSINT)**: Subfinder queries public sources (certificate transparency logs, VirusTotal, Shodan, etc.) and never touches the target directly — a good, quiet first pass. Gobuster actively brute-forces names against the target's DNS and needs authorization/scope, since it sends many live requests. In practice: passive first (Subfinder), then active brute force (Gobuster) with authorization, to catch what passive sources missed.

## Web Enumeration Tips

### Banner Grabbing / Headers

```bash
curl -IL https://www.inlanefreight.com
```

Reveals the server software, framework hints, and misconfigurations. **EyeWitness** can screenshot and fingerprint multiple targets, also spotting possible default credentials.

### Whatweb

```bash
whatweb 10.10.10.121
whatweb --no-errors 10.10.10.0/24
```

Extracts server version, frameworks, and applications in use — a quick way to identify technologies worth researching further.

### Certificates

HTTPS certificate details (subject/issuer) can reveal company names and email addresses, potentially useful for a phishing engagement if in scope.

### robots.txt

Tells crawlers what not to index, but often reveals interesting paths (e.g., `/private`, `/uploaded_files`) that are worth visiting directly even though they're "disallowed."

### Source Code

`CTRL+U` to view page source — developer comments sometimes leak test credentials or other useful details.

## Skills Practiced

- Directory/file brute-forcing with Gobuster
- DNS subdomain enumeration (active and passive)
- Web fingerprinting (headers, Whatweb, certificates, robots.txt, source code)

## Key Takeaways

- Web enumeration isn't just directory brute force — headers, certs, robots.txt, and page source are all free sources of information.
- Passive subdomain enumeration (Subfinder) should come before active brute force (Gobuster dns), partly for authorization reasons.
- A `403` isn't a dead end — it confirms the resource exists, even if we can't access it yet.
