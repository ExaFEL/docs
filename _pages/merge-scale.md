---
layout: "page"
title: "Merging, scaling and post-refinement"
permalink: merge-scale
---

The output of the steps outlined in the [discovery and refinement steps](disc-refine.html) generally holds information on Miller indices, integrated intensities, and errors for the observed reflections$$~$$[[1](http://viper.lbl.gov/cctbx.xfel/index.php/Merging)]. The merging process aims to unify these data into a single set. Namely, the data is combined and adjusted so all values are consistent and within the same range.

## Merging and scaling with **cxi.merge**

For the scaling and merging procedure, we use the program `cxi.merge`. Again, to view all available options the `-h` flag can be passed, or also with `-v` for more information. For the merging of a data set determined from the previous steps, we can supply the following command:

  ```Bash
    cxi.merge data=<integrated data dir> pixel_size=0.11 \
    output.prefix=cxim1416_merged model=<PDB reference for scaling> \
    nproc=<num of cores> backend=FS scaling.mtz_file=cxim1416_out.mtz
  ```
The `model` option provides a [PDB](http://www.rcsb.org/pdb/home/home.do) model to the command, and allows for a known structure to be used to assist with the scaling of the data. The pixel size chosen is the known value for the CSPAD detectors. `backend` informs the command to write all data to a flat file during the processing (can also specify databases installed on local/remote systems). The above command can be further modified with different options to better suit the dataset being examined. Example usage of the commands are given on the [cctbx.xfel wiki](http://viper.lbl.gov/cctbx.xfel/index.php/2017_cxi_merge_tutorial).

## Post-refinement with **prime**

The program `prime` [1](http://dx.doi.org/10.7554/eLife.05421) is used on the resulting merged data to further refine the model and parameters.

[tutorial here](http://viper.lbl.gov/cctbx.xfel/index.php/2017_prime_tutorial)
