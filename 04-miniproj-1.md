# CS 259 Mini-Project 1: GPU Kernel Optimization

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
Make sure `/usr/local/cuda/bin` is in your `PATH`.

A CPU reference implementation for both kernels is provided in a separate repository.
Clone it, build, and run to see expected outputs and get familiar with the data layouts:

```bash
git clone git@github.com:PolyArch/cs259-miniproj-ref.git
cd cs259-miniproj-ref
make
./conv
./attention
```

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

Implement CUDA kernels for the following two workloads.

### Part 1 — Convolution

Implement a CUDA kernel for direct 2D convolution with the following VGG configurations:

| Layer | Ny × Nx | Ky × Kx | Ni  | Nn  | Stride | Batch |
|-------|---------|---------|-----|-----|--------|-------|
| Conv1 | 224×224 | 3×3     | 64  | 64  | 1      | 1, 16 |
| Conv2 | 14×14   | 3×3     | 512 | 512 | 1      | 1, 16 |

The data layout used in the reference implementation is:
- Weights: `[Ky][Kx][Nn][Ni]`
- Input:   `[B][NYPAD][NXPAD][Ni]` (zero-padded)
- Output:  `[B][NYSCL][NXSCL][Nn]`

You are free to change the data layout, use any CUDA features (shared memory, tensor
cores, warp-level intrinsics), or restructure the computation.

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

# Limit to the first N launches of a kernel (useful if you run many iterations)
ncu --launch-count 1 --kernel-name my_kernel_name ./my_program
```

### Useful metric sets

```bash
# Full default report (good starting point -- covers compute, memory, occupancy)
ncu --set full ./my_program

# Memory throughput: how hard you are hitting DRAM vs L2 vs L1
ncu --metrics \
  l1tex__t_bytes_pipe_lsu_mem_global_op_ld.sum,\
  l1tex__t_bytes_pipe_lsu_mem_global_op_st.sum,\
  lts__t_bytes_equiv_l1sectormiss_pipe_lsu_mem_global_op_ld.sum,\
  dram__bytes_read.sum,\
  dram__bytes_write.sum \
  ./my_program

# Achieved memory bandwidth (GB/s) and compute throughput (TFLOPS)
ncu --metrics \
  sm__throughput.avg.pct_of_peak_sustained_elapsed,\
  gpu__dram_throughput.avg.pct_of_peak_sustained_elapsed \
  ./my_program

# Occupancy: how many warps are active vs the maximum
ncu --metrics \
  sm__warps_active.avg.pct_of_peak_sustained_active \
  ./my_program
```

### Saving and viewing reports

```bash
# Save a report to a file
ncu --set full -o my_report ./my_program

# Open in Nsight Compute GUI (if available)
ncu-ui my_report.ncu-rep
```

### Interpreting the output for roofline analysis

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

Answer the following questions **for each kernel** (Conv1, Conv2, prefill, decode).
Where configurations differ (e.g., batch size or context length), report results for
all and discuss any differences.

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

- Estimate the arithmetic intensity (FLOPs / bytes transferred to/from DRAM)
- Identify whether the kernel is compute-bound or memory-bandwidth-bound
- Mark where it falls on the roofline

Use `ncu` to measure actual DRAM traffic rather than estimating from the algorithm
alone — real hardware behavior (caching, padding, reuse) can differ significantly
from the theoretical minimum. What does your roofline placement tell you about the
potential for further optimization?

### 5. Optimizations

What optimizations did you try? Which had the most impact, and which had little or
no effect? If an optimization did not help, explain why.

---

## What to Turn In

- A PDF report answering the questions above, including roofline plots
- Your CUDA source file(s) as a separate attachment (do not zip with the PDF)

Turn in via Canvas using your group submission.
