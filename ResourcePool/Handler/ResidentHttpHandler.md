Resource Pool: HTTP Handler
===============================

Execution
=========

A resident handler runs within the Executor scope. For Kubernetes the operation will run within the same pod and same Docker image as the Executor. Whereas Handlers are run as separate pods / images and have their own memory & CPU constraints.

_Note: Be mindful of memory & CPU constraints when deploying your Executor pod_

Multi-Purpose Handler
=====================

The HTTP Request handler also provides the HTTP Agenda Post handler functionality (posts the body to the Agenda endpoint).

Source Control
==============

Execution / Testing
===================

Local
-----

Main class: HandlerEntryPoint

This class has a set of command line arguments for running locally. See the comments on the main method for any requirements.

Inputs
======

HttpRequestHandlerInput
-----------------------

The AgendaPost handler has reduced capabilities focusing specifically on posting an Agenda.

|Field|Type|AgendaPost Supported|Notes|
|-----|----|--------------------|-----|
|url|String|No
|headers|ParamMap|Yes
|postData|Object|Yes
|postDataEncoding|String|Yes|**TODO: this should be postDataFormat** base64 , json, ""(blank - passthrough)
|contentType|String|No
|requestMethod|String|No

### postDataEncoding

The postData may be submitted in multiple forms.

|postDataEncoding|Expected Input Object|Notes|
|----------------|---------------------|-----|
|base64|Base64 Encoded String|
|json|Map<String, Object> (generally whatever json would store an object as in its generic form)|"" (blank) NOT NULL|

Outputs
=======

HttpRequestHandlerOutput
------------------------
|Field|Type|Notes|
|-----|----|-----|
|responseCode|Integer|HTTP response code|
|responseBody|String|The raw response body|

Configuration
=============

Managed currently in the external.properties of the executor.
