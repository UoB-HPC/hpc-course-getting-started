MPI
===

This section will deal with running MPI jobs on BlueCrystal.
It will cover the `mpirun` launcher used to execute parallel jobs and how this interacts with the [queueing system](3_Queueing_Systems.md).
It is **not** an MPI programming tutorial, and it requires you to already be familiar with MPI terminology, e.g. what is a _rank_ and how it relates to threads, processes, and compute nodes.

[The last part of this section](#further-reference) lists some useful documentation links.

## MPI Implementations

When compiling MPI programs, you will need to choose an MPI implementation.
The most common choices are [Open MPI](https://www.open-mpi.org/), [MPICH](http://www.mpich.org/), and [Intel MPI](https://software.intel.com/en-us/mpi-library).
The first two are open-source libraries which you can install on your own machine, and all three are available on BlueCrystal.
While feature-wise they should be largely equivalent, some options may differ in name and performance might vary.
You are encouraged to explore all the available options and discover any differences on your own.

### BCp4

On Phase 4, use the Intel MPI.
It is part of the compiler modules, e.g. `languages/intel/2020-u4`.

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
If you want to use _both_ Intel MPI and Compilers, the commands are named by joining `mpi` with the normal Intel Compiler command:

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
mpigcc for the Intel(R) MPI Library 2018 Update 3 for Linux*
Copyright(C) 2003-2018, Intel Corporation.  All rights reserved.
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
Target: x86_64-redhat-linux
Configured with: ../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
Thread model: posix
gcc version 4.8.5 20150623 (Red Hat 4.8.5-11) (GCC)
$ mpiicc -v
mpiicc for the Intel(R) MPI Library 2018 Update 3 for Linux*
Copyright(C) 2003-2018, Intel Corporation.  All rights reserved.
icc version 18.0.3 (gcc version 4.8.5 compatibility)
```

Once you know which wrapper to use, just replace the regular compiler command:

```bash
# Without MPI
$ icc -o test-nompi test.c

# With MPI
$ mpiicc -o test-mpi test.c
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

### SLURM and BCp4

SLURM provides its own parallel launcher, called `srun`.
The degree to which this integrates with the available hardware and software varies, but in general you can replace `mpirun` run with `srun` and expect everything to work fine.
Some SLURM systems, e.g. CS-series Crays, don't provide `mpirun` and _require_ you to use `srun`.

The advantage of using `srun` is that it automatically reads your run configuration from your job script, so you usually don't need to specify the number of ranks. You do however need to include the `--mpi=pmi2` argument to ensure we are using the correct version of MPI.
The following example script, which only uses `sbatch` arguments and passes just the binary to `srun`, runs 8 MPI processes evenly split between two nodes:

```bash
#SBATCH --nodes 2
#SBATCH --ntasks-per-node 4

srun --mpi=pmi2 ./test-mpi
```

On BCp4, you can use both `mpirun` and `srun`.
We recommend using `srun`, because your parallel configuration will be automatically read from your job script, so you won't have to repeat it in `mpirun` arguments.

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

We have provided a [set of working MPI examples](https://github.com/UoB-HPC/hpc-course-examples/tree/master/mpi), ranging from a simple ["Hello World" MPI program](https://github.com/UoB-HPC/hpc-course-examples/tree/master/mpi/example1) to an implementation of [the "halo exchange" message passing pattern](https://github.com/UoB-HPC/hpc-course-examples/tree/master/mpi/example5) you need for the MPI assignment.

## Further reference

Here are some handy links to MPI docs:

- Open MPI
    - [v3.1](https://www.open-mpi.org/doc/v3.1/) (a recent, supported version)
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
