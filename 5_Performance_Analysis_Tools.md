Performance Analysis Tools
==========================

As the name suggests, High _Performance_ Computing revolves around analysing and improving the performance of various applications and hardware platforms.
In order to identify, understand, an attempt to tackle performance issues, you first need to collect a range of data that shows how your application and host hardware interact.
This section presents some of the tools that are commonly used to collect and interpret such data.
It is by no means an exhaustive list, but it should provide a good starting point.

Note that you are not _required_ to use everything shown below.
While it is likely that you will need _some_ tools to perform your performance analysis, you are not expected to utilise _all_ of them.
You should try to identify which tools are most useful for the result _you_ are trying to achieve and focus on getting the most out of them.

While all tools apart from those in the [_Other tools_ section](#other-notable-tools) should work on BlueCrystal, not all of them may be installed by default.
For the open-source tools, you can always download the source code and compile them yourself.
For the Intel tools, if you require anything that is not installed, [obtaining a student licence](https://software.intel.com/en-us/qualify-for-free-software/student) and installing them on your own machine is likely the easiest option.

<!-- TODO: GPU: nvprof, OpenCL Intercept Layer, oclgrind -->

## Summary of tools

This table shows a summary of all the tools presented below and their support on BCp3.

| Tool       | Installed | Compatible | Usage                                                 |
| ---------- | :-------: | :--------: | ----------------------------------------------------- |
| perf       | ✔         | ✔          | Run `perf`                                            |
| gprof      | ✔         | ✔          | Compile with `gcc -pg`, run `gprof`                   |
| PAPI       | ✔         | ✔          | `module load libraries/{gnu,intel}_builds/papi-5.3.0` |
| Valgrind   | ✔         | ✔          | Run `valgrind`                                        |
| TAU        | ✔         | ✔          | See `module av \|& grep -i tau-`                      |
| vTune      | ✔         | ✔          | `module load intel-cluster-studio/vtune/vtune-2015`   |
| Advisor    | ✗         | ✗          | Install through student licence on own machine        |
| MPI Tracer | ✗         | ✗          | Install through student licence on own machine        |
| Extrae     | ✗         | ✔          | Install from source                                   |

## Command-line tools

### perf

One of the easiest ways you can start obtaining performance data about your application is using a tool that comes with the Linux kernel: `perf`.
In fact, `perf` is a collection of tools, and you pick which tools to use through arguments.
The general syntax is:

```
$ perf <tool> [<options>] -- <application>
```

You can use perf to record a profile of your application's run, then look at it alongside the code using `perf record` and `perf annotate`, respectively.
It also offers a quick way to access hardware performance counters: `perf stat`.
For the latter, run `perf list` to show the available counters, then select the ones you want to query with `-e`.
Here is an example where I've picked some of the counters at the top of the list:

```
$ perf stat -e cycles,instructions,cache-references,cache-misses -- ./test

 Performance counter stats for './test':

       292,217,550 cycles                    #    0.000 GHz
       675,095,645 instructions              #    2.31  insns per cycle
         2,298,419 cache-references
             5,093 cache-misses              #    0.222 % of all cache refs

       0.094668510 seconds time elapsed
```

See `man perf` for a list of all the available tools and their associated manpages.
There is also [plenty of documentation available online](https://perf.wiki.kernel.org/index.php/Tutorial).

### gprof

grpof is a GNU profiler for Linux.
It is a command line tool that can sample your application at run time and produce a summary of where time was spent executing code.
It is available by default on BlueCrystal, but it can only be used with the GNU compiler.

To use gprof, follow these steps:

1. Compile your application using GCC and the `-pg` flag.
2. Run your produced binary. This will generate a `gmon.out` file.
3. To produce a profiler report, use the following syntax:
      ```
      $ gprof [<options>] <executable> gmon.out
      ```

Here is an example for the [STREAM benchmark](https://www.cs.virginia.edu/stream/), where the `-l` option is used to show a line-by-line profile:

```
$ gcc -O3 -fopenmp -pg -g -o stream.gprof stream.c
$ ./stream.gprof
$ gprof -l stream.gprof gmon.out
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total
 time   seconds   seconds    calls  Ts/call  Ts/call  name
 31.77      0.72     0.72                             main._omp_fn.7 (stream.c:345 @ 400a70)
 30.44      1.41     0.69                             main._omp_fn.6 (stream.c:335 @ 400bb0)
 28.24      2.05     0.64                             main._omp_fn.5 (stream.c:325 @ 400cf0)
  2.21      2.10     0.05                             main._omp_fn.2 (stream.c:267 @ 400fac)
  2.21      2.15     0.05                             main._omp_fn.3 (stream.c:288 @ 400e90)
  1.76      2.19     0.04                             main._omp_fn.2 (stream.c:269 @ 400fd4)
  1.32      2.22     0.03                             checkSTREAMresults (stream.c:463 @ 4014c4)
  1.10      2.25     0.03                             checkSTREAMresults (stream.c:465 @ 4014ec)
  0.44      2.26     0.01                             main._omp_fn.3 (stream.c:286 @ 400e7d)
  0.44      2.27     0.01                             main._omp_fn.5 (stream.c:323 @ 400ccd)
  0.22      2.27     0.01                             checkSTREAMresults (stream.c:464 @ 4014d8)
  0.00      2.27     0.00        1     0.00     0.00  checkSTREAMresults (stream.c:434 @ 401480)
  0.00      2.27     0.00        1     0.00     0.00  checktick (stream.c:385 @ 401160)
# More output omitted
```

Note that gprof doesn't "understand" OpenMP, so there is no breakdown by thread or by parallel region.
It will also not support MPI.
For more usage instructions, see `man gprof` or [the online documentation](https://sourceware.org/binutils/docs/gprof/index.html#SEC_Contents).

### PAPI

[PAPI](http://icl.cs.utk.edu/papi/) is a collection of tools and libraries to instrument and record the performance of an application.
It provides three main components:

- Binaries that can be used directly to obtain performance data. This is one of the easiest ways to access hardware counters.
- A programming interface (API) that allows you to call on PAPI from your code and record data programmatically. This requires source code modifications.
- A library that other tools can use and build on. For example, valgrind and Extrae (presented below) can be compiled with PAPI support to enable additional data to be collected.

PAPI is available as a module on BCp3:

```
$ module av |& grep -i papi
libraries/gnu_builds/papi-5.3.0
libraries/intel_builds/papi-5.3.0
```

If you need a newer version, it can also be [compiled from source](http://icl.cs.utk.edu/papi/software/index.html).

A good place to start is checking which hardware data PAPI can access:

```
$ papi_avail
Available events and hardware information.

    Name        Code    Avail Deriv Description (Note)
PAPI_L1_DCM  0x80000000  Yes   No   Level 1 data cache misses
PAPI_L1_ICM  0x80000001  Yes   No   Level 1 instruction cache misses
PAPI_L2_DCM  0x80000002  Yes   Yes  Level 2 data cache misses
PAPI_L2_ICM  0x80000003  Yes   No   Level 2 instruction cache misses
PAPI_L3_DCM  0x80000004  No    No   Level 3 data cache misses
PAPI_L3_ICM  0x80000005  No    No   Level 3 instruction cache misses
# More lines omitted
```

Then, read [the online manual](http://icl.cs.utk.edu/projects/papi/wiki/Main_Page) to learn how to use the library's functionality.

### Valgrind

[Valgrind](http://valgrind.org/) is a collection of tools for instrumentation and analysis of binaries.
[Memcheck](http://valgrind.org/info/tools.html#memcheck) is the tool for debugging memory issues, which may be useful if you have issues with invalid accesses or segmentation faults.
There are also [Cachegrind and Callgrind](http://valgrind.org/info/tools.html#cachegrind) which can give you insight into how you application uses the cache, although you should be careful if you try to use this with MPI or OpenMP.

Here is sample basic output from the Cachegrind tool:

```
$ valgrind --tool=cachegrind ./test

==31134==
==31134== I   refs:      832,790
==31134== I1  misses:      1,255
==31134== LLi misses:      1,207
==31134== I1  miss rate:    0.15%
==31134== LLi miss rate:    0.14%
==31134==
==31134== D   refs:      311,489  (226,286 rd   + 85,203 wr)
==31134== D1  misses:      9,581  (  8,034 rd   +  1,547 wr)
==31134== LLd misses:      5,996  (  4,631 rd   +  1,365 wr)
==31134== D1  miss rate:     3.1% (    3.6%     +    1.8%  )
==31134== LLd miss rate:     1.9% (    2.0%     +    1.6%  )
==31134==
==31134== LL refs:        10,836  (  9,289 rd   +  1,547 wr)
==31134== LL misses:       7,203  (  5,838 rd   +  1,365 wr)
==31134== LL miss rate:      0.6% (    0.6%     +    1.6%  )
```

Valgrind is already installed on BCp3 and you don't need to load any module; just run `valgrind`.
Documentation is [available online](http://valgrind.org/docs/manual/manual.html) and there is also a [quick start guide](http://valgrind.org/docs/manual/QuickStart.html).
See `man valgrind` for CLI usage information.

### TAU

[TAU](https://www.cs.uoregon.edu/research/tau/home.php) is a flexible open-source profiler.
It supports hardware counters, OpenMP and MPI code, and some version even work with GPU applications.
On BCp3, there are several modules available, and you should choose the one for the compiler and library you are using:

```
$ module av |& grep -i tau-
tools/gnu_builds/tau-2.23.1-openmp
tools/gnu_builds/tau-2.23.1-openmpi
tools/intel_builds/tau-2.23-openmp
tools/intel_builds/tau-2.23-openmpi
tools/intel_builds/tau-2.23.1-openmp
tools/intel_builds/tau-2.23.1-openmpi
tools/intel_builds/tau-2.27-intel-16u2-mpi
```

The tool has plenty of [documentation online](https://www.cs.uoregon.edu/research/tau/docs.php), including video guides.
There is also a [tutorial for new users](http://tau.uoregon.edu/tau.ppt), which is a good place to start.

Basic usage is as follows:

1. Compile your application using the `tau_cc.sh` wrapper script. This is used to specify what TAU will record and then to call on the underlying compiler.
2. Run the application. This will produce `profile.*` files for each of your MPI ranks and OpenMP threads.
3. Run `pprof` display a text report of the collected profiler data. There are other visualisations available, including GUIs. See the documentation for more details.

The following is a basic example:

```
$ module load tools/gnu_builds/tau-2.23.1-openmp
$ tau_cc.sh -O3 -fopenmp -o stream.tau.gcc stream.c
$ OMP_NUM_THREADS=16 ./stream.tau.gcc
$ ls -d profile.0.0.*
profile.0.0.0  profile.0.0.10  profile.0.0.12  profile.0.0.14  profile.0.0.2  profile.0.0.4  profile.0.0.6  profile.0.0.8
profile.0.0.1  profile.0.0.11  profile.0.0.13  profile.0.0.15  profile.0.0.3  profile.0.0.5  profile.0.0.7  profile.0.0.9
$ pprof
NODE 0;CONTEXT 0;THREAD 0:
---------------------------------------------------------------------------------------
%Time    Exclusive    Inclusive       #Call      #Subrs  Inclusive Name
              msec   total msec                          usec/call
---------------------------------------------------------------------------------------
100.0          143        1,353           1          44    1353319 .TAU application
 89.4            1        1,210          44          44      27504 parallel begin/end [OpenMP]
 88.9        0.729        1,202          42          42      28639 for enter/exit [OpenMP]
 46.4        0.002          627           1           1     627323 parallelfor (parallel begin/end) [OpenMP location: file:/panfs/panasas01/cosc/ap13004/workspace/STREAM/stream.c <267, 272>]
 46.4          321          627           1           1     627306 parallelfor (loop body) [OpenMP location: file:/panfs/panasas01/cosc/ap13004/workspace/STREAM/stream.c <267, 272>]
 32.7            3          443          44          44      10069 barrier enter/exit [OpenMP]
# More output omitted
```

Note that TAU is a complex tool and may require significant effort to learn.
We suggest you also consider the Intel tools below, [vTune](#intel-vtune) and [Advisor](#intel-advisor), before you decide to use TAU as you main profiler.

### likwid

A newer collection of tools aiming to simplify performance analysis and benchmarking is [likwid](https://github.com/RRZE-HPC/likwid).
It can access hardware counters, run microbenchmarks to reveal some peak capabilities of your platform, and help with binding threads and MPI ranks.
One of its aims is to present results in a user-friendly way.

Here is a snippet of the output produced when using likwid to look at L2 cache counters on the STREAM benchmark:

```
$ env OMP_NUM_THREADS=1 likwid-perfctr -g L2 ./stream.ivy.intel
+----------------------------+---------+--------+-----+--------+------------+
|            Event           | Counter |   Sum  | Min |   Max  |     Avg    |
+----------------------------+---------+--------+-----+--------+------------+
|   INSTR_RETIRED_ANY STAT   |  FIXC0  |  95886 |   0 |  51966 |  3995.2500 |
| CPU_CLK_UNHALTED_CORE STAT |  FIXC1  | 519359 |   0 | 217214 | 21639.9583 |
|  CPU_CLK_UNHALTED_REF STAT |  FIXC2  | 808272 |   0 | 324459 |      33678 |
|    L1D_REPLACEMENT STAT    |   PMC0  |   4327 |   0 |   1921 |   180.2917 |
|      L1D_M_EVICT STAT      |   PMC1  |   1073 |   0 |    454 |    44.7083 |
|     ICACHE_MISSES STAT     |   PMC2  |  13846 |   0 |   6563 |   576.9167 |
+----------------------------+---------+--------+-----+--------+------------+
```

If you want to try likwid, you will need to install it from source.
There is a guide on how to do this, along with detailed usage instructions and examples, on [their wiki](https://github.com/RRZE-HPC/likwid/wiki).
Note that this is relatively new tool, so there are no guarantees about how easy it is to install or how well it works on BlueCrystal.

## Graphical interface tools

### Intel vTune

[vTune Amplifier](https://software.intel.com/en-us/vtune) is a graphical toolkit for performance analysis from Intel.
It contains a profiler that supports both OpenMP and MPI, and which can access a wide range of hardware counters.
The GUI then provides various options to visualise the results.

![vTune](https://i.imgur.com/pDVFa8x.png)
_Example of the vTune Amplifier interface_

To use vTune on BCp3, load its module, then run the GUI:

```
$ module load intel-cluster-studio/vtune/vtune-2015
$ amplxe-gui &
```

Of course, you will need to enable SSH X forwarding to be able to view the GUI on your machine.
When you connect to BlueCrystal (and any other intermediate hops you may be jumping though), use `ssh -X`.
This was discussed in the [_Connecting_ section](1_Connecting_to_BlueCrystal.md#running-graphical-programs).

Once the GUI is running, you will need to crete a new _project_ for each binary that you want to analyse.
The _Hotspots_ analysis is a good place to start, as it will give you a high-level overview of what's happening in your program.

Intel have a [quick start guide](https://software.intel.com/en-us/get-started-with-vtune-linux-os) which we recommend reading to introduce you to vTune.
The same page links to [more tutorials](https://software.intel.com/en-us/articles/intel-vtune-amplifier-tutorials) that cover various tasks which can be done in vTune.
The guides have step-by-step instructions, including screenshots and explanations of the UI elements, so they are well worth the time.

vTune can also be used from the command line, which doesn't require you to use X forwarding over SSH.
If you want to use this approach, the command is called `amplxe-cl` and you can find [a guide on Intel's website](https://software.intel.com/en-us/vtune-amplifier-help-running-command-line-analysis).

**Important**: There licence on BlueCrystal only allows a limited number of users to run vTune simultaneously, and it is likely that this number is significantly smaller than the number of students on the unit.
Therefore, occasionally you may be unable to run vTune if it is also being used by many others at the same time, e.g. during labs.
The only thing you can do is wait until a licence becomes available.
However, it also means that you should terminate your sessions as soon as possible, so that you don't hold up a licence unnecessarily.

### Intel Advisor

[Intel Advisor](https://software.intel.com/en-us/advisor) is a performance tool that focuses on vectorisation and parallelisation of applications.
As such, it is less comprehensive than vTune, but it can be easier to work with when vectorisation and threading is the main focus.
Advisor is not installed on BCp3 (partly due to some legacy configuration that make it difficult to run it), but you can install it on your own machine using the student licence program.

Advisor's binaries are named similarly to those in vTune.
The GUI is called `advixe-gui`, and the CLI interface is `advixe-cl`.

![Advisor table](https://i.imgur.com/HMytbaD.png) <br />
_Screenshot of the Advisor profiler table_

![Advisor mix](https://i.imgur.com/XkVQT1C.png) <br />
_Screenshot of Advisor's instruction mix summary_

We recommend using Advisor if you are attempting to vectorise your code.
The tool will show you exactly which loops were vectorised and the strategies used for each, and when loops aren't vectorised, it will suggest issues that prevented vectorisation.
Together with compiler reports, it should give you enough information to improve your code.
Advisor will also show estimated speed-ups from vectoristion, as well as data that you need if you want to plot a roofline graph.

Similarly to vTune, Intel have a [getting started guide](https://software.intel.com/en-us/get-started-with-advisor) which presents the commands and UI elements.
We recommend reading this first, then referring to [the usage guide](https://software.intel.com/en-us/advisor-user-guide) for further documentation.

### Intel MPI Trace Analyzer and Collector

When writing MPI programs, it is important that you understand your communication patterns and how this affect the performance (and correctness!) of your application.
One tool that can help with visualising your MPI application's structure and identifying potential issues is [Intel's MPI Trace tools](https://software.intel.com/en-us/intel-trace-analyzer).

![ITAC](https://software.intel.com/sites/default/files/managed/f0/b5/intel-trace-analyzer-app.png) <br />
_Screenshot of the Intel Trace Analyzer and Collector_

To use this tool, you will need to install it on your own machine through the Intel student licence.
Documentation is [available online](https://software.intel.com/en-us/articles/intel-trace-analyzer-and-collector-documentation), and there is a [getting started guide](https://software.intel.com/en-us/get-started-with-itac).
There is also a [page explaning the GUI](https://software.intel.com/en-us/articles/introducing-intel-trace-analyzer-gui).

### Extrae and Paraver

This is a pair of tools that often go hand-in-hand developed at the [Barcelona Supercomputing Center](https://www.bsc.es/).
[Extrae](https://tools.bsc.es/extrae) handles the collection stage, and its output is generally visualised using [Paraver](https://tools.bsc.es/paraver).

Below is an example screenshot of Paraver from their website.
If you want to use these tools, you'll have to compile from source, but the code comes with [installation instructions](https://github.com/bsc-performance-tools/extrae/blob/master/INSTALL).

![Paraver](https://tools.bsc.es/sites/default/files/pictures/main-paraver-generalOverview_files/1333.gif) <br />
_Paraver screenshot. <https://tools.bsc.es/paraver>_


## Other notable tools

The tools in this section aren't available on BlueCrystal or as free download, so you will likely not be able to use them.
However, they are important in the wider HPC context, so we mention them for completeness.

### Cray tools

The Cray software stack—which runs exclusively on Cray machines—includes a range of tools collectively known as _perftools_.
One of those tools is the versatile CrayPAT profiler, which can collect data form any combination of CPU, GPU, memory, I/O, and various networking frameworks/languages.
Another is Reveal, a code analyser that helps with extracting parallelism from serial code by identifying dependencies and suggesting potential way to get around them.

Below is an example report produced by CrayPAT.
You can read the [documentation online](https://pubs.cray.com/content/S-2474/7.0.0/cray-performance-measurement-and-analysis-tools-installation-guide/use-craypat-craypat-lite-apprentice2-or-reveal).

```
CrayPat/X:  Version 6.5.2 Revision ba33e9b  08/22/17 20:38:22
Experiment:                  lite  lite/sample_profile
Number of PEs (MPI ranks):      1
Numbers of PEs per Node:        1
Numbers of Threads per PE:     36
Number of Cores per Socket:    18
Execution start time:  Wed Sep 26 16:49:16 2018
System name and speed:  pascal-002  1200 MHz (approx)
Intel Broadwell CPU  Family:  6  Model: 79  Stepping:  1


Avg Process Time:       0.57 secs
High Memory:           801.8 MBytes     801.8 MBytes per PE
MFLOPS (aggregate):    42.25 M/sec      42.25 M/sec per PE
I/O Write Rate:     1.834425 MBytes/sec

Table 1:  Profile by Function

  Samp% | Samp | Imb. |  Imb. | Group
        |      | Samp | Samp% |  Function=[MAX10]
        |      |      |       |   Thread=HIDE

 100.0% | 53.0 |   -- |    -- | Total
|-----------------------------------------------------------------------
|  50.9% | 27.0 |   -- |    -- | USER
||----------------------------------------------------------------------
||  15.1% |  8.0 |   -- |    -- | checkSTREAMresults
||  13.2% |  7.0 |  0.4 |  5.7% | main.REGION@li.334
||  13.2% |  7.0 |  0.6 |  9.0% | main.REGION@li.344
||   9.4% |  5.0 |  1.5 | 21.6% | main.REGION@li.324
||======================================================================
|  24.5% | 13.0 |   -- |    -- | OMP
||----------------------------------------------------------------------
||  13.2% |  7.0 |   -- |    -- | _cray$mt_execute_parallel_with_proc_bind
||  11.3% |  6.0 |   -- |    -- | _cray$mt_start_two_code_parallel
||======================================================================
|  17.0% |  9.0 |   -- |    -- | ETC
||----------------------------------------------------------------------
||  11.3% |  6.0 |   -- |    -- | pthread_create@@GLIBC_2.2.5
||   5.7% |  3.0 |   -- |    -- | fullscan_barrier_list]
||======================================================================
|   5.7% |  3.0 |   -- |    -- | PTHREAD
||----------------------------------------------------------------------
||   5.7% |  3.0 |   -- |    -- | pthread_join
||======================================================================
|   1.9% |  1.0 |   -- |    -- | RT
||----------------------------------------------------------------------
||   1.9% |  1.0 |   -- |    -- | nanosleep
|=======================================================================
```

### Arm tools

Arm provide the Arm Forge, a collection of tools previously known as the Allinea Forge.
They primarily target Arm platforms, but some of the tools also run on x86.
The two main tools are DDT, a parallel debugger that support OpenMP and MPI, and MAP, a graphical profiler that aims to be both easy-to-use and feature-rich.

![Arm DDT](https://static.docs.arm.com/101136/1822/images/DDTWithVersionControlInformation.png) <br />
_Screenshot of Arm DDT_

![Arm MAP](https://static.docs.arm.com/101136/1822/images/MapOpenMpSourceCodeView.png) <br />
_Screenshot of Arm MAP in line profiling mode_

You can find more details [on the Arm Developer website](https://developer.arm.com/products/software-development-tools/hpc/arm-forge).
