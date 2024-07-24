# Feature: Centralizing workflow definitions.

The OMOTES architecture is constructed to allow different types of workflows to be performed.
Each job has a specific type and OMOTES ensures that the job is performed by a worker
capable of processing this specific 'job type'. It is key that the job is delivered
to the worker in a robust fashion.

Relevant components:
- SDK: Can submit jobs and waits for status updates, progress updates and the result.
- Orchestrator: Coordinates that submitted jobs are submitted to the workers through Celery,
  are persisted at PostgreSQL while they are processing and updates the SDK at every step.
- RabbitMQ: Communication between SDK & orchestrator
- PostgreSQL: The database with a record of all submitted jobs that have not yet finished. This
  component is used to keep a record of all submitted jobs across a reboot of the orchestrator.

__Goal__: Robust job submission where a (software) failure anywhere in the architecture does not 
result in data loss. Jobs can be both very short (< 1 second calculation time) as well as long
(multiple hours of calculation time) and the architecture should be performant to handle lots
of short jobs quickly as well as quick updates on long running jobs.

## Considerations
- There may be multiple SDKs at once who each submit jobs. However, we assume that only the SDK
  which submits the job will need to be updated on progress & the result.
- We assume each job only has a single result.
- We are relying the at-least once delivery guarantee of messages at each hop. This ensures
  that a message is send by RabbitMQ and ONLY acknowledged once all required processing has
  successfully occurred. In case something goes wrong, the message will stay in the queue
  until a later moment. Example: The orchestrator receives a `JobSubmission` message from the
  `job_submissions` queue. It will only acknowledge this message if:
   1. The job is registered successfully in PostgreSQL
   2. The job is successfully submitted through Celery to the workers.
   3. The job is registered as successfully submitted in PostgreSQL.
   4. The `JobStatusUpdates` for REGISTERED and ENQUEUED are successfully published to RabbitMQ with
      the SDK as its destination.

   If any of these steps fail, the message will remain in RabbitMQ until the issue that causes
   the failure is resolved (such as a software update, reboot of another component etc.).
   Data is persisted in both RabbitMQ and PostgreSQL and the other components are responsible for
   ensuring the data remains persisted.

## Steps
We assume the orchestrator, relevant worker(s) & SDK are all online and the workflow definitions
have been shared by the orchestrator to the SDK. The steps and coordination between components
are shown in the next image.

![Steps to submit and process a job](/Feature_Job_Submission/job_submission_happy_path.drawio.png)

