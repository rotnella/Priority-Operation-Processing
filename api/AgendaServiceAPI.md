API: Agenda Service
====================

Web Service Endpoint
====================

General
-------

All web method calls are POST unless specified.

Endpoint : Run
-----------------

Takes a JSON payload and an agendaTemplateId and creates an Agenda ready for scheduling and execution.

See [API: Agenda Template](AgendaTemplateAPI)

### POST

Path: /<stagename>/agenda/service/run

Internally the class is called a RunAgendaRequest.

### Request Body Fields
| Field        | Purpose        | Required  | Notes |
| ------------- |:-------------:| -----:| -----:|
| payload | The payload to submit for transformation using the AgendaTemplate | Y | The parameter in the template is **pop.payload** |
| agendaTemplateId | The id of the AgendaTemplate to use with the payload as input | Y |

### Functionality

This method is used to create an Agenda by combining the payload and template.

### Visibility

@todo docs for visibility

Endpoint : Rerun
-------------------

Takes an agendaId and will rerun/retry an agenda based on specified state.

### Rerun Feature Support

*   Offer ability to reset the state of operations so execution can continue as necessary (mixing the following as needed by the caller):

    *   Resume from last failed operation

    *   Resume from last pending operation

    *   Restart from the beginning operation. All operations are restarted

    *   Force reset of specified operations

*   Skip operations in a specified state (paused, failed, succeeded) Why would we support skipping failed operations? If this is needed the executor will need to change to support it. Paused is not designed/ implemented.


### Rerun Context for Operations

Each operation will receive their previous OperationProgress in order to make decisions on how to proceed.

#### Use Case

An operation may have lost connection and upon rerun it first tries to regain its previous connection before restarting. A jobId for Elemental is an example of data that may be made available across executions.

### POST

Path: /<stagename>/agenda/service/rerun

Internally the class is called a RerunAgendaRequest.

### Request Body Fields

| Field        | Purpose        | Required  | Type | Notes |
| ------------- |:-------------:| -----:| -----:| -----:|
| agendaId | The id of the Agenda to run | Y | String |
| params | List of optional parameters (see below) | N | String\[\] |

#### Parameters

These allow us to expand the capabilities of the endpoint without changing the API. Some parameters are flags while others have one or more values. If a value consists of multiple items it is comma separated.

| Parameter        | Purpose        | Has Value?  | Value Definition | Value Example | Notes |
| ------------- |:-------------:| -----:| -----:| -----:| -----:|
| resetAll | Flag indicating that all operation progress should be reset | N | N/A | Default action |
| operationsToReset | Provide a list of operations by name to reset. | Y | Multi | `=test.``1``,test.``2` | The operation name in the Agenda is needed, not the full OperationProgress id. |
| skipExecution | Flag indicating to perform all the specified resets but to not submit the Agenda for execution | N | N/A |
| continue | Flag indicating to only resubmit the Agenda for execution. | N | N/A |

This will default to how the Executor handles the existing state of the AgendaProgress and OperationProgress.

#### On reset of an AgendaProgress

*   ProcessingState: WAITING

*   ProcessingStateMessage: pending


#### On Reset of an OperationProgress

*   ProcessingState: WAITING

*   ProcessingStateMessage: pending


#### Executor Rerun Handling

The executor by default will attempt to run any operations that were not completed successfully according to the AgendaProgress (never run, partial progress, or failed). No specific progress reset is needed for this as of this writing.

The only critical information for the executor are the following at startup:

*   The payloads of completed+successful operations (provides the inputs for other operations potentially)

*   The OperationProgress of partially executed/failed operations. This may have information to allow a handler to continue processing instead of restarting completely.
