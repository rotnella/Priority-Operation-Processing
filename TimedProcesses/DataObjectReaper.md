Timed Process: Data Object Reaper
===========================

General
=======

We have an AWS lambda that reaps objects on an interval with special settings. There is one cloudwatch scheduled event per table reaper. Each table may require special settings.

Source Code
===========

Source
------

Cloudformation Templates
------------------------


Parameters
==========
|Field|Type|Required|Default|Description / Notes|
|-----|----|--------|-------|-------------------|
|maximumExecutionSeconds|int|N|60|The maximum amount of time the execution should run for. This will not result in an absolute exit at the specified time (fuzzy).**This time should be less than the lambda's own execution timeout!**|
|tableName|String|Y||The name of the table to scan/delete items from|
|idFieldName|String|Y|||The name of the id field in the table|
|deleteCallDelayMillis|long|N|0|The number of milliseconds to delay between delete calls (dynamo has capacity limits to contend with).|
|logDeleteOnly|Bool|N||Flag indicating if the delete actions should only be logged|
|reapAgeMinutes|int|Y||The age in minutes an object must be for reap consideration.Reap objects where: <br/>now - reapAgeMinutes > timeField value|
|scanDelayMillis|long|N|0|The number of milliseconds to delay between scan calls (dynamo has capacity limits to contend with).|
|targetBatchSize|int|N|50|The number of objects to attempt to gather before attempting to delete.|
|objectScanLimit|int|N|50|The number of objects to allow dynamo to scan before returning (may return 0 to objectScanLimit items)|
|timeFieldName|String|Y||The name of the time field to read to determine if the object should be reaped. If the field is undefined the numeric comparison will not pass and the object will not be selected for reaping.There is no delete batch size because Dynamo has a fixed batch size of 25. We fill the items to delete up to this limit with each delete attempt.|


Sample Cloud Watch Json
-----------------------
```
{
"deleteCallDelayMillis":500,
"idFieldName":"id",
"maximumExecutionSeconds":120,
"objectScanLimit":25,
"reapAgeMinutes":4320,
"scanDelayMillis":500,
"tableName":"POP-Agenda-dev",
"targetBatchSize":50,
"timeFieldName":"updatedTime"
}
```
Log Messages
============

As of this writing here are some of the interesting log messages to look for in logs (see code for others):

BatchedReapCandidatesRetriever
------------------------------

`Attempting to scan POP-ProgressAgenda-dev` `for` `approximately` `50` `items`

`Scan of POP-ProgressAgenda-dev produced` `0` `items`

BatchedDeleter
--------------

`Attempting to delete` `2` `items from POP-ProgressAgenda-dev`

`Deleting the following ids from POP-Agenda-dev: 52ae4b71-740f-40cc-``9098``-1232b9f0c549,cb6d539a-e487-``4777``-b789-a6f4b5430316`
