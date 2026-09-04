---
layout: post
title: "Project Trolley-X: Investigating Autonomous Indoor Navigation"
date: 2026-07-01 10:00:00 +0400
categories: [Planning, Introduction]
pinned: true
---

**Authors:** Ashwin Murali Thanalapati, Mohammed Shalaby, Vignesh Lakshmanaswamy  
**Module:** PDE4445 Robotics Dissertation Project (2025-2026)  
**Project Title:** Scientific Evaluation of UWB-LiDAR Sensor Fusion for Robust Indoor Human-Robot Collaboration

### **Research Question**
Many autonomous cart projects focus on mechanical construction. This dissertation investigates the performance limits of low-cost sensor fusion in dynamic, unstructured environments. The main research question is:

> *"How does the integration of predictive sensor fusion (Kalman-filtered UWB and velocity-dependent LiDAR) improve the navigational stability and collision-avoidance capabilities of an autonomous follow-cart in dynamic pedestrian environments?"*

### **Technical Questions**
The team will analyze data from the prototype. The analysis will address two technical questions:

**1. Estimation and control**
* **Question:** How does a Kalman filter applied to Ultra-Wideband (UWB) telemetry reduce position jitter and stabilize motor velocity when compared with raw UWB data during hands-free operation?

**2. Predictive safety logic**
* **Question:** Does a predictive, velocity-dependent LiDAR braking-zone model reduce false emergency stops when compared with a static three-zone system in unstructured spaces?



### **Research Motivation**
Industrial autonomous mobile robots (AMRs) use high-cost, proprietary sensor systems, such as 3D LiDAR and depth cameras. The literature contains limited evidence about the reliability of low-cost, retrofittable hardware. This project investigates the **performance limits of budget-constrained robotics**.

### **Method**
The project uses an evaluation-focused method:
* **Quantitative performance logging:** Compare raw sensor inputs with filtered state outputs. Use Root Mean Square Error metrics to measure stability.
* **Standardized stress testing:** Test LiDAR braking performance with different real-world obstacles. Measure the rate of false emergency stops.
* **Critical discussion:** Compare the results with published research on UWB jitter and reactive collision avoidance.



***

*The next updates will describe the literature review, the method, and the data collection phase.*
