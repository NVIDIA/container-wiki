Quickstart
==========

.. contents:: Table of Contents
   :local:


What is the NVIDIA GPU Operator?
--------------------------------

The GPU operator manages NVIDIA GPU resources in a Kubernetes cluster.
It allows to and automates tasks related to bootstrapping GPU nodes.

GPUs are a special resource in the cluster, and require a few components to be present
before application workloads can be deployed onto the GPU.
These components include the NVIDIA drivers (to enable CUDA), Kubernetes device plugin,
the container runtime and others such as automatic node labelling, monitoring etc.

This is a technical preview release of the GPU operator. The operator can be deployed using a Helm chart.

Installation of the GPU Operator
--------------------------------

Ensure the following prerequisites are met:

* Nodes must not be pre-configured with NVIDIA components (driver, container runtime, device plugin).
* The ``i2c_core`` and ``ipmi_msghandler`` kernel modules need to be loaded

.. code-block:: sh

   $ curl -L https://git.io/get_helm.sh | bash
   $ helm version
   version.BuildInfo{Version:"v3.0.2", GitCommit:"19e47ee3283ae98139d98460de796c1be1e3975f", GitTreeState:"clean", GoVersion:"go1.13.5"}

   $ helm repo add nvidia https://nvidia.github.io/gpu-operator
   $ helm repo update

   $ helm install --devel nvidia/gpu-operator --generate-name

   # To check the gpu-operator version
   $ helm ls


Running a Sample GPU Application
--------------------------------

.. code-block:: sh

   # Create a tensorflow notebook example
   $ kubectl apply -f https://nvidia.github.io/gpu-operator/notebook-example.yml

   # Grab the token from the pod once it is created
   $ kubectl get pod tf-notebook
   $ kubectl logs tf-notebook
   ...
   [I 23:20:42.891 NotebookApp] jupyter_tensorboard extension loaded.
   [I 23:20:42.926 NotebookApp] JupyterLab alpha preview extension loaded from /opt/conda/lib/python3.6/site-packages/jupyterlab
   JupyterLab v0.24.1
   Known labextensions:
   [I 23:20:42.933 NotebookApp] Serving notebooks from local directory: /home/jovyan

      Copy/paste this URL into your browser when you connect for the first time,
          to login with a token:
             http://localhost:8888/?token=MY_TOKEN
   You can now access the notebook on http://localhost:30001/?token=MY_TOKEN

Platforms Supported
-------------------

+----------------+----------------------------------------+
|      What      |               Supported                |
+================+========================================+
| GPUs           | Pascal+ (including Tesla V100 and T4)  |
+----------------+----------------------------------------+
| Ubuntu         | 18.04.3 LTS                            |
+----------------+----------------------------------------+
| RHCOS          | rhcos4.1+                              |
+----------------+----------------------------------------+
| Kubernetes     | v1.13+                                 |
+----------------+----------------------------------------+
| Openshift      | v4.1+                                  |
+----------------+----------------------------------------+
| Helm           | v2+                                    |
+----------------+----------------------------------------+

Known Limitations
-----------------

* GPU Operator will fail on nodes already setup with NVIDIA components (driver, runtime, device plugin).
* Removing the GPU Operator will require you to reboot your nodes.


Getting Help
------------

Please open `an issue on the GitHub project <https://github.com/NVIDIA/gpu-operator/issues/new>`_ for any questions. Your feedback is appreciated.
