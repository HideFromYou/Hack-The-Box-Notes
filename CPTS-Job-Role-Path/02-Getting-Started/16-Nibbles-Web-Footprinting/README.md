# Nibbles: Web Footprinting

## Overview

Continuing the Nibbles walkthrough: Nmap's script scans came up mostly empty, so the next step is manually footprinting the web application on port 80 — fingerprinting the technology stack, discovering hidden directories, and pulling identifying details out of exposed files.

## Learning Objectives

- Fingerprint web technologies with whatweb
- Discover hidden directories and files with gobuster
- Pin down an exact CMS version from exposed source/README files
- Recognize login brute-force protections and know when to stop
- Extract useful intel from exposed XML configuration files
- Build targeted password guesses from information gathered on the site itself

## Technology Fingerprinting

```bash
whatweb <ip>
```

Initially just showed Apache/Ubuntu — nothing CMS-specific yet. Browsing the site manually showed a plain "Hello world!" page, but the page source contained an HTML comment referencing a `/nibbleblog/` directory (also visible via a plain `curl` request).

Pointing whatweb directly at that path revealed more:

```bash
whatweb http://<ip>/nibbleblog
```

This confirmed the site runs **Nibbleblog**, a free PHP blogging engine, with jQuery/HTML5/PHP in use.

Googling "nibbleblog exploit" turned up a known file upload vulnerability — an authenticated attacker can upload and execute arbitrary PHP — with an existing Metasploit module targeting version 4.0.3.

## Directory Discovery

```bash
gobuster dir -u http://<ip>/nibbleblog/ --wordlist /usr/share/seclists/Discovery/Web-Content/common.txt
```

Found: `/admin`, `/admin.php`, `/content`, `/languages`, `/plugins`, `/README`, `/themes` (plus forbidden `.hta`/`.htaccess`/`.htpasswd`, as expected).

A second gobuster run against the site root confirmed no other directories or ports exist beyond what was already found.

## Pinpointing the CMS Version

```bash
curl http://<ip>/nibbleblog/README
```

Confirmed the exact version: v4.0.3 "Coffee" — matching the vulnerable version targeted by the Metasploit module found earlier.

Directory listing was also enabled on `/nibbleblog/themes/`, revealing the installed themes (echo, medium, note-2, simpler, techie) — not directly useful here, but worth noting as exposed information.

## Login Attempts and Blacklist Protection

A login page exists at `admin.php`. Trying common credentials (`admin:admin`, `admin:password`) failed, and the application triggered an IP blacklist after too many attempts:

```
Nibbleblog security error - Blacklist protection
```

This rules out brute-forcing tools like Hydra against the login form — repeated failed attempts just lock the attacking IP out.

## Extracting Info from XML Files

The exposed `content/private/` directory (found via gobuster) contained readable XML files:

```bash
curl -s http://<ip>/nibbleblog/content/private/users.xml | xmllint --format -
```

`xmllint --format -` pretty-prints XML piped in from curl. This confirmed the `admin` username exists, plus the current list of blacklisted IPs.

```bash
curl -s http://<ip>/nibbleblog/content/private/config.xml | xmllint --format -
```

No password was exposed here, but the site title, slogan, and notification email (`admin@nibbles.com`) all referenced "Nibbles" — the same as the box name.

## Guessing Credentials from Site Content

This is the key insight of the lesson: rather than relying only on generic wordlists, use information gathered from the site itself to build a targeted guess — the same thinking behind tools like CeWL, which crawl a site to build a custom wordlist from its own content.

Here, the repeated "Nibbles" branding in the config suggested the admin password might simply be the box's own name. Combined with the confirmed `admin` username from `users.xml`, this was enough to successfully authenticate to `admin.php` — the login itself happens at the very end of this stage, continuing into the foothold lesson.

## Skills Practiced

- Web technology fingerprinting with whatweb
- Directory and file discovery with gobuster
- Extracting intel from exposed XML configuration files
- Deriving a targeted credential guess from site-specific content rather than generic wordlists

## Key Takeaways

- An HTML comment or a stray path reference in page source can reveal an entire application that automated scanning missed.
- A exposed `README` or version file is often the fastest way to pin an exact CMS version — faster than fingerprinting it indirectly.
- Repeated failed logins can trigger blacklist protections; recognize this quickly and pivot away from brute-forcing rather than wasting time or getting locked out.
- The most effective password guesses often come from information the application exposes about itself — box names, branding, and contact emails are all fair game.
