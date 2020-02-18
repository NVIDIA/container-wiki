Release Process and Phases
==========================

.. # define a hard line break for HTML
.. |br| raw:: html

   <br />

Feature Planning and Release
----------------------------

The GPU Operator release is described in the below table.
+-----------------------------+-----------------+
|           What              |       Week      |
+=============================+=================+
| Feature Planning            | Week -2         |
+-----------------------------+-----------------+
| Feature Freeze              | Day 0           |
+-----------------------------+-----------------+
| Schedule Finalized          | End of Week 0   |
+-----------------------------+-----------------+
| Nightly Releases            | Any Time        |
+-----------------------------+-----------------+
| Code Freeze                 | Day 3 of Week 3 |
+-----------------------------+-----------------+
| Github Tech Preview Release | Day 1 of Week 4 |
+-----------------------------+-----------------+
| Nightly Releases            | Any Time        |
+-----------------------------+-----------------+
| Code Freeze                 | Day 2 of Week 7 |
+-----------------------------+-----------------+
| NGC Release                 | Day 1 of Week 8 |
+-----------------------------+-----------------+

**Nightly Releases** are published after each successful CI run and only go through an automated QA. |br|
**Github Releases** require the results of fully qualified QA to be published. |br|
**NGC Releases** require external approval, security scanning, and a published fully qualified QA.

**Automated QA** is the set of tests that are ran automatically after each Merge Request. |br|
**Fully Qualified QA** is the test plan we run through and metrics we collect before releasing. |br|

**Feature Freeze** is the day after which no new features will be accepted. |br|
**Code Freeze** is the day after which no new "Feature" Merge Requests will be accepted. Engineers should focus on bug fixes and QA.

Release Process Goals
---------------------

As we improve the QA process of the different release, we will be striving towards the following goals:
* P0: Reduce the cost of "ngc" releases
* P0: Reduce the cost of "github" releases
* P1: Merge ngc release in github releases
* P1: Merge github releases in sprints

Release Phases
--------------

**Feature Freeze**: All enhancements wishing to be included in the current release must have been agreed on. |br|
**Planning Freeze**: All enhancements wishing to be included in the current release must have been agreed on. |br|
**Code Freeze**: All enhancements wishing to be included in the current release must have been agreed on. |br|

Write a container-wiki page which contains the following information:
* The Feature List
* The Test Results
* The KPI
* The release notes
* The schedule and target dates
