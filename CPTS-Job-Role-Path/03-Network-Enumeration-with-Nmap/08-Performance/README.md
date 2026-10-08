# Performance

## Overview

Nmap exposes several performance-tuning options that trade scan speed against accuracy. This lesson covers RTT timeouts, max retries, rate control, and the six named timing templates, each backed by measured tradeoffs between speed and completeness.

## Learning Objectives

- Understand which options control speed, frequency, timeout, and retry behavior
- See the measured tradeoff between RTT timeout aggressiveness and missed hosts
- See the measured tradeoff between retry count and missed ports
- Understand the six timing templates and when more aggressive timing becomes risky

## Performance Options Overview

- **`-T <0-5>`** — speed (named timing templates)
- **`--min-parallelism`** — frequency
- **`--max-rtt-timeout`** — timeout
- **`--min-rate`** — packets sent simultaneously
- **`--max-retries`** — retry count

## RTT Timeouts

The default initial RTT timeout starts at 100ms.

```bash
# default
sudo nmap 10.129.2.0/24 -F

# optimized
sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
```

The optimized scan found **2 fewer hosts** but took only about **1/4 of the time**. A too-short RTT timeout risks missing real hosts — this is a genuine tradeoff, not just a theoretical one.

## Max Retries

Default is 10 retries; it can be reduced to 0 (no re-send if no response — skip immediately).

```bash
sudo nmap 10.129.2.0/24 -F | grep "/tcp" | wc -l                 # default: 23 ports found
sudo nmap 10.129.2.0/24 -F --max-retries 0 | grep "/tcp" | wc -l # reduced: 21 ports found
```

Same tradeoff as RTT timeouts — speed gain, but can overlook real open ports.

## Rates

`--min-rate <N>` tells Nmap to try to send at least N packets per second simultaneously — a significant speed gain when the network is known to be whitelisted/well-provisioned (e.g. a white-box test).

```bash
sudo nmap 10.129.2.0/24 -F -oN tnet.default                  # 29.83s
sudo nmap 10.129.2.0/24 -F -oN tnet.minrate300 --min-rate 300 # 8.67s, same 23 ports found — no loss here
```

## Timing Templates (-T 0-5)

Six named presets control overall aggressiveness, each with pre-set manual option values behind them. The default (no `-T` specified) is `-T3` (normal).

```
-T0 / -T paranoid
-T1 / -T sneaky
-T2 / -T polite
-T3 / -T normal   (default)
-T4 / -T aggressive
-T5 / -T insane
```

Too-aggressive timing risks triggering security system blocks due to the volume of traffic generated.

```bash
sudo nmap 10.129.2.0/24 -F -oN tnet.default  # 32.44s
sudo nmap 10.129.2.0/24 -F -oN tnet.T5 -T 5   # 18.07s, same port count found — no loss here either
```

Full option breakdown per template: https://nmap.org/book/performance-timing-templates.html
More: https://nmap.org/book/man-performance.html

## Skills Practiced

- Tuning RTT timeout, retry count, and packet rate independently
- Measuring the actual speed/accuracy tradeoff of performance tuning rather than assuming it
- Selecting an appropriate timing template for the engagement's stealth requirements

## Key Takeaways

- Every performance option that increases speed does so by reducing how long/how many times Nmap waits for a response — which means every one of them risks silently dropping real results.
- The tradeoff isn't always bad: `--min-rate` and `-T5` showed no loss of accuracy in the measured examples, while `--initial-rtt-timeout`/`--max-rtt-timeout` and `--max-retries 0` did cost real hosts/ports — tune based on what's actually being sacrificed, not just the time saved.
- Aggressive timing isn't just a scanning-speed decision — it's also a detection-risk decision, since high packet volume is what triggers IDS/IPS alerting.
