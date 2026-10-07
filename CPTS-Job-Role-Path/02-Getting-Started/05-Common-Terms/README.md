# Common Terms

## Overview

Fundamental terminology and technologies that recur throughout a penetration testing career: shells, ports, web servers, and the OWASP Top 10.

## Learning Objectives

- Understand what a shell is and the three main shell delivery types
- Understand ports, TCP vs. UDP, and well-known port numbers
- Understand the role of a web server and the OWASP Top 10

## What is a Shell?

On Linux, a shell is a program that takes keyboard input and passes commands to the OS. Most Linux systems use **Bash** (an enhanced version of the original Unix `sh`); other shells include Zsh, Tcsh, Ksh, and Fish.

"Getting a shell" on a box means the target has been exploited and we have shell-level (bash/sh) access, obtained via a web app or network/service vulnerability, or via credentials.

| Shell Type | Description |
|---|---|
| Reverse shell | Initiates a connection back to a listener on our attack box |
| Bind shell | Binds to a port on the target and waits for our connection |
| Web shell | Runs OS commands via the browser; typically not fully interactive |

## What is a Port?

Analogy: a port is like a window or door on a house — if left open/unlocked, it can allow unauthorized access. Ports are virtual, software-based endpoints managed by the OS, tied to a specific process/service, letting a host differentiate traffic types over the same connection.

- **TCP**: connection-oriented; a handshake must complete before data is sent; server listens for connections.
- **UDP**: connectionless; no handshake, no delivery guarantee; useful when error-checking isn't needed or is handled by the app; suited to time-sensitive tasks.

There are 65,535 TCP ports and 65,535 UDP ports.

| Port(s) | Protocol |
|---|---|
| 20/21 (TCP) | FTP |
| 22 (TCP) | SSH |
| 23 (TCP) | Telnet |
| 25 (TCP) | SMTP |
| 80 (TCP) | HTTP |
| 161 (TCP/UDP) | SNMP |
| 389 (TCP/UDP) | LDAP |
| 443 (TCP) | SSL/TLS (HTTPS) |
| 445 (TCP) | SMB |
| 3389 (TCP) | RDP |

Recognizing common ports by number alone (21 = FTP, 80 = HTTP, 88 = Kerberos) without looking them up comes with practice, and helps prioritize enumeration and attacks.

## What is a Web Server?

A back-end application that handles HTTP traffic from the client browser, routes requests, and sends back responses — usually on TCP 80/443. Since web apps are public-facing, vulnerabilities can lead to back-end server compromise, making them a high-value target and a large attack surface.

## OWASP Top 10 (2021)

Standardized, non-exhaustive list of the top 10 most dangerous web application vulnerability categories, maintained by OWASP, and a common starting point for web assessment methodology.

| # | Category | Description |
|---|---|---|
| 1 | Broken Access Control | Restrictions not properly implemented, allowing access to other accounts/data/functionality |
| 2 | Cryptographic Failures | Crypto failures leading to sensitive data exposure or compromise |
| 3 | Injection | User-supplied data not validated/sanitized (SQLi, command injection, LDAP injection) |
| 4 | Insecure Design | App not designed with security in mind |
| 5 | Security Misconfiguration | Missing hardening, insecure defaults, open cloud storage, verbose errors |
| 6 | Vulnerable and Outdated Components | Using unsupported/out-of-date components |
| 7 | Identification and Authentication Failures | Attacks on identity, authentication, session management |
| 8 | Software and Data Integrity Failures | No protection against integrity violations (e.g., untrusted plugins/CDNs) |
| 9 | Security Logging and Monitoring Failures | Breaches go undetected without logging/monitoring |
| 10 | Server-Side Request Forgery (SSRF) | App fetches a remote resource without validating the user-supplied URL |

## Skills Practiced

- Shell terminology and types
- TCP/UDP fundamentals and well-known ports
- OWASP Top 10 categories

## Key Takeaways

- Memorizing common ports by number is a core pentester skill that speeds up enumeration.
- TCP = reliable/connection-oriented; UDP = fast/connectionless, no delivery guarantee.
- The OWASP Top 10 is a starting checklist, not an exhaustive vulnerability list.
