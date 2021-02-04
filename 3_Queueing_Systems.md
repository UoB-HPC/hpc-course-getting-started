# Queueing Systems

**Important**: Before reading this section, you need to be familiar with the concepts of _login nodes_ and _compute nodes_.
In a typical supercomputer system, when you connect to the machine you will get to a _login node_, which is shared with other users connected to the system.
This is where you usually compile your application and set up your environment.
In contrast, running applications happens on _compute nodes_, which are allocated and freed as needed, and are generally dedicated to a single user for the duration of their application's execution.

Supercomputers house a large amount of resources, and it is common for many users to be running their applications at the same time.
However, unlike in a desktop scenario, application performance is important—often critical—so it is common for each user to run on a dedicated part of the system.
In order to manage the allocation of resources to users, supercomputers run _workload managers_ (WLMs) that often implement a _job queue_.

The typical workflow with a WLM can be summarised as follows:
1. The user defines their _job_. This includes the application to be run and the amount of hardware resources that it will need, e.g. number of processor cores, amount of RAM, and so on. This is done on a login node.
2. The job is submitted to the system's queue. It is not unusual to have tens or hundreds of jobs in the queue at a given time.
3. The WLM looks at available resources together with queued jobs and any higher/lower priorities applied to them in order to decide which job(s) will run next.
4. When the requested resources are available and the job starts running, those resources (on compute nodes) become allocated to it and no other job will be able to use it until the current one is stopped.
5. The job is run according to its definition. It will stop when it either completes, crashes, or exceeds the requested resources, e.g. it has used more CPU time than the job definition requested.
6. When the job is done, its resources are freed so they can be used for other jobs.

BlueCrystal uses **SLURM** on Phase 4.

## BCp4 – SLURM

### Listing jobs

To see _all_ the jobs queued on the system, type `squeue`:

```bash
$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           1340878       gpu  2ycu2u1  ck14921 PD       0:00      1 (Resources)
           1340605       gpu   run_tf   yl1220 PD       0:00      1 (Priority)
           1340938       gpu run_atte  hd12584 PD       0:00      1 (Priority)
# Many lines omitted...
           1330979       cpu      m12  wk14463  R 11-02:07:08      1 compute480
           1329938       cpu texit000    ggpoh  R 12-14:19:34      1 compute329
           1328086       cpu      m22  wk14463  R 13-18:04:55      1 compute503
```

The state is usually either running (`R`) or pending (`PD`).
When a job is running, you can see which compute nodes it is using in the rightmost column.
When it is pending, the reason why it hasn't started yet is shown.

To see a _single user's jobs_, use `squeue -u <username>`:

```bash
$ squeue -u $USER
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           1340971       cpu     bash  ab12345  R       0:05      1 compute382
```

You can also filter jobs by reservation (`-R`) or account (`-A`):

```bash
$ squeue -R COSC024002
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           2481514 veryshort     bash  ab12345  R       0:06      1 compute084

$ squeue -A COSC024002
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           2481514 veryshort     bash  ab12345  R       0:15      1 compute084
```

You can see all the details about a job, even after it has completed, using `scontrol`:

```bash
$ scontrol show -d job 1340971
JobId=1340971 JobName=bash
   UserId=ab12345(999999) GroupId=mven(16621) MCS_label=N/A
   Priority=988 Nice=0 Account=default QOS=normal WCKey=*cosc17r
   JobState=COMPLETED Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=0 Reboot=0 ExitCode=0:0
   DerivedExitCode=0:0
   RunTime=00:05:22 TimeLimit=2-00:00:00 TimeMin=N/A
   SubmitTime=2018-09-25T12:22:37 EligibleTime=2018-09-25T12:22:37
   StartTime=2018-09-25T12:22:37 EndTime=2018-09-25T12:27:59 Deadline=N/A
   PreemptTime=None SuspendTime=None SecsPreSuspend=0
   Partition=cpu AllocNode:Sid=bc4login1:12671
   ReqNodeList=(null) ExcNodeList=(null)
   NodeList=compute382
   BatchHost=compute382
   NumNodes=1 NumCPUs=28 NumTasks=28 CPUs/Task=1 ReqB:S:C:T=0:0:*:*
   TRES=cpu=28,mem=28000M,node=1
   Socks/Node=* NtasksPerN:B:S:C=28:0:*:* CoreSpec=*
     Nodes=compute382 CPU_IDs=0-27 Mem=28000
   MinCPUsNode=28 MinMemoryCPU=1000M MinTmpDiskNode=0
   Features=(null) Gres=(null) Reservation=(null)
   OverSubscribe=OK Contiguous=0 Licenses=(null) Network=(null)
   Command=bash
   WorkDir=/mnt/storage/home/ab12345
   Power=
```

In SLURM, resources are organised into _partitions_ (as opposed to PBS queues), which can be listed with `sinfo`:

```bash
$ sinfo
PARTITION       AVAIL  TIMELIMIT  NODES  STATE NODELIST
veryshort          up    6:00:00      2    mix compute[473,507]
veryshort          up    6:00:00    442  alloc compute[078-263,265-273,275-381,383-441,443-445,450-472,474-506,508-525],highmem[10-13]
veryshort          up    6:00:00      8   idle compute[264,274,382,442,446-449]
cpu*               up 14-00:00:0      2    mix compute[473,507]
cpu*               up 14-00:00:0    438  alloc compute[078-263,265-273,275-381,383-441,443-445,450-472,474-506,508-525]
cpu*               up 14-00:00:0      8   idle compute[264,274,382,442,446-449]
hmem               up 14-00:00:0      8  alloc highmem[10-17]
gpu                up 7-00:00:00     25    mix gpu[01,03-14,16-24,27-28,30]
gpu_veryshort      up    1:00:00      2   idle gpu[31-32]
# Some lines omitted...
```

By state, nodes can be free (`idle`), fully in use (`alloc`), or partially in use (`mix`). Note that partitions are not necessarily disjoint.

You can query a specified partition only using `-p`:

```bash
$ sinfo -p hmem
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
hmem         up 14-00:00:0      8  alloc highmem[10-17]
```

More details and usage example about these commands are in the manpages: `man squeue`, `man sinfo`, `man scontrol`.

### Deleting jobs

Use `scancel` to remove jobs from the queue or stop in-progress ones:

```bash
$ scancel 1340971
```

### Submitting jobs

Unlike PBS, SLURM distinguishes between two ways to run jobs:

- You can run your application directly using `srun`. You can use your binary directly, and resources will be allocated and freed automatically. You cannot do this with PBS.
- You can use a job script, which is submitted using `sbatch`. This is similar to using `qsub`.

Both approaches take the same arguments, and the syntax is as follows:

```bash
$ srun   [options] /path/to/binary
$ sbatch [options] /path/to/script
```

The following table lists a few common job control options:

| SLURM argument  | Meaning             |
| --------------- | ------------------- |
| `-p <partition>`    | The partition to run on |
| `-N <n>` | Request `n` nodes |
| `--ntasks-per-node <c>` | Request `c` tasks to be run on each node (often related to number of cores required) |
| `-t <t>` | Specifies that your job should be allowed to run for at most `t` (time). You should specify `t` as `hh:mm:ss` |
| `-J <name>` | Sets the job's name, so you can easily identify it later |
| `-o <file>` | Sets a name for the file where the job's output will be saved. If you don't set this, an automatically generated named will be used |
| `--exclusive` | Does not allow other jobs to be scheduled on your allocated compute nodes, even if you don't fully utilise their resources |
| `--gres=gpu:<g>` | Request `g` GPUs. GPUs are only present in nodes in the `gpu` partition, where each node has 2 GPUs |
| `-A <account>` | SLURM allows users to be organised into groups (_accounts_) that share resources. A user can be part of serveral groups simultaneously, so `-A` is used to pick which account to use |
| `--reservation <name>` | Nodes can be reserved for subsets of users. If you are part of a reservation, specify its name to use it |

If you compare this to the equivalent PBS table above, note that `-j oe` and `-V` are implied on SLURM.

**Important**: If you are taking the COMS30005 unit in 2019, there is a reservation set up for you on the `veryshort` partition to make sure you have a number of dedicated nodes available for the duration of the course. To use the reservation, add the following options to your job submission commands: `-p veryshort -A COMS30005 --reservation COMS30005`.

#### Job files

SLURM job files work virtually the same way as [PBS job files](#job-files) (please read this section before continuing if you haven't used job files before).
The notable differences are:

- Job paramters are prefixed with `#SBATCH` in the script.
- You need to use SLURM arguments, and the script is submitted using `sbatch`.

Here is the same example script shown above, but using SLURM paramters instead:

```bash
$ cat my.job
#!/bin/bash
#SBATCH --job-name LBM
#SBATCH -o lbm.out
#SBATCH -p veryshort
#SBATCH -A COMS30005
#SBATCH --reservation COMS30005
#SBATCH --nodes 1
#SBATCH --ntasks-per-node 16
#SBATCH -t 00:05:00

$HOME/work/d2q9-bgk
```

#### Interactive jobs

Use `srun --pty bash` to run an interactive job:

```bash
[ab12345@bc4login1 ~]$ srun -N1 --tasks-per-node 28 --pty bash
[ab12345@compute084 ~]$ echo "Now running on a compute node."
Now running on a compute node.
```

Please note that using an interactive session will keep the node(s) requested allocated _for the whole session_, not just when you are actively running commands.
Since all the resources are shared with the other users on the system, **only use an interactive job for tasks that you cannot do on a login node or through job scripts, and give up your allocation as soon as you have finished**.

### More resources

You can find documentation both in the manpages, e.g. `man srun`, as well as [online](https://slurm.schedmd.com/documentation.html).
You can also find [online version of the manpages](https://slurm.schedmd.com/man_index.html).
However, please note that web-based documentation may target a different version than what is used on BCp4, and not all features supported by SLURM may be enabled and available on BlueCrystal.
If in doubt, always check the manpage on the system.

You may also find useful the [SLURM command summary sheet](https://slurm.schedmd.com/pdfs/summary.pdf) or the [SLURM job management cheat sheet from the Iowa State University](https://gif.biotech.iastate.edu/slurm-slurm-job-management-cheat-sheet).
If you have previously used PBS, [this page from the University of Southern California](https://hpcc.usc.edu/support/documentation/pbs-to-slurm/) gives a mapping between commonly used SLURM and PBS commands.

ACRC have [online documentation for BCp4](https://www.acrc.bris.ac.uk/protected/bc4-docs/).

## Environment modules and queueing systems

One important difference between PBS and SLURM is how environment is preserved when you run a job:

- For **PBS**, _the job runs in a clean environment_, so any modules you load or variables you set in your interactive session on the login node will not be automatically forwarded to compute jobs. However, your shell start-up scripts will still be executed, e.g. commands in `.bashrc` will be run.
- For **SLURM**, _jobs start in the same environment that you had on the login node_, so you don't _need to_ load all the required module in your job script.

Regardless of which system you're using, it is good practice to only have loaded those modules that are required for your job.
This avoid issues where your job is affected by modules you may have loaded for testing (or other purpuses) and forgot about.
Therefore, consider starting your job script with a `module purge`, followed by `load`s for _only the required modules_.
