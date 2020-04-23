Resource Pool: Agenda Puller
=====================================

Overview
========

The Agenda Puller is responsible for getting Agendas ready to process and dynamically allocating executor pods to process for each Agenda.

Requirements
============

*   API for returning viable agendas for processing

*   getAgenda call is fast an scalable

*   Launched by Kubernetes

*   Dockerized

*   Stays alive

*   Pulls at configurable increments

*   Can run in different "modes" like the Handlers


ConfigMaps
==========


Limited Resource Handling
=========================

Agenda Insight Execution Limit
------------------------------
x
### How to use it

In your ConfigMap for the puller setup a new entry in the external-properties section: insight.execution.limit with the maximum number of Agendas that you would like to have running at the same time for the insight. Example below for a limit of 10.

`insight.execution.limit=``10`

### How it works

The puller offers a basic limitation by simply evaluating the number of executors running a given insight. This is accomplished by two major pieces of information.

1.  Each executor pod is labeled with the Insight id it is ex executing: pop.insightId

    1.  Note: We also label each pod with the ResourcePool id: pop.resourcePoolId

2.  A property to configure the limit: insight.execution.limit

    1.  The puller will not retrieve more Agendas if the limit number is met/exceeded.

    2.  This property is optional.


Not implemented: Limited Resource Polling
-----------------------------------------

Some systems we work with may have limitations on the amount of work that can be handled (Elemental for example). These may be due to license and or resource limits.

To work within these limits the Puller must not take additional work when the limit is met. An additional component will be responsible for persisting a value within Kubernetes that the Puller can use to decide if more work should be retrieved.

![images/download/attachments/226434733/PullerLimitedResource.svg](images/download/attachments/226434733/PullerLimitedResource.svg)

A rough plan is as follows:

1.  Pod is launched on an interval that evaluates the state of the limited software

2.  Persist the current value (either available slots or some other numeric)

3.  Puller evaluates the value from Kubernetes (this and how to check against the value are configurable)

4.  Puller pulls work if the limit is not met


We will likely need to maintain a data store. We could use kubernetes but we would need to select an object to persist to (annotations). Or launch and maintain a data store (likely a simple NoSQL service like MongoDB with credentials).

### NoSQL Solution

The values for the state of the limited resource are persisted in a NoSQL server.

Requirements:

*   Backup is not needed (all data could be lost and it would be repopulated anyway)

*   Accessible by the puller and the pod responsible for evaluating the limited resource

*   Built into the deployment process for a new ResourcePool setup

*   1 Instance is needed per Pool (multiple pullers can use it)

*   Handling of minimal I/O


#### Issues

Depending on how limited resource handling is designed/implemented these may be easily avoided.

*   Due to timing the puller could pull down any number of Agendas and start them because the limited resource operation does not take place until some time into the Agenda processing.

*   If an Agenda requires multiple instances of a limited resource the resource may be overloaded

    *   Both parallel and sequential usage have their own issues


### Alternatives

Additionally we may want to offer other methods of limited resource handling. These are some quick descriptions.

*   Over simplify: Simply limit the number of executors that can run for the given insight (this could be an easily queryable value against the Kubernetes cluster)

*   GetAgenda: Avoid pulling more Agendas at the API level based on an expanded request:

    *   Limited operation name

    *   Slots available


Dropwizard Logback.xml Usage
============================

Kubernetes / Docker execution
-----------------------------

The puller service/deployment is a special case due to the use of Dropwizard. The logback.xml is not natively supported with dropwizard. Instead the config yaml is generally how logging is configured with Dropwizard. To continue using the logback.xml requires the implementation of a LoggingFactory (in the Puller project this is LogbackAutoConfigLoggingFactory). An instance of this factory must be returned by the Configuration object dropwizard is run with (overriding Configuration.getLoggingFactory).

### Log Level

I think there's a few layers of conversion and mismatch here.

LOG\_LEVEL is setup as an environment var.

The agent docker image start.sh uses that to setup -DLogLevel=${LOG\_LEVEL}

But... the logback.xml has <root level="${LOG\_LEVEL:-INFO}"> (this is a system property lookup, not an environment variable)  

The log level (specifically LOG\_LEVEL env var) cannot be controlled in the env.properties file as a direct value. Instead it should be specified for use in JAVA\_EXTRA as follows:

`JAVA_EXTRA=``"$JAVA_EXTRA -DLOG_LEVEL=INFO"`

### References

[https://github.com/dropwizard/dropwizard/issues/1567](https://github.com/dropwizard/dropwizard/issues/1567)

[https://gist.github.com/fedotxxl/0b3cc5e5e4eaeffdcde1f9834796edc6](https://gist.github.com/fedotxxl/0b3cc5e5e4eaeffdcde1f9834796edc6)

Local Execution
---------------

When running locally you can specify the **\-Dlogback.configurationFile** command line argument (VM option, not an application argument) with a value indicating the path of the logback.xml to use. The sample command line arguments in the PullerEntryPoint main comments include this. The project includes a logback\_local.xml to allow for _**exception-free**_ local execution (the exceptions from attempting to syslog to non-existent urls).

Also you may optionally use this VM option (assuming running via IDEA) to see the logback specific messages:

`-Dlogback.statusListenerClass=ch.qos.logback.core.status.OnConsoleStatusListener`

### References

[https://stackoverflow.com/questions/21885787/setting-logback-xml-path-programmatically](https://stackoverflow.com/questions/21885787/setting-logback-xml-path-programmatically)
