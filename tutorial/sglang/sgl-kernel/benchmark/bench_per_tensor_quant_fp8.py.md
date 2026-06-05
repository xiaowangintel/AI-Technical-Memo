# bench_per_tensor_quant_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_per_tensor_quant_fp8.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `per tensor quantization FP8` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `per tensor quantization FP8` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and module setup
````python
import itertools
import math
import os
from typing import Any, Dict, List, Optional, Tuple

import numpy as np
import torch
import triton
import triton.testing

from sglang.jit_kernel.per_tensor_quant_fp8 import per_tensor_quant_fp8
from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 13-14: Comments and local context
````python

# Optional imports
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 15-21: Optional dependency handling
````python
try:
    from vllm import _custom_ops as ops

    VLLM_AVAILABLE = True
except ImportError:
    ops = None
    VLLM_AVAILABLE = False
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 23-23: Imports and module setup
````python
from sglang.srt.utils import is_hip
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 25-29: Constants and configuration
````python
_is_hip = is_hip()

IS_CI = is_in_ci()

fp8_type_ = torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn
````
**EN:** This block defines shared constants or configuration values such as `_is_hip`, `IS_CI`, `fp8_type_`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_is_hip`, `IS_CI`, `fp8_type_`），供后续函数或控制流程复用。

### Lines 32-39: `vllm_scaled_fp8_quant` definition
````python
def vllm_scaled_fp8_quant(
    input: torch.Tensor,
    scale: Optional[torch.Tensor] = None,
) -> Tuple[torch.Tensor, torch.Tensor]:
    if not VLLM_AVAILABLE:
        # Fallback to SGLang implementation
        return sglang_scaled_fp8_quant(input, scale)
    return ops.scaled_fp8_quant(input, scale)
````
**EN:** This section defines `vllm_scaled_fp8_quant` and implements the core logic associated with vllm scaled FP8 quantization.
**CN:** 该部分定义 `vllm_scaled_fp8_quant`，并实现与 vllm scaled FP8 quantization 相关的核心逻辑。

### Lines 42-54: `sglang_scaled_fp8_quant` definition
````python
def sglang_scaled_fp8_quant(
    input: torch.Tensor,
    scale: Optional[torch.Tensor] = None,
) -> Tuple[torch.Tensor, torch.Tensor]:
    fp8_type_: torch.dtype = torch.float8_e4m3fn
    output = torch.empty_like(input, device=input.device, dtype=fp8_type_)
    is_static = True
    if scale is None:
        scale = torch.zeros(1, device=input.device, dtype=torch.float32)
        is_static = False
    per_tensor_quant_fp8(input, output, scale, is_static)

    return output, scale
````
**EN:** This section defines `sglang_scaled_fp8_quant` and implements the core logic associated with sglang scaled FP8 quantization.
**CN:** 该部分定义 `sglang_scaled_fp8_quant`，并实现与 sglang scaled FP8 quantization 相关的核心逻辑。

### Lines 57-77: `calculate_diff` definition
````python
def calculate_diff(batch_size: int, seq_len: int):
    """Calculate difference between VLLM and SGLang implementations."""
    device = torch.device("cuda")
    x = torch.rand((batch_size, seq_len), dtype=torch.float16, device=device)

    if not VLLM_AVAILABLE:
        print("⚠️ vLLM not available, skipping comparison")
        return

    vllm_out, vllm_scale = vllm_scaled_fp8_quant(x)
    sglang_out, sglang_scale = sglang_scaled_fp8_quant(x)

    scale_diff = torch.abs(vllm_scale - sglang_scale).item()
    output_diff = torch.abs(vllm_out.float() - sglang_out.float()).mean().item()

    if torch.allclose(
        vllm_out.to(torch.float32), sglang_out.to(torch.float32), rtol=1e-3, atol=1e-5
    ) and torch.allclose(vllm_scale, sglang_scale, rtol=1e-3, atol=1e-5):
        print("✅ All implementations match")
    else:
        print("❌ Implementations differ")
````
**EN:** This section defines `calculate_diff`. It calculates the `diff` path used by the module. Docstring summary: Calculate difference between VLLM and SGLang implementations.
**CN:** 该部分定义 `calculate_diff`。它负责计算模块中与 `diff` 相关的处理路径。 文档字符串摘要：Calculate difference between VLLM and SGLang implementations.

### Lines 78-80: Comments and local context
````python


# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 81-86: Conditional logic
````python
if IS_CI:
    batch_size_range = [16]  # Single batch size for CI
    seq_len_range = [64]  # Single sequence length for CI
else:
    batch_size_range = [16, 32, 64, 128]
    seq_len_range = [64, 128, 256, 512, 1024, 2048]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 88-88: Constants and configuration
````python
configs = list(itertools.product(batch_size_range, seq_len_range))
````
**EN:** This block defines shared constants or configuration values such as `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `configs`），供后续函数或控制流程复用。

### Lines 91-98: Conditional logic
````python
if VLLM_AVAILABLE:
    line_vals = ["vllm", "sglang"]
    line_names = ["VLLM", "SGL Kernel"]
    styles = [("blue", "-"), ("green", "-")]
else:
    line_vals = ["sglang"]
    line_names = ["SGL Kernel"]
    styles = [("green", "-")]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 101-129: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size", "seq_len"],
        x_vals=configs,
        line_arg="provider",
        line_vals=line_vals,
        line_names=line_names,
        styles=styles,
        ylabel="us",
        plot_name="per-tensor-quant-fp8-performance",
        args={},
    )
)
def benchmark(batch_size, seq_len, provider):
    dtype = torch.float16
    device = torch.device("cuda")

    x = torch.randn(batch_size * seq_len, 4096, device=device, dtype=dtype)

    quantiles = [0.5, 0.2, 0.8]

    if provider == "vllm":
        fn = lambda: vllm_scaled_fp8_quant(x.clone())
    elif provider == "sglang":
        fn = lambda: sglang_scaled_fp8_quant(x.clone())

    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(fn, quantiles=quantiles)

    return 1000 * ms, 1000 * max_ms, 1000 * min_ms
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 132-134: Command-line entry point
````python
if __name__ == "__main__":
    calculate_diff(batch_size=4, seq_len=4096)
    benchmark.run(print_data=True)
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `vllm_scaled_fp8_quant`, `sglang_scaled_fp8_quant`, `calculate_diff`, `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.jit_kernel.per_tensor_quant_fp8`, `sglang.srt.utils`, `sglang.utils`
- **External / 外部**: `itertools`, `math`, `numpy`, `os`, `torch`, `triton`, `triton.testing`, `typing`, `vllm`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
