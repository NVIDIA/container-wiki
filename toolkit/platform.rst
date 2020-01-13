Platform support Information
============================

.. contents:: Table of Contents
   :local:

Linux Distribution Matrix
-------------------------

The following table describes the Linux Distribution matrix of the ``nvidia-container-toolkit`` package.

+----------------+-------------------------+--------+---------+
|  Distribution  |      Support Status     | x86_64 | ppc64le |
+================+=========================+========+=========+
| Ubuntu 14.04   |  Not Supported anymore  |   N/A  |   No    |
+----------------+-------------------------+--------+---------+
| Debian 8       |  Not Supported anymore  |   N/A  |   No    |
+----------------+-------------------------+--------+---------+
| Ubuntu 19.04   |        Best Effort      |   Yes  |   Yes   |
+----------------+-------------------------+--------+---------+
| Ubuntu 19.10   |        Best Effort      |   Yes  |   Yes   |
+----------------+-------------------------+--------+---------+
| Ubuntu 16.04   |            Yes          |   Yes  |   Yes   |
+----------------+-------------------------+--------+---------+
| Ubuntu 18.04   |            Yes          |   Yes  |   Yes   |
+----------------+-------------------------+--------+---------+
| Debian 9       |            Yes          |   Yes  |   No    |
+----------------+-------------------------+--------+---------+
| Debian 10      |            Yes          |   Yes  |   No    |
+----------------+-------------------------+--------+---------+
| Amazon Linux 1 |            Yes          |   Yes  |   No    |
+----------------+-------------------------+--------+---------+
| Amazon Linux 2 |            Yes          |   Yes  |   No    |
+----------------+-------------------------+--------+---------+
| Centos 7.X     |            Yes          |   Yes  |   Yes   |
+----------------+-------------------------+--------+---------+
| Centos 8.X     |            Yes          |   Yes  |   Yes   |
+----------------+-------------------------+--------+---------+
| RHEL 7.X       |            Yes          |   Yes  |   Yes   |
+----------------+-------------------------+--------+---------+
| RHEL 8.X       |            Yes          |   Yes  |   Yes   |
+----------------+-------------------------+--------+---------+
| SLES 15.X      |            Yes          |   Yes  |   No    |
+----------------+-------------------------+--------+---------+
| Opensuse 15.X  |            Yes          |   Yes  |   No    |
+----------------+-------------------------+--------+---------+

Additional Support Information
------------------------------

Do you support the nvidia-docker2 package?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the context of the docker CLI, this package is no longer supported and can be considered deprecated.

However in the context of Kubernetes or Docker swarm, these orchestrators still rely on the old ``nvidia-docker2`` package. Until this is solved in both orchestrators, the packages will continue to be maintained and we will continue to provide support to issues that user face.

Do you support the Docker docker-ce packages?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All stable releases of the ``docker-ce`` package, installed from https://docs.docker.com/install/ and starting from docker 19.03, are supported with the ``nvidia-container-toolkit`` package.

All stable releases of the ``docker-ce`` package, installed from https://docs.docker.com/install/ and starting from docker 18.06 are supported with the ``nvidia-docker2`` package.

*Note that Edge, Test and Nightly releases are not officially supported but we will provide best effort support.*

Do you support the Canonical docker.io packages?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All stable releases of the ``docker.io`` package, provided by Canonical and starting from docker 19.03, are supported with the ``nvidia-container-toolkit`` package.

All stable releases of the ``docker.io`` package, provided by Canonical and starting from docker 18.06, are supported with the ``nvidia-docker2`` package.

Do you support the Red Hat docker packages?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All stable releases of the ``docker`` package, provided by Red Hat and starting from docker 18.06, are supported with the ``nvidia-container-toolkit`` package.

*Note that Red Hat doesn't provide updates for this package and at the time of writing, no 19.03 package were available.*

Do you support Jetson platforms (AArch64)?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Yes - beta support of the NVIDIA Container Runtime is now available on Jetson platforms (AGX, TX2 and Nano). See `this link <https://github.com/NVIDIA/nvidia-docker/wiki/NVIDIA-Container-Runtime-on-Jetson>`_ for more information on getting started.

Do you support macOS?
~~~~~~~~~~~~~~~~~~~~~

No, we do not support macOS (regardless of the version), however you can use the native macOS Docker client to deploy your containers remotely (refer to the `dockerd documentation <https://docs.docker.com/engine/reference/commandline/dockerd/#description>`_\ ).

Do you support Microsoft Windows?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No, we do not support Microsoft Windows (regardless of the version), however you can use the native Microsoft Windows Docker client to deploy your containers remotely (refer to the `dockerd documentation <https://docs.docker.com/engine/reference/commandline/dockerd/#description>`_\ ).

Do you support Microsoft native container technologies (e.g. Windows server, Hyper-v)?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No, we do not support native Microsoft container technologies.

Do you support Optimus (i.e. NVIDIA dGPU + Intel iGPU)?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Yes, from the CUDA perspective there is no difference as long as your dGPU is powered-on and you are following the official driver instructions.

Do you support PowerPC64 (ppc64le)?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Yes, little-endian only.
