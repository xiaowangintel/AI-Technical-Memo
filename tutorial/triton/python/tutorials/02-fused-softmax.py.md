# 02-fused-softmax.py — Code Analysis / 代码分析

## Source / 来源

- **Source file / 源文件:** `/root/xw/triton/python/tutorials/02-fused-softmax.py`

- **EN:** Shows how to fuse row-wise softmax into a single Triton kernel that keeps one row in on-chip memory and reduces DRAM traffic.

- **CN:** 展示如何把按行 softmax 融合成单个 Triton 内核，使一整行数据常驻片上存储，从而减少 DRAM 访问。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15 — Tutorial overview / 教程概述

```python
"""
Fused Softmax
=============

In this tutorial, you will write a fused softmax operation that is significantly faster
than PyTorch's native op for a particular class of matrices: those whose rows can fit in
the GPU's SRAM.

In doing so, you will learn about:

* The benefits of kernel fusion for bandwidth-bound operations.

* Reduction operators in Triton.

"""
```

**EN:** The docstring frames softmax as a bandwidth-bound operation and previews two central ideas: kernel fusion and reductions in Triton.

**CN:** 文档字符串把 softmax 定位为典型的带宽受限算子，并提前点出两个核心主题：内核融合以及 Triton 中的归约。

### Lines 24-40 — Imports and backend helpers / 导入与后端辅助函数

```python
import torch

import triton
import triton.language as tl
from triton.runtime import driver

DEVICE = triton.runtime.driver.active.get_active_torch_device()


def is_hip():
    return triton.runtime.driver.active.get_current_target().backend == "hip"


def is_cdna():
    return is_hip() and triton.runtime.driver.active.get_current_target().arch in ('gfx940', 'gfx941', 'gfx942',
                                                                                   'gfx90a', 'gfx908')
```

**EN:** Besides importing `torch`, `triton`, and `tl`, the file defines `is_hip()` and `is_cdna()` so later occupancy calculations can adapt to CUDA vs. AMD backends. This is important because register accounting differs across GPU architectures.

**CN:** 除了导入 `torch`、`triton` 和 `tl` 外，文件还定义了 `is_hip()` 与 `is_cdna()`，方便后续占用率计算适配 CUDA 与 AMD 后端。这样做很重要，因为不同 GPU 架构的寄存器统计方式并不相同。

### Lines 42-59 — Naive PyTorch softmax / 朴素的 PyTorch softmax

```python
def naive_softmax(x):
    """Compute row-wise softmax of X using native pytorch

    We subtract the maximum element in order to avoid overflows. Softmax is invariant to
    this shift.
    """
    # read  MN elements ; write M  elements
    x_max = x.max(dim=1)[0]
    # read MN + M elements ; write MN elements
    z = x - x_max[:, None]
    # read  MN elements ; write MN elements
    numerator = torch.exp(z)
    # read  MN elements ; write M  elements
    denominator = numerator.sum(dim=1)
    # read MN + M elements ; write MN elements
    ret = numerator / denominator[:, None]
    # in total: read 5MN + 2M elements ; wrote 3MN + 2M elements
    return ret
```

**EN:** `naive_softmax` spells out the stabilized algorithm in several tensor steps: row max, subtraction, exponentiation, row sum, and division. The inline comments count memory traffic to show why multiple unfused passes are expensive.

**CN:** `naive_softmax` 用多个张量步骤明确写出数值稳定版 softmax：行最大值、平移、指数、按行求和、再做除法。行内注释还统计了访存次数，用来说明多个未融合步骤为什么代价高。

### Lines 63-81 — Why fusion helps / 为什么融合能提速

```python
# When implemented naively in PyTorch, computing :code:`y = naive_softmax(x)` for :math:`x \in R^{M \times N}`
# requires reading :math:`5MN + 2M` elements from DRAM and writing back :math:`3MN + 2M` elements.
# This is obviously wasteful; we'd prefer to have a custom "fused" kernel that only reads
# X once and does all the necessary computations on-chip.
# Doing so would require reading and writing back only :math:`MN` bytes, so we could
# expect a theoretical speed-up of ~4x (i.e., :math:`(8MN + 4M) / 2MN`).
# The `torch.jit.script` flags aims to perform this kind of "kernel fusion" automatically
# but, as we will see later, it is still far from ideal.

# %%
# Compute Kernel
# --------------
#
# Our softmax kernel works as follows: each program loads a set of rows of the input matrix X strided by number of programs,
# normalizes it and writes back the result to the output Y.
#
# Note that one important limitation of Triton is that each block must have a
# power-of-two number of elements, so we need to internally "pad" each row and guard the
# memory operations properly if we want to handle any possible input shapes:
```

**EN:** These comments quantify the waste in the naive implementation and explain the key Triton constraint for this kernel: each row is padded to a power-of-two block so one program can process the full row efficiently.

**CN:** 这一段注释量化了朴素实现的额外访存，并解释了该内核的重要 Triton 约束：每一行会被填充到 2 的幂大小，这样一个 program 就能高效处理整行。

### Lines 84-109 — Fused softmax kernel / 融合 softmax 内核

```python
@triton.jit
def softmax_kernel(output_ptr, input_ptr, input_row_stride, output_row_stride, n_rows, n_cols, BLOCK_SIZE: tl.constexpr,
                   num_stages: tl.constexpr):
    # starting row of the program
    row_start = tl.program_id(0)
    row_step = tl.num_programs(0)
    for row_idx in tl.range(row_start, n_rows, row_step, num_stages=num_stages):
        # The stride represents how much we need to increase the pointer to advance 1 row
        row_start_ptr = input_ptr + row_idx * input_row_stride
        # The block size is the next power of two greater than n_cols, so we can fit each
        # row in a single block
        col_offsets = tl.arange(0, BLOCK_SIZE)
        input_ptrs = row_start_ptr + col_offsets
        # Load the row into SRAM, using a mask since BLOCK_SIZE may be > than n_cols
        mask = col_offsets < n_cols
        row = tl.load(input_ptrs, mask=mask, other=-float('inf'))
        # Subtract maximum for numerical stability
        row_minus_max = row - tl.max(row, axis=0)
        # Note that exponentiation in Triton is fast but approximate (i.e., think __expf in CUDA)
        numerator = tl.exp(row_minus_max)
        denominator = tl.sum(numerator, axis=0)
        softmax_output = numerator / denominator
        # Write back output to DRAM
        output_row_start_ptr = output_ptr + row_idx * output_row_stride
        output_ptrs = output_row_start_ptr + col_offsets
        tl.store(output_ptrs, softmax_output, mask=mask)
```

**EN:** Each program processes rows in a persistent loop. It loads one padded row into SRAM, uses `tl.max` and `tl.sum` for reduction, computes `exp(row - max)` for numerical stability, and writes the normalized row back once.

**CN:** 每个 program 通过持久化循环处理多行数据。它把一整行（按需要填充后）读入 SRAM，使用 `tl.max` 和 `tl.sum` 做归约，借助 `exp(row - max)` 保证数值稳定，并最终只写回一次归一化结果。

### Lines 115-123 — Device property query / 设备属性查询

```python
properties = driver.active.utils.get_device_properties(DEVICE.index)
NUM_SM = properties["multiprocessor_count"]
NUM_REGS = properties["max_num_regs"]
SIZE_SMEM = properties["max_shared_mem"]
WARP_SIZE = properties["warpSize"]
target = triton.runtime.driver.active.get_current_target()
kernels = {}
```

**EN:** The tutorial queries SM count, register limits, shared-memory size, warp size, and target information from the active driver. These values are later used to estimate occupancy rather than hard-coding launch counts.

**CN:** 教程从当前驱动中查询 SM 数量、寄存器上限、共享内存大小、warp 大小和目标信息。后续会利用这些数据估算占用率，而不是把启动规模写死。

### Lines 124-175 — Host-side launch logic / 主机端启动逻辑

```python
def softmax(x):
    n_rows, n_cols = x.shape

    # The block size of each loop iteration is the smallest power of two greater than the number of columns in `x`
    BLOCK_SIZE = triton.next_power_of_2(n_cols)

    # Another trick we can use is to ask the compiler to use more threads per row by
    # increasing the number of warps (`num_warps`) over which each row is distributed.
    # You will see in the next tutorial how to auto-tune this value in a more natural
    # way so you don't have to come up with manual heuristics yourself.
    num_warps = 8

    # Number of software pipelining stages.
    num_stages = 4 if SIZE_SMEM > 200000 else 2

    # Allocate output
    y = torch.empty_like(x)

    # pre-compile kernel to get register usage and compute thread occupancy.
    kernel = softmax_kernel.warmup(y, x, x.stride(0), y.stride(0), n_rows, n_cols, BLOCK_SIZE=BLOCK_SIZE,
                                   num_stages=num_stages, num_warps=num_warps, grid=(1, ))
    kernel._init_handles()
    n_regs = kernel.n_regs
    size_smem = kernel.metadata.shared
    if is_hip():
        # NUM_REGS represents the number of regular purpose registers. On CDNA architectures this is half of all registers available.
        # However, this is not always the case. In most cases all registers can be used as regular purpose registers.
        # ISA SECTION (3.6.4 for CDNA3)
        # VGPRs are allocated out of two pools: regular VGPRs and accumulation VGPRs. Accumulation VGPRs are used
        # with matrix VALU instructions, and can also be loaded directly from memory. A wave may have up to 512 total
        # VGPRs, 256 of each type. When a wave has fewer than 512 total VGPRs, the number of each type is flexible - it is
        # not required to be equal numbers of both types.
        NUM_GPRS = NUM_REGS
        if is_cdna():
            NUM_GPRS = NUM_REGS * 2

        # MAX_NUM_THREADS represents maximum number of resident threads per multi-processor.
        # When we divide this number with WARP_SIZE we get maximum number of waves that can
        # execute on a CU (multi-processor)  in parallel.
        MAX_NUM_THREADS = properties["max_threads_per_sm"]
        max_num_waves = MAX_NUM_THREADS // WARP_SIZE
        occupancy = min(NUM_GPRS // WARP_SIZE // n_regs, max_num_waves) // num_warps
    else:
        occupancy = NUM_REGS // (n_regs * WARP_SIZE * num_warps)
    occupancy = min(occupancy, SIZE_SMEM // size_smem)
    num_programs = NUM_SM * occupancy

    num_programs = min(num_programs, n_rows)

    # Create a number of persistent programs.
    kernel[(num_programs, 1, 1)](y, x, x.stride(0), y.stride(0), n_rows, n_cols, BLOCK_SIZE, num_stages)
    return y
```

**EN:** The `softmax` wrapper chooses the next power-of-two block size, sets heuristics for warps and pipeline stages, allocates output, then warms up the kernel to inspect register/shared-memory usage. From that metadata it estimates how many persistent programs can reside per SM and launches only that many programs for the whole matrix.

**CN:** `softmax` 包装函数先选择不小于列数的最小 2 次幂块大小，再设置 warp 数和流水线 stage 的启发式参数，分配输出张量，并通过预热编译拿到寄存器与共享内存占用。随后依据这些元数据估算每个 SM 可驻留多少个持久化 program，并只启动这么多 program 来覆盖整张矩阵。

### Lines 178-190 — Irregular-shape test / 非规则形状测试

```python
# %%
# Unit Test
# ---------

# %%
# We make sure that we test our kernel on a matrix with an irregular number of rows and columns.
# This will allow us to verify that our padding mechanism works.

torch.manual_seed(0)
x = torch.randn(1823, 781, device=DEVICE)
y_triton = softmax(x)
y_torch = torch.softmax(x, axis=1)
assert torch.allclose(y_triton, y_torch), (y_triton, y_torch)
```

**EN:** The correctness test deliberately uses dimensions that are not powers of two or tile multiples. Matching `torch.softmax` here confirms that masking and padding logic are correct.

**CN:** 正确性测试故意使用既不是 2 的幂、也不是 tile 整数倍的矩阵尺寸。若结果与 `torch.softmax` 一致，就说明 mask 和填充逻辑实现正确。

### Lines 203-214 — Benchmark configuration / 基准测试配置

```python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=['N'],  # argument names to use as an x-axis for the plot
        x_vals=[128 * i for i in range(2, 100)],  # different possible values for `x_name`
        line_arg='provider',  # argument name whose value corresponds to a different line in the plot
        line_vals=['triton', 'torch', 'naive_softmax'],  # possible values for `line_arg``
        line_names=["Triton", "Torch", "Naive Softmax"],  # label name for the lines
        styles=[('blue', '-'), ('green', '-'), ('red', '-')],  # line styles
        ylabel="GB/s",  # label name for the y-axis
        plot_name="softmax-performance",  # name for the plot. Used also as a file name for saving the plot.
        args={'M': 4096},  # values for function arguments not in `x_names` and `y_name`
    ))
```

**EN:** The benchmark sweeps the column dimension `N` while fixing `M=4096`, and compares Triton against `torch.softmax` plus the explicitly unfused `naive_softmax` baseline.

**CN:** 基准测试固定 `M=4096`，扫描列维度 `N`，并同时对比 Triton、`torch.softmax` 以及显式未融合的 `naive_softmax` 基线。

### Lines 215-226 — Benchmark body / 基准测试主体

```python
def benchmark(M, N, provider):
    x = torch.randn(M, N, device=DEVICE, dtype=torch.float32)
    stream = getattr(torch, DEVICE.type).Stream()
    getattr(torch, DEVICE.type).set_stream(stream)
    if provider == 'torch':
        ms = triton.testing.do_bench(lambda: torch.softmax(x, axis=-1))
    if provider == 'triton':
        ms = triton.testing.do_bench(lambda: softmax(x))
    if provider == 'naive_softmax':
        ms = triton.testing.do_bench(lambda: naive_softmax(x))
    gbps = lambda ms: 2 * x.numel() * x.element_size() * 1e-9 / (ms * 1e-3)
    return gbps(ms)
```

**EN:** A fresh stream is created before timing so benchmark work is isolated. The throughput metric uses two tensor-sized transfers—one read and one write—matching the fused kernel's intended memory behavior.

**CN:** 计时前会创建一条新的 stream，以隔离基准测试工作负载。吞吐量指标按一次读和一次写这两次张量级传输来计算，与融合内核的目标访存行为一致。

### Lines 229-235 — Run and interpret results / 运行并解读结果

```python
benchmark.run(show_plots=True, print_data=True)

# %%
# In the above plot, we can see that:
#  - Triton is 4x faster than the Torch JIT. This confirms our suspicions that the Torch JIT does not do any fusion here.
#  - Triton is noticeably faster than :code:`torch.softmax` -- in addition to being **easier to read, understand and maintain**.
#    Note however that the PyTorch `softmax` operation is more general and will work on tensors of any shape.
```

**EN:** The final `run` call executes the benchmark, and the concluding comments interpret the expected result: fusion lets Triton outperform the unfused approach and often beat the generic PyTorch kernel for this specific row-fitting regime.

**CN:** 最后的 `run` 调用真正执行基准测试，而结尾注释解释了预期现象：融合让 Triton 明显优于未融合实现，并且在“单行能放入片上存储”的场景下通常也优于通用的 PyTorch softmax。

## Key Concepts / 关键概念

- **EN:** Softmax is bandwidth-bound, so reducing memory traffic can matter more than reducing arithmetic.
  **CN:** Softmax 往往受限于内存带宽，因此减少访存次数通常比减少算术量更重要。

- **EN:** A single Triton program can keep one row in SRAM, enabling max-reduction, exponentiation, sum-reduction, and normalization without intermediate global writes.
  **CN:** 单个 Triton program 可以把一整行保存在 SRAM 中，从而在没有中间全局写回的情况下完成最大值归约、指数、求和归约和归一化。

- **EN:** Persistent programs are launched according to occupancy estimates from compiler metadata, not simply one program per row.
  **CN:** 持久化 program 的启动数量由编译元数据估算出的占用率决定，而不是简单地做到“一行一个 program”。

- **EN:** Backend-specific hardware properties matter for performance tuning, especially on HIP/CDNA devices.
  **CN:** 后端相关的硬件属性对性能调优很重要，尤其是在 HIP/CDNA 设备上。

## Dependencies / 依赖关系

- **EN:** `naive_softmax()` provides the bandwidth-heavy reference used for comparison.
  **CN:** `naive_softmax()` 提供了一个高访存开销的参考实现，用于对比。

- **EN:** `softmax()` depends on `softmax_kernel` and on runtime device properties obtained from `triton.runtime.driver`.
  **CN:** `softmax()` 依赖 `softmax_kernel`，也依赖从 `triton.runtime.driver` 读取的运行时设备属性。

- **EN:** `torch.softmax` is the correctness oracle and one of the benchmark providers.
  **CN:** `torch.softmax` 既是正确性对照，也是基准测试中的一个对比实现。

- **EN:** `triton.testing` supplies `Benchmark`, `perf_report`, and `do_bench` for automated measurement.
  **CN:** `triton.testing` 提供 `Benchmark`、`perf_report` 与 `do_bench`，用于自动化性能测量。
