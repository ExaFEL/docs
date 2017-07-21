---
layout: "page"
title: "Building and using Docker images with Shifter"
permalink: docker
---

For reproducibility and to better take advantage of the Cori eco-system, cctbx is built within a [Docker](https://docker.com/) image. Docker allows for an operating system to be run within a container (similar to a virtual machine), and ensures that within this container the installed packages can have all dependencies and the environment correctly set. NERSC makes use of this containerised format by allowing Docker images to run on Cori with its [Shifter](http://www.nersc.gov/research-and-development/user-defined-images/) software. This allows users to define the container locally on their workstation, load the container to the cloud, whereby it can be pulled to Cori. On Cori the container can then be loaded onto a compute node, and process the data with the specified container environment snapshot.

## Creating a Docker container

To create a Docker (Shifter) container for use on Cori, the Docker daemon must first be [installed locally on your workstation](https://docs.docker.com/engine/installation/). 


## Loading the Docker container on Cori using Shifter
asdf


## Submitting batch commands using Shifter
asdf