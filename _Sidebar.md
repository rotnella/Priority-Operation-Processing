Priority Operation Processing
====================
1. [Submission](Submission)
2. [Scheduling](Scheduling)
    - [Binding Agenda to a Queue](BindingAgendaToQueue)
3. [Execution](Execution)\
_the ResourcePool_
    - [Puller](Puller)
    - [Executor](Executor)
    - [Handler](Handler)
        - Pod Handler
            - [Sample](SampleHandler)
        - Executor Handler
            - [Http Caller](HttpHandler)
            - [Logger](HttpHandler)

Data Object API
====================
[Agenda](AgendaAPI)\
_the workflow_ \
[Agenda Template](AgendaTemplateAPI)\
_the workflow definition_ \
[Customer](CustomerAPI)\
[Insight](InsightAPI)\
_the scheduling queue definition_ \
[Operation Progress](OperationProgressAPI)\
_the state of the running Agenda operations_ \
[Progress](ProgressAPI)\
_the state of the running Agendas_ \
[ResourcePool](ResourcePoolAPI)\
_the processing resources_

Service Agenda
====================
[Agenda Service](AgendaServiceAPI)\
_the workflow submission_ \
[Progress Service](ProgressSummaryAPI)\
_rolled up agenda progress summary_ \
[ResourcePool Service](ResourcePoolServiceAPI)\
_getting work and updating progress_


Timed Processes
====================
[AgendaReclaimer](AgendaReclaimer)\
_restarting stuck Agendas_ \
[AgendaRetry](AgendaRetry)\
_retrying failed Agendas_ \
[DataObjectReaper](DataObjectReaper)\
_reaping expired data objects_ \
[PodReaper](PodReaper)\
_reaping stuck Kubernetes pods_

Installation
====================
[Install](Install)
- [Install AWS Components](InstallAWSComponents)
- [Install AWS Roles](InstallAWSRoles)
- [Install Kubernetes Components](InstallKubernetesComponents)

Development
------------
[DevKubernetesSetup](DeveloperKubernetesSetup)\
[RunLocalExecution](RunLocalExecution)\
    - [RunWithMiniKube](RunLocalExecutionWithMiniKube)

Demo / Examples
------------
[SampleAuthorizer](SampleAuthorizer)\
[SampleHandler](SampleHandler)
