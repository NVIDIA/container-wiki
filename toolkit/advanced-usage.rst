:tocdepth: 2

Advanced Usage
==============

.. contents:: Table of Contents
   :local:

General Topics
--------------

Backward compatibility
~~~~~~~~~~~~~~~~~~~~~~

To help transitioning code from 1.0 to 2.0, a bash script is provided in ``/usr/bin/nvidia-docker`` for backward compatibility.
It will automatically inject the ``--runtime=nvidia`` argument and convert ``NV_GPU`` to ``NVIDIA_VISIBLE_DEVICES``.

Existing ``daemon.json``
~~~~~~~~~~~~~~~~~~~~~~~~

If you have a custom ``/etc/docker/daemon.json``, the ``nvidia-docker2`` package will override it.
In this case, it is recommended to install `nvidia-container-runtime <https://github.com/nvidia/nvidia-container-runtime#installation>`__ instead and register the new runtime manually.

Default runtime
~~~~~~~~~~~~~~~

The default runtime used by the Docker® Engine is `runc <https://github.com/opencontainers/runc>`_, our runtime can become the default one by configuring the docker daemon with ``--default-runtime=nvidia``.
Doing so will remove the need to add the ``--runtime=nvidia`` argument to ``docker run``.
It is also the only way to have GPU access during ``docker build``.

Environment variables
~~~~~~~~~~~~~~~~~~~~~

The behavior of the runtime can be modified through environment variables (such as ``NVIDIA_VISIBLE_DEVICES``).
Those environment variables are consumed by `nvidia-container-runtime <https://github.com/nvidia/nvidia-container-runtime>`__ and are documented `here <https://github.com/nvidia/nvidia-container-runtime#environment-variables-oci-spec>`_.
Our official CUDA images use default values for these variables.

NVIDIA MPS
----------

Description
~~~~~~~~~~~

A container image is available for the `Multi-Process Service (MPS) <https://docs.nvidia.com/deploy/pdf/CUDA_Multi_Process_Service_Overview.pdf>`_ control daemon. 

**Only Volta MPS is supported.**

More information on Volta MPS can be found in the `Volta architecture whitepaper <http://images.nvidia.com/content/volta-architecture/pdf/volta-architecture-whitepaper.pdf>`_:

..

   Volta provides very high throughput and low latency for deep learning inference particular when 
   there is a batching system in place to aggregate images to submit 
   to the GPU simultaneously to 
   maximize performance. Without such a batching system, individual inference jobs do not fully 
   utilize execution resources of a GPU. Volta MPS provides an easy option to improve throughput 
   while satisfying latency targets, by permitting many individual inference jobs to be submitted 
   concurrently to the GPU and improving overall GPU utilization.


Requirements
~~~~~~~~~~~~


#. NVIDIA GPU with Architecture >= Volta (7.0)
#. A `supported version of Docker <https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#which-docker-packages-are-supported>`_.
#. The `NVIDIA Container Runtime for Docker <https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0>`_).

If you are using Docker Compose, it might further restrict the version of the Docker Engine you need.

Docker Compose
~~~~~~~~~~~~~~

You need a version of `Docker Compose <https://docs.docker.com/compose/>`_ that supports the Compose file format version ``2.3``.

A ``docker-compose.yml`` file is provided in the ``sample`` repository on GitLab:
https://gitlab.com/nvidia/samples/tree/master/mps

Example
^^^^^^^

.. code-block::

   $ git clone https://gitlab.com/nvidia/samples.git /tmp/samples
   $ cd /tmp/samples/mps
   $ export NVIDIA_VISIBLE_DEVICES=0
   $ export CUDA_MPS_ACTIVE_THREAD_PERCENTAGE=33 
   $ docker-compose up

Note: If you want the CUDA sample (here nbody) to run on multiple GPUs, you will need to edit the CLI arguments passed to the nbody executable.
e.g:

.. code-block::

   cat cuda-samples/Dockerfile
   FROM nvidia/cuda:9.0-base-ubuntu16.04

   RUN apt-get update && apt-get install -y --no-install-recommends \
           cuda-samples-$CUDA_PKG_VERSION && \
       rm -rf /var/lib/apt/lists/*

   WORKDIR /usr/local/cuda/samples/5_Simulations/nbody

   RUN make -j"$(nproc)"

   # Edit the numdevices option so that it can run on multiple devices
   CMD ["./nbody", "-benchmark", "-i=10000", "-numdevices=8"]

Details
^^^^^^^

To learn more about the implementation details of containerizing MPS, you can look at the comments in the ``docker-compose.yml`` file.

The following diagram summarizes the flow and the interactions for Docker Compose:

.. image:: https://user-images.githubusercontent.com/3645581/46986109-7ae66900-d0a2-11e8-93ba-c571aae2c9b2.png
   :target: https://user-images.githubusercontent.com/3645581/46986109-7ae66900-d0a2-11e8-93ba-c571aae2c9b2.png
   :alt: mps on docker-compose

Internals of the NVIDIA Container Toolkit
-----------------------------------------

Challenges
~~~~~~~~~~

In order to execute a GPU application on your machine, you need to have the NVIDIA driver installed. The NVIDIA driver is composed of multiple kernel modules:

.. code-block::

   $ lsmod | grep nvidia
   nvidia_uvm            711531  2 
   nvidia_modeset        742329  0 
   nvidia              10058469  80 nvidia_modeset,nvidia_uvm

It also provides a collection of user-level driver libraries and utility binaries (such as ``nvidia-smi`` and ``nvidia-modprobe`` that enable your application to communicate with the kernel modules and therefore the GPU devices:

.. code-block::

   $ ldconfig -p | grep -E 'nvidia|cuda'
   libnvidia-ml.so (libc6,x86-64) => /usr/lib/nvidia-361/libnvidia-ml.so
   libnvidia-glcore.so.361.48 (libc6,x86-64) => /usr/lib/nvidia-361/libnvidia-glcore.so.361.48
   libnvidia-compiler.so.361.48 (libc6,x86-64) => /usr/lib/nvidia-361/libnvidia-compiler.so.361.48
   libcuda.so (libc6,x86-64) => /usr/lib/x86_64-linux-gnu/libcuda.so
   ...

These libraries are tied to the driver version. This means that each driver version provides its own set of user-level libraries. Additionally note that these libraries are not backwards or forward compatible with the driver. This means you cannot use the libraries provided by driver 400.X with driver 500.X.

One of the early idea for containerizing GPU applications was to install the user-level driver libraries inside the container (for instance using option ``--no-kernel-module`` from the driver installer) . However,  the user-level driver libraries are tied to the version of the kernel module and all Docker containers share the host OS kernel. The version of the kernel modules had to match **exactly** (major and minor version) the version of the user-level libraries. Trying to run a container with a mismatched environment would immediately yield an error inside the container:

.. code-block::

   $ nvidia-smi 
   Failed to initialize NVML: Driver/library version mismatch

This approach made the images non-portable, making image sharing impossible and thus defeating of the main advantage of Docker.

The solution the NVIDIA Contaner Toolkit provides is to make the images agnostic of the driver version at build time. At runtime, the images are then specified by mounting the user-level libraries from the host using the `--volume <https://docs.docker.com/engine/reference/run/#volume-shared-filesystems>`_ argument of ``docker run``.

The NVIDIA driver supports multiple host OSes, there are multiple ways to install the driver (e.g. runfile or deb/rpm package) and the installer can also be customized. Across distributions, there is therefore no portable location for the driver files. The list of files to import can also depend on your driver version.

GPUs are exposed as separate device files in ``/dev``,  along with additional devices:

.. code-block::

   $ ls -l /dev/nvidia*
   crw-rw-rw- 1 root 195,   0 Apr 20 11:42 /dev/nvidia0
   crw-rw-rw- 1 root 195,   1 Apr 20 11:42 /dev/nvidia1
   crw-rw-rw- 1 root 195, 255 Apr 20 11:42 /dev/nvidiactl
   crw-rw-rw- 1 root 247,   0 Apr 20 11:42 /dev/nvidia-uvm

Docker allows users to whitelist access to specific devices (using `cgroups <https://www.kernel.org/doc/Documentation/cgroup-v1/devices.txt>`_) when starting a container by passing the argument ` ``--device`` <https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities>`_ to ``docker run``. On a multi-GPUs machine, a common use case is to launch multiple jobs in parallel, each one using a subset of the available GPUs. The most basic solution is to use the environment variable ``CUDA_VISIBLE_DEVICES``, but this doesn’t guarantee proper isolation. By using device whitelisting in Docker, you can restrict which GPUs a container will be able to access. For instance, container A is granted access to ``/dev/nvidia0`` while container B is granted access to ``/dev/nvidia1``. Devices ``/dev/nvidia-uvm`` and ``/dev/nvidiactl`` do not correspond to a GPU and they must be accessible for all containers.

The first challenge is to map the device files (or in other words, the minor number of the device) ordering to the PCI bus ordering (as reported by ``nvidia-smi``). This is important when you have different models of GPUs on your machine and you want to assign a container to one GPU in particular. The GPU numbering reported by ``nvidia-smi`` doesn’t always match the minor number of the device file:

.. code-block::

   $ nvidia-smi -q
   GPU 0000:05:00.0
    Minor Number: 3

   GPU 0000:06:00.0
    Minor Number: 2

The second challenge is related to the ``nvidia_uvm`` kernel module, it is not loaded automatically at boot time,  thus ``/dev/nvidia-uvm`` is not created and the container might have insufficient permission to load the kernel module itself. The kernel module must be manually loaded before starting any CUDA container.

nvidia-docker
~~~~~~~~~~~~~

GPUs are enumerated using function ``nvmlDeviceGetCount`` from the `NVML library <https://developer.nvidia.com/nvidia-management-library-nvml>`_ and the corresponding device minor is obtained with the function ``nvmlDeviceGetMinorNumber``. If the device minor number is N, the device file is simply /dev/nvidiaN.
Isolation is controlled using the environment variable ``NV_GPU``, by passing the indices of the GPUs to isolate, for instance:

.. code-block::

   $ NV_GPU=0,1 nvidia-docker run -ti nvidia/cuda nvidia-smi

The ``nvidia-docker`` wrapper will find the corresponding device files and add the ``--device`` arguments to the command-line before passing control to ``docker``.

To manually load ``nvidia_uvm`` and create ``/dev/nvidia-uvm``, we use the command ``nvidia-modprobe -u -c=0`` on the host when starting the ``nvidia-docker-plugin`` daemon.

Alternatives
~~~~~~~~~~~~

If you don’t want to use the ``nvidia-docker`` wrapper, you can add the command-line arguments manually:

.. code-block::

   $ docker run --device=/dev/nvidiactl --device=/dev/nvidia-uvm --device=/dev/nvidia0
   `

This needs to be used in combination with the command-line arguments for mounting the volume containing the the user-level driver libraries.
Listing the available GPUs can be done using ``nvmlDeviceGetCount`` from NVML or ``cudaGetDeviceCount`` from the CUDA API. We recommend using NVML since it also provides ``nvmlDeviceGetMinorNumber`` to find the device file to mount. Having a correct mapping between the device file and the isolated GPU is essential if you want to gather utilization metrics using NVML or nvidia-smi. If you still want to use the CUDA API, be sure to unset the environment variable ``CUDA_VISIBLE_DEVICES``, otherwise some GPUs on the system might not be listed.

To load ``nvidia_uvm``, you should also use ``nvidia-modprobe -u -c=0`` if available. If it’s not, you need to do the ``mknod`` `manually <http://docs.nvidia.com/cuda/cuda-getting-started-guide-for-linux/#runfile-verifications>`_.


