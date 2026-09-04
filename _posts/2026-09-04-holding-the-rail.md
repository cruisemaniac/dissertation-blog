---
layout: post
title: "Holding the Rail: How Trolley-X Finally Kept Its 5 Volts"
date: 2026-09-04 09:00:00 +0400
categories: [Updates, Research]
---

Last time we chased a ghost — an under-voltage that dragged the 5&nbsp;V rail below threshold, dropped the LiDAR mid-handshake, and left the trolley correctly, stubbornly, holding STOP. We ended that post promising numbers and a power redesign before we spent money. This is where the hunt actually ended, and it was not where the buck's little voltmeter was pointing.

### The Handheld Meter Lies (By Being Honest)
We went back in with a meter and measured everything in the chain. The LM2596 output read a healthy 5.48&nbsp;V. The wire-end, right where the loom meets the Pi, read 5.5&nbsp;V. By every reading a multimeter can give you, the rail was fine. And yet the Pi kept browning out.

The catch is that a handheld meter samples slowly and shows you an average. The brown-out that breaks a scan is a *millisecond* transient — a load step that comes and goes faster than the meter's display can ever update. So the meter was not lying, exactly. It was telling the truth about the wrong timescale. To see the real story we had to stop asking the wire and start asking the Pi.

### Asking the Pi Directly
The Pi 5 carries its own power-management IC, and you can read the voltage it *actually* receives with `vcgencmd pmic_read_adc`. The `EXT5V_V` figure is the ground truth — the 5&nbsp;V as it lands on the board, past every wire and every connector.

At idle it read 5.05&nbsp;V. Not the 5.5&nbsp;V we measured at the wire-end a few centimetres away — a fifth of a volt had already vanished. Then we put the CPU under `stress-ng`, and `EXT5V_V` sagged to **4.61&nbsp;V**. That is well under the Pi 5's happy band, and exactly the kind of dip that fires the under-voltage detector and knocks the LiDAR's USB adapter off the bus.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-lm2596-rail-voltage.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-lm2596-rail-voltage.jpeg' | relative_url }}" alt="LM2596 buck converter reading roughly 5.5 volts at its output" style="cursor: zoom-in;" />
  </a>
</div>

### The Loss Was in the Contacts, Not the Converter
5.5&nbsp;V at the wire-end, 4.61&nbsp;V at the chip under load. The whole drop was happening in the last inch — in the Dupont jumpers feeding the Pi's 5&nbsp;V GPIO pins. A single small header contact carrying a Pi 5's peak current is a real resistance, and under a fast load step it burns off as a voltage drop right where it hurts most. This is why we had been fighting a ghost: no meter reading on the *supply* side could ever show a fault that lived entirely in the *connector*.

The first thing that helped confirm it was splitting the feed. Instead of one 5&nbsp;V and one ground pin doing all the work, we spread the current across two 5&nbsp;V and two ground pins. That alone pulled the rail back above 4.85&nbsp;V under load — more parallel contacts, less drop across each. It was the proof we needed, but it was still asking a row of friction-fit jumpers to carry serious current, and that is not a foundation you build an autonomous platform on.

### The Fix: Give the Pi Its Own Battery
So we stopped trying to squeeze clean power through a GPIO header and gave the Pi a supply built for exactly this job. The Pi 5 now runs from a 10000&nbsp;mAh USB power bank over USB-C, with `usb_max_current_enable=1` set so the firmware negotiates the full current budget.

The difference was immediate and boring, which is exactly what you want from a power rail. `EXT5V_V` now sits at **4.90–4.99&nbsp;V, rock-steady**, under the worst case we could throw at it — CPU maxed, LiDAR spinning and scanning, UWB live, Arduino on the bus. `dmesg` came up clean, no under-voltage line in sight. The USB-C connector and the power bank's own protection are simply built to deliver this current in a way a Dupont pin never will.

There is a second, quieter win here. The power bank isolates the compute and sensor domain from the motor supply entirely. The LiPo and the L298Ns now do their own noisy, high-current work without the Pi's rail ever hearing about it — no shared path for a motor surge to sag the sensors. For a robot whose whole safety case rests on the LiDAR seeing clearly, decoupling the brain from the muscle is worth as much as the steady voltage itself.

### An Honest Footnote
We will be straight about it: a power bank is a *good* answer, not the *final* one. A proper onboard buck sized for the Pi's true peak — soldered, strain-relieved, fed straight from the pack — is still on the parts list and still the permanent plan. What the power bank buys us is a rail we can trust today, so the dissertation work can move forward on ranging and braking instead of stalling on volts.

And it closes the loop the last post opened. The fail-safe did its job when the rail collapsed; now the rail does not collapse. The trolley holds STOP when it is blind, and it is no longer blind for a reason we can measure and fix. With power finally out of the critical path, the next post gets to be about the sensors doing what they were mounted to do.
