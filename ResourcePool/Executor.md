Resource Pool : Agenda Executor
========================================

Functionality
=============

The executor is responsible for interpreting and executing an Agenda.

*   Operation types are mapped to docker images via a registry of operation type names

*   Operations are run in parallel as long as there are no dependencies. As soon as all the dependencies of an operation are met the operation may run.

    *   There is a thread pool limit (defaults to 50, configurable)

*   The executor supports the concept of the following handlers:

    *   Resident (may deprecate): Operation type handler that is built into the executor.

    *   External: Kubernetes launched pod running the operation's image

*   Operation progress and state is monitored via the Kubernetes API.

    *   The progress is tracked via an annotation on the operation pod

    *   The kubernetes module we use also monitors for logging from a pod. A quiet pod indicates a problem so there is a timeout if logging stops.

*   Progress is reported to the Priority Operation Processing (POP) API by the executor.

    *   It gathers all the progress for the running/completed operations and sends the information

    *   Previously completed and pending operations are not reported.

*   The executor runs an Agenda and exits (after communicating its own success/failure)


Source Code
===========

Kubernetes Configuration
========================

Inputs
======

Agenda
------
|Field|Purpose|Type|Notes|
|-----|-------|----|-----|
|operations|The collection of operations to perform|Operation\[\]|See [API: Agenda](AgendaAPI) _operation_|
|params|Map of parameters specific|
|id|Id of the original request|String|

### Operation
|Field|Purpose|Type|Notes|
|-----|-------|----|-----|
|payload|The payload to pass to the operation handler|Object|This should never be a String for the sake of _**sanity**_.|
|type|The type of operation|String|Examples: encode, analysis|
|id|Unique identifier|String|
|name|Unique identifier, specifically used to allow reference to the output of this operation|String|

Command Line Arguments
----------------------

[ResourcePool: Handler](Handler) _common command line arguments_

Environment Variables
---------------------

[ResourcePool: Handler](Handler) _common environment variables_

|Property|Purpose|Default Value|Notes|
|--------|-------|-------------|-----|
|PROGRESS\_ID|Specifies the progress id to report Agenda Progress with. This is generally provided by the Puller when launching the executor so it can report on the Agenda before it even converts it from JSON. For AgendaProgress testing it can be overwritten in the debugger.

Properties Files Values
-----------------------

[ResourcePool: Handler](Handler) _common property file values_

|Property|Purpose|Default Value|Notes|
|--------|-------|-------------|-----|
|executor.reap.self|Flag indicating if the executor should request its own delete|true|

Development
===========

HandlerEntryPoint
-----------------

As with most handlers there is a way to run the executor locally based on command line arguments. See the comments on the HandlerEntryPoint.main method. The payload file for the Executor is an Agenda (ExecutorHandlerInput extends Agenda and has no additional fields).

Local Podconfig Registry
------------------------

StaticPodConfigRegistryClient is a class that represents the pod ConfigMapping for all the handlers the local executor can use. We lightly maintain this so it may be out of date. By default the StaticPodConfigRegistryClient is used by local execution.

### Example

Sample Handler is controlled by this code in the StaticPodConfigRegistryClient class.
```
podConfigMap.put("sample",
makePOPBasePod("lab-myhandler-pod-01")
.setImageName("docker-my.repo.com/myimage:1.0.2")
.setNamePrefix("my-kubname-space")
);
```

Agenda Tokens
=============

These are defined as constants in AgendaToken.java in the API module. Agenda tokens may be used as part of an agenda. The values indicated below will be replaced as defined.

|Token|Content|Notes|
|-----|-------|-----|
|pop.agendaId|The id of the Agenda being processed (or a generated UUID if null)|
|pop.operationName|The name of the operation being processed|

Progress Reporting
==================

The executor is the sole reporter of progress to the HTTP API. It collects the progress of outstanding operations and reports it. It compiles a roll-up percent complete by a basic sum of the percents of the ops divided by the op count (at least for now).

Operation Processing
====================

Sequential vs. Parallel
-----------------------

The original implementation of the executor was simply parallel for prototyping. This functionality exists within the SequentialAgendaProcessor (only lightly maintained).

The ParallelOperationAgendaProcessor is the primary entry point for the current processing functionality.

Operation Input
---------------

The input of an operation is processed as follows:

1.  The payload has any references replaced based on the presence of specially formed strings. The string below indicates to pull data from a prior operation named **sample.1** and specifically take the data from the **/actionData** node.
```
    @<sample.1.out::/actionData>
    ```

    Note: The **::path** is optional and is a JSON pointer string ([https://tools.ietf.org/html/rfc6901)](https://tools.ietf.org/html/rfc6901)

2.  The payload is passed to the operation handler for execution.


More information on reference replacement: [API: Agenda Template](AgendaTemplateAPI)

Operation Output
----------------

The output of an operation is processed as follows in the executor:

1.  The output payload is persisted in a context map with the key (operation.name).out (.out is appended) and the value is the payload as json. This output can be used for future operations.


Parallel Processing
-------------------

Parallel processing in the executor is achieved using a single cross thread BlockingQueue employing a timeout for re-evaluation. The implementation was kept fairly simple to avoid the complexity and bugs that could come up with this type of system. Essentially there is a queue for tracking completed operations. This queue is added to by operations completing and read from by the main OperationConductor for post processing. This queue read with a timeout allows for future development with the executor so it can perform additional actions such as cancel, additional logging, etc.

![images/download/attachments/226435206/ExecutorMultiThreaded.svg](images/download/attachments/226435206/ExecutorMultiThreaded.svg)

Note: The ExecutorService in the diagram above is a Java ExecutorService (not a reference to some custom part of the Executor)

There are a few phases that the OperationConductor works through:

1.  Initialization:

    1.  Create the ExecutorService/ThreadPool

2.  Drain all the pending operations (wait for all the pending operations to be at least started)

    1.  Start any ready (all prerequisites met) pending operations

3.  Drain all the running operations

    1.  Wait on all running operations to be complete


Operation Continuation (Retry)
------------------------------

[Priority Operation Processing: Agenda Retry](AgendaRetry)

Operation Generators
--------------------

A handler may produce a list of operations for execution within the current Agenda. These operations will be added to the pending operations collection for consideration and execution (pending any dependencies).

The output of an operation that produces operations/params should be passed into an **agendaUpdate** operation. The **agendaUpdate** operation must be tagged as a generator using the following parameter: **agenda.operation.generator**

If the **agenda.operation.generator** param is not set on the **agendaUpdate** operation params the Agenda will be updated in the Priority Operation Processing (POP) brain but the executor will not inject and execute them. **There is some room for improvement here but should be approached cautiously.**

The operation that generates is kept separate from the agendaUpdate for the purpose of progress tracking and the fact that the generator may be an expensive external call. On success the generator should not have to repeat execution if the agendaUpdate simply fails.

The value of the **agenda.operation.generator** parameter contains the following information (may expand as necessary):

|Field|Description|Required|Example|Notes|
|-----|------------|-------|-------|-----|
|operationsJsonPointer|The JSON pointer string to the entry of the output payload containing the list of Operations. The main point of this is because the output of an operation can include both new operations and other unrelated information.|Y|"/agenda/operations"|[https://tools.ietf.org/html/rfc6901](https://tools.ietf.org/html/rfc6901)|
|paramsJsonPointer|The JSON pointer string to the entry of the output payload containing the object with params that should be appended to the Agenda params field.|N|"/agenda/params"|

### Example Operation Array from an Agenda (tagged as a generator – see the params block)
```
[
{
"payload": {
"inputs":null,
"inputStreams":"@<mediaInfo.1.out::/inputStreams>",
"outputStreams":null,
"originalRequest":"@<mypayload.1.out::/request>"
},
"type":"pretranscode",
"id":null,
"name":"pretranscode.1"
},
{
"payload": {
"operations""@<pretranscode.1.out::/agenda/operations>"
},
"params": {
"agenda.operation.generator": {
"operationsJsonPointer":"/operations"
}
},
"type":"agendaUpdate",
"id":"agendaUpdate.id",
"name":"agendaUpdate.1"
}
]
```
### Example Output Payload from PreTranscode (truncated a bit)

Note: The path to the operations matches the above indicated json pointer.
```
{
"otherData": {
"that": {
"has":"nothing to do with the operations generated."
}
},
"agenda": {
"operations": [
{
"payload": {
"inputs": [
{
"type":"video",
"url":"/mymounted/file/small.mp4",
"label":null,
"credentials": {},
"params": {}
}
],
"outputs": [
{
"type":"video",
"url":"/mymounted/file/agenda-@<pop.agendaId>/streams/demux/small_video_0.mp4",
"label":null,
"credentials": {},
"params": {
"source_streams": [0],
"mute_audio":false,
"allow_overwrite":true
},
"outputStreamRefs":null
}
],
"params":null
},
"type":"transcode",
"id":"c14cbguid",
"name":"ffmpeg.demux.video.0",
"params": {}
}
]
}
}
```
All generated operations will be persisted with the following ParamsMap entries:

|Field|Value Type|Purpose|Notes|
|-----|----------|-------|-----|
|generated.operation.parent|String|The name of the operation that generated this operation|This is critical for progress reset. On reset of an operation generator all the child generated operations should be removed from the Agenda as well as their OperationProgress.|

**Consider:** Another approach to dependency would be to use the explicit dependsOn in the params of the generated Operations. This would unify declared dependencies as well as the tracking of generated operations into the same logic.

### External Changes

Without the option to retry an Agenda (especially partially) there would be no real need to make external changes. Because we want to continue from the point of failure or reset Operations by name we need to properly impact any downstream operations.

#### Priority Operation Processing API - Agenda Mutation

*   Endpoint (expandAgenda)

*   Adds Operations to an Agenda

*   Generates any OperationProgress objects for the new Operations

*   Calling this method is built into the updateAgenda resident handler, specifically kept separate from the execution of the operation generator. Operation generators may be external and technically succeed while the agenda update fails. There's no reason to repeat the op generation if the failure is only on Agenda update.


**Priority Operation Processing API - Rerun Changes**

*   When an operation that is a generator is reset the child Operations/OperationProgress should be removed

    *   This should be recursive in relation to generators that generate other generators(!)
