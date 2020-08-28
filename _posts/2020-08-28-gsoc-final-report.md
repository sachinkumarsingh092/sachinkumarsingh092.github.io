---
title: Final Work Product Submission for GSoC'20
tags: [GSoC]
style: fill
color: primary
description: A short description of what work was done, what code got merged, what code didn't get merged, and what's left to do.
---

# Image registration with the celestial sphere and non-linear warping in GNU Astro.

{% include elements/figure.html image="../assets/Heckert_GNU_white.svg" caption="GNU" %}

The aim of this project is to re-align different images to a common coordinate system and make a common cataloge (registration). After registration, we have to build the library to remove non-linear warpings from them. Finally all of this seperate projects have to be mergd with the existing GNU Astro ecosystem.

### Work Done

The proposal was changed a bit before starting the project. It was decided to make functionlities for distortion conversions for easy use of GNU Astro in an analysis pipeline using different software.

The main repository for the distortion converion is [here](https://gitlab.com/sachinkumarsingh092/gnuastro-test-files).

- Implemented TPV to SIP distrotion conversions.
- Implemented SIP (reverse and forward) to TPV distortion conversions. 
- Implemented scripts for generating the polynomial equations which are finally used for conversions.

All commits regarding this project is given [here](https://gitlab.com/sachinkumarsingh092/gnuastro-test-files/-/commits/master).


### Tasks

### Others

- [GSoC'20 blog-post](https://sachinkumarsingh092.github.io/blog/gsoc-post1).
- [GNU Astro repository on savannah](git.savannah.gnu.org/cgit/gnuastro.git/).
