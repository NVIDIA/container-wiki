Setting up
----------

Which Docker packages are supported?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  All stable releases of ```docker-ce```_ installed from
   https://docs.docker.com/install/ starting from docker 19.03
-  The package provided by Canonical: ``docker.io`` starting from docker
   19.03.
-  The package provided by Red Hat: ``docker`` starting from docker
   19.03.

Note that Edge, Test and Nightly releases are not officially supported
but we will provide best effort support.

What is the minimum supported Docker version?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Docker 19.03 which adds support for the ``--gpus`` option.

How do I install the NVIDIA driver?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| The recommended way is to use your `package manager`_ and install the
  ``cuda-drivers`` package (or equivalent).
| When no packages are available, you should use an official
  `“runfile”`_.

| Alternatively, and as a technology preview, the NVIDIA driver can be
  deployed through a container.
| Refer to the `documentation`_ for more information.

I’m getting ``The following signatures were invalid: EXPKEYSIG`` while trying to install the packages, what do I do?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Make sure you fetched the latest GPG key from the repositories. Refer to
the `repository instructions`_ for your distribution.

Platform support
----------------

Do you support Jetson platforms (AArch64)?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Yes - beta support of the NVIDIA Container Runtime is now available on
Jetson platforms (AGX, TX2 and Nano). See `this link`_ for more
information on getting started.

Is macOS supported?
^^^^^^^^^^^^^^^^^^^

No, we do not support macOS (regardless of the version), however you can
use the native macOS Docker client to deploy your containers remotely
(refer to the `dockerd documentation`_).

Is Microsoft Windows supported?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

No, we do not support Microsoft Windows (regardless of the version),
however you can use the native Microsoft Windows Docker client to deploy
your containers remotely (refer to the `dockerd documentation`_).

Do you support Microsoft native container technologies (e.g. Windows server, Hyper-v)?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

No, we do not support native Microsoft container technologies.

Do you support Optimus (i.e. NVIDIA dGPU + Intel iGPU)?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Yes, from the CUDA perspective there is no difference as long as your
dGPU is powered-on and you are following the official driver
instructions.

What distributions are officially supported?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| For your host distribution, the list of supported platforms is
  available `here`_.
| For your container

.. _``docker-ce``: https://docs.docker.com/release-notes/docker-ce/
.. _package manager: http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#package-manager-installation
.. _“runfile”: http://www.nvidia.com/object/unix.html
.. _documentation: https://github.com/NVIDIA/nvidia-docker/wiki/Driver-containers-(EXPERIMENTAL)
.. _repository instructions: https://nvidia.github.io/nvidia-docker/
.. _this link: https://github.com/NVIDIA/nvidia-docker/wiki/NVIDIA-Container-Runtime-on-Jetson
.. _dockerd documentation: https://docs.docker.com/engine/reference/commandline/dockerd/#description
.. _here: http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#system-requirements
