# Requirements for workers:
- A worker deployment may consist of one or more containers. (Example workflow 2 of multiple containers)
- The worker deployment must be able to scale horizontally.
- Worker deployment must register itself in the cluster as present:
	- Subscribe to relevant queue.
	- Notify the job type it supports