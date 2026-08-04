---
layout: post
title: "August 3 Update: Trolley-X Fabrication Complete"
date: 2026-08-03 09:00:00 +0000
categories: [Updates, Research]
---

After wrapping up the initial fabrication and basic mechanical assembly yesterday, Trolley-X finally has a physical footprint. The transition from CAD screens and whiteboards to a tangible, rolling chassis is always a massive milestone in any robotics build.

Here is a look at the hardware baseline as it sits on the workbench right now.

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <img src="/assets/images/IMG_0520.jpeg" alt="Trolley-X chassis on the workbench" />
  <img src="/assets/images/IMG_0521.jpeg" alt="Trolley-X chassis from another angle" />
  <img src="/assets/images/IMG_0522.jpeg" alt="Trolley-X with motors and drivetrain components" />
  <img src="/assets/images/IMG_0524.jpeg" alt="Close-up of the assembled hardware" />
  <img src="/assets/images/IMG_0527.JPG" alt="Additional view of the fabricated robot" />
</div>

### The Mechanical Foundation
The core of Trolley-X is built for a 4-wheel-drive skid-steer (differential drive) setup. The heavy lifting is handled by:

* 4x DC Gear Motors: Each motor is equipped with a built-in rotary encoder.

* Dual L298N Motor Drivers: We mounted two separate red L298N H-bridge modules to the chassis. Because each driver channel has a strict 2-Amp limit, splitting the load across two boards ensures we don't fry any silicon when the cart pulls a heavy load.

* The Power Source: An 11.1V 3S LiPo battery, stepped down where necessary via an LM2596 buck converter.

### The Brains
With the mechanical assembly locked, the rover is waiting for its nervous system to be wired up. Sitting on the desk, ready for integration, are the three main logic units:

* Raspberry Pi 5: The main computer. This will run the ROS 2 navigation stack, the diff_drive_controller, and the sensor fusion nodes.

* Arduino Uno R3: A classic 8-bit, 5V microcontroller.

* ESP-WROOM-32: A modern 32-bit, dual-core microcontroller with onboard Wi-Fi and Bluetooth.

### The Sensor Suite
To achieve the "follow-me" tracking and local navigation, the hardware manifest includes:

* Slamtec RPLiDAR A1: For 2D obstacle avoidance.

* MPU6050 IMU: A 6-axis accelerometer and gyroscope to keep the robot's heading accurate.

* REYAX RYUW122_Lite (Incoming): Ultra-Wideband (UWB) modules that will handle the highly precise Time-of-Flight distance tracking between the cart and the user.

With the chassis fabricated and the components gathered, the physical build is complete. Now begins the hardest part: wiring the electronics without creating a data bottleneck.