# Connecting Using VPN

## Overview

A VPN (virtual private network) is a secured communications channel over a shared public network, letting us connect to and access resources on a private network as if we were directly connected to it. It encrypts traffic to prevent eavesdropping. At a high level, our connection is routed through the VPN server rather than our ISP, so data appears to originate from the VPN server's public IP.

## Learning Objectives

- Understand the two main types of remote access VPN
- Connect to the HTB lab network via OpenVPN
- Understand the networking changes a VPN connection introduces

## Types of Remote Access VPN

| Type | Description |
|---|---|
| SSL VPN | Uses the browser as the client; connects to an SSL VPN gateway; can be limited to web apps or extend to the internal network without installing software |
| Client-based VPN | Requires client software; once connected, the host works mostly as if directly on the company network, within the limits of the server config |

## Why Use a VPN

Commercial VPN services (NordVPN, Private Internet Access) can obscure browsing traffic and disguise a public IP, but offer no guarantee of anonymity or privacy — the provider could log data or not follow its advertised practices. Useful for bypassing network/firewall restrictions or on hostile networks (e.g., airport Wi-Fi). **Never rely on a VPN to protect you from the consequences of nefarious activity.**

## Connecting to HTB VPN

HTB (and similar vulnerable-lab services) require connecting via VPN to reach the private lab network — hosts inside cannot reach the internet directly. Treat this network as **hostile**:

- Connect only from a VM
- Disallow SSH password authentication on the attack VM
- Lock down any web servers
- Never leave sensitive data on the attack VM; don't reuse the same VM for HTB/CTFs and client work

```bash
sudo openvpn user.ovpn
```

Output ending in `Initialization Sequence Completed` confirms a successful connection.

Check the VPN adapter:

```bash
ifconfig
```

Look for a `tun0` interface with an assigned IP (e.g., `10.10.x.2`).

Check reachable networks:

```bash
netstat -rn
```

Example routing table shows the HTB Academy machine network (e.g., `10.129.0.0/16`) reachable via `tun0` through the `10.10.14.0/23` VPN network.

## Skills Practiced

- Connecting to a lab network via OpenVPN
- Verifying VPN connectivity (`ifconfig`, `netstat -rn`)
- Treating lab/VPN networks as hostile

## Key Takeaways

- HTB and similar labs require VPN access; the target network has no direct internet access.
- Always verify the `tun0` interface and routing table after connecting.
- Treat any pentest lab network as hostile — isolate it from sensitive data and credentials.
