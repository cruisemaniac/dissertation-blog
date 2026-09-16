---
layout: post
title: "How Accurate Is the Follow? Measuring the UWB Range"
date: 2026-09-16 20:30:00 +0400
categories: [Updates, Research]
---

The last posts made the cart follow. This post asks a harder question. How well does the cart know where I am? The whole follow depends on one number: the UWB range from each cart anchor to the tag I carry. So I measured that number against the truth.

### The Setup

The test is static. The cart stands still. I stand the tag on the centre line, straight ahead of the cart. I measure the true distance with a tape. A small ROS&nbsp;2 node, `static_capture`, then records about 30&nbsp;range samples over 6&nbsp;seconds. It writes the mean, the error, and the standard deviation to a CSV file. I repeat this at set marks from 0.75&nbsp;m to 3.0&nbsp;m.

![The test layout: the cart on the left, the tag on the breadboard on the right, on a tiled floor for a straight line]({{ "/assets/images/2026-09-16-uwb-accuracy-setup.jpeg" | relative_url }})

The tag is a REYAX RYUW122 module on an ESP32. The tape runs from the cart to the tag along the floor tiles, so the line stays straight.

<figure>
  <a href="{{ '/assets/images/2026-09-16-uwb-tag-250cm.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-09-16-uwb-tag-250cm.jpeg' | relative_url }}" alt="Close view of the UWB tag on the breadboard with the tape measure" style="cursor: zoom-in;" />
  </a>
</figure>

<figure>
  <a href="{{ '/assets/images/2026-09-16-uwb-module-tape.jpeg' | relative_url }}" target="_blank" rel="noopener">
    <img src="{{ '/assets/images/2026-09-16-uwb-module-tape.jpeg' | relative_url }}" alt="The tape measure at the tag station, near the 300 cm mark" style="cursor: zoom-in;" />
  </a>
</figure>

### The Numbers

Both anchors are accurate, and both are linear. A straight-line fit gives an R-squared above 0.997 for each anchor. The chart shows the sweep. The left panel plots the measured range against the true distance. The right panel plots the error against the module's stated &plusmn;10&nbsp;cm limit.

![UWB static ranging accuracy from 0.75 m to 3.0 m]({{ "/assets/images/2026-09-16-uwb-accuracy-chart.png" | relative_url }})

The results, over the sweep:

- The **left anchor** reads a near-constant 4&nbsp;cm long. Its precision (the sample standard deviation) is 2.5&nbsp;cm.
- The **right anchor** has a small scale error. It reads about 12&nbsp;cm short at 0.75&nbsp;m and tracks well after that. Its precision is 3.2&nbsp;cm.
- The **two-anchor mean** is accurate to 3.3&nbsp;cm RMSE. This value is better than the module's &plusmn;10&nbsp;cm specification.

For a low-cost UWB pair, this is a solid result. The follow controller can trust the range.

### The Right-Anchor Surprise

An earlier moving test had worried me. During that run the right anchor fed bad ranges. Some readings were near zero. Others broke the geometry of the two anchors. I suspected a faulty module or a bad mount.

This static test clears the module. Head-on and with a clear view, the right anchor is accurate to a few centimetres. So the earlier fault was not the hardware. It was the operating condition during motion. The likely cause is a blocked or reflected signal (non-line-of-sight) at some cart angles as I walk. That points the fix at the mount and the view, not at the part. This is a much better place to be.

### The Calibration

I used the sweep to calibrate each anchor. I set a software offset of &minus;0.04&nbsp;m on the left and &minus;0.20&nbsp;m on the right. The offset removes the mean bias. The precision does not change. The remaining spread, near 3&nbsp;cm, is the true noise floor. It sets the range-noise term in the Kalman filter.

### Next

The static accuracy is known. The next step is the moving accuracy. I will log the fused estimate against a known path while the cart follows. That test will show how the Kalman filter smooths the walk, and how the non-line-of-sight dropouts behave in motion.
