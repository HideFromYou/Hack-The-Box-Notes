# MSSQL

## Overview

This lesson covers MSSQL (Microsoft SQL Server) fundamentals, clients, default system databases, dangerous settings, and footprinting via Nmap's `ms-sql-*` NSE scripts, Metasploit's `mssql_ping`, and authenticated interaction with Impacket's `mssqlclient.py`.

## Learning Objectives

- Understand MSSQL's Windows-centric deployment and common clients
- Know the 5 default system databases and their purpose
- Identify dangerous MSSQL configuration patterns
- Footprint MSSQL with Nmap NSE, Metasploit, and connect with `mssqlclient.py`

## Protocol Fundamentals

MSSQL — Microsoft's closed-source SQL relational database management system, built natively for Windows (strong `.NET` integration), though Linux/macOS versions exist. Default port: **TCP 1433**.

**Clients:** `SQL Server Management Studio (SSMS)` (GUI, installable separately from the server — meaning a developer/admin workstation with saved credentials in SSMS can itself be a target), `mssql-cli`, SQL Server PowerShell, HeidiSQL, SQLPro, and **Impacket's `mssqlclient.py`** — the one pentesters reach for most, since Impacket ships by default on most pentesting distros:

```bash
locate mssqlclient
```

## Default System Databases

| Database | Description |
|---|---|
| `master` | Tracks all system information for the SQL Server instance |
| `model` | Template for every new database — changes here propagate to all future new DBs |
| `msdb` | Used by SQL Server Agent for scheduling jobs/alerts |
| `tempdb` | Temporary objects |
| `resource` | Read-only, system objects shipped with SQL Server |

## Default Configuration

Freshly installed, network-accessible MSSQL typically runs as `NT SERVICE\MSSQLSERVER`. Client connections commonly use **Windows Authentication** by default — the underlying Windows OS (local SAM or AD domain controller) handles the login, not MSSQL itself. Useful for centralized auditing/access control in AD environments, but a compromised AD account can directly translate into MSSQL access, and vice versa — privilege escalation/lateral movement risk flows both ways. **Encryption is not enforced by default** on client connections.

## Dangerous Settings

- MSSQL clients connecting **without encryption**
- **Self-signed certificates** when encryption is used — these can be spoofed
- **Named pipes** enabled
- Weak/default **`sa`** credentials — admins often forget to disable this built-in account

## Footprinting with Nmap

```bash
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes \
  --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER \
  -sV -p 1433 <TARGET>
```

Reveals hostname, instance name, exact MSSQL version/service pack, named pipe path, and whether the `sa` account has an empty password — all valuable recon to record before attempting any connection.

## Footprinting with Metasploit

```
use auxiliary/scanner/mssql/mssql_ping
set rhosts <TARGET>
run
```
Confirms server name, instance name, version, TCP port, and named pipe path — a faster, more targeted alternative to the full NSE script bundle above.

## Authenticated Interaction — `mssqlclient.py`

```bash
python3 mssqlclient.py <user>@<TARGET> -windows-auth
```
`-windows-auth` forces Windows Authentication (matching the server's default). Once connected, interact directly via T-SQL:

```sql
SQL> select name from sys.databases
```
Listing databases first gives a lay of the land — the 5 default system databases will always appear; anything beyond those is the actual target application data.

## Skills Practiced

- Identifying MSSQL's Windows-centric default auth model and encryption posture
- Nmap NSE-based MSSQL footprinting (`ms-sql-info`, `ms-sql-ntlm-info`, `ms-sql-dac`, etc.)
- Metasploit `mssql_ping` auxiliary scanning
- Authenticated MSSQL interaction via Impacket's `mssqlclient.py`, listing databases via T-SQL

## Key Takeaways

- Windows Authentication tying MSSQL access to the Windows SAM/AD identity means a compromised domain account is often also a direct MSSQL access path, and MSSQL compromise can in turn become a lateral-movement/privesc vector back into AD — the two attack surfaces aren't separate in practice.
- SSMS being a client-side-installable GUI means credentials can live on any admin/developer workstation that's ever managed the database, not just the server itself — widening the realistic attack surface well beyond the MSSQL host.
- A single heavy Nmap NSE scan (`ms-sql-info` + friends) front-loads a lot of recon (version, named pipes, NTLM info) before a single authentication attempt is made — worth running before trying any credentials.
- Default `sa` credentials left enabled remains a recurring, high-impact MSSQL misconfiguration — same theme as other services in this module (SNMP community strings, MySQL root, SMTP VRFY): a default/well-known credential left unchanged is consistently the easiest way in.
