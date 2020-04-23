API: OperationProgress
======================

### /progress/agenda/operation/{operation id}

### Header
|Field|Type|Description|
|-----|----|-----------|
|contentType|String|This will be application/json for the immediate term, but we might also support application/xml later|

### Request Fields
|Field|Type|Value|Notes|
|-----|----|-----------|-----|
|id|String|
|operation ID||(Part of EndpointDataObject) recommend this be the id of the AgendaProgress + operation name.|
|cid|String|(Part of EndpointDataObject)|
|customerId|String|(Part of EndpointDataObject)
|agendaProgressId|String|AgendaProgress id If this is a separate object|
|processingState|Enum|waiting<br/>* ** added<br/>** scheduled<br/>** queued<br/>** executing<br/>*complete<br/>** failed<br/>** succeeded|
|processingStateMessage|String|Percents, sub states, whatever|
|operation|String|Workflow operation (ie: Analyze, Encode, Packaging). This is the operation.type field value|
|diagnostics|OperationDiagnostic\[\]|Detailed information of operation attempts|
|attemptCount|Integer|Total number of attempts (includes the first attempt that failed and the current attempt being reported)|
|attemptTime|Timestamp|UTC When did the current attempt take place|
|startedTime|Timestamp|Time the operation started (inclusive of all retries)|
|completedTime|Timestamp|Time the job phase was completed|
|resultPayload|Object|The resulting payload||
|params|ParamsMap|Free form map of parameters.|

### Creation

Created when an Agenda is submitted (per operation listed).

### Removal

Reaped (or possibly ttl)

Progress API Objects / Enums
============================

We will need customerIds on everything to isolate objects from being accessed by the wrong user.

ProcessingState
---------------
Enumeration describing the state of the given item (Operation, Agenda, whatever).

|Value|Definition|Messages (examples)|
|-----|----------|-------------------|
|waiting|waiting to start (pods or dependencies pending)|
|added|queued|
|scheduled
|executing|actively processing
|percent complete values|
|complete|
|completed| (failed or succeeded)|
|failed|
|succeeded|
