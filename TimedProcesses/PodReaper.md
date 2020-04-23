Timed Process: Pod Reaper
===================

Requirements
============

*   Remove completed pods (error or successful)

    *   This is primarily due to the fact that the executor is launched un-managed (the puller does not track the pod id at all)

*   Run on an interval (see CronJob.yaml)

*   Configurable reap age for pods.


Source Code
===========

Kubernetes Configuration
========================


Inputs
======

N/A - There are no inputs for the pod reaper, only configuration settings.

Command Line Arguments
----------------------

[ResourcePool: Handler](Handler) _common command line arguments_

Environment Variables
---------------------

[ResourcePool: Handler](Handler) _common environment variables_

Properties Files Values
-----------------------

[ResourcePool: Handler](Handler) _common property file values_

### Properties Settings

|Property|Value Type|Notes|
|--------|----------|-----|
|cp.kubernetes.masterUrl|String|Generally not needed if running as a pod. Will default to the cluster the pod is running in.|
|cp.kubernetes.namespace|String|
|pod.reap.age.minutes|Number|The age in minutes that a pod must be to be reaped.|
|pod.reap.batch.size|Number|The amount of items to delete in a single request to kubernetes.|
