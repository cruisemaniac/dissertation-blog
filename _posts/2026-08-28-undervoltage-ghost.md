---
layout: post
title: "Chasing a Ghost: The Undervoltage That Kept Trolley-X in STOP"
date: 2026-08-28 18:00:00 +0400
categories: [Updates, Research]
---

Fresh off getting the LiDAR and UWB talking, we propped Trolley-X up on blocks and launched the full ROS 2 stack on the Pi 5 for the very first time. It came up — and then flatly refused to move. What followed was a multi-hour hunt that ended somewhere we did not expect: not in the code, but in a single buck converter.

### The Symptom: A LiDAR That Reads but Won't Scan
Every launch told the same story. The `sllidar` node connected, read the serial number, firmware, and a health status of OK — and then, roughly two seconds in, died with `Can not start scan: 80008002`. At the same moment, our safety supervisor logged exactly what it is designed to log when it is flying blind:

```
STOP zone: scan timeout -> holding STOP
```

So the trolley was doing the *right* thing — it just had nothing to see.

### The Red Herrings
The first suspects were the boring ones. A stale driver process holding the serial port. Then `ModemManager`, which loves to probe `/dev/ttyUSB*` and corrupt a handshake — a classic on Raspberry Pi. We stopped it. No change. Then the scan mode: the motor was clearly spinning and the health reads were clean, so we forced the driver into plain Standard mode. Still `80008002`. The failure was *deterministic* — same error, same two-second mark, every single time — and that was the clue. Random contention is flaky. This was a wall.

### The Kernel Tells the Truth
`dmesg` gave the whole game away:

```
hwmon hwmon2: Undervoltage detected!
cp210x ttyUSB0: failed set request 0x12 status: -110
```

The Pi's own under-voltage detector was firing on repeat, and the LiDAR's CP210x USB adapter was timing out on a control transfer (`-110` is a timeout). The little voltmeter on the buck reads a comfortable ~5&nbsp;V — but that display samples slowly and shows an average. The brown-out that breaks the scan is a *millisecond* transient it can never show.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-lm2596-rail-voltage.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-lm2596-rail-voltage.jpeg' | relative_url }}" alt="LM2596 buck converter reading roughly 5 volts" style="cursor: zoom-in;" />
  </a>
</div>

The mechanism finally made sense. Reading the LiDAR's serial number is a tiny electrical blip, so it succeeds. But *commanding a scan* powers up the laser and receiver on top of the already-spinning motor. That combined load step drags the 5&nbsp;V rail below threshold faster than the converter can recover, the CP210x drops out mid-handshake, and the SDK surfaces the whole thing as a scan timeout.

### Root Cause: One Buck, Three Mouths
Here is the real problem. A single LM2596 is feeding the Pi 5, the RPLiDAR (through the Pi's USB), *and* a UWB sensor — all off the same output pin. A cheap LM2596 realistically holds around 2&nbsp;to&nbsp;2.5&nbsp;A before it droops and heats; a Pi 5 alone can ask for up to 5&nbsp;A under load. Stack the LiDAR's scan-start transient on top of that and the rail simply collapses. The thin hookup wire between the converter and the Pi drops even more voltage under current, and the LiPo sags as it discharges. Every link in the chain is working against us.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-chassis-wiring-topdown.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-chassis-wiring-topdown.jpeg' | relative_url }}" alt="Top-down view of the Trolley-X wiring loom and shared power rail" style="cursor: zoom-in;" />
  </a>
</div>

### The Silver Lining: The Safety Layer Did Its Job
It is worth stopping on the one thing that went completely right. The trolley lost its scans, assumed the worst, and held STOP rather than driving blind into a room it could no longer see. Under-voltage led to a sensor dropout, and the fail-safe engaged exactly as designed. That is not a footnote — it is a genuine reliability result for the braking model, and we got to watch it fire under a real fault instead of a contrived one.

The fix from here is electrical, not software: give the LiDAR its own regulated 5&nbsp;V supply, feed the Pi from a converter sized for its true peak with headroom to spare, and stop three hungry loads from fighting over one tired rail. We are sizing that power redesign now — and taking the numbers to our supervisor before we commit to parts. More once Trolley-X can hold its rail steady under a full scan.
