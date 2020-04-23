API: Customer
======================

Data Service Endpoint
=====================

GET/POST/PUT/DELETE
-------------------

Path: **< stagename >** /customer

### Query Parameters

byId

byCid

byLinkId

byResourcePoolId

### Header

|Field|Type|Description|
|-----|----|-----------|
|contentType|String|This will be application/json for the immediate term, but we might also support application/xml later|

### Request Body Fields
|Field|Purpose|Type|Notes|
|-----|----|-----------|----|
|id|Id of the Customer|String|**This must match the customerId or things don't behave well**|
|customerId|Id of the Customer from the customer table entry|String||
|cid|CorrelationId to combine all data objects created within a request submission|String||
|title|An easy way to reference|String||

Example Payload
---------------
```
{  
"id": "http://my.account.id/data/Account/3",  
"cid": null,  
"customerId": "http://my.account.id/data/Account/3",  
"updatedTime": 1576608440403,  
"addedTime": 1576608440403,  
"billingCode": null,  
"title": "POP-Rocks",  
"resourcePoolId": "bcd58865-6280-4922-a2c7-527f3a712cee"  
}
```
