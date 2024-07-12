# Feature: Centralizing workflow definitions.

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
Steps when the SDK starts up (and the orchestrator is already available) and requests the current workflow definition:
![Steps when the SDK initiates a request.](/Technical%20Architecture/Communicating%20available%20workflows.sdk%20initiated.v1.drawio.png)

Steps when the orchestrator starts up (and 1 or more SDKs are already available):
![Steps when the orchestrator starts up.](/Technical%20Architecture/Communicating%20available%20workflows.orchestrator%20initiated.v1.drawio.png)


There are also 2 other situations:
1) The SDK is started but the orchestrator is not yet online.
2) The orchestrator starts up but the SDK is not yet online.

In both situations their respective messages may be send but they are dropped silently by the broker.
The `available_workflow.<SDK_ID>` and `request_available_workflows` queues are managed by their subscriber
and if the queue does not exist, the message isn't routed by RabbitMQ.
