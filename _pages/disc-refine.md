---
layout: "page"
title: "Spot discovery and refinement"
permalink: disc-refine
---

Continuing with the processing steps discussed in the [detector calibration steps](cspad-calib), we now examine indexing and spot discovery. These steps are once again summarised from the [CCN article](http://www.phenix-online.org/newsletter/CCN_2016_07.pdf#page=18), [cctbx.xfel wiki](http://viper.lbl.gov/cctbx.xfel/index.php), with citations given for other locations where necessary. At this stage in the processing pipe-line our goal is to find Bragg peaks, index them, refine them and perform integration over the data (if necessary).

Ideally, we aim to predict the Miller indices $$h,k,l$$ of the crystal lattice unit cell, and hence use this model to better determine the structure of the overall data. This is challenging, as the data is collected in single shots at undetermined crystal orientations, and only partially samples the overall structure factor. By examing the data to within regions masked by the crystal model allows for a lowering of overall noise in the collected data. As a result Bragg spots that would otherwise be difficult to observe can contribute to the collection and analysis, allowing a better calculation of the structure factor. [[1](doi:10.1038/nmeth.2887),[2](doi:10.7554/eLife.05421)]


For the purpose of determining the structure factors of the lattice, the following steps should be performed:

- Identify Bragg peaks and construct a crystal lattice model.
- Use that model to mask data sites and integrate over these Bragg sites.
- Iteratively refine the model using the newly analysed data until sufficiently converged.
- Merge the data to determine the structure factors.

However, in this guide, we aim to improve and refine the crystal model, as well as better determine the experimental parameters that affect diffraction intensities. Merging will be performed on a large scale once we have optimised the experimental model and parameters.

## Bragg spot discovery and integration
For this stage in the process, we must parameterise the spotfinding process to ensure it finds the best Bragg peaks to determine a crystal model for further refinement and analysis. For this we can use the <!---[DIALS](http://dials.lbl.gov/) spot-finding algorithm -->
cctbx.xfel program `cctbx.xfel.xtc_process`, which will use the [DIALS](http://dials.lbl.gov/) back-end for performing spot-finding and integration. By providing appropriate parameters to `cctbx.xfel.xtc_process` we can optimally determine a crystal model.

  ```Bash
  sample code here; currently tied to LCLS
  ```


Following the output of the `cctbx.xfel.xtc_process` command above, a new geometric refinement of the detector positions for the CSPAD can be calculated using the command `cspad.cbf_metrology`.





## Merging
