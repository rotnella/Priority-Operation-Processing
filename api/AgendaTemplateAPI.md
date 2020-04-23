API: Agenda Template
=============================

Overview
========

API to define a template to describe the Agenda to execute. The Agenda Template is just an Agenda with variables that are expected to be filled out when creating an Agenda. For example, an Agenda Template will define the list of operations to perform. When a payload comes in, we fill out the Agenda Template with the incoming meta-data and submit the Agenda for processing.

See [API: Agenda](AgendaAPI) to see the executable Agenda from the Agenda Template.

| Payload | Agenda Template | Resulting Agenda |
|-------------|-------------|-----|
|{ "title":"xyz"} | ```{"templateParameters":{"pop.payload"=null} "agenda":{"params:{"payload":"@<pop.payload::title>"}}}``` |{"params":{"payload":"xyz"}}|

POST / PUT
==========
**< stagename >**/agendatemplate

Query Parameters
----------------
byId

byCid

byLinkId

Header
------
| Field        | Type           | Description  |
| ------------- |:-------------:| -----:|
| contentType | String | This will be application/json for the immediate term, but we might also support application/xml later |

Request Body Fields
-------------------
| Field        | Purpose           | Type  | Required Method | Notes |
| ------------- |:-------------:| -----:|:-------------:|:-------------:|
| id | Id of the original request | String | GET, PUT | |
| customerId | Id of the Customer from the customer table entry | String | POST / PUT ||
| cid | CorrelationId to combine all data objects created within a request submission | String ||
| title | An easy way to reference the template | String ||
| allowedCustomerIds | Array of customers allowed to use this template. | String\[\] | This does not allow the customer to modify this template.|
| templateParameters | The variable name you can reference from the input request payload and can be reference throughout the template | String\[\] | Currently, pop.payload is supported |
| staticParameters | Variables that are constant for all Agendas produced from this template and can be reference throughout the template | String\[\] |
| params | Map of parameters specific |||
| **agenda | See [API: Agenda](AgendaAPI) |||

Agenda Parameter Replacement
============================

Processing
----------

### Creating the Parameter Map

The parameter map is a string -> string/object mapping that combines all the template parameters and static parameters specified in the Agenda template.

#### TemplateParameters

Strings / Objects in this collection on the Agenda Template are mapped by the name of the field with a value from the input payload.

#### StaticParameters

Strings / Objects in this collection on the Agenda Template and are mapped by the name of the field with the value of the field. These are constants that can be used throughout the Agenda Template for easy referencing.

### Parameter Replacement

Using the parameter map described above the Agenda Template's agenda operations are traversed attempting to replace any substitution tokens that match an item in the map.

At this time a token is formatted as follows: **@<tokenName>**

Replacement processing is contextual based on a number of factors**:  
**

| Value in template        | Parameter Value           | Parameter Value Type  | Result | Notes |
| ------------- |:-------------:| -----:|:-------------:|:-------------:|
| "@<tokenName>" | "example text" | String | "example text"||
| "@<tokenName>" | { "datafield":"value"} | Object | { "datafield":"value"} ||
| "example:@<tokenName>" | "example text" | String | "example:example text" ||
| "example:@<tokenName>:@<tokenName>:@<tokenName>" | "3" | String | "example:3:3:3" ||
| "example:@<tokenName>" | { "datafield":"value"} | Object | "example:@<tokenName>" | (reference will not be replaced) Composite strings cannot be created  with complex objects |
| "@<sample.out>" | "operation":{"name":"sample"} sample operation's output    {"inputs"["oneinput":"xyz","twoinput":"abc"]} | Object | {"inputs":["oneinput":"xyz","twoinput":"abc"]| <operation.name>.out is a defined variable for referencing other operation's outputs.|
| "@<sample.out::/inputs[0]>" | "operation": | {"name":"sample"} | sample operation's output |{"inputs":["oneinput":"xyz","twoinput":"abc"]}| Object |{ "oneinput":"xyz"}| :: is the path separator to reference Json paths after the variable. |
| "@<sample.out::/files/duration?>" | "operation":{"name":"sample"} sample operation's output{["oneinput":"xyz","twoinput":"abc"]}| Object | null | This is an example of a default value (optional reference that will default to null). |
| "example:@<sample.out::/files/duration?0>" |"operation":{"name":"sample"} sample operation's output{"inputs":["oneinput":"xyz","twoinput":"abc"]}| String | "example:0" | This is an example of a default value (optional reference that will default to the string of characters after '?'). |

### Other items for consideration

[https://tools.ietf.org/html/rfc6901](https://tools.ietf.org/html/rfc6901) JSON Pointers are supported by Jackson using the at method.

Example Payload
===============

```{  
"id": null,  
"cid": null,  
"customerId": "http://my.account.id/data/Account/3333333",  
"allowedCustomerIds": [ ... An array of customerIds that can use this agenda template ... ],  
"title": "Accelerate",  
"templateParameters": {  
"pop.payload": null  
},  
"staticParameters": null,  
"params": null,  
"agenda": {  
"id": null,  
"cid": null,  
"customerId": null,  
"updatedTime": null,  
"addedTime": null,  
"operations": \[  
{  
"payload": {  
"originalRequest": "@<pop.payload>"  
},  
"type": "ldap",  
"id": null,  
"name": "ldap.1",  
"params": null  
},  
{  
"payload": {  
"inputs": "@<ldap.1.out::/transformRequest/inputs>"  
},  
"type": "analysis",  
"id": null,  
"name": "mediaInfo.1",  
"params": null  
},  
{  
"payload": {  
"inputs": null,  
"inputStreams": "@<mediaInfo.1.out::/inputStreams>",  
"outputStreams": null,  
"originalRequest": "@<ldap.1.out::/transformRequest>"  
},  
"type": "accelerate",  
"id": null,  
"name": "accelerate.1",  
"params": null  
},  
{  
"payload": {  
"url": null,  
"headers": null,  
"postData": "@<accelerate.1.out::/agenda>",  
"postDataEncoding": "json",  
"contentType": null,  
"requestMethod": null  
},  
"type": "agendaPost",  
"id": null,  
"name": "agendaPost.1",  
"params": null  
},  
{  
"payload": {  
"logMessages": \[  
"operationType=logHandler operationName=analysis bitrate=@<mediaInfo.1.out::/inputStreams/video/0/params/bitrate> duration=@<mediaInfo.1.out::/inputStreams/video/0/params/duration> externalFileId=@<mediaInfo.1.out::/inputs/0/params/externalFileId>"  
\]  
},  
"type": "logMessages",  
"id": "1",  
"name": "Log.1",  
"params": null  
}  
\],  
"params": {},  
"jobId": null,  
"linkId": null,  
"progressId": null,  
"agendaInsight": null  
},  
"isDefaultTemplate": false,  
"isGlobal": true,  
"global": true  
}
```
