# Timeseries DB component

Options:
- Influxdb v2
- TimescaleDB

# Overview
<Insert table of features and characteristics>


# Performance results
Write performance
- 10-11GB partially repetative data (compressed to 1.2GB)
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