API: ResourcePool
==========================

Overview
========

A Resource Pool is a Kubernetes cluster that pulls for full workflows called Agendas. A Resource Pool may have one or more queues it pulls from filtered by Insights.

![images/download/attachments/226435076/Fission-ResourcePool-2.jpg](images/download/attachments/226435076/Fission-ResourcePool-2.jpg)

Features
--------

Agenda Puller
-------------

*   The puller runs as a Kubernetes pod that pulls for work by calling [Agenda Service](AgendaServiceAPI) _getAgenda_

*   Creates On-Demand Kubernetes Pod for Agenda Execution


### Agenda Executor

*   Executes all operations in Agenda by spinning up resources dynamically

*   Metrics / Alerting

*   Progress reporting

*   Parallel processing

*   Node Graph / DAG model processing


### Handler

*   Executor for a single Agenda Operation

*   Metrics / Alerting

*   API owner

*   Progress reporting


Technologies
------------

*   Kubernetes

*   Kubernetes Annotations

*   Docker

*   Bananas

*   Graphite


Data Service Endpoint
=====================

POST / PUT
----------

Path: **< stagename >** /resourcepool


### Query Parameters

byId

byCid

byLinkId

### Header

|Field|Type|Description|
|-----|----|-----------|
|contentType|String|This will be application/json for the immediate term, but we might also support application/xml later|


### Request Body Fields
|Field|Purpose|Type|Notes|
|-----|----|-----------|----|
|id|Id of the original requestString|GET, PUT|
|customerId|Id of the Customer from the customer table entry|String|POST / PUT
|cid|Correlation identifier to combine all data objects created within a request submission|String
|title|An easy way to reference|String
|updatedTime|Generated upon PUT|Timestamp
|addedTime|Generated upon POST|Timestamp
|insightIds|Array of insight identifiers, generated from insight association|String\[\]

Example Payload
---------------
```
{  
"id": null,  
"cid": null,  
"customerId": "http://my.customer.url/data/Account/3333333",  
"updatedTime": 1576615017354,  
"addedTime": null,  
"title": "POPPool-dev",  
"insightIds": \[  
\]  
}
```
