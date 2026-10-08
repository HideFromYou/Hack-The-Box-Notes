# Firewall Evasion - Easy Lab

## Overview

The first of three hands-on labs applying the evasion techniques from the Firewall and IDS/IPS Evasion lesson against a client's IDS/IPS-protected machine. In this fictional framing, the client progressively hardens their defenses between each test. The Easy lab's goal is to identify the target's operating system.

## Learning Objectives

- Recognize when loud, default scan behavior will trigger a defensive response
- Apply quiet, targeted scanning instead of broad/aggressive scanning
- Use manual banner grabbing to fingerprint a target without relying on noisy automated version/OS detection

## Scenario

The target exposes a `status.php` page displaying an alert counter. Scanning too loudly or too many times triggers a ban — so quiet, careful scanning technique matters more than raw speed. The goal is to identify the target's OS.

## Practical Notes (Exercise)

Target: `<TARGET_IP>`.

**Methodology:**

1. Before scanning, the alert-counter page (`status.php`) was kept in view to monitor how the target's detection system reacted to each probe — this turned the counter into direct, real-time feedback on how "loud" a given scan was, rather than guessing.
2. Instead of reaching for an aggressive `-A` or `-O` scan (which bundles version detection, OS detection, traceroute, and default scripts into one loud burst of traffic), the approach started with a quiet, minimal SYN scan against a small, targeted port range to find what was open without triggering the ban threshold.
3. Once an open port was identified (an SSH service), OS fingerprinting was done **manually** rather than via Nmap's own `-O` OS-detection engine — a plain `nc` connection to the open port was used to grab the raw service banner directly.
4. The banner returned by the manual `nc` connection was enough to infer the underlying OS/distribution without ever running a loud, multi-probe OS-detection scan against the target.
5. This mirrors the core lesson from the Firewall/IDS/IPS Evasion section: a single quiet SYN scan plus a manual banner grab accomplishes the same enumeration goal as an aggressive scan, without the volume of traffic that trips an alert threshold.

**Key technique:** quiet/minimal SYN scan to find the open port, then manual `nc` banner grab against that port to fingerprint the OS — avoiding any Nmap OS-detection (`-O`) or aggressive (`-A`) scan that would spike the alert counter.

## Skills Practiced

- Using an application-level feedback signal (an alert counter) to calibrate scan loudness
- Minimal, targeted SYN scanning in place of broad/aggressive scanning
- Manual banner grabbing with `nc` as a quiet substitute for Nmap's OS-detection engine

## Key Takeaways

- When a target exposes direct feedback on detection (like a visible alert counter), that feedback should drive scan technique choice, not just intuition about what "seems quiet."
- OS fingerprinting doesn't require Nmap's `-O` engine — a single manual banner grab against one open service can be enough, and it's far quieter.
- The fewer probes sent, the less opportunity a defensive system has to correlate them into a detected attack pattern.
