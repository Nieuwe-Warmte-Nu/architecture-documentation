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

The proposal is to create a `job_deletions` queue to which an SDK can publish. The orchestrator will
subscribe to this queue and remove the data from the time series database. A `JobDelete` message
will contain the output ESDL id of the omotes job.  
Also on a `JobCancel` message, the orchestrator should, after termination of the celery job, remove
any time series data, if present.  
Also in case of an error or timeout, all timeseries entries, if any, should be removed.

### How to handle abrupt worker terminations?

When an omotes worker is terminated abruptly (due to cancellation, time out, error) there is no way
that the influxdb data can be removed, since the output ESDL id has not been passed through.  
A solution to this would be to create an `timeseries_esdl` table in the orchestrator postgres database
with `esdl_id`, `registered_at`, `job_id`, `job_reference` and `inactive_at` columns. The
`registered_at`, `job_id` and `job_reference` are not used but could be useful for admins.  
This database will contain active ESDL's which will be added on completion of a job.
It will also contain possible stale/inactive ESDL's identified by a non-null 'inactive_at' value.
These will be created when an influxdb is found for which no 'timeseries_esdl' entry can be found (
because it was not created due to a job crash, or because the job is still running). When a
specified time has passed since 'inactive_at' (allowing for job completion), the 'timeseries_esdl'
entry will be removed, as well as the influxdb data.  

This table is used as follows, upon:

- completion of a job: the output ESDL id is registered in the table with `inactive_at` set to
  `NULL`
- deletion/cancellation of a job: the output ESDL id is removed from the table (and the related
  influx db table is removed)

There is a daily influx cleanup job which checks if all influxdb databases are in the `timeseries_esdl`
table, and then takes the following actions:

- if not present: register with `inactive_at` set to the current time
- if present and `inactive_at` is not `NULL` and a specified duration (env var: a week) has passed
  since `inactive_at`: remove the influx database and the `timeseries_esdl` table entry
- lastly any `timeseries_esdl` table entries for which no influxdb database is found should be removed

### Omotes-rest

A `/job/<job_id>/delete` endpoint will be added, similar to the `/job/<job_id>/cancel` endpoint.
