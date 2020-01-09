Frequently Asked Questions
==========================

How do I install the NVIDIA driver?
-----------------------------------

| The recommended way is to use your `package manager`_ and install the
  ``cuda-drivers`` package (or equivalent).
| When no packages are available, you should use an official
  `runfile`_.

| Alternatively, and as a technology preview, the NVIDIA driver can be
  deployed through a container.
| Refer to the `documentation`_ for more information.

I’m getting ``The following signatures were invalid: EXPKEYSIG`` while trying to install the packages, what do I do?
--------------------------------------------------------------------------------------------------------------------

Make sure you fetched the latest GPG key from the repositories. Refer to
the `repository instructions`_ for your distribution.


.. _package manager: http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#package-manager-installation
.. _runfile: http://www.nvidia.com/object/unix.html
.. _documentation: https://github.com/NVIDIA/nvidia-docker/wiki/Driver-containers-(EXPERIMENTAL)
.. _repository instructions: https://nvidia.github.io/nvidia-docker/
