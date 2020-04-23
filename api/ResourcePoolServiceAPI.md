API: ResourcePool Service
==========================

Web Service Endpoint
====================

General
-------

All web method calls are POST unless specified.

Method: getAgenda
-----------------

Path: **< stagename >** /resourcepool/service/getAgenda

### POST Data

Internally the class is called a GetAgendaRequest.

### Field
|Field|Purpose|Required|Notes|
|-----|-------|--------|-----|
|insightId|The id of the insight to retrieve Agendas for|Y
|count|Indicator of the number of Agenda objects to retrieve|Y

### Functionality

This method is used to retrieve one or more Agendas associated with the insightId specified.
The getAgenda API is intended for use by a ResourcePool. A ResourcePool (puller in this case) calls getAgenda to get a list of viable agendas to process. For an agenda to be viable it must adhere to the requested insight. The getAgenda call will return agendas meeting the constraints for customers that are next in line. This should be pre-sorted so it's a quick "pull off bin" (de-queue).

Order of Operations
-------------------

*   getAgenda is called by puller

*   The ResourcePool+Insight are confirmed as valid for the user

*   The insight is mapped to a queue of objects containing an agendaId

*   The number of necessary agenda objects are returned


Each ResourcePool will have a resource pool access ID that is registered with POP. Customers are associated to a specific resource pool. We determine this via the customerID and insightID on the call.

### Visibility
@todo add documentation

#### Users

#### Processing

When the Agenda is POST'd it is associated with a ResourcePool and Insight. The ResourcePool id will need to be persisted on the Progress related objects (for use with the updateAgendaProgress).

Method: updateAgendaProgress
----------------------------

Path: **< stagename >** /resourcepool/service/updateAgendaProgress

### POST Data

An AgendaProgress object

#### Required Fields

*   id


### Functionality

This method is used to update an AgendaProgress and any associated OperationProgress objects.

### Visibility

#### Users

#### Processing

Before the progress related objects are updated the following are confirmed:

1.  Grab the customerId

2.  Get the AgendaProgress by ID

3.  Get the ResoucePool from AgendaProgress.resourcePoolID

4.  If the ResourcePool.customerID matches the allowedAccount list, then we OK the progress update. If not, we throw an authorize exception.

5.  If the ResourcePool is global, we OK the progress update.

6.  If the user is a service user or super user, we OK the progress update.
