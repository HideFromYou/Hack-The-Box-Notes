# SMB

## Overview

This lesson covers SMB protocol fundamentals and version history, dangerous `smb.conf` settings, and a full enumeration toolkit — `smbclient`, `rpcclient`, RID brute-forcing, SMBmap, CrackMapExec, and enum4linux-ng — for footprinting shares, users, and groups via null/anonymous sessions.

## Learning Objectives

- Understand SMB's channel model, ACL-based access control, and version history
- Identify dangerous `smb.conf` settings
- Enumerate shares, users, and groups with `smbclient`, `rpcclient`, and RID brute-forcing
- Use SMBmap, CrackMapExec, and enum4linux-ng for automated, cross-checked enumeration

## Protocol Fundamentals

SMB (Server Message Block) is a Windows-native protocol for file/printer/resource sharing, TCP-based (uses a 3-way handshake). Access is controlled via ACLs (execute/read/full access per user or group), independent of local server-side permissions.

**Samba** is the Linux/Unix implementation of SMB (it implements CIFS, the SMBv1 dialect). CIFS/SMBv1 use NetBIOS ports 137–139; modern SMB (v2/v3) uses port 445 exclusively.

**SMB version history:**

| Version | Introduced | Notable Features |
|---|---|---|
| CIFS | NT 4.0 | NetBIOS |
| SMBv1 | Windows 2000 | Direct TCP |
| SMBv2.0 | Vista / Server 2008 | Performance, signing, caching |
| SMBv2.1 | Windows 7 / Server 2008 R2 | Locking |
| SMBv3.0 | Windows 8 / Server 2012 | Multichannel, encryption, remote storage |
| SMBv3.0.2 | Windows 8.1 / Server 2012 R2 | — |
| SMBv3.1.1 | Windows 10 / Server 2016 | Integrity checking, AES-128 |

Samba v3+ can join an AD domain; v4+ can act as an AD domain controller. Daemons: `smbd` (file/print sharing), `nmbd` (NetBIOS naming). The NetBIOS Name Server (NBNS), enhanced into WINS, handles name registration on a network.

## smb.conf

Config file: `/etc/samba/smb.conf` — a global section (applies to all shares, can be overridden per-share) plus per-share sections.

Key settings: `workgroup`, `server string`, `path`, `browseable`, `guest ok`, `read only`, `create mask`.

**Dangerous settings:**

| Setting | Risk |
|---|---|
| `browseable=yes` | Share is visible when browsing |
| `read only=no` | Write access allowed |
| `writable=yes` | Write access allowed |
| `guest ok=yes` | No authentication required |
| `enable privileges=yes` | Honors privileges assigned to a user/group |
| `create mask=0777` | New files get fully open permissions |
| `directory mask=0777` | New directories get fully open permissions |
| `logon script=<script>` | Script executes on user login |
| `magic script` / `magic output` | Script executes when a file is closed |

Restart the service after a config change:

```bash
sudo systemctl restart smbd
```

## smbclient Enumeration

```bash
smbclient -N -L //<TARGET>          # list shares, null/anonymous session
smbclient //<TARGET>/<share>        # connect to a share
```

Interactive commands: `help`, `ls`, `get <file>`, `put <file>`, `!<local_cmd>` (run a local command without breaking the session).

Admin-side: `smbstatus` shows who's connected, from where, to which share, and the SMB version/signing/encryption in use.

## Nmap

```bash
sudo nmap <TARGET> -sV -sC -p139,445
```

This often gives more limited info compared to the manual tools below.

## rpcclient

MS-RPC interaction:

```bash
rpcclient -U "" <TARGET>
```

Useful queries: `srvinfo`, `enumdomains`, `querydominfo`, `netshareenumall`, `netsharegetinfo <share>`, `enumdomusers`, `queryuser <RID>`, `querygroup <RID>`.

**RID brute-forcing** (fallback when `enumdomusers` is denied but `queryuser` still works):

```bash
for i in $(seq 500 1100); do rpcclient -N -U "" <TARGET> -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo ""; done
```

Alternative via Impacket:

```bash
samrdump.py <TARGET>
```

## SMBmap

Shows permissions per share at a glance:

```bash
smbmap -H <TARGET>
```

## CrackMapExec

Shares plus OS/domain/signing/SMBv1 info together:

```bash
crackmapexec smb <TARGET> --shares -u '' -p ''
```

## enum4linux-ng

Automated all-in-one enumeration — NetBIOS names, SMB dialect support, null/guest session checks, domain info, OS info, users, groups, shares (with per-share mapping/listing tests), password policy, and printers:

```bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt
./enum4linux-ng.py <TARGET> -A
```

## Skills Practiced

- Reasoning about SMB version history and its security-relevant feature additions (signing, encryption)
- Spotting dangerous `smb.conf` settings
- Share/user/group enumeration via `smbclient` and `rpcclient`
- RID brute-forcing as a fallback enumeration technique
- Cross-checking results across SMBmap, CrackMapExec, and enum4linux-ng

## Key Takeaways

- Anonymous access to SMB, even unintentionally, can leak enough (shares, usernames, permissions) to put a whole network at risk.
- Always cross-check SMB enumeration with 2+ tools — different implementations (`smbclient`, `rpcclient`, CrackMapExec, enum4linux-ng) can surface different results from the same target.
- RID brute-forcing is a valuable fallback precisely because `enumdomusers` and `queryuser` are often gated independently — a denial on one doesn't mean the other is also blocked.
- `smb.conf`'s global-vs-per-share override model means a single share can be far more dangerous than the server's general posture suggests.
