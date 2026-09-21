---
layout: post
title: "Does the Filter Earn Its Place?"
date: 2026-09-21 06:00:00 +0400
categories: [Updates, Research]
---

My research question asks a specific thing: does UWB *plus a Kalman filter* follow better than UWB alone? I had assumed the answer was yes. But an assumption is not a result. So I measured it.

### The wrong way to compare

The obvious test is to do one follow with the filter on, one with it off, and compare. I did that. The filter-off run looked smoother. That result is worthless, and here is why: I walked a different path each time. The raw run happened to be a gentler walk, so of course it looked calmer. Two runs by a human in a small room are never the same walk. Comparing them measures my footwork, not the filter.

### The right way to compare

The fix is to compare on the *same* input. Every run logs the two raw anchor ranges. So I can take one run, and from those same logged ranges compute two things at every instant: the bearing the raw method would produce, and the bearing the Kalman filter produces. Same data, same moments, only the filter differs. Now the walk cancels out.

![Raw bearing against Kalman bearing on the same logged ranges]({{ "/assets/images/2026-09-21-raw-vs-kalman.png" | relative_url }})

The orange line is the raw two-anchor bearing. It swings across the full range and slams into the limits. It is unusable for steering; a controller fed this would spin constantly. The blue line is the same data through the Kalman filter. It is smooth, and it tracks where I actually was.

### The number

Across three runs, the filter cuts the bearing's step-to-step jitter by 74 to 83 percent, and it roughly halves the spread. That is consistent, and because it is a same-input comparison, it is honest.

So the answer to the research question is yes, with a sharp edge to it. The Kalman filter is not a cosmetic smoother laid over a signal that already works. The raw two-anchor bearing does not work for steering at this baseline. The filter is what makes it usable at all. That is a stronger claim than "the filter helps," and the data backs it.

### The honest caveat

This is one operator, a handful of runs, a 2 by 3.3 m space. It shows the filter's effect on the bearing clearly, but the wider trial set is still small. And the filter cannot invent information that is not there: the underlying limit is still the 0.24 m baseline, which the filter smooths but cannot widen. The filter makes the weak signal usable. A wider baseline would make it strong.
