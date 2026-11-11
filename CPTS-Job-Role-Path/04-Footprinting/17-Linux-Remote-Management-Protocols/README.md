# Linux Remote Management Protocols

## Overview

This section covers services used to administer or access Linux/Unix systems remotely. During footprinting, identify the service, determine its version and configuration, and check whether it exposes authentication or trust weaknesses.

## SSH

SSH provides encrypted remote shell access and commonly listens on TCP/22.

### Enumeration

```bash
nmap -sV -p22 <TARGET_IP>
```

Useful checks include:
- Identify the SSH implementation and version.
- Review supported authentication methods and host-key information.
- Test only credentials or keys discovered during the authorized assessment.
- Check whether other exposed services or files reveal SSH usernames or key material.

### Client connection

```bash
ssh <USERNAME>@<TARGET_IP>
ssh -i <PRIVATE_KEY> <USERNAME>@<TARGET_IP>
```

A private key should be protected with restrictive permissions. On Linux, SSH commonly rejects keys that are accessible by other users.

## Rsync

Rsync synchronizes files and directories between systems. It commonly uses TCP/873 when operating as a daemon.

### Enumeration

```bash
nmap -sV -p873 <TARGET_IP>
nc -nv <TARGET_IP> 873
```

An exposed rsync daemon may disclose module names and, depending on configuration, allow listing or retrieving files without authentication.

### Listing modules

```bash
rsync <TARGET_IP>::
```

### Listing and retrieving a module

```bash
rsync --list-only <TARGET_IP>::<MODULE>/
rsync -av <TARGET_IP>::<MODULE>/ ./loot/
```

Treat discovered files as sensitive. Do not modify or upload data unless the assessment scope explicitly permits it.

## R-Services

The traditional Berkeley r-commands provide remote access and file operations:
- rsh — remote shell
- rlogin — remote login
- rexec — remote command execution
- rcp — remote file copy

These services are legacy protocols and may rely on host-based trust (for example, `.rhosts` or `hosts.equiv`) rather than strong cryptographic authentication. Traffic is generally unencrypted.

Common ports:
- TCP/512 — rexec
- TCP/513 — rlogin
- TCP/514 — rsh

### Enumeration

```bash
nmap -sV -p512-514 <TARGET_IP>
```

If an r-service is exposed, assess whether trust relationships or weak access controls permit access without a password. Validate findings carefully and within scope.

## Pentester checklist

- [ ] Identify SSH and record its version and authentication surface.
- [ ] Check for exposed rsync daemon modules and anonymous listing.
- [ ] Identify legacy r-services on TCP/512–514.
- [ ] Correlate usernames, keys, and files discovered through other services.
- [ ] Record evidence without publishing credentials or private keys.

## Key takeaways

- SSH is encrypted, but exposed keys, weak credentials, and poor access controls can still create risk.
- An rsync daemon can unintentionally expose entire directory trees.
- R-services are legacy protocols whose trust-based design and lack of encryption make them risky on modern networks.
- Enumeration findings should be correlated across services rather than treated independently.
