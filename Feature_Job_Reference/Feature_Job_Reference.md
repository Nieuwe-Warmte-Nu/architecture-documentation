# Feature: Add an optional job reference when submitting a job. 

A user of the frontend should be able to supply a job reference when submitting a job.
This reference is useful for a number of use cases:
- The output ESDL name is appended with the reference so the user can note which workflow was
activated on this ESDL.
- The name is used throughout the SDK, orchestrator & worker in logging for support purposes. The 
user may not necessarily know the job uuid but will know the reference of the job.
- The frontend should save the reference in their UI so a user can see the reference of past jobs.

Relevant components:
- omotes-rest: Receive the job reference as part of the job submission.
- SDK: Receive the job reference as part of the job submission.
- Orchestrator: Log the job reference and pass it to the workers.
- Worker: Use the job reference to change the name of the output ESDL (happens in worker SDK and
  not in each individual worker)

__Goal__: Receive a reference to a job and use it throughout the architecture for logging and for
customizing the output ESDL and other values at the workers.

## Considerations
- Job references are not unique and not part of the job identifier.

## Steps
As this is a value which is passed throughout the architecture, there are no unique steps.
It follows the same steps as a job submission.

