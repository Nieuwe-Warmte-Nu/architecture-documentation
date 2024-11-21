# Feature: Job queues cleanup

When a new job is submitted via the SDK, the `result`, `progress`, and `status` queues associated with the submitted job are declared and created in RabbitMQ (see the **Queue design** diagram at the root README for reference). Normally, three queues will be automatically removed via SDK after the job is completed and the job result message is consumed by the SDK [1]. This avoids leftover job queues piled up in the RabbitMQ. Nonetheless, in some erroneous circumstances, the job queues may not be cleaned up and left infinitely in the RabbitMQ due to faulty situations, for instance, the SDK goes offline unexpectedly.

__Goal__: To avoid leftover job queues piling up in the RabbitMQ infinitely, job queues should be removed automatically by the system when they are unused and after reaching their designated Time-To-Live (TTL) setting.

# Considerations

1. Job queues should be removed automatically when they are unused and older than 48 hours.
2. The SDK client should be notified when the job queues do not exist or already expired when trying to reconnect. 
3. <del>In case of a result queue, check if it is empty.</del>
4. <del>Log messages in the system when leftover queues are found and removed.</del>

The considerations 2 & 3 were dropped after the initial implementation and discussion [2] as the complexity added to the system outweighs the added values of these features, especially since this feature originates from _edge_ case handling. More detailed discussion and considerations refer to the issue description [3].

# Implementation

The implementation utilizes the Queue TTL feature of RabbitMQ [4], which autodeletes the queue when it is unused (no subscriber) and reaches the designated TTL setting. The TTL is by default set to be 48 hours and can be overwritten with the `auto_cleanup_after_ttl` flag when calling the SDK `submit_job()` or `connect_to_submitted_job()` method. When the queue is expired and removed, the message it may or may not contain will be dropped silently.

# Steps

The feature has been tested under the following scenarios.

```
1. Happy path
    1. The SDK client submits a job.
    2. The job is finished, returned to the queue, and consumed by the SDK client.
    3. All job queues are removed as intended.

2. Immediate reconnect
    1. The SDK client submits a job
    2. The job starts running, but the SDK client goes offline for 2 seconds (Queue TTL is activated).
    3. The SDK client attempts to reconnect after 2 seconds, first checking if queues still exist.
    4. The queues still exist, the SDK client reconnects to the queues (Queue TTL is deactivated).
    5. The job is finished, returned to the queue, and consumed by the SDK client.
    6. All job queues are removed as intended.

3. Reconnect after the job is finished
    1. The SDK client submits a job.
    2. The job starts running, but the SDK client goes offline for 15 seconds (Queue TTL is activated).
    3. The job is finished, returned to the queue, and waited to be consumed.
    4. The SDK client attempts to reconnect after 15 seconds, first checking if queues still exist.
    5. The queues still exist, the SDK client reconnects to the queues (Queue TTL is deactivated).
    6. The SDK client consumes the job result message and removes all job queues as intended.

4. Never reconnect
    1. The SDK client submits a job.
    2. The job starts running, but the SDK client goes offline forever.
    3. The job is finished, returned to the queue, and waited to be consumed.
    4. The job messages are dropped silently and queues are removed after reaching queue TTL.

5. Reconnect after queues have already expired
    1. The SDK client submits a job.
    2. The job starts running, but the SDK client goes offline for a long period (longer than the provided TTL).
    3. The job is finished, returned to the queue, and waited to be consumed.
    4. The job messages are dropped silently and queues are removed after reaching queue TTL.
    5. The SDK client attempts to reconnect, first checking if queues still exist.
    6. The queues do not exist, reconnection is aborted and an exception is thrown to the SDK client.
```


# Notes
1. If `auto_disconnect_on_result` is set to true when calling the SDK `submit_job()` or `connect_to_submitted_job()` method.
2. Revision of the feature requirements: https://github.com/Project-OMOTES/omotes-system/issues/80#issuecomment-2343761440
3. Clean up left-over job status, progress & result queues: https://github.com/Project-OMOTES/omotes-system/issues/80
4. RabbitMQ Queue TTL: https://www.rabbitmq.com/docs/ttl#queue-ttl