API: Agenda Progress
======================

Data Service Endpoints
======================

Agenda Progress
---------------

This is the parent object for progress/status tracking as related to any Agenda.  

### /progress/agenda/{id}

### Header

|Field|Type|Description|
|-----|----|-----------|
|contentType|String|This will be application/json for the immediate term, but we might also support application/xml later|

### Request Fields
|Field|Type|Notes|
|-----|----|-----------|
|id|String|
|cid|String|
|customerId|String|
|agendaId|String|
|linkId|String|Multiple AgendaProgress objects may have the same linkId|
|externalId|String|(optional) External identifier|
|title|String|Type of Agenda|
|processingState|Enum|waiting<br/>* ** added<br/>** scheduled<br/>** queued<br/>** executing<br/>*complete<br/>** failed<br/>** succeeded|
processingStateMessage|String|Percents, sub states, whatever|
|operationProgress|OperationProgress\[\]|See [Operation Progress](OperationProgressAPI)|
|updatedTime|Timestamp|
|addedTime|Timestamp|Time the Agenda was added|
|startedTime|Timestamp|Time the Agenda started|
|completedTime|Timestamp|Time the Agenda was completed|
|percentComplete|Double|Rolled up _computed from the ops_. Gathered by combining the ops|
|params|ParamsMap|Free form map of parameters._May eventually contain a copy of the AgendaParameters (TBD)_|

### Creation

Created when a payload is submitted.

When Agendas are submitted directly without a template we will need to go through the same creation process and response (to include the progress details).

### Removal

Reaped (or possibly ttl)
