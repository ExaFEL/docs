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

## Post-refinement with **prime.postrefine**

The program `prime` [[1](http://dx.doi.org/10.7554/eLife.05421)] is used to perform post-refinement on the resulting data following the integration steps, as well as the subsequent merging. As such, `prime.postrefine` can be seen as an alternative approach to using `cxi.merge`, where the authors suggest it can improve results using lower numbers of crystal images. As before, the full list of available arguments can be viewed by running `prime.postrefine -h`. For more extensive coverage on the use of `prime` please see the [tutorial](http://viper.lbl.gov/cctbx.xfel/index.php/2017_prime_tutorial).

For the purpose of these steps, we will show example commands to post-refine and merge the data from the integration stages. This step will potentially be the final in our data analysis pipeline, though convergence may require several iterations. For a generic PHIL file, the command `prime.run > params.phil` without any arguments generates the required file format for later editing.

For the purpose of this command, we will modify the above PHIL file, changing the below values with the specified parameters.

  ```python
    ### params.phil
    ### prime.postrefine parameters file

    data = <integrated data location>
    #if target unit cell group is known, can be used to discard outliers
    target_unit_cell = 100 100 50 90 90 120 #{a b c} {\alpha \beta \gamma}
    target_space_group = P6
    n_residues = 128

    #CSPAD detector pixel size (mm^2)
    pixel_size_mm = 0.11
    #Sample postrefine parameters
    postref {
      scale {
        d_min = 1.
        d_max = 50.
        sigma_min = 1.5
        partiality_min = 0.1
      }
      allparams {
        flag_on = True
        d_min = 1.
        d_max = 50.
        sigma_min = 1.5
        partiality_min = 0.1
        uc_tolerance = 5
      }
    }
    #Sample postrefine parameters
    merge {
     d_min = 1.
     d_max = 50.
     sigma_min = -3.0
     partiality_min = 0.1
     uc_tolerance = 5
    }
    n_processors = <available cores>
  ```

To perform the processing step we now pass run `prime.run ./params.phil`. The program output will be placed into a directory `Prime_Run_<run number>`, with all the merged statistics available in the file `log.txt`. This will conclude the processing steps of the data, which can now be examined for further analysis.
