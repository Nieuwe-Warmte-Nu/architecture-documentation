# Feature: Send progress updates and error messages from workers to frontend

The code from the mesido and simulator pip packages is run inside
workers: [optimizer worker](https://github.com/Project-OMOTES/optimizer-worker)
and [simulator worker](https://github.com/Project-OMOTES/simulator-worker).
The feature described here allows for progress updates during worker runs, and error handling, to
the sdk (front-end).

## Progress updates during worker runs
A worker registers a specific function as its worker's task. This task receives a number of
arguments including an [`update_progress_function`](https://github.com/Project-OMOTES/omotes-sdk-python/blob/3.2.2/src/omotes_sdk/internal/worker/worker.py#L336).
To initiate running of the mesido and simulator code, a specific start function in each of the pip
packages is called in the workers. This start function is passed the `update_progress_function`
callback function. Upon calling this function the following happens:

1. The worker sends a `TaskProgressUpdate` protobuf message to the `omotes_task_progress_events`
   rabbitmq queue
2. The orchestrator receives this `TaskProgressUpdate` message and:
    - updates the status of the job in the orchestrator postgres database if it is the first
      progress update.
    - sends a `JobProgressUpdate` protobuf message with `jobs.<job_uuid>.progress` as routing key
3. The sdk receives this `JobProgressUpdate` message and the `callback_on_progress_update` is
   executed
4. (omotes-rest will update the omotes-rest postgres database in `callback_on_progress_update`)

## Worker error handling

When an exception is thrown during the execution of a worker the celery worker is terminated.
The job will be of `ERROR` result type and the exception message will be displayed in the job logs.

`TODO`: catch specific exception with a 'nice' error message to be relayed separately to the
front-end.
