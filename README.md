# splunkd_pstacks_threads

Typically if indexers have more than 500+ active threads in use something may be wrong or there is a lock contention (ie: file descriptor leak, hung processes, etc). 
If that happens we want to collect pstacks on a host if thread count exceeds 500 threads and trigger pstack collection once there are more than 500 active threads.

The following search will show active threads:

```
| tstats
max(data.t_count) as data.t_count
where index=_introspection host IN(*idx*) sourcetype::splunk_resource_usage  by host _time span=60s
| timechart span=60s max(data.t_count) as "thread count"  by host limit=0 | eval danger=500 
```


How do we know which indexer to trigger pstack collection on if the issue happens randomly on different indexers? 

option A:
create an alert based on search below to run every ~15m and see if any indexer has thread count exceeding 500

```
index=_introspection host IN (*idx*) sourcetype=splunk_resource_usage 
| stats max("data.t_count") as thread_count by host | search thread_count>500
```

option B:

- install script on all indexers and run the script in the background
- create alert so we know which host to fetch pstacks off of and generate diag.
- upload pstacks and diag to Splunk Support.
