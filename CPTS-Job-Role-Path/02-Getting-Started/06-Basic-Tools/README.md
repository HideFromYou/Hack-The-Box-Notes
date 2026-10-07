# Basic Tools

## Overview

SSH, Netcat, Tmux, and Vim aren't pentesting tools per se, but are used daily and are critical to the penetration testing workflow.

## Learning Objectives

- Use SSH for remote access and as a pivot point
- Use Netcat for banner grabbing and port interaction
- Use Tmux to manage multiple terminal sessions
- Use Vim for basic file editing on remote systems

## Using SSH

Secure Shell: port 22 by default, secure remote access. Supports password or passwordless (public/private key pair) authentication. Can access systems on the same network or the internet, forward/proxy to other networks, and transfer files. Client-server model (e.g., OpenSSH).

Cleartext credentials or a private key found during an assessment can be used to connect directly. SSH is typically much more stable than a reverse shell and can serve as a "jump host" to pivot, transfer tools, and set up persistence.

```bash
ssh Bob@10.10.10.10
```

## Using Netcat

A network utility for interacting with TCP/UDP ports (`netcat`/`ncat`/`nc`). Primary use: connecting to shells. Can also connect to any listening port and interact with the service directly:

```bash
netcat 10.10.10.10 22
# SSH-2.0-OpenSSH_8.4p1 Debian-3
```

This is called **banner grabbing** — identifying a service from the banner it sends on connect. Preinstalled on most Linux distros; Windows alternatives: a Windows netcat build, or PowerCat (PowerShell). Netcat can also transfer files.

**Socat** is a similar, more capable tool (port forwarding, serial devices, upgrading a shell to a fully interactive TTY) and is worth having in the toolkit.

## Using Tmux

A terminal multiplexer: multiple windows in one terminal, switchable on the fly.

```bash
sudo apt install tmux -y
tmux
```

- Prefix key: `CTRL+B`
- New window: prefix, then `C`
- Switch window: prefix, then window number
- Vertical split: prefix, then `SHIFT+%`
- Horizontal split: prefix, then `SHIFT+"`
- Switch panes: prefix, then arrow keys

Tmux also supports logging sessions, useful for documentation during engagements.

## Using Vim

A keyboard-only text editor, often already present on compromised Linux systems, so learning it allows editing files remotely without a GUI.

```bash
vim /etc/hosts
```

Opens in **normal mode** (read-only navigation). Press `i` for **insert mode** to edit; `Esc` to return to normal mode.

| Command | Description |
|---|---|
| `x` | Cut character |
| `dw` | Cut word |
| `dd` | Cut full line |
| `yw` | Copy word |
| `yy` | Copy full line |
| `p` | Paste |

Prefix a number to repeat a command (e.g., `4yw` copies 4 words).

Command mode (press `:`):

| Command | Description |
|---|---|
| `:1` | Go to line 1 |
| `:w` | Write/save |
| `:q` | Quit |
| `:q!` | Quit without saving |
| `:wq` | Write and quit |

## Practical Notes (Exercise)

Completed the section's banner-grabbing exercise: the spawned target ran SSH on a **non-standard port**, shown in the Target(s) box as `IP:PORT`, not port 22. Key lessons:

- Always check the actual port given in the Target(s) panel before assuming 22/80/443.
- `nc -v <IP> <PORT>` shows connection state (refused/timed out/succeeded) when nothing appears to happen.
- Reaching lab IPs (e.g., `10.129.x.x`) requires either Pwnbox or a VPN-connected VM.
- A package version suffix (e.g., Ubuntu's `4ubuntu0.1`) can hint at the OS release, but distro backports can patch CVEs without changing the upstream version string — always verify with `nmap -sV` rather than trusting a raw banner alone.

## Skills Practiced

- SSH remote access and pivoting
- Banner grabbing with Netcat
- Session management with Tmux
- Remote file editing with Vim

## Key Takeaways

- SSH access (via leaked creds/keys) is often more stable than a reverse shell.
- Banner grabbing is a fast first step to fingerprint a service.
- Tmux and a working TTY are essential for staying organized during a long engagement.
- Vim is often the only editor available on a compromised host — worth knowing the basics cold.
