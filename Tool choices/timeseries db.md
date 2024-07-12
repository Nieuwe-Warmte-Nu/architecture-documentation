# Timeseries DB component

Options:
- Influxdb v2
- TimescaleDB

# Requirements
- HA i.v.m. data redudantie <-- Should be possible now. Build it later in project if discussion deems necessary.
- Lots of data -> Storage efficiency / compression?
- Retention options
- RBAC
- Efficient retrieval while writing (many readers & writers at the same time)
- Discussion point: Backup strategy (disaster recovery). Maybe we do not build it now, but it should be possible.
- Per job output grouping (grouped by labels, database or another grouping mechanism) 


# Performance results
Write performance
- 10-11GB partially repetative data (compressed to benchmark data 1.2GB on disk, not db)
- Disk io bottleneck huge. Ramdisk to elevate during benchmark. Need NVMe disk. Became CPU bounded.
- CPU == 2 cores, 4 threads. Autovacuum makes it CPU bound => periodically spikes CPU usage.
- Execution time of benchmark on ramdisk was 265 seconds and with autovacuum off it became 217 seconds.
- Max write speed: 38,6 MB per second (146399,75 rows / sec) for 10GB dataset COPY with autovacuum on. 47,2MB per seconds with autovacuum off.
	- Unsure what the bottleneck is in autovacuum off scenario.
- Gigabit network link is okay.
- 10GB filesystem became 1.0GB write ahead log & 11GB data
- Timescaledb appears not to have compressed the data.
- Memory usage is ~7GB (so 8-16GB geheugen)
- Used 8 worker threads. PostgreSQL spawns a new process per connection which allows us to scale across connections for large inserts.

Read performance

Need to check 

# Conclusion
We chose TimescaleDB..

# Relevant links / literature