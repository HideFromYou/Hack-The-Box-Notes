# Firewall Evasion - Hard Lab

## Overview

The third and final of three hands-on labs applying the evasion techniques from the Firewall and IDS/IPS Evasion lesson. After further hardening — including the admin's own IDS/IPS training and a modified service communication setup — this lab requires identifying the version of a newly added service, hinted as handling "large amounts of data" for the client's customers.

## Learning Objectives

- Apply the `--source-port` trusted-port trick to reach a service hidden behind strict firewall rules
- Recognize when Nmap's own `-sV` probe is too "loud" at the application layer and gets blocked even after the firewall lets the SYN through
- Use a manual, quiet `ncat`/`nc` connection as a substitute for automated version detection in that scenario

## Scenario

After further hardening, the exercise requires identifying a specific added service's version — hinted as something handling "large amounts of data" for the client's customers. This maps directly onto the source-port manipulation example from the Firewall and IDS/IPS Evasion lesson: a service running on an unusual high port.

## Practical Notes (Exercise)

Target: `<TARGET_IP>`.

**Methodology:**

1. A default scan against the suspected high port showed it as filtered (reported with a generic service guess like `ibm-db2`), consistent with a firewall rule blocking the connection by default.
2. Applying the `--source-port 53` trick — making the scan traffic appear to originate from port 53, a port loosely configured firewalls tend to trust as legitimate DNS response traffic — caused the SYN scan against that same port to succeed (SYN-ACK received), confirming the firewall was selectively trusting traffic that looked like it came from DNS.

```bash
sudo nmap <TARGET_IP> -p<PORT> -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

3. The important nuance: getting past the firewall at the SYN level is not the same as getting a usable application-layer response. Nmap's own `-sV` probe against this port — even after the SYN succeeded with `--source-port 53` — got reset by the firewall and reported back as `tcpwrapped`. This happens because Nmap's active version-detection probe sends its own data to elicit a response, and that probe traffic itself looks "noisy"/suspicious to the firewall, even though the initial three-way handshake was accepted.
4. The fix was to drop Nmap's automated version detection entirely for this port and instead open a **manual, quiet connection** with `ncat`, still using the same trusted source port, and just wait silently rather than sending any probe data:

```bash
ncat -nv --source-port 53 <TARGET_IP> <PORT>
```

5. Because `ncat` only opens the connection and waits rather than actively probing, it didn't trigger the same firewall reset that Nmap's `-sV` did — and the server sent its real banner on its own once the connection was simply held open, revealing the actual service and version.

**Key technique:** `--source-port 53` to get the SYN through the firewall, then a manual `ncat`/`nc` connection (same trusted source port, no active probing) in place of Nmap's `-sV`, since Nmap's own version-detection probe is loud enough at the application layer to get reset/`tcpwrapped` even after the handshake itself succeeds.

## Skills Practiced

- Combining source-port manipulation with manual service interaction
- Diagnosing the difference between a firewall blocking a connection vs. blocking a specific probe/traffic pattern after the connection is already established
- Choosing a quiet, passive connection method over an automated active-probing tool when the automated tool itself is the thing getting blocked

## Key Takeaways

- Firewall evasion isn't just about getting past the firewall at the handshake level — the application-layer interaction method matters too, since a loud automated tool can still get blocked even after the initial SYN/ACK succeeds.
- `tcpwrapped` from `-sV` doesn't necessarily mean the service is unreachable — it can mean the firewall is specifically reacting to Nmap's own active probing pattern, distinct from the connection itself.
- A manual `ncat`/`nc` connection that simply opens and listens, without sending active probe traffic, can succeed precisely where an automated version-detection tool fails, because it never gives the firewall an "aggressive scan" pattern to react to.
- Progressive hardening across the three labs (quiet SYN + banner grab, UDP protocol awareness, source-port + manual quiet connection) builds a layered toolkit — each technique on its own would have failed by the Hard lab; stacking them is what gets through.
