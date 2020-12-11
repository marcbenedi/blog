+++
title = "How to know which CPUs have been allocated to a job?"
date = 2020-12-11T10:23:25+01:00
draft = false
author = "Marc Benedi"
authorTwitter = "marcbenedi"
cover = ""
tags = ["Linux", "Slurm"]
keywords = ["Slurm"]
description = "Check how to get the physical CPU IDs allocated to a Slurm job"
showFullContent = false
+++

# TL;DR

We can get the abstract CPU IDs that Slurm allocated for a job through `scontrol`[[4]]:

```bash
$ scontrol show job -d $SLURM_JOBID
...
  Nodes=node CPU_IDs=0-4,8-9 Mem=128000 GRES_IDX=
...
```

This will list all the information regarding the job, including the CPU IDs. The problem is that these IDs do not match the actual physical cores (which is what commands such as `htop` show us)[[3]]. To match these abstract IDs to the physical ones, we need to check the output of the command `lstopo`:

```text
$ lstopo --only pu
PU L#0 (P#0)
PU L#1 (P#6)
...
```

As you can see in the output, for each *physical unit (PU)* we get two values: the *logic ID (L)* and the *physical ID (P)*.
In the Slurm documentation[[2]], it can be found that they use the *logic ID* as *abstract ID*.

# Full story 🥸

I'm currently setting up a HPC using Slurm[[1]] for the *Visual Computing & Artificial Intelligence group*[[5]] as a student assistant. One of the features that we want is being able to detect inactive jobs to free the allocated resources. For that, I currently check the GPU and CPU usage. The nodes of the cluster have a *Telegraf* service which sends metrics to an *InfluxDB*. Getting the GPU metrics was easy: we created a *prolog* script which stored the environment variables (which includes the GPUs ID if any requested) and our API would read the content of that file and query InfluxDB. 

Regarding CPU usage, the first approach was setting up the *ProfileInfluxDB* plugin[[6]] which would collect jobs' metrics and send them to our InfluDB. However, for our use case, this plugin was not suitable since the metrics were not sent to the database often enough (in our set up they were sent every 20 mins approx.). This is explained in [[6]] in the following note:

*NOTE: Collected information is written from every compute node where a job runs to the influxd instance listening on the ProfileInfluxDBHost. **In order to avoid overloading the influxd instance with incoming connection requests, the plugin uses an internal buffer which is filled with samples**. Once the buffer is full, a HTTP API write request is performed and the buffer is emptied to hold subsequent samples. A final request is also performed when a task ends even if the buffer isn't full.*

My second approach was getting the CPU IDs assigned to each job and then querying the *InfluxDB* with the metrics sent by the *Telegraf* service. This approach was very similar to the one I was using for the GPUs, the only difference was the source of the IDs. After a little bit of *Googling*, I found this answer in stackoverflow[[4]] which explained how to get the IDs from `scontrol` (See **TL;DR**). I decided to implement this solution as it looked very promising. After implementing it, I realized that some jobs were classified as active when they should not. After looking at the metrics reported by *InfluxDB*, I decided to check through htop what was running on those CPUs. What I saw really worried me: Some jobs were running on CPUs that were not assigned to them 😨! As you may have guessed (especially if you went over the **TL;DR** section), that was not the actual problem. I won't bother you explaining all the things I checked but my main lead was that something was wrong with the multi-threading configuration (which was a problem that we actually had 😅). 

Going crazy around Slurm's documentation I found the following clue[[3]]:

*A NOTE ON CPU NUMBERING: The number and layout of logical CPUs known to Slurm is described in the node definitions in slurm.conf. **This may differ from the physical CPU layout on the actual hardware**. For this reason, Slurm generates its own internal, or "abstract", CPU numbers. These numbers may not match the physical, or "machine", CPU numbers known to Linux.*

So that was it: Slurm was not using the actual *physical IDs* but some internal numbering. The next step was obvious: How do we go from abstract IDs to physical IDs? Again, I went through Slurm's documentation until I found another clue[[2]]:

*NOTE: Since Slurm must be able to perform resource management on heterogeneous clusters having various processing unit numbering schemes, a logical core index must be specified instead of the physical core index. That logical core index might not correspond to your physical core index number. Core 0 will be the first core on the first socket, while core 1 will be the second core on the first socket. **This numbering coincides with the logical core number (Core L#) seen in "lstopo -l" command output.***

Finally! I had found the key to solving the puzzle `lstopo` 🏆.

To not repeat myself, all the pieces are put together in the **TL;DR** section.

# References 📑

[1]: https://slurm.schedmd.com/
[[1]]: https://slurm.schedmd.com/

[2]: https://slurm.schedmd.com/gres.conf.html
[[2]]: https://slurm.schedmd.com/gres.conf.html

[3]: https://slurm.schedmd.com/cpu_management.html
[[3]]: https://slurm.schedmd.com/cpu_management.html

[4]: https://stackoverflow.com/a/55655127/7484221
[[4]]: https://stackoverflow.com/a/55655127/7484221

[5]: https://www.niessnerlab.org/
[[5]]: https://www.niessnerlab.org/

[6]: https://slurm.schedmd.com/acct_gather.conf.html
[[6]]: https://slurm.schedmd.com/acct_gather.conf.html