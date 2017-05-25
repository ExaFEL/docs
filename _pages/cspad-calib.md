---
layout: page
title: Calibrating the CSPad detector with experimental data
permalink: cspad-calib
---

With the toolchain [built, installed](psana-cctbx-install.html) and [verified to work](build-test.html)
the next step is to work through the processing pipeline using some real data.

For the processing steps listed below, we will be making use of the guidelines given by this [CCN article](http://www.phenix-online.org/newsletter/CCN_2016_07.pdf#page=18). Additional information can be found on the [cctbx.xfel wiki](http://viper.lbl.gov/cctbx.xfel/index.php/Main_Page). As per the information in the newsletter article, the process of operating the cctbx.xfel pipeline involves three main stages:

    Calibration
    Discovery
    Batch processing

Further information on these stages is provided in the above resource locations.


# Step 1. Calibration

This step refers to the initialisation of the appropriate parameters for the detector used in the experiment. The alignment of the detector segments, determining which pixels are hot/cold (always bright, or always dark), calculating averages, and determining the detector gain are among the necessary steps for successfully analysing the data.

Here, we have assumed the `$PERM` environment variable is still set from the installation process outlined in post (install_post). For the purpose of this work, we are using [XTC](https://confluence.slac.stanford.edu/display/PSDMInternal/Pds+Xtc) data as collected from the [CSPAD](http://slac.stanford.edu/pubs/slacpubs/15250/slac-pub-15284.pdf) detector at [LCLS](https://confluence.slac.stanford.edu/display/PSDM/CSPAD+Alignment) (that is, raw data from the LCLS detectors). To be found by psana-cctbx.xfel, this data must be maintained in a specific directory hierarchy. Assuming a dataset named `cxim1416`, the following layout must be adhered to:

```bash
$PERM/psdm/CXI/cxim1416/xtc
```

Additionally, the calibration data for the detector must be also maintained in the same directory:

```bash
$PERM/psdm/CXI/cxim1416/calib
```
While the path to these directories can be specified and known by the given hierarchy and using the `SIT_*` environment variables, it is also possible to directly specify the paths to `cxi.mpi_average`<and potentially cxi.xtc_dump, cxi.xtc_process>. That is to say, we can negate the use of the environment variable `SIT_PSDM_DATA`, and pass the directories as arguments.

## Averaging

For this stage, we begin by analysing data from a dark run (i.e. where X-rays were not incident upon the target), as this can tell which pixels are trustworthy for further examinations. We perform this using the `cxi.mpi_average` command, as follows:

```bash
cxi.mpi_average --experiment=cxim1416 --run=54 \
--address=CxiDs1.0:Cspad.0 --detz-offset=585.5 \
--verbose -g 6.85 --xtc-dir=$PERM/psdm/cxi/cxim1416/xtc \
--calib_dir=$PERM/psdm/cxi/cxim1416/calib
```

where `--experiment=cxim1416` is the experiment name, `--run=54` is the specific run of the data to be analysed, `--address=CxiDs1.0:Cspad.0` specifies the detector type, `--detz-offset=585.5` is the Z-axis offset of the X-ray source from the detector, `-g 6.85` is the gain ratio betwen low and high gain detector regions, `--xtc-dir=$PERM/psdm/cxi/cxim1416/xtc` and `--calib_dir=$PERM/psdm/cxi/cxim1416/calib` are the previously mentioned data and calibration directory paths.

The above command generates average (`cxim1416_avg-r0054.cbf`), standard deviation (`cxim1416_stddev-r0054.cbf`), and composite maximum (`cxim1416_max-r0054.cbf`) output images over all events detected for the given run number.

The XTC file naming scheme is composed of a numerical identifier (`eXYZ-`), with mapping to name determined from `$PERM/psdm/data/ExpNameDb/experiment-db.dat`, followed by the run number (`rXYZW-`) indicating a specific instance of data collection. Given the throughput of data, there are multiple data acquisition (DAQ) systems working simulataneously on the detector, and each output their own data stream (`sXY-`). The recombination of this data is performed by psana, and abstracted away from cctbx.xfel users. Next, the chunk number is given, wherein the data is broken up into chunks of a given size. As the data is streamed from the detector, when a certain threshold is reached, the data can be broken into a new chunk (`cXY`). For the above command, the processing would be performed on the streams matching `e780-r0054-s[0-9][0-9]-c[0-9][0-9].xtc`.

## Pixel mask
From the previous step, we have created three images which can be used to determine which pixels can be trusted for the experimental data. These are based on the detected pixel intensities, measured in analogue-to-digital units (ADU), $$I_{jk}^{\xi}$$, where $$j,k$$ are pixel indices along the XY plane, and $$\xi \in \{\textrm{avg,std,max} \}$$ denotes the average, standard deviation, and maximum images respectively. For the average image data:

$$ \begin{align}
I_{jk}^{\textrm{avg}} &\leq 0 &&\rightarrow \textrm{dead pixel}  \\
I_{jk}^{\textrm{avg}} &\gt 2\times10^3 &&\rightarrow \textrm{hypersensitive pixel.}
    \end{align}
$$

Similarly for the standard deviation image,

$$ \begin{align}
I_{jk}^{\textrm{std}} &\leq 0 &&\rightarrow \textrm{dead pixel}  \\
I_{jk}^{\textrm{std}} &\gt 10 &&\rightarrow \textrm{hypersensitive pixel,}
    \end{align}
$$

and for the composite maximum,

$$ \begin{align}
I_{jk}^{\textrm{max}} &\lesssim 300 &&\rightarrow \textrm{dead pixel,}  \\
    \end{align}
$$

where the composite maximum has only a lower bounded value. Assuming a 1/0 Boolean flag for every pixel, and the above intensity ranges, we can define a mask for trusted and untrusted pixels, $$F_{jk}$$, as

$$ F_{jk} = \begin{cases}
1, & \textrm{if } (0 \leq I_{jk}^{\textrm{avg}} \leq 2\times10^3) \wedge (0 \leq I_{jk}^{\textrm{std}} \leq 10) \wedge (300 \leq I_{jk}^{\textrm{max}}) \\
0, & \textrm{otherwise}
\end{cases}.
$$

where $$ \wedge$$ denotes a logical AND operation. Performing an element-wise (Hadamard) product of the resulting mask values with the collected data allows only the values that are trusted to "sieve" through the mask and be used in further analysis.

### Creating the mask
To create the mask we make use of the averaged and standard deviation data for a dark run as discussed above, as well as a maximum image of a bright run we intend to apply the mask to. Assuming `--run=54` for a dark run, and `--run=74` for a bright run, we construct the mask as follows:

  ```bash
  cxi.make_dials_mask --maxproj-min=300 -o mask.pickle \
  cxim1416_avg-r0054.cbf cxim1416_stddev-r0054.cbf cxim1416_max-r0074.cbf
  ```

   The output file `mask.pickle` is in a python specific file format known as [pickle](http://viper.lbl.gov/cctbx.xfel/index.php/File_formats), for holding binary and metadata. We can set the lower threshold of the max limit manually (`--maxproj-min=300`), as it may require fine tuning to find the optimal value for the dataset. The resulting file maximum file `--run=74` can be visualised with the [DIALS](http://dials.lbl.gov/) image viewer as using the generated mask as

  ```bash
  dials.image_viewer cxim1416_max-r0074.cbf mask=mask.pickle
  ```

### Quadrant alignment
The CSPad detector features $$32\times 2$$ Si detectors, of which are distributed over four quadrants for the detector. It is necessary to determine the optimal alignment of these quadrants relative to one another, as it can vary across experiments due to the modularity of the detector. As such, one method to obtain and correct for the inter-quadrant spacings is through the use of powder diffraction rings. This assumes crystals of all possible orientations contribute to the diffraction process, and therefore generate rings at a radius determined by the beam wavelength. This is due to all possible Fourier components (Bragg peaks) being observed. By assuming that these rings are concentric over a $$ 2\pi$$ angular sweep, the quadrant offsets can be adjusted to optimise this annular condition.

This is checked with the command `cspad.quadrants_cbf`. Assuming the beam centre is appropriately positioned relative to the detector quadrants, a rotation of the quadrant through $$ \pi/4$$ about the beam center yield high correlation with the unrotated data. The above command performs a planar search for the offsets that optimise this correlation, and use these to determine the alignment of the quadrants. Following the correlation examination, the command outputs a new file with the corrected quadrant values. Assuming the composite maximum for a bright run (`--run=74`) the command is run as follows:

```bash
cspad.quadrants_cbf ./cxim1416_max-r0074.cbf
```

For this data set, the resulting output was

```bash
Doing cross-correlation on panel ARRAY_D0Q0S1A0
Searching a grid with dimensions (41, 41)
max cc  0.1589 is at (6, 1)
Doing cross-correlation on panel ARRAY_D0Q1S1A0
Searching a grid with dimensions (41, 41)
max cc  0.1611 is at (-14, -3)
Doing cross-correlation on panel ARRAY_D0Q2S1A0
Searching a grid with dimensions (41, 41)
max cc  0.1491 is at (-7, 11)
Doing cross-correlation on panel ARRAY_D0Q3S1A0
Searching a grid with dimensions (41, 41)
max cc  0.1794 is at (-4, 11)
Average CC:  0.1621
Saving result to ./cxim1416_max-r0074_cc.cbf
```
indicating very low correlations overall ($$\textrm{CC} \approx 16\%$$). To better align the quadrants, we should chose a data set that offers a much higher cross correlation. One such method for this is to examine all possible maxima for a dataset and to use that with highest correlation values for the quadrant alignment. Alternatively, we can generate a maxima of the maxima, and use this to calculate the cross-correlation with the following command:

```bash
cxi.cspad_average *_max_*.cbf -m all_max.cbf
cspad.quadrants_cbf all_max.cbf
```
with the resulting output file named `all_max_cc.cbf`. The refined alignment data will be stored in this CBF file, and must be converted to the appropriate SLAC file format before use. The specific file format is unique to SLAC, with full details on the CSPad detector geometry, alignment and associated file format information found [here](https://confluence.slac.stanford.edu/display/PSDM/Detector+Geometry) and [here](https://confluence.slac.stanford.edu/display/PSDM/CSPAD+Geometry+and+Alignment). This conversion is performed by running the command

```bash
cxi.cbfheader2slaccalib cbf_header=all_max_cc.cbf
```
with the command generating `0-end.data` as an output file. The file must then be placed into the `calib` folder in the following location:

```bash
mv $PERM/psdm/cxi/<experiment name>/calib/<detector type>/<detector name>/geometry/0-end.data $PERM/psdm/cxi/<experiment name>/calib/<detector type>/<detector name>/geometry/0-end.data.v0;
mv ./0-end.data $PERM/psdm/cxi/<experiment name>/calib/<detector type>/<detector name>/geometry/0-end.data.v1
```

Note that due to the presence of the previous `0-end.data` geometry calibration file, we renamed this to indicate it was version 0, and moved the refined file to the location with post-fix for version 1. This allows for a history of the files to be made, and is performed also for subsequent refinements. For the `cxim1416` dataset used previously, the above command becomes

```bash
mv $PERM/psdm/cxi/cxim1416/calib/CsPad::CalibV1/CxiDs1.0:Cspad.0/geometry/0-end.data $PERM/psdm/cxi/cxim1416/calib/CsPad::CalibV1/CxiDs1.0:Cspad.0/geometry/0-end.data.v0;
mv ./0-end.data $PERM/psdm/cxi/cxim1416/calib/CsPad::CalibV1/CxiDs1.0:Cspad.0/geometry/0-end.data.v1
ln -sf $PERM/psdm/cxi/cxim1416/calib/CsPad::CalibV1/CxiDs1.0:Cspad.0/geometry/0-end.data.v1 $PERM/psdm/cxi/cxim1416/calib/CsPad::CalibV1/CxiDs1.0:Cspad.0/geometry/0-end.data
```
where the updated file was soft-linked to the `0-end.data` naming convention. This procedure is repeated for future refinements. Next, we will focus on finding [Bragg peaks, indexing, and refinement](disc-refine.html).

<!---
---

Note:
  The CCN newsletter article mentions performing manual calibration using the LCLS tool `calibman`, which is installed as part of psana. Running the calibman command line interface requires having the MySQL package available for python. This is installed with conda as

  ```bash
conda install mysql-python
  ```
  However, note that calibman requires a user to be within the LCLS psana environment, and currently does not work on external systems.
  -->
