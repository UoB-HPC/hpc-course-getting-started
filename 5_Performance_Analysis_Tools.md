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
### PAPI
### valgrind
### TAU

### likwid

A newer collection of tools aiming to simplify performance analysis and benchmarking is [likwid](https://github.com/RRZE-HPC/likwid).
It can access hardware counters, run microbenchmarks to reveal some peak capabilities of your platform, and help with binding threads and MPI ranks.
One of its aims is to present results in a user-friendly way.

Here is a snippet of the output produced when using likwid to look at L2 cache counters on the [STREAM benchmark](https://www.cs.virginia.edu/stream/):

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
