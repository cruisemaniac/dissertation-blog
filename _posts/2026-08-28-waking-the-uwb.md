---
layout: post
title: "UWB Power-On Reset"
date: 2026-08-28 21:00:00 +0400
categories: [Updates, Research]
---

The temporary undervoltage correction worked. We increased the LM2596 output so that the Pi 5 received approximately 5&nbsp;V under load. This is not the final power design. The change allowed the RPLiDAR to spin and scan. It also allowed the UWB module to operate from the Pi. The UWB module then failed to communicate through serial after each boot. A manual reset sequence restored communication.

### Symptom: UWB Does Not Communicate After Boot

At each power-up, the REYAX RYUW122 received power but did not send data through its serial port. Communication returned only after a manual reset sequence. We connected `NRST` to ground at header pin 9. We then connected `NRST` to the 3.3&nbsp;V rail at pin 17. The module communicated after this low-to-high transition. The same sequence was required after every boot.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-pi5-integration.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-pi5-integration.jpeg' | relative_url }}" alt="Raspberry Pi 5 header where the UWB reset line is jumpered" style="cursor: zoom-in;" />
  </a>
</div>

### Cause: Reset Timing

This is a power-on-reset sequencing problem. It is not a wiring problem. The module must release `NRST` after the supply voltage has stabilized. The release must be a clean low-to-high transition. When `NRST` connects directly to 3.3&nbsp;V, it rises with the supply during power-up. The chip may not be ready at this time. The module then enters a state in which the UART does not start. The manual low-to-high transition creates the required reset edge.

### Corrective Action: Automate the Reset

A person cannot perform this reset sequence on an autonomous platform. The reset must occur automatically. Two options were available:

- **Hardware power-on-reset circuit:** Connect a pull-up resistor of approximately 10&nbsp;kΩ from 3.3&nbsp;V to `NRST`. Connect a capacitor of approximately 10&nbsp;µF from `NRST` to ground. At power-up, the capacitor holds `NRST` low while it charges. The line rises approximately 100&nbsp;ms later, after the supply has stabilized. The module then leaves reset. A small diode across the resistor allows the capacitor to discharge quickly when power is removed. The reset circuit then prepares for the next boot.

- **GPIO-controlled reset:** Connect `NRST` to an unused GPIO. A startup service can drive the line low and then high before the UWB node starts. This method is deterministic. It also lets software reset the module when required. One service can reset more than one anchor.

We chose the GPIO-controlled reset. We connected `NRST` to GPIO17 (header pin 11). A small `systemd` service runs at boot. The service drives `NRST` low for about 150&nbsp;ms, then releases it high, then holds it high. The module has no reliable internal pull-up, so the service must hold the line. The manual reset ritual is no longer needed.

### Result

The reset is now automatic. After a reboot, the module answers an `AT` command with `+OK` on its serial port. No manual step is needed.

**Update, 2026-09-04:** The design now carries two UWB anchors, one per front pillar. Each anchor uses its own Pi&nbsp;5 UART. The right anchor is on `/dev/ttyAMA0` and the left anchor is on `/dev/ttyAMA2`. The `uwb-reset` service drives both reset lines: GPIO17 for the right anchor and GPIO27 for the left. Both anchors answer `AT` with `+OK` after boot. The next task is to read range data from both anchors into ROS&nbsp;2. The Kalman filter will later fuse this data into a stable follow-me signal.
