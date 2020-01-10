Quality Assurance
===============

Setups that are tested
----------------------

1. Vanilla Kubernetes, 1 Tesla GPU node - AWS
2. Vanilla Kubernetes, 1 Tesla GPU node - Ubuntu 18.04
3. Openshift 4.2,      3 CPU nodes and 2 RHCOS Tesla GPU node

Positive Tests
--------------

1. Installation Sanity
   a. CUDA samples application
   b. TF notebook pod
   c. No pods crashing

2. Monitoring Feature
   a. CUDA samples running and check the exporter metrics

Negative Tests
--------------

Automated End to End Tests
--------------------------
