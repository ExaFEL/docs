---
layout: post
title: ExaFEL processing steps
---

--
# ExaFEL processing steps
--

To ensure the port of cctbx.xfel to Cori goes smoothly, it is necessary to identify and outline the relevant steps required to process data from LCLS.

As a quick guide, I will outline the steps required for this, and introduce example code where necessary.

## Installing the psana-cctbx.xfel toolchain

Firstly, we must ensure that the environment is correctly initialised to process the data. The following steps are a summary of those given for [psana](https://confluence.slac.stanford.edu/display/PSDM/Offsite+Installation#OffsiteInstallation-InstallationofaSingleCondaEnvironment), and for [cctbx.xfel](http://viper.lbl.gov/cctbx.xfel/index.php/Installation). It is encouraged to read the documentation at these locations for further information. The following commands assume a bash environment by default (alternative shells should also work, with alternative instructions given where necessary).

1. Install [Miniconda](https://conda.io/miniconda.html) for Python 2.7.
2. Once installed, update the conda installation: 

    ```bash
     conda update -y conda 
     ```
     
3. Create a conda environment for the installation:
 
    ```bash
    conda create -n myEnv 
    ```
    
4. Activate the newly created conda environment:
 
    ```bash
    source activate myEnv 
    ```
    
5. Install psana from the LCLS conda channel, replacing `X` with the appropriate RH Linux version {`5,6,7`}: 

    ```bash
    conda install -y --channel lcls-rhelX psana-conda
    ```
    
6. Install the following additional dependencies for building and running psana and cctbx:
    
    ```
    conda install h5py mpich2 wxpython pil libtiff
    ```
    
7. Create a permanent location to install and build cctbx (referred to as `$PERM` in the following steps): 
 
    ```bash
    export PERM=<my_dir>; mkdir $PERM
    ```
    
    
8. Copy the experiment name database from SLAC to your local system. This step assumes the end-user has a SLAC account, and can access *psexport.slac.stanford.edu*:

    ```bash
mkdir -p $PERM/psdm/data/ExpNameDb;
rsync -t psexport.slac.stanford.edu:/reg/g/psdm/data/ExpNameDb/experiment-db.dat $PERM/psdm/data/ExpNameDb/
```
9. Export the following psana environment variables:

    ```bash
    export SIT_DATA=$PERM/psdm/data
    export SIT_ROOT=$PERM/psdm/data
    export SIT_PSDM_DATA=$SIT_ROOT
    ```
10. Create a cctbx.xfel directory, and acquire the bootstrap program for building and installation (`--no-check-certificate` can often be required): 
 
    ```bash 
    mkdir $PERM/cctbx.xfel; cd $PERM/cctbf.xfel
    wget https://raw.githubusercontent.com/cctbx/cctbx_project/master/libtbx/auto_build/bootstrap.py --no-check-certificate
    ```

    
11. Download and build the cctbx.xfel packages using the conda environment activated python. This step assumes the end-user has access to the LBNL system **cci**. For the purpose of this guide we will currently focus only on LBNL developers.

    ```bash 
    python bootstrap.py hot update --builder=xfel --cciuser=<cciusername> --sfuser=<githubusername>
    ```

12. Assuming C++ compilers exist on the path, the following step will build the XFEL version of cctbx; specify the number of available cores to enable parallel compilation: 
 
    ```bash
    python bootstrap.py build --builder=xfel --with-python=`which python` --nproc=<# cores available for compile>
    ```
13. The path environment variables are set up by running the following command:
  
    ```bash
    source $PERM/cctbx.xfel/build/setpaths.sh #Bash users
    ``` 
    or
     
    ```bash
    source $PERM/cctbx.xfel/build/setpaths.csh #csh users
    ```

With the above steps the psana-cctbx.xfel build should now be installed, with all accessible commands working.


## Testing the psana-cctbx.xfel build
To test the previous bbuild steps, we will run through and document the processing of an example dataset.
