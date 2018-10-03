# Software Modules

On supercomputers, it is common practice to manage software using [environment modules](http://modules.sourceforge.net/).
Modules allow the admins to install a wide variety of software, potentially including several versions of the same package, while trying to avoid conflicts.

The basic working principle for modules is simple: instead of installing everything in the default system location, e.g. `/usr/local`, each software package is installed into its own directory.
Because these individual directories are not by default on the system's paths, it will not appear as if the packages are actually installed.
Instead, each package has an associated _module file_, which defines the paths to these individual directories.
When a user wants to use a particular package, they first need to _load_ the associated module, which adds the custom paths to the system's list.
From then on, everything will be virtually the same as having the package installed as usual.

## Common modules operations

Although the implementation of the modules package itself is different between BCp3 and BCp4, there should be very few user-facing differences.
The commands listed below apply to both phases—and virtually any other modules implementation you may come across.

If you try the examples in your terminal, keep in mind that some of the snippets below are from Phase 3 and some are from Phase 4.
Therefore, you may see and need to use slightly different module names.
The [worked example](#a-worked-example) is all done on Phase 3.

### Listing loaded modules

To see what modules you _have loaded_, use the `list` (shorthand `li`) command:

```bash
$ module list
Currently Loaded Modulefiles:
  1) shared                3) torque/4.2.4.1        5) default-environment
  2) dot                   4) moab/7.2.9
```

### Listing available modules

Use `available` (shorthand `avail` or `av`) to see _all defined modules_:

```bash
$ module available
--------------------------------- /cm/shared/modulefiles ---------------------------------
acml/gcc/64/5.1.0
acml/gcc/fma4/5.1.0
acml/gcc/mp/64/5.1.0
# Many more lines not shown...
```

You can also specify a pattern to `avail`, which will filter available modules (but see note below!):

```bash
$ module av gcc

--------------------------------- /cm/shared/modulefiles ---------------------------------
gcc/4.7.0
```

Note that **the pattern is only matched at the beginning of the module name**.
So, a module called `languages/gcc` will be filtered _out_ by the command above.
You can avoid this issue by filtering modules yourself using `grep`, but note that `module` _outputs to standard error_ (see [How do I capture the module command output?](https://modules.readthedocs.io/en/latest/FAQ.html#how-do-i-capture-the-module-command-output) for an explanation why):

```bash
$ module av |& grep -i gcc
acml/gcc/64/5.1.0
fftw3/gcc/64/3.3.3
gcc/4.7.0
languages/R-3.4.4-ATLAS-gcc-7.1.0
languages/gcc-7.1.0
openmpi/gcc/64/2.1.1
# Many lines omitted...
```

### Loading modules

To _load_ a module, use the `add` or `load` commands:

```bash
$ module li
No modules loaded
$ module load GCC/7.2.0-2.29
$ module li

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
$ module li

Currently Loaded Modules:
  1) GCCcore/7.2.0
  2) binutils/2.29-GCCcore-7.2.0
  3) GCC/7.2.0-2.29
$ module rm GCC/7.2.0-2.29
$ module li

Currently Loaded Modules:
  1) GCCcore/7.2.0
  2) binutils/2.29-GCCcore-7.2.0
```

You can also use `purge` to _unload **all** loaded modules_:

```bash
$ module li

Currently Loaded Modules:
  1) GCCcore/7.2.0
  2) binutils/2.29-GCCcore-7.2.0
$ module purge
$ module li
No modules loaded
```
Note that _`purge` will also remove modules loaded by default_, so you may find that certain functionality no longer works.
Only use this if you're confident you understand all the modules in your environment.
On BCp3, these modules are loaded by default, so you should manually load them after a `purge`:

```
shared dot torque/4.2.4.1 moab/7.2.9 default-environment
```

### Sessions and persisting modules

Any changes you make to your environment will only persist until you log out.
When you log back in, you will find that you only have the default modules loaded.

If you want to add more modules to be loaded automatically when you log in, one way to do this is to configure your shell to run your required `module` commands.
If you use bash, you can use your `.bashrc` file for this (more information on `.bashrc` in the [answer and comments here](https://unix.stackexchange.com/a/129144); other shells will have similar mechanisms, e.g. `.zshrc` or `config.fish`):

```bash
$ grep module .bashrc
module load tools/git
```

If you use this method, _you_ are responsible for any issues you create with different library versions.
When you encounter unexplained bugs and when you collect performance data, it is always worth running with a clean environment.
See the [note about how environment is preserved in compute jobs in Queueing Systems](3_Queueing_Systems.md#Environment-modules-and-queueing-systems).

## A worked example

Assume you want to use a more recent version of the GNU Compiler on Phase 3.
When you first log in, you only have the default modules loaded:

```bash
$ module li
Currently Loaded Modulefiles:
  1) shared                3) torque/4.2.4.1        5) default-environment
  2) dot                   4) moab/7.2.9
```

If you examine the `gcc` command, you will notice that it points to the (old) version packages with the operating system:

```bash
$ gcc -v
# Some lines omitted
gcc version 4.4.7 20120313 (Red Hat 4.4.7-3) (GCC)
$ which gcc
/usr/bin/gcc
```

You first need to find the module that you want:

```bash
$ module av |& grep -i gcc
# Many lines omitted
languages/gcc-5.3
languages/gcc-6.1
languages/gcc-7.1.0
```

Say you select version 7.1.0.
Now you need to load it:

```bash
$ module load languages/gcc-7.1.0
```

You can now check that you are using the desired version:

```bash
$ module li |& grep -i gcc
  6) languages/gcc-7.1.0
$ which gcc
/cm/shared/languages/GCC-7.1.0/bin/gcc
$ gcc -v
# Some lines omitted
gcc version 7.1.0 (GCC)
```

Loading the `languages/gcc-7.1.0` module has put the installation directory of the version 7.1.0 at the front of the system's path, so binaries and libraries are now searched here first, effectively _hiding_ the system version.
If you are interested, you can look at everything the module does with the `show` command (the [modulefile reference](http://modules.sourceforge.net/man/modulefile.html) may come in handy):

```bash
$ module show languages/gcc-7.1.0
-------------------------------------------------------------------
/cm/shared/modulefiles/languages/gcc-7.1.0:

module-whatis    adds GCC 7.1.0 to your environment variables
prepend-path     PATH /cm/shared/languages/GCC-7.1.0/bin
prepend-path     LD_LIBRARY_PATH /cm/shared/languages/GCC-7.1.0/lib64
prepend-path     LD_LIBRARY_PATH /cm/shared/languages/GCC-7.1.0/lib
prepend-path     MANPATH /cm/shared/languages/GCC-7.1.0/share/man
-------------------------------------------------------------------
```
