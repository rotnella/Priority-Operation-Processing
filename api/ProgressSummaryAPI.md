API: Progress Summary
======================

Web Service Methods
===================

ProgressSummary
---------------

### POST

### /progress/agenda/{id}


Request
-------
|Field|Type|Values|Notes|
|-----|----|-----------|----|
|linkId|String|The linkId to get the AgendaProgress objects by|

Response
--------

This is necessary as our jobs may consist of multiple Agendas. Each agenda will have an AgendaProgress. This object will be constructed on the fly (not actually persisted anywhere).  

|Field|Type|Values|Notes|
|-----|----|-----------|----|
|id|String|
|processingState|Enum|waiting<br/>* ** added<br/>** scheduled<br/>** queued<br/>** executing<br/>*complete<br/>** failed<br/>** succeeded|This is a rollup across all Agendas with the same linkId|
|AgendaProgressSet|AgendaProgress\[\]|
