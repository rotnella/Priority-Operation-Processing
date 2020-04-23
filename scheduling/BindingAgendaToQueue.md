Scheduling: Routing Agendas to Resource Pools
======================================================

1.  A Resource Pool needs added via [API: ResourcePool](ResourcePoolAPI)

2.  An [Insight](InsightAPI) needs added to bind types of Agendas to be processed by the [Resource Pool's Agenda Puller](Puller)

3.  A customer entry in the [Customer](CustomerAPI) table needs to set resourcePoolId to the configured Resource Pool entry

4.  An [Agenda Puller](Puller) needs to be configured and deployed to get Agenda's by the Insight.id configured above.


How to Routing Agendas For Scheduling and Execution
===================================================

In the Customer database, the entry associated with each Customer has a Resource Pool ID in it. That is the Resource Pool that the Agenda will run on.

Here is an example Customer entry:

```
{  
"id": "http://myaccount.url/2708244013",  
"cid": null,  
"customerId": "http://myaccount.url/2708244013",  
"updatedTime": 1576864676937,  
"addedTime": 1576864676937,  
"billingCode": null,  
"title": "Dev Playback Experiment",  
"resourcePoolId": "599f2241-3877-4301-b3d2-guid"  
}
```
Here is the entry associated with that resourcePoolId:

```
{  
"id": "599f2241-3877-4301-b3d2-guid",  
"cid": null,  
"customerId": "http://resource.pool.account/2707858829",  
"updatedTime": null,  
"addedTime": null,  
"title": "POPPool-AOR",  
"insightIds": \[  
"3c85d43e-5cf8-4b82-9e57-guid",  
"522d6c52-546a-4d2e-9f6c-guid2",  
"9d13d89d-5726-457d-9b28-guid4"  
\]  
}
```
The insightIds indicate what type of work the Resource Pool is able to do (and some other useful info, too). Here is an example Insight entry to define the queueing.
```
{  
"id": "9d13d89d-5726-457d-9b28-dsfwt",  
"cid": null,  
"customerId": "http://resource.pool.account/2707858829",  
"updatedTime": 1575491153681,  
"addedTime": null,  
"allowedCustomerIds": null,  
"resourcePoolId": "599f2241-3877-4301-b3d2-guid",  
"queueName": "My-SQS-unique-queue-name",  
"queueSize": 20,  
"schedulingAlgorithm": "FirstInFirstOut",  
"mappers": {  
"operationType": \[  
"transcode"  
\]  
},  
"title": "POPPool-AOR-Transcode-Queue",  
"isGlobal": true,  
"global": true  
}
```
