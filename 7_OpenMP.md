# OpenMP

This section discusses compiling and running OpenMP programs on BlueCrystal Phase 3.
As you will find, and unlike with [MPI](../6_MPI.md), there is almost no difference compared to when dealing with serial problems.
Still, there are a number of issues that you should be aware of and hints worth remembering.

## OpenMP implementations

OpenMP libraries are implemented by and shipped with all the compilers available on BCp3.
You _do not_ need to load any additional module or use a compiler wrapper.
However, be aware that each compiler vendor uses their own implementation of OpenMP, so some may perform better than others for a given code.

## Compiling OpenMP programs

You should use the latest version of you compiler of choice, so that you can take advantage of as many optimisations and as much functionality as possible.
There are:

- GCC 7.1:
```bash
module load languages/gcc-7.1.0
```
- Intel 16:
```bash
module load languages/intel-compiler-16-u2
```

When compiling, you need to use a specific flag to enable OpenMP, _in addition to optimisation flags_, even at high levels of optimisations.
If you don't, then your program will still compile, but the `pragma` directives will be ignored and no OpenMP will be used.
The flags you should use are:

| Compiler | Flag       |
| -------- | ---------- |
| GCC      | `-fopenmp` |
| Intel    | `-qopenmp` |

## Running OpenMP jobs

There is no special action you need to take to run an OpenMP program.
If it has been compiled appropriately, it will spawn threads as expected on its own.
However, you should always set the `OMP_NUM_THREADS` environment variable, which tell the OpenMP runtime how many threads to use for parallel regions.
If you don't set this, a system default value will be use, and on some systems this is 1, i.e. serial execution!

The most common ways to do this are:

- Export the variable in your job script on interactive session. This will make it available to all further commands.
```bash
export OMP_NUM_THREADS=16
./application
```
- Use `env` to only pass it to one command. This is useful if you only want to set it temporarily, e.g. for testing with different values.
```bash
env OMP_NUM_THREADS=16 ./application
```

### Thread placement and binding

In addition to setting the number of threads to be used, several environment variables can be used to influence the behaviour of the OpenMP runtime with regards _where_ the threads are run.
The most important ones are `OMP_PLACES` and `OMP_PROC_BIND`; they are generally used together.

One easy way to think about specifying thread affinity this ways is that `PLACES` states _where_, i.e. at what hardware level, to look, and then `PROC_BIND` says _how_ the affinity should be set here.
Two situations are commonly identified:

1. You want to have the threads as close together as possible, e.g. to minimise communication costs. This usually means placing threads on cores that are next to each-other, filling up a socket before moving to the next one.
```bash
export OMP_PLACES=cores OMP_PROC_BIND=close
```
2. You want to fill up your NUMA nodes equally, e.g. to spread memory access over the two sockets as much as possible.
```bash
export OMP_PLACES=cores OMP_PROC_BIND=spread
```

There is also [more advanced syntax](https://gcc.gnu.org/onlinedocs/libgomp/OMP_005fPLACES.html) for defining several levels of _places_, each with its own placement strategy.
If you go down this route, make sure you read the documentation carefully and check that you are actually binding the way you want to.

TACC have [a set of affinity examples](http://pages.tacc.utexas.edu/~eijkhout/pcse/html/omp-affinity.html) that shows more combinations.

If you start using these variables, it is useful to know that you can set `OMP_DISPLAY_ENV=true` to have the runtime prints its relevant variables before running your program.
This can be helpful if you think the same variable might be set several times and you want to confirm what value is being used.

## OpenMP examples

We have written a set of [OpenMP examples](https://github.com/UoB-HPC/hpc-course-examples/tree/master/openmp) relevant for this course.
These discuss basic implementation patterns, as well as pitfalls you need to be aware of when parallelising your code.

## Further reference

The [OpenMP specification](https://www.openmp.org/wp-content/uploads/openmp-4.5.pdf) is a great resource for clarifying any doubts you might have about how OpenMP should behave.
If the spec doesn't say it, then you should treat it as undefined behaviour.
It is well organised and very easy to read, so don't be intimidated!

Documentation is also provided by the compilers by the compilers available on BCp3 (which implements OpenMP):

- [GNU (libgomp)](https://gcc.gnu.org/onlinedocs/libgomp/index.html)
- [Intel (libiomp)](https://software.intel.com/en-us/cpp-compiler-developer-guide-and-reference-openmp-support)
