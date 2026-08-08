---
layout: post
title: "Trolley-X Sprint - Locking the Architecture and First Power On Check"
date: 2026-08-04 09:00:00 +0000
categories: [Updates, Research]
---

Building reliable autonomous systems—specifically physical robots and vehicles—requires strict attention to hardware bottlenecks long before you start tuning your ROS 2 navigation stack. Today’s sprint was all about stripping out the noise, mapping the power distribution, and locking in the exact electronic architecture for Trolley-X.

### The Bottleneck: 8-Bit Limitations vs. Serial Demands
Initially, the plan was to run the drive logic and sensor tracking through the Arduino Uno R3. The math quickly showed this would cause a catastrophic failure.

The incoming REYAX UWB modules require hardware UART (Serial) to communicate fast enough for real-time tracking, but the Uno only has one hardware serial port—which must be dedicated to talking to the Raspberry Pi 5. Forcing the Uno to use software serial for UWB would cause it to miss the high-speed hardware interrupt ticks from the motor encoders, completely ruining the cart's odometry.

### The Solution: The Dual-Board Architecture
To keep the processing fast and the data streams clean, we divided the labor into a strict dual-board architecture feeding into the Pi 5.

* The Drivetrain Boss (Arduino Uno R3): The Uno is strictly dedicated to the physical movement of the rover. It outputs clean 5V PWM signals to the L298N motor drivers and uses its two hardware interrupt pins (Pins 2 and 3) to track the front-left and front-right motor encoders.

* The Sensor Middleman (ESP-WROOM-32): We brought in the ESP32 strictly to handle UWB tracking. Because it features three hardware serial ports, it can rapidly parse the UWB distance data via UART without breaking a sweat.

* The Brain (Raspberry Pi 5): The Pi natively runs ROS 2. It reads the separate, clean data streams from both the Uno and the ESP32 via standard USB cables, eliminating the need for complex low-level serial math.

### Today’s Milestone Checkpoints

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <img src="{{ '/assets/images/IMG_0526.jpeg' | relative_url }}" alt="Current state of Wiring" />
</div>

With the architecture locked, we made serious headway on the physical chassis today:

* Task 1 (Motors Verified): We applied raw power and verified that the DC gearboxes and L298N drivers are alive and pushing current.

* Task 2 (Skid-Steer Mapped): The left motors are wired in parallel to the top L298N, and the right motors are wired in parallel to the bottom L298N.

* Encoder Optimization: To keep the Uno's processor stable, we are only reading the front two motor encoders. Because the rear motors are physically locked in parallel on the same driver channels, the Pi 5 can safely assume the rear wheels are matching the front speeds.

* IMU Re-routed: The MPU6050 accelerometer is bypassing the microcontrollers entirely. It will wire directly into the 3.3V I2C pins on the Pi 5 to ensure safe logic levels and native ROS 2 integration.

With the high-voltage side mapped and the architecture separated, the immediate next step is wiring the low-voltage logic lines between the L298N boards and the Arduino Uno. Once the common grounds are tied, Trolley-X is ready to be propped up on blocks for its very first test drive.