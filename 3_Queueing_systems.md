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
The two are very similar, although commands and parameters are named differently.
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

Note that the `S` column show the _state_ of the jobs: `H`eld, `Q`ueued, `R`unning, `C`ompleted, `E`rrored.

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
7452534.master.c     ap13004     veryshor LBM                 --    --     --     --   01:00:00  C       --
7452536.master.c     ap13004     teaching test3             13380   --     --     --   00:10:00  R  00:00:04
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
$ qsub <options> /path/to/application
```

The following table lists a few common options:

| `qsub` argument | Meaning             |
| --------------- | ------------------- |
| `-q <queue>`    | The queue to run on |
| `-lnodes=<n>:ppn=<c>` | Request `n` nodes and `c` cores on each node |
| `-lwalltime=<t>` | Specifies that your job should be allowed to run for at most `t` (time). You should specify `t` as `hh:mm:ss` |
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


For example, to run LBM on a single node and 16 cores for a maximum of 10 minutes, I could use the following command:

```bash
$ qsub -q teaching -lnodes=1:ppn=16,walltime=0:10:0 ./d2q9-bgk
```

To run the same application on 4 full nodes:

```bash
$ qsub -q teaching -lnodes=4:ppn=16 mpirun ./d2q9-bgk
```

**Note**: If your application uses MPI, you will need to use an MPI launcher.
Read [the MPI page](5_MPI.md) for more details.

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

$HOME/work/d2q9-bgk
```

This can be run by simply using:

```bash
$ qsub my.job
```

The result is the same as specifying the commands directly on the CLI:

```bash
$ qsub -N LBM -o lbm.out -j oe -q teaching -l nodes=1:ppn=16,walltime=00:05:00 $HOME/work/d2q9-bgk
```

Of course, in a script you can have more than a single command, and they will all be run on the allocated compute node(s).
There are also special environment variables that are set by PBS when it runs your job and which can be used to get information about your allocation; [this page](https://wiki.hpcc.msu.edu/display/hpccdocs/Advanced+Scripting+using+PBS+Environment+Variables) lists some of them.

<!-- TODO: Create and link to a full jobfile example -->

**Note**: Make sure you pass your job script to `qsub`,
If you don't, it will just be run as a regular shell script _on the login node_.
The following are examples of the job file **not** being submitted to the queue:

```bash
$ ./my.job
$ bash my.job
```

### More resources

Because there are several variants of forks and PBS, each with different configurations and features enabled, the best place to look up how to use a particular command (or which command to use) is in the manpages.
Start with `man pbs` for a brief description of each command, then look at its particular manpage, e.g. `man qsub`.
Check the `SEE ALSO` section at the bottom of those manpages for a list of other related manpages.

Iowa State University have a [PBS job management cheat sheet](https://gif.biotech.iastate.edu/torque-pbs-job-management-cheat-sheet) that summarises some of the content presented above.
If you have previously used SLURM, [this page from the University of Southern California](https://hpcc.usc.edu/support/documentation/pbs-to-slurm/) gives a mapping between commonly used SLURM and PBS commands.

## BCp4 – SLURM

<!-- TODO: SLURM section -->

_Work in progress._

### More resources

You can find documentation both in the manpages, e.g. `man srun`, as well as [online](https://slurm.schedmd.com/documentation.html).
You can also find [online version of the manpages](https://slurm.schedmd.com/man_index.html).
However, please note that web-based documentation may target a different version than what is used on BCp4, and not all features supported by SLURM may be enabled and available on BlueCrystal.
If in doubt, always check the manpage on the system.

You may also find useful the [SLURM command summary sheet](https://slurm.schedmd.com/pdfs/summary.pdf) or the [SLURM job management cheat sheet from the Iowa State University](https://gif.biotech.iastate.edu/slurm-slurm-job-management-cheat-sheet).
If you have previously used PBS, [this page from the University of Southern California](https://hpcc.usc.edu/support/documentation/pbs-to-slurm/) gives a mapping between commonly used SLURM and PBS commands.

## Common commands

## Environment modules and queueing systems

One important difference between PBS and SLURM is how environment is preserved when you run a job:

- For **PBS**, _the job runs in a clean environment_, so any modules you load or variables you set in your interactive session on the login node will not be automatically forwarded to compute jobs. However, your shell start-up scripts will still be executed, e.g. commands in `.bashrc` will be run.
- For **SLURM**, _jobs start in the same environment that you had on the login node_, so you don't _need to_ load all the required module in your job script.

Regardless of which system you're using, it is good practice to only have loaded those modules that are required for your job.
This avoid issues where your job is affected by modules you may have loaded for testing (or other purpuses) and forgot about.
Therefore, consider starting your job script with a `module purge`, followed by `load`s for _only the required modules_.
