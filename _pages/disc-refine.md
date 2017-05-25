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

## Gain estimation
Prior to performing spotfinding, it is important to have an estimate of the detector gain. We can estimate this from a sample of the XTC data streams by first exporting a collection of image from the stream for examination. This is performed with the command `cctbx.xfel.xtc_dump`. An example paramterisation of the call for this command is as follows:

  ```coffeescript
  cctbx.xfel.xtc_dump input.experiment=cxim1416 input.run_num=74 \
    input.address=input.address=CxiDs1.0:Cspad.0 format.file_format=cbf \
    format.cbf.detz_offset=585 \
    input.xtc_dir=/net/viper/raid1/mlxd_LM14/psdm/cxi/cxim1416/xtc \
    input.calib_dir=/net/viper/raid1/mlxd_LM14/psdm/cxi/cxim1416/calib \
    dispatch.max_events=100
  ```
where we have used many of the same parameter values from the calibration steps examined previously. The `dispatch.max_events=100` line ensures that only 100 images are output. To estimate the gain from these images, the command `dials.estimate_gain` is used. Lookping over all possible CBF files is performed with:

  ```bash
    for img in $(ls *.cbf); do
      echo ${img} >> gain.text
      dials.estimate_gain img >> gain.txt
      echo "" >> gain.txt
    done
  ```
where the resulting output will be placed into `gain.txt`. Examining the output will allow for an estimated gain for the following section.

## Bragg spot discovery and indexing
For this stage in the process, we must parameterise the spotfinding process to ensure it finds the best Bragg peaks to determine a crystal model for further refinement and analysis. For this we can use the <!---[DIALS](http://dials.lbl.gov/) spot-finding algorithm -->
cctbx.xfel program `cctbx.xfel.xtc_process`, which will use the [DIALS](http://dials.lbl.gov/) back-end for performing spot-finding and integration. By providing appropriate parameters to `cctbx.xfel.xtc_process` we can discover a sufficient number of Bragg peaks to index and determine a model crystal structure. As for the gain estimation example above, a sample command for dataset `cxim1416` using run number 76 is:

  ```coffeescript
  cctbx.xfel.xtc_process input.experiment=cxim1416 input.run_num=74 \
   input.address=CxiDs1.0:Cspad.0 format.file_format=cbf \
   format.cbf.detz_offset=585 \
   input.xtc_dir=/net/viper/raid1/mlxd_LM14/psdm/cxi/cxim1416/xtc \
   input.calib_dir=/net/viper/raid1/mlxd_LM14/psdm/cxi/cxim1416/calib \
   spotfinder.filter.min_spot_size=2 spotfinder.threshold.xds.gain=7 \
  spotfinder.threshold.xds.global_threshold=100
  ```
For the above command we have specified many of the same parameter values as used with `cxi.mpi_average` previously. We have also chosen to examine peaks with a spot isze of at least 2, using an estimated gain of 7 (determined from the previous section), and a minimum count value of 100 for being considered a backgroundf value. This command will iterate through the data for the given run number, and determine the spot positions, proceed to determine which peaks can contribute to the model, and then proceed to index them to construct a crystal model. Sample output during the processing of data for the above parameters is:

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

Though not an exhaustive list of the parameterisation of `cctbx.xfel.xtc_process`, the full list of available options is available by passing the option `-c`. The provided list will be given in [PHIL file format](http://cctbx.sourceforge.net/libtbx_phil.html). To modify these values treat them hierarchically, where parents are passed directly to the command, followed by `.` to access and set children, where the above command shows examples of using this format. One thing to note is that we have not parameterised any element of the indexing or refinement stages of the process. This can be performed by appending some additional options to the end of the `cctbx.xfel.xtc_process` command abiove,  with names and uses determined from the full list of options.

Following the output of the `cctbx.xfel.xtc_process` command above, a new geometric refinement of the detector positions for the CSPAD can be calculated using the command `cspad.cbf_metrology`.

## Refinement of detector parameters
