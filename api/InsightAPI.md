API: Insight
=====================

Overview
========

An Insight is the key component for mapping an Agenda to a queue for processing. Upon Agenda submission the Resource Pool associated to a Customer is queried. A Resource Pool may have many Insights not all are visible to the Customer.

Insight is 1:1 with a Queue

See [Priority Operation Processing: 2. Scheduling](Scheduling)

POST / PUT
==========

Query Parameters
----------------

byId

byCid

byLinkId

byResourcePoolId

Header
------
|Field|Type|Description|
|-----|----|-----------|
|contentType|String|This will be application/json for the immediate term, but we might also support application/xml later|

### Request Body Fields
|Field|Purpose|Type|Notes|
|-----|----|-----------|----|
|id|Id of the original request|String|GET, PUT|
|customerId|Id of the Customer from the customer table entry|String|POST / PUT|
|cid|CorrelationId to combine all data objects created within a request submission|String|
|title|An easy way to reference|String|
|allowedCustomerIds|Array of customers allowed to use this data object.|String[]|This does not allow the customer to modify this data object.|
|Mappers|Agenda tag that must match (prep is one, not needed for exec) <br/>**Registered Map Keys:** operationName, operationType, paramMap <br/>Example: "operationName" : \[ "encode", "analysis" \]|Map<String, Set<String>|
|QueueName|SQS queue name|String|
|QueueSize|Max queue count|int|
|SchedulingAlgorithm|String|FIFO, Round Robin|
