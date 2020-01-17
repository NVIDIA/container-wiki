NVIDIA Container Runtime on Jetson (Beta)
=========================================

Introduction
------------

***Starting with v4.2.1, NVIDIA JetPack includes a beta version of NVIDIA Container Runtime with Docker integration for the Jetson platform. This enables users to run GPU accelerated Deep Learning and HPC containers on Jetson devices.**\ _

The NVIDIA runtime enables graphics and video processing applications such as DeepStream to be run in containers on the Jetson platform. The purpose of this document is to provide users with steps on getting started with running Docker containers on Jetson using the NVIDIA runtime. The beta supports Jetson AGX Xavier, Jetson TX2 series, Jetson TX1, and Jetson Nano devices. 

Installation
------------

NVIDIA Container Runtime with Docker integration (via the *nvidia-docker2* packages) is included as part of `NVIDIA JetPack <https://developer.nvidia.com/embedded/jetpack>`_. It is available for install via the `NVIDIA SDK Manager <https://docs.nvidia.com/sdk-manager/index.html>`_ along with other JetPack components as shown below in Figure 1. Note that the version of JetPack would vary depending on the version being installed.


.. image:: https://lh3.googleusercontent.com/_IrW289rk7TV-KjJNcxc8RZxoAyBjaoyjAxSBTTbYK97izactu5UhTgRsw3kFO8widR_Ze_R1UjgSqHpcenVL3rBB8y9qd5NkSb8Ciw6G4i3lMCzQ4HbTjpwhDclM7LWMp4I-c_9
   :target: https://lh3.googleusercontent.com/_IrW289rk7TV-KjJNcxc8RZxoAyBjaoyjAxSBTTbYK97izactu5UhTgRsw3kFO8widR_Ze_R1UjgSqHpcenVL3rBB8y9qd5NkSb8Ciw6G4i3lMCzQ4HbTjpwhDclM7LWMp4I-c_9
   :alt: 

*Figure 1: Jetpack Installation step 2*

After JetPack is installed to your Jetson device, you can check that the NVIDIA Container Runtime is installed by running the following commands:

.. code-block:: diff

   $ sudo dpkg --get-selections | grep nvidia
   libnvidia-container-tools           install
   libnvidia-container0:arm64          install
   nvidia-container-runtime            install
   nvidia-container-runtime-hook       install
   nvidia-docker2              install

   $ sudo docker info | grep nvidia
   + Runtimes: nvidia runc

If you don’t see the packages in the first command or if you don’t see the runtime head to the Troubleshooting section.

Hello-world!
------------

Once done with the installation process, let's go ahead and create a cool graphics application. Users have access to an L4T base container image from NGC for Jetson available `here <https://ngc.nvidia.com/catalog/containers/nvidia:l4t-base>`_.

Users can extend this base image to build their own containers for use on Jetson devices. In this example, we will run a simple N-body simulation using the CUDA nbody sample. Since this sample requires access to the X server, an additional step is required as shown below before running the container. 

.. code-block::

   # Allow containers to communicate with Xorg
   $ sudo xhost +si:localuser:root
   $ sudo docker run --runtime nvidia --network host -it -e DISPLAY=$DISPLAY -v /tmp/.X11-unix/:/tmp/.X11-unix nvcr.io/nvidia/l4t-base:r32.3.1

   root@nano:/# apt-get update && apt-get install -y --no-install-recommends make g++
   root@nano:/# cp -r /usr/local/cuda/samples /tmp
   root@nano:/# cd /tmp/samples/5_Simulations/nbody
   root@nano:/# make
   root@nano:/# ./nbody

You should see the following result:


.. image:: https://lh3.googleusercontent.com/i2W0kbAvSi-qqeD4VxK44gXH2N0svJz2GBM9cRFoLoDNuNtTV9ruYQv_EUFwZZQEI30xJyouxdkHYVFAkR8I7I23zN9JrHG9_tNnOnaqYsV3swTpjxPj2CcUBaAN0nLR2dFoE8Ht
   :target: https://lh3.googleusercontent.com/i2W0kbAvSi-qqeD4VxK44gXH2N0svJz2GBM9cRFoLoDNuNtTV9ruYQv_EUFwZZQEI30xJyouxdkHYVFAkR8I7I23zN9JrHG9_tNnOnaqYsV3swTpjxPj2CcUBaAN0nLR2dFoE8Ht
   :alt: 


Building CUDA in Containers on Jetson
-------------------------------------

Docker gives you the ability to build containers using the “docker build” command. Let's start with an example of how to do that on your Jetson device:

.. code-block:: shell

   $ mkdir /tmp/docker-build && cd /tmp/docker-build
   $ cp -r /usr/local/cuda/samples/ ./
   $ tee ./Dockerfile <<EOF
   FROM nvcr.io/nvidia/l4t-base:r32.3.1

   RUN apt-get update && apt-get install -y --no-install-recommends make g++
   COPY ./samples /tmp/samples

   WORKDIR /tmp/samples/1_Utilities/deviceQuery
   RUN make clean && make

   CMD ["./deviceQuery"]
   EOF

   $ sudo docker build -t devicequery .
   $ sudo docker run -it --runtime nvidia devicequery

   CUDA Device Query (Runtime API) version (CUDART static linking)

   Detected 1 CUDA Capable device(s)

   Device 0: "Xavier"
     CUDA Driver Version / Runtime Version          10.0 / 10.0
     CUDA Capability Major/Minor version number:    7.2
     Total amount of global memory:                 15692 MBytes (16454430720 bytes)
     ( 8) Multiprocessors, ( 64) CUDA Cores/MP:     512 CUDA Cores
     GPU Max Clock rate:                            1500 MHz (1.50 GHz)
     Memory Clock rate:                             1377 Mhz
     Memory Bus Width:                              256-bit
     L2 Cache Size:                                 524288 bytes
   ...

   deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 10.0, CUDA Runtime Version = 10.0, NumDevs = 1
   Result = PASS

Known limitation: The base l4t image doesn’t allow you to statically compile with all CUDA libraries. Only libcudadevrt.a and libcudart_static.a are included.

Enabling Jetson Containers on an x86 workstation (using qemu)
-------------------------------------------------------------

One of the very cool features that are now enabled is the ability to build Arm CUDA binaries on your x86 machine without needing a cross compiler.
You can very easily run AArch64 containers on your x86 workstation by using qemu’s virtualization features. This section will go over the steps to enable that. The next section will go over the workflow that allows you to build on x86 and then run on Jetson.
Installing the following packages should allow you to enable support for AArch64 containers on x86:

.. code-block:: shell

   $ sudo apt-get install qemu binfmt-support qemu-user-static

   # Check if the entries look good.
   $ sudo cat /proc/sys/fs/binfmt_misc/status
   enabled

   # See if /usr/bin/qemu-aarch64-static exists as one of the interpreters.
   $ cat /proc/sys/fs/binfmt_misc/qemu-aarch64
   enabled
   interpreter /usr/bin/qemu-aarch64-static
   flags: OCF
   offset 0
   magic 7f454c460201010000000000000000000200b700
   mask ffffffffffffff00fffffffffffffffffeffffff

Make sure the F flag is present, if not head to the troubleshooting section, as this will result in a failure to start the Jetson container.
You’ll usually find errors in the form: exec user process caused "exec format error"

Building Jetson Containers on an x86 workstation (using qemu)
-------------------------------------------------------------

Now that you have the required setup, we can get to building an Arm CUDA application on x86. Simply copy your code inside your container and run nvcc.

.. code-block::

   $ mkdir /tmp/docker-build && cd /tmp/docker-build
   $ cp -r /usr/local/cuda/samples/ ./

   $ tee ./Dockerfile <<EOF
   FROM nvcr.io/nvidia/l4t-base:r32.3.1

   RUN apt-get update && apt-get install -y --no-install-recommends make g++
   COPY ./samples /tmp/samples

   WORKDIR /tmp/samples/1_Utilities/deviceQuery
   RUN make clean && make

   CMD ["./deviceQuery"]
   EOF

   $ sudo docker build -t docker://USERNAME/devicequery .
   $ sudo docker push docker://USERNAME/devicequery

   # On your Jetson enabled device:
   $ sudo docker run -it --runtime nvidia docker://USERNAME/devicequery

   CUDA Device Query (Runtime API) version (CUDART static linking)

   Detected 1 CUDA Capable device(s)

   Device 0: "Xavier"
     CUDA Driver Version / Runtime Version          10.0 / 10.0
     CUDA Capability Major/Minor version number:    7.2
     Total amount of global memory:                 15692 MBytes (16454430720 bytes)
     Memory Bus Width:                              256-bit
     L2 Cache Size:                                 524288 bytes
   ...

   deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 10.0, CUDA Runtime Version = 10.0, NumDevs = 1
   Result = PASS

Alternatively you can get a shell running, mount your code inside the container and compile it.

.. code-block::

   $ sudo docker run -it -v /usr/local/cuda:/usr/local/cuda http://nvcr.io/nvidia/l4t-base:r32.3.1
   root@x86host:/# apt-get update && apt-get install -y --no-install-recommends make g++
   root@x86host:/# cp -r /usr/local/cuda/samples /tmp
   root@x86host:/# cd /tmp/samples/5_Simulations/nbody
   root@x86host:/# make

Known limitation: Unfortunately you won’t be able to run any binary that calls into the NVIDIA driver on the x86 host.

Troubleshooting
---------------

No package show are shown in dpkg’s output
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: diff

   $ sudo dpkg --get-selections | grep nvidia
   - libnvidia-container-tools         install
   - libnvidia-container0:arm64            install
   - nvidia-container-runtime          install
   - nvidia-container-runtime-hook         install
   - nvidia-docker2                install

You need to reinstall the NVIDIA Container Runtime for Docker using the JetPack process. Make sure that no errors are shown in the UI.

nvidia-docker2 package is missing from dpkg’s output
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You will need to manually register the runtime:

.. code-block:: diff

   $ sudo dpkg --get-selections | grep nvidia
   libnvidia-container-tools           install
   libnvidia-container0:arm64          install
   nvidia-container-runtime            install
   nvidia-container-runtime-hook           install
   - nvidia-docker2                install

   $ ls /etc/docker/daemon.json
   ls: cannot access '/etc/docker/daemon.json': No such file or directory

   $ sudo tee /etc/docker/daemon.json <<EOF
   {
       "runtimes": {
           "nvidia": {
               "path": "/usr/bin/nvidia-container-runtime",
               "runtimeArgs": []


   }
   EOF
   $ sudo pkill -SIGHUP dockerd

Docker info doesn’t show the NVIDIA runtime
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: diff

   $ sudo docker info | grep nvidia

   $ sudo pkill -SIGHUP dockerd
   $ sudo docker info | grep nvidia
   + Runtimes: nvidia runc

Generating and viewing logs
^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can enable logs by uncommenting the debug field in /etc/nvidia-container-runtime/config.toml

.. code-block:: shell

   $ cat /etc/nvidia-container-runtime/config.toml 
   disable-require = false
   #swarm-resource = "DOCKER_RESOURCE_GPU"

   [nvidia-container-cli]
   #root = "/run/nvidia/driver"
   #path = "/usr/bin/nvidia-container-cli"
   environment = ["PATH=/tmp"]
   debug = "/var/log/nvidia-container-runtime-hook.log"
   #ldcache = "/etc/ld.so.cache"
   load-kmods = true
   #no-cgroups = false
   #user = "root:video"
   ldconfig = "@/sbin/ldconfig.real"

/usr/local/cuda is readonly
^^^^^^^^^^^^^^^^^^^^^^^^^^^

One of the limitations of the beta is that we are mounting the cuda directory from the host. This was done with size in mind as a development CUDA container weighs 3GB, on Nano it’s not always possible to afford such a huge cost. We are currently working towards creating smaller CUDA containers.

Running or building a container on x86 (using qemu+binfmt_misc) is failing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The container does not start (hangs) or there is an error saying ‘exec format error’. Check if the interpreter is available to the containers.

.. code-block:: shell

   # See if /usr/bin/qemu-aarch64-static exists as one of the interpreters.
   $ cat /proc/sys/fs/binfmt_misc/qemu-aarch64
   enabled
   interpreter /usr/bin/qemu-aarch64-static
   flags: OCF
   offset 0
   magic 7f454c460201010000000000000000000200b700
   mask ffffffffffffff00fffffffffffffffffeffffff

If the flags does not include ‘F’ then the kernel is loading the interpreter lazily. The easiest fix is to have the binfmt-support package version >= 2.1.7, which automatically includes the --fix-binary (F) option. The other option is to run containers with /usr/bin/qemu-aarch64-static mounted inside the container:

.. code-block::

   # volume mount /usr/bin/qemu-aarch64-static
   docker run -it -v /usr/bin/qemu-aarch64-static:/usr/bin/qemu-aarch64-static -v /usr/local/cuda:/usr/local/cuda http://nvcr.io/nvidia/l4t-base:r32.3.1
   If running `docker build`; perhaps a better option is to use ‘podman’ (https://podman.io/) instead. Install podman on the system and run `podmand build` with `-v /usr/bin/qemu-aarch64-static:/usr/bin/qemu-aarch64-static`. Example:
   # volume mount /usr/bin/qemu-aarch64-static
   sudo podman build -v /usr/bin/qemu-aarch64-static:/usr/bin/qemu-aarch64-static -t <image_tag> .

Mount Plugins
^^^^^^^^^^^^^

The NVIDIA software stack, so that it can ultimately run GPU code, talks to the NVIDIA driver through a number of userland libraries (e.g: libcuda.so). Because the driver API is not stable, these libraries are shipped and installed by the NVIDIA driver.

In effect, what that means is that having a container which contains these libraries, ties it to the driver version it was built and ran against. Therefore moving that container to another machine becomes impossible. The approach we decided to take is to mount, at runtime, these libraries from your host filesystem into your container.

Internally the NVIDIA Container Runtime stack uses a plugin system to specify what files may be mounted from the host to the container. You can learn more about this system here: https://github.com/NVIDIA/libnvidia-container/blob/jetson/design/mount_plugins.md

Supported Devices
^^^^^^^^^^^^^^^^^

The nvidia container runtime exposes select device nodes from the host to container required to enable the following functionality within containers:


* frame buffer
* video decode (nvdec)
* video encode (msenc)
* color space conversion & scaling (vic)
* CUDA & TensorRT (through various nvhost devices)
* Deep learning accelerator (DLA)
* display (based on for eglsink, 3dsink, overlaysink)

Note that the decode, encode, vic and display functionality can be accessed from software using the associated gstreamer plugins available as part of the GStreamer version 1.0 based accelerated solution in L4T.

In terms of camera input, USB and CSI cameras are supported. In order to access cameras from inside the container, the user needs to mount the device node that gets dynamically created when a camera is plugged in – eg: /dev/video0. This can be accomplished using the --device option supported by docker as documented here: https://docs.docker.com/engine/reference/commandline/run/#add-host-device-to-container---device
