# DNS

## Overview

This lesson covers DNS server types and record types, BIND9 configuration structure, dangerous server settings, and a full footprinting toolkit — SOA/ANY queries, zone transfers, and subdomain brute-forcing — for mapping a domain's DNS infrastructure.

## Learning Objectives

- Distinguish the different DNS server roles (root, authoritative, non-authoritative, caching, forwarding, resolver)
- Know the common DNS record types and what each one is used for
- Identify dangerous BIND9 server settings, especially around zone transfers
- Perform SOA/ANY queries, attempt zone transfers, and brute-force subdomains

## DNS Server Types

DNS resolves domain names to IPs via a globally distributed hierarchy with no central database.

- **Root servers** — TLD authority, last resort, 13 globally, coordinated by ICANN
- **Authoritative nameservers** — give binding answers for their own zone, fall back to root if they can't answer
- **Non-authoritative nameservers** — gather info via recursive/iterative querying, not authoritative for any zone themselves
- **Caching servers** — cache answers for a TTL set by the authoritative server
- **Forwarding servers** — just relay queries elsewhere
- **Resolvers** — local, non-authoritative, run on the client/router itself

DNS is mainly unencrypted by default (sniffable) — DoT (DNS over TLS), DoH (DNS over HTTPS), and DNSCrypt exist as encryption solutions.

## DNS Record Types

| Record | Purpose |
|---|---|
| A | IPv4 address |
| AAAA | IPv6 address |
| MX | Mail server(s) |
| NS | Domain's nameservers |
| TXT | Free-form — SPF/DMARC/site verification/etc |
| CNAME | Alias pointing to another domain name |
| PTR | Reverse lookup — IP → domain name |
| SOA | Zone info + admin contact (email uses `.` instead of `@`, e.g. `hostmaster.inwx.net.` = `hostmaster@inwx.net`) |

SOA query:

```bash
dig soa <DOMAIN>
```

## BIND9 Configuration Structure

BIND9 (the most common Linux DNS server) splits `named.conf` into global options (apply to all zones) and per-zone options (override global when both exist). Local config files typically: `named.conf.local`, `named.conf.options`, `named.conf.log`.

A **zone file** (BIND format) fully describes one DNS zone — it must have exactly one SOA record and at least one NS record; a syntax error makes the whole zone unusable (SERVFAIL responses). A **reverse zone file** maps the last IP octet to an FQDN via PTR records.

**Master vs. Slave:**

- **Primary/master** — the direct source of truth for a zone (manually edited or dynamically updated)
- **Secondary/slave** — pulls zone data from a master for redundancy/load distribution/primary protection; fetches the SOA at refresh intervals and compares serial numbers to detect staleness

A secondary can itself be a master to further-downstream slaves; a primary is always a master.

## Dangerous DNS Server Settings

| Setting | Risk |
|---|---|
| `allow-query` | Which hosts can query the server at all |
| `allow-recursion` | Which hosts can send recursive queries |
| `allow-transfer` | Which hosts can receive full zone transfers |
| `zone-statistics` | Collects zone stats |

## Footprinting Commands

Query another nameserver directly (`@` targets a specific DNS server instead of your default resolver):

```bash
dig ns <DOMAIN> @<DNS_SERVER_IP>
```

DNS server version via CHAOS-class TXT query (only works if not disabled server-side):

```bash
dig CH TXT version.bind <DNS_SERVER_IP>
```

ANY query (not a guarantee of showing every zone entry, but often reveals a lot, e.g. SPF/verification TXT records):

```bash
dig any <DOMAIN> @<DNS_SERVER_IP>
```

## Zone Transfer (AXFR)

The single most impactful DNS misconfiguration to check for. If `allow-transfer` is too permissive (e.g. set to "any"), anyone can pull the **entire** zone file, including internal hostnames/IPs never meant to be public:

```bash
dig axfr <DOMAIN> @<DNS_SERVER_IP>
```

Also try on suspected internal/sub-zones specifically — they may have a separate, more loosely configured `allow-transfer` rule:

```bash
dig axfr internal.<DOMAIN> @<DNS_SERVER_IP>
```

A successful internal zone transfer can reveal domain controllers, VPN gateways, workstations, etc. with real internal IPs — essentially handing over an internal network map.

## Subdomain Brute-Forcing

When zone transfer isn't possible, brute-force subdomains with a wordlist (e.g. SecLists):

```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do dig $sub.<DOMAIN> @<DNS_SERVER_IP> | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt; done
```

An automated tool that attempts zone transfer **and** subdomain brute-force together:

```bash
dnsenum --dnsserver <DNS_SERVER_IP> --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt <DOMAIN>
```

`-p 0 -s 0` disables Google scraping/WHOIS lookups, focusing purely on DNS enumeration.

> **Note:** this section's hands-on exercise (4 questions covering FQDN enumeration, a zone transfer TXT flag, a DC's IP, and a specific-octet hostname) has not been attempted yet — deferred to a later session.

## Skills Practiced

- Interpreting DNS server roles and record types
- Reading BIND9 zone file structure and master/slave replication
- Querying SOA/NS/ANY records and attempting AXFR zone transfers
- Subdomain brute-forcing manually and via `dnsenum`

## Key Takeaways

- A misconfigured `allow-transfer` is arguably the single highest-impact DNS finding possible — a successful AXFR can hand over an entire internal network map in one query.
- Internal/sub-zones are worth testing separately from the main zone — they sometimes carry their own, more permissive `allow-transfer` rule.
- An `ANY` query isn't guaranteed to show every record, but it's a fast way to surface TXT-based SPF/verification data alongside the basics.
- Subdomain brute-forcing is the fallback when zone transfer fails, and `dnsenum` automates both attempts in a single tool.
