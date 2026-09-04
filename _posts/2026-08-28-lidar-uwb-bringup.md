---
layout: post
title: "LiDAR and UWB Integration"
date: 2026-08-28 09:00:00 +0400
categories: [Updates, Research]
---

The chassis is wired and the dual-board architecture is defined. Today we integrated the two sensors that support this dissertation. The LiDAR detects obstacles for braking. The Ultra-Wideband sensor measures the user's position. We mounted and connected both sensors. We also established communication with both sensors.

### LiDAR Integration

The Slamtec RPLiDAR A1 is mounted on the top deck. It produces a two-dimensional map of the area around the cart. This sensor supports the second research question about velocity-dependent braking zones. Its `/scan` stream goes to the safety supervisor. The supervisor controls when the trolley can move.

We brought up the LiDAR on the Pi 5 with the `sllidar_ros2` driver. The unit enumerated correctly. It reported its serial number, firmware version 1.29, and an OK health status on the first test. The top deck contains the safety-critical components. These components are the LiDAR, the hardware emergency-stop button, and the short-range ultrasonic sensors.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-sensor-deck-profile.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-sensor-deck-profile.jpeg' | relative_url }}" alt="Trolley-X sensor deck: LiDAR dome, E-stop, and ultrasonics" style="cursor: zoom-in;" />
  </a>
</div>

### UWB Integration

The second sensor is an Ultra-Wideband device. The REYAX RYUW122 measures the time-of-flight distance between the cart and the user. The first research question uses this raw distance signal as input to a Kalman filter. The filter will reduce measurement jitter and stabilize follow-me behavior.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-ryuw122-uwb-module.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-ryuw122-uwb-module.jpeg' | relative_url }}" alt="REYAX RYUW122 Ultra-Wideband module" style="cursor: zoom-in;" />
  </a>
</div>

The module did not respond after the first connection. The supply voltage and wiring were correct. The serial port did not respond. The cause was the `NRST` reset line. This breakout board does not provide a reliable internal pull-up. A floating `NRST` line can keep the module in reset. The UART then does not start. We connected `NRST` to the Pi's 3.3&nbsp;V rail at physical pin 17. This connection holds the module out of reset. The RYUW122 responded after we made this connection.

### Connection to the Raspberry Pi

Both sensors connect to the Raspberry Pi 5, which runs ROS 2. The LiDAR uses USB. The UWB module uses serial communication. This arrangement keeps the data streams separate. It also allows the Pi to perform sensor fusion without using the microcontrollers.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/2026-08-28-pi5-integration.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-08-28-pi5-integration.jpeg' | relative_url }}" alt="Raspberry Pi 5 wired into the sensor loom" style="cursor: zoom-in;" />
  </a>
</div>

Both sensors responded during the first full bring-up on the Pi. We placed the chassis on blocks for this test. The test then revealed a power problem. The problem was an overloaded buck converter, not either sensor. The next post describes this problem.
