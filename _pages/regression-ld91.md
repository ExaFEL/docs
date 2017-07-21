---
layout: "page"
title: "Analysing the LD91 data using Cori"
permalink: ld91-knl
---

Using the Shifter (Docker) container created [previously](docker)




## Data processing on Cori
For the purposes of an example, we will load the Shifter container onto a KNL load, set the required paths and process some images from averaging, spotfinding, indexing, and finally metrology. We will outline all the necessary commands and steps, and include batch job submission where necessary.

1. Acquire a Cori KNL node, and load the respective Docker container to be accessible on the node

```bash
salloc -N 1 -p debug --image=docker:mlxd/xfel:latest -t 00:30:00 -C knl -A <repo name> \
--volume="<path to data>/cxi:/reg/d/psdm/CXI; \
<path to data>/cxi:/reg/d/psdm/cxi; \
<path to experiment name database>/psdm/:/reg/g; \
```
- Alternatively, to load a container with a dynamically mountable **modules** directory, use

```bash
salloc -N 1 -p debug --image=docker:mlxd/xfel:latest_modules -t 00:30:00 -C knl -A lcls \
--volume="<path to data>/cxi:/reg/d/psdm/CXI; \
<path to data>/cxi:/reg/d/psdm/cxi; \
<path to experiment name database>/psdm/:/reg/g; \
<path to modules>/modules:/modules"
``` 

2. Load the Shifter container and run a terminal, setting the required paths for cctbx.xfel
```bash
shifter /bin/bash
source /build/setpaths.sh
```

3. Perform averaging of the data using bright and dark runs (dark=run89 for LD91)
```bash
cxi.mpi_average -x cxid9114 -r 89 -a CxiDs2.0:Cspad.0 -d 572 -v -g 6.85 -R # Dark
cxi.mpi_average -x cxid9114 -r 96 -a CxiDs2.0:Cspad.0 -d 572 -v -g 6.85 # Bright
```
The above command will however only run on a single core. This is because we have not yet parameterised the MPI environment for Shifter. To do this, we can create an **average.sh** file which contains all our necessary commands and use all available KNL cores using `srun -n 68`. The command will be as follows:

```bash

```

4. Make a mask using the averaged data
```bash
#!/bin/bash
export SIT_DATA=/reg/g/psdm/data;
export SIT_PSDM_DATA=/reg/d/psdm;
export SIT_ROOT=/reg/g/psdm/data;
source /build/setpaths.sh
cxi.make_dials_mask --maxproj-min=50 -o mask.pickle ../cxid9114_avg-r0089.cbf ../cxid9114_stddev-r0089.cbf ../cxid9114_max-r0096.cbf
```