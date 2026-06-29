---
layout: post
title: "What we already built (the simulation)"
date: 2026-06-27
author: Mohammed Shalaby
tags: [software, simulation]
---

<!-- DRAFT SCAFFOLD — write in your own words and replace the placeholders. This post recaps PRIOR coursework (CW2); state that clearly and link it — the dissertation extends it, it isn't re-submitted here. -->

## What it is

_A complete ROS 2 Jazzy simulation in Gazebo Harmonic, built and graded as PDE4445 Coursework 2. Note clearly that this is prior work we are building on, and link the repo and report._

> Prior work: this simulation was submitted as Coursework 2. This dissertation develops it into hardware.

## The architecture

_The four ROS 2 packages, one line each:_

- **behaviour** — follow-me, obstacle-stop, teleop
- **navigation** — Kalman filter, UWB simulator, state machine
- **description** — URDF / Xacro robot model
- **gazebo** — world, ROS–Gz bridge, RViz config

![Architecture diagram](/assets/images/architecture.png)

## What it proves

_UWB + Kalman following at a maintained 1.0 m; three-zone LiDAR safety (warn / slow / stop); a supermarket world with dynamic pedestrians. Embed the demo video and a couple of RViz/Gazebo screenshots._

![Gazebo world](/assets/images/gazebo.png)

_Demo video:_ <!-- embed your CW2 demo video link -->

## The gap it leaves

_Everything above was simulated — including UWB. Real hardware, real sensor noise, and a physical UWB rig are what the dissertation tackles next._
