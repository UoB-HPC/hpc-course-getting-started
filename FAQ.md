Frequently Asked Questions
==========================

This page contains a collection of answers to common questions.
It is being constantly updated.

## Logging into BlueCrystal

**Q**: I am having trouble logging into BlueCrystal, I've forgotten my password, etc. What should I do? <br />
**A**: Email <hpc-help@bristol.ac.uk> explaining your problem.

**Q**: I've think I've set up an SSH key, but I am still being asked for my password. What could have gone wrong? <br />
**A**: It's likely that you either 1) haven't placed your public key in `authorized_keys` on BlueCrystal, 2) you have not set `0644` permissions on `authorized_keys`, or 3) you haven't set your `.ssh/config` to use the right username and key file. Try following the manual steps again to make sure everything is there.

**Q**: I have not used Linux much before and I don't know many commands, or I don't know which terminal editor to use and how it works. What should I do? <br />
**A**: There are plenty of resources online explaining your options and how to use them. For CLI text editors, a few options are suggested in [the _Connecting_ section](1_Connecting_to_BlueCrystal#editing-files-remotely), and all of them have manpages, a large userbase, and plenty of documentation online.

## Compiling and running code

**Q**: Can I access GitHub from BlueCrystal, or do I need to pull my repository locally and upload it manually? <br />
**A**: You can access any online resource _from inside BC_; what you can't do is get _into BC_ from outside the University's network. Load the `tools/git/2.35.1` module and clone your repository directly on BlueCrystal.

**Q**: I have changed my Makefile, but typing `make` doesn't run the updated commands. What do I need to do? <br />
**A**: Make will only recompile if _source files_ have changed, not Makefiles themselves. You can ask it to rebuild anyway with a flag: `make -B`.

**Q**: I am encountering segmentation faults or bus errors, but they don't seem to happen all the time. Is it an issue with the compute nodes? <br />
**A**: These errors [generally mean you have attempted to access invalid memory locations](https://stackoverflow.com/questions/212466/what-is-a-bus-error), e.g. outside your program's memory space. If you have a parallel program, they may depend on the order of execution of some statements, which can be non-deterministic. Try using Valgrind to check for memory access issues and see if you can reproduce the problem with a single process/thread.

**Q**: When compiling with ICC, I get a `Catastrophic error: could not set locale "" to allow processing of multibyte characters` error. What does this mean? <br />
**A**: Read <https://software.intel.com/en-us/articles/cdiag912>. In particular, you need to `export LANG=C LC_ALL=C`; you will need to add this to your `.bashrc`, so that it's automatically done every time you log in.

**Q**: On Phase 4, why does my application crash under `mpirun`? <br />
**A**: While the reason could be down to a bug in _your application_, you should use `srun` as your parallel launcher, and not `mpirun`. Make sure to `export I_MPI_PMI_LIBRARY=/usr/lib64/libpmi.so` _before_ you run your application.



## The queueing system

**Q**: When I try to submit a job, there is no job output file and I get a `Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)` error in my UNIX mail. What went wrong? <br />
**A**: This normally happens when your `authorized_keys` file has been damaged. Try running the following command (note that there are _two_ angle brackets, _not one_!), then attempt your job again:
```
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
```

**Q**: When I try to submit a job using `qsub`, I get the following error message: `qsub:  file must be an ascii script`. What am I doing wrong? <br />
**A**: You are trying to pass an application binary directly to `qsub`, which it doesn't allow. You need to write a [job script](3_Queueing_Systems.md#job-files). There is a note about this in the [previous part](3_Queueing_Systems.md#submitting-jobs).

## Performance tools

**Q**: What does the vTune `Amplxe-cl Cannot enable Hardware Event-based Sampling: problem with the driver` error mean? <br />
**A**: You can run the Hotspot analysis in VTune, but the more advanced analyses need an additional driver installed. Due to some (legacy) configuration issues, this is hard to set up on BCp3. If you need advanced analyses, try running vTune on your own machine.

## Other system tools

**Q**: When I try to push/pull from a remote git repository on BCp4, I get an SSL library error. What am I doing wrong? <br />
**A**: There seem to be intermittent issues with git on Phase 4 if you have Intel modules loaded. Log out completely, log back in, then attempt your git operations again _without loading any Intel module_.

<!-- Template
**Q**: <br />
**A**:
-->

