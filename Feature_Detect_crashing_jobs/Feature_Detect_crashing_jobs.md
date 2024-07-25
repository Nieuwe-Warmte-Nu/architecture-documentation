# Feature: Detect an indefinitely crashing job and how to remove it.

Within the OMOTES architecture every component will ensure that data (including job messages)
are persisted at every step to prevent work from entering an inconsistent state or becoming lost.
However, this introduces the risk of persisting a job which will crash everytime and therefor
becomes 'stuck'. This is especially a risk for jobs which are submitted into Celery and cause
a hard crash (such as an OOM or out of memory exception) everytime they are processed. The jobs
will be reprocessed indefinitely until (manual) intervention.

Normally RabbitMQ uses the feature ['delivery limit' or 'Poison Message Handling'](https://www.rabbitmq.com/docs/quorum-queues#poison-message-handling)
to move these messages from their original queue to a a 'dead letter queue' once the message
has been (re)delivered a set amount of times. However, RabbitMQ only supports this feature when
the Quorum queue type is used and in our architecture this is not possible as we require classic
queue type to utilize the message priority feature. Therefore we have opted to build this mechanism
ourselves within OMOTES. 

Upon analysis of the architecture, the only location where we need this mechanism is at the Celery
cluster where jobs are delivered to workers. Anywhere else (such as a job submission which is 
received by the orchestrator) may cause issues as well but the preferred solution would be to solve
the underlying issue (such as a software bug or offline component) in favor of moving the messages.

We have also decided that the required steps to clean up a job which has crashed multiple times
would be to:

- Send the (error) result to the SDK
- Clean up the job from the PostgreSQL.
- Log the issue in the orchestrator.
- Remove the job from the Celery cluster (silently) by cancelling it.

To implement this feature there is the choice between the worker and orchestrator components to
check how often a job has been tried. In the end we have opted to choose the orchestrator.
The reasoning:

- Let the worker check the delivery limit.
  - Downside: Worker would need to connect with PostgreSQL to write a record of delivery and check
    if the message has already reached the delivery limit.
  - Downside: We need to also implement that the worker checks the received Celery task id against
    the one persisted in PostgreSQL (currently Orchestrator performs this task).
    - We need to also implement a waiting mechanism as the orchestrator would only persist the
      Celery task id after the job is submitted to Celery. There might a slight delay where
      PostgreSQL does not yet contain the task id but the orchestrator will add it soon. (This is
      also the reason why we had to add the init barriers in the orchestrator)
      Another way to implement this would be to let the workers 'race' to be the first to submit
      its Celery task id to the PostgreSQL database. The first one to succeed wins and is allowd to
      perform the job. The other workers should drop the job silently. But we would still require to
      move this feature from the orchestrator to the worker.
 
- Let the orchestrator check the delivery limit.
  - Use the first progress update ('start job' situation) to write the delivery into the delivery
    record.
  - If the delivery limit is reached, we should cancel the job but it may take a little time for
    the cancellation to reach the workers.
  - So there is a risk that the job may be redelivered to a worker and started before it can get
    cancelled wasting a little bit of resources. But then, soon, it will get cancelled.
  - Downside: We do not get control over the Celery task message. If a task gets cancelled, the
    message is dropped silently by Celery. So we have no information from the worker e.g. which
    worker performed the job, any intermediate logs etc.. However, it appears we do not need this
    information either at the moment so it may not be a downside.
  - Performance consideration: Worker does not need to do any checking before performing a task.

Advise is to let the orchestrator do the check as to keep the architecture simple. The architecture
already performs cancellations when the SDK sends a cancel, when the job times out or when a single
job is send to the Celery workers multiple times it will cancel the 'wrong' Celery tasks. Adding
this responsibility to the orchestrator would keep the workers simpler and the orchestrator the
only component to actively cancel jobs.

Relevant components:
- SDK: Will eventually receive the (error) result.
- Orchestrator: Will detect the multiple delivery attempts of the job and cancels the job once
  the limit is reached.
- Worker: Will attempt to process the job and will hard crash at some point.
- PostgreSQL: Will persist the job information while it is being processed but also will persist
  each delivery attempt of the job.
- RabbitMQ: Communication between SDK & orchestrator

__Goal__: To detect that a job causes multiple hard crashes and to remove the job from processing.
The SDK should be notified and any resource, such as the rows in PostgreSQL describing this job,
should be cleaned up.

## Considerations
- There may be multiple SDKs at once.
- They may be 1 or more workers capable of processing this job.
- Messages in RabbitMQ are persisted within durable queues so they will not be removed automatically.
- RabbitMQ is configured to use classic queues so the delivery limit functionality of quorum queues
  is not available.
- There is a little time between the orchestrator noticing the job needs to be cancelled and the job
  being retried by the workers indefinitely. This period is accepted.


## Steps
We assume the orchestrator, relevant worker(s) & SDK are all online and the workflow definitions
have been shared by the orchestrator to the SDK and a job has been submitted by the SDK.
The steps and coordination between components are shown in the next image.

![Steps detect an indefinitely crashing job and how to remove it.](/Feature_Detect_crashing_jobs/detect_crashing_jobs.drawio.png)
