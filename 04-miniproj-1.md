---
layout: page
title: Mini-project 1
---

**Due:** April 28th, EOD  
**Group size:** 1–4 (recommended: 2)

---

## Summary

The goal of this assignment is to gain hands-on experience with GPU programming and
to understand the compute and memory characteristics of two important deep learning
kernels: convolution and attention.

You will implement CUDA kernels for both, profile their performance, and analyze
where they fall on the roofline model.

---

## Getting Started

You should have received an email with account access to the course GPU machine.
Make sure `/usr/local/cuda/bin` is in your `PATH`. You are also welcome to use any
other GPU you have access to — a personal machine, Google Colab, or a cloud instance
all work fine.

For an introduction to CUDA, this is a good starting point:  
https://developer.nvidia.com/blog/even-easier-introduction-cuda/

The official programming guide covers GPU features in depth:  
https://docs.nvidia.com/cuda/cuda-c-programming-guide/

To query your GPU's parameters (peak FLOPS, memory bandwidth, SM count, etc.),
build and run the NVIDIA CUDA samples:

```bash
git clone https://github.com/nvidia/cuda-samples
cd cuda-samples
git checkout cd3bc1fa8e949ca016c6396c47124fdcfd75fb4b
make -j24
./Samples/1_Utilities/deviceQuery/deviceQuery
```

---

## Task

Implement CUDA kernels for the following two workloads. For each, start with a
naive implementation that gets correct results, then iteratively apply optimizations
and measure the impact of each. Example optimizations to consider:

- **Tiling / shared memory**: load a tile of inputs or weights into shared memory
  so threads in a block can reuse them without repeated DRAM accesses
- **Memory coalescing**: restructure the data layout or access pattern so that
  adjacent threads read adjacent memory addresses
- **Warp-level primitives**: use `__shfl_sync` for reductions within a warp without
  going through shared memory
- **Increased occupancy**: tune block size and register usage to keep more warps
  resident on each SM

You don't need to try all of these — pick what seems promising for your kernel and
explain what you tried and why. You are not graded on how fast your kernel is, but
squeezing out higher utilization is its own reward.

### Part 1 — Convolution

Implement a CUDA kernel for direct 2D convolution with the following VGG configurations:

| Layer | Ny × Nx | Ky × Kx | Ni  | Nn  | Stride |
|-------|---------|---------|-----|-----|--------|
| Conv1 | 224×224 | 3×3     | 64  | 64  | 1      |
| Conv2 | 14×14   | 3×3     | 512 | 512 | 1      |

Pick either batch size 1 or batch size 16 and use it consistently. Batch 16 tends to
be easier to get high utilization from (better weight reuse), but batch 1 is simpler
and can be more representative of inference scenarios.

The data layout used in the reference implementation is:
- Weights: `[Ky][Kx][Nn][Ni]`
- Input:   `[B][NYPAD][NXPAD][Ni]` (zero-padded)
- Output:  `[B][NYSCL][NXSCL][Nn]`

### Part 2 — Attention

Implement CUDA kernels for single-head attention with head dimension D=64. This
represents one head of a typical multi-head attention layer. Implement both the
**prefill** case (S queries attending causally to S keys) and the **decode** case
(a single new query attending to a KV cache of size C):

| Case    | Sizes           |
|---------|-----------------|
| Prefill | S = 4096, 65536 |
| Decode  | C = 4096, 65536 |

The reference implementation provides `standard_prefill`, `flash_prefill`, and
`standard_decode`. You may implement either standard or flash attention for your GPU
kernel. Note that for the larger context size (S=65536), standard attention may not
fit in GPU memory — the provided flash attention implementation may serve as a useful
reference in that case.

For both kernels, you are free to change the data layout, use any CUDA features
(shared memory, tensor cores, warp-level intrinsics), or restructure the computation.

### Reference Implementation

A CPU reference implementation for both kernels is provided at
[github.com/PolyArch/cs259-miniproj-ref](https://github.com/PolyArch/cs259-miniproj-ref).
Clone it, build, and run to see expected outputs:

```bash
git clone git@github.com:PolyArch/cs259-miniproj-ref.git
cd cs259-miniproj-ref
make
./conv
./attention
```

---

## Profiling with Nsight Compute (ncu)

`ncu` is NVIDIA's GPU kernel profiler (the modern replacement for the deprecated
`nvprof`). It collects hardware performance counters for a single kernel launch and
produces detailed breakdowns of compute throughput, memory throughput, occupancy,
and bottlenecks.

### Basic usage

```bash
# Profile all kernels in your binary
ncu ./my_program

# Profile only a specific kernel by name
ncu --kernel-name my_kernel_name ./my_program
```

### Useful metric sets

```bash
# Full default report (good starting point -- covers compute, memory, occupancy)
# Also generates a roofline chart when opened in ncu-ui
ncu --set full ./my_program
```

For more targeted queries, you can pass individual metrics with `--metrics`.
Some useful ones:

| Metric | What it measures |
|--------|-----------------|
| `dram__bytes_read.sum` | Total bytes read from DRAM (HBM) for the kernel |
| `dram__bytes_write.sum` | Total bytes written to DRAM (HBM) for the kernel |
| `l1tex__t_bytes_pipe_lsu_mem_global_op_ld.sum` | Bytes requested from L1 for global loads (before the cache) |
| `lts__t_bytes_equiv_l1sectormiss_pipe_lsu_mem_global_op_ld.sum` | Bytes that missed L1 and went to L2 |
| `sm__throughput.avg.pct_of_peak_sustained_elapsed` | Overall SM utilization as % of peak |
| `gpu__dram_throughput.avg.pct_of_peak_sustained_elapsed` | DRAM bandwidth utilization as % of peak |
| `sm__warps_active.avg.pct_of_peak_sustained_active` | Occupancy: active warps as % of the SM maximum |
| `smsp__sass_thread_inst_executed_op_ffma_pred_on.sum` | FP32 fused multiply-add instructions executed (1 FMA = 2 FLOPs) |

Example:
```bash
ncu --metrics dram__bytes_read.sum,dram__bytes_write.sum,smsp__sass_thread_inst_executed_op_ffma_pred_on.sum \
  ./my_program
```

### Saving and viewing reports

```bash
# Save a report to a file
ncu --set full -o my_report ./my_program

# Copy the report to your local machine and open in the Nsight Compute GUI
# Download: https://developer.nvidia.com/tools-overview/nsight-compute/get-started
ncu-ui my_report.ncu-rep
```

A full list of available metrics and what they measure is in the
[Nsight Compute Kernel Profiling Guide](https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html#metrics-reference).

### Roofline analysis

The roofline model plots achieved performance (GFLOPS) against arithmetic intensity
(FLOPs/byte), with a diagonal memory-bandwidth ceiling and a horizontal compute
ceiling. A kernel below the diagonal is memory-bound; above it, compute-bound.
For a primer, see [this NVIDIA blog post](https://developer.nvidia.com/blog/roofline-and-empirical-roofline-toolkit/)
or the [Nsight Compute roofline documentation](https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html#roofline)
(note: `ncu --set full` generates a roofline chart automatically when opened in ncu-ui).

The two numbers most useful for placing your kernel on the roofline are:

- **Achieved memory bandwidth**: from `dram__bytes_read.sum` and
  `dram__bytes_write.sum` divided by kernel duration
- **Achieved compute throughput**: from `smsp__sass_thread_inst_executed_op_ffma_pred_on.sum`
  (counts FP32 FMAs, each counting as 2 FLOPs) divided by kernel duration

The ratio of FLOPs to DRAM bytes gives you the arithmetic intensity. Compare it to
the ridge point of your GPU's roofline (peak FLOPS / peak memory bandwidth) to
determine which resource is the bottleneck.

---

## Questions

Present data for all 6 workloads (Conv1, Conv2, prefill S=4096, prefill S=65536,
decode C=4096, decode C=65536). Answer the questions for each kernel type you
implemented — discussing how behavior differs across the configurations within it.

### 1. Parallelization Strategy

What is your parallelization strategy? What problem dimensions did you map to
blocks and threads? Are there limitations to this approach — for example, does it
scale well with batch size, context length, or channel count?

### 2. Algorithmic FLOPs

Compute the algorithmic FLOP count for each configuration. Show your derivation.

For convolution, count multiply-add operations over the kernel and channel dimensions.
For attention, account for the QK dot products and the weighted sum over values —
note that prefill involves a triangular (causal) access pattern.

Report your results in GFLOPs.

### 3. Execution Time

What is the measured execution time for each configuration? What is the achieved
GFLOPS (using your algorithmic FLOP count from Q2)?

### 4. Roofline Analysis

Plot your results on the roofline model for your GPU. For each kernel configuration:

- Compute the theoretical arithmetic intensity (algorithmic FLOPs / minimum bytes
  required to read inputs and write outputs once)
- Measure the actual DRAM traffic using `ncu` (`dram__bytes_read.sum`,
  `dram__bytes_write.sum`) and compute the achieved arithmetic intensity from that
- Place both on the roofline and identify whether the kernel is compute-bound or
  memory-bandwidth-bound

How do the theoretical and measured arithmetic intensities compare? What does the
difference tell you, and what does your roofline placement say about the potential
for further optimization?

### 5. Optimizations

What optimizations did you try? Which had the most impact, and which had little or
no effect? If an optimization did not help, explain why.

---

## What to Turn In

- A PDF report answering the questions above, including roofline plots
- Your CUDA source file(s) as a separate attachment (do not zip with the PDF)

Turn in via Canvas using your group submission.
