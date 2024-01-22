Getting Started with the HPC Course
===================================


This is a collection of documents aiming to introduce the environment and typical workflows in HPC systems.
The tutorial addresses — and should be useful for — the University of Bristol High Performance Computing course.
Note, however, that we likely cover more material than is directly relevant for the assignments—just because a technique or issue is mentioned in this tutorial, it does _not_ mean that you necessarily need to tackle it in your submitted code or report.

## Structure

This repository aims to serve both as introductory material for the course labs and as reference to be used throughout the assignment.
When reading this material for the first time, we suggest that you follow the ordering presented in the [section below](#suggested-reading-order) and try the commands and code yourself.
However, each page aims to be self-contained and cover its topic without relying on a specific reading order.

### Suggested reading order

If you are unfamiliar with HPC systems, it's worth covering the basics of connecting to and using as supercomputer before moving on to programming tools.
We suggest you go through this tutorial as follows:

0. [Prerequisites](0_Prerequisites.md) – _Please read this first!_
1. [Connecting to BlueCrystal](1_Connecting_to_BlueCrystal.md)
2. [Software Modules and BlueCrystal](2_Modules.md)
3. [Queueing Systems](3_Queueing_Systems.md)
4. [HPC Compilers](4_Compilers.md)
5. [Performance Analysis Tools](5_Performance_Analysis_Tools.md)
6. [OpenMP](7_OpenMP.md)
7. [MPI](6_MPI.md)
8. [OpenCL<sup>†</sup>](8_OpenCL.md)

<!-- TODO: If we switch to BCp4, add SLURM instructions and update modules names -->

Note that this is not required reading _per se_.
The purpose of this tutorial is to help you get started with your assignment, but as long as you acquire the necessary skills to complete the unit, feel free to skip sections or use any alternative or additional material.

Items marked with † cover advanced topics and may exceed the scope of the Introduction course.

### FAQs

In addition to the reference pages, we are collecting answers to common questions in [a dedicated FAQ page](FAQ.md).
If you encounter a problem, please check this page before asking a question to see if an answer isn't already available.

## Additional Material

In the past, we have used various content that you may find useful throughout the unit. The list below is in no particular order.

<!-- - [Using BlueCrystal labsheet on BlackBoard](https://www.ole.bris.ac.uk/bbcswebdav/pid-3307009-dt-content-rid-9643695_2/courses/COMS30005_2018/Open%20Access%20for%20CS/labs/intro-handout.pdf) -->
- [OpenMP and MPI Examples on GitHub](https://github.com/UoB-HPC/hpc-course-examples)
- [OpenMP Tutorial Videos by Tim Mattson @ Intel](https://www.youtube.com/watch?v=nE-xN4Bf8XI&list=PLLX-Q6B8xqZ8n8bwjGdzBJ25X2utwnoEG)
- [GPU Programming Examples on GitHub](https://github.com/UoB-HPC/advanced-hpc-examples) (likely relevant for Advanced HPC only)

In addition, there are many books and links to online resources [on the unit's BlackBoard page](https://www.ole.bris.ac.uk/webapps/blackboard/content/listContentEditable.jsp?content_id=_4566993_1&course_id=_240828_1&mode=reset).

----

## Issues

Please report any problems or mistakes and suggest any potential improvements using [Issues](https://github.com/UoB-HPC/hpc-course-getting-started/issues).
