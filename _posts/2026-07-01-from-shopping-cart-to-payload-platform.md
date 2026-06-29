---
layout: post
title: "From shopping cart to payload platform"
date: 2026-07-01
author: Ashwin Murali Thanalapati
tags: [scope, planning]
---

<!-- DRAFT SCAFFOLD — write in your own words and replace the placeholders. This is your "decisions & trade-offs" showcase; markers reward visible reasoning. -->

## Where we started

_Originally Trolley-X was a smart shopping trolley with a depth camera for item recognition and self-checkout._

## What the landscape looked like

_A short scan: commercial follow-carts (Foxtech FOLO — UWB/LiDAR following) versus retail smart carts (Amazon Dash Cart, Caper — vision checkout, but the shopper still pushes them). Table below._

| System | Approach | Autonomous follow? |
|---|---|---|
| Foxtech FOLO | UWB / LiDAR follow-cart | Yes |
| Amazon Dash Cart | Vision + weight, self-checkout | No (pushed) |
| Caper | Vision, self-checkout | No (pushed) |

## The realisation

_Vision-based self-checkout was the costly, integration-heavy part; autonomous following + safety is the buildable, already-validated core._

## What we changed

_Removed the Intel NUC and RealSense depth camera; re-scoped to a retrofittable multi-payload follow-me cart; kept UWB follow + three-zone LiDAR safety; return-to-home as a stretch goal._

![BOM before and after](/assets/images/bom-before-after.png)

## Why

_Self-funded (the grant didn't land), a tighter engineering focus, and a sharper dissertation contribution. The BOM dropped from AED 9,140 to about AED 6,440._
