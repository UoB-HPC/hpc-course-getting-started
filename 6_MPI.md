MPI
===

This section will deal with running MPI jobs on BlueCrystal Phase 3.
It will cover the `mpirun` launcher used to execute parallel jobs and how this interacts with the [queueing system](3_Queueing_Systems.md).
It is **not** an MPI programming tutorial, and it requires you to already be familiar with MPI terminology, e.g. what is a _rank_ and how it relates to threads, processes, and compute nodes.

[The last part of this section](#further-reference) lists some useful documentation links.

## MPI Implementations

When compiling MPI programs, you will need to choose an MPI implementation.
The most common choices are [Open MPI](https://www.open-mpi.org/), [MPICH](http://www.mpich.org/), and [Intel MPI](https://software.intel.com/en-us/mpi-library).
The first two are open-source libraries which you can install on your own machine, and all three are available on BCp3.
While feature-wise they should be largely equivalent, some options may differ in name and performance might vary.
You are encouraged to explore all the available options and discover any differences on your own.

On BCp3, the MPI libraries are available as modules:

- The latest version of Intel MPI is part of the same module as the compiler (`languages/intel-compiler-16-u2`), so you don't need to load an additional module.
    -  However, if you want a more advanced setup, older versions are available as separate modules:
    ```
    intel-mpi/64/4.0.3/008
    intel-mpi/64/4.1.0/024
    ```
- Open MPI built with the GNU and Intel compilers:
  ```
  openmpi/gcc/64/1.6.4
  openmpi/gcc/64/1.6.5
  openmpi/gcc/64/2.1.1
  openmpi/intel/64/1.6.5
  ```
- MPICH built with GCC:
  ```
  mpich/ge/gcc/64/1.2.7
  mpich/ge/open64/64/1.2.7
  mpich2/ge/gcc/64/1.4.1p1
  ```

If you require features that are only available in newer versions of Open MPI or MPICH, you can build from source.

## Compiling MPI programs

The choice of MPI library is (mostly) independent of the compiler choice.
Therefore, you should be able to use (for example) the GNU compiler with any of the MPI libraries listed above.
In practice, there are sometimes issues when you use a proprietary MPI implementation with compilers from other vendors.
Although unlikely, this means that you _may_ encounter issues when using, for example, Intel MPI with GCC.
However, this shouldn't deter you from exploring your options!

MPI implementations generally provide a _compiler wrapper_, which is a command that calls the underlying compiler with the parameters necessary for the MPI code.
The advantage of using this wrapper is that you _don't_ need to manually pass the compiler and linker flags for the library, and you can change to a different implementation without changing your build command.

For the open-source libraries, commands are usually named as follows:

| Language | Command  |
| -------- | -------- |
| C        | `mpicc`  |
| C++      | `mpicxx` |
| Fortran  | `mpif90` |

If you use Intel software, the commands above will use Intel _MPI_ with the GNU compilers.
If you want to use _both_ Intel MPI and Compiler, the commands are named by joining `mpi` with the normal Intel Compiler command:

| Language | Command    |
| -------- | ---------- |
| C        | `mpiicc`   |
| C++      | `mpiicpc`  |
| Fortran  | `mpiifort` |

If in doubt what the right command is called, first load the modules for your desired compiler and MPI library, then use your shell's autocomplete to list the available options:

```
$ mpi<TAB><TAB>
mpicc           mpiicc          mpiicpc         mpigcc
mpigxx          mpicxx          mpiexec         mpivars.sh
mpif77          mpif90          mpiifort
```

Then, run with `-v` to check what compiler and library will be used:

```
$ mpicc -v
mpigcc for the Intel(R) MPI Library 5.1.3 for Linux*
Copyright(C) 2003-2015, Intel Corporation.  All rights reserved.
...
gcc version 7.1.0 (GCC)

$ mpiicc -v
mpiicc for the Intel(R) MPI Library 5.1.3 for Linux*
Copyright(C) 2003-2015, Intel Corporation.  All rights reserved.
icc version 16.0.2 (gcc version 7.1.0 compatibility)
```

Once you know which wrapper to use, just replace the regular compiler command:

```bash
# Without MPI
$ gcc -o test-nompi test.c

# With MPI
$ mpicc -o test-mpi test.c
```

## Running MPI jobs

MPI applications generally run several processes, which need to be orchestrated, e.g. started, synchronised, and terminated.
This is done by using an MPI _launcher_, usually called `mpirun` or `mpiexec` (synonyms), which creates as many instances of your application as you instruct it and manages the processes over their lifetime.

The simplest MPI launch specifies only the number of ranks (`-np`) and the command to run:

```bash
$ mpirun -np 4 ./test-mpi
```

However, there are many more options, and you should read about them in `mpirun --help` and in the online documentation: [Open MPI](https://www.open-mpi.org/doc/v3.1/man1/mpirun.1.php), [MPICH](http://www.mpich.org/static/docs/latest/www/www1/mpiexec.html), [Intel MPI](https://software.intel.com/en-us/mpi-developer-reference-linux-mpirun).
Make sure that you look at right documentation for the version of the library you are using, and keep in mind that **some options may differ between implementations**.

**Important**: In order to run more than a single MPI rank, you _need to_ use a launcher.
If you don't—and just run your binary directly—only a single instance will run, so any MPI code will be redundant.

### Tagging output

One useful setting for debugging is tagging each line of output with the number of the rank that produced it.
Since the order in which print statements will be executed across several ranks is not fixed, this will help you identify which rank printed each line:

```
# Without tags
$ mpirun -np 4 ./hi
Hello from rank 2, on node31-031.
Hello from rank 0, on node31-031.
Hello from rank 3, on node31-031.
Hello from rank 1, on node31-031.

# With tags
$ mpirun -np 4 --tag-output ./hi
[1,2]<stdout>:Hello from rank 2, on node31-031.
[1,0]<stdout>:Hello from rank 0, on node31-031.
[1,3]<stdout>:Hello from rank 3, on node31-031.
[1,1]<stdout>:Hello from rank 1, on node31-031.
```

The `--tag-output` option works with Open MPI; with Intel MPI, use `-l`.
Other libraries likely have similar options, but they might have different names.

### Binding processes

By default, `mpirun` will allow you to launch one rank per CPU core available.
However, sometimes you may want to launch _fewer_ ranks than you have cores, e.g. because  each rank might run several threads internally, or _more_ ranks per core, particularly if your processor supports [simultaneous multithreading](https://en.wikipedia.org/wiki/Simultaneous_multithreading) and you want to run a rank per hardware thread.
The process of assigning processes (or threads) to hardware resources is commonly referred to as _binding_.

Most MPI implementations offer some support for binding ranks as part of the launcher.
For example, you can restrict each rank to run on a specific core (as opposed to any core, which is the default):

```bash
# Without binding
$ mpirun -np 4 ./hi
Hello from rank 2, on node31-031. (core affinity = 0-15)
Hello from rank 0, on node31-031. (core affinity = 0-15)
Hello from rank 3, on node31-031. (core affinity = 0-15)
Hello from rank 1, on node31-031. (core affinity = 0-15)

# With binding
$ mpirun -np 4 -bind-to-core ./hi
Hello from rank 0, on node31-031. (core affinity = 0)
Hello from rank 1, on node31-031. (core affinity = 1)
Hello from rank 2, on node31-031. (core affinity = 2)
Hello from rank 3, on node31-031. (core affinity = 3)
```

Another example is splitting the total number of processes between several nodes:

```bash
# Without mapping (all on first node)
$ mpirun -np 4 ./hi
Hello from rank 0, on compute091.
Hello from rank 1, on compute091.
Hello from rank 2, on compute091.
Hello from rank 3, on compute091.

# With mapping (split across 2 nodes)
$ mpirun -np 4 -npernode 2 ./hi
Hello from rank 0, on compute091.
Hello from rank 1, on compute091.
Hello from rank 2, on compute092.
Hello from rank 3, on compute092.
```

There are many more options available, and they are all explained in the manuals.
As above, the options may slightly differ with the implementation use.

## MPI examples

We have provided a [set of working MPI examples](https://github.com/UoB-HPC/hpc-course-examples/tree/master/mpi/examples), ranging from a simple ["Hello World" MPI program](https://github.com/UoB-HPC/hpc-course-examples/tree/master/mpi/examples/example1) to the kinds of ['halo exchange' message passing pattern](https://github.com/UoB-HPC/hpc-course-examples/tree/master/mpi/examples/example11) you need for the MPI assignment.

## Further reference

Here are some handy links to MPI docs:

- Open MPI
    - [v3.1](https://www.open-mpi.org/doc/v3.1/) (latest version)
    - [v2.1](https://www.open-mpi.org/doc/v2.1/) (on BCp3)
    - [v1.6](https://www.open-mpi.org/doc/v1.6/) (on BCp3)
- [Intel MPI guides](https://software.intel.com/en-us/mpi-developer-guide-linux)
- MPICH
    - [Installation guide](http://www.mpich.org/static/downloads/3.2.1/mpich-3.2.1-installguide.pdf)
    - [User guide](http://www.mpich.org/static/downloads/3.2.1/mpich-3.2.1-userguide.pdf)
    - [Latest manpages](http://www.mpich.org/static/docs/latest/www/)
    - [Older manpages](http://www.mpich.org/documentation/manpages/)

You can find some MPI programming tutorials [on the MPICH guides page](http://www.mpich.org/documentation/guides/).
The MPI standard spec is also [available online](https://www.mpi-forum.org/docs/).
