# MySQL

## Overview

This lesson covers MySQL/MariaDB fundamentals, the LAMP/LEMP stack context, dangerous server settings, and footprinting via Nmap's `mysql*` NSE scripts plus direct `mysql` client interaction — including why Nmap's automated findings must always be manually verified.

## Learning Objectives

- Understand MySQL's client-server model and common deployment context (LAMP/LEMP)
- Identify security-relevant MySQL configuration settings
- Footprint a MySQL service with Nmap NSE scripts and the `mysql` CLI client
- Recognize Nmap NSE false positives and verify findings manually

## Protocol Fundamentals

MySQL is an open-source SQL relational database management system (Oracle-developed). Client-server model: the **MySQL server** handles storage/distribution (data in tables — columns, rows, types); **MySQL clients** send structured queries (insert/delete/modify/retrieve) via SQL.

**Common deployment:** LAMP (Linux, Apache, MySQL, PHP) or LEMP (Nginx variant) — MySQL as the central data store behind PHP-driven dynamic websites (e.g. WordPress stores posts/usernames/passwords in its own DB, normally localhost-only).

**What typically lives in a web app's MySQL DB:** headers, texts, meta tags, forms, customer/user info, administrators/moderators, emails, permissions, passwords (ideally pre-hashed by the application before storage, not plaintext), links, and other sensitive content.

**MariaDB** — a fork of MySQL, created after MySQL's original lead developer left following Oracle's acquisition of MySQL AB. Commonly used interchangeably/compatibly with MySQL tooling (the `mysql` CLI client connects to both).

SQL injection errors surfaced by a misbehaving web app are a common indirect footprinting source — verbose error messages often confirm the app interacts with the DB differently than developers intended.

## Dangerous Settings

| Setting | Risk |
|---|---|
| `user` / `password` | Plaintext credentials in config — if file permissions are wrong or another read/shell primitive exists, full DB compromise follows directly |
| `admin_address` | IP allowed on the administrative interface — misconfiguration widens exposure |
| `debug` | Verbose debugging output, can leak sensitive internals |
| `sql_warnings` | Verbose warning strings on single-row `INSERT`s — same leak risk as `debug` |
| `secure_file_priv` | Limits file import/export — misconfigured, widens file read/write primitives (relevant to SQLi → RCE chains, covered later in SQL Injection Fundamentals/SQLMap) |

## Footprinting with Nmap

```bash
sudo nmap <TARGET> -sV -sC -p3306 --script mysql*
```

Runs the full `mysql*` NSE script family: `mysql-brute`, `mysql-databases`, `mysql-dump-hashes`, `mysql-empty-password`, `mysql-enum`, `mysql-info` (protocol/version/capabilities/auth plugin), `mysql-users`, `mysql-variables`, `mysql-vuln-cve2012-2122`.

**Critical caveat — always verify manually:** `mysql-brute`/`mysql-empty-password` can report false positives (e.g. "root:\<empty\> - Valid credentials" when the real root account actually has a fixed password). Confirm directly:

```bash
mysql -u root -h <TARGET>
```
A real `ERROR 1045 (28000): Access denied ... (using password: NO)` disproves the NSE script's empty-password claim.

## Manual Interaction

```bash
mysql -u <user> -p<password> -h <TARGET>
```
(no space after `-p`)

| Command | Description |
|---|---|
| `show databases;` | List all databases |
| `use <database>;` | Select a database |
| `show tables;` | List tables in the selected database |
| `show columns from <table>;` | List a table's columns |
| `select * from <table>;` | Dump a table |
| `select * from <table> where <column> = "<string>";` | Search for a value |

**System databases always present:** `information_schema` (ANSI/ISO-standard metadata, mostly pulled from `sys`) and `sys` (MySQL's own management metadata/tables — e.g. `host_summary` shows connecting hosts and unique user counts). Neither holds application data — the actual target data lives in whatever custom database(s) the app created.

## Skills Practiced

- Nmap NSE-based MySQL footprinting (`mysql-info`, `mysql-enum`, `mysql-brute`, etc.)
- Manually verifying/disproving NSE script findings before trusting them
- Direct `mysql` CLI interaction — listing databases/tables, dumping/searching table data

## Key Takeaways

- Nmap NSE output is a lead, not a verified fact — `mysql-brute`/`mysql-empty-password` false-positiving an empty root password (when a real fixed password exists) is a concrete, recurring failure mode; always confirm with a direct `mysql -u <user> -h <target>` attempt.
- `admin_address`/`user`/`password` living in plaintext config files makes any independent file-read or shell primitive on the host a direct path to full MySQL credential disclosure.
- `information_schema`/`sys` are always present and never hold the actual application data — don't mistake exploring them for finding the target's real content; the custom app-created database(s) are the actual goal.
- MySQL exposed externally is almost always a forgotten temporary setup or workaround, not intentional design — itself a useful signal when found during a pentest.
