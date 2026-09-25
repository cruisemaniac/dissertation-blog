---
layout: post
title: "Carrying a Load"
date: 2026-09-25 23:55:00 +0400
categories: [Updates, Research]
---

A follow-me cart that follows well when it is empty has done half its job. The point of the cart is to carry things. So tonight we put weight on it and ran the follow again.

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;margin:1.5em 0;">
  <iframe src="https://www.youtube-nocookie.com/embed/PYX0VG_MnSU" title="Trolley-X follow test" style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

If the video does not load, watch it here: [youtu.be/PYX0VG_MnSU](https://youtu.be/PYX0VG_MnSU).

### The test

One session, two parts. First, a normal follow with no payload. Then we stopped, disconnected the UWB so we could inspect the cart, put two 2 kg dumbbells on it, and followed again. The payload was 4 kg. I cut the inspection periods out of the log, because the logger repeats the last value while the UWB is off, and those rows are not real data.

I measured both parts with the same method as the report. The hold error is the fused range minus the 1.02 m set-point, while the cart holds still. The speed comes from the wheel encoders, as a mean while the cart drives.

| Metric | No payload | 4 kg |
|---|---|---|
| Length / time driving | 63 s / 19% | 28 s / 46% |
| Hold error, MAE / RMSE | 14.2 / 19.4 cm | 19.1 / 38.1 cm |
| Hold bias | −11.2 cm | +14.3 cm |
| Filter consistency (NIS mean) | 0.87 | 1.52 |
| Bearing jitter cut by the filter | 82% | 71% |
| Commanded speed | 0.256 m/s | 0.233 m/s |
| Actual speed | 0.202 m/s | 0.153 m/s |
| Actual / commanded | 0.79 | 0.66 |

### What the load changed

The drive, and only the drive. With 4 kg on board, the cart reached 66 percent of the speed it was asked for, against 79 percent empty. For the same command, it was about a quarter slower. The effect on following is easy to see in the hold bias. Empty, the cart held about 11 cm too close. Loaded, it held about 14 cm too far. It fell behind me, because the motors could not deliver the speed the controller asked for, and nothing in the loop corrects for that.

### What the load did not change

The estimate. The filter stayed consistent with the measured noise, and it still cut the bearing jitter by 71 to 82 percent. That is what I expected, because the UWB does not care what the cart carries. But it is good to see it in the data and not only in theory. It also tells me where to put the next effort. The weak point under load is not the sensing. It is the open-loop drive: the base controller turns a speed into a PWM value and trusts that the motor delivers it. Speed feedback from the encoders into the drive would close that gap.

### The honest caveat

This is one run, and the loaded part is short: 28 seconds, with less than 13 seconds of driving. 4 kg is a light load, and we have not measured the cart's own mass, so I cannot say what fraction of the cart it is. Treat this as an indication, not a result. A proper payload test needs heavier steps, several runs per step, and the stopping distance measured at each one.
