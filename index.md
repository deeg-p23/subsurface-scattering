---
title: "Subsurface Scattering"
permalink: /
layout: default
---

# Introduction
Hero Image: Stanford Dragon using a skin-like BSSRDF

# Methods
In its current state, the BSSRDF model strictly computes the dipole diffusion approximation [1], with which the throughput is multiplied, and the radiance reaccumulates; all of which occurs after accumulation by a BXDF evaluation.

For $N_{sss}$ samples, exit rays are sampled following a weighted MIS method for BSSRDF profiles [2]. This method uniquely samples an axis (along the hit's surface normal, direction of change of $u$ coordinate, or the $v$ coordinate), then samples the radius and angle of a point on a normal-oriented disk, and casts a ray into it. If BSSRDF data is acquired, the evaluation of the approximation, MIS weight, and PDF starts from there.

# Results

# Postmortem
No MIS?
Wanted to do tabulation for faster queries, akin to PBRT's TabulatedBSSRDF implementation.

# References
[1] Jensen et. al, 2001
[2] King et. al, 2013