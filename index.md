---
title: "Subsurface Scattering"
permalink: /
layout: default
---

# Introduction
"Hero" Image: *Skin Dragon*



# Methods
In its current state, the BSSRDF model strictly computes the dipole diffusion approximation [1], with which the throughput is multiplied, and the radiance reaccumulates; all of which occurs after accumulation by a BXDF evaluation.

For $N_{sss}$ samples, exit rays are sampled following a weighted MIS method for BSSRDF profiles [2]. This method uniquely samples an axis (along the hit's surface normal, direction of change of $u$ coordinate, or the $v$ coordinate [3]), then samples the radius and angle of a point on a normal-oriented disk, and casts a ray into it. If BSSRDF data is acquired, the evaluation of the approximation, MIS weight, and PDF starts from there.

For the dipole model, the BSSRDF term is computed, $S_d$: getting the product of the Fresnel Transmittance for the incident and outgoing angles, the approximated diffuse BSSRDF $R_d$ which has our distance between the sampled and hit points as an input, and $\frac{1}{\pi}$ as a safe $c$ constant to go with. Additionally, to satisfy the rendering equation, the final outgoing radiance for this sample is multiplied by the direct radiance of a uniformly sampled light in the scene, alongside its cosine term.

Getting the appropriate end PDF involves accounting for the probability of the axis chosen, the PDF given by the distance to the sampled direct light's area, and the disc PDF, obtained by some more gaussian sampling. Finally, the weight of the given sample is computed according to the axis, and the diffuse radiance is multiplied by the aggregated inverse PDF and the weight.

# Results

# Postmortem
No MIS?
Wanted to do tabulation for faster queries, akin to PBRT's TabulatedBSSRDF implementation.
Uniformly sampling only a single light for the $L_i$ term in $S_d$, although it appears fine in general cases when $N_{sss} \geq 25$, is certainly not a way to go.

# References
[1] Jensen et. al, 2001
[2] King et. al, 2013
[3]