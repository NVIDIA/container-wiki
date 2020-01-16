Quickstart
==========

.. contents:: Table of Contents
   :local:

What is Docker?
---------------

From the `official Docker website <https://www.docker.com/what-docker>`_ :

..
   Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run: code, runtime, system tools, system libraries – anything that can be installed on a server. This guarantees that the software will always run the same, regardless of its environment.


More high-level information about Docker can be found on the `Moby project README <https://github.com/moby/moby/blob/master/README.md>`_. Technical documentation can be found `here <https://docs.docker.com/>`_.

Benefits of GPU containerization
--------------------------------

Containerizing GPU applications provides several benefits, among them:


* Ease of deployment
* Isolation of individual devices
* Run across heterogeneous driver/toolkit environments
* Requires only the NVIDIA driver to be installed on the host
* Facilitate collaboration: reproducible builds, reproducible performance, reproducible results.

Background of the NVIDIA Container Toolkit
------------------------------------------

Docker® containers are often used to seamlessly deploy CPU-based applications on multiple machines. With this use case, containers are both *hardware-agnostic* and *platform-agnostic*. This is obviously not the case when using NVIDIA GPUs since it is using specialized hardware and it requires the installation of the NVIDIA driver. As a result, Docker Engine does not natively support NVIDIA GPUs with containers.

To solve this problem, one of the early solutions that emerged was to fully reinstall the NVIDIA driver inside the container and then pass the character devices corresponding to the NVIDIA GPUs (e.g. ``/dev/nvidia0`` ) when starting the container. However, this solution was brittle: the version of the host driver had to exactly match driver version installed in the container. The Docker images could not be shared and had to be built locally on each machine, defeating one of the main advantages of Docker.

To make the Docker images portable while still leveraging NVIDIA GPUs, the container images must be agnostic of the NVIDIA driver. This repository provides utilities to enable GPU support inside the container runtime.

Prerequisites of the NVIDIA Container Toolkit
---------------------------------------------

The list of prerequisites for running the NVIDIA runtime is described below.
For information on how to install Docker for your Linux distribution, please refer to the `Docker documentation <https://docs.docker.com/engine/installation>`_.


* GNU/Linux x86_64 with kernel version > 3.10
* Docker >= 19.03
* NVIDIA GPU with Architecture > Fermi (2.1)
* `NVIDIA drivers <http://www.nvidia.com/object/unix.html>`_ ~= 361.93 (untested on older versions)

Your driver version might limit your CUDA capabilities (see `CUDA requirements <CUDA#requirements>`_ )

Installation of the NVIDIA Container Toolkit
--------------------------------------------

#. Ensure you are on a :ref:`supported distribution <linux-distribution-matrix>`
#. Install the ``nvidia-container-toolkit`` package:

   .. code-block::

     # Ubuntu/Debian Package Repositories
     $ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
     $ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
     $ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

     # RHEL / Centos Package Repositories
     $ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
     $ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo


     $ sudo apt-get install -y nvidia-container-toolkit
     $ sudo yum install -y nvidia-container-toolkit

Usage of the NVIDIA Container Toolkit
-------------------------------------

The NVIDIA runtime is integrated with the Docker CLI and GPUs can be accessed seamlessly by the container via the Docker CLI options. Some examples are shown below. 

.. code-block::

   # Starting a GPU enabled container
   $ docker run --gpus all nvidia/cuda nvidia-smi

   # Start a GPU enabled container on two GPUs
   $ docker run --gpus 2 nvidia/cuda nvidia-smi

   # Starting a GPU enabled container on specific GPUs
   $ docker run --gpus '"device=1,2"' nvidia/cuda:9.0-base nvidia-smi
   $ docker run --gpus '"device=UUID-ABCDEF,1"' nvidia/cuda:9.0-base nvidia-smi
