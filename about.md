---
layout: page
title: About
permalink: /about/
---

## The project

Trolley-X is an autonomous, retrofittable cart that follows its operator
hands-free, carries varied payloads, and navigates safely around people. It
is built for the PDE4439 Robotics Dissertation Project at Middlesex University
Dubai, and extends a validated ROS 2 Jazzy simulation into a physical
prototype.

The dissertation's research question is:

> How does the integration of predictive sensor fusion (Kalman-filtered UWB
> and velocity-dependent LiDAR) improve the navigational stability and
> collision-avoidance capabilities of an autonomous follow-cart in dynamic
> pedestrian environments?

Industrial autonomous mobile robots lean on high-cost, proprietary sensor
suites — 3D LiDAR, depth cameras. This project asks how far low-cost,
retrofittable hardware can get instead, and where it breaks down. That means
the blog covers failures as much as progress: undervoltage faults, a motor
driver that couldn't supply torque for a turn, a filter seeding on a bad
reading and spinning the cart — each logged and fixed in public, because the
fixes are as much the finding as the working demo.

## How it works

One ROS 2 launch file starts six nodes:

- **`sllidar_ros2`** — a Slamtec RPLiDAR A1, publishing `/scan`.
- **`safety_braking`** — a sectorized LiDAR safety supervisor that gates
  motion.
- **`arduino_base_controller`** — the drivetrain bridge, also publishing
  wheel odometry and IMU data.
- **`uwb_ranging`** — two REYAX RYUW122 Ultra-Wideband anchors on the cart,
  each ranging to a tag carried by the operator.
- **`uwb_localizer`** — an Extended Kalman Filter fusing the two UWB ranges
  with odometry into a single position estimate.
- **`follow_controller`** — turns that fused estimate into a motion command.

## The team

- **Ashwin Murali Thanalapati** — sensor fusion & control (UWB-Kalman follow,
  three-zone LiDAR safety, evaluation)
- **Mohammed Shalaby** — ROS 2 architecture & systems integration
- **Vignesh Lakshmanasamy** — mechanical & power (chassis, drivetrain, power
  system)

_Supervisor: Dr. Judhi Prasetyo · Module: PDE4439, Middlesex University
Dubai._
