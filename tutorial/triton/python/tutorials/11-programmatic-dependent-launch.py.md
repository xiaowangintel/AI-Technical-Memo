# 11-programmatic-dependent-launch.py — Code Analysis / 代码分析

## Source / 来源

- **Source file / 源文件:** `/root/xw/triton/python/tutorials/11-programmatic-dependent-launch.py`
- **Purpose (EN):** Demonstrates Triton programmatic dependent launch (PDL) on a vector-add kernel using CUDA grid dependency control instructions.
- **目的（中文）:** 演示如何在 Triton 向量加法内核中使用程序化依赖启动（PDL），并结合 CUDA 的 grid dependency control 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15

````python
"""
Programmatic Dependent Launch
=====================
This script demonstrates the use of programmatic dependent launch (PDL) ontop of the vector-add example using Triton.

For CUDA reference on programmatic dependent launch see https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programmatic-dependent-launch-and-synchronization.
For PTX reference on programmatic dependent launch see https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#parallel-synchronization-and-communication-instructions-griddepcontrol.

.. code-block:: bash
    python 11-programmatic-dependent-launch.py
"""

import torch
import triton
import triton.language as tl
````
**EN:** The module docstring states the goal: extend a simple vector-add example with programmatic dependent launch. The imports show that the example is purely host-side Python plus a Triton kernel written with `triton.language`.
**CN:** 模块文档字符串直接说明目标：在简单向量加法示例上加入程序化依赖启动。导入部分表明该示例由 Python 主机端代码和用 `triton.language` 编写的 Triton 内核组成。

### Lines 18-23

````python
def is_cuda():
    return triton.runtime.driver.active.get_current_target().backend == "cuda"


def supports_pdl():
    return is_cuda() and torch.cuda.get_device_capability()[0] >= 9
````
**EN:** These helpers isolate capability checks. `is_cuda()` verifies the active Triton target backend, while `supports_pdl()` additionally requires compute capability 9.x or newer, matching the hardware generation that exposes the needed CUDA/PTX support.
**CN:** 这两个辅助函数封装了能力检测。`is_cuda()` 判断当前 Triton 目标是否为 CUDA，`supports_pdl()` 则进一步要求设备计算能力至少为 9.x，对应支持相关 CUDA/PTX 能力的硬件代际。

### Lines 27-55

````python
@triton.jit
def add_kernel(x_ptr,  #
               y_ptr,  #
               output_ptr,  #
               n_elements,  #
               BLOCK_SIZE: tl.constexpr,  #
               USE_GDC: tl.constexpr,  #
               ):
    pid = tl.program_id(axis=0)
    block_start = pid * BLOCK_SIZE
    offsets = block_start + tl.arange(0, BLOCK_SIZE)
    mask = offsets < n_elements
    if USE_GDC:
        # GDC wait waits for ALL programs in the the prior kernel to complete before continuing.
        # This ensures any memory operations happen before the wait in program order,
        # e.g. if the prior kernel writes to x or y the new values will be visible.
        tl.extra.cuda.gdc_wait()

    x = tl.load(x_ptr + offsets, mask=mask)
    y = tl.load(y_ptr + offsets, mask=mask)
    if USE_GDC:
        # GDC launch dependents hints the runtime system to launch dependent kernels.
        # These dependent kernels must also be launched with PDL enabled.
        # Once GDC launch has been issued by ALL programs or
        # programs have finished, the dependent grid can begin if there are enough resources.
        # Note: this by itself provides no additional memory-ordering guarentees, unlike `gdc_wait`
        tl.extra.cuda.gdc_launch_dependents()
    output = x + y
    tl.store(output_ptr + offsets, output, mask=mask)
````
**EN:** The kernel is structurally a standard blocked vector-add: program id selects a tile, `tl.arange` builds element offsets, and `mask` guards the tail. The PDL-specific logic is guarded by the `USE_GDC` constexpr: `gdc_wait()` enforces ordering with prior dependent work, while `gdc_launch_dependents()` tells the runtime that downstream dependent grids may start once all programs reach the launch point or finish.
**CN:** 这个内核本质上仍是标准的分块向量加法：program id 选定 tile，`tl.arange` 生成元素偏移，`mask` 处理尾块。PDL 相关逻辑由 `USE_GDC` 这个 constexpr 控制：`gdc_wait()` 用于与前序依赖工作建立顺序关系，而 `gdc_launch_dependents()` 则通知运行时：当所有 program 都到达该点或结束后，可以启动下游依赖 grid。

### Lines 58-68

````python
def add(x: torch.Tensor, y: torch.Tensor, launch_pdl: bool = True):
    output = torch.empty_like(x)
    assert x.device == y.device and output.device == x.device
    n_elements = output.numel()
    grid = lambda meta: (triton.cdiv(n_elements, meta['BLOCK_SIZE']), )
    add_kernel[grid](
        x, y, output, n_elements, BLOCK_SIZE=1024,
        USE_GDC=launch_pdl,  # set constexpr in kernel to use grid dependence control
        launch_pdl=launch_pdl,  # launch kernel with PDL flag set enabled
    )
    return output
````
**EN:** The Python launcher allocates the output tensor, validates device placement, computes a 1D grid with `triton.cdiv`, and launches the kernel. A key detail is that PDL is configured twice: `USE_GDC` specializes device code at compile time, while `launch_pdl` enables the launch-time runtime behavior expected by dependent kernels.
**CN:** Python 侧 launcher 负责分配输出张量、检查设备一致性、用 `triton.cdiv` 计算一维 grid，并启动内核。这里的关键点是 PDL 需要双重配置：`USE_GDC` 在编译期特化设备代码，而 `launch_pdl` 在运行时打开与依赖内核配套的启动语义。

### Lines 71-80

````python
def validate(n_elements):
    x = torch.rand(n_elements, device="cuda", dtype=torch.float32)
    y = torch.rand(n_elements, device="cuda", dtype=torch.float32)

    torch_result = x + y
    add_result = add(x, y)

    torch_vs_add = "✅" if torch.allclose(torch_result, add_result, atol=1.0) else "❌"
    print(f"Number of Elements={n_elements} verification naive vs: ", end="")
    print(f"add: {torch_vs_add}")
````
**EN:** `validate` builds random CUDA tensors, compares Triton output to the eager PyTorch result, and prints a simple pass/fail marker. The numerical tolerance is loose (`atol=1.0`) because the focus of the tutorial is launch semantics rather than numerical edge cases.
**CN:** `validate` 生成随机 CUDA 张量，将 Triton 结果与 PyTorch 的即时计算结果对比，并打印简单的通过/失败标记。这里容差较宽（`atol=1.0`），因为本教程关注的是启动语义，而不是数值精度边界问题。

### Lines 83-107

````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["size"],
        x_vals=[2**i for i in range(23, 28, 1)],
        x_log=False,
        line_arg="provider",
        line_vals=["pdl-fp32", "fp32"],
        line_names=["PDL", "No PDL"],
        styles=[("red", "-"), ("blue", "-")],
        ylabel='GB/s',
        plot_name="pdl-performance",
        args={},
    ))
def benchmark(size, provider):
    x = torch.rand(size, device="cuda", dtype=torch.float32)
    y = torch.rand(size, device="cuda", dtype=torch.float32)

    quantiles = [0.5, 0.2, 0.8]

    fn = lambda: add(x, y, "pdl" in provider)

    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(fn, quantiles=quantiles, rep=100)

    gbps = lambda ms: 3 * x.numel() * x.element_size() * 1e-9 / (ms * 1e-3)
    return gbps(ms), gbps(max_ms), gbps(min_ms)
````
**EN:** The benchmark is wrapped in `triton.testing.perf_report`, which defines the problem sizes, plot labels, and output style. `benchmark()` toggles PDL by checking whether the provider name contains `pdl`, runs the kernel under CUDA graphs with `do_bench_cudagraph`, and reports effective bandwidth based on two reads plus one write per element.
**CN:** 基准测试通过 `triton.testing.perf_report` 包装，统一定义问题规模、绘图标签和输出样式。`benchmark()` 通过 provider 名中是否包含 `pdl` 来切换 PDL，使用 `do_bench_cudagraph` 在 CUDA Graph 下测量性能，并按每个元素“两次读 + 一次写”计算有效带宽。

### Lines 110-116

````python
if __name__ == "__main__":

    if supports_pdl():
        validate(1024)
        benchmark.run(print_data=True, show_plots=True, save_path=".")
    else:
        print("PDL is not supported on this device")
````
**EN:** The `__main__` block keeps the tutorial self-contained. It only runs validation and plotting when the device supports PDL; otherwise it exits cleanly with an explanatory message instead of failing at launch time.
**CN:** `__main__` 代码块让教程可以独立执行。只有当设备支持 PDL 时才运行验证与绘图；否则会给出明确提示并正常退出，避免在启动时直接报错。

## Key Concepts / 关键概念

- **PDL and GDC / PDL 与 GDC**
  - **EN:** Programmatic dependent launch in Triton is exposed through a launch flag plus CUDA grid-dependency-control intrinsics inside the kernel.
  - **CN:** Triton 中的程序化依赖启动由启动参数和内核内部的 CUDA grid-dependency-control 原语共同实现。
- **Blocked vector-add structure / 分块向量加法结构**
  - **EN:** Even with advanced launch semantics, the kernel still follows the familiar Triton pattern of program ids, offsets, masks, loads, compute, and store.
  - **CN:** 即便加入高级启动语义，内核结构仍遵循 Triton 常见模式：program id、offset、mask、load、compute、store。
- **Launch-time vs compile-time control / 运行时与编译时控制**
  - **EN:** The example distinguishes compile-time specialization (`USE_GDC`) from runtime launch configuration (`launch_pdl`).
  - **CN:** 该示例清楚地区分了编译期特化（`USE_GDC`）和运行时启动配置（`launch_pdl`）。
- **Bandwidth benchmarking / 带宽基准测试**
  - **EN:** Performance is measured as effective GB/s from total transferred bytes rather than FLOP/s, which fits a memory-bound add kernel.
  - **CN:** 性能指标使用有效 GB/s 而非 FLOP/s，更符合这个内存带宽受限的加法内核。

## Dependencies / 依赖关系

- **`torch`**
  - **EN:** Creates CUDA tensors for validation and benchmarking.
  - **CN:** 用于创建 CUDA 张量并执行验证与基准测试。
- **`triton`**
  - **EN:** Provides kernel launching, target detection, and testing helpers.
  - **CN:** 提供内核启动、目标设备检测和测试工具。
- **`triton.language as tl`**
  - **EN:** Defines device-side primitives such as `program_id`, `arange`, `load`, and CUDA GDC intrinsics.
  - **CN:** 提供设备端原语，如 `program_id`、`arange`、`load` 以及 CUDA 的 GDC 指令封装。
- **CUDA compute capability >= 9**
  - **EN:** Required for PDL support in this tutorial.
  - **CN:** 本教程中的 PDL 功能要求 CUDA 计算能力至少为 9。
