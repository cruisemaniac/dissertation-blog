---
layout: post
title: "Waking the UWB: A Power-On Reset Puzzle"
date: 2026-08-28 21:00:00 +0400
categories: [Updates, Research]
---

The stopgap from the undervoltage fight held. We nudged the LM2596's output up so the Pi 5 sees a steady ~5&nbsp;V under load — it is not the final power design, and that conversation is still open, but it got the RPLiDAR spinning and scanning and let the UWB run straight off the Pi. And of course, the moment one gremlin dies, another steps into its place. This time the UWB came up completely mute on serial after every single boot — until we performed a very specific little ritual by hand.

### The Symptom: A Module That Boots Silent
Every power-up, the REYAX RYUW122 was powered but said nothing on its serial port. The only thing that revived it was a manual sequence on the reset line: take `NRST`, touch it to ground (header pin 9) to pull it low, then move it over to the 3.3&nbsp;V rail (pin 17) to bring it back high. The instant that low-then-high happened, the module started talking. Every boot, the same dance.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-pi5-integration.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-pi5-integration.jpeg' | relative_url }}" alt="Raspberry Pi 5 header where the UWB reset line is jumpered" style="cursor: zoom-in;" />
  </a>
</div>

### Why It Happens: Reset Timing, Not Reset State
This is a power-on-reset sequencing problem, not a wiring mistake. A reset line like `NRST` is not simply "high means run" — the module expects to see `NRST` *released* only after its supply has come up and settled, as a clean low-to-high edge at the right moment. When we tie `NRST` straight to 3.3&nbsp;V, it rises *with* the rail at power-on, before the chip's internals are ready, and there is never a proper release edge. The module latches into a state where its UART never starts. Grounding `NRST` and then lifting it back to 3.3&nbsp;V is us hand-crafting the exact edge it wanted all along — a manual power-on reset.

### The Fix: Automate the Edge
Doing this by hand on every boot is obviously a non-starter for a platform meant to be autonomous, so the reset has to become automatic. There are two clean ways to get there:

* A hardware power-on-reset circuit. A pull-up resistor (~10&nbsp;kΩ) from 3.3&nbsp;V to `NRST`, with a capacitor (~10&nbsp;µF) from `NRST` to ground. At power-up the capacitor holds `NRST` low while it charges, then the line rises ~100&nbsp;ms later — after the supply is stable — releasing the module cleanly. A small diode across the resistor lets the capacitor drain quickly at power-off so the reset re-arms for the next boot. Three components, no code.

* A GPIO-driven reset. Wire `NRST` to a spare GPIO and pulse it low-then-high from a startup service before the UWB node launches. It costs a couple of lines of code, but it is fully deterministic and lets us re-reset a wedged module from software instead of reaching for a jumper.

We are moving to an automated reset — most likely the RC circuit for its sheer simplicity, with the GPIO route held in reserve for when we want software control over the line. Either way, the goal is the same: the whole stack comes up hands-free from a single power switch.

With the rail steadied and the UWB finally answering on demand, the next job is the interesting one — actually pulling ranging data off the module and into ROS 2, the raw signal our Kalman filter will eventually smooth into a stable follow-me lock. More on that soon.
