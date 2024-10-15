# Feature: Send progress updates and error messages from workers to frontend

The code from the mesido and simulator pip packages is run inside
workers: https://github.com/Project-OMOTES/optimizer-worker
and https://github.com/Project-OMOTES/simulator-worker.  
The feature described here allows for progress updates during worker runs, and error handling, to
the sdk (front-end).

## progress updates during worker runs

To initiate running of the mesido and simulator code, a specific start function in each of the pip
packages is called in the workers. This start function receives an
`update_progress_function` callback function. Upon calling this function the following happens:

1. the worker sends a `TaskProgressUpdate` protobuf message to the `omotes_task_progress_events`
   rabbitmq queue
2. the orchestrator receives this `TaskProgressUpdate` message and:
    - updates the orchestrator postgres database
    - sends a `JobProgressUpdate` protobuf message with `jobs.<job_uuid>.progress` as routing key
3. the sdk receives this `JobProgressUpdate` message and the `callback_on_progress_update` is
   executed
4. (omotes-rest will update the omotes-rest postgres database in `callback_on_progress_update`)

## Worker error handling

When an exception is thrown during the execution of a worker the celery worker is terminated.
The job will be of `ERROR` result type and the exception message will be displayed in the job logs.

`TODO`: catch specific exception with a 'nice' error message to be relayed separately to the
front-end.
