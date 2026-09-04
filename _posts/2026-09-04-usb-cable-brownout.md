---
layout: post
title: "A Faulty USB Cable and a Boot Brownout"
date: 2026-09-04 18:00:00 +0400
categories: [Updates, Research]
---

The previous post corrected the under-voltage fault with a USB power bank. The rail then held 4.90–4.99&nbsp;V under a full load. A new power fault appeared during a routine cable change. This post records the fault and the diagnosis.

### The Fault

I replaced the micro-USB cable of the RPLIDAR A1. The Pi 5 then froze. The red LED turned on.

On the Pi 5, the red LED is the power LED. It stays off in normal operation. It turns on only when the board detects a power fault. A steady red LED is therefore a power warning, not a boot state.

### The Boot Loop

I pressed the reset button. The Pi did not start on the first attempt. The LED cycled green, red, green, red, green. The board rebooted two or three times. It then stayed green.

This pattern is a brownout-retry loop. The board starts to boot. The current demand rises. The 5&nbsp;V rail falls below its threshold. The power detector trips and resets the board. The board then tries again. After a few attempts the rail holds and the boot completes.

The Pi draws its highest current during boot. The CPU starts. The USB devices enumerate. A supply that is stable at idle can still sag during this surge.

### Isolation of the Cause

I removed the LiDAR. I then booted the Pi. The boot was clean. The kernel log showed no under-voltage message. Only the Arduino appeared on the USB bus.

I read the throttle register:

```
sudo vcgencmd get_throttled
throttled=0x0
```

The value `0x0` means no under-voltage and no throttle event in this session. The Pi is healthy. The fault appears with the LiDAR connected, and not without it.

The throttle register has a limit. It resets at each power-on. A failed boot resets the board before the kernel runs. The register therefore cannot record the failed attempts. It records only the boot that succeeds.

### Evidence of the Resets

The kernel log kept one trace of the earlier resets:

```
systemd-journald: File /var/log/journal/.../system.journal corrupted or
uncleanly shut down, renaming and replacing.
```

A brownout is an unclean shutdown. An unclean shutdown corrupts the open journal file. The service renames and rebuilds the file, so the fault is not fatal. The root filesystem still mounted correctly. The message confirms that the board lost power without a clean stop.

### Cause and Corrective Action

The LiDAR motor draws a current surge at start-up. This surge runs through the cable that I replaced. A thin or damaged micro-USB cable adds resistance or a fault. The extra load pulls the 5&nbsp;V rail below its threshold. The board then browns out.

The corrective actions are:

- Use a short, thick, data-rated micro-USB cable for the LiDAR.
- Move the LiDAR load off the Pi USB port where possible.
- Test a cold boot with an official 5&nbsp;V / 5&nbsp;A Pi 5 supply. A clean boot confirms the power bank as the weak source.
- Back up the SD card. Repeated brownouts can corrupt the card.

### Result

The Pi is not damaged. The root filesystem is intact. The power bank holds the rail at steady load. The weak point is the cold-start surge, and it is worst when the LiDAR shares the Pi supply. The permanent power design must supply the Pi boot current with margin. The next post continues with UWB ranging.
