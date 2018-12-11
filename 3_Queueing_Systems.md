# Queueing Systems

**Important**: Before reading this section, you need to be familiar with the concepts of _login nodes_ and _compute nodes_.
In a typical supercomputer system, when you connect to the machine you will be on a _login node_, which is shared with other users connected to the system.
This is where you usually compile your application and set up your environment.
In contrast, applications are run on _compute nodes_, which are allocated and freed as needed, and are generally dedicated to a single user.

Supercomputers house a large amount of resources, and it is common for many users to be running their applications at the same time.
However, unlike in a desktop scenario, application performance is important—often critical—so it is common for each user to run on a dedicated part of the system.
In order to manage allocating resources to users, supercomputers run _workload managers_ (WLMs) that often implement a _job queue_.

Typical workflow with a WLM can be summarised as follows:
1. The user defines their _job_. This includes the application to be run and the amount of hardware resources that it will need, e.g. number of processor cores, amount of RAM, and so on. This is done on a login node.
2. The job is submitted to the system's queue. It is not unusual to have 10s–100s of jobs in the queue at a given time.
3. The WLM looks at available resources together with queued jobs and any higher/lower priorities applied to them to decide which job(s) will run next.
4. When the requested resources are available and the job starts running, those resources (on compute nodes) become allocated to it and no other job will be able to use it until the current one is stopped.
5. The job is run according to its definition. It will stop when it either completes, crashes, or exceeds the requested time.
6. When the job is done, its resources are freed so they can be used for other jobs.

BlueCrystal uses **PBS** on Phase 3 and **SLURM** on Phase 4.
The two are very similar, although commands and parameters are named differently, and there is an [important difference regarding how they preserve environment variables by default](#environment-modules-and-queueing-systems) of which you need to be aware.
The following sections show commonly used options for each system.

## BCp3 – PBS

### Listing jobs

To see _all_ the jobs queued on the system, type `qstat`:

```bash
$ qstat
Job ID                    Name             User            Time Use S Queue
------------------------- ---------------- --------------- -------- - -----
7147359.master             run_R.sh         epxlp                  0 H short
7375113.master             ELASTICITY.sh    ac17547                0 Q long
7405428.master             NLTHA1.bat       ll15668                0 Q long
7408732.master             runMB.mpi        bzxdp           5741:19: R long
7408946.master             ...3773_7_Rstart jl15245         5561:56: R long
7413612.master             create_viz       oc13378         307:00:4 R himem
7414883.master             ...eit-duos-chr6 hs12828         3953:11: R himem
# Many lines omitted...
```

Note that the `S` column show the _state_ of the jobs: `H`eld, `Q`ueued, `R`unning, `E`nding, `C`ompleted.

There are many options that can be given to `qstat` to filter and organise output.
For example, a system can have _multiple queues_, which are often organised by the type of the hardware contained or the maximum amount of resources that can be requested.
`qstat -q` will show all the queues available and the number of jobs in each:

```bash
$ qstat -q

Queue            Memory CPU Time Walltime Node  Run Que Lm  State
---------------- ------ -------- -------- ----  --- --- --  -----
gpu                --      --    360:00:0   --   26   0 --   E R
long               --      --    360:00:0   --  167   2 --   E R
medium             --      --    240:00:0   --   75  66 --   E R
testq              --      --    01:00:00   --    0   0 --   E R
veryshort          --      --    12:00:00   --   17   5 --   E R
visualisation      --      --    576:00:0   --    0   0 --   E R
himem              --      --    360:00:0   --   13   2 --   E R
teaching           --      --    24:00:00   --    0   0 --   E R
# Many lines omitted...
```

If you're taking COMS30005, you will want to use the `teaching` queue, as this is reserved for students' jobs.
You can check the summary of _a single queue_ by giving its name to `qstat -q`:

```bash
$ qstat -q teaching

Queue            Memory CPU Time Walltime Node  Run Que Lm  State
---------------- ------ -------- -------- ----  --- --- --  -----
teaching           --      --    24:00:00   --    1   0 --   E R
                                               ----- -----
                                                   1     0
```

To see a _single user's jobs_, use `qstat -u <username>`:

```bash
$ qstat -u $USER

                                                                               Req'd    Req'd       Elap
Job ID               Username    Queue    Jobname          SessID NDS   TSK    Memory   Time    S   Time
-------------------- ----------- -------- ---------------- ------ ----- ------ ------ --------- - ---------
7452534.master.c     ab12345     veryshor LBM                 --    --     --     --   01:00:00  C       --
7452536.master.c     ab12345     teaching test3             13380   --     --     --   00:10:00  R  00:00:04
```

In the output above, my `LBM` job ran on the `veryshort` queue and is now `C`ompleted, while `test3` is still `R`unnin on the `teaching` queue.

See `man qstat` for the complete list of options.

### Deleting jobs

You can cancel a job using `qdel <jobid>`. For example, the following command will cancel the `test3` job above:

```bash
$ qdel 7452536
```

Note that you _only need the numeric ID_ up to the `.`.

### Submitting jobs

To run applications on compute nodes, you need to _submit jobs_ to a queue; this is done using `qsub`.
You can submit several jobs at the same time, but there is no guarantee that they will be executed simultaneously.
If you do submit more than one job at a time, keep in mind that BlueCrystal is a shared resource, so be nice to everyone else using the system.

When you use `qsub`, you need to specify the resources that your job will require—if you don't, you will be given some system defaults, which you should not rely on.
You will generally want to specify the number of nodes and processor cores that your application needs, the required time, and the queue to use.
This is done using the following syntax:

```bash
$ qsub <options> /path/to/script
```

**Note**: `qsub` will _only_ accept shell scripts, so you _can't run your application's binary directly_.

The following table lists a few common job control options:

| `qsub` argument | Meaning             |
| --------------- | ------------------- |
| `-q <queue>`    | The queue to run on |
| `-l nodes=<n>:ppn=<c>` | Request `n` nodes and `c` cores on each node |
| `-l walltime=<t>` | Specifies that your job should be allowed to run for at most `t` (time). You should specify `t` as `hh:mm:ss` |
| `-N <name>` | Sets the job's name, so you can easily identify it later |
| `-o <file>` | Sets a name for the file where the job's output will be saved. If you don't set this, an automatically generated named will be used |
| `-j oe` | By default, the job's standard output and standard error are saved in separate files. Setting this option will use a single file for both |
| `-v <VAR>=<val>` | Export the `VAR` environment variable with the value `val` to the job's environment |
| `-V` | Export _all_ environment variables from the login session to the job's environment. Only use this if you have a specific reason for doing so (and you know what you're doing!) |


Note that the `-l` options can be combined using a `,`, so the following two are equivalent:

```bash
$ qsub -l walltime=1:0:0 -l nodes=1:ppn=16
$ qsub -l walltime=1:0:0,nodes=1:ppn=16
```

**Important**: Make sure you only ask for resources that can be given to you.
If your request cannot be fulfilled, your job will be blocked in the queue forever.
For example, the compute nodes in BCp3 are dual-socket 8-core machines, so asking for more than 16 cores per node will prevent your job from running.


For example, to run a job on a single node and 16 cores for a maximum of 10 minutes, I could use the following command:

```bash
$ qsub -q teaching -lnodes=1:ppn=16,walltime=0:10:0 ./my.job
```

To run the same application on 4 full nodes:

```bash
$ qsub -q teaching -lnodes=4:ppn=16 ./my.job
```

**Note**: If your application uses MPI, you will need to use an MPI launcher.
Read [the MPI page](6_MPI.md) for more details.

#### Job files

Since most of the time you will be running the same configuration—or, at least, a few similar configurations—for an application, it is a good idea to write a _job script_ instead of typing the commands every time.
A job script is just a regular shell script, but it allows you to include PBS parameters in the script itself.
To use this:

1. Create a script that runs your application.
2. At the top of the file, add each option that you would pass to `qsub` on a line starting with `#PBS`.
3. Use this script as your application when running `qsub`.

Here is a simple example job script:

```bash
$ cat my.job
#!/bin/bash
#PBS -N LBM
#PBS -o lbm.out
#PBS -j oe
#PBS -q teaching
#PBS -l nodes=1:ppn=16,walltime=00:05:00

$HOME/work/a.out
```

This can be run by simply using:

```bash
$ qsub my.job
```

The result is the same as specifying the commands directly on the CLI:

```bash
$ qsub -N LBM -o lbm.out -j oe -q teaching -l nodes=1:ppn=16,walltime=00:05:00 $HOME/work/a.out
```

If the same argument is specified both on the command line and inside the jobs file, its value on the command line takes precedence.

Of course, in a script you can have more than a single command, and they will all be run on the allocated compute node(s).
There are also special environment variables that are set by PBS when it runs your job and which can be used to get information about your allocation; [this page](https://wiki.hpcc.msu.edu/display/hpccdocs/Advanced+Scripting+using+PBS+Environment+Variables) lists some of them.

<!-- TODO: Create and link to a full jobfile example -->

**Note**: Make sure you pass your job script _to `qsub`_,
If you don't, it will just be run as a regular shell script _on the login node_.
The following are examples of the job file **not** being submitted to the queue:

```bash
$ ./my.job
$ bash my.job
```

When the job is submitted, the queueing system caches a copy of the job script used.
This means that if you change the script while the job is waiting in the queue, changes will _not_ be reflected in the job and the original version used for submission will be used.
If you need to mke changes, you will need to cancel any existing jobs and submit new ones.

#### Interactive jobs

Sometimes it can be useful to be able to run commands _on a compute node_ interactively, as if you were directly connected to it.
You can do this through an _interactive job_, for which you can ask with `qsub -I`.
You can use other `qsub` arguments to select your resources, e.g. as above.
When the job begins, you will be give a shell on one of the compute nodes allocated to your job (which will reflect in you shell prompt changing), and commands you type there will run directly on the node.
To finish your job, simply `exit` from your shell.
The following is an example of using an interactive job:

```bash
[ab12345@newblue1 ~]$ echo "This runs on a login node."
This runs on a login node.
[ab12345@newblue1 ~]$ qsub -I -lnodes=1,walltime=1:0:0 -q teaching
qsub: waiting for job 7456439.master.cm.cluster to start
qsub: job 7456439.master.cm.cluster ready

[ab12345@node31-033 ~]$ echo "This runs on a compute node."
This runs on a compute node.
[ab12345@node31-033 ~]$ exit
logout

qsub: job 7456439.master.cm.cluster completed
[ab12345@newblue1 ~]$ echo "Now back to the login node."
Now back to the login node.
```

Please note that using an interactive session will keep the node(s) requested allocated _for the whole session_, not just when you are actively running commands.
Since all the resources are shared with the other users on the system, **only use an interactive job for tasks that you cannot do on a login node or through job scripts, and give up your allocation as soon as you have finished**.

### More resources

Because there are several variants of forks and PBS, each with different configurations and features enabled, the best place to look up how to use a particular command (or which command to use) is in the manpages.
Start with `man pbs` for a brief description of each command, then look at its particular manpage, e.g. `man qsub`.
Check the `SEE ALSO` section at the bottom of those manpages for a list of other related manpages.

Iowa State University have a [PBS job management cheat sheet](https://gif.biotech.iastate.edu/torque-pbs-job-management-cheat-sheet) that summarises some of the content presented above.
If you have previously used SLURM, [this page from the University of Southern California](https://hpcc.usc.edu/support/documentation/pbs-to-slurm/) gives a mapping between commonly used SLURM and PBS commands.

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

If you compare this to the equivalent PBS table above, note that `-j oe` and `-V` are implied on SLURM.

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
#SBATCH -p teaching
#SBATCH --nodes 1
#SBATCH --ntasks-per-node 16
#SBATCH -t 00:05:00

$HOME/work/d2q9-bgk
```

#### Interactive jobs

Use `srun --pty bash` to run an interactive job:

```bash
[ab12345@bc4login1 ~]$ srun -N1 --tasks-per-node 28 --pty bash
[ab12345@compute382 ~]$ echo "Now running on a compute node."
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

## Environment modules and queueing systems

One important difference between PBS and SLURM is how environment is preserved when you run a job:

- For **PBS**, _the job runs in a clean environment_, so any modules you load or variables you set in your interactive session on the login node will not be automatically forwarded to compute jobs. However, your shell start-up scripts will still be executed, e.g. commands in `.bashrc` will be run.
- For **SLURM**, _jobs start in the same environment that you had on the login node_, so you don't _need to_ load all the required module in your job script.

Regardless of which system you're using, it is good practice to only have loaded those modules that are required for your job.
This avoid issues where your job is affected by modules you may have loaded for testing (or other purpuses) and forgot about.
Therefore, consider starting your job script with a `module purge`, followed by `load`s for _only the required modules_.
