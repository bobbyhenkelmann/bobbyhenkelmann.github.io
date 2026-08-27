---
layout: page
title: Pinger Detection
---

[← Back to home](/)

<div style="display: flex; gap: 10px;">
  <img src="/images/h2o3dtop.png" alt="H20 3D top view" style="width: 48%;">
  <img src="/images/h2o3dbottom.png" alt="H20 3D bottom view" style="width: 48%;">
</div>

A key component of the RoboSub Autonomy Challenge is the ability to listen for pingers, which are placed in front of two tasks in the pool to tell a robot where they are and in which order to attempt the tasks to receive maximum points. To solve this, I built Hydrophones Board, which uses a phased array of 4 Teledyne TC4013 (link) hydrophones to identify pings and the heading to their source. All of the amplification, filtering, and signal processing is done on this single PCB, which passes the results on to our sub's main computer over RS232. 

## The Problem
[2-4 sentences: what needed solving, why it mattered, what constraints you were working under]

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
