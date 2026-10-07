# Service Scanning

## Overview

Once we can reach a target, the first step is identifying the operating system and available services. A service is an application that performs a useful function for other users/computers; machines hosting them are called servers. We're interested in services that are misconfigured or vulnerable — coercing a service into an unintended action, such as executing a command of our choosing.

## Learning Objectives

- Run basic and advanced Nmap scans
- Use Nmap scripts for targeted enumeration
- Enumerate FTP, SMB, and SNMP manually
- Use banner grabbing to fingerprint services

## Nmap Basics

Ports range 1–65,535; 1–1,023 are well-known/privileged. Port 0 is reserved and not used; binding to it falls through to the next available port above 1,024.

```bash
nmap 10.129.42.253
```

By default, Nmap scans the 1,000 most common **TCP** ports.

- **STATE**: `open`, `closed`, or `filtered` (a firewall may be allowing access only from specific addresses)
- **SERVICE**: the name typically mapped to the port number — not confirmed until Nmap actually interacts with the service

Port hints: `3389` (RDP) suggests Windows; `22` (SSH) suggests Linux/Unix (but can run on Windows too).

### Advanced Scan

```bash
nmap -sV -sC -p- 10.129.42.253
```

- `-sC` — run default Nmap scripts for more detail
- `-sV` — version scan; fingerprints service protocol, application, and version (1,000+ signatures)
- `-p-` — scan all 65,535 TCP ports (much slower)

Example findings: `vsftpd 3.0.3` with anonymous FTP login allowed; `OpenSSH 8.2p1 Ubuntu 4ubuntu0.1`; `Apache httpd 2.4.41 (Ubuntu)` with a phpinfo() page title; `Samba smbd 4.6.2`.

**Inferring OS from package versions:** an OpenSSH package suffix like `Ubuntu 4ubuntu0.1` can be cross-referenced against Ubuntu changelogs to infer the exact OS release (e.g., `20.04 Focal Fossa`). Not fully reliable — newer packages can be backported onto an older OS.

## Nmap Scripts

`-sC` runs many default scripts, but specific ones are sometimes needed (e.g., auditing a known Citrix NetScaler CVE):

```bash
locate scripts/citrix
nmap --script <script name> -p<port> <host>
```

## Banner Grabbing

```bash
nmap -sV --script=banner <target>
nc -nv 10.129.42.253 21
# 220 (vsFTPd 3.0.3)
```

Can also be automated across a subnet: `nmap -sV --script=banner -p21 10.10.10.0/24`.

## FTP

```bash
nmap -sC -sV -p21 10.129.42.253
ftp -p 10.129.42.253
```

Common commands inside an FTP session: `ls`, `cd`, `get <file>`, `exit`. Anonymous login (`anonymous` / any password) is often enabled by misconfiguration and can expose files directly.

## SMB

Prevalent on Windows; shares can hold credentials; some versions are vulnerable to RCE (e.g., EternalBlue). Enumerate carefully.

```bash
nmap --script smb-os-discovery.nse -p445 10.10.10.40
nmap -A -p445 10.129.42.253
```

List and connect to shares with `smbclient`:

```bash
smbclient -N -L \\10.129.42.253           # -L list shares, -N no password prompt
smbclient \\10.129.42.253\users           # guest attempt
smbclient -U bob \\10.129.42.253\users    # authenticated attempt
```

Inside an smbclient session: `ls`, `cd <dir>`, `get <file>`.

## SNMP

Community strings act as a plaintext password in SNMP v1/v2c (default `public`/`private` often unchanged); encryption/authentication only arrived in SNMPv3. Can reveal process parameters (sometimes credentials on command lines), routing info, and installed software versions.

```bash
snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0
onesixtyone -c dict.txt 10.129.42.254    # brute force community strings
```

## Skills Practiced

- Nmap scanning (default, version, script, full-port)
- FTP/SMB/SNMP manual enumeration
- Banner grabbing with Netcat and Nmap scripts

## Key Takeaways

- A default Nmap scan only checks the top 1,000 TCP ports — always consider `-p-` for a full sweep.
- Service version strings can hint at OS version, but backports make this unreliable on its own.
- Anonymous FTP, open SMB shares, and default SNMP community strings are common, easy wins during enumeration.
- Banner grabbing is the fastest way to identify exactly what's running on a port.
