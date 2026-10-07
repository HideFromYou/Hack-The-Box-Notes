# Knowledge Check

## Overview

The module's final skills assessment: attack a target independently, applying the full methodology covered across the module, with no official walkthrough provided. This lesson captures the general assessment approach plus methodology notes from working through the exercise target.

## Learning Objectives

- Apply the full enumeration-to-root methodology independently, end to end
- Recognize when a seemingly dead attack surface (a broken upload feature) warrants pivoting to an alternate code path
- Use `searchsploit` results critically — matching the right CVE to the right exposed functionality
- Apply GTFOBins-style reasoning to a sudo misconfiguration on a scripting-language binary

## Assessment Approach

No walkthrough is provided for this stage — only general tips, applying everything from earlier in the module:

1. **Enumeration/scanning** — a quick Nmap scan for open ports first, followed by a full port scan.
2. **Web footprinting** — check any web ports for running applications and hidden files/directories (whatweb and Gobuster are the suggested tools). If a hostname is identified, consider adding it to `/etc/hosts` — not always required, but often necessary for applications that rely on their own vhost for certain functionality.
3. **Exploit research** — once the technology stack is identified, search Searchsploit and Google for public exploits or manual exploitation techniques.
4. **Shell stabilization** — after an initial foothold, use the Python3 pty trick to upgrade to a full interactive TTY.
5. **Privilege escalation enumeration** — combine manual checks with automated helper scripts (LinEnum, LinPEAS) to look for misconfigurations, vulnerable services, and cleartext credentials. Organize findings offline rather than trying to track everything in your head.
6. Worth noting generally: an assessment target can have more than one valid path to an initial foothold (for example, both a Metasploit module and a manual technique), and more than one well-known privilege escalation technique to root — understanding multiple approaches is more valuable than finding just one that works.

## Practical Notes (Exercise)

Methodology notes from working through this stage's target — no IP, hostname, or flag values recorded, since this was a private assessment instance rather than a shareable named box.

### Enumeration

The target ran a CMS with an exposed admin panel reachable using default credentials. `searchsploit <cms name>` surfaced multiple historical vulnerabilities across different versions of that CMS — a reminder that a product-name-only search can return several unrelated-looking advisories that need to be cross-referenced against the exact functionality actually exposed.

### Exploring the Upload Feature's Dead End

The admin panel's file-upload feature (used for a media/file manager) turned out to be built on a legacy Flash-based (SWF) uploader. Since Flash has been fully deprecated and removed from all modern browsers, the upload button did nothing visible when clicked. This was diagnosed by inspecting the page's `<head>`/sidebar JavaScript (locating the `.swf` reference) via View Source and the browser's DevTools Network/Console tabs.

Several manual extension-bypass tricks were tried directly against the upload endpoint with curl, reusing the authenticated session cookie and the page's `<noscript>` fallback form (which posts to the same backend endpoint the broken Flash button would have used). A trailing special character on the filename bypassed the application's extension blacklist (which did an exact-match check), and the file did save to the uploads directory — confirmed visually through the file manager listing — but the web server's PHP handler required the filename to end in a literal `.php`, so nothing executed. Plain and alternate PHP-related extensions were all rejected cleanly by the blacklist.

### Pivoting to the Theme Editor

Rather than continuing to fight the blocked upload path, `searchsploit` surfaced a known unauthenticated RCE CVE for the same CMS, with an existing Metasploit module. This CVE doesn't touch the upload form at all — it abuses the admin panel's **theme editor**, which lets an authenticated admin directly overwrite theme template files with arbitrary content saved as `.php`. That code path has no extension filtering whatsoever, since it's meant for legitimate PHP template editing.

The Metasploit module automates the full chain: it leaks the application's API key/salt to forge a valid admin session cookie without needing real credentials, fetches a CSRF token, posts the PHP payload to the theme editor endpoint, then requests the saved file to trigger it — returning a shell as the web server's service account.

### Privilege Escalation via GTFOBins

```bash
sudo -l
```

showed a passwordless sudo rule for a scripting-language interpreter binary (PHP) — a textbook GTFOBins case. The interpreter's own code-execution function, invoked through `sudo`, inherits root privileges since sudo doesn't drop them for calls made from within the privileged interpreter process itself:

```bash
sudo php -r 'system("/bin/sh -i");'
```

This returned an immediate root shell.

## Skills Practiced

- Independent end-to-end methodology application without a walkthrough
- Diagnosing a non-functional legacy feature (Flash) rather than assuming it's simply broken and unexploitable
- Critical cross-referencing of `searchsploit` results against actual exposed functionality
- GTFOBins-style sudo misconfiguration exploitation on an interpreter binary

## Key Takeaways

- When an obvious attack surface is broken or heavily filtered, look for a different feature in the same application that achieves the same underlying goal (here, arbitrary PHP write) through a completely different, unfiltered code path.
- `searchsploit` searches by product name alone can surface several unrelated CVEs — always cross-reference which one actually matches the exposed functionality before committing to it.
- A passwordless sudo rule on any interpreter (PHP, Python, Perl, etc.) is almost always a direct root shell via GTFOBins — check interpreter binaries specifically, not just obvious system utilities.
- Finishing an assessment independently, with no walkthrough, is the clearest signal that the module's methodology has actually been internalized rather than just followed along.
