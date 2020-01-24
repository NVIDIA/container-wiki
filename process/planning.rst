Release Process and Phases
==========================

.. # define a hard line break for HTML
.. |br| raw:: html

   <br />

Definitions
-----------

**An Initiative** is a concept that describes a large body of work. It spans
multiple releases and is a collection of Jira epics. e.g:

* Initiatives: GPU Operator, NVIDIA Container Toolkit, GPL compliance and
  NVIDIA Container Toolkit on Jetson
* Not Initiatives: RHEL Support of NVIDIA Container Toolkit, "Kubernetes", ...


**An Epic** typically describes a feature or a user story. It should not span
multiple releases but can span multiple sprints. |br|
An epic is broken down into multiple tasks and filled / written by the Product
Manager.

The Product Manager should fill Epics according to the following template:

* Overview of the feature (What?)
* Rationale (Why?)
* End user benefit (Value?)
* Product Requirements
* Priority of the feature: P0/P1/P2/Unassigned
* Milestone / Release


**A Task** a task represents work that needs to be done. It is associated with
an epic and is filled by an engineer. A task should outline a clear success
measure / criteria, it should also have a work estimate.

If we estimate a task to be more than three days, it should be split in
multiple other tasks.


**A Milestone or Release** describes the moment in time where a new version of
the software product is created. It is typically made available to external
users, requires a QA process, release notes and docs to be updated.

Planning and Execution
----------------------

**Feature Request Process**
  Requires an external or internal team member to open an epic on the team's
  Jira project. That epic should not be assigned to a milestone or sprint.

  If the request is large enough (e.g: multiple features or closer to an
  initiative), the requester should write a software design document and link
  it in the Jira Epics.

  Finally the requester should reach out to the team's distribution list with
  the created Epics and business/priority context. Note that the team's
  distribution list contains engineers, product managers and managers.


**Prioritizing Features**
  The team runs a few meeting so that we can plan and prioritize feature in
  a coherent fashion: 

* Bi-weekly sprint planning, the team follows an agile process.
  Features are assigned to engineers during this meeting.
* Monthly Feature Grooming Process, replaces one of the weekly team meetings.
  Unprioritized features gets prioritized according to PM, Management and
  Engineering inputs.
* Bi-monthly roadmap planning, where a more long term roadmap is planned.
