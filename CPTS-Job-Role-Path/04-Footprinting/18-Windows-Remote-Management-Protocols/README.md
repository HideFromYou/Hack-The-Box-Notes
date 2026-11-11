# Windows Remote Management Protocols

## Overview

This section covers common protocols and technologies used to administer Windows hosts remotely. During footprinting, identify exposed management services and determine what authentication and access controls are in place.

## RDP — Remote Desktop Protocol

RDP provides remote graphical access to Windows systems. It commonly listens on TCP/3389.

### Enumeration

```bash
nmap -sV -p3389 <TARGET_IP>
nmap -p3389 --script rdp-enum-encryption <TARGET_IP>
```

The results can help identify the service and assess supported security settings. RDP exposure alone does not prove that a host is vulnerable or that credentials are valid.

### Connection

```bash
xfreerdp /v:<TARGET_IP> /u:<USERNAME>
```

Use credentials only when they have been obtained within the authorized scope.

## WinRM — Windows Remote Management

WinRM is Microsoft's remote management service, based on WS-Management. Common ports:
- TCP/5985 — HTTP
- TCP/5986 — HTTPS

### Enumeration

```bash
nmap -sV -p5985,5986 <TARGET_IP>
```

### Interaction

With the appropriate authorized credentials, tools such as Evil-WinRM can provide a PowerShell session:

```bash
evil-winrm -i <TARGET_IP> -u <USERNAME> -p '<PASSWORD>'
```

WinRM availability does not mean every account can log in remotely. Authorization depends on account rights and host configuration.

## WMI — Windows Management Instrumentation

WMI is a Windows management infrastructure used to query system information and administer Windows hosts. Remote WMI commonly relies on DCOM/RPC and Windows authentication. It is not represented by one simple dedicated TCP port; RPC endpoint mapping and dynamic RPC ports are relevant.

### Enumeration considerations

- Identify Windows hosts and exposed RPC-related services.
- Check whether the assessment account is authorized for remote WMI queries.
- Use WMI to gather host, operating-system, process, or service information when permitted.
- Distinguish network reachability from successful authenticated WMI access.

## Common footprinting workflow

1. Scan likely management ports, including TCP/3389, TCP/5985, and TCP/5986.
2. Identify service versions and available protocol information.
3. Note whether management traffic is encrypted (for example, RDP security settings or WinRM over HTTPS).
4. Test discovered credentials only within scope.
5. Correlate management access with other enumeration results.

## Pentester checklist

- [ ] Identify RDP and inspect its security configuration.
- [ ] Check WinRM HTTP/HTTPS availability.
- [ ] Understand WMI's reliance on RPC/DCOM and Windows permissions.
- [ ] Separate exposed services from confirmed authenticated access.
- [ ] Document evidence and avoid storing reusable credentials in public notes.

## Key takeaways

- RDP provides graphical remote access; WinRM provides remote management through WS-Management.
- WMI is a management framework, not simply another single-port service.
- Successful remote management depends on both network exposure and account authorization.
- Windows management services can provide valuable enumeration opportunities, but access must be verified rather than assumed.
