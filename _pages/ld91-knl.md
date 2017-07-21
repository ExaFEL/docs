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
    <path to experiment name database>/psdm/:/reg/g;" 
    ```
Alternatively, to load a container with a dynamically mountable **modules** directory, use
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
The above commands will however only run on a single core. This is because we have not yet parameterised the MPI environment for Shifter. To do this, we can create an **avg.sh** file which contains all our necessary commands and use all available KNL cores using `srun -n 68`. The file **avg.sh** will be defined as:
    ```bash
    #!/bin/bash
    ### avg.sh ###
    export SIT_DATA=/reg/g/psdm/data;
    export SIT_PSDM_DATA=/reg/d/psdm;
    export SIT_ROOT=/reg/g/psdm/data;
    source /build/setpaths.sh
    cxi.mpi_average -x cxid9114 -r ${1} -a CxiDs2.0:Cspad.0 -d 572 -v -g 6.85
    ```
which will be called using `srun` as
    ```bash
    srun -n 68 -c 1 --cpu_bind=cores shifter ./avg.sh <run_number>
    ```
Here we can parameterise which run number to process.

4. Make a mask using the averaged data
    ```bash
    #!/bin/bash 
    ### make_mask.sh ###
    export SIT_DATA=/reg/g/psdm/data;
    export SIT_PSDM_DATA=/reg/d/psdm;
    export SIT_ROOT=/reg/g/psdm/data;
    source /build/setpaths.sh
    cxi.make_dials_mask --maxproj-min=50 -o mask.pickle ../cxid9114_avg-r0089.cbf ../cxid9114_stddev-r0089.cbf ../cxid9114_max-r0096.cbf
    ```
    which again can be called using `srun` as
    ```bash
    srun -n 68 -c 1 --cpu_bind=cores shifter ./avg.sh <run_number>
    ```
5. With the detector masks created we can now perform spotfinding and indexing on the data. For this we must define a PHIL file format to parameterise the spotfinding and indexing steps. To perform these steps for an initial spotfinding and indexing run of LD91 we used the PHIL file **process.phil** given by:
    ```xml
    mp{ #Parameterise the multiprocessing method to use MPI and a client-server submission model
        method=mpi
        mpi{
            method=client_server
        }
    }
    dispatch{ #Turn off the integration steps and CBF output in initial indexing for metrology
        integrate=False
        dump_indexed=False
    }
    input{ #Experiment details
        address=CxiDs2.0:Cspad.0
        experiment=cxid9114
        run_num=113
    }
    format { #Output format details and specifics for LD91
        file_format=cbf
        cbf{
            detz_offset=572.3938
            invalid_pixel_mask=mask.pickle
            override_energy=8950
            cspad{
                gain_mask_value=6.85
                common_mode.algorithm=custom
                common_mode.custom_parameterization=5,50
            }
        }
    }
    border_mask {
        border=1
    }
    spotfinder { #Sptofinding parameters
        filter.min_spot_size=2
        threshold.xds.gain=25
        threshold.xds.global_threshold=100
    }
    indexing{ #Indexing parameters
        known_symmetry {
                space_group = P43212
                unit_cell = 78.9 78.9 38.1 90 90 90
        }
        method=real_space_grid_search
        refinement_protocol.d_min_start=1.7
    }
    refinement { #Refinement parameters; currently disabled
        parameterisation{
                beam.fix=all
            detector.fix_list=Dist,Tau1
            auto_reduction{
                action=fix
                min_nref_per_parameter=1
            }
            crystal {
                    unit_cell {
                        restraints {
                            tie_to_target{
                                values=78.9,78.9,38.1,90,90,90
                            sigmas=1,1,1,0,0,0
                        }
                    }
                }
            }
        }
    }
    integration{ #Integration parameters; currently disabled
        integrator=stills
        profile.fitting=False
        background {
            algorithm=simple
            simple {
                model.algorithm = linear2d
                outlier.algorithm = plane
            }
        }
    }
    profile {
        gaussian_rs {
            min_spots.overall = 0
        }
    }
    ```
This can be submitted to `cctbx.xfel.xtc_process` in the format
    ```bash
    srun -n 68 -c 1 --cpu_bind=cores shifter ./process.sh $PWD $PWD/process.phil
    ```
where **process.sh** is defined as
    ```bash
    #!/bin/bash
    export SIT_DATA=/reg/g/psdm/data;
    export SIT_PSDM_DATA=/reg/d/psdm;
    export SIT_ROOT=/reg/g/psdm/data;
    source /build/setpaths.sh

    cd $1
    cctbx.xfel.xtc_process $2
    ```
The script will change into the current working directory, run the shifter shifter container with **process.sh** which takes **process.phil** as an argument.




---
To check the performance of the code run the `grep start *.txt | wc -l` in the `rXYZ/<tag>/out/debug` directory. This will list the number of files processed at any instance of time. If the total number of events are known, an estimate of the overall time required can be estimated by taking the change in event-number of time.