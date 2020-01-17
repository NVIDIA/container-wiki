Quality Assurance
=================

.. contents:: Table of Contents
   :local:

Tested Platforms
----------------

* Vanilla Kubernetes, 1 Tesla GPU node - AWS.
* Vanilla Kubernetes, 1 Tesla GPU node - Ubuntu 18.04.
* Openshift 4.1, 3 CPU nodes and 2 RHCOS Tesla GPU node.

End to End Stories
------------------

As a cluster admin, I want to be able to install the GPU Operator with helm, Kubernetes, Ubuntu and Docker.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Validate that all pods are running.
* Validate that we can run a CUDA application.
* Validate that we can run a Tensorflow notebook.

As a cluster admin, I want to be able to install the GPU Operator with helm, Openshift 4.1, RHCOS and CRIO.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Validate that all pods are running.
* Validate that we can run a CUDA application.
* Validate that we can run a Tensorflow notebook.

As a cluster admin, I want to be able to gather GPU metrics after installing the GPU Operator.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Validate that we can gather metrics.
* Validate that we can plug the GPU metrics in something like Grafana or prometheus. Is it validating instructions and/or is this automatable? Probably not.

ipmi_msghandler isn't loaded
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As a cluster admin, I want to ensure that CPU nodes are not configured by the GPU Operator.**
* Validate that all pods in the `gpu-operator-resources` have the GPU label requirement

Tainted Nodes
~~~~~~~~~~~~~

As a cluster admin, I want to ensure that the GPU Operator doesn't deploy a failing monitoring container.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Validate that dcgm doesn't get deployed on Openshift <= 4.2

Key Performance Indicator
-------------------------

Quality Assurance Score Card
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Number of E2E Tests
* Unit Tests Coverage
* Number of Manual Tests / Number of Tests
* Number of Automated Tests / Number of Tests
* Number of user stories covered / Number of stories written
* Number of regression tests
* [REJECTED] Burndown Rate: rejected because the amount of work required is to high.

Performance Score Card
~~~~~~~~~~~~~~~~~~~~~~

* Installation Time
* Scalability of the GPU Operator
* Memory Consumption
* CPU Consumption
* Network Consumption (kb/minute)

Security Score Card
~~~~~~~~~~~~~~~~~~~

**Best practices**

* Small surface (ports opened, software installed, permissions given)
* Container Best Practices (slim image, latest version)
* Bill of materials present
* Bonus: provide a link to a website that provides security information
* TBD

**Threat modeling (-100)**

**Automation in place**

* CVE Analysis
* Automated publication

Bill of Materials Score Card
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Number of Dependencies
* Number of Licenses
* License Blacklist and Whitelist
