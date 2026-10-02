---
layout: page
title: Pinger Detection
---

[← Back to home](/)

<div style="display: flex; justify-content: center; align-items: center; gap: 10px;">
  <img src="/images/h2o3dtop.png" alt="H2O 3D top view" style="height: 300px; width: auto;">
  <img src="/images/h2o3dbottom.png" alt="H2O 3D bottom view" style="height: 300px; width: auto;">
</div>

## Motivation
A key component of the RoboSub Autonomy Challenge is the ability to listen for acoustic pingers in the pool. During each competition run, two Teledyne Benthos ALP-365 pingers placed in front of tasks to communicate to the AUV which one to complete first for maximum points. They do this by sending out periodic 4 ms pulses at a specified frequency (at RoboSub there are usually 4 courses in the same pool being used at once, so each course has to have its own frequency). So to make these signals useful, we need a way to characterize their frequency and direction of origin. 

<div style="display: flex; justify-content: center; gap: 10px;">
  <img src="/images/highlevel.png" alt="H20 3D top view" style="width: 90%;">
</div>

## High Level Idea
Pulses from the pinger travel as pressure waves through the water, which can be converted to a more useful analog voltage by a transducer. With some additional amplification and filtering, we can eliminate much of the ambient pool noise and higher order harmonics, sample the result faster than the Nyquist Rate (in this case, 80 ksps), and at that point we have what we need to calculate the strength of the signal's frequency components. 

But how do we calculate where it came from? For that we can use a few more transducers and some convenient geometry. 

<div style="display: flex; justify-content: center; gap: 10px;">
  <img src="/images/pingsetup.png" alt="H20 3D top view" style="width: 60%;">
</div>

Say we model the situation as in the image above, with sound waves from a pinger pulse approaching with horizontal velocity v (in a pool at 70°F, about 1480 m/s) and heading 𝛳. Note the the transducer array is a right isosceles triange with two equal side lengths d. This wave front will hit transducer 1 first, then transducer 3, followed by transducer 2. As it turns out, this order, and the timing delays between each transducer, are directly related to the approach angle 𝛳: 

<div style="display: flex; justify-content: center; align-items: center; gap: 10px;">
  <img src="/images/x12.png" alt="H2O 3D top view" style="height: 300px; width: auto;">
  <img src="/images/x32.png" alt="H2O 3D bottom view" style="height: 300px; width: auto;">
</div>

$\Delta x$, the difference in path length between a pair of transducers, is directly proportional to something we can measure: the relative phase in the sinusoidal voltages at each transducer.

$$
\Delta x = \frac{2\pi f\phi}{v}
$$

So if we combine our two equations for path length difference we get the equation below for heading purely as a function of phase differences, with no dependence on $d$:

$$
\theta = \text{atan2}(\phi_{23},\phi_{21})
$$

Keep in mind this is just for horizontal heading. We'll add a 4th transducer to account for vertical displacement in a similar fashion, but in the relatively shallow competition pool the horizontal is much more relevant to us. 

## Analog Front End

Next I'll dive a little deeper into the hardware that actually implements this solution. The job of the analog front end is to supply a clean, sinusoidal voltage from the pulse produced by a pinger to an ADC. 


<div style="display: flex; justify-content: center; gap: 10px;">
  <img src="/images/piezomodel.png" alt="H20 3D top view" style="width: 60%;">
</div>


That starts at the output of the transducer, which houses a piezoelectric material that produces charge in response to mechanical stress. This can be modeled as a current source whose value is the rate of change of the charge on a capacitor (see this <a href="https://www.allaboutcircuits.com/technical-articles/understanding-and-modeling-piezoelectric-sensors/" target="_blank" rel="noopener noreferrer">article</a> for an explanation). 

<div style="display: flex; justify-content: center; align-items: center; gap: 10px;">
  <img src="/images/tc4013model.png" alt="H2O 3D top view" style="height: 300px; width: auto;">
  <img src="/images/tc4013sensitivity.png" alt="H2O 3D bottom view" style="height: 300px; width: auto;">
</div>

By itself, this arrangement would produce a small voltage across the terminals of the capacitor. If we multiply the rated acoustic output of our ALP-365 pinger (177 dB μPa @ 1m) with the receiving sensitivity of our TC4013 hydrophones (about -213 dB V/μPa @ 1m), this comes out to 15.8 mV RMS or 22.4 mV amplitude around the closest range we would see. A little small, but not an unworkable signal strength. The bigger issue is when this circuit has to drive a filter or ADC input. In this 1m scenario the charge that accumulates on the capacitor has an RMS value of just 47.4 pC (Q = CV = 3nF * 15.8 mV RMS), which means any load placed on the signal can significantly alter it. 

<div style="display: flex; justify-content: center; gap: 10px;">
  <img src="/images/amp.png" alt="H20 3D top view" style="width: 60%;">
</div>

The charge amplifier circuit above remedies many of these issues. It acts as an integrator that converts charge from the piezo into voltage, with high input impedance to minimize signal loss. Additionally, it allows us to set the reference voltage at mid-rail instead of 0 (Vs+ = 5V, REF = 2.5V). The voltage seen out of this amplifier, as a function of Q, is: 

$$
V_{\text{out}} = -\frac{Q}{C_f}
$$

So building on our prediction for Q from earlier, we can estimate the voltage amplitude coming out of these amplifiers: 

$$
V_{\text{out, max}} = \frac{(47.4\,\text{pC})\sqrt{2}}{300\,\text{pF}} = 0.223\\text{V}
$$

*add scope images

Next up in the signal chain is filtering. A pool full of people and submarines swimming around produces all kinds of acoustic noise that can impact our signal quality. Since we know the bandwidth of available pinger frequencies ahead of time, it makes sense to bandpass filter our signal for that range and eliminate frequency components we don't want. 

<div style="display: flex; justify-content: center; gap: 10px;">
  <img src="/images/ad.png" alt="H20 3D top view" style="width: 80%;">
</div>


The filters for this board were designed using Analog Devices' <a href="https://tools.analog.com/en/filterwizard/?ADICID=PDSR_Global_Filter-Wizard-NB_Google_PSC_202603&gad_source=1&gad_campaignid=23683883414&gbraid=0AAAAACxqTx_Xfe7f-cRkkzkx6p-IV7UOw&gclid=CjwKCAjwoOjVBhArEiwAUwDakwNL_bR7jN2Bn_kd2Vxo9xlC5q8A0Da15s5ZbXMa50bfxaklUu15phoCGoQQAvD_BwE" target="_blank" rel="noopener noreferrer">filter wizard</a> tool, as shown above. This configuration produces a passband from 25.5 kHz to 41.5 kHz and relatively low passband ripple in exchange for a more gradual rolloff. For this application, our main concern beyond passing the right frequencies is phase matching between channels, and this design strikes a good balance between those demands and noise rejection. 

<div style="display: flex; justify-content: center; gap: 10px;">
  <img src="/images/filter.png" alt="H20 3D top view" style="width: 80%;">
</div>

Above is the filter schematic realized in Altium. 

Testing

## Analog to Digital
At this point, the voltage out of our filters looks something like the image below. Pulses 4 milliseconds long, spaced by 0.5, 1 or 2 seconds depending on the settings of the pinger, with some frequency between 25 and 40 kHz.

<div style="display: flex; justify-content: center; gap: 10px;">
  <img src="/images/filterout.png" alt="H20 3D top view" style="width: 80%;">
</div>

The next task is to sample the signal, which means driving an ADC to fill a buffer of size N with samples, and once its full performing our signal analysis to calculate frequency, phase and heading. The next section will cover that processing in firmware, but there are a few constraints for sampling that have to be resolved first. 

First, each of the 4 channels needs to be sampled simultaneously, or as close as possible. Our method for heading calculation is completely dependent on accurate phase information, which we can't get if there's some unknown delay in the sampling of each channel. 

Second, the Shannon-Nyquist Sampling Theorem tells us that in order to reconstruct a continous analog signal from its samples, our sampling rate needs to be greater than twice the highest frequency component of the signal. In this case, the Benthos ALP-365 can go up to 40 kHz, which means our minimum sampling rate is 80 kHz. 

Finally, because we receive pulses instead of a continuous sinusoid at 25-40 kHz, we have to be able to sample and process fast enough to guarantee that for each pulse, we will have a full buffer of samples that sits completely inside it. In other words, we need to continuously fill new buffers every 2 ms, if not faster, and for our processing of those buffers to be able to keep up with that pace. 

## Two Methods
The simpler way to approach this is to use internal ADCs on the STM32. The chip used for this project, an H7 series, has 3 separate ADC peripherals with 16 bit resolution that can run comfortably over 1Msps, easily hitting the timing requirements of a single channel. We can also get near simultaneous sampling by triggering ADC conversions from a timer peripheral. The tradeoff is the fourth channel is more difficult to integrate, 

external 

adc config

sampling state machine

## Signal Processing

IQ modulation

Phase and heading calculatution

state machine

serial


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
