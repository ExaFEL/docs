---
#
# You don't need to edit this file, it's empty on purpose.
# Edit minima's home layout instead if you wanna make some changes
# See: https://jekyllrb.com/docs/themes/#overriding-theme-defaults
#
layout: default
---

## Getting started
  - [Software installation](psana-cctbx-install): steps for performing the installation of the data analysis software pipeline.
  - Software build tests: Test routines to ensure the data analysis software pipeline has been successfully built. Currently in progress.

## Data processing steps
The following steps must be undertaken to analyse crystallographic data sets using the ExaFEL software pipeline:
  1. [Calibration](cspad-calib): the necessary steps to calibrate the detector prior to analysing data.
  2. [Discovery](disc-refine): using the initial calibration, spot finding and integration is performed to determine a crystal structure. Further refinement and calibration is performed here, until convergence has been reached.
  3. [Batch](batch): perform large scale parallel processing and refinement of the data using the optimised parameters from the previous stages.
