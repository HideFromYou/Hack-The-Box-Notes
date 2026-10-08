# Domain Information

## Overview

This lesson covers fully passive domain reconnaissance: certificate transparency logs, Shodan, and DNS record enumeration via `dig`, used to map a company's subdomains and infer third-party service providers without ever touching the target directly.

## Learning Objectives

- Use SSL certificates and crt.sh to passively discover subdomains
- Confirm which discovered subdomains are actually company-hosted vs. third-party
- Use Shodan to passively enumerate exposed ports/services on discovered IPs
- Use `dig any` and interpret TXT records to identify third-party providers

## Passive-Only Reconnaissance

This stage is passive-only (no active scans) — navigate the target like a normal "visitor" to stay hidden. Start with the company's main website and its service descriptions, reading it with a developer's eye to infer what tech stack each service would need.

## SSL Certificates and crt.sh

The **SSL certificate** of the main site often lists multiple domains/subdomains on a single certificate.

**crt.sh** (Certificate Transparency logs, RFC 6962) finds subdomains that ever had an SSL certificate issued for them, entirely passively:

```bash
https://crt.sh/?q=<DOMAIN>
curl -s https://crt.sh/\?q\=<DOMAIN>\&output\=json | jq .
```

Extract a unique subdomain list:

```bash
curl -s https://crt.sh/\?q\=<DOMAIN>\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

## Confirming Company-Hosted Subdomains

Not every discovered subdomain is actually hosted by the company — some are third-party, and those can't be tested without separate permission. Confirm which ones resolve to the company's own infrastructure:

```bash
for i in $(cat subdomainlist); do host $i | grep "has address" | grep <DOMAIN> | cut -d" " -f1,4; done
```

## Shodan

Shodan lets you passively view already-indexed exposed ports, services, and banners for discovered IPs, without touching the target yourself:

```bash
for i in $(cat ip-addresses.txt); do shodan host $i; done
```

## DNS Records via dig

```bash
dig any <domain>
```

This shows A/MX/NS/TXT/SOA records together. The key insight is that **TXT records reveal third-party providers** via verification strings and SPF includes:

- Atlassian → dev/collaboration tooling worth investigating
- Google → possible exposed Google Drive
- LogMeIn → centralized remote access, high-value if compromised
- Mailgun → watch for API vulnerabilities like IDOR/SSRF
- Outlook/O365 → possible Azure blob/file storage via SMB
- A hosting-provider TXT verification string often doubles as the account username/ID for that platform

## Skills Practiced

- Passive subdomain discovery via certificate transparency logs
- Filtering discovered subdomains down to company-owned infrastructure
- Passive service/banner reconnaissance via Shodan
- Reading DNS TXT records to infer third-party tooling and its risk implications

## Key Takeaways

- Certificate transparency logs (crt.sh) expose historical subdomains entirely passively — a certificate issued once stays logged even if the subdomain is later retired.
- A discovered subdomain isn't automatically in scope — resolving it against the target's own IP range is what separates company-hosted assets from third-party ones you can't test.
- TXT records are an underrated goldmine: each third-party verification string or SPF include points to a specific external provider with its own distinct risk profile.
