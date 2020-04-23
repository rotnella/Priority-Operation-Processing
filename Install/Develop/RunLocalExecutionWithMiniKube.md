Minikube Executor Launching Sample Tutorial
==
Description
==
This tutorial covers running an executor locally in IDEA that launches the Sample Handler and associated pod with alpine in minikube.

By the end of the tutorial you should be able to do the following:
* Launch the Sample Handler from IDEA and trigger a pod to execute in Minikube
* Launch the Executor from IDEA and trigger the Sample handler to execute in Minikube (and have it launch one too!)

What is not covered in this Tutorial:
* NFS / File mounts

Prerequisites
==

:warning: @todo The maven stuff is still very thePlatform...

1.  You have a Mac (if you have anything else these instructions may fail)
1.  You have IntelliJ IDEA setup
    1.  Please be sure any project you open is configured for maven 3.3.9 (your paths must apply)
1.  Install Docker: [https://hub.docker.com/editions/community/docker-ce-desktop-mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac)
1.  Install Minikube: [https://minikube.sigs.k8s.io/docs/start/macos/](https://minikube.sigs.k8s.io/docs/start/macos/)
    1. Please be sure HyperKit is installed: https://minikube.sigs.k8s.io/docs/start/macos/#hypervisor-setup (NOTE: "If Docker for Desktop is installed, you already have HyperKit" according to the docs)
1.  (optional, UI for viewing pods etc.) Install Kubernetic: [https://kubernetic.com/](https://kubernetic.com/)
1. Install kubectl [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
    1. Basic working knowledge of how to use kubectl

Setup
==
Configure Minikube Memory/CPU
--
The default HyperKit config for Minikube is limited. We need to tweak it to offer a bit more CPU/Memory.

Stop minikube:

`minikube stop`

Edit the following file:
`$HOME/.minikube/machines/minikube/config.json`

Tweak the following entries as indicated under the **Driver** section: (technically you can put anything but we will allocate some CPUs to our utility pod)
* `"CPU": 4,`
* `"Memory": 8192,`

Start minikube:

`minikube start --driver=hyperkit`

Configure POP + Minikube
--
Setup Kubernetes Service User and Namespace, see [DeveloperKubernetesSetup](DeveloperKubernetesSetup)

Configure Sample Handler ConfigMap
--
Navigate to where you sync'd this folder: `/deploy/resourcepool/kubernetes/configmap/sample/`

### Apply the ConfigMap
This will create a ConfigMap by this name: pop-sample-01 (name is specified in the yaml)

`kubectl create -f ConfigMap.yaml`

### Updating / Deleting the ConfigMap

If necessary you can update the configMap

`kubectl replace -f ConfigMap.yaml`

If all else fails you can delete it as well by name (and recreate)

`kubectl delete configmap pop-sample-01`

### Minikube Docker Daemon

Please setup your docker repo in the terminal to be the Minikube repo.

`eval $(minikube docker-env)`

:warning: `eval $(minikube docker-env)` must be run in each terminal window you use

Reference: [https://minikube.sigs.k8s.io/docs/tasks/docker_daemon/](https://minikube.sigs.k8s.io/docs/tasks/docker_daemon/)

### Build the sample handler images

1. Navigate to `/handler/handler-sample-impl/package`
1. Run `mvn clean install`
1. This will produce the docker image pop-sample:1.0.0
1. Confirm ths image is in your docker repo with `docker images`

### Pull the Alpine image
The external execution the Sample Handler performs is configured with the alpine:3.11 image (Controlled in the ConfigMap)

`docker pull alpine:3.11`

Execution
==

:warning: In all cases the pods created by these are not being reaped automatically for demonstration purposes. Please manually remove them.

Running the Sample Handler from IDEA
--
This will launch the sample from IDEA and trigger Minikube to launch an external pod (alpine)

1.  Load the root level pom.xml from your pop directory: `/`
1.  Please double check your maven setup as mentioned in the prerequisites!
1.  Open the `/handler/handler-sample-impl/impl/src/main/java/com/comcast/pop/handler/sample/impl/HandlerEntryPoint.java` file
1.  Create a "run" configuration for the main method.
    1.  Working directory: `(your custom path here)/`
    1.  Program arguments: `-launchType local -externalLaunchType kubernetes -propFile ./handler/handler-sample/package/local/config/external-minikube.properties -payloadFile ./handler/handler-sample/package/local/payload.json`
1.  Take a look at the input payload: `handler/handler-sample/package/local/payload.json`
1.  Attempt to run the new configuration!

### Result
The desired message from the sample run log is something along the lines of

`11:43:14.211 [main] DEBUG com.comcast.pop.handler.sample.impl.executor.kubernetes.KubernetesExternalExecutor - External execution produced: bin`
(followed by the result of the `ls` command)

The above implies the sample executed successfully!

Running the Executor from IDEA
--
If the above worked there's a very good chance this will too!

This will launch the executor from IDEA and trigger Minikube to launch the sample handler (which in turn will launch another pod)

1.  Load the root level pom.xml from: `/`
1.  Please double check your maven setup as mentioned in the prerequisites!
1.  Open the `./handler/handler-executor/impl/src/main/java/com/comcast/pop/handler/executor/impl/HandlerEntryPoint.java` file
1.  Create a "run" configuration for the main method.
    1.  Working directory: `(your custom path here)/`
    1.  Program arguments: `-launchType local -externalLaunchType kubernetes -propFile ./handler/handler-executor/package/local/config/external-minikube.properties -payloadFile ./handler/handler-executor/package/local/payload.json`
1.  Take a look at the input payload: `handler/handler-executor/package/local/payload.json`
1.  Attempt to run the new configuration!

### Result
The desired message from the executor run log is something along the lines of

`11:17:13.082 [main] INFO com.comcast.pop.handler.executor.impl.processor.JsonContextUpdater - Persisting ContextKey: [sample.1.out] OperationId: [01] with OUTPUT Payload: {"actionData":"firstAction"}`

The above implies the executor/sample executed successfully!

Troubleshooting
==
List all the pods
--
`kubectl get pods`

Describe a pod (critical if one appears to get stuck)
--
`kubectl describe pod <pod name>`

General
--
Look through the executor logs for errors!

Developing Handlers and Further Exploration
==

That was a ton of work! _How much of that is necessary once I've been through this?_

* Each handler has a ConfigMap that will need to exist (be applied to minikube) and be tweaked according to your needs. You can use the sample ConfigMap.yaml as a reference.
  * When running locally the executor uses a Java based map of handlers (by default): **StaticPodConfigRegistryClient**
  * You will need to potentially update the StaticPodConfigRegistryClient class in the Executor to point to a new handler/configMap as necessary.  
  * When the executor is run in Kubernetes it uses a Json registry. See the ConfigMap.
* Handlers must be built into docker images for kubernetes/minikube to access it!
* Learn more about allowing your Minikube instance to access local docker images (not covered nor tested for the creation of this tutorial)

Puller in IDEA
--
Just like the Executor and Sample instructions above you can run the Puller with a local Agenda to launch the executor in Minikube (launching sample, etc.).

* Apply the Executor ConfigMap: `./deploy/resourcepool/kubernetes/configmap/executor/ConfigMap.yaml`
    * This is the file you would need to adjust when you add new handlers for the executor to launch!
* Setup the PullerEntryPoint (`./handler/handler-puller/impl/src/main/java/com/comcast/pop/handler/puller/impl/PullerEntryPoint.java`)
    * Args: `-launchType local -externalLaunchType kubernetes -propFile ./handler/handler-puller/package/local/config/external-minikube-local-agenda.properties`
* Review the Agenda: `./handler/handler-puller/impl/src/main/resources/Agenda.json`
