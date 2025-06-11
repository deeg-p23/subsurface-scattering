---
title: "Subsurface Scattering"
permalink: /
layout: default
---

# Introduction

<p align="center">
  <img src="https://github.com/user-attachments/assets/5d0df5a8-842c-40c2-b850-f7fd0b4f48f9" width="100%"/>
</p>
<p align="center">
  "Hero" Image, Skin Dragon
</p>

As a personal goal for the end of this course, I aim to learn how to model different forms of light transport. This particular project kicks this journey off by implementing **subsurface scattering**, with the primary motivation of being able to model the translucency of materials like skin, and the glossiness on materials like marble. The implementation is based on the BSSRDF model proposed by Jensen et. al (2001), approximating dipole diffusion to compute outgoing radiance after multiple bounces under the local surface of a given intersection. 

The above image showcases the effect of subsurface scattering through smooth skin.

# Methods
In its current state, the BSSRDF model strictly computes the dipole diffusion approximation [1], with which the throughput is multiplied, and the radiance reaccumulates; all of which occurs after accumulation by a BXDF evaluation.

For $N_{sss}$ samples, exit rays are sampled following a weighted MIS method for BSSRDF profiles [2]. This method uniquely samples an axis (along the hit's surface normal, direction of change of $u$ coordinate, or the $v$ coordinate [3]), then samples the radius and angle of a point on a normal-oriented disk, and casts a ray into it. If BSSRDF data is acquired, the evaluation of the approximation, MIS weight, and PDF starts from there.

For the dipole model, the BSSRDF term is computed, $S_d$: getting the product of the Fresnel Transmittance for the incident and outgoing angles, the approximated diffuse BSSRDF $R_d$ which has our distance between the sampled and hit points as an input, and $\frac{1}{\pi}$ as a safe $c$ constant to go with. Additionally, to satisfy the rendering equation, the final outgoing radiance for this sample is multiplied by the direct radiance of a uniformly sampled light in the scene, alongside its cosine term. Without this last term, scattered light will centers more generally about the object, rather than directing the magnitude of the neighboring lights realistically.

Getting the appropriate end PDF involves accounting for the probability of the axis chosen, the PDF given by the distance to the sampled direct light's area, and the disc PDF, obtained by some more gaussian sampling. Finally, the weight of the given sample is computed according to the axis, and the diffuse radiance is multiplied by the aggregated inverse PDF and the weight.

In addition to the project's primary implementations, tinyobjloader was also included to collect triangle data from .obj files and instantiate Triangle objects, in the same vein as the .test files handle instancing and buffeirng geometry data.

# Development

My original implementation was written as essentially an in-sequence read of just the original 2001 paper. A rapid-fire approach like that definitely caused more headaches than it was worth, compared to taking a step back and assessing other guides to get a better overview on the subject as I did now.

I started with a backlit version of the original dragon scene from homework 3. I seemed to have gotten the diffuse approximation generally correct (lighting bleeding through faces opposite to the light source), the PDF was blown to immensely low values. I also made a gross assumption that a BXDF material was entirely replaceable with a BSSRDF, so this may have been another issue during the early stage of the project.

<p align="center">
  <img src="https://github.com/user-attachments/assets/f0b00085-2792-4806-a978-17d57b2539c6" width="45%" style="display:inline-block;"/>
  <img src="https://github.com/user-attachments/assets/b6141043-8856-43a8-85c1-19b282615a1d" width="45%" style="display:inline-block;"/>
</p>
<p align="center">
  Somewhat of a noir look.
</p>

I also hadn't pieced together the requirement of SSS samples, so my radiance was initially tied to the quantity of Monte Carlo samples made, which obviously made the renders nastily brighter and more saturated.

Eventually, I got to a stage where I resolved those misinterpretations of the model. Outstanding tasks at this point were to fix artifacts from shadow and BSSRDF ray cast issues, and to make use of the aforementioned sampling methods rather than only disk and uniform sampling.

<p align="center">
  <img src="https://github.com/user-attachments/assets/bc244dc5-0ad3-46ba-a4d9-7109908d0c59" width="45%" style="display:inline-block;"/>
</p>
<p align="center">
  A Paper Dragon?
</p>

# Results
Since only the diffuse approximation was properly implemented, I chose to model as much as I could that capitalized on the properties of such a BSSRDF. I had a particular interest in rendering scenes that showcased the illuminatory effect translucent objects give under direct lighting.

The following renders showcase the same dragon with opposed diffuse and specular colors, one reddish and one blueish. Most importantly however, the Hot dragon uses a GGX BRDF with a roughness of 1, and a BSSRDF that has high absorption. The Icy dragon uses a phong BRDF with a shininess of 100, and a BSSRDF that has high scattering.

<p align="center">
  <img src="https://github.com/user-attachments/assets/0615d62c-7858-4e2a-a149-d2c2d3d5e4ab" width="45%" style="display:inline-block;"/>
  <img src="https://github.com/user-attachments/assets/530d6d37-9891-4888-bd69-645276567329" width="45%" style="display:inline-block;"/>
</p>
<p align="center">
  Hot and Icy
</p>

Next up, same scene, but I swapped a back light for a top one in order to shine a bit more light on the effects changing the mean cosine of the scattering angle ($g$) has on an object. 

The refraction in a naturally metallic looking phong material can go from a foil-like appearance, to something more like a tough crystal. From the top-left clockwise, $g=(0.15, 0.5, 0.85, 0.95)$.

<p align="center">
  <img src="https://github.com/user-attachments/assets/a1bd6f18-f402-4a2b-be47-ac8247a69fb7" width="45%" style="display:inline-block; margin:1%;" />
  <img src="https://github.com/user-attachments/assets/fdcbe76d-bde1-4d05-a265-675f9af80fa3" width="45%" style="display:inline-block; margin:1%;" />
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/0565d592-ba81-47b6-ac0b-eaf4f8c066d3" width="45%" style="display:inline-block; margin:1%;" />
  <img src="https://github.com/user-attachments/assets/7e86cb78-47d3-4282-928d-e85be1c493f6" width="45%" style="display:inline-block; margin:1%;" />
</p>
<p align="center">
  Shiny 4x
</p>

Lastly, here's another angle of the Skin Dragon with slightly different properties to showcase light carried by the BSSRDF reflecting off of shinier surfaces. The left figure has a shinier wall and does not use any of the general importance sampling techniques, and the right figure a rougher wall using the BRDF importance sampling technique implemented in class. Significant less noise would be found in a render with MIS enabled.

<p align="center">
  <img src="https://github.com/user-attachments/assets/d110c000-7f37-4dd1-96c3-2067e3aa2e7e" width="45%" style="display:inline-block;"/>
  <img src="https://github.com/user-attachments/assets/c71fee96-5857-4267-b6cd-3e5759bf0c9c" width="45%" style="display:inline-block;"/>
</p>
<p align="center">
  I'm a big fan of the Stanford Dragon, clearly.
</p>

# Postmortem
There's certainly a lot left for me to continue from where I left off here.

The most (visually) important task on the to-do list though is giving the single scattering term another shot for this model, to hopefully soon present a neat looking statue. An attempt was made, but the term had caused some artifacts along sharper faces, as in the following render.

<p align="center">
  <img src="https://github.com/user-attachments/assets/e16cc25d-1ec1-4505-941a-fae57ea7e642" width="45%" style="display:inline-block;"/>
</p>
<p align="center">
  It still looks pretty cool though.
</p>

Another task is to reorganize class structures. Attributing instances of Materials to certain Objects will make future light transport implementations much more intuitive, and also omit the quantity of redundant computations that do occur.

Speaking of redundant computations, there's definitely a lot of room for improving efficiency by just precomputing a lot of simplified data we otherwise find mid-render. PBRT's TabulatedBSSRDF perfectly demonstrates this by setting up unique scattering profiles with tables of data ready to go, making for much faster queries. This is another thing I would've liked to try with this project, so it's something for me to do a bit more research on moving forward *(In general, the chapters of the PBR book covering SSS have been great reads, so I'm eager to check out more from there)*.

Though I wish I could have setup a more varied gallery of renders and materials, I'm pretty pleased with what I've been able to learn of light transport techniques -- and more holistically of course -- about ray tracing from the entirety of this course.

Thanks!

- Diego Pereyra

# References
Unnumbered sources are not specifically referenced anywhere in this document, but are acknowledged as used during the course of this project.
### Paper(s)

[1] Jensen, Henrik Wann, et al. “A Practical Model for Subsurface Light Transport.” Stanford University, 2001. https://graphics.stanford.edu/papers/bssrdf/bssrdf.pdf

[2] King, Alan, et al. “Practical Multiple Importance Sampling for BSSRDFs.” Sony Pictures Imageworks Technical Paper, 2013. https://www.imageworks.com/sites/default/files/2023-10/BSSRDF-importance-sampling-imageworks-library-BSSRDF-sampling.pdf

### Resource(s)

[3] Pharr, Matt, et al. “The BSSRDF.” In Physically Based Rendering: From Theory to Implementation, 3rd ed., 2018. https://pbr-book.org/3ed-2018/Volume_Scattering/The_BSSRDF

Gao, Duan. “Understanding BSSRDF and Subsurface Scattering.” Personal Blog, 2022. https://gao-duan.github.io/blogs/bssrdf/index.html

A Graphics Guy. “Practical Tips for Implementing Subsurface Scattering in a Ray Tracer.” Personal Blog, 2023. https://agraphicsguynotes.com/posts/practical_tips_for_implementing_subsurface_scattering_in_a_ray_tracer/#how-does-it-fit-in-a-path-tracing-algorithm

### Library/Libraries

syoyo. tinyobjloader: A Single-Header Wavefront OBJ Loader. GitHub Repository, 2025 https://github.com/tinyobjloader/tinyobjloader
