# Software Modules

On supercomputers, it is common practice to manage software using [environment modules](http://modules.sourceforge.net/).
Modules allow the admins to install a wide variety of software, potentially including several versions of the same package, while trying to avoid conflicts.

The basic working principle for modules is simple: instead of installing everything in the default system location, e.g. `/usr/local`, each software package is installed into its own directory.
Because these individual directories are not by default on the system's paths, it will not appear as if the packages are actually installed.
Instead, each package has an associated _module file_, which defines the paths to these individual directories.
When a user wants to use a particular package, they first need to _load_ the associated module, which adds the custom paths to the system's list.
From then on, everything will be virtually the same as having the package installed in a standard location.

## Common modules operations

Although the implementation of the modules package itself is different between BCp3 and BCp4, there should be very few user-facing differences.
The commands listed below apply to both phases—and virtually any other modules implementation you may come across.

If you try the examples in your terminal, keep in mind that the snippets below are from Phase 4.
Therefore, you may see and need to use slightly different module names on other systems.

### Listing loaded modules

To see what modules you _have loaded_, use the `list` (shorthand `li`) command:

```bash
$ module list

Currently Loaded Modules:
  1) tools/git/2.18.0

```

### Listing available modules

Use `available` (shorthand `avail` or `av`) to see _all defined modules_:

```bash
$ module available

--------------------------------- /mnt/storage/easybuild/modules/local ---------------------------------
apps/grace/5.1.25                              languages/anaconda3/2019.07-3.7.3-biopython             (D)
apps/gromacs/5.1.4-plumed-mpi-intel            languages/gcc/9.1.0
apps/gromacs/5.1.4-mpi-intel                   languages/go/1.10
apps/gromacs/2018-mpi-gpu-intel                languages/intel/2016-u3-cuda-8.0
apps/gromacs/2019.3-mpi-intel           (D)    languages/intel/2017.01
# Many more lines not shown...
```

You can also specify a pattern to `avail`, which will filter available modules (but see the note below!):

```bash
$ module av gcc

-------------------------------------------------------- /mnt/storage/easybuild/modules/local --------------------------------------------------------
languages/gcc/9.1.0    libs/cuda/9.0-gcc-5.4.0-2.26    libs/cuda/10.0-gcc-5.4.0-2.26 (D)    libs/gsl/2.5-gcc-5.5.0
```

Note that on most systems **the pattern is only matched at the beginning of the module name**.
So, a module called `languages/gcc` will be filtered _out_ by the command above.
You can avoid this issue by filtering modules yourself using `grep`, but note that `module` _outputs to standard error_ (see [How do I capture the module command output?](https://modules.readthedocs.io/en/latest/FAQ.html#how-do-i-capture-the-module-command-output) for an explanation why):

```bash
$ module av |& grep -i gcc
languages/gcc/9.1.0
libs/cuda/9.0-gcc-5.4.0-2.26
libs/cuda/10.0-gcc-5.4.0-2.26                           (D)
libs/gsl/2.5-gcc-5.5.0
ATLAS/3.10.2-GCC-5.4.0-2.26-LAPACK-3.6.1acml/gcc/64/5.1.0
fftw3/gcc/64/3.3.3
gcc/4.7.0
languages/R-3.4.4-ATLAS-gcc-7.1.0
languages/gcc-7.1.0
openmpi/gcc/64/2.1.1
# Many lines omitted...
```

On BCp4 in particular, the modules are split into sections in such a way that modules intended for end-users are displayed at the top, under the `/mnt/storage/easybuild/modules/local` heading.
You will not need to use modules outside this section for your coursework, so please **do not** use those unless you know _exactly_ what you are doing.

### Loading modules

To _load_ a module, use the `add` or `load` commands:

```bash
$ module list
No modules loaded
$ module load GCC/7.2.0-2.29
$ module list

Currently Loaded Modules:
  1) GCCcore/7.2.0
  2) binutils/2.29-GCCcore-7.2.0
  3) GCC/7.2.0-2.29
```

As shown above, loading a module may automatically load other modules marked as dependencies.

It is common practice for the version of a package be to listed as the last component of the module name.
If you don't give a specific version, then the default one will be loaded.
The default version is marked when you list modules with `avail`:

```bash
$ module av GCC
--------------------------- /mnt/storage/easybuild/modules/all ---------------------------
   GCC/4.9.3-2.25
   GCC/5.4.0-2.26
   GCC/6.4.0-2.28
   GCC/7.2.0-2.29 (D)

  Where:
   D:  Default Module
```

### Unloading modules

Use `rm` or `unload` to remove a previously loaded module:

```bash
$ module list

Currently Loaded Modules:
  1) GCCcore/7.2.0
  2) binutils/2.29-GCCcore-7.2.0
  3) GCC/7.2.0-2.29
$ module rm GCC/7.2.0-2.29
$ module list

Currently Loaded Modules:
  1) GCCcore/7.2.0
  2) binutils/2.29-GCCcore-7.2.0
```

**Warning**: The `languages/intel` modules on BCp4 are _not_ set up to unload properly.
If you try to remove them, it will appear as if everything worked fine—they will disappear from the list of loaded modules—but _the environment will not be cleaned as it should_.
If you have loaded one of those modules and wish to remove it, you will need to log out of the system and log back in ([which gives you the default environment](#sessions-and-persisting-modules), before loading any modules).

You can use `purge` to _unload **all** loaded modules_:

```bash
$ module list

Currently Loaded Modules:
  1) GCCcore/7.2.0
  2) binutils/2.29-GCCcore-7.2.0
$ module purge
$ module list
No modules loaded
```

Note that _`purge` will also remove modules loaded by default_, so you may find that certain functionality no longer works.
Only use this if you're confident you understand all the modules in your environment, and keep in mind you can always log off and back on to ensure you are using the default environment.

### Sessions and persisting modules

Any changes you make to your environment will only persist until you log out.
When you log back in, you will find that you only have the default modules loaded.

If you want to add more modules to be loaded automatically when you log in, one way to do this is to configure your shell to run your required `module` commands.
If you use bash, you can use your `.bashrc` file for this (more information on `.bashrc` in the [answer and comments here](https://unix.stackexchange.com/a/129144); other shells will have similar mechanisms, e.g. `.zshrc` or `config.fish`):

```bash
$ grep module .bashrc
module load tools/git-2.18.0
```

We recommend you do _not_ add module commands to your shell's start-up script, because it may prevent you from getting a "clean" environment when you expect one.
If you still decide to do it, _you_ are responsible for any issues you create, e.g. with conflicting library versions.
When you encounter unexplained bugs and when you collect performance data, it is always worth running with a clean environment.
See the [note about how environment is preserved in compute jobs in Queueing Systems](3_Queueing_Systems.md#Environment-modules-and-queueing-systems).

## A worked example

Assume you want to use a more recent version of the GNU Compiler on Phase 4.
When you first log in, you only have the default modules loaded:

```bash
$ module list
No modules loaded

```

If you examine the `gcc` command, you will notice that it points to the (old) version packaged with the operating system:

```bash
$ gcc -v
# Some lines omitted
gcc version 4.8.5 20150623 (Red Hat 4.8.5-11) (GCC)
$ which gcc
/usr/bin/gcc
```

You first need to find the module that you want:

```bash
$ module av |& grep -i gcc
-------------------------------------------------------- /mnt/storage/easybuild/modules/local --------------------------------------------------------
languages/gcc/7.2.0 languages/gcc/8.2.0  languages/gcc/9.1.0
# Many lines omitted
```

Say you select version 9.1.0.
Now you need to load it:

```bash
$ module load languages/gcc/9.1.0
```

You can now check that you are using the desired version:

```bash
$ module list

Currently Loaded Modules:
  1) languages/gcc/9.1.0

$ which gcc
/mnt/storage/software/languages/gcc-9.1/bin/gcc
$ gcc -v
# Some lines omitted
gcc version 9.1.0 (GCC)
```

Loading the `languages/gcc/9.1.0` module has put the installation directory of the version 9.1.0 at the front of the system's path, so binaries and libraries are now searched here first, effectively _hiding_ the system version.
If you are interested, you can look at everything the module does with the `show` command (the [modulefile reference](http://modules.sourceforge.net/man/modulefile.html) may come in handy):

```
$ module show languages/gcc-7.5.0
-------------------------------------------------------------------
help([[
Description
===========
The GNU Compiler Collection includes front ends for C, C++, Objective-C, Fortran, Ada, Go, and D, as well as libraries for these languages.
More information
================
- Homepage: https://gcc.gnu.org
]])
whatis("The GNU Compiler Collection includes front ends for C, C++, Objective-C, Fortran, Ada, Go, and D, as well as libraries for these languages")
whatis("Homepage: https://gcc.gnu.org")
prepend_path("PATH","/mnt/storage/software/languages/gcc-9.1/bin")
prepend_path("LD_LIBRARY_PATH","/mnt/storage/software/languages/gcc-9.1/lib64")
prepend_path("LD_LIBRARY_PATH","/mnt/storage/software/languages/gcc-9.1/lib")
prepend_path("MANPATH","/mnt/storage/software/languages/gcc-9.1/share/man")
-------------------------------------------------------------------
```
