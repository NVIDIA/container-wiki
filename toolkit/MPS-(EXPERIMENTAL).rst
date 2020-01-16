.. role:: raw-html-m2r(raw)
   :format: html


**Note that the features described below are currently experimental**.

Description
-----------

A container image is available for the `Multi-Process Service (MPS) <https://docs.nvidia.com/deploy/pdf/CUDA_Multi_Process_Service_Overview.pdf>`_ control daemon. 

**Only Volta MPS is supported.**

More information on Volta MPS can be found in the `Volta architecture whitepaper <http://images.nvidia.com/content/volta-architecture/pdf/volta-architecture-whitepaper.pdf>`_\ :

..

   Volta provides very high throughput and low latency for deep learning inference particular when 
   there is a batching system in place to aggregate images to submit 
   to the GPU simultaneously to 
   maximize performance. Without such a batching system, individual inference jobs do not fully 
   utilize execution resources of a GPU. Volta MPS provides an easy option to improve throughput 
   while satisfying latency targets, by permitting many individual inference jobs to be submitted 
   concurrently to the GPU and improving overall GPU utilization.


Requirements
------------


#. NVIDIA GPU with Architecture >= Volta (7.0)
#. A `supported version of Docker <https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#which-docker-packages-are-supported>`_.
#. The `NVIDIA Container Runtime for Docker <https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0>`_\ ).

If you are using Docker Compose, it might further restrict the version of the Docker Engine you need.

Docker Compose
--------------

You need a version of `Docker Compose <https://docs.docker.com/compose/>`_ that supports the Compose file format version ``2.3``.

A ``docker-compose.yml`` file is provided in the ``sample`` repository on GitLab:\ :raw-html-m2r:`<br>`
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

