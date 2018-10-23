---
layout: page
title: Installing the psana2-cctbx toolchain
permalink: psana2-cctbx-install
---

Step by step installation instructions for getting a native build of CCTBX.XFEL with PSANA-2. Currently these steps have only been tested on Cori PII, a RHEL-7 type system.

## Installing the psana-cctbx.xfel toolchain

Firstly, we must ensure that the environment is setup before installing CCTBX.XFEL. The following commands assume a bash environment by default (alternative shells should also work, with alternative instructions given where necessary).

1. Install [Miniconda](https://conda.io/miniconda.html) for Python 2.7.
2. Once installed, source the conda installation and update the base environment:

    ```bash
     source ./miniconda/etc/profile.d/conda.sh
     conda update -y conda
     ```

3. Create a conda environment for the installation:

    ```bash
    conda create -n myEnv python=2.7
    ```

4. Activate the newly created conda environment:

    ```bash
    conda activate myEnv
    ```

5. Install psana from the LCLS-II conda channel

    ```bash
    conda install -c lcls-ii psana
    ```

6. Install the following additional dependencies for building and running psana and cctbx:

    ```
    conda install future h5py mpich2 wxpython pillow libtiff mock pytest jinja2 scikit-learn tabulate tqdm
    conda install --channel conda-forge orderedset
    python -m pip install procrunner
    ```
   
7. Create a cctbx.xfel directory, and acquire the bootstrap program for building and installation (`--no-check-certificate` can often be required):

    ```bash
    mkdir $PERM/cctbx.xfel; cd $PERM/cctbx.xfel
    wget https://raw.githubusercontent.com/cctbx/cctbx_project/master/libtbx/auto_build/bootstrap.py --no-check-certificate
    ```

8. Download and build the cctbx.xfel packages using the conda environment activated python.

    ```bash
    python bootstrap.py hot update --builder=dials
    ```

9. Get an interactive session on Cori KNL for this step. Assuming C++ compilers exist on the path, the following step will build the XFEL version of cctbx; specify the number of available cores to enable parallel compilation:

    ```bash
    python bootstrap.py build --builder=dials --with-python=`which python` --nproc=<# cores available for compile>
    ```

10. This step also needs an interactive session. To use the Cray MPI libraries properly on Cori PII, this step is necessary. It uninstalls the mpi4py that comes with conda, switches compilers to Cray and then installs mpi4py from scratch:

    ```bash
    conda uninstall mpi4py mpich2 --force
    module swap PrgEnv-intel PrgEnv-cray
    MPICC=cc pip install mpi4py
    ```

11. The path environment variables are set up by running the following command:

    ```bash
    source $PERM/cctbx.xfel/build/setpaths.sh #Bash users
    ```
    or

    ```bash
    source $PERM/cctbx.xfel/build/setpaths.csh #csh users
    ```



With the above steps the psana2-cctbx.xfel build should now be installed, with all accessible commands working.

