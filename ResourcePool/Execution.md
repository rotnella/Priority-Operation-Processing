Step 3: Execution
=====================

Priority Operation Processing: API: ResourcePool

Overview
--------

A [Resource Pool](ResourcePoolAPI) is a Kubernetes cluster that pulls for full workflows called Agendas. A Resource Pool may have one or more queues it pulls from filtered by Insights.

![images/download/attachments/226435076/Fission-ResourcePool-2.jpg](images/download/attachments/226435076/Fission-ResourcePool-2.jpg)

### Features

### Agenda Puller

*   The puller runs as a Kubernetes pod that pulls for work by calling [Agenda Service](AgendaServiceAPI) _getAgenda_

*   Creates On-Demand Kubernetes Pod for Agenda Execution


#### Agenda Executor

*   Executes all operations in Agenda by spinning up resources dynamically

*   Metrics / Alerting

*   Progress reporting

*   Parallel processing

*   Node Graph / DAG model processing


#### Handler

*   Executor for a single Agenda Operation

*   Metrics / Alerting

*   API owner

*   Progress reporting


### Technologies

*   Kubernetes

*   Kubernetes Annotations

*   Docker

*   Bananas

*   Graphite
