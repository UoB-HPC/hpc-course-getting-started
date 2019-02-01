# OpenCL

This section discusses compiling and running OpenCL programs on BlueCrystal Phase 3.
As with [OpenMP](7_OpenMP.md), this is relatively straight-forward: you only need to load an OpenCL module and run on a node with your your desired hardware, e.g. GPUs.

Note that this is **not** an OpenCL programming tutorial.

## OpenCL implementations

OpenCL implementations are generally distributed by hardware vendors to target their own hardware.
For example, NVIDIA distribute their OpenCL library as part of the CUDA package, and so it can only be used on GPUs that support CUDA.
Conversely, Intel distribute their own implementation, which targets Intel CPUs and GPUs (although not all models are supported, particularly older ones).

One thing to be aware of is that there are several versions of the OpenCL standard.
Most vendors only support [OpenCL 1.2](https://www.khronos.org/registry/OpenCL/sdk/1.2/docs/man/xhtml/), and not [OpenCL 2.0](https://www.khronos.org/registry/OpenCL/sdk/2.0/docs/man/xhtml/) or newer, so make sure you read the appropriate documentation.
On BCp3, you will be using version 1.2.

If you want to install OpenCL on your own machine:

* On **Linux**, you should follow the instructions for your distribution and hardware. If you have a discrete GPU, the easiest way is through the [AMD](https://www.amd.com/en/support/kb/faq/amdgpu-installation)/[NVIDIA](https://developer.nvidia.com/cuda-downloads) drivers. If you only have integrated (Intel) graphics or you want to use OpenCL on the CPU, use the [Intel implementation](https://software.intel.com/en-us/intel-opencl) if your hardware supports it, or try [Beignet](https://www.freedesktop.org/wiki/Software/Beignet/) otherwise.
* On **macOS**, OpenCL should be installed and ready to be used by default. It _should_ work on both CPU and GPU, although results may vary depending on hardware.
* On **Windows**, this is just asking for trouble and you shouldn't do it...

## Compiling OpenCL programs

On BCp3, you can use OpenCL on the GPUs with the `cuda/toolkit/6.5` module.
After loading the module, all you need to do is link against the OpenCL library, e.g. by adding `-lOpenCL` to your linker flags.
You should use GCC to compile your host code, so your compilation line may look similar to:

```bash
gcc -std=c99 -O3 d2q9-bgk.c -lm -lOpenCL -o d2q9-bgk
```

If you want to try OpenCL on the CPU, use the `intel-opencl/1.2-3.0.67279` module.
However, you should be aware that people have had mixed success with this in the past.
You should also (already) know that programs optimised for GPUs don't necessarily perform well on CPUs, especially when running on older-generation hardware such as Sandy Bridge.

## Running OpenCL jobs

To run an OpenCL program, you don't need to do anything in addition to having an OpenCL device available.
When targeting a GPU on BCp3, this means requesting a node with a GPU through the queueing system.
This is done by adding the `gpus=<n>` constraint, where `n` is the number of GPUs you want to use and can be either `1` or `2` (since GPU nodes in BCp3 have two GPUs per node).
Your request may look similar to:

```bash
qsub -lnodes=1:ppn=16:gpus=1 ...
```

You can use all [other `qsub` parameters](3_Queueing_Systems.md#bcp3--pbs) as usual.

## Further reference

* The [OpenCL homepage](https://www.khronos.org/opencl/) has links to many documentation resources.
* Just as with OpenMP, [the OpenCL specification](https://www.khronos.org/registry/OpenCL/specs/opencl-1.2.pdf) is not hard to read and will answer a lot of questions. Note that some behaviour is still implementation-dependent.
* There are [online OpenCL reference pages](https://www.khronos.org/registry/OpenCL/sdk/1.2/docs/man/xhtml/) and a [cheatsheet-style reference card](https://www.khronos.org/registry/OpenCL/sdk/1.2/docs/OpenCL-1.2-refcard.pdf).
* NVIDIA have a [page of OpenCL docs and examples](https://developer.nvidia.com/opencl); so do [AMD](https://rocm-documentation.readthedocs.io/en/latest/Programming_Guides/Opencl-programming-guide.html).
* Finally, a (long) list of OpenCL material can be found [on the Khronos Group website](https://www.khronos.org/opencl/resources). This page has links to all the implementations and bindings for difference programming languages.
