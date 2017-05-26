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
Prior to performing spotfinding, it is important to have an estimate of the detector gain. The gain for pixel at position $$x,y$$ is given as $$\gamma_{x,y}$$, and is temporally invariant for the experiment. We can estimate this from a sample of the XTC data streams by first exporting a collection of images from the stream for examination. This is performed with the command `cctbx.xfel.xtc_dump`. An example parameterisation of the call for this command is as follows:

  ```coffeescript
  cctbx.xfel.xtc_dump input.experiment=cxim1416 input.run_num=74 \
    input.address=CxiDs1.0:Cspad.0 format.file_format=cbf \
    format.cbf.detz_offset=585  \
    input.xtc_dir=/net/viper/raid1/mlxd_LM14/psdm/cxi/cxim1416/xtc \
    input.calib_dir=/net/viper/raid1/mlxd_LM14/psdm/cxi/cxim1416/calib \
    dispatch.max_events=100
  ```
where we have used many of the same parameter values from the calibration steps examined previously. The `dispatch.max_events=100` line ensures that only 100 images are output. To estimate the gain from these images, the command `dials.estimate_gain` is used. Looping over all possible CBF files is performed with:

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
   format.cbf.detz_offset=585 format.cbf.invalid_pixel_mask=mask.pickle \
   input.xtc_dir=/net/viper/raid1/mlxd_LM14/psdm/cxi/cxim1416/xtc \
   input.calib_dir=/net/viper/raid1/mlxd_LM14/psdm/cxi/cxim1416/calib \
   spotfinder.filter.min_spot_size=2 spotfinder.threshold.xds.gain=7 \
   spotfinder.threshold.xds.global_threshold=100 dispatch.integrate=False \
   output.dir="./" >> xtc_process.log &
  ```

For the above command we have specified many of the same parameter values as used with `cxi.mpi_average` previously. We have also chosen to pass some modifications to the parameters; to examine peaks with a spot size of at least 2 (`spotfinder.filter.min_spot_size=2`), using an estimated gain of 7 (`spotfinder.threshold.xds.gain=7`, determined from the previous section),  a minimum count value of 100 for being considered a background value (`spotfinder.threshold.xds.global_threshold=100`), and using the pixel mask determined previously (`format.cbf.invalid_pixel_mask=mask.pickle`). This command will iterate through the data for the given run number, and determine the spot positions, proceed to determine which peaks can contribute to the model, and then proceed to index them to construct a crystal model. The integration step is skipped for the above command (`dispatch.integrate=False`) as we intend to discovery and index for the initial steps only, allowing the process to run much faster. Sample print output during the processing of data for the above parameters is:

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
The print-statement output of `cctbx.xfel.xtc_process` can be followed by running `tail -f xtc_process.log`. Though not an exhaustive list of the parameterisation of `cctbx.xfel.xtc_process`, the full list of available options is available by passing the option `-c`. The provided list will be displayed in [PHIL file format](http://cctbx.sourceforge.net/libtbx_phil.html). To modify these values treat them hierarchically, where parents are passed directly to the command, followed by `.` to access and set children. The previous command shows examples of using this format. One thing to note is that we have not parameterised any element of the indexing or refinement stages of the process. This can be performed by appending some additional options to the end of the `cctbx.xfel.xtc_process` command above, with names and uses determined from the full list of options.

Upon completion, the command will generate output files into the specified directory (currently `./`). CBF images will be created for all indexed files, as well as files containing the indexing data from the run (`refine_experiments.json` and `indexed.pickle`).

## Common Mode correction
Where the gain was discussed to be constant across time for a given pixel, the common mode is a spatially invariant offset to a detector sensor for a given shot. To ensure use of the common mode corection algorithm the option `format.cbf.common_mode.algorithm=default` can be specified, or alternatively `custom` can be chosen. More details can be found in the CCN article. This section will be updated with changes to the models and with newer steps as they are determined. For the purposes of what is being discussed here, the `default` setting will be sufficient.


## Refinement of detector parameters
Assuming the above command ran successfully, an initial set of indexed data should be available. This can then be used to better refine the tile positions on the CSPAD detector, and can be calculated using the command `cspad.cbf_metrology`. Firstly, we can concatenate the indexing results from several different images into a single dataset (`dials.combine_experiments`), and can be run as

  ```bash
  dials.combine_experiments reference_from_experiment.average_detector=True \
   reference_from_experiment.average_hierarchy_level=0 \
   output.experiments_filename=cspad_combined_experiments.json \
   output.reflections_filename=cspad_combined_reflections.pickle \
   cspad_combine.phil
  ```

For the refinement, it is necessary to generate a PHIL file, containing all the essential parameters. Assuming the combination dataset is `cspad_combined_experiments.json` and `cspad_combined_reflections.pickle` as output from above, the refinement command can be run as:

  ```bash
  export TAG=<some postfix string>
  cspad.cbf_metrology tag=${TAG} ./ n_subset=2000 \
    split_dataset=True cxim1416-refine.phil rmsd_filter.enable=False
  ```

where `cxim1416-refine.phil` is a PHIL file containing the parameters for refinement. The options can be examined by passing the `-c` option, or by referring to the CCN article for further information. The resulting output file will be `0-end.data.${TAG}_1`, and can then be used as the new version of the metrology file. To ensure the new file is a better fit for the experimental observations, the commands `cspad.detector_shifts` and `cspad.detector_statistics` can be run as:

  ```bash
  cspad.detector_shifts ${TAG}_1_filtered_experiments.json \
   ${TAG}_1_filtered_reflections.pickle \
   ${TAG}_1_filtered_experiments_level2.json \
   ${TAG}_1_filtered_reflections_level2.pickle >> det_shift.log;
  cspad.detector_statistics tag=${TAG} >> det_stat.log;
  ```

The output of these two commands will hold information on the quality of the refinement, and can be used to determine if further refinement steps are required.

## Integration
With the successful refinement of the detector metrology using the previous steps, the data can now be integrated effectively, by specifying `dispatch.integrate=True` for `cctbx.xfel.xtc_process`. Additional output files will now be generator in the form of `int-0-<timestamp>.pickle`, which are the input arguments for the data merging and scaling commands `cxi.merge` and `prime.postrefine.` We will discuss the use of these commands [here](merge-scale.html).
