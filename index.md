---
layout: default
---
This landing page for the ExaFEL project will feature an updated list of the necessary steps to process data using the developed software pipeline, citing examples where applicable.

---
<br/>

## Getting started
  - [Software installation](psana-cctbx-install): steps for performing the installation of the data analysis software pipeline.
  - Software build tests: Test routines to ensure the data analysis software pipeline has been successfully built. Currently in progress.

<br/>

---
<br/>


## Data processing steps
The following steps must be undertaken to analyse crystallographic data sets using the ExaFEL software pipeline:
  1. [Initial calibration](cspad-calib): the necessary steps to perform initial calibration of the detector prior to analysing data.
  2. [Discovery, refinement and integration](disc-refine): using the initial calibration, spot finding and detector refinement is performed to determine the crystal structure and better calculate the experimental parameters. Integration of the data is performed upon completion of the refinement.
  3. [Merging, scaling and post-refinement](merge-scale): perform merging of the processed data using the optimised parameters from the previous stages.

  <br/>

## Processing on Cori Phase II
  1. [Shifter image creation](docker): building cctbx.xfel and loading onto Cori using the Shifter (Docker) container utility.
  2. [Processing LD91](ld91-knl): data processing steps, performance, and analysis using cctbx.xfel on Cori Knight's Landing.


  ---
  <br/>
