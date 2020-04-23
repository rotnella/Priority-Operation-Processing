Resource Pool: Logger Handler
=================================

Execution
=========

A resident handler runs within the Executor scope. For Kubernetes the operation will run within the same pod and same Docker image as the Executor. Whereas Handlers are run as separate pods / images and have their own memory & CPU constraints.

_Note: Be mindful of memory & CPU constraints when deploying your Executor pod_

Objective
=========

The Log handler is simply logs a configured message.

UseCase
-------

A workflow has some operation output that the customer wants to log for debugging and/or dashboards.

Inputs
======
|Field|Type|Notes|
|-----|----|-----|
|payload|Object|
|.logMessages|String\[\]|Messages to log allowing variable referencing|

Function
========

Creates a logger line for each string in logMessages array.

Example Agenda Operation
========================
```
{  
"payload": {  
"logMessages": [  
"operationType=logHandler operationName=analysis bitrate=@<mediaInfo.1.out::/inputStreams/video/0/params/bitrate> duration=@<mediaInfo.1.out::/inputStreams/video/0/params/duration> externalFileId=@<mediaInfo.1.out::/inputs/0/params/externalFileId>"  
]  
}
```
