# Common Pitfalls

## Overview

A collection of common, easily-missed issues that cost time during labs and engagements — mostly VPN connectivity problems, a classic Burp Suite proxy mistake, and SSH key troubleshooting.

## Learning Objectives

- Systematically troubleshoot HTB VPN connectivity issues
- Recognize and fix the "browser stopped loading pages" Burp Suite proxy mistake
- Regenerate SSH keys when facing connection issues

## VPN Connectivity Troubleshooting

Work through these checks in order when the VPN seems connected but nothing is reachable:

1. Confirm OpenVPN actually connected successfully — look for `Initialization Sequence Completed` at the end of the connection output:

```bash
sudo openvpn ./htb.ovpn
```

2. Check that the VPN adapter actually has an IP address assigned:

```bash
ip -4 a show tun0
```

3. Check the routing table to confirm which networks route through the VPN interface:

```bash
sudo netstat -rn
```

4. Ping the VPN gateway to confirm basic connectivity:

```bash
ping -c 4 <gateway-ip>
```

**One-device limit:** the HTB VPN only allows a single device connected at a time. A second simultaneous connection attempt (for example, connecting from Pwnbox and a local VM at the same time) will fail outright — this isn't a configuration problem, it's enforced by design.

**Latency/lag:** if there's noticeable lag talking to lab targets, switch to a geographically closer VPN server region from the HTB dashboard (Lab Access → Labs → OpenVPN). Free accounts get 1–3 servers per region; VIP unlocks access to faster VIP-only servers. Detailed VPN troubleshooting guidance is also available on the official HTB Help page.

## Burp Suite Proxy Issues

A very common mistake: closing Burp Suite without turning off the browser's proxy configuration first. The browser keeps trying to route traffic through a proxy that's no longer running, and pages simply stop loading — with no obvious error pointing at the actual cause.

Fix: check the FoxyProxy icon (or the browser's manual connection settings) and make sure it's set back to "Turn Off" whenever browsing suddenly breaks after closing Burp.

## SSH Key Issues

If facing persistent SSH connectivity or authentication problems, regenerating the local SSH keypair is often the fastest fix:

```bash
ssh-keygen
```

This is interactive — it prompts for a save path (defaults to `~/.ssh/id_rsa`) and an optional passphrase.

## Skills Practiced

- Systematic VPN connectivity troubleshooting
- Diagnosing proxy-related browser connectivity issues
- SSH key regeneration

## Key Takeaways

- Most "the VPN isn't working" issues are actually one of a handful of specific checks — adapter IP, routing table, gateway reachability — work through them in order instead of restarting the connection blindly.
- The HTB VPN's single-device limit is a frequent source of confusing, intermittent-looking failures that have nothing to do with configuration.
- A browser that suddenly can't load anything after closing Burp is almost always a leftover proxy setting, not a network issue.
