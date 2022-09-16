# `wasi-parallel`

A proposed [WebAssembly System Interface](https://github.com/WebAssembly/WASI)
API for parallel computation.

### Current Phase

`wasi-parallel` is currently in [Phase 1].

[Phase 1]: https://github.com/WebAssembly/WASI/blob/main/Proposals.md#phase-1---feature-proposal-cg

### Champions

- [Andrew Brown](https://github.com/abrown)
- [Mingqiu Sun](https://github.com/mingqiusun)

### Phase 4 Advancement Criteria

`wasi-parallel` must have at least two complete independent implementations.

## Table of Contents

- [Introduction](#introduction)
- [Goals ](#goal)
- [Non-goals](#non-goals)
- [API walk-through](#api-walk-through)
- [Detailed design discussion](#detailed-design-discussion)
- [Considered alternatives](#considered-alternatives)
- [Stakeholder Interest & Feedback](#stakeholder-interest--feedback)
- [References & acknowledgements](#references--acknowledgements)



### Introduction

`wasi-parallel` addresses the need for parallel execution. By treating
parallelism as a system capability, this API allows parallel workloads to be
offloaded to a variety of devices, from CPUs to GPUs to FPGAs. The current
specification is a subset of the features provided by other parallel programming
frameworks (e.g., OpenMP, OpenCL).

WebAssembly lacks support for parallel execution in general and this can be a
significant performance lag in several domains (ML, HPC). SIMD (128-bit or
[larger]) does not fully address the issue: many programs benefit from parallel
execution and standalone WebAssembly engines have no standard way to access this system
capability (unlike browser Web Workers).

[larger]: https://github.com/WebAssembly/flexible-vectors

`wasi-parallel` was introduced in 2021 (see the [slides and meeting notes]),
prior to the `wasi-threads` proposal. If you are solely interested in spawning
CPU threads, `wasi-threads` is the right API (see the [considered
alternatives](#considered-alternatives) sections for more details).

[slides and meeting notes]: https://github.com/WebAssembly/meetings/blob/main/wasi/2021/WASI-08-12.md



### Goals

- __improve performance__: this API should make it possible to improve the
  performance of certain parallel applications, especially those designed around
  a "parallel for" construct.
- __any kind of parallel device__: this proposal aims for parallel execution on
  heterogeneous devices (e.g., CPU, GPU, FPGA?).



### Non-goals

- __modify core WebAssembly__: the current proposal does not propose changes to
  the WebAssembly instruction set.
- __replicate a parallel programming model__: many parallel programming
  frameworks already exist (e.g., OpenMP, OpenCL, `pthreads`); this API does not
  intend to match all features of any existing framework. Here, we explore the
  possibility of compiling programs written under those frameworks but do not
  guarantee compilation of existing parallel programs (i.e., due to "missing"
  `wasi-parallel` functionality).



### API walk-through

TODO




### Detailed design discussion

The design of `wasi-parallel` is still in an experimental phase. Suggestions are
welcome as an [issue]!

[issue]: https://github.com/WebAssembly/wasi-parallel/issues

#### Device selection

First, the user must be able to pick a parallel device to execute on. Early
feedback on the design (from the browser ecosystem) indicated that, since not
all hosts would support all kinds of parallel devices, the command to retrieve a
device should always succeed. This means that the kind of device the user
selects is only a hint and can be overriden by the host.

```wit
get-device: func(hint: device-kind) -> expected<device, error>
```

#### Buffer management

Since the parallel device could be something other than the CPU, there must be
some way to indicate what regions of WebAssembly memory will be used by the
device. The host is then responsible for transferring the memory to and from the
device. The host, however, is not required to copy the memory &mdash
implementations of a parallel CPU device could simply pass around pointers to
shared memory.

```wit
create-buffer: func(device: device, size: u32, kind: buffer-access-kind) -> expected<buffer, error>
```
```wit
write-buffer: func(data: list<u8>, buffer: buffer) -> expected<unit, error>
```
```wit
read-buffer: func(buffer: buffer) -> expected<list<u8>, error>
```

#### Kernel definition

There must be a way to indicate what code should be run in parallel. Several
other designs were discarded to reach the current mechanism: a binary-encoded
WebAssembly module that exports a `kernel` function and imports a shared
`memory`. When invoked by `parallel-for`, this kernel is instantiated by the
host and scheduled on the parallel device; the call returns once the parallel
execution is complete.

```wit
parallel-for: func(device: device, kernel: list<u8>, num-iterations: u32, block-size: u32, buffers: list<buffer>) -> expected<unit, error>
```



### Considered alternatives

Other approaches are possible:

#### `wasi-threads`

The [`wasi-threads`] proposal aims to expose host thread creation to a
WebAssembly program. With some caveats, `wasi-threads` can be used to implement
`pthreads`. The primary use case is exposing CPU-executed, OS-managed
threads; users who simply need threads should look there first.

Because `wasi-parallel` aims to allow parallel execution on more than just CPUs,
the API is quite different. Memory may be synchronized between devices, so it
includes buffer management APIs. And the number of iterations to execute must be
known up front for the device driver (e.g., GPU) to schedule the iterations
optimally.

[`wasi-threads`]: https://github.com/WebAssembly/wasi-threads

#### Host APIs

This API standardizes a subset of the functionality available for parallel
execution on a host system. Users of WebAssembly programs could instead use this
parallelism natively on the host side and expose custom APIs for their workload
&mdash; in other words, skip standardization. This is a legitimate approach that
users should consider, especially if the host hardware is known and fixed.
`wasi-parallel` is targeted at situations in which parallelism is needed but the
exact host environment is unknown.



### Stakeholder Interest & Feedback

TODO before entering Phase 3.




### References & acknowledgements

Many thanks for valuable feedback and advice from:

- [Enrico Galli](https://github.com/egalli)
- [Petr Penzin](https://github.com/penzn)
