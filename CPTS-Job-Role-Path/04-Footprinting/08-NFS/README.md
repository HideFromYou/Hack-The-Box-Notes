# NFS

## Overview

This lesson covers NFS protocol fundamentals and version differences, its critical UID/GID-based trust weakness, dangerous `/etc/exports` settings, and hands-on footprinting/mounting of NFS exports.

## Learning Objectives

- Understand NFS's RPC foundation and version differences (NFSv2/v3/v4/v4.1)
- Explain NFS's core authentication weakness and why it should stay off untrusted networks
- Identify dangerous `/etc/exports` options, especially `no_root_squash`
- Footprint, mount, and browse NFS exports

## Protocol Fundamentals

NFS (Network File System) is the Linux/Unix equivalent of SMB, but it is a completely different protocol — NFS and SMB clients can't talk to each other directly. It's built on ONC-RPC/SUN-RPC (port 111) and uses XDR for system-independent data representation.

**Version differences:**

| Version | Notes |
|---|---|
| NFSv2 | Older, originally UDP-only |
| NFSv3 | More features, variable file size, better errors; not fully NFSv2-compatible |
| NFSv4 | Adds Kerberos, works through firewalls/internet, no portmapper needed, ACL support, stateful, single port 2049 only — first version with real **user** authentication rather than just client-machine authentication |
| NFSv4.1 | Adds pNFS (parallel/clustered access) and session trunking (multipathing) |

## Critical Weakness (NFSv2/v3)

NFSv2/v3 have no real authentication mechanism — authorization is derived purely from UNIX UID/GID matching, which the **server** must trust blindly from the **client's** claim. If the client and server don't have matching UID/GID-to-user mappings, no further server-side check catches this. For this reason, NFS should only be used on trusted networks.

## /etc/exports

Maps exported folders to allowed hosts/subnets plus options:

```
rw / ro              → read-write / read-only
sync / async         → synchronous (safer, slower) / asynchronous (faster)
secure / insecure    → restrict to ports <1024 / allow any port
no_subtree_check     → disables subdirectory tree checking
root_squash          → maps root (UID/GID 0) to "anonymous" on the client side — prevents root from having full access via the mount
```

Example admin-side export and reload:

```bash
echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
systemctl restart nfs-kernel-server
exportfs
```

**Dangerous settings:**

| Setting | Risk |
|---|---|
| `rw` | Read-write access |
| `insecure` | Allows any port, not just <1024 |
| `nohide` | Exposes a filesystem mounted below an already-exported directory as its own export |
| `no_root_squash` | Root stays root — files created as root via the mount keep UID/GID 0, meaning real root-level write access through NFS. A classic privesc vector when combined with SSH access: upload a SUID binary as root via the mount, then trigger it from the SSH session |

## Footprinting

```bash
sudo nmap <TARGET> -p111,2049 -sV -sC
```

The `rpcinfo` NSE script lists all active RPC services (rpcbind, nfs, mountd, nlockmgr, nfs_acl) with their ports/protocols.

All NFS-specific NSE scripts together:

```bash
sudo nmap --script nfs* <TARGET> -sV -p111,2049
```

- `nfs-ls` — file listing with permissions/UID/GID/size
- `nfs-showmount` — exports
- `nfs-statfs` — filesystem stats

## Standalone Tools

```bash
showmount -e <TARGET>
```

## Mounting

```bash
mkdir target-NFS
sudo mount -t nfs <TARGET>:/ ./target-NFS/ -o nolock
```

`-o nolock` avoids NFS lock-daemon issues; `:/` mounts the root export, or a specific export path can be specified after the colon.

```bash
ls -l mnt/nfs/      # owner/group shown by NAME (if UID/GID match local users on your own machine)
ls -n mnt/nfs/      # raw UID/GID numbers
```

Unmount when done:

```bash
cd ..
sudo umount ./target-NFS
```

## Skills Practiced

- Reasoning about NFS's UID/GID trust model and why it's weaker than SMB's ACL model
- Spotting dangerous `/etc/exports` options
- Footprinting NFS via Nmap's `rpcinfo` and `nfs*` NSE scripts
- Mounting and browsing NFS exports, reading ownership both by name and raw UID/GID

## Key Takeaways

- NFSv2/v3's authentication model is fundamentally about trusting the client's claimed UID/GID — there is no independent server-side identity check, which is why NFS belongs only on trusted networks.
- `no_root_squash` is the single most dangerous NFS export option — it's a direct path to root-level file writes, especially potent when chained with separate SSH access (plant a SUID binary via the mount, trigger it over SSH).
- `ls -l` vs `ls -n` matters: ownership showing by name only works if the attacker's own machine happens to share UID/GID mappings with the target — `-n` always shows the ground truth.
- NFSv4 is a meaningful security improvement over v2/v3 specifically because it introduces real user-level (not just machine-level) authentication.
