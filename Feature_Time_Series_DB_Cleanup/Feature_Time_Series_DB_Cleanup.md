# Feature: Time series database cleanup

## Problem

Omotes 'forgets' about jobs when they are completed and the results have been retrieved.
Except for the time series data which are stored in a database, and the results are kept
indefinitely. This will cause the database to grow without an option to clean up, since omotes does
not know which data is still relevant.
Most front-ends will have an option to delete omotes runs which should also remove the time series
data. Currently, this is possible by directly accessing the times series database. This is a bit
cumbersome since every frontend will need to implement this, knowledge of the database structure is
needed, and it will be database type dependent.

## Proposal

If omotes deals with removing the time series data, only omotes code needs to be updated in the case
of:

- an update of the database structure
- use of a different type of time series database

The proposal is to always delete any time series present when cancelling a job. This
`job_cancellations` queue will therefore be renamed to `job_deletions`.
The orchestrator will be responsible for removing the data from the time series database.
This deletion will not be immediate but after some period, to allow possible job results to come in.

### How to handle abrupt worker terminations and crossing events?

Issues might arise in the following situations:

- Abrupt termination of an omotes worker that has already written time series data: the output ESDL
  id has not been passed through, so the time series data(base) cannot be removed directly.
- Crossing events: for instance a successful job result is received after a job deletion.

Therefore, a `esdl_time_series_info` table is created in the orchestrator postgres database with
ID's of 'active' ESDL's for which the time series data is to be preserved. These ESDL ID's will be
added on completion of a job. It will also contain possible stale/inactive ESDL's identified by a
non-null 'deactivated_at' value. These will be set when a job is deleted, when time series data is
deleted, or when a time series database is found for which no `esdl_time_series_info` entry can be
found (because it was not created due to a job crash, or because the job is still running). When a
specified time has passed since `deactivated_at` (allowing for job completion), the
`esdl_time_series_info` entry will be removed, as well as the influxdb data.    
It will contain the following columns: `esdl_id`, `registered_at`, `job_id`, `job_reference` and
`deactivated_at`. The `registered_at`, `job_id` and `job_reference` are not used but could be useful
for admins.

The time series data will be managed by a new `EsdlTimeSeriesManager` which does a regular check (
every `JOB_RETENTION_SEC` env var seconds) if all the time series data are represented in the
`esdl_time_series_info` table, and then takes the following actions:

1. Time series database cleanup
    - if an ESDL time series data collection (database) has no corresponding `esdl_time_series_info`
      table entry: register with `deactivated_at` set to the current time, marking the time series
      data for deletion
    - if an ESDL time series data collection (database) has a corresponding `esdl_time_series_info`
      table entry, and `deactivated_at` is non-`NULL`, and a specified duration (`JOB_RETENTION_SEC`
      env var) has passed since `deactivated_at`: remove the time series database and the
      `esdl_time_series_info` table entry
2. Postgres `esdl_time_series_info` table cleanup
    - if a `esdl_time_series_info` table entry has no corresponding time series data, and a
      specified duration (`JOB_RETENTION_SEC` env var) has passed since `registered_at`: remove
      this then stale `esdl_time_series_info` table entry

### Scenarios

The following scenarios can occur:

1. Job completed successfully
    1. Upon receiving the job result a `esdl_time_series_info` table row (with `esdl_id`) is created
       with `deactivated_at` set to `NULL` which guards the time series data from deletion.
    2. In case a `esdl_time_series_info` table row with the `job_id` already exist (for instance
       when the job has been deleted, see below), this row will be updated with the `esdl_id`. The
       time series data is thus marked for deletion.
2. Job succeeded non-successfully
    1. No `esdl_time_series_info` table row is created which results in deletion by the
       `EsdlTimeSeriesManager` of any time series
       data that might have been created.
3. Job deletion by `job_id`
    1. If a corresponding `esdl_time_series_info` table row is found, and `deactivated_at` is
       `NULL`: set `deactivated_at` to `now`, marking it for deletion.
    2. If no `esdl_time_series_info` table row is found, a new entry is created with the `job_id`
       and `deactivated_at` set to `now` (without the `esdl_id` since that is not available). This
       ensures that, in case a successful result is received in the future, the time series data
       will still be deleted.

### Omotes-rest

Remove the `/job/<job_id>/cancel` endpoint and update the `/job/<job_id>/delete` endpoint to post to
call the `job_delete` function which will now cancel and delete.
