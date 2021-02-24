---
layout: post
title: Salloc Command
tags: [HPC, Salloc, Slurm Allocate]
excerpt_separator: <!--more-->
---

<!--more-->

`salloc` = slurm allocate 

Usage: `salloc [OPTIONS...] [command [args...]]`

Parallel run options:

| -A   | `--account=name`                 | charge job to specified account                              |
| ---- | -------------------------------- | :----------------------------------------------------------- |
| -b   | `--begin=time`                   | defer job until HH:MM MM/DD/YY                               |
|      | `--bell`                         | ring the terminal bell when the job is allocated             |
|      | `--bb= <spec>`                   | burst buffer specifications                                  |
|      | `--bbf=<file_name>`              | burst buffer specifications file                             |
| -c   | `--cpus-per-task=ncpus`          | number of cpus required per task                             |
|      | `--comment=name`                 | arbitrary comment                                            |
|      | `--cpu-freq=min[-max[:gov]]`     | requested cpu frequency (and governor)                       |
|      | `--delay-boot=mins`              | delay boot for desired node features                         |
| -d   | `--dependency=type:jobid[:time]` | defer job until condition on jobid is satisfied              |
|      | `--deadline=time`                | remove the job if no ending possible before this deadline (start > (deadline - time[-min]))|
|      |                                  |                                                              |
|      |                                  |                                                              |
|      |                                  |                                                              |
|      |                                  |                                                              |
|      |                                  |                                                              |
|      |                                  |                                                              |
|      |                                  |                                                              |
|      |                                  |                                                              |

