---
layout: post
title: "Eyes and Ears: Bringing the LiDAR and UWB Online"
date: 2026-08-28 09:00:00 +0400
categories: [Updates, Research]
---

With the chassis wired and the dual-board architecture locked, today was about perception — giving Trolley-X the two senses the entire dissertation rests on. One sensor to *see* the world and brake for it, and one to *know where its human is*. Both got mounted, wired, and talking today. Neither did it without a fight.

### The LiDAR: A Spinning Wall of Distance
The Slamtec RPLiDAR A1 now sits on the top deck, spinning up a 2D picture of everything around the cart. This is the sensor behind our second research question — the velocity-dependent braking zones — so it feeds its `/scan` stream straight into the safety supervisor that decides when the trolley is allowed to move.

Bring-up on the Pi 5 went through the `sllidar_ros2` driver. The unit enumerated cleanly, reported its serial number, firmware 1.29, and a health status of OK on the first try. The top deck is now genuinely the safety-critical layer of the robot: the LiDAR dome, a hardware emergency-stop mushroom, and the short-range ultrasonic sensors all live up there together.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-sensor-deck-profile.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-sensor-deck-profile.jpeg' | relative_url }}" alt="Trolley-X sensor deck: LiDAR dome, E-stop, and ultrasonics" style="cursor: zoom-in;" />
  </a>
</div>

### The UWB: Ranging by Time-of-Flight
The other new sense is Ultra-Wideband. The REYAX RYUW122 handles the precise time-of-flight distance measurement between the cart and the user — the raw signal that our first research question then runs through a Kalman filter to smooth out the jitter and stabilise the follow-me behaviour.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-ryuw122-uwb-module.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-ryuw122-uwb-module.jpeg' | relative_url }}" alt="REYAX RYUW122 Ultra-Wideband module" style="cursor: zoom-in;" />
  </a>
</div>

And here is the hour we did not budget for. Straight out of the box, the module was completely silent — power was fine, wiring looked right, but the serial port simply would not respond. The culprit was the `NRST` reset line. On this breakout it has no dependable internal pull-up, so left floating it sits in (or right on the edge of) reset, and the UART never wakes up. The fix was a single jumper: tie `NRST` to the Pi's 3.3&nbsp;V rail (physical pin 17) to actively hold the module *out* of reset. The instant that wire went in, the RYUW122 started answering. One wire, one hour, one lesson filed away.

### Wiring Into the Brain
Both sensors now route into the Raspberry Pi 5 running ROS 2 — the LiDAR over USB, the UWB over serial. That keeps the data streams clean and lets the Pi do the fusion natively without dragging the microcontrollers into it.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-pi5-integration.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-pi5-integration.jpeg' | relative_url }}" alt="Raspberry Pi 5 wired into the sensor loom" style="cursor: zoom-in;" />
  </a>
</div>

With both sensors finally answering, we propped the stack up on blocks for its first full bring-up on the Pi — and immediately walked into a power gremlin that had nothing to do with either sensor, and everything to do with a single overloaded buck converter. That is the next post.
