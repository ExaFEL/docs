---
layout: page
title: Installing the psana-cctbx toolchain
permalink: psana-cctbx-install
---

To ensure the port of cctbx.xfel to Cori goes smoothly, it is necessary to identify and outline the relevant steps required to process data from LCLS.

As a quick guide, I will outline the steps required for this, and introduce example code where necessary.

## Installing the psana-cctbx.xfel toolchain

Firstly, we must ensure that the environment is correctly initialised to process the data. The following steps are a summary of those given for [psana](https://confluence.slac.stanford.edu/display/PSDM/Offsite+Installation#OffsiteInstallation-InstallationofaSingleCondaEnvironment), and for [cctbx.xfel](http://viper.lbl.gov/cctbx.xfel/index.php/Installation). It is encouraged to read the documentation at these locations for further information. The following commands assume a bash environment by default (alternative shells should also work, with alternative instructions given where necessary).

1. Install [Miniconda](https://conda.io/miniconda.html) for Python 2.7.
2. Once installed, source the conda installation and update the base environment:

    ```bash
     source ./miniconda/etc/profile.d/conda.sh
     conda update -y conda
     ```

3. Create a conda environment for the installation:

    ```bash
    conda create -n myEnv
    ```

4. Activate the newly created conda environment:

    ```bash
    conda activate myEnv
    ```

5. Install psana from the LCLS conda channel, replacing `X` with the appropriate RH Linux version {`5,6,7`}:

    ```bash
    conda install -y --channel lcls-rhelX psana-conda
    ```

6. Install the following additional dependencies for building and running psana and cctbx:

    ```
    conda install future h5py mpich2 wxpython pillow libtiff mock pytest jinja2 scikit-learn tabulate
    conda install --channel conda-forge orderedset
    python -m pip install procrunner
    ```

7. Create a permanent location to install and build cctbx (referred to as `$PERM` in the following steps):

    ```bash
    export PERM=<my_dir>; mkdir $PERM
    ```


8. Copy the experiment name database from SLAC to your local system. This step assumes the end-user has a SLAC account, and can access *psexport.slac.stanford.edu*. A public facing database will be included with the psana conda installation at a later date.

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
    mkdir $PERM/cctbx.xfel; cd $PERM/cctbx.xfel
    wget https://raw.githubusercontent.com/cctbx/cctbx_project/master/libtbx/auto_build/bootstrap.py --no-check-certificate
    ```

11. Download and build the cctbx.xfel packages using the conda environment activated python.

    ```bash
    python bootstrap.py hot --builder=dials
    python bootstrap.py update --builder=dials
    ```

12. Assuming C++ compilers exist on the path, the following step will build the XFEL version of cctbx; specify the number of available cores to enable parallel compilation:

    ```bash
    python bootstrap.py build --builder=dials --with-python=`which python` --nproc=<# cores available for compile>
    ```
13. The path environment variables are set up by running the following command:

    ```bash
    source $PERM/cctbx.xfel/build/setpaths.sh #Bash users
    ```
    or

    ```bash
    source $PERM/cctbx.xfel/build/setpaths.csh #csh users
    ```

With the above steps the psana-cctbx.xfel build should now be installed, with all accessible commands working. A script for the above commands can be found [here](https://raw.githubusercontent.com/ExaFEL/exafel_project/master/bin/install.sh).
