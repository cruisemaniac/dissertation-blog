---
layout: post
title: "The First Fix Is a Liar: A Settle Window for the Start"
date: 2026-09-21 05:00:00 +0400
categories: [Updates, Research]
---

After the robustness gates, one fault remained, and it showed at the worst moment: the start. The cart would spin the instant it began to move. This post is the fix. It is small.

### What I saw

Standing start, tag held straight ahead. The cart would pivot hard to one side before it drove a single centimetre. Then it would correct itself and follow normally. So it was not lost, and it was not the drivetrain. It was acting on a bad first reading.

### The cause

The follow gets my angle from the difference between the two anchor ranges. The anchors are only 0.24&nbsp;m apart, so that difference is small, and one noisy range throws the angle by tens of degrees. At the very start, the filter has just one pair of ranges to work from. If that pair is noisy, the first angle is wrong.

I could see it in the log. For a full second before the filter started, both anchors read about 1.5&nbsp;m, equal, which means straight ahead. Then, at the exact instant the filter seeded, the right anchor glitched low, to 1.47&nbsp;m against 1.66&nbsp;m on the left. The filter placed me 46 degrees off to the side. The controller believed it and spun to face a tag that was already in front of it. Half a second later the filter settled to near zero, but the cart had already turned.

### The fix

The rule is simple: do not turn in place on the first fix. For 0.7&nbsp;s after the cart acquires the tag, it will not pivot. If the first angle is off, the cart eases forward with a gentle steer instead of spinning. Two things then happen. The filter takes in more ranges and settles. And my first steps give it real motion, which sharpens the angle. After 0.7&nbsp;s the cart pivots as normal.

I tested it on the same standing start. The first fix came in 16 degrees off. The old code would have spun. The new code eased forward at a soft steer and settled to near straight within a second. No spin, then a clean 62-second follow.

### The theme

This is the fourth gate in a week, and they all say the same thing. The follow is only as good as the measurements it trusts. Reject the impossible pair. Ignore the too-far range. Stop when the ranges stop. And do not turn on the first, unsettled fix. None of these touched the Kalman filter. The filter was always right. The work was teaching the system which numbers to ignore.

The real limit behind all of it is the 0.24&nbsp;m baseline. It makes the angle hard to see, so the follow lives close to the noise. A wider baseline, more anchors, or a module that reports the angle directly would fix the cause instead of the symptoms. That is the next step.
