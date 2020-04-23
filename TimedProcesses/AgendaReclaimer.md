Timed Process: Agenda Reclaimer
=========================

General
=======

Progress monitoring will be performed inside a Lambda.

Source
------


Environment Vars
================

All environment variables are required.

| Variable| Description / Notes|
|---------|--------------------|
|IDENTITY\_URL|The URL of the identity instance to use.|
|IDM\_USER|The IDM user to use|
|IDM\_ENCRYPTED\_PASS|The encrypted password. (assumes a KMS encrypted PasswordAES string)|
|AGENDA\_PROGRESS\_URL|The URL of the AgendaProgress endpoint|

Parameters
==========

|Field|Type|Required|Default|Description / Notes|
|-----|----|--------|-------|-------------------|
|maximumExecutionSeconds|int|N|60|The maximum amount of time the execution should run for. This will not result in an absolute exit at the specified time (fuzzy).**This time should be less than the lambda's own execution timeout!**|
|tableName|String|Y||The name of the table to scan items from|
|idFieldName|String|Y||The name of the id field in the table to scan|
|timeFieldName|String|Y||The name of the time field in the table to scan|
|reclaimAgeMinutes|int|N|0|The age in minutes an object must be for reclaim consideration. Reclaim objects where:<br/>*   ProcessingState = Executing <br/>*   now - reclaimAgeMinutes > timeField value|
|scanDelayMillis|long|N|0|The number of milliseconds to delay between scan calls (dynamo has capacity limits to contend with).|
|targetBatchSize|int|N|50|The number of objects to attempt to gather before attempting to reclaim.|
|objectScanLimit|int|N|50|The number of objects to allow dynamo to scan before returning (may return 0 to objectScanLimit items)|
|logReclaimOnly|bool|N|false|Flag indicating if the reclaim actions should be logged only|

Sample Cloudwatch Json
----------------------
```
{
"maximumExecutionSeconds":60,
"agendaProgressEndpointURL":"https://www.myurl.com",
"idFieldName":"id",
"maximumExecutionSeconds":120,
"objectScanLimit":25,
"reclaimAgeMinutes":4320,
"scanDelayMillis":500,
"tableName":"POP-ProgressAgenda-dev",
"targetBatchSize":50,
"timeFieldName":"updatedTime"
}
```

Progress Timeout Fail
=====================

General
-------

When the updated time on an incomplete (started) AgendaProgress passes some threshold we should mark the work as failed (with an indicator of timeout).

Consider adding in a response/evaluation so if an executor does somehow come back to life it knows the progress was failed due to timeout.

Evaluation
----------

Check for EXECUTING AgendaProgress objects with an updateTime < threshold time.

Action
------

Mark the AgendaProgress as follows:

*   ProcessingState: Complete

*   ProcessingStateMessage: failed (CompleteStateMessage)

*   Add a DiagnosticEvent describing the timeout and associated details.


Re-evaluate the TaskCallback evaluation of a failed item. Make sure it is not only scanning the OperationProgress elements for failure and DiagnosticEvents.

The associated Task should be marked as failed.

Progress Timeout Reclaim / Retry
================================

This is an extended implementation of the progress timeout above.

Related: [Agenda Retry](AgendaRetry)

General
-------

When the updated time on an incomplete (started) AgendaProgress passes some threshold we should attempt to restart the work.

OR

If an Agenda fails attempt to retry it.

Evaluation
----------

Check for AgendaProgress objects that fail/timeout.

### Timeout

Retry if:

*   Executing AgendaProgress objects with an updateTime < threshold time.

*   De-queued AgendaProgress objects with an updateTime < threshold time ( **this state does not exist yet** ).


### Failure

Retry if:

*   AgendaProgress indicates failure

*   If some number of attempts have not been made yet (implies a counter is on the AgendaProgress object)

*   Failure is retryable (this requires the handlers and executor to provide some hints)


Action
------

*   Reclaimer

    *   Update the Agenda Progress to indicate it is a retry attempt

    *   Re-add to ready-agenda (should have some kind of indicator of priority to jump past other work when the queue is next populated)

*   Executor

    *   Pull the existing Agenda Progress for existing operation progress (is there a flag indicating this?)

    *   Populate the in-memory payloads for the completed operations

    *   Restart any failed operations (is there any information passed to the handler? – we should for the sake of things like jobId)

*   Handler

    *   Detect existing progress and proceed accordingly


Development / Issues To Resolve
-------------------------------

*   There is no AgendaProgress ProcessingState for "removed from queue but not yet started" - for the sake of timeout this gap should be covered.

*   The first iteration could just be run the Agenda again vs. attempting to continue from the point of failure (or be parallel efforts).
