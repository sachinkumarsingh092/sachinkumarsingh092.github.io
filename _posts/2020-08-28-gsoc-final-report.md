---
title: Final GSoC'20 report for GNU Astro
tags: [GSoC]
style: fill
color: primary
description: A short description of what work was done, what code got merged, what code didn't get merged, and what's left to do.
---

{% include elements/figure.html image="../assets/GSoC.png" caption="GNU" %}

The aim of this project is to re-align different images to a common coordinate system and make a common cataloge. This id called image registration. After registration, we have to build the library to remove non-linear warpings from them. Finally all of this seperate projects have to be mergd with the existing GNU Astro ecosystem.

### Work Done

The proposal was changed a bit before starting the project. It was decided to make functionlities for distortion conversions for easy use of GNU Astro in an analysis pipeline using different software.

The main repository for the distortion converion is [here](https://gitlab.com/sachinkumarsingh092/gnuastro-test-files).

- Implemented TPV to SIP distrotion conversions.
- Implemented SIP (reverse and forward) to TPV distortion conversions. 
- Implemented scripts for generating the polynomial equations which are finally used for conversions.

All commits regarding this project are [here](https://gitlab.com/sachinkumarsingh092/gnuastro-test-files/-/commits/master).

Final merged commits in [savannah](http://git.savannah.gnu.org/cgit/gnuastro.git/):

- [All convertions between different WCS distortions](http://git.savannah.gnu.org/cgit/gnuastro.git/commit/?id=7da840d48a1364a339ec48a06d9b6fb2ca5be9ad)
- [Fixed --wcsdistortion when used without HDUs data](http://git.savannah.gnu.org/cgit/gnuastro.git/commit/?id=7dca196b6f7f588482772f3c059866647e812689).
- [Fixed default size with --wcsdistortion=SIP when no input data is provided.](http://git.savannah.gnu.org/cgit/gnuastro.git/commit/?id=808c95dc56baf023928eeab3edf8bc6e3f572de0)


### Tasks Done

- TPV and SIP keywords when WCS distortions are necessary.([#15128](https://savannah.gnu.org/task/index.php?15128))
- Calculating reverse coefficients of TPV to SIP.([#15670](https://savannah.gnu.org/task/index.php?15670))
- Calculating forward coefficients of TPV to SIP.([#15669](https://savannah.gnu.org/task/index.php?15669))
- Calculating SIP to TPV coversions.([#15676](https://savannah.gnu.org/task/index.php?15676))
- Assuming a default size for --wcsdistortion when no data included.([#15704](https://savannah.gnu.org/task/index.php?15704))
- Make a HEALPix grid on the gaia index catalogue.([#15700](https://savannah.gnu.org/task/index.php?15700))
- Make quad structure with hashes and store in kd-tree.([#15713](https://savannah.gnu.org/task/index.php?15713))

All the code that is finally merged to main repository is [here](http://git.savannah.gnu.org/cgit/gnuastro.git/log/?qt=author&q=Sachin+Kumar+Singh).
All tasks that are done are listed [here](https://savannah.gnu.org/task/index.php?go_report=Apply&group=gnuastro&func=browse&set=custom&msort=0&report_id=100&advsrch=0&status_id=0&resolution_id=0&assigned_to=180314&category_id=0&bug_group_id=0&history_search=0&history_field=0&history_event=modified&history_date_dayfd=28&history_date_monthfd=8&history_date_yearfd=2020&chunksz=100&spamscore=5&boxoptionwanted=1#options).

### What's left

- Due to change in proposal, the non-linear warping is to be done later on when maatching is properly working.

### Others

- [GSoC'20 blog-post](https://sachinkumarsingh092.github.io/blog/gsoc-post1).
- [GNU Astro repository on savannah](git.savannah.gnu.org/cgit/gnuastro.git/).
