---
layout: post
title: "Chasing a Ghost: Making the UWB Follow Robust"
date: 2026-09-20 04:30:00 +0400
categories: [Updates, Research]
---

The cart follows and it turns. So the next job is trust. A follow that works in a clean test but drives into a wall in a messy one is not done. This post is about faults I found this week and the small, cheap fixes for each. None of them touched the Kalman filter maths. All of them were about which measurements the filter is allowed to believe.

### The tag is not always in a good place

The follow reads two UWB anchors on the cart. Each anchor gives a range to the tag I carry. The bearing comes from the difference between the two ranges. The two anchors sit only 0.24&nbsp;m apart, so that difference is small, and anything that corrupts one range corrupts the bearing.

I found three ways to corrupt it.

The first was a black box. I had left the emergency-stop box on the deck, between the two anchors. It blocked the line of sight from one anchor to the tag. The blocked anchor read long, the bearing flipped, and the cart pulled the wrong way. I moved the box off the anchor line and the fault went away. A radio link needs a clear path. An anchor is not a place to store hardware.

The second was distance. My trial area is 3.3&nbsp;m. The room is larger. When the tag sat near the far wall, at about 4.3&nbsp;m, the range was still valid, but the bearing was not. A 0.24&nbsp;m baseline cannot resolve an angle at that range; the difference between the two ranges is smaller than the noise. The cart read that noise as a hard turn and spun. The fix is a cap. The localizer now ignores any range beyond 3.5&nbsp;m, just past my trial area. Far, weak bearings can no longer steer the cart.

The third was the floor. I set the controller down on the ground at the end of a run and the cart pivoted. That looked like the cart reacting to the tag's height. It was not. The controller has no height term. What happened is that a tag on the floor gives bad ranges, because the signal grazes the ground and bounces. One anchor read 0.23&nbsp;m while the other read 0.54&nbsp;m. For a 0.24&nbsp;m baseline, that is impossible: two ranges to one point cannot differ by more than the distance between the anchors. The bad pair made a false bearing, and the cart turned on it.

### A gate you already have, set too loose

The impossible-pair case had a simple answer, and it was already half-built. The localizer already had a consistency check: reject a range pair when the two ranges differ by more than the baseline plus a margin. The margin was 0.15&nbsp;m, so the threshold was 0.39&nbsp;m. The 0.31&nbsp;m floor glitch slipped under it.

I looked at the data before I picked a new number. During a normal follow, with the tag in front, the two ranges differ by a median of 0.04&nbsp;m and stay under 0.12&nbsp;m. The physical maximum for this baseline is 0.24&nbsp;m. The glitches all sat above 0.29&nbsp;m, with a clear gap below them. So I dropped the margin to 0.05&nbsp;m. The threshold is now 0.29&nbsp;m. It rejects the impossible pairs and keeps every real reading. This is not new code. It is one number, chosen from the data instead of guessed.

### The worst fault: chasing a ghost

The last fault was the dangerous one. In one run I walked out past the trial area. The cart could no longer get a valid range, so the filter had nothing to correct itself with. A Kalman filter that gets no measurement does not stop. It coasts. It keeps predicting from its last velocity. The estimated range climbed on its own, at a steady half a metre per second, all the way to 6.5&nbsp;m, while I stood still.

The controller believed I was far ahead and moving away. So it drove forward to catch up. There was a stationary object between the cart and me. The cart drove into it.

The LiDAR was not at fault. The chart below shows it. The top panel is the estimated range running away, past the 3.5&nbsp;m cap, long after I had stopped. The bottom panel is the real obstacle, seen clearly by the LiDAR, closing to the 0.30&nbsp;m stop zone. The grey bands are where the cart was driving forward. The safety layer did slow the cart and then stop it, but by then the front had already reached the object. The cart was driving into a wall it could see, because it was chasing a tag that was not there.

![The estimated range coasts away while the LiDAR clearly sees the obstacle the cart drives into]({{ "/assets/images/2026-09-20-phantom-chase.png" | relative_url }})

The fix follows the same rule as the others: do not act on data you do not have. The localizer now watches how long it has been since it last accepted a range. If that time passes half a second, it drops the estimate instead of coasting. With no estimate, the controller holds. When I come back into range, a clean pair of ranges seeds the filter again and the follow resumes. A follow that cannot see its target should wait, not guess and charge.

### What I learned

Every one of these faults was a bad measurement, not bad maths. The filter was sound. The controller was sound. The problem each time was that the system trusted a number it should have thrown away, or kept trusting an old number after the new ones stopped coming. The fixes are all gates: reject the impossible pair, ignore the too-far range, and stop when the ranges stop. Cheap to write, and each one removes a way for the cart to hurt itself or someone near it.

There is still work on the safety margin. The stop zone fires at 0.30&nbsp;m from the LiDAR, and the front of the cart sits ahead of the LiDAR, so the stop comes too late for a fast approach. That is the next fix. But the cart no longer chases ghosts, and that was the one that scared me.
