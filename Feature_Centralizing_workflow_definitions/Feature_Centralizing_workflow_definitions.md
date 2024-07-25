# Feature: Centralizing workflow definitions.

The OMOTES architecture is constructed to allow different types of workflows to be performed.
Each job has a specific type and OMOTES ensures that the job is performed by a worker
capable of processing this specific 'job type'. However, it is also required to accompany the
'job type' with zero or more parameters in `params_dict` to complete the job submission.
We refer to the description of all components that make up a job submission as a
'workflow definition'. Currently, we a workflow definition contains:

- Workflow type or a.k.a. job type: A string that identifies the type of job or workflow.
- The parameters within params_dict. For each parameter also the required type, an optional
  default value and a number of other attributes are required.

The workflow definition is required at both the orchestrator to define which workflows are
configured within the OMOTES instance, but also at the SDK to validate that a job submission
meets all the requirements. As the same information is required at multiple components,
this feature explains how we can share the information from the orchestrator to the SDK
and how to propagate any changes.

Relevant components:
- SDK: Can request and cache the current workflow definitions.
- Orchestrator: The workflow definitions are configured here and maintained and communicated to SDK.
- RabbitMQ: Communication between SDK & orchestrator

__Goal__: Workflow definitions should only be defined and maintained at one location, however, the SDK and orchestrator
both need to know the current workflow definitions. Therefore, they are communicated when necessary to keep
the state of the current workflow definitions the same between the orchestrator & SDK.

## Considerations
- There may be multiple SDKs at once.
- We have decided to pursue an event-based approach. The SDK should send a RequestAvailableWorkflows message when it
  boots up to request the current workflow definitions from the orchestrator. The orchestrator should just send
  the current AvailableWorkflows when it starts up.
  - This is required as the SDK and the orchestrator are separate components and do not share life cycles.
    The orchestrator may start, stop, restart independent of when the SDK starts, stops and restarts (and vice versa).
- The SDK cannot function without knowing the current workflow definitions.

## Steps
With OMOTES it is possible that either the orchestrator or the SDK starts first. We describe
these different starting situations 2 seperate scenarios.

### The SDK starts first and then the orchestrator (re)starts

1a. The SDK starts but the orchestrator has not yet started. The SDK will send the
'RequestAvailableWorkflows' request but the orchestrator has not yet started and the
`request_available_workflows` queue does not yet exist so the message is dropped by the broker.
![Steps when the SDK starts but the orchestrator has not yet started.](/Feature_Centralizing_workflow_definitions/Communicating%20available%20workflows.sdk%20starts%20but%20no%20orchestrator.v1.drawio.png)

1b. Steps when the orchestrator (re)starts and 1 or more SDKs are already available. The orchestrator
will read the workflow definitions from disk and propagate it to the SDKs regardless if it contains
changes or if it is the same as previous time the orchestrator started.
![Steps when the orchestrator starts up.](/Feature_Centralizing_workflow_definitions/Communicating%20available%20workflows.orchestrator%20initiated.v1.drawio.png)


### The orchestrators starts first and then the SDK (re)starts

2a. The orchestrator starts but the SDK has not yet started. The orchestrator will publish the
workflow definitions to the `available_workflows` routing key but no queues are bound to the routing
key so the message is dropped silently.
![Steps when the orchestrator starts but the SDK has not yet started.](/Feature_Centralizing_workflow_definitions/Communicating%20available%20workflows.orchestrator%20starts%20but%20no%20SDK.v1.drawio.png)

2b. Steps when the SDK starts and the orchestrator is already available. Other SDKs may already
be online as well. The SDK requests the current workflow definition and awaits a response from
the orchestrator during start up.
![Steps when the SDK initiates a request.](/Feature_Centralizing_workflow_definitions/Communicating%20available%20workflows.sdk%20initiated.v1.drawio.png)


__Note__: Whenever an SDK requests the current workflow definitions, all SDKs will receive
the current workflow definitions. This may lead to a specific SDK receiving the same workflow
definitions multiple times. This has no negative side-effects as the SDK will load the workflow
definitions each time and overwrite the previous. As previous and new definitions are equal,
the SDK will behave equally. While it would be possible to negate this effect within the feature
to ensure the orchestrator sends the workflow definitions to only the requesting SDK, it would
complicate the architecture unnecessarily. The orchestrator would need to be able to send
the workflow definitions both to a specific SDK as well as broadcast the definitions to all SDKs
on start up. In the current implementation, the orchestrator always broadcasts the workflow
definitions.
