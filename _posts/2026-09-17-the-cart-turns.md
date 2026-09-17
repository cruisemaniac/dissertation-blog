---
layout: post
title: "The Cart Turns: MOSFET Drivers and a Roll of Tape"
date: 2026-09-17 01:30:00 +0400
categories: [Updates, Research]
---

The follow logic worked. The cart held its distance and tracked the operator. But it would not turn. On the floor it would drive forward and then just sit and hum when it had to rotate. This post fixes that. It took a driver swap and, in the end, a roll of masking tape.

### The Wall

First I measured the drivetrain, because a guess is not data. The battery held 10.8&nbsp;V at both motor drivers under load. The command was correct. The safety layer was not blocking it. The PWM was already at its ceiling. So none of those was the fault.

Then I measured the voltage at the motor terminals during a turn command. It read about 2.2&nbsp;V. The supply was 10.8&nbsp;V, but the motor saw 2.2&nbsp;V. The old L298N drivers were dropping the rest. The L298N is a bipolar H-bridge. It drops 2 to 3&nbsp;V or more under load, and the drop grows with current — exactly the condition of a stalled turn. A 6&nbsp;V motor fed 2&nbsp;V has no torque to break the sideways scrub of a four-wheel skid-steer. So the motor drew current and made noise, but the cart did not move.

The fault was the driver, not the code, the battery, or the wiring.

### New Drivers

My supervisor had pointed me at the BTS7960 earlier, so I had four on hand. The BTS7960 is a MOSFET H-bridge. It drops about 0.1&nbsp;V instead of 2 to 3&nbsp;V. On the same 10.8&nbsp;V rail, the motor now sees nearly the full voltage.

I removed both L298Ns and wired in two BTS7960 modules, one per side. Each side's two motors run in parallel on one module. Each module needs two PWM lines, `RPWM` and `LPWM`, one per direction, plus an enable.

![The BTS7960 motor driver wired to the Pi and the Arduino]({{ "/assets/images/2026-09-17-bts7960-driver.jpeg" | relative_url }})

The firmware changed with the hardware. The drive logic now writes `RPWM` or `LPWM`, never both. I hold the enable pin high to arm the bridges. Most important, I dropped the PWM ceiling to 120. With the old L298N the big voltage drop protected the 6&nbsp;V motors by accident. With the near-zero-drop MOSFETs, the motor sees almost the full rail, so a lower ceiling now keeps it near 6&nbsp;V.

The cart started to rotate. That alone was the wall coming down.

### The Roll of Tape

One problem remained. During a spin, one wheel would stall while the others turned. The front is heavy — the LiDAR, the UWB pillars, the mast. As the cart tried to rotate it rocked, the weight shifted between corners, and whichever wheel was loaded scrubbed too hard to turn. Front-left and front-right traded off.

The fix was not more electronics. It was grip. A skid-steer turns by scrubbing its tyres sideways, and my tyres gripped the tile too well. So I wrapped all four tyres in a band of masking tape to lower the friction. Less grip, less scrub, less load fighting the turn.

![A wheel wrapped in masking tape to cut the scrub]({{ "/assets/images/2026-09-17-taped-tyre.jpeg" | relative_url }})

That was it. All four wheels now pull together and the cart spins in place, cleanly.

### What I Learned

Three things stood out. First, a driver's voltage drop can be the hidden wall — the L298N looked fine on paper and failed under load, and only a measurement at the motor showed it. Second, a skid-steer fights its own tyres to turn; the grip that helps it drive straight is the grip that stops it rotating. Third, not every robotics problem needs a clever fix. Sometimes the answer is a roll of masking tape.

Next: a full follow run on the floor, now that the cart can turn, to see how well it tracks me around a corner.
