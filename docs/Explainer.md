# Design Motivation and Considerations

wasi-parallel was first discussed on August 12th, 2021 at the WASI CG meeting. Both slides and
meeting notes are available at
[wasi/2021/WASI-08-12.md](https://github.com/WebAssembly/meetings/blob/main/wasi/2021/WASI-08-12.md).
This document will summarize, but not exhaustively cover, the points there.

## Why wasi-parallel?
WebAssembly lacks support for parallel execution in general and this is a signicant performance lag
in several domains (ML, HPC). SIMD (128-bit or
[larger](https://github.com/WebAssembly/flexible-vectors/)) does not fully address the issue: many
programs benefit from parallel execution and standalone engines have no standard way to access this
system capability (unlike browser Web Workers).

## Why WASI?
Treating parallelism as a system capability via WASI allows parallel workloads to be offloaded to a
variety of devices from CPU to GPU and FPGA; this proposal can make use of the [threads
proposal](https://github.com/WebAssembly/threads/) atomic instructions once those are standardized.
The WebAssembly threads proposal stops short of defining the mechanism for thread creation under the
assumption that browsers will use Web Workers and standalone engines will develop a thread-creation
mechanism. The _parallel for_ described by this proposal is a mechanism that can leverage threading
capability in a CPU environment, but not a substitute for it--e.g., WASI could also standardize a
pthread-style API in the future.

## Design Goals
- if possible, this proposal will avoid modifications to the WebAssembly specification, instead
  adding the necessary types and functions as a part of its WASI API
- this proposal aims for execution on heterogeneous devices (e.g. CPU, GPU, FPGA?)
- many existing parallel frameworks provide a _parallel for_ construct (e.g. OpenMP, SYCL, TBB) and
  we aim to match the abstraction level of algorithms programmed under those models; while this
  proposal aims to explore the possibility of compiling programs written under those frameworks to
  wasi-parallel, there is no requirement to guarantee compilation of existing parallel programs
- this proposal cannot meet the requirements of all parallel programming models (e.g. pthreads) and
  is primarily focused on _parallel for_
