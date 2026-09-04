---
layout: post
title: "August 3 Update: Trolley-X Fabrication Complete"
date: 2026-08-03 09:00:00 +0000
categories: [Updates, Research]
---

We completed the initial fabrication and mechanical assembly yesterday. Trolley-X now has a physical chassis. The chassis has moved from the CAD model and the design notes to a rolling platform.

The images show the current hardware baseline on the workbench.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/IMG_0520.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/IMG_0520.jpeg' | relative_url }}" alt="Trolley-X chassis on the workbench" style="cursor: zoom-in;" />
  </a>
  <a href="{{ '/assets/images/IMG_0521.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/IMG_0521.jpeg' | relative_url }}" alt="Trolley-X chassis from another angle" style="cursor: zoom-in;" />
  </a>
  <a href="{{ '/assets/images/IMG_0522.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/IMG_0522.jpeg' | relative_url }}" alt="Trolley-X with motors and drivetrain components" style="cursor: zoom-in;" />
  </a>
  <a href="{{ '/assets/images/IMG_0524.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/IMG_0524.jpeg' | relative_url }}" alt="Close-up of the assembled hardware" style="cursor: zoom-in;" />
  </a>
  <a href="{{ '/assets/images/IMG_0527.JPG' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/IMG_0527.JPG' | relative_url }}" alt="Additional view of the fabricated robot" style="cursor: zoom-in;" />
  </a>
</div>

### Mechanical System
Trolley-X uses a four-wheel-drive skid-steer differential-drive layout. The main components are:

* **Four DC gear motors:** Each motor has a built-in rotary encoder.

* **Two L298N motor drivers:** We mounted two L298N H-bridge modules on the chassis. Each driver channel has a 2 A limit. The two boards distribute the motor load and prevent an overload on one board.

* **Power source:** An 11.1 V 3S LiPo battery supplies the system. An LM2596 buck converter reduces the voltage where required.

### Control Hardware
The mechanical assembly is complete. The following control units are ready for integration:

* **Raspberry Pi 5:** This is the main computer. It will run the ROS 2 navigation stack, `diff_drive_controller`, and the sensor-fusion nodes.

* **Arduino Uno R3:** This is an 8-bit, 5 V microcontroller.

* **ESP-WROOM-32:** This is a 32-bit, dual-core microcontroller with Wi-Fi and Bluetooth.

### Sensors
The sensor system supports follow-me tracking and local navigation. It includes:

* **Slamtec RPLiDAR A1:** This sensor supports 2D obstacle avoidance.

* **MPU6050 IMU:** This sensor contains a six-axis accelerometer and gyroscope. It measures the robot heading.

* **REYAX RYUW122_Lite (incoming):** These Ultra-Wideband (UWB) modules will measure the time-of-flight distance between the cart and the user.

The physical build is complete. The next task is to wire the electronics. The wiring must support the required data rates without creating a bottleneck.