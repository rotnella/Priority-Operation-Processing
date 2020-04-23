Sample Handler
==

Description
==
The sample handler is just that, an example of what a handler looks like code wise and how it operates. A handler can do anything and the input object to the handler is completely open ended.

Result
==
The sample allows you to specify a result payload (for use in subsequent operations in the [Agenda](AgendaAPI) ). This is specified in the `resultPayload` field. Like the input to your handler the output is completely open ended.

Sample Actions
==
The sample handler supports a couple of basic actions, each requiring different parameters to be populated.

Sample action parameters
--

| Parameter | Description | Associated action(s) |
| - | - | - |
| sleepMilliseconds | (optional) The number of milliseconds to sleep before performing the action | All |
| logMessage | The message to log | log |
| externalArgs | The arguments to pass to the externally launch | externalExecute |
| exceptionMessage | The exception message | exception |

Actions
--

### log
This will log the specified message.

### exception
This will cause a RuntimeException to be thrown with the exceptionMessage specified.

### externalExecute
This will attempt to launch the specified command and arguments externally (currently only Kubernetes pod launching is supported in the Sample).

Sample Payload Explained
==

:warning: These have inline comments with ## as a comment prefix... don't copy and paste this invalid json!

Payload files can be found here: `/handler/handler-sample-impl/package/local`

### payload-local.json
```
{
  "actions": [ ## Array of actions to perform
    {
      "action": "log", ## The action to perform
      "paramsMap": { ## the parameters of the action
        "sleepMilliseconds": 5000, ## causes the action to delay for 5000ms
        "logMessage": "This is a custom log message" ## the log message
      }
    }
  ],
  "resultPayload": { ## result payload to respond with
    "data": "someData"
  }
}
```

### payload.json
```
{
  "actions": [ ## Array of actions to perform
    {
      "action": "externalExecute",  ## The action to perform
      "paramsMap": { ## the parameters of the action
        "externalArgs": [ ## the array of arguments to run externally
          "ash",
          "-c",
          "ls",
          "-la"
        ]
      }
    }
  ],
  "resultPayload": { ## result payload to respond with
    "data": "someData"
  }
}
```
Execution
==

Prerequisites
--
* IDEA
* Kuberentes (minikube or otherwise) _(optional)_
  * Should go through [Run Local Execution with Minikube](RunLocalExecutionWithMiniKube)

Setup and execute a local run
--
1. Navigate to `/handler/handler-sample-impl/impl/src/main/java/com/comcast/pop/handler/sample/impl/HandlerEntryPoint.java`
1.  Create a "run" configuration for the main method.
    1.  Working directory: `(your custom path here)/pop`
    1.  Program arguments: `-launchType local -externalLaunchType local -propFile ./handler/handler-sample-impl/package/local/config/external.properties -payloadFile ./handler/handler-sample-impl/package/local/payload-local.json`
1.  Attempt to run the new configuration!

### Result
You should see the following lines (might be mixed with others)
```
11:14:22.214 [main] INFO com.comcast.pop.handler.sample.impl.processor.SampleActionProcessor - Performing Action: log
11:14:22.214 [main] INFO com.comcast.pop.handler.sample.impl.action.BaseAction - Performing requested sleep: 5000ms
11:14:27.055 [OperationProgressThread] INFO com.comcast.pop.handler.base.reporter.LogReporter - Progress: {"id":null,"cid":null,"customerId":null,"title":null,"updatedTime":null,"addedTime":null,"agendaProgressId":null,"processingState":"EXECUTING","processingStateMessage":null,"operation":null,"diagnosticEvents":null,"percentComplete":80.0,"attemptCount":null,"attemptTime":null,"startedTime":null,"completedTime":null,"resultPayload":null,"params":{"JobId":"89045f60-3a5c-41c1-ba47-9309940eb1b0"}}
11:14:27.232 [main] INFO com.comcast.pop.handler.sample.impl.action.LogAction - This is a custom log message
```

Setup and execute a kuberenetes run (requires Minikube setup)
--
1. Navigate to `/handler/handler-sample-impl/impl/src/main/java/com/comcast/pop/handler/sample/impl/HandlerEntryPoint.java`
1.  Create a "run" configuration for the main method.
    1.  Working directory: `(your custom path here)/pop`
    1.  Program arguments: `-launchType local -externalLaunchType kubernetes -propFile ./handler/handler-sample-impl/package/local/config/external-minikube.properties -payloadFile ./handler/handler-sample-impl/package/local/payload.json`
1.  Attempt to run the new configuration!

### Result
You should see the following lines (this is a small set of the lines logged)
```
11:34:51.338 [main] INFO com.comcast.pop.handler.sample.impl.processor.SampleActionProcessor - Performing Action: externalExecute
(...)
11:34:52.078 [main] INFO com.comcast.pop.handler.sample.impl.executor.kubernetes.KubernetesExternalExecutor - Getting progress until the pod pop-sample-ext9f9f525f-67b0-4b77-ad24-c3d6ddfc138b is finished.
(...)
11:34:56.289 [main] INFO com.comcast.pop.handler.sample.impl.executor.kubernetes.KubernetesExternalExecutor - External execution completed with pod status Succeeded
(...)
11:34:56.289 [main] DEBUG com.comcast.pop.handler.sample.impl.executor.kubernetes.KubernetesExternalExecutor - External execution produced: bin
dev
etc
home
lib
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```
