Resource Pool: Handler
================

Priority Operation Processing Diagram
===============

![images/download/attachments/237552655/DFHDesign-overview-simple.jpg](images/download/attachments/237552655/DFHDesign-overview-simple.jpg)

What is a Handler?
------------------

In the diagram above the gray circles are representing a handler. A handler is a piece of software that has an input and output API and performs some operation based on the inputs. A handler is eventually dockerized and called by the POP Executor software on-demand. Priority Operation Processing (POP) was developed for media content processing. The above diagram is an example of a customer's transcoding operations. POP software includes handlers to support transcoding, thumbnail generation, filmstrips and packaging of media. However, Priority Operation Processing (POP) software is flexible and can handle other types of handlers, not limited to media file processing.

References
==========

|Component|Description|Reference|
|---------|-----------|---------|
|POP Agenda API|Each Agenda.operation is one to one with a Handler.|[Priority Operation Processing: Agenda](AgendaAPI)|
|POP Executor|Software that will call Handlers based on the Agenda's operation list and whether or not dependencies are met.|[Priority Operation Processing: Resource Pool :  Executor](Executor)|
|POP Handler Structure||[Priority Operation Processing: Handler](Handler)|

Run the Executor and your Handler locally

[Handler : Run Executor, Puller, and Handler Locally](HandlerDevelopment)

EFS/NFS pod and connecting to pods

Details the process for configuring a custom pod to access an NFS as well as how to connect to a running pod.

[EFS Access and Direct Pod Execution](@todo add doc)


### Sample Handler

The Sample handler shows handler code structure and performs a simple task.

GIT : handler/sample/

Handler with external binary calls
----------------------------------

The handlers make calls to an external binary like ffmpeg which is launched as a separate pod and can have its own CPU request/configuration.

### Common command line Arguments
Command Line Arguments
----------------------

Standardized all handlers to operate with the following args:

|Argument|Purpose|Options|Defaults|
|--------|-------|-------|--------|
|externalLaunchType|The type of launch for externally called things (stuff the handler calls out to)|[Kubernetes,Local,Docker]||Kubernetes|
|launchType|The type of launch for this handler. This is necessary to determine the Reporter type.| [Kubernetes,Local,Docker]|Kubernetes|
|propFile|The properties file to use when loading the handler.||./config/external-properties |
|payloadFile|The file containing the payload to run the handler with. This is an **override**. N/A payload comes from the environment variable.|
|oauthCertPath|The path to the X.ca.cert you generated for use locally testing against a kubernetes cluster|
|oauthTokenPath|The path to the X.sa.token you generated for use locally testing against a kubernetes cluster|





Output
======

At this time we plan to use Kubernetes annotations to persist progress/failure/success data. The annotation names are pending but will need to be centralized so both the executor and the handler can read/write them.

Pod annotation updates (and any PATCH methods) are not permitted by the default service user when a pod is executing.


Shared Classes / Modules
========================

Instead of having a base class from which all handlers are derived we are using a number of supporting modules to create handlers (at least for now). Everything could move...

|Module|Class(es)|Purpose|Notes|
|------|---------|--------|-----|
|pop-handler-field-retriever|LaunchDataWrapper|Provides a general access to:[ arguments, environment variables, properties (1 prop file)]. Also includes override capabilities via args:[ payload, properties file]|DefaultLaunchDataWrapper is the only implementation at this time. This can be completely overridden and tweaked if other arguments are needed to support a given handler.|
|pop-handler-base|BaseOperationContext(Factory),HandlerProcessor,<br/>BaseHandlerEntryPoint,InMemoryHandler|Provides basic structure for a handler (via the BaseHandlerEntryPoint)|Previous implementations have been copy/paste duplicates.|
|pop-handler-kubernetes-support|KubernetesOperationContextFactory,KubeConfigFactoryImpl|Provides a basic set of kubernetes related classes supporting the handler base module.|
|pop-handler-reporter-kubernetes|KubernetesReporter,KubernetesReporterSet|Kubernetes specific reporters (intended for status, not logging)|
|pop-handler-reporter-logger|LogReporter|Generic logger reporters|



PodConfig Registry
==================

The PodConfig registry provides the basic settings for launching other pods. This concept is used primarily with the Puller and Executor.

Kubernetes Execution
--------------------

The configmap contains the registry as a json object. This allows us to reconfigure the executor/puller easily without introducing an entire service for the configuration. For example, the executor uses the operation type on an operation to look up in the registry which handler to launch. The **basePodConfig** provides a template so fields don't have to be replicated. If a field is overridden the entire field is applied (no merging of the template and the override field).
