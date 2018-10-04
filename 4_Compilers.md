# Compilers

Compilers generate machine code from source program code.
If a compiler is able to optimise better for the target platform, then the application will run faster.

The compilers most commonly used in HPC on x86 are [the GNU Compiler](https://www.gnu.org/software/gcc/) and [the Intel Compiler](https://software.intel.com/en-us/intel-compilers/).
They are both available on BCp3, as well as BCp4.
There is no easy way to tell which compiler will be faster, as this can vary greatly depending on the code and the setting used to compile the application.
In general, you should try all the options and see which works better in your case.
If there are several versions of the same compiler, it makes sense to use the latest one, although there may be cases where an older version is faster, i.e. performance regressions.

The remainder of this section assumes you are working on BCp3.
If you are on Phase 4, some module names will likely be different.

## Compiler modules

On BCp3, the GCC modules are named `languages/gcc-<version>`:

```
$ module av languages/gcc

---------------------------------------------------------------------- /cm/shared/modulefiles ----------------------------------------------------------------------
languages/gcc-4.6.1 languages/gcc-4.7   languages/gcc-4.8.4 languages/gcc-4.9.1 languages/gcc-5.1   languages/gcc-6.1
languages/gcc-4.6.4 languages/gcc-4.8.1 languages/gcc-4.8.5 languages/gcc-5.0   languages/gcc-5.3   languages/gcc-7.1.0
```

Similarly, Intel Compiler modules are names `languages/intel-compiler-<version>`:

```
$ module av languages/intel-compiler

---------------------------------------------------------------------- /cm/shared/modulefiles ----------------------------------------------------------------------
languages/intel-compiler-13               languages/intel-compiler-15-impi          languages/intel-compiler-15-update-3
languages/intel-compiler-13-impi          languages/intel-compiler-15-impi-update-2 languages/intel-compiler-16
languages/intel-compiler-14               languages/intel-compiler-15-impi-update-3 languages/intel-compiler-16-u2
languages/intel-compiler-15               languages/intel-compiler-15-update-2
```

Load the module for the desired compiler and version before building your application.

**Note**: The `gcc` command is available in the system by default, i.e. without loading a module, because the Linux distribution comes packaged with a compiler.
However, note that this version is significantly older than what is available through modules, and so may generate slower code:

```
$ gcc -v
gcc version 4.4.7 20120313 (Red Hat 4.4.7-3) (GCC)
$ module load languages/gcc-7.1.0
$ gcc -v
gcc version 7.1.0 (GCC)
```

Also note that the alias `cc` refers to the system's default compiler.
This is _not a different compiler_, just a command alias:

```
$ ls -l $(which cc)
lrwxrwxrwx 1 root root 3 Apr  4  2013 /usr/bin/cc -> gcc
```

## General considerations

When building your application, it may be helpful to consider the following compiler options:

| Option                                         | Relevant GCC flags                             | Relevant Intel flags   |
| ---------------------------------------------- | ---------------------------------------------- | ---------------------- |
| Optimisation level                             | `-O<level>`                                    | `-O<level>`            |
| Target platform                                | `-march=<arch>`, `-mcpu=<cpu>`, `-mtune=<cpu>` | `-x<platform>`         |
| Fast (potentially unsafe!) maths optimisations | `-ffast-math`, `-funsafe-math-optimizations `  | `-fast`                |
| Vectorisation reports                          | `-fopt-info-vec-<level>` | `-qopt-report=<level> -qopt-report-phase=vec` |

The following sections discuss some GCC- and Intel-specific options.

## Compiling with GCC

A complete list of the GCC flags can be found [in the GCC online docs](https://gcc.gnu.org/onlinedocs/gcc/Option-Summary.html).

The default optimisation level with GCC is equivalent to `-O0`.
Be careful that this may not be the same across all compilers.

When specifying the target platform, you should be able to use `-march=native` to have the compiler detect what processor it's running on and optimise for it in particular.
However, there are cases when the detected processor type isn't accurate.
Be careful if the CPU you're compiling on is different than that one that will run your code.

If you attempt to make use of the hardware's SIMD features, you can use the vectorisation report to check if vectorised code has been generated.
The following flavours of `-fopt-info-vec-*` may be useful:
- `-fopt-info-vec-optimized` to show where vectorisation was used
- `-fopt-info-vec-missed` to show failed attempts to vectorise
- `-fopt-info-vec-all` to show all vectorisation-related messages

GCC's vectorisation reports are rather verbose and can be hard to parse.
Look for lines similar to the following that show reasons why vectorisation may have failed:

```
test.c:38:3: note: bad loop form.
test.c:39:5: note: reduction used in loop.
test.c:39:5: note: Unknown def-use cycle pattern.
```

However, not that these report contain merged reports from several optimisation passes, so it is possible to have _some_ passes fail but still obtain SIMD code in the end.

If you get no vectorisation report output, try using the legacy flag instead: `-ftree-vectorizer-verbose=<level>`, where level 5 is as good place to start.

## Compiling with Intel

Virtually all the options of the Intel compiler, including flags, pragmas and intrinsics, can be found [here](https://software.intel.com/en-us/cpp-compiler-18.0-developer-guide-and-reference).

You can use `-xHOST` to optimise for the platform you are compiling on.
As with GCC, consider that this may not always work as expected.
Also note that GNU and Intel have different names for the same platforms.

The default optimisation level with Intel is equivalent to `-O2`.
Be careful that this may not be the same across all compilers.

To generate vectorisation report, start with `-qopt-report=1 -qopt-report-phase=vec`.
If you need more verbose messages, increase to `-qopt-report=2`.
The output is stored in a `.optrpt` file that matches the name of your binary, with a separate section for each loop in your program.
If you have nested loops, expect to see the same nesting structure in the optimisation report.

Where vectorisation was successful, you should see this line:

```
remark #15300: LOOP WAS VECTORIZED
```

If vectorisation failed, you should see a message listing a cause that prevented optimisation:

```
remark #15344: loop was not vectorized: vector dependence prevents vectorization
```

Finally, Intel have a [tutorial page on using auto-vectorisation](https://software.intel.com/en-us/cpp-compiler-auto-vectorization-tutorial-tutorial-linux-and-macos-version) that you may find useful.
