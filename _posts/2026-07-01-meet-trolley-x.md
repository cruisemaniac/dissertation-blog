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

### **The Core Research Question**
While many autonomous cart projects focus on pure mechanical construction, our dissertation aims to investigate the performance limitations of low-cost sensor fusion in highly dynamic, unstructured environments. Our research is guided by the following overarching question:

> *"How does the integration of predictive sensor fusion (Kalman-filtered UWB and velocity-dependent LiDAR) improve the navigational stability and collision-avoidance capabilities of an autonomous follow-cart in dynamic pedestrian environments?"*

### **Scientific Inquiries**
To answer this, our team is analyzing data collected from our prototype to investigate two specific technical problems:

**1. Estimation & Control**
* **Inquiry:** How does applying a Kalman filter to Ultra-Wideband (UWB) telemetry reduce positional jitter and stabilize motor velocity compared to raw UWB data during hands-free operation?

**2. Predictive Safety Logic**
* **Inquiry:** What is the impact of a predictive, velocity-dependent LiDAR braking zone model on reducing false-positive emergency stops compared to a static three-zone system in unstructured spaces?



### **Research Motivation**
Industrial-grade autonomous mobile robots (AMRs) rely on high-cost, proprietary sensor suites (e.g., 3D LiDAR, depth cameras). There is a significant gap in the literature regarding whether low-cost, retrofittable hardware can achieve comparable reliability. Our work is not simply about building a cart; it is an investigation into the **limits of performance for budget-constrained robotics**.

### **Our Scientific Approach**
To answer our research questions, we are adopting an evaluation-focused methodology:
* **Quantitative Performance Logging:** Comparing raw sensor inputs vs. filtered state outputs to measure stability (e.g., using Root Mean Square Error metrics).
* **Standardized Stress Testing:** Evaluating LiDAR safety braking performance against diverse, real-world obstacles to assess false-positive rates.
* **Critical Discussion:** Comparing our results against established literature on UWB jitter and reactive collision avoidance to draw meaningful conclusions.



***

*Stay tuned for our upcoming Literature Review and Methodology updates as we begin our data collection phase!*
