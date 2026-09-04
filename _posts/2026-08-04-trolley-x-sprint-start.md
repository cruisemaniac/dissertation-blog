---
layout: post
title: "Trolley-X Sprint - Locking the Architecture and First Power On Check"
date: 2026-08-04 09:00:00 +0000
categories: [Updates, Research]
---

Reliable autonomous systems require early identification of hardware bottlenecks. This is especially important for physical robots and vehicles. Today we mapped the power distribution and defined the electronic architecture for Trolley-X before tuning the ROS 2 navigation stack.

### Hardware Serial Limitation
The initial plan assigned drive control and sensor tracking to the Arduino Uno R3. The interface requirements showed that this plan would fail.

The REYAX UWB modules require hardware UART for real-time tracking. The Uno has one hardware serial port. That port must communicate with the Raspberry Pi 5. Software serial for the UWB modules could cause the Uno to miss the motor encoder interrupts. The missed interrupts would corrupt the cart odometry.

### Dual-Board Architecture
We assigned the tasks to two boards. Both boards send data to the Raspberry Pi 5.

* **Drivetrain controller (Arduino Uno R3):** The Uno controls rover movement. It sends 5 V PWM signals to the L298N motor drivers. It uses hardware interrupt pins 2 and 3 to track the front-left and front-right motor encoders.

* **Sensor controller (ESP-WROOM-32):** The ESP32 handles UWB tracking. It has three hardware serial ports. It can parse UWB distance data through UART without interrupting the motor encoder measurements.

* **Main computer (Raspberry Pi 5):** The Pi runs ROS 2. It reads separate data streams from the Uno and the ESP32 through USB cables. This design avoids complex low-level serial processing.

### Milestones

<div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin: 1.5rem 0;">
  <a href="{{ '/assets/images/IMG_0526.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/IMG_0526.jpeg' | relative_url }}" alt="Current state of Wiring" style="cursor: zoom-in;" />
  </a>
</div>

After we defined the architecture, we completed the following checks:

* **Motors verified:** We applied power. The DC gearboxes and L298N drivers operated correctly.

* **Skid-steer mapped:** The left motors are wired in parallel to the top L298N. The right motors are wired in parallel to the bottom L298N.

* **Encoder configuration:** The Uno reads the front two motor encoders. The rear motors are wired in parallel with the front motors on the same driver channels. The Pi 5 can therefore use the front encoder speeds for the rear wheels.

* **IMU rerouted:** The MPU6050 connects directly to the 3.3 V I2C pins on the Pi 5. This connection provides the required logic level and supports native ROS 2 integration.

The high-voltage connections are mapped. The board functions are separated. The next step is to wire the low-voltage logic lines between the L298N boards and the Arduino Uno. After we connect the common grounds, we will place Trolley-X on blocks for its first drive test.