---
layout: post
title:  "Oozie Study Notes"
date:   2015-07-07
categories: Study-Notes
excerpt: git
comments: true
---

* content
{:toc}

## Basics ##

* The fork and join nodes must be used in pairs. 
The join node assumes concurrent execution paths are children of the same fork node.

* All computation/processing tasks triggered by an action node are remote to Oozie

*If the failure is of transient nature, Oozie will perform retries after a pre-defined time interval. The number of retries and timer interval for a type of action must be pre-configured at Oozie level. Workflow jobs can override such configuration. 
All the commands within fs action do not happen atomically, 
if a fs action fails half way in the commands being executed, 
successfully executed commands are not rolled back.

* subworkflow:

1. The child workflow job runs in the same Oozie system instance where the parent workflow job is running.
2. The app-path element specifies the path to the workflow application of the child workflow job.
3. The propagate-configuration flag, if present, indicates that the workflow job configuration should be propagated to the child workflow.
4. The configuration section can be used to specify the job properties that are required to run the child workflow job.
5. The configuration of the sub-workflow action can be parameterized (templatized) using EL expressions.

