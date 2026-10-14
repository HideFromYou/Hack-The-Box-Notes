# Oracle TNS

## Overview

This lesson covers Oracle TNS (Transparent Network Substrate) fundamentals, the `tnsnames.ora`/`listener.ora` config files, default Oracle service credentials, and a full footprinting/exploitation workflow — Nmap for service/SID discovery, ODAT for credential brute-forcing, and `sqlplus` for authenticated database interaction, including privilege escalation to `sysdba` and file upload via `utlfile`.

## Learning Objectives

- Understand TNS's role as the communication layer between Oracle clients and databases
- Read `tnsnames.ora` (client-side) and `listener.ora` (server-side) configs
- Know default Oracle service credentials worth checking
- Footprint Oracle TNS with Nmap, brute-force credentials with ODAT, and interact via `sqlplus`

## Protocol Fundamentals

Oracle TNS (Transparent Network Substrate) is the communication protocol between Oracle databases and client applications, supporting multiple network stacks (TCP/IP, IPX/SPX) plus IPv6/SSL-TLS in newer versions. Common in healthcare/finance/retail for large, complex database deployments. Provides name resolution, connection management, load balancing, and (optional) encryption over the TCP/IP layer.

**Default port:** TCP 1521 (configurable). The listener supports multiple protocols and can bind to specific interfaces or all of them. Remote management was possible in Oracle 8i/9i but removed in 10g/11g.

## Default Configuration — `tnsnames.ora` / `listener.ora`

Located in `$ORACLE_HOME/network/admin`, both plaintext.

- **`tnsnames.ora`** (client-side) — maps service names to network addresses. Each entry: a service name, `ADDRESS` (host/port/protocol), and `CONNECT_DATA` (service name/SID, server type).
- **`listener.ora`** (server-side) — defines what the listener process listens for/forwards (SID list, listener address, Oracle home paths).

| Setting | Description |
|---|---|
| `DESCRIPTION` | Names the database + connection type |
| `ADDRESS` | Network address (host + port) |
| `PROTOCOL` | Network protocol in use |
| `CONNECT_DATA` | Service name/SID, protocol, instance identifier |
| `SERVICE_NAME` / `INSTANCE_NAME` | What the client connects to |
| `SERVER` | Dedicated or shared connection type |
| `USER` / `PASSWORD` | Plaintext auth credentials (if present) |
| `SECURITY` / `VALIDATE_CERT` / `SSL_VERSION` | TLS/SSL connection security settings |
| `TRACE_LEVEL` / `TRACE_DIRECTORY` / `LOG_FILE` | Logging/tracing config |

**PL/SQL Exclusion List** — a user-created blacklist file (`$ORACLE_HOME/sqldeveloper`) of PL/SQL packages/types barred from execution via the Oracle Application Server.

**Default credentials worth remembering:**

| Service/Version | Default Password |
|---|---|
| Oracle 9 | `CHANGE_ON_INSTALL` |
| Oracle 10 | None set by default |
| Oracle DBSNMP service | `dbsnmp` |

Also: organizations sometimes still run the **finger service** alongside Oracle — risky if an attacker knows a home directory, since it can leak further account info.

## SID (System Identifier)

A SID uniquely identifies a database instance (a set of processes + memory structures managing that instance's data). Clients must specify the correct SID to connect — wrong SID = failed connection. If unspecified, the client falls back to `tnsnames.ora`'s default.

## Footprinting with Nmap

```bash
sudo nmap -p1521 -sV <TARGET> --open
```
Confirms the TNS listener and its version.

```bash
sudo nmap -p1521 -sV <TARGET> --open --script oracle-sid-brute
```
Brute-forces/guesses valid SIDs (e.g. returns `XE` for Oracle Express Edition).

## Setting Up ODAT

**ODAT (Oracle Database Attacking Tool)** — open-source Python tool to enumerate and exploit Oracle DB vulnerabilities (credential brute-forcing, SQLi, RCE, privesc). Requires `cx_Oracle` + build dependencies installed first (see module for the full `apt`/`pip`/`git clone` setup sequence).

```bash
./odat.py -h
```

**Full automated sweep** (tries all modules against a target):
```bash
./odat.py all -s <TARGET>
```
Includes credential brute-forcing against known accounts — e.g. recovering `scott:tiger` (a famous long-standing Oracle default demo account).

## `sqlplus` — Authenticated Interaction

Setup requires `oracle-instantclient-sqlplus` (plus a `ld.so.conf.d` fix if `libsqlplus.so` isn't found).

```bash
sqlplus <user>/<password>@<TARGET>/<SID>
```

**Enumeration once connected:**
```sql
select table_name from all_tables;
select * from user_role_privs;
```

**Escalating to `sysdba`** — if the account has the right grants, re-connect with elevated role:
```bash
sqlplus <user>/<password>@<TARGET>/<SID> as sysdba
```
This surfaces the full `SYS` role/privilege set (DBA, DATAPUMP_*, APEX_ADMINISTRATOR_ROLE, etc.) — a direct full-database-control escalation path when the account is authorized for it.

**Extracting password hashes** (for offline cracking):
```sql
select name, password from sys.user$;
```

## File Upload via ODAT (`utlfile`)

Requires knowing the web root path:

| OS | Default web root |
|---|---|
| Linux | `/var/www/html` |
| Windows | `C:\inetpub\wwwroot` |

```bash
echo "Oracle File Upload Test" > testing.txt
./odat.py utlfile -s <TARGET> -d <SID> -U <user> -P <password> --sysdba --putFile <webroot> testing.txt ./testing.txt
```
Verify via direct HTTP request to the uploaded file — a path toward a web shell if the target also runs a web server from that root.

## Skills Practiced

- Reading `tnsnames.ora`/`listener.ora` to understand Oracle TNS connection configuration
- Nmap-based TNS service detection and SID brute-forcing (`oracle-sid-brute`)
- ODAT installation and full-sweep credential brute-forcing
- `sqlplus` authenticated enumeration, `sysdba` privilege escalation, password hash extraction, and file upload via `utlfile`

## Key Takeaways

- Oracle's own long-lived default credentials (`scott:tiger`, `dbsnmp:dbsnmp`, Oracle 9's `CHANGE_ON_INSTALL`) remain a recurring, high-value first guess — the same "default/well-known credential" theme seen across every service in this module (SNMP, MySQL, MSSQL).
- Gaining low-privilege access (e.g. `scott`) doesn't end the engagement — re-authenticating `as sysdba` is a direct, simple-looking privilege escalation step when the account happens to be authorized, turning a limited foothold into full DBA control.
- `sys.user$` password hash extraction (offline-crackable) is a standard next step once any elevated access is achieved, mirroring the same "dump creds, crack offline" pattern used against OS-level credential stores elsewhere in pentesting.
- File upload via `utlfile` only matters if a co-located web server's root directory is known/guessable — a reminder that cross-service knowledge (what else is running on the host) directly enables exploitation of a database-only foothold.
