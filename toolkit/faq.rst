Frequently Asked Questions
==========================

.. contents:: Table of Contents
   :local:

General Questions
-----------------

How do I install the NVIDIA driver?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The recommended way is to use your `package manager <http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#package-manager-installation>`_ and install the ``cuda-drivers`` package (or equivalent).
When no packages are available, you should use an official `"runfile" <http://www.nvidia.com/object/unix.html>`_.

Alternatively, and as a technology preview, the NVIDIA driver can be deployed through a container.

``The following signatures were invalid: EXPKEYSIG``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Make sure you fetched the latest GPG key from the repositories. Refer to the `repository instructions <https://nvidia.github.io/nvidia-docker/>`_ for your distribution.

Container Runtime
-----------------

Do you support OpenCL?
~~~~~~~~~~~~~~~~~~~~~~

Yes, we now provide images on `DockerHub <https://hub.docker.com/r/nvidia/opencl/>`_.

Do you support OpenGL?
~~~~~~~~~~~~~~~~~~~~~~

Yes, `EGL <https://devblogs.nvidia.com/parallelforall/egl-eye-opengl-visualization-without-x-server/>`_ is supported for headless rendering, but this is a **beta** feature. There is no plan to support GLX in the near future.

Images are available at `nvidia/opengl <https://hub.docker.com/r/nvidia/opengl/>`_. If you need CUDA+OpenGL, use `nvidia/cudagl <https://hub.docker.com/r/nvidia/cudagl/>`_.
If you are a `NGC <https://github.com/NVIDIA/nvidia-docker/wiki/NGC>`_ subscriber and require GLX for your workflow, please fill out a `feature request <https://devtalk.nvidia.com/default/board/221/feature-requests/>`_ for support consideration.

Do you support Vulkan?
~~~~~~~~~~~~~~~~~~~~~~

No, Vulkan is not supported at the moment. However we plan on supporting this feature in the future.

Do you support CUDA Multi Process Service (a.k.a. MPS)?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No, MPS is not supported at the moment. However we plan on supporting this feature in the future, and `this issue <https://github.com/NVIDIA/nvidia-docker/issues/419>`_ will be updated accordingly.

Do you support running a GPU-accelerated X server inside the container?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No, running a X server inside the container is not supported at the moment and there is no plan to support it in the near future (see also `OpenGL support <#is-opengl-supported>`_\ ).

Do you support sharing a GPU between multiple containers?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Yes. This is no different than sharing a GPU between multiple processes outside of containers.
Scheduling and compute preemption vary from one GPU architecture to another (e.g. CTA-level, instruction-level).

Do you support limiting the GPU resources (e.g. bandwidth, memory, CUDA cores) taken by a container?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No. Your only option is to set the GPU clocks at a lower frequency before starting the container.

Do you support enforcing exclusive access for a GPU?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is not currently supported but you can enforce it:

* At the container orchestration layer (Kubernetes, Swarm, Mesos, Slurm…) since this is tied to resource allocation.
* At the driver level by setting the `compute mode <http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#compute-modes>`_ of the GPU.

Does it have a performance impact on my GPU workload?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No, usually the impact should be in the order of less than 1% and hardly noticeable.
However be aware of the following (list non exhaustive):

* *GPU topology and CPU affinity* \
  You can query it using ``nvidia-smi topo`` and use `Docker CPU sets <https://docs.docker.com/engine/admin/resource_constraints/#cpu>`_ to pin CPU cores.
* *Compiling your code for your device architecture*\ \
  Your container might be compiled for the wrong achitecture and could fallback to the JIT compilation of PTX code (refer to the `official documentation <http://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html#gpu-compilation>`_ for more information).
  Note that you can express `these constraints <https://github.com/nvidia/nvidia-container-runtime#nvidia_require_>`_ in your container image.
* *Container I/O overhead*\ \
  By default Docker containers rely on an overlay filesystem and bridged/NATed networking.
  Depending on your workload this can be a bottleneck, we recommend using `Docker volumes <https://docs.docker.com/engine/admin/volumes/volumes/>`_ and experiment with different `Docker networks <https://docs.docker.com/engine/userguide/networking/>`_.
* *Linux kernel accounting and security overhead*\ \
  In rare cases, you may notice than some kernel subsystems induce overhead.
  This will likely depend on your kernel version and can include things like: cgroups, LSMs, seccomp filters, netfilter...

``unsatisfied condition: cuda >= X.Y``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Your CUDA container image is incompatible with your driver version.
Upgrade your driver or choose an `image tag <https://hub.docker.com/r/nvidia/cuda/>`_ which is supported by your driver (see also `CUDA requirements <https://github.com/NVIDIA/nvidia-docker/wiki/CUDA#requirements>`_\ )

I have multiple GPU devices, how can I isolate them between my containers?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

GPU isolation is achieved through the CLI option ``--gpus``. Devices can be referenced by index (following the PCI bus order) or by UUID.

e.g:

.. code-block::

   # If you have 4 GPUs, to isolate GPUs 3 and 4 (/dev/nvidia2 and /dev/nvidia3)
   $ docker run --gpus device=2,3 nvidia/cuda:9.0-base nvidia-smi

Why is ``nvidia-smi`` inside the container not listing the running processes?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``nvidia-smi`` and NVML are not compatible with `PID namespaces <http://man7.org/linux/man-pages/man7/pid_namespaces.7.html>`_.
We recommend monitoring your processes on the host or inside a container using ``--pid=host``.

The default behavior is an AND gate between host PID and GPU access:

=========  ============ ===============
Host PID   GPU Visible  Process Visible
=========  ============ ===============
Yes        Yes          Yes
No         Yes          No
Yes        No           No
No         No           No
=========  ============ ===============

Why is my container slow to start?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You probably need to enable `persistence mode <http://docs.nvidia.com/deploy/driver-persistence/index.html>`_ to keep the kernel modules loaded and the GPUs initialized.
The recommended way is to start the ``nvidia-persistenced`` daemon on your host.

Can I use it with Docker-in-Docker (a.k.a. DinD)?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are running a Docker client inside a container: simply mount the Docker socket and proceed as usual.
If you are running a Docker daemon inside a container: this case is untested.

Why is my application inside the container slow to initialize?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Your application was probably not compiled for the compute architecture of your GPU and thus the driver must `JIT <https://devblogs.nvidia.com/parallelforall/cuda-pro-tip-understand-fat-binaries-jit-caching/>`_ all the CUDA kernels from PTX.
In addition to a slow start, the JIT compiler might generate less efficient code than directly targeting your compute architecture (see also `performance impact <#does-it-have-a-performance-impact-on-my-gpu-workload>`_\ ).

Is the JIT cache shared between containers?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No. You would have to handle this manually with `Docker volumes <https://docs.docker.com/engine/admin/volumes/volumes/>`_.

What is causing the CUDA ``invalid device function`` error?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Your application was not compiled for the compute architecture of your GPU, and no PTX was generated during build time. Thus, JIT compiling is impossible (see also `slow to initialize <#why-is-my-application-inside-the-container-slow-to-initialize>`_\ ).

Why do I get ``Insufficient Permissions`` for some ``nvidia-smi`` operations?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Some device management operations require extra privileges (e.g. setting clocks frequency).
After learning about the security implications of doing so, you can add extra `capabilities <https://docs.docker.com/engine/security/security/#linux-kernel-capabilities>`_ to your container using ``--cap-add`` on the command-line (\ ``--cap-add=SYS_ADMIN`` will allow most operations).

Can I profile and debug my GPU code inside a container?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Yes but as stated above, you might need extra privileges, meaning extra `capabilities <https://docs.docker.com/engine/security/security/#linux-kernel-capabilities>`_ like ``CAP_SYS_PTRACE`` or tweak the `seccomp profile <https://docs.docker.com/engine/security/seccomp/>`_ used by Docker to allow certain syscalls.

Container images
----------------

What do I have to install in my container images?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Library dependencies vary from one application to another. In order to make things easier for developers, we provide a set of `official images <#do-you-provide-official-docker-images>`_ to base your images on.

Do you provide official Docker images?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Yes, container images are available on `Docker Hub <https://github.com/NVIDIA/nvidia-docker/wiki/Docker-Hub>`_ and on the `NGC registry <https://github.com/NVIDIA/nvidia-docker/wiki/NGC>`_.

Can I use the GPU during a container build (i.e. ``docker build``\ )?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Yes, as long as you `configure your Docker daemon <https://github.com/NVIDIA/nvidia-docker/wiki/Advanced-topics#default-runtime>`_ to use the ``nvidia`` runtime as the default, you will be able to have build-time GPU support. However, be aware that this can render your images non-portable (see also `invalid device function <#what-is-causing-the-cuda-invalid-device-function-error>`_\ ).

Are my container images built for version 1.0 compatible with 2.0 and 3.0?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Yes, for most cases. The main difference being that we don’t mount all driver libraries by default in 2.0 and 3.0. You might need to set the ``CUDA_DRIVER_CAPABILITIES`` environment variable in your Dockerfile or when starting the container. Check the documentation of `nvidia-container-runtime <https://github.com/nvidia/nvidia-container-runtime#environment-variables-oci-spec>`_.

How do I link against driver APIs at build time (e.g. ``libcuda.so`` or ``libnvidia-ml.so``\ )?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use the library stubs provided in ``/usr/local/cuda/lib64/stubs/``. Our official images already take care of setting `\ ``LIBRARY_PATH`` <https://gitlab.com/nvidia/cuda/blob/ubuntu16.04/9.0/devel/Dockerfile#L12>`_.
However, do not set ``LD_LIBRARY_PATH`` to this folder, the stubs must not be used at runtime.

The official CUDA images are too big, what do I do?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``devel`` `image tags <https://hub.docker.com/r/nvidia/cuda/>`_ are large since the CUDA toolkit ships with many libraries, a compiler and various command-line tools.
As a general rule of thumb, you shouldn’t ship your application with its build-time dependencies. We recommend to use `multi-stage builds <https://docs.docker.com/engine/userguide/eng-image/multistage-build/>`_ for this purpose. Your final container image should use our ``runtime`` or ``base`` images.
As of CUDA 9.0 we now ship a ``base`` `image tag <https://hub.docker.com/r/nvidia/cuda/>`_ which bundles the strict minimum of dependencies.

Why aren't CUDA 10 images working with nvidia-docker v1?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Starting from CUDA 10.0, the CUDA images require using nvidia-docker v2 and won't trigger the GPU enablement path from nvidia-docker v1.

Ecosystem enablement
--------------------

Do you support Docker Swarm mode?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Not currently, support for Swarmkit is still being worked on in the upstream Moby project. You can track our progress `on gitub here <https://github.com/moby/moby/issues/33439>`_.

Do you support Docker Compose?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Yes, use Compose format ``2.3`` and add ``runtime: nvidia`` to your GPU service. Docker Compose must be version `1.19.0 <https://github.com/docker/compose/releases/tag/1.19.0>`_ or higher. You can find an example `here <https://github.com/NVIDIA/gpu-monitoring-tools/blob/master/exporters/prometheus-dcgm/docker/docker-compose.yml>`_.
Note that you'll have to install the old ``nvidia-docker2`` packages.

Do you support Kubernetes?
~~~~~~~~~~~~~~~~~~~~~~~~~~

Since Kubernetes 1.8, the recommended way is to use our official `device plugin <https://github.com/NVIDIA/k8s-device-plugin>`_.
