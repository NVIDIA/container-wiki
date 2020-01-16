:tocdepth: 2

Deprecated Features, Software and Images
========================================

.. contents:: Table of Contents
   :local:

.. _nvidia-docker-version-1-0:

Version 1.0
-----------

The list of prerequisites for running ``nvidia-docker`` is described below.
For information on how to install Docker for your Linux distribution, please refer to the `Docker documentation <https://docs.docker.com/engine/installation>`_.


#. GNU/Linux x86_64 with kernel version > 3.10
#. Docker >= 1.9 (official ``docker-engine`` , ``docker-ce`` or ``docker-ee`` only)
#. NVIDIA GPU with Architecture > Fermi (2.1)
#. `NVIDIA drivers <http://www.nvidia.com/object/unix.html>`_ >= 340.29 with binary ``nvidia-modprobe``

Your driver version might limit your CUDA capabilities (see `CUDA requirements <CUDA#requirements>`_ )

#. Install the repository for your distribution by following the instructions `here <http://nvidia.github.io/nvidia-docker/>`_.
#. Install the ``nvidia-docker`` package

.. code-block:: sh

  # For Ubuntu distributions
  sudo apt-get install nvidia-docker

  # For Centos distributions
  sudo yum install nvidia-docker

  # Basic usage
  nvidia-docker run --rm nvidia/cuda nvidia-smi

The package installation will automatically setup the ``nvidia-docker-plugin`` and depending on your distribution, register it with the init system.


Version 2.0
-----------

Quickstart
~~~~~~~~~~

The list of prerequisites for running nvidia-docker 2.0 is described below.
For information on how to install Docker for your Linux distribution, please refer to the `Docker documentation <https://docs.docker.com/engine/installation>`_.


#. GNU/Linux x86_64 with kernel version > 3.10
#. Docker >= 1.12
#. NVIDIA GPU with Architecture > Fermi (2.1)
#. `NVIDIA drivers <http://www.nvidia.com/object/unix.html>`_ ~= 361.93 (untested on older versions)

Note: Your driver version might limit your CUDA capabilities (see `CUDA requirements <CUDA#requirements>`_)

#. Install the repository for your distribution by following the instructions `here <http://nvidia.github.io/nvidia-docker/>`_.
#. Install the ``nvidia-docker2`` package and reload the Docker daemon configuration:

.. code-block:: sh

   # For Ubuntu distributions
   sudo apt-get install nvidia-docker2
   sudo pkill -SIGHUP dockerd

   # For Centos distributions
   sudo yum install nvidia-docker2
   sudo pkill -SIGHUP dockerd

nvidia-docker registers a new container runtime to the Docker daemon.
You must select the ``nvidia`` runtime when using ``docker run``:

.. code-block::

   docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi


Notes about Version 2.0
~~~~~~~~~~~~~~~~~~~~~~~

Version 1.0 of the nvidia-docker package must be cleanly removed before continuing.
You must stop and remove **all** containers started with nvidia-docker 1.0.

.. code-block:: sh

   docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f

   # For Ubuntu distributions
   sudo apt-get purge nvidia-docker

   # For Centos distributions
   sudo yum remove nvidia-docker

For older versions of Docker, you must pin the versions of both ``nvidia-docker2`` and ``nvidia-container-runtime`` when installing, for instance:

.. code-block::

   sudo apt-get install -y nvidia-docker2=2.0.1+docker1.12.6-1 nvidia-container-runtime=1.1.0+docker1.12.6-1

Use ``apt-cache madison nvidia-docker2 nvidia-container-runtime`` or ``yum search --showduplicates nvidia-docker2 nvidia-container-runtime`` to list the available versions.


.. _nvidia-caffe:

NVIDIA Caffe
------------

Docker image based on `NVIDIA/caffe <https://github.com/NVIDIA/caffe>`_, NVIDIA's fork of `BVLC/caffe <https://github.com/BVLC/caffe>`_.

See the requirements for `CUDA <CUDA#requirements>`_ since the Caffe image is based on a cuDNN runtime image.

This image can be used as a base image for other images like DIGITS, or it can be used directly to train networks:

.. code-block:: sh

   # Run a Caffe training job
   docker run --rm --runtime=nvidia nvidia/caffe caffe train --solver <args>

Check the `DockerHub <https://hub.docker.com/r/nvidia/caffe/>`__ for available tags.

.. _nvidia-digits:

NVIDIA DIGITS
-------------

Docker image based on `DIGITS <https://github.com/NVIDIA/DIGITS>`_.

See the requirements for `CUDA <CUDA#requirements>`_ since the DIGITS image is based on a cuDNN runtime image.

.. code-block:: sh

   # Run DIGITS on host port 5000
   docker run --runtime=nvidia --name digits -d -p 5000:5000 nvidia/digits

If you want to use a dataset stored in a host directory, you will need to import it inside the container using a volume:

.. code-block:: sh

   # Run DIGITS with the mnist dataset stored on the host under /opt/mnist
   docker run --runtime=nvidia --name digits -d -p 5000:5000 -v /opt/mnist:/data/mnist nvidia/digits

If you want to persist jobs across multiple DIGITS containers, you can use a named volume. For DIGITS 3.0:

.. code-block:: sh

   # Run DIGITS storing jobs in a host volume named digits-jobs

   # For DIGITS 3.0
   docker run --runtime=nvidia --name digits -d -p 5000:34448 -v digits-jobs:/usr/share/digits/digits/jobs nvidia/digits:3.0

   # For DIGITS 3.3 and 4.0
   docker run --runtime=nvidia --name digits -d -p 5000:34448 -v digits-jobs:/jobs nvidia/digits:4.0

   # For DIGITS 5.0
   docker run --runtime=nvidia --name digits -d -p 5000:5000 -v digits-jobs:/jobs nvidia/digits:5.0

   # For DIGITS 6.0
   docker run --runtime=nvidia --name digits -d -p 5000:5000 -p 6006:6006 -v digits-jobs:/jobs nvidia/digits:6.0


Check the `DockerHub <https://hub.docker.com/r/nvidia/digits/>`__ for available tags.
