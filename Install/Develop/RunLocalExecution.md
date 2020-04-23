Run Executor, Puller, and Handler Locally
=======================================================

Architecture Review
===================

Priority Operation Processing (POP) is responsible for the following:

*   Receiving customer input

*   Producing a viable workflow (Agenda)

*   Schedule and execute an Agenda


This document covers how to build and execute an Agenda on your local environment. The scheduling and API components are run using AWS. The processing components are run via Kubernetes. There are a combination of ways to run the processing components locally.

Installation
============

Prerequisites
-------------
- Java
- Maven 3.3.9
- Docker
- At a minimum the sample handler should compile with **mvn clean install** ([SampleHandler](SampleHandler))
- IDEA or some developer IDE. IDEA will be used as example

Components
==========

[The Puller](Puller)
----------

The puller is responsible for contacting AWS for Agendas to execute. The puller does not execute the agenda, instead it will launch an Agenda Executor pod in Kubernetes.

*   Run Puller locally and Agenda Executor as a Pod with remote Kubernetes : _Run Puller in IDEA --> Launches Agenda Executor in Kubernetes --> Launches handlers in Kubernetes_

*   Run Puller locally and Agenda Executor locally as a Pod with MiniKube : _Run Puller in IDEA --> Launches Agenda Executor in Minikube --> Launches handlers in Minikube_


[Agenda Executor](Executor)
---------------

The executor is responsible for executing the operations within an Agenda. Each operation is 1:1 with a handler. The executor will either run the handlers in memory, or launch a pod in Kubernetes.

*   Run Agenda Executor locally and Operation Handlers in memory : _Run Agenda Executor in IDEA --> Launches handlers in memory_

*   Run Agenda Executor locally and Operation Handlers as Pods with remote Kubernetes : _Run Agenda Executor in IDEA --> Launches handlers in Kubernetes_

*   Run Agenda Executor locally and Operation Handlers locally as Pods with MiniKube : _Run Agenda Executor in IDEA --> Launches handlers in Minikube_


[Handler](Handler)
-----------------

The handler is responsible for performing the work of a given operation. Some handlers launch external processes as necessary (if required at all by the operation handler).

*   Run Handler logic in memory : _Run Agenda Executor in IDEA --> Launches handler in memory and any processes as external binary launches_

*   Run Handler logic in memory and any external processes with remote Kubernetes : _Run Payload in IDEA --> Launches handler in memory_ and processes in Kubernetes

*   Run Handler logic in memory and any external processes with Minikube : _Run Payload in IDEA --> Launches handler in memory_ and processes in Minikube


Running
=======

Running with only MiniKube
-----------------------
[Running with Minikube](RunLocalExecutionWithMiniKube)

Running with Kubernetes
-----------------------
- Gain access to Kubernetes and setup environment

Run [Resource Pool : Agenda Executor](Executor)

The Agenda Executor will use a deployed handler in Kubernetes.

1.  Open executor project in IDEA : git depot: /handler/executor/  

2.  Kubernetes Auth requires the use of a cert/token so the given component can communicate with the cluster.

    1.  If command line arg based: (files created in the in the [DeveloperKubernetesSetup](DeveloperKubernetesSetup) instructions )

        1.  Add these: -oauthCertPath (cert file path) -oauthTokenPath (token file path)

    2.  If environment var based:

        1.  Set Environment Variable OATH\_CERT with the value inside the \*.cert file you created in the [DeveloperKubernetesSetup](DeveloperKubernetesSetup) instructions  

        2.  Set Environment Variable OATH\_TOKEN with the value inside the \*.token file you created in the [DeveloperKubernetesSetup](DeveloperKubernetesSetup) instructions

3.  Set up a run command for HandlerEntryPoint.java

4.  Set Program Arguments/ApplicationParameters to the command listed in the comments section of HandlerEntryPoint.java for Kubernetes

    Something like >> -launchType local -externalLaunchType kubernetes -propFile ./handler/main/package/local/config/external.properties -payloadFile ./handler/main/package/local/payload.json


Run A Handler
=====================
The instructions above apply to running a handler as well.

[Sample Handler](SampleHandler)
--------------
*   Impl: handler/handler-sample-impl

*   API: handler/handler-sample-api


Run Puller
==========

Launch Agenda Executor using Kubernetes
---------------------------------------

The Puller will use the deployed Agenda Executor in Kubernetes.

1.  Open puller project in IDEA : handler/handler-puller/

2.  Set up a run command for PullerApp.java

3.  Set Program Arguments/ApplicationParameters to the command listed in the comments section of PullerApp .java for Kubernetes  
    Something like >> \-launchType local -externalLaunchType kubernetes -propFile ./handler/main/package/local/config/external.properties

4.  Kubernetes Auth:

    1.  If command line arg based (requires KubernetesLaunchDataWrapper) :

    2.  Add these: -oauthCertPath (cert file path) -oauthTokenPath (token file path)

    3.  If environment var based:

    4.  Set Environment Variable OATH\_CERT with the value inside the \*.cert file you created in the [DeveloperKubernetesSetup](DeveloperKubernetesSetup) instructions

    5.  Set Environment Variable OATH\_TOKEN with the value inside the \*.token file you created in the [DeveloperKubernetesSetup](DeveloperKubernetesSetup) instructions

Building and pushing an image
-----------------------------

Build

Navigate to the folder with the docker file and run this command. You'll want to pick a unique image name so you don't clash with other devs/images!  

This may take a while!

`docker build -t <unique name>:<desired tag> .`

Tag the image so it can be pushed to the repository.

`docker tag <unique name>:<desired tag> docker-your.repo.com/<unique name>:<desired tag>`

Push

You need to push the image to the docker repository for use when launching the utility pod in kubernetes.  

`docker push docker-your.repo.com/<image name>:<tag>`

Sample of the build and push commands

`docker build -t sample_util:``1.0` `.`

`docker tag sample_util:``1.0` `docker-your.repo.com/sample_util:``1.0`

`docker push docker-your.repo.com/sample_util:``1.0`

Usage with a local handler
--------------------------

Adjust your external.properties file to have the docker prototype repository repo based image name.  

@todo Tim change this configuration.
`#cp.handler.analysis.imageName=docker-your.repo.com/<image name>:<image tag>`

`cp.handler.analysis.imageName=docker-your.repo.com/sample_util:``1.0`

Troubleshooting
===============

Invalid / Missing Cert Error
----------------------------

Empty input at sun.security.provider.X509Factory.engineGenerateCertificate(X509Factory.java:106)

If you see the above message your cert is invalid.

*   Double check the cert is for the right kubernetes cluster

*   Double check you are targeting the right kubernetes cluster

*   Make sure the OATH\_CERT and OATH\_TOKEN env vars are set
