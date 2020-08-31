---
title: Final GSoC'20 report for GNU Astro
tags: [GSoC]
style: fill
color: primary
description: A short description of what work was done, what code got merged, what code didn't get merged, and what's left to do.
---

![alt text](../assets/GSoC.png "Google Summer of Code 2020")
The aim of this project is to re-align different images to a common coordinate system and make a common cataloge. This is called image registration. After registration, we have to build the library to remove non-linear warpings from them. Finally all these seperate projects have to be merged with the existing GNU Astro ecosystem.

### Work Done

The proposal was changed a bit before starting the project. It was decided to make functionlities for distortion conversions for easy use of GNU Astro in an analysis pipeline using different software.

##### The main repository for the distortion converion is [here](https://gitlab.com/sachinkumarsingh092/gnuastro-test-files).

- Implemented TPV to SIP distrotion conversions.
- Implemented SIP (reverse and forward) to TPV distortion conversions. 
- Implemented scripts for generating the polynomial equations which are finally used for conversions.

All commits for this project are [here](https://gitlab.com/sachinkumarsingh092/gnuastro-test-files/-/commits/master).

Final merged commits in [savannah](http://git.savannah.gnu.org/cgit/gnuastro.git/):

- [All convertions between different WCS distortions](http://git.savannah.gnu.org/cgit/gnuastro.git/commit/?id=7da840d48a1364a339ec48a06d9b6fb2ca5be9ad).
- [Fixed --wcsdistortion when used without HDUs data](http://git.savannah.gnu.org/cgit/gnuastro.git/commit/?id=7dca196b6f7f588482772f3c059866647e812689).
- [Fixed default size with --wcsdistortion=SIP when no input data is provided.](http://git.savannah.gnu.org/cgit/gnuastro.git/commit/?id=808c95dc56baf023928eeab3edf8bc6e3f572de0).

##### The main repository for the quad-hash matching algorithm is [here](https://gitlab.com/sachinkumarsingh092/gnuastro-matching-fits-nohealpix).

- Implemented an efficient method to find possible quads in catalogue without the use of any external library (such as HEALPix).
- Implemented multithreaded operations when finding a quad and making its hash code improving efficiency during large data computations.
- Implemented functions to help in visualisations during development.
- Implemented an index-based `kd-tree` which is highly space/RAM efficient based on `Quickselect` method.
- Implemented nearest neighbours searches for quads using kd-tree.

All commits for this project are [here](https://gitlab.com/sachinkumarsingh092/gnuastro-matching-fits-nohealpix/-/commits/master).

Final merged commits in [savannah](http://git.savannah.gnu.org/cgit/gnuastro.git/):

- [Implemented new k-d tree library](http://git.savannah.gnu.org/cgit/gnuastro.git/commit/?id=cf6ca3cc98a4df0dd5c927ce02115d07d480aea9).

The final matching method to finally find variables for WCS calculations are on [this](https://gitlab.com/sachinkumarsingh092/gnuastro-matching-fits-nohealpix/-/tree/make-wcs-matrix) branch which will be merged after a more robust method for matching is found.

### Tasks Done and discusssions on Savannah

- Implemented TPV and SIP keywords when WCS distortions are necessary.([#15128](https://savannah.gnu.org/task/index.php?15128))
- Implemented calculations for reverse coefficients of TPV to SIP.([#15670](https://savannah.gnu.org/task/index.php?15670))
- Implemented calculations for forward coefficients of TPV to SIP.([#15669](https://savannah.gnu.org/task/index.php?15669))
- Implemented calculations for SIP to TPV coversions.([#15676](https://savannah.gnu.org/task/index.php?15676))
- Fixed problem during building of wcsdistortion library in macOS([#58623](https://savannah.gnu.org/bugs/index.php?58623)).
- Fixed error in calculation of SIP to TPV coversions([#58640](https://savannah.gnu.org/bugs/index.php?58640)).
- Implemented fix for the default size for `--wcsdistortion` when no data provided.([#15704](https://savannah.gnu.org/task/index.php?15704))
- Made a HEALPix grid on the gaia index catalogue.([#15700](https://savannah.gnu.org/task/index.php?15700))
- Made quad structure with hashes and store in kd-tree.([#15713](https://savannah.gnu.org/task/index.php?15713))

All the code that is finally merged to main repository is [here](http://git.savannah.gnu.org/cgit/gnuastro.git/log/?qt=author&q=Sachin+Kumar+Singh).
All tasks that are done (including tasks before the coding period) are listed [here](https://savannah.gnu.org/task/index.php?go_report=Apply&group=gnuastro&func=browse&set=custom&msort=0&report_id=100&advsrch=0&status_id=0&resolution_id=0&assigned_to=180314&category_id=0&bug_group_id=0&history_search=0&history_field=0&history_event=modified&history_date_dayfd=28&history_date_monthfd=8&history_date_yearfd=2020&chunksz=100&spamscore=5&boxoptionwanted=1#options). All bugs fixed (including bugs fixed before the coding period) are listed [here](https://savannah.gnu.org/bugs/index.php?go_report=Apply&group=gnuastro&func=browse&set=custom&msort=0&report_id=100&advsrch=0&status_id=0&resolution_id=0&assigned_to=180314&category_id=0&bug_group_id=0&history_search=0&history_field=0&history_event=modified&history_date_dayfd=29&history_date_monthfd=8&history_date_yearfd=2020&chunksz=50&spamscore=5&boxoptionwanted=1#options).

### What's left

- More robust method to match the query quads to reference quads is to be found. Our aim is to achieve a highly efficient software which is both space/RAM and CPU efficient. The current method is brute force approach which can possibly fail in certain cases. Range-based matching within a specific threshold is required.
- Due to change in proposal, the non-linear warping is to be done later on when matching is working properly.

### Final Words

GSoC was a wonderful experience for me. I now feel a bit more comfortable with the GNU Astro's codebase. I learned a lot of useful stuff like index-based implementations, use tool like GDB for debugging, Git for version control, valgrind for memory leak checking and much more stuff. 
I will try my best to regularly contribute to GNU Astro. Thanks to [Mohammad Akhlaghi](https://akhlaghi.org/) and various others in the GNU Astro development team for such a wonderful experience and the knowledge.

### Others

- [GSoC'20 blog-post](https://sachinkumarsingh092.github.io/blog/gsoc-post1).
- [GNU Astro repository on savannah](git.savannah.gnu.org/cgit/gnuastro.git/).
