---
layout: post
title: "Undervoltage Caused Trolley-X to Hold STOP"
date: 2026-08-28 18:00:00 +0400
categories: [Updates, Research]
---

After we connected the LiDAR and UWB, we placed Trolley-X on blocks. We started the full ROS 2 stack on the Pi 5 for the first time. The stack started, but the trolley did not move. The investigation lasted several hours. It identified a fault in one buck converter, not in the software.

### Symptom: LiDAR Does Not Scan

Each launch produced the same result. The `sllidar` node connected. It read the serial number, firmware, and OK health status. After approximately two seconds, it stopped with `Can not start scan: 80008002`. The safety supervisor recorded the following message:

```
STOP zone: scan timeout -> holding STOP
```

The trolley therefore performed the required safe action. It held STOP because it had no scan data.

### Initial Checks

We first checked for a stale driver process that held the serial port. We then stopped `ModemManager`, which can probe `/dev/ttyUSB*` during communication. The result did not change. The scan motor was spinning and the health status was OK. We therefore set the driver to Standard mode. The error remained `80008002`. The failure was deterministic. It occurred at the same time and produced the same error during every test. This result indicated a repeatable hardware condition.

### Kernel Evidence

`dmesg` gave the whole game away:

```
hwmon hwmon2: Undervoltage detected!
cp210x ttyUSB0: failed set request 0x12 status: -110
```

The Pi's under-voltage detector reported a fault. The LiDAR CP210x USB adapter timed out during a control transfer. The value `-110` indicates a timeout. The buck converter display showed approximately 5&nbsp;V. The display samples slowly and shows an average value. The scan failure was caused by a millisecond-scale transient. The display could not show this transient.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-lm2596-rail-voltage.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-lm2596-rail-voltage.jpeg' | relative_url }}" alt="LM2596 buck converter reading roughly 5 volts" style="cursor: zoom-in;" />
  </a>
</div>

The mechanism is now clear. Reading the LiDAR serial number creates a small electrical load, so that operation succeeds. A scan command powers the laser and receiver. The motor is already spinning at this time. The combined load reduces the 5&nbsp;V rail below its threshold. The converter cannot recover quickly enough. The CP210x then loses the connection during the handshake. The SDK reports this loss as a scan timeout.

### Root Cause: Shared Buck Converter

A single LM2596 supplies the Pi 5, the RPLiDAR through the Pi USB port, and one UWB sensor. The converter can supply approximately 2 to 2.5 A before its output decreases and its temperature increases. The Pi 5 can require up to 5 A under load. The LiDAR scan-start transient adds to this load. The 5 V rail then collapses. The thin wire between the converter and the Pi causes an additional voltage drop. The LiPo voltage also decreases during discharge. These conditions combine to produce the fault.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-chassis-wiring-topdown.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-chassis-wiring-topdown.jpeg' | relative_url }}" alt="Top-down view of the Trolley-X wiring loom and shared power rail" style="cursor: zoom-in;" />
  </a>
</div>

### Safety Response

The trolley lost its scan data and held STOP. It did not drive without obstacle data. The under-voltage caused a sensor dropout. The fail-safe responded as designed. This test provides a reliability result for the braking model because the fault occurred in a real hardware condition.

The corrective action is electrical. Provide the LiDAR with a separate regulated 5&nbsp;V supply. Power the Pi from a converter that supports its peak current with additional capacity. Do not connect the three loads to one converter output. We are calculating the requirements for this redesign. We will review the values with our supervisor before we purchase parts.
