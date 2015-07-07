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

[Source](http://archive.cloudera.com/cdh4/cdh/4/oozie/WorkflowFunctionalSpec.html)
* The fork and join nodes must be used in pairs. 
The join node assumes concurrent execution paths are children of the same fork node.

* All computation/processing tasks triggered by an action node are remote to Oozie

* If the failure is of transient nature, Oozie will perform retries after a pre-defined time interval. The number of retries and timer interval for a type of action must be pre-configured at Oozie level. Workflow jobs can override such configuration. 
All the commands within fs action do not happen atomically, 
if a fs action fails half way in the commands being executed, 
successfully executed commands are not rolled back.

* subworkflow:

1. The child workflow job runs in the same Oozie system instance where the parent workflow job is running.
2. The app-path element specifies the path to the workflow application of the child workflow job.
3. The propagate-configuration flag, if present, indicates that the workflow job configuration should be propagated to the child workflow.
4. The configuration section can be used to specify the job properties that are required to run the child workflow job.
5. The configuration of the sub-workflow action can be parameterized (templatized) using EL expressions.

* Java action:

The main Java class must not call System.exit(int n) as this will make the java action to do an error transition regardless of the used exit code.

* Workflow Jobs Recovery (re-run)

Oozie must provide a mechanism by which a a failed workflow job can be resubmitted and executed starting after any action node that has completed its execution in the prior run. This is specially useful when the already executed action of a workflow job are too expensive to be re-executed.

It is the responsibility of the user resubmitting the workflow job to do any necessary cleanup to ensure that the rerun will not fail due to not cleaned up data from the previous run.

When starting a workflow job in recovery mode, the user must indicate either what workflow nodes in the workflow should be skipped or whether job should be restarted from the failed node. At any rerun, only one option should be selected. The workflow nodes to skip must be specified in the oozie.wf.rerun.skip.nodes job configuration property, node names must be separated by commas. On the other hand, user needs to specify oozie.wf.rerun.failnodes to rerun from the failed node. The value is true or false . All workflow nodes indicated as skipped must have completed in the previous run. If a workflow node has not completed its execution in its previous run, and during the recovery submission is flagged as a node to be skipped, the recovery submission must fail.

The recovery workflow job will run under the same workflow job ID as the original workflow job.

To submit a recovery workflow job the target workflow job to recover must be in an end state (=SUCCEEDED=, FAILED or KILLED ).

A recovery run could be done using a new worklfow application path under certain constraints (see next paragraph). This is to allow users to do a one off patch for the workflow application without affecting other running jobs for the same application.

A recovery run could be done using different workflow job parameters, the new values of the parameters will take effect only for the workflow nodes executed in the rerun.

The workflow application use for a re-run must match the execution flow, node types, node names and node configuration for all executed nodes that will be skipped during recovery. This cannot be checked by Oozie, it is the responsibility of the user to ensure this is the case.

Oozie provides the int wf:run() EL function to returns the current run for a job, this function allows workflow applications to perform custom logic at workflow definition level (i.e. in a decision node) or at action node level (i.e. by passing the value of the wf:run() function as a parameter to the task).

* User retries

Oozie adminstrator can allow more error codes to be handled for User-Retry. By adding this configuration =oozie.service.LiteWorkflowStoreService.user.retry.error.code.ext= to oozie.site.xml and error codes as value, these error codes will be considered as User-Retry after system restart.

Examples of User-Retry in a workflow aciton is :

```xml
<workflow-app xmlns="uri:oozie:workflow:0.3" name="wf-name">
<action name="a" retry-max="2" retry-interval="1">
</action>
```

* Suspend on nodes

Specifying oozie.suspend.on.nodes in a job.properties file lets users specify a list of actions that will cause Oozie to automatically suspend the workflow upon reaching any of those actions; like breakpoints in a debugger. To continue running the workflow, the user simply uses the -resume command from the Oozie CLI . Specifying a * will cause Oozie to suspend the workflow on all nodes.

For example:

```shell
oozie.suspend.on.nodes=mr-node,my-pig-action,my-fork
```

Specifying the above in a job.properties file will cause Oozie to suspend the workflow when any of those three actions are about to be executed.
