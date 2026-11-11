# Footprinting Lab — Hard

## Status

<badge>Completed</badge>

## Lab notes

The Hard practical lab was completed against a Linux host.

## Enumeration and attack chain

1. TCP and UDP Nmap scans identified SSH, POP3, IMAP/IMAPS, POP3S, and SNMP.
2. `onesixtyone` identified an SNMP community string. `snmpwalk` exposed host information and credentials for a local user.
3. Password-based SSH authentication was not accepted, so enumeration continued through IMAPS.
4. `openssl s_client` established a TLS connection to IMAPS. Manual IMAP commands enumerated mail folders and retrieved a message containing SSH key material.
5. The key was saved with restrictive permissions and used for SSH authentication.
6. After obtaining a shell, `.bash_history` and `.mysql_history` revealed previous MySQL activity and database/table names.
7. The MySQL client was used to inspect the `users` database and query the target account.

## Tools used

- Nmap
- onesixtyone
- snmpwalk
- openssl s_client
- IMAP commands
- SSH client
- Linux shell history inspection
- MySQL client

## Key takeaways

- A failed authentication method on one service does not invalidate a credential or access path elsewhere.
- SNMP can disclose operational information and, when misconfigured, credentials.
- Mail services can contain sensitive operational artifacts such as SSH keys.
- Shell history can reveal previous administrative activity and database usage.
- The strongest lesson was correlating evidence across SNMP, IMAPS, SSH, Linux user artifacts, and MySQL.

Credentials, private keys, flags, and target-specific secrets are intentionally excluded from this public repository.
