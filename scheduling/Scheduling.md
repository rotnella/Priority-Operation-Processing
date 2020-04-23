Step 2: Scheduling
========================

Supporting Documentation
========================

*   [Priority Operation Processing: API: ResourcePool](ResourcePoolAPI)

*   [Priority Operation Processing: Resource Pool: Agenda Puller](Puller)


Scheduling
----------

The Priority Operation Processing (POP) Brain is responsible for authentication, data submission (CRUD), data storage, client callbacks and scheduling workflows.

### Features

*   Authorized Customer Data Visibility

*   API Endpoints

    *   [Agenda](AgendaAPI)

    *   [Progress](ProgressAPI)

    *   [Resource Pool](ResourcePoolAPI)

        *   [Insights](InsightAPI)

    *   [Customer](CustomerAPI)

*   Scheduling

*   Data Management

*   Metrics, Alerts

*   linkId for connecting all Data Objects

*   Agenda Retries

    *   Configurable

    *   Retries at the point of failure



### Technologies

*   API Gateway

*   AWS Authorizer

*   Lambda

*   SQS

*   Dynamo

*   CloudFormation

*   CloudWatch



Binding an Agenda to an Insight/Queue for processing
====================================================

A [ResourcePool](ResourcePoolAPI) (Kubernetes farm consuming work) will identify the type of work the resource pool can consume based on a set of agenda insights / queue definitions.

Priority Operation Processing (POP) will schedule pending work for each Insight / Queue.

Resource Pool:
--------------

A puller to pull for work and resources to process the work (typically a Kubernetes farm)

Insight:
--------

A queue definition for the type of work to be pulled

Steps for scheduling:
---------------------

1.  Gather next customers in line that have work for each Resource Pool's insight

2.  Add list of customers to the appropriate queue defined by insight

3.  Watch for work exhaustion and refill queue


Example Resource Pool's Insights
--------------------------------
| Insight Key        | Value         
| ------------- |:-------------:|
| Execute | operationType="mediaInfo" |


Example Agenda
--------------
```
{
"operations":
[
{
"payload":"xyz",
"type":"transcode"
}
]
}
```
The Agenda in the above example with map to the Insight "Execute" defined for the Resource Pool above. The puller will get next Agendas from the queue by insightId once they are scheduled.

How does it work?
=================

Adding new Agendas (customer)
-----------------------------

1.  Customer submits either a payload with an AgendaTemplate.id or a ready to run Agenda

2.  The agenda is mapped to an Insight for the Resource Pool.

3.  A new Ready Agenda entry is created with the AgendaId, CustomerId, InsightId, and a state of **available**

4.  The Agenda is considered available and is idle


Scheduling (background processing)
----------------------------------

1.  Queues are monitored for item counts (if low/empty)

2.  Fairness is used to select which customers with items in Ready Agenda will be given a chance to queue

3.  The filter for the given customer and insight is applied to select which Agendas should be queued for the customer

4.  All selected items are queued (within the limits of the queue/fairness)

5.  Items in Ready Agenda are moved to queue


Pulling available Agendas (puller)
----------------------------------

1.  Puller requests work by insightId

2.  The next Agenda in queue for that Insight is returned (or more if more were requested)
