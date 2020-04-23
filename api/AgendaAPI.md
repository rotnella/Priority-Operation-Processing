API: Agenda
====================

Overview
========

API to define the workflow to execute.

Features
--------

*   Simple list of operations

*   Node Graph / DAG model

*   Variable referencing between operations

*   Non-strict API for inputs / outputs of each operation

*   Agenda decoupling from underlying execution technologies / APIs. Each operation has a JSON payload that adheres to the API of the POP Handler that will perform the op.

*   The Agenda [Executor](Executor) will immediately run an operation as long as dependency variables are met

*   The Agenda Executor will look up Docker image by operation type and spin up a new Kubernetes Pod to for the operation


Data Service Endpoint
=====================

POST
----

Path: /<stagename>/agenda

### Query Parameters
byId

byCid

byLinkId

byResourcePoolId

### Header
| Field        | Type           | Description  |
| ------------- |:-------------:| -----:|
| contentType   | String | This will be application/json for the immediate term, but we might also support application/xml later |

### Request Body Fields
| Field        | Purpose        | Type  | Notes |
| ------------- |:-------------:| -----:| -----:|
| id | Id of the original request | String| |
| params | Map of parameters specific | Map<String, String> |
| **operations** | The collection of operations to perform | Operation\[\] | See operation definition |
| **.operation** | Operation to perform | Operation | 1:1 with a POP Handler |
| ..payload | The payload to pass to the operation handler | Object | This should never be a String for the sake of _**sanity**_. |
| ..type | The type of operation | String | Examples: encode, analysis |
| ..id | Unique identifier | String||
| ..name | Unique identifier, specifically used to allow reference to the output of this operation | String ||
| ..params | Map of parameters for the operation to consume | Map<String, String> ||

PUT
---

This operation is not supported.

Example Payload
---------------

```{
"id": "9630ff37-668a-4ca0-81aa-d1d1ea2564ee",
"customerId": "http://mycustomer.url/Account/2708467301",
"updatedTime": 1583346692310,
"addedTime": 1583346692310,
"operations": \[
{
"payload": {
"originalRequest": {
"id": "551964cf-f850-4e99-a384-b9154ec342c7",
"customerId": "http://mycustomer.url/Account/2708467301",
"updatedTime": 1583346691731,
"addedTime": 1583346691731,
"inputs": \[
{
"type": "video",
"url": "fasp://aspera.asg.com:33001/POP/small.mp4",
"credentials": {
"password": "REMOVED",
"username": "REMOVED"
},
"params": {
"owner": "http://mycustomer.url/Account/2708467301",
"extension": "mp4",
"externalFileId": "http://my.metadata.url/MediaFile/1706446403796",
"airdate": 1583346166000,
"language": "en",
"bitrate": 551194,
"title": "my test",
"audioCodec": "AAC LC",
"audioSampleRate": 48000,
"displayAspectRatio": "16:9",
"duration": 5533,
"frameRate": 30,
"audioChannels": 1,
"size": 383631,
"checksum": "A3AC7DDABB263C2D00B73E8177D15C8D",
"width": 560,
"contentType": "video",
"expirationDate": 1583432566000,
"height": 320,
"videoCodec": "AVC"
}
}
\],
"linkId": "http://my.data.link.id/someguid",
"agendaTemplateTitle": "Template that was used",
"params": {
"cacheSourceFiles": true,
"progressId": "6142eb9b-ca77-49cb-8e53-30e48aa38ae4",
"externalId": "http://external.id.i.can.use/31127938796115",
"serviceUserName": "admin"
},
"externalId": "http://external.id.i.can.use/31127938796115"
}
},
"type": "firstop",
"name": "firstop.1"
}
\],
"params": {
"externalId": "http://external.id.i.can.use/31127938796115"
},
"jobId": "551964cf-f850-4e99-a384-b9154ec342c7",
"linkId": "http://external.id.i.can.use/31127938796115",
"progressId": "6142eb9b-ca77-49cb-8e53-30e48aa38ae4",
"agendaInsight": {
"resourcePoolId": "2b718927-22a8-4427-89a7-41326826758e",
"insightId": "4beabfcc-30f3-4b16-9c6f-073e9498e8fa"
}
}
```

Parameters
----------

These are all optional params map entries

| Parameter        | Function        | Notes  |
| ------------- |:-------------:| -----:|
| defaultInsight | Attempts to use the specified insight (by title or by id). This applies only if the specified insight can be found. ||
| doNotRun | Flag indicating this Agenda is not for execution. It will not be scheduled/queued. ||
| maximumAttempts | The maximum attempts override for an Agenda (applies internally to the AgendaProgress) ||
