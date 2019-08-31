## Overview

This repository contains a representative subset of the first-party DNN training workloads on Microsoft's internal Philly clusters. The trace is a sanitized subset of the workload described in ["Analysis of Large-Scale Multi-Tenant GPU Clusters for DNN Training Workloads"](https://arxiv.org/abs/1901.05758) in ATC’19.  This work was done as part of Microsoft Research's [Project Fiddle](https://aka.ms/msr-fiddle).

We include in this repository a [jupyter notebook](https://github.com/msr-fiddle/philly-traces/blob/master/analysis/Philly%20Trace%20Analysis.ipynb) that highlights the main characteristics of the traces and shows how to parse them (a huge thank you to [Keshav Santhanam](https://github.com/santhnm2) for putting this together).

We provide the trace as is.  If you do use this trace in your research, please make sure to cite our ATC’19 paper (mentioned above).

## Trace Details

### Main characteristics:
*	**Dataset size**: 6.6 GB
*	**Compressed dataset size**: 0.98 GB
*	**Number of files**: 5 files
*	**Duration**: Jobs submitted between 2017-08-07 - 2017-12-22
*	**Total number of jobs**: 117325

### Schema:
#### `cluster_job_log`

**Description**: Contains information about each job, including each individual
successful scheduling attempt.

**Format**: JSON

**Example entry**:

```
{
    "status": "Pass",
    "vc": "ee9e8c",
    "jobid": "application_1506638472019_14199",
    "attempts": [
        {
            "start_time": "2017-10-07 01:12:09",
            "end_time": "2017-10-07 01:13:23",
            "detail": [
                {
                    "ip": "m47",
                    "gpus": [
                        "gpu0",
                        "gpu1",
                        "gpu2",
                        "gpu3",
                        "gpu4",
                        "gpu5",
                        "gpu6",
                        "gpu7"
                    ]
                }
            ]
        },
        {
            "start_time": "2017-10-07 01:13:30",
            "end_time": "2017-10-09 06:53:12",
            "detail": [
                {
                    "ip": "m412",
                    "gpus": [
                        "gpu0",
                        "gpu1",
                        "gpu2",
                        "gpu3",
                        "gpu4",
                        "gpu5",
                        "gpu6",
                        "gpu7"
                    ]
                }
            ]
        }
    ],
    "submitted_time": "2017-10-07 01:11:39",
    "user": "ce2f4c"
}
```

**List of keys**:
* `status`: The job's status upon completion. One of `Pass`, `Killed`, or `Failed`.
* `vc`: The hash of the virtual cluster the job was run in.
* `jobid`: The id of the job.
* `attempts`: A list of `dict`s where each `dict` has the following keys:
    * `start_time`: The start time of the attempt.
    * `end_time`: The end time of the attempt.
    * `detail`: A list of `dict`s where each `dict` has the following keys:
        * `ip`: The id of the server the attempt was scheduled on.
        * `gpus`: A list of GPUs used by the attempt.
* `submitted_time`: The time the job was submitted to the scheduler.
* `user`: A hash of the user id.

**Notes**:
* A job may have no recorded scheduling attempts.
* A scheduling attempt may have no recorded `start_time` and/or `end_time` -
  this could be the result of a logging error.
* If a job has a `None` value for its last attempt's `end_time`, the job was
  still running at the time the snapshot was taken.

---

#### `cluster_gpu_util`

**Description**: Provides a per-minute record of each GPU's utilization as
reported by `nvidia-smi`.

**Format**: CSV

**Columns**:

| time | machineId | gpu0_util | gpu1_util | gpu2_util | gpu3_util | gpu4_util | gpu5_util | gpu6_util | gpu7_util |
|------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|

**Example entry**:

```
2017-10-03 00:08:00 PDT,m29,60.8,99.366666667,100.0,63.333333333,100.0,100.0,100.0,100.0,
```

**Notes**: 
* Some `gpu*_util` values may be `"NA"`, indicating the GPU was offline at the
  time of measurement.

---

#### `cluster_cpu_util`

**Description**: Provides a per-minute record of each server's CPU utilization.

**Format**: CSV

**Columns**:

| time | machine_id | cpu_util |
|------|------------|----------|

**Example entry**:

```
2017-11-27 00:04:00 PST,m29,31.845
```

**Notes**:
* Some `cpu_util` values may be `"NA"`, indicating the server was
offline at the time of measurement.

---

#### `cluster_mem_util`

**Description**: Provides a per-minute record of each server's memory
utilization.

**Format**: CSV

**Columns**:

| time | machine_id | mem_total | mem_free |
|------|------------|-----------|----------|

**Example entry**:

```
2017-10-03 00:06:00 PDT,m29,528272672.0,2030730.6667
```

**Notes**:
* Some `mem_total` and `mem_free` values may be `"NA"`, indicating the server
was offline at the time of measurement.

---

#### `cluster_machine_list`

**Description**: Lists the number of GPUs and per-GPU memory available on each
server in the cluster.

**Format**: CSV

**Columns**:

| machineId | number of GPUs | single GPU mem |
|-----------|----------------|----------------|

**Example entry**:
```
m31,8, 24GB
```
