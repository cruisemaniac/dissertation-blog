---
layout: post
title: "Trolley-X Power Rail Measurements and Correction"
date: 2026-09-04 09:00:00 +0400
categories: [Updates, Research]
---

The previous post described an under-voltage fault. The fault reduced the 5&nbsp;V rail below its threshold. It disconnected the LiDAR during the handshake. The trolley correctly held STOP. We then measured the system and designed a power correction before purchasing parts. The measurements identified a fault that the buck converter display did not show.

### Handheld Meter Measurements

We measured each point in the power chain. The LM2596 output measured 5.48&nbsp;V. The wire end near the Pi measured 5.5&nbsp;V. These readings indicated that the rail was within the expected range. The Pi still experienced brownouts.

A handheld meter samples slowly and displays an average value. The scan failure is caused by a millisecond-scale transient. This load step starts and ends before the display can update. The meter therefore did not show the fault. We needed to measure the voltage at the Pi.

### Pi Voltage Measurement

The Pi 5 contains a power-management IC. The command `vcgencmd pmic_read_adc` reads the voltage at the Pi. The `EXT5V_V` value represents the 5&nbsp;V voltage after the wires and connectors.

At idle, the value was 5.05&nbsp;V. The wire-end measurement was 5.5&nbsp;V. The voltage had decreased by 0.45&nbsp;V before it reached the Pi. We then loaded the CPU with `stress-ng`. The `EXT5V_V` value decreased to **4.61&nbsp;V**. This value is below the Pi 5 operating range. The voltage dip activates the under-voltage detector and disconnects the LiDAR USB adapter.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-lm2596-rail-voltage.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-lm2596-rail-voltage.jpeg' | relative_url }}" alt="LM2596 buck converter reading roughly 5.5 volts at its output" style="cursor: zoom-in;" />
  </a>
</div>

### Voltage Drop at the Connectors

The wire-end measured 5.5&nbsp;V. The Pi measured 4.61&nbsp;V under load. Most of the voltage drop occurred in the final connection. Dupont jumpers supplied power to the Pi 5 GPIO pins. A small header contact has measurable resistance when it carries the Pi 5 peak current. A fast load step therefore creates a voltage drop at the connector. A measurement on the supply side cannot show a fault that occurs in the connector.

We confirmed this result by splitting the feed. We used two 5&nbsp;V pins and two ground pins instead of one pin for each connection. The rail then remained above 4.85&nbsp;V under load. The parallel contacts reduced the voltage drop at each contact. This result confirmed the cause. However, friction-fit jumpers are not a suitable long-term power connection for an autonomous platform.

### Corrective Action: Separate Pi Supply

We stopped using the GPIO header as the main power connection. The Pi 5 now uses a 10000&nbsp;mAh USB power bank through USB-C. The setting `usb_max_current_enable=1` allows the firmware to negotiate the full current budget.

The result was immediate. `EXT5V_V` remains at **4.90–4.99&nbsp;V** under the worst test condition. The test used a fully loaded CPU, a spinning and scanning LiDAR, an active UWB module, and the Arduino on the bus. The `dmesg` output contains no under-voltage message. The USB-C connector and the power bank protection system support this current more reliably than a Dupont pin.

The power bank also isolates the computer and sensor supply from the motor supply. The LiPo and the L298N drivers carry out the high-current motor work separately. A motor surge therefore does not share a supply path with the Pi. This separation reduces the risk that a motor surge will reduce the sensor voltage. LiDAR data is required for the robot safety function, so this separation is important.

### Limitation and Next Step

The power bank is a temporary solution. It is not the final power design. The permanent design will use an onboard buck converter sized for the Pi peak current. The converter will use soldered and strain-relieved connections. It will connect directly to the battery pack. The power bank provides a reliable rail while we continue work on ranging and braking.

This result completes the investigation from the previous post. The fail-safe held STOP when the rail collapsed. The corrected supply now prevents the rail from collapsing during the tested load. The trolley holds STOP when it has no scan data. The cause of the previous sensor dropout is now measured and corrected. The next post will continue with sensor ranging and braking.
