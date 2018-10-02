Frequently Asked Questions
==========================

This page contains a collection of answers to common questions.
It is being constantly updated.

## Compiling and running code

**Q**: I am encountering segmentation faults or bus errors, but they don't seem to happen all the time. Is it an issue with the compute nodes? <br />
**A**: These errors [generally mean you have attempted to access invalid memory locations](https://stackoverflow.com/questions/212466/what-is-a-bus-error), e.g. outside your program's memory space. If you have a parallel program, they may depend on the order of execution of some statements, which can be non-deterministic. Try using Valgrind to check for memory access issues and see if you can reproduce the problem with a single process/thread.

**Q**: When compiling with ICC, I get a `could not set locale` error. What does this mean? <br />
**A**: Read <https://software.intel.com/en-us/articles/cdiag912>.

## The queueing system

**Q**: When I try to submit a job, there is no output and I get a `Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)` error. What went wrong? <br />
**A**: This normally happens when your `authorized_keys` file has been damaged. Try running the following command, then attempt your job again:
```
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
```

## Performance tools

**Q**: What does the vTune "Amplxe-cl Cannot enable Hardware Event-based Sampling: problem with the driver" error mean? <br />
**A**: You can run the Hotspot analysis in VTune, but the more advanced analyses need an additional driver installed. Due to some (legacy) configuration issues, this is hard to set up on BCp3. If you need advanced analyses, try running vTune on your own machine.

<!-- Template
**Q**: <br />
**A**:
-->
