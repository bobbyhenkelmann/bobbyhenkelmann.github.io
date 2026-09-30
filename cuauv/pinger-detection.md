---
layout: page
title: Pinger Detection
---

[← Back to home](/)

<div style="display: flex; gap: 10px;">
  <img src="/images/h2o3dtop.png" alt="H20 3D top view" style="width: 48%;">
  <img src="/images/h2o3dbottom.png" alt="H20 3D bottom view" style="width: 48%;">
</div>

## Motivation
A key component of the RoboSub Autonomy Challenge is the ability to listen for acoustic pingers in the pool. During each competition run, two Teledyne Benthos ALP-365 pingers placed in front of tasks to communicate to the AUV which one to complete first for maximum points. They do this by sending out periodic 4 ms pulses at a specified frequency (at RoboSub there are usually 4 courses in the same pool being used at once, so each course has to have its own frequency). So to make these signals useful, we need a way to characterize their frequency and direction of origin. 

<div style="display: flex; gap: 10px;">
  <img src="/images/highlevel.png" alt="H20 3D top view" style="width: 90%;">
</div>

## High Level Idea
Pulses from the pinger travel as pressure waves through the water, which can be converted to a more useful analog voltage by a transducer. With some additional amplification and filtering, we can eliminate much of the ambient pool noise and higher order harmonics, sample the result faster than the Nyquist Rate (in this case, 80 ksps), and at that point we have what we need to calculate the strength of the signal's frequency components. 

But how do we calculate where it came from? For that we can use a few more transducers and some convenient geometry. 

Say we model the situation as in the image above, with sound waves from a pinger pulse approaching with velocity v (in a pool at 70°F, about 1480 m/s) and heading 𝛳. Note the the transducer array is a right isosceles triange with two equal side lengths d. This wave front will hit transducer 1 first, then transducer 3, followed by transducer 2. As it turns out, this order, and the timing delays between each transducer, are directly related to the approach angle 𝛳: 

$\Delta x$, the difference in path length between a pair of transducers, is directly proportional to something we can measure: the relative phase in the sinusoidal voltages at each transducer.

$$
\Delta x = \frac{2\pi f\phi}{v}
$$

So if we combine our two equations for path length difference we get the equation below for heading purely as a function of phase differences, with no dependence on $d$:

$$
\theta = \text{atan2}(\phi_{23},\phi_{21})
$$


## What I Built
[The technical meat. Your approach, key design decisions, and *why* you made them — this is the part that shows engineering judgment, not just "I used X." Trade-offs are good to mention here.]

<div style="display: flex; gap: 10px; flex-wrap: wrap;">
  <img src="/images/h2obottom.png" alt="H2O board bottom" style="width: 23%;">
  <img src="/images/h2ognd.png" alt="H2O board ground layer" style="width: 23%;">
  <img src="/images/h2opower.png" alt="H2O board power layer" style="width: 23%;">
  <img src="/images/h2otop.png" alt="H2O board top" style="width: 23%;">
</div>

## Results
[Did it work? Specs, measurements, test data if you have it. If it's still in progress, say so plainly — "currently in bring-up" is a fine, honest status.]

## What I'd Do Differently
[Optional but strong — 1-2 sentences on a lesson learned or what you'd change with more time/parts/budget. Shows reflection, not just execution.]

**Skills used:** [KiCad, embedded C, ArduPilot, etc. — comma-separated tag list]
