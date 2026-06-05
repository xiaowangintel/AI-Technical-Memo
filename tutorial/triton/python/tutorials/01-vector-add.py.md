# 01-vector-add.py — Code Analysis / 代码分析

## Source / 来源

- **Source file / 源文件:** `/root/xw/triton/python/tutorials/01-vector-add.py`

- **EN:** Introduces Triton's SPMD programming model with a simple elementwise vector-add kernel, then validates and benchmarks it against PyTorch.

- **CN:** 用一个简单的按元素向量加法内核介绍 Triton 的 SPMD 编程模型，并与 PyTorch 做正确性验证和性能对比。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15 — Tutorial overview / 教程概述

```python
"""
Vector Addition
===============

In this tutorial, you will write a simple vector addition using Triton.

In doing so, you will learn about:

* The basic programming model of Triton.

* The `triton.jit` decorator, which is used to define Triton kernels.

* The best practices for validating and benchmarking your custom ops against native reference implementations.

"""
```

**EN:** The opening docstring states that this example is intentionally small: it teaches the basic Triton execution model, how `@triton.jit` turns a Python function into a GPU kernel, and why validation/benchmarking matter for custom ops.

**CN:** 开头的文档字符串说明这个示例刻意保持简单：它用于讲清 Triton 的基础执行模型、`@triton.jit` 如何把 Python 函数变成 GPU 内核，以及为什么自定义算子必须做正确性验证与性能测试。

### Lines 17-26 — Imports and active device / 导入与当前设备

```python
# %%
# Compute Kernel
# --------------

import torch

import triton
import triton.language as tl

DEVICE = triton.runtime.driver.active.get_active_torch_device()
```

**EN:** `torch` provides tensors and the reference implementation, while `triton` and `triton.language as tl` provide the kernel DSL and runtime helpers. `DEVICE` is resolved from Triton's active driver so later tensors and kernels use the same backend device.

**CN:** `torch` 提供张量和参考实现，`triton` 与 `triton.language as tl` 提供内核 DSL 与运行时辅助函数。`DEVICE` 从 Triton 当前激活的驱动中获取，确保后续张量和内核都落在同一个后端设备上。

### Lines 29-54 — Triton vector-add kernel / Triton 向量加法内核

```python
@triton.jit
def add_kernel(x_ptr,  # *Pointer* to first input vector.
               y_ptr,  # *Pointer* to second input vector.
               output_ptr,  # *Pointer* to output vector.
               n_elements,  # Size of the vector.
               BLOCK_SIZE: tl.constexpr,  # Number of elements each program should process.
               # NOTE: `constexpr` so it can be used as a shape value.
               ):
    # There are multiple 'programs' processing different data. We identify which program
    # we are here:
    pid = tl.program_id(axis=0)  # We use a 1D launch grid so axis is 0.
    # This program will process inputs that are offset from the initial data.
    # For instance, if you had a vector of length 256 and block_size of 64, the programs
    # would each access the elements [0:64, 64:128, 128:192, 192:256].
    # Note that offsets is a list of pointers:
    block_start = pid * BLOCK_SIZE
    offsets = block_start + tl.arange(0, BLOCK_SIZE)
    # Create a mask to guard memory operations against out-of-bounds accesses.
    mask = offsets < n_elements
    # Load x and y from DRAM, masking out any extra elements in case the input is not a
    # multiple of the block size.
    x = tl.load(x_ptr + offsets, mask=mask)
    y = tl.load(y_ptr + offsets, mask=mask)
    output = x + y
    # Write x + y back to DRAM.
    tl.store(output_ptr + offsets, output, mask=mask)
```

**EN:** Each Triton program handles one contiguous block of `BLOCK_SIZE` elements. The kernel uses `tl.program_id(0)` to choose its block, `tl.arange` to build element offsets, a mask to protect tail elements, then `tl.load`/`tl.store` to move data between DRAM and registers.

**CN:** 每个 Triton program 处理一个长度为 `BLOCK_SIZE` 的连续数据块。内核通过 `tl.program_id(0)` 选择自己负责的块，用 `tl.arange` 生成元素偏移，用 mask 保护尾部越界元素，再通过 `tl.load`/`tl.store` 在 DRAM 与寄存器之间搬运数据。

### Lines 57-78 — Python wrapper and launch grid / Python 包装函数与启动网格

```python
# %%
# Let's also declare a helper function to (1) allocate the `z` tensor
# and (2) enqueue the above kernel with appropriate grid/block sizes:


def add(x: torch.Tensor, y: torch.Tensor):
    # We need to preallocate the output.
    output = torch.empty_like(x)
    assert x.device == DEVICE and y.device == DEVICE and output.device == DEVICE
    n_elements = output.numel()
    # The SPMD launch grid denotes the number of kernel instances that run in parallel.
    # It is analogous to CUDA launch grids. It can be either Tuple[int], or Callable(metaparameters) -> Tuple[int].
    # In this case, we use a 1D grid where the size is the number of blocks:
    grid = lambda meta: (triton.cdiv(n_elements, meta['BLOCK_SIZE']), )
    # NOTE:
    #  - Each torch.tensor object is implicitly converted into a pointer to its first element.
    #  - `triton.jit`'ed functions can be indexed with a launch grid to obtain a callable GPU kernel.
    #  - Don't forget to pass meta-parameters as keywords arguments.
    add_kernel[grid](x, y, output, n_elements, BLOCK_SIZE=1024)
    # We return a handle to z but, since `torch.cuda.synchronize()` hasn't been called, the kernel is still
    # running asynchronously at this point.
    return output
```

**EN:** The `add` helper allocates the output tensor, checks device placement, computes the total element count, and defines a 1D launch grid with `triton.cdiv`. Calling `add_kernel[grid](...)` is Triton's launch syntax: runtime tensor arguments become pointers, while `BLOCK_SIZE` is a compile-time meta-parameter.

**CN:** `add` 辅助函数负责分配输出张量、检查设备位置、计算元素总数，并通过 `triton.cdiv` 定义一维启动网格。`add_kernel[grid](...)` 是 Triton 的启动语法：运行时张量参数会变成指针，而 `BLOCK_SIZE` 是编译期元参数。

### Lines 81-93 — Correctness check / 正确性检查

```python
# %%
# We can now use the above function to compute the element-wise sum of two `torch.tensor` objects and test its correctness:

torch.manual_seed(0)
size = 98432
x = torch.rand(size, device=DEVICE)
y = torch.rand(size, device=DEVICE)
output_torch = x + y
output_triton = add(x, y)
print(output_torch)
print(output_triton)
print(f'The maximum difference between torch and triton is '
      f'{torch.max(torch.abs(output_torch - output_triton))}')
```

**EN:** The script seeds PyTorch, builds two random vectors on the active device, computes both the Torch reference and Triton result, then prints the maximum absolute difference. This verifies numerical agreement before any performance discussion.

**CN:** 脚本先设定随机种子，在当前设备上生成两个随机向量，分别计算 Torch 参考结果和 Triton 结果，再打印最大绝对误差。在讨论性能之前，这一步先确认数值一致性。

### Lines 107-119 — Benchmark configuration / 基准测试配置

```python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=['size'],  # Argument names to use as an x-axis for the plot.
        x_vals=[2**i for i in range(12, 28, 1)],  # Different possible values for `x_name`.
        x_log=True,  # x axis is logarithmic.
        line_arg='provider',  # Argument name whose value corresponds to a different line in the plot.
        line_vals=['triton', 'torch'],  # Possible values for `line_arg`.
        line_names=['Triton', 'Torch'],  # Label name for the lines.
        styles=[('blue', '-'), ('green', '-')],  # Line styles.
        ylabel='GB/s',  # Label name for the y-axis.
        plot_name='vector-add-performance',  # Name for the plot. Used also as a file name for saving the plot.
        args={},  # Values for function arguments not in `x_names` and `y_name`.
    ))
```

**EN:** The `@triton.testing.perf_report` decorator describes a benchmark sweep: vector sizes grow exponentially, results are plotted on a log-scale x-axis, and the comparison is between two providers, Triton and Torch.

**CN:** `@triton.testing.perf_report` 装饰器定义了一个基准测试扫描：向量大小按指数增长，横轴使用对数尺度，并在 Triton 与 Torch 两个实现之间做对比。

### Lines 120-129 — Benchmark body / 基准测试主体

```python
def benchmark(size, provider):
    x = torch.rand(size, device=DEVICE, dtype=torch.float32)
    y = torch.rand(size, device=DEVICE, dtype=torch.float32)
    quantiles = [0.5, 0.2, 0.8]
    if provider == 'torch':
        ms, min_ms, max_ms = triton.testing.do_bench(lambda: x + y, quantiles=quantiles)
    if provider == 'triton':
        ms, min_ms, max_ms = triton.testing.do_bench(lambda: add(x, y), quantiles=quantiles)
    gbps = lambda ms: 3 * x.numel() * x.element_size() * 1e-9 / (ms * 1e-3)
    return gbps(ms), gbps(max_ms), gbps(min_ms)
```

**EN:** For each size, the benchmark allocates fresh FP32 inputs and uses `triton.testing.do_bench` to measure median/min/max runtime. The throughput formula counts three tensor transfers—two reads and one write—so the reported metric is effective GB/s for this memory-bound operation.

**CN:** 对每个规模，基准函数都会新建 FP32 输入，并用 `triton.testing.do_bench` 测量中位数、最小和最大耗时。吞吐量公式按三次张量传输计算——两次读取和一次写回——因此这里报告的是这个带宽受限算子的有效 GB/s。

### Lines 132-135 — Benchmark execution / 执行基准测试

```python
# %%
# We can now run the decorated function above. Pass `print_data=True` to see the performance number, `show_plots=True` to plot them, and/or
# `save_path='/path/to/results/' to save them to disk along with raw CSV data:
benchmark.run(print_data=True, show_plots=True)
```

**EN:** The final call runs the benchmark and can print raw numbers and display plots. In Triton tutorials, this `run(...)` call is the standard bridge from benchmark specification to actual measurements.

**CN:** 最后一行真正执行基准测试，并可打印原始数据和显示图表。在 Triton 教程里，这种 `run(...)` 调用就是把基准配置转成实际测量结果的标准入口。

## Key Concepts / 关键概念

- **EN:** Triton uses an SPMD model: many program instances run the same kernel over different slices of data.
  **CN:** Triton 使用 SPMD 模型：大量 program 实例执行同一个内核，只是各自处理不同的数据切片。

- **EN:** `tl.constexpr` marks compile-time parameters such as `BLOCK_SIZE`, enabling shape-dependent code generation.
  **CN:** `tl.constexpr` 用于标记 `BLOCK_SIZE` 这类编译期参数，使编译器能生成依赖形状的代码。

- **EN:** Masking is essential whenever the problem size is not an exact multiple of the tile size.
  **CN:** 当问题规模不是 tile 大小的整数倍时，mask 是防止越界访存的关键机制。

- **EN:** Kernel launches are asynchronous with respect to the host unless explicit synchronization is inserted.
  **CN:** 除非显式同步，否则 Triton 内核启动对主机端来说是异步的。

## Dependencies / 依赖关系

- **EN:** `torch` supplies tensor allocation, random input generation, and the baseline `x + y` reference.
  **CN:** `torch` 负责张量分配、随机输入生成，以及基线实现 `x + y`。

- **EN:** `triton` provides launch utilities such as `cdiv` and the testing/benchmark framework.
  **CN:** `triton` 提供 `cdiv` 等启动辅助函数，以及测试和基准测试框架。

- **EN:** `triton.language` provides the kernel-side primitives: `program_id`, `arange`, `load`, and `store`.
  **CN:** `triton.language` 提供内核侧原语：`program_id`、`arange`、`load` 和 `store`。

- **EN:** `add()` depends directly on `add_kernel`, and `benchmark()` depends on both `add()` and PyTorch's reference implementation.
  **CN:** `add()` 直接依赖 `add_kernel`，而 `benchmark()` 同时依赖 `add()` 和 PyTorch 的参考实现。
