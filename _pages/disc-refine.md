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

## Bragg spot discovery and indexing
For this stage in the process, we must parameterise the spotfinding process to ensure it finds the best Bragg peaks to determine a crystal model for further refinement and analysis. For this we can use the <!---[DIALS](http://dials.lbl.gov/) spot-finding algorithm -->
cctbx.xfel program `cctbx.xfel.xtc_process`, which will use the [DIALS](http://dials.lbl.gov/) back-end for performing spot-finding and integration. By providing appropriate parameters to `cctbx.xfel.xtc_process` we can discover a sufficient number of Bragg peaks to index and determine a model crystal structure.

  ```coffeescript
  cctbx.xfel.xtc_process input.experiment=cxim1416 input.run_num=74 \
   input.address=CxiDs1.0:Cspad.0 format.file_format=cbf \
   format.cbf.detz_offset=585 \
   input.xtc_dir=/net/viper/raid1/mlxd_LM14/psdm/cxi/cxim1416/xtc \
   input.calib_dir=/net/viper/raid1/mlxd_LM14/psdm/cxi/cxim1416/calib
  ```
For the above command we have specified many of the same parameter values as used with `cxi.mpi_average` previously. This command will iterate through the data for the given run number, and determine the spot positions, proceed to determine which peaks can contribute to the model, and then proceed to index them to construct a crystal model. Sample output during the processing of data for the above parameters is:

  ```txt
    Accepted 2016-07-08T22:05Z24.133
    Processing shot 20160708220524133
    Setting spotfinder.filter.min_spot_size=6
    Configuring spot finder from input parameters
    ----------------------------------------------------------------------------
    Finding strong spots in imageset 0
    ----------------------------------------------------------------------------


    Finding spots in image 1 to 1...
    Setting chunksize=1
    Extracting strong pixels from images
     Using multiprocessing with 1 parallel job(s)

    Found 393702 strong pixels on image 1

    Extracted 217561 spots
    Removed 210778 spots with size < 6 pixels
    Removed 5 spots with size > 100 pixels
    Calculated 6778 spot centroids
    Calculated 6778 spot intensities
    Filtered 6155 of 6778 spots by peak-centroid distance
    Found 6155 bright spots
    Found max_cell: 495.9 Angstrom
    Setting d_min: 9.69
    FFT gridding: (256,256,256)
    Number of centroids used: 0
  ```

Though not an exhaustive list of the parameterisation of `cctbx.xfel.xtc_process`, the full list of available optiomns to be given is determined by passing the option `-c`. The provided list will be given in [PHIL file format](http://cctbx.sourceforge.net/libtbx_phil.html). To modify these values treat them hierarchically, where parents are pass directly, followed by `.` to access children, where the above command shows examples of using this format.

Following the output of the `cctbx.xfel.xtc_process` command above, a new geometric refinement of the detector positions for the CSPAD can be calculated using the command `cspad.cbf_metrology`.

## Refinement of detector parameters
