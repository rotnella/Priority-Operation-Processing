Install Kubernetes Components
=======================================================
The execution components run in Kubernetes as pods. The following are necessary for execution:
- [Puller](Puller) _always running_
- [Executor](Executor) _dynamically allocated by Puller_
- [Handlers](Handler) _dynamically allocated by Executor_

Prerequisites
===============
- [Docker](https://www.docker.com/) repository accessible by Kubernetes & installed locally
- [Kubernetes](https://kubernetes.io/)
- Maven 3.3.9
- Local clone of this repo!

Work in progress :warning:
==
All below...

Building / Deploying the Docker Images
==
You will need the clone of the repo to build all the necessary modules and create the images.

:warning: If you are working with Minikube please see this guide on using the built in docker repo. [[Run Local Execution With Minikube|RunLocalExecutionWithMiniKube]]

Build
--
From the root of the repo run:

`mvn clean install`

Confirm
--
Run this command and confirm the puller, executor, and sample images are listed:

`docker images`
### Sample result (trimmed down to the items of interest)
```
cppull            1.0.0  1a847b2e3c3a  About a minute ago   176MB
fhexec            1.0.3  39e12ebdc816  About a minute ago   180MB
pop-sample    1.0.2  5fa24507d7a6  2 minutes ago        147MB
```

Tag and Push
--

```
docker tag cppull:1.0.0 <your docker repo>/<your docker namespace>/cppull:1.0.0
docker tag fhexec:1.0.0 <your docker repo>/<your docker namespace>/fhexec:1.0.3
docker tag pop-sample:1.0.0 <your docker repo>/<your docker namespace>/pop-sample:1.0.2
```

```
docker push <your docker repo>/<your docker namespace>/cppull:1.0.0
docker push <your docker repo>/<your docker namespace>/fhexec:1.0.3
docker push <your docker repo>/<your docker namespace>/pop-sample:1.0.2
```

Kubernetes Setup
==
Please setup the namespace and the service account (and local credentials if needed): [[Kubernetes Setup|@todo put in link]]

Kubernetes Configs Update / Apply
==

:warning: If your namespace in Kubernetes is not **pop** be sure to update the ConfigMaps accordingly.

Location: `./deploy/resourcepool/kubernetes/configmap`

Puller
--
### ConfigMap.yaml
* Change `agendaProviderUrl` url
* Change executor image repo
* Tweak `pullAlways` and `imagePullPolicy` as desired
* Tweak `reapCompletedPods` as desired

Apply with `kubectl apply -f ConfigMap.yaml`

### Deployment.yaml
* Change the `image` to your docker repo
* Tweak the `imagePullPolicy` as desired

(do not apply yet!)

### Service.yaml
* No specific changes recommended.

(do not apply yet!)

Executor (ConfigMap.yaml)
--
* Change `agenda.progress.url` url
* Setup any new entries in the `registry-json` section (if you plan to use sample you'll need the `imageName` updated to point to your docker repo)
* tweak `pullAlways` as desired
* tweak `reapCompletedPods` as desired

Apply with `kubectl apply -f ConfigMap.yaml`

Sample (ConfigMap.yaml)
--
* Change `agenda.progress.url` url
* Change `pop.kubernetes.podconfig.docker.imageName` to point to an image available in your docker repo.
* tweak `pullAlways` as desired
* tweak `reapCompletedPods` as desired

Apply with `kubectl apply -f ConfigMap.yaml`

Puller Deployment
==

1. Navigate to `./deploy/resourcepool/kubernetes/configmap/puller`
1. Start the puller deployment with: `kubectl apply -f Deployment.yaml`
1. Start the puller service with: `kubectl apply -f Service.yaml`
1. Watch the logs for successful calls to getAgenda
