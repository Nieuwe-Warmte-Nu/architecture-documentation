# Orchestrator component

Options:
- Airflow (Works but heavy architecture)
- Argo (Works only for k8s but seems to do what we want. Arbitrary code/scripts is convoluted it can only schedule containers)

Quick non-options:
- Keboola (No selfhosting so not an option. No further analysis)
- Kubeflow (Is meant for ML pipelines, not our use case)
- Crossplane (Is meant to manage cloud resources through a k8s cluster. Does not allow for workflows.)
- Luigi (Is meant for batch processing and submitting jobs using CLI)
- Prefect (Paid rbac)
- Mage (No scheduling)
- Dagster (No REST api to trigger workflows.)
- Kestra (Is for continuous datapipelines and cannot schedule a pod and wait for it to complete)

# Overview
<Insert table of features and characteristics>

# Requirements
- HA or persistency if orchestrator crashes or is updated. Can it resume from the previous state?
- QoS: Allow for configuring priority on jobs. E.g. if a user sneds in 200 jobs and another user only 1, the second user should gain priority. But still, the 200 jobs should not be enqueued for ever
- Job queue: If there are too many jobs, new jobs may be enqueued.
- Lifecycle management of cluster resources (pods): Start them, kill them on errors, clean up.
- Is able to connect to broker to send. (Just in case, not necessary for first version of design)
- Can schedule multiple steps after each other.
- Can schedule pods with env var information.
- Can retrieve arguments for job when a new job is started through the API.
- Lightweight: prefer single component without extra pools of resources.
- Selfhosted
- Integration with both k8s and docker


# Performance results


# Conclusion
We chose TimescaleDB..

# Relevant links / literature