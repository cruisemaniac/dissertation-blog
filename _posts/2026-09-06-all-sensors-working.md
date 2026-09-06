---
layout: post
title: "All Sensors Working!"
date: 2026-09-06 20:00:00 +0400
categories: [Updates, Research]
---

The earlier posts brought the LiDAR and the UWB online one at a time. They also fixed the power faults that stopped them. This post runs the whole sensor stack together for the first time. One launch starts every node. The system then produces a single fused estimate of the operator's position.

### The Stack

One launch file starts six nodes:

- `sllidar_ros2` — the RPLIDAR A1. It publishes `/scan`.
- `safety_braking` — the sectorized LiDAR safety supervisor.
- `arduino_base_controller` — the drivetrain bridge. It also publishes the wheel odometry and the IMU.
- `uwb_ranging` — the two cart anchors. Each reports its range to the operator's tag.
- `uwb_localizer` — an Extended Kalman Filter. It fuses the ranges with the odometry.
- `follow_controller` — the follow logic. It turns the fused estimate into a motion request.

The data topics are `/scan`, `/odom`, `/imu/data`, `/uwb/left`, `/uwb/right`, `/follow/target`, and `/safety/state`.

### Bring-up

I kept the motor battery unplugged. This test checks the sensors only. The cart must not move.

I built the workspace and started the stack:

```
colcon build --packages-select trolley_core
source install/setup.bash
ros2 launch trolley_core follow.launch.py
```

Every node started. `ros2 topic list` showed all seven data topics.

![The full topic list after one launch]({{ "/assets/images/Screenshot 2026-09-07 at 00.57.20.png" | relative_url }})

### The Sensors Report

I checked each sensor in turn.

The LiDAR published at a steady rate:

```
ros2 topic hz /scan
average rate: 7.187
```

![The LiDAR scan rate]({{ "/assets/images/Screenshot 2026-09-07 at 00.57.46.png" | relative_url }})

The safety supervisor reported the four sectors:

```
data: STOP F0.32 R0.31 B0.76 L0.82
```

The values are the nearest obstacle in each sector, in metres. The front and the right were inside the 0.5&nbsp;m stop zone. The state was therefore STOP. This is correct. The cart sat on a bench with objects close to it. In open space the state clears.

The left anchor reported its range to the tag:

```
range: 0.40
```

The filter reported the tag position in the cart frame:

```
point:
  x: 0.77
  y: 0.25
```

![The sensor readings: safety state, UWB range, and the fused target]({{ "/assets/images/Screenshot 2026-09-07 at 00.59.20.png" | relative_url }})

![The sensor readings: safety state, UWB range, and the fused target]({{ "/assets/images/Screenshot 2026-09-07 at 00.59.30.png" | relative_url }})

### The Fused Target

The two anchors give two ranges. Two ranges alone are noisy, and the short anchor baseline weakens the bearing. The wheel odometry gives the cart's own motion between range updates. The Extended Kalman Filter fuses the two. It holds a constant-velocity model of the operator. It rejects range outliers with an innovation gate. It then reports one position.

The estimate was `x = 0.77&nbsp;m` and `y = 0.25&nbsp;m`. This places the operator 0.77&nbsp;m ahead of the cart and 0.25&nbsp;m to the left. The range is 0.81&nbsp;m. The bearing is 18&nbsp;degrees to the left.

This single point is what the follow controller needs. It steers on the bearing. It sets the speed from the range and a stand-off gap.

### Result

The full perception stack ran at once and agreed. The LiDAR, the safety zones, the wheel odometry, the IMU, the two UWB anchors, and the Kalman filter all produced live data from one launch. The operator's position tracked as the tag moved.

The drivetrain is the next step. The motor battery, the E-stop, and the follow controller close the loop. The cart then moves toward the fused target. The next post records that test.
