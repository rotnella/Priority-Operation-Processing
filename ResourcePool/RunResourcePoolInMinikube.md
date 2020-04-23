Run a ResourcePool in Minikube
==
You can run a ResourcePool locally in Minikube with all the components launching as on-demand pods. The tutorial covers the basics. By the end you will be able to submit an Agenda to the API Gateway endpoint and have your Minikube based ResourcePool pick up and process it!

Prerequisites
--
* You must have the endpoint CloudFormation deployed. You need the API Gateway URL.
* You have proven your endpoint CloudFormation deployment by running through: [Endpoint Functional Tests](EndpointFunctionalTests) (the steps include the creation of some data used by this tutorial)
* You have gone through this tutorial [Running Local Execution with Minikube](RunLocalExecutionWithMiniKube)
    * Lots of critical setup for Minikube use is covered.
* No puller component is currently running in the minikube cluster.

Setup
==

ConfigMap Updates
--

### Executor
Path: `./deploy/resourcepool/kubernetes/configmap/executor/ConfigMap.yaml`
#### Adjustments
* `agenda.progress.url`: <your endpoint url (including stage path)>/pop/resourcepool/service
* `pullAlways`: false (required for Minikube/Docker shared repo functionality)
* `reapCompletedPods`: false (optional, so you can view the logs)

#### Apply Updates (from the folder with the ConfigMap)
`kubectl replace -f ConfigMap.yaml`

### Puller
Path: `./deploy/resourcepool/kubernetes/configmap/puller/ConfigMap.yaml`
#### Adjustments
* `agendaProviderUrl`: <your endpoint url (including stage path)>/pop/resourcepool/service
* `insightId`: '547cc659-f7a3-4fbe-b171-a2d0ad355b4d'
* `pullAlways`: false (required for Minikube/Docker shared repo functionality)

#### Apply Updates and launch puller  (from the folder with the ConfigMap)
```
kubectl replace -f ConfigMap.yaml
kubectl apply -f Deployment.yaml
```

#### Service.yaml
The Service.yaml is not needed for the puller unless you wish to expose the alive check. Please evaluate and update as necessary.


Data Updates
--
If you are using the data detailed in [Endpoint Functional Tests](EndpointFunctionalTests) you only need to make one modification.
Add "sample" as an entry in the Insight object. This configures the insight to associate with Agendas with the operation type "sample" when they are submitted.
This is a DynamoDB Json snippet
```
(...)
  "mappers": {
    "M": {
      "operationType": {
        "SS": [
          "sample",
          "testOperation"
        ]
      }
    }
  },
(...)
```

Execution
==
1. Load the main pom.xml in IDEA
1. Navigate to this file: `./endpoint/endpoint-functional-test/src/test/java/com/comcast/pop/endpoint/test/manual/ManualTest.java`
1. Run the submitAgenda test.
    1. This test uses this payload file and defaults to the customerId used in the functional tests: `./endpoint/endpoint-functional-test/src/test/resources/com/comcast/pop/endpoint/test/manual/sampleAgenda.json`
1. Wait patiently for it to complete (kubernetic can help show what's going on)

:warning: The configuration does not clean up the pods (as previously noted in the doc). Please manually remove them. The puller is also left executing. Remember to remove and shutdown anything.

Sample results snippet
--
```
15:44:40.897 [main] INFO com.comcast.pop.endpoint.test.manual.ManualTest - Final Progress : {
  "id" : "17908d83-b5ce-4de7-88a3-4a839113e55b",
  "cid" : "d4ea676b-14d2-4125-9dbf-ac239a802f58",
  "customerId" : "account/3131523765",
  "title" : null,
  "updatedTime" : 1585867477158,
  "addedTime" : 1585867442127,
  "agendaId" : "f5f0922a-3bfb-47e9-8d07-b686fa71a844",
  "linkId" : "8b1740aa-c5ca-483a-b0e3-1c0efce95255",
  "externalId" : null,
  "processingState" : "COMPLETE",
  "processingStateMessage" : "succeeded",
  "operationProgress" : [ {
    "id" : "17908d83-b5ce-4de7-88a3-4a839113e55b-sample.1",
    "cid" : "d4ea676b-14d2-4125-9dbf-ac239a802f58",
    "customerId" : "account/3131523765",
    "title" : null,
    "updatedTime" : 1585867477119,
    "addedTime" : 1585867442294,
    "agendaProgressId" : "17908d83-b5ce-4de7-88a3-4a839113e55b",
    "processingState" : "COMPLETE",
    "processingStateMessage" : "succeeded",
    "operation" : "sample.1",
    "diagnosticEvents" : null,
    "percentComplete" : 100.0,
    "attemptCount" : 0,
    "attemptTime" : null,
    "startedTime" : 1585867459518,
    "completedTime" : 1585867471416,
    "resultPayload" : "{\"actionData\":\"output!\"}",
    "params" : null
  } ],
  "diagnosticEvents" : null,
  "attemptsCompleted" : 1,
  "maximumAttempts" : 3,
  "startedTime" : 1585867449945,
  "completedTime" : 1585867474220,
  "percentComplete" : 100.0,
  "params" : { },
  "agendaInsight" : {
    "resourcePoolId" : "599f2241-3877-4301-b3d2-aa7e4274e42a",
    "insightId" : "547cc659-f7a3-4fbe-b171-a2d0ad355b4d"
  }
}
```
