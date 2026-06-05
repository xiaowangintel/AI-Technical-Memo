# bench_per_token_quant_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_per_token_quant_fp8.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `per token quantization FP8` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `per token quantization FP8` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and module setup
````python
import itertools
import os
from typing import Optional, Tuple

import torch
import triton
import triton.testing
from sgl_kernel import sgl_per_token_quant_fp8

from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 11-12: Comments and local context
````python

# Optional vLLM import
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 13-19: Optional dependency handling
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

### Lines 21-21: Imports and module setup
````python
from sglang.srt.utils import is_hip
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 23-27: Constants and configuration
````python
_is_hip = is_hip()

IS_CI = is_in_ci()

fp8_type_ = torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn
````
**EN:** This block defines shared constants or configuration values such as `_is_hip`, `IS_CI`, `fp8_type_`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_is_hip`, `IS_CI`, `fp8_type_`），供后续函数或控制流程复用。

### Lines 28-29: Comments and local context
````python

# Get correct FP8 E4M3 maximum value
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 30-34: Conditional logic
````python
if _is_hip:
    FP8_E4M3_MAX = 224.0  # ROCM uses 224.0
else:
    # For CUDA, get the actual max value from the type
    FP8_E4M3_MAX = float(torch.finfo(fp8_type_).max)
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 37-60: `torch_per_token_quant_fp8` definition
````python
def torch_per_token_quant_fp8(
    input: torch.Tensor,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Pure PyTorch reference implementation for per-token FP8 quantization."""
    device = input.device
    dtype = input.dtype

    # Find max absolute value per token (row) - exactly like CUDA kernel
    max_vals = torch.abs(input).max(dim=1)[0]  # [num_tokens]

    # Calculate scale per token - exactly like CUDA kernel: scale = max_value / FP8_E4M3_MAX
    scales = max_vals / FP8_E4M3_MAX  # [num_tokens]

    # No special zero handling - directly compute 1.0 / scale like CUDA kernel
    scale_inv = 1.0 / scales  # [num_tokens]

    # Quantize: input * scale_inv, then clamp to FP8 range
    quantized_float = input * scale_inv.unsqueeze(1)  # Broadcast scale_inv
    quantized_float = torch.clamp(quantized_float, -FP8_E4M3_MAX, FP8_E4M3_MAX)

    # Convert to FP8 - use more explicit conversion
    quantized_fp8 = quantized_float.to(fp8_type_)

    return quantized_fp8, scales
````
**EN:** This section defines `torch_per_token_quant_fp8` and implements the core logic associated with torch per token quantization FP8. Docstring summary: Pure PyTorch reference implementation for per-token FP8 quantization.
**CN:** 该部分定义 `torch_per_token_quant_fp8`，并实现与 torch per token quantization FP8 相关的核心逻辑。 文档字符串摘要：Pure PyTorch reference implementation for per-token FP8 quantization.

### Lines 63-69: `vllm_per_token_quant_fp8` definition
````python
def vllm_per_token_quant_fp8(
    input: torch.Tensor,
) -> Tuple[torch.Tensor, torch.Tensor]:
    if not VLLM_AVAILABLE:
        # Fallback to SGLang implementation
        return sglang_per_token_quant_fp8(input)
    return ops.scaled_fp8_quant(input, use_per_token_if_dynamic=True)
````
**EN:** This section defines `vllm_per_token_quant_fp8` and implements the core logic associated with vllm per token quantization FP8.
**CN:** 该部分定义 `vllm_per_token_quant_fp8`，并实现与 vllm per token quantization FP8 相关的核心逻辑。

### Lines 72-79: `sglang_per_token_quant_fp8` definition
````python
def sglang_per_token_quant_fp8(
    input: torch.Tensor,
) -> Tuple[torch.Tensor, torch.Tensor]:
    scale = torch.zeros(input.size(0), device=input.device, dtype=torch.float32)
    output = torch.empty_like(input, device=input.device, dtype=fp8_type_)
    sgl_per_token_quant_fp8(input, output, scale)

    return output, scale
````
**EN:** This section defines `sglang_per_token_quant_fp8` and implements the core logic associated with sglang per token quantization FP8.
**CN:** 该部分定义 `sglang_per_token_quant_fp8`，并实现与 sglang per token quantization FP8 相关的核心逻辑。

### Lines 82-153: `calculate_diff` definition
````python
def calculate_diff(batch_size: int, seq_len: int, hidden_dim: int):
    """Compare Torch reference, VLLM, and SGLang implementations."""
    device = torch.device("cuda")
    x = torch.rand(
        (batch_size * seq_len, hidden_dim), dtype=torch.float16, device=device
    )

    # Get all three implementations
    torch_out, torch_scale = torch_per_token_quant_fp8(x)
    vllm_out, vllm_scale = vllm_per_token_quant_fp8(x)
    sglang_out, sglang_scale = sglang_per_token_quant_fp8(x)

    if not VLLM_AVAILABLE:
        print("⚠️ vLLM not available, skipping vLLM comparison")
        # Only compare Torch vs SGLang
        torch_sglang_scale_diff = torch.abs(torch_scale - sglang_scale).mean().item()
        torch_sglang_out_diff = (
            torch.abs(torch_out.float() - sglang_out.float()).mean().item()
        )
        print(f"Scale difference (Torch vs SGLang): {torch_sglang_scale_diff:.8f}")
        print(f"Output difference (Torch vs SGLang): {torch_sglang_out_diff:.8f}")
        return

    print(f"\n=== Comparison for hidden_dim={hidden_dim} ===")

    # Compare scales
    torch_vllm_scale_diff = torch.abs(torch_scale - vllm_scale).mean().item()
    torch_sglang_scale_diff = torch.abs(torch_scale - sglang_scale).mean().item()
    vllm_sglang_scale_diff = torch.abs(vllm_scale - sglang_scale).mean().item()

    print(f"Scale differences:")
    print(f"  Torch vs VLLM:   {torch_vllm_scale_diff:.8f}")
    print(f"  Torch vs SGLang: {torch_sglang_scale_diff:.8f}")
    print(f"  VLLM vs SGLang:  {vllm_sglang_scale_diff:.8f}")

    # Compare outputs
    torch_vllm_out_diff = torch.abs(torch_out.float() - vllm_out.float()).mean().item()
    torch_sglang_out_diff = (
        torch.abs(torch_out.float() - sglang_out.float()).mean().item()
    )
    vllm_sglang_out_diff = (
        torch.abs(vllm_out.float() - sglang_out.float()).mean().item()
    )

    print(f"Output differences:")
    print(f"  Torch vs VLLM:   {torch_vllm_out_diff:.8f}")
    print(f"  Torch vs SGLang: {torch_sglang_out_diff:.8f}")
    print(f"  VLLM vs SGLang:  {vllm_sglang_out_diff:.8f}")

    # Check tolerances
    rtol, atol = 1e-3, 1e-5

    torch_vllm_match = torch.allclose(
        torch_out.float(), vllm_out.float(), rtol=rtol, atol=atol
    ) and torch.allclose(torch_scale, vllm_scale, rtol=rtol, atol=atol)
    torch_sglang_match = torch.allclose(
        torch_out.float(), sglang_out.float(), rtol=rtol, atol=atol
    ) and torch.allclose(torch_scale, sglang_scale, rtol=rtol, atol=atol)

    if hidden_dim == 1368:
        rtol = 1e-2
        # we found vllm sglang has diff when hidden dim is not dividable by 16
        # and we believe SGLang is closer to Torch implementation

    vllm_sglang_match = torch.allclose(
        vllm_out.float(), sglang_out.float(), rtol=rtol, atol=atol
    ) and torch.allclose(vllm_scale, sglang_scale, rtol=rtol, atol=atol)

    print(f"Matches (rtol={rtol}, atol={atol}):")
    print(f"  Torch vs VLLM:   {'✅' if torch_vllm_match else '❌'}")
    print(f"  Torch vs SGLang: {'✅' if torch_sglang_match else '❌'}")
    print(f"  VLLM vs SGLang:  {'✅' if vllm_sglang_match else '❌'}")
````
**EN:** This section defines `calculate_diff`. It calculates the `diff` path used by the module. Docstring summary: Compare Torch reference, VLLM, and SGLang implementations.
**CN:** 该部分定义 `calculate_diff`。它负责计算模块中与 `diff` 相关的处理路径。 文档字符串摘要：Compare Torch reference, VLLM, and SGLang implementations.

### Lines 154-156: Comments and local context
````python


# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 157-164: Conditional logic
````python
if IS_CI:
    batch_size_range = [16]  # Single batch size for CI
    seq_len_range = [64]  # Single sequence length for CI
    hidden_dim_range = [2048]  # Single hidden dimension for CI
else:
    batch_size_range = [16, 32, 64, 128]
    seq_len_range = [64, 128, 256, 512, 1024, 2048, 4096]
    hidden_dim_range = [1368, 2048, 4096]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 166-166: Constants and configuration
````python
configs = list(itertools.product(batch_size_range, seq_len_range, hidden_dim_range))
````
**EN:** This block defines shared constants or configuration values such as `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `configs`），供后续函数或控制流程复用。

### Lines 169-211: `benchmark_quantization` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size", "seq_len", "hidden_dim"],
        x_vals=configs,
        line_arg="provider",
        line_vals=(
            ["torch", "vllm", "sglang"] if VLLM_AVAILABLE else ["torch", "sglang"]
        ),
        line_names=(
            ["Torch Reference", "VLLM", "SGL Kernel"]
            if VLLM_AVAILABLE
            else ["Torch Reference", "SGL Kernel"]
        ),
        styles=(
            [("red", "-"), ("blue", "-"), ("green", "-")]
            if VLLM_AVAILABLE
            else [("red", "-"), ("green", "-")]
        ),
        ylabel="us",
        plot_name="per-token-dynamic-quant-fp8-performance",
        args={},
    )
)
def benchmark_quantization(batch_size, seq_len, hidden_dim, provider):
    dtype = torch.float16
    device = torch.device("cuda")

    x = torch.randn(batch_size * seq_len, hidden_dim, device=device, dtype=dtype)

    quantiles = [0.5, 0.2, 0.8]

    if provider == "torch":
        fn = lambda: torch_per_token_quant_fp8(x.clone())
    elif provider == "vllm":
        if not VLLM_AVAILABLE:
            return (0, 0, 0)
        fn = lambda: vllm_per_token_quant_fp8(x.clone())
    elif provider == "sglang":
        fn = lambda: sglang_per_token_quant_fp8(x.clone())

    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(fn, quantiles=quantiles)

    return 1000 * ms, 1000 * max_ms, 1000 * min_ms
````
**EN:** This section defines `benchmark_quantization` and implements the core logic associated with benchmark quantization. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark_quantization`，并实现与 benchmark quantization 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 214-228: Command-line entry point
````python
if __name__ == "__main__":
    # Test various hidden dimensions for correctness - simplified for CI
    if IS_CI:
        test_dims = [2048]  # Single dimension for CI
        batch_size, seq_len = 4, 64  # Smaller values for CI
    else:
        test_dims = [1368, 2048, 4096]
        batch_size, seq_len = 4, 4096

    for dim in test_dims:
        calculate_diff(batch_size=batch_size, seq_len=seq_len, hidden_dim=dim)

    print("\n" + "=" * 60)
    print("Starting performance benchmark...")
    benchmark_quantization.run(print_data=True)
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `torch_per_token_quant_fp8`, `vllm_per_token_quant_fp8`, `sglang_per_token_quant_fp8`, `calculate_diff`, `benchmark_quantization`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.srt.utils`, `sglang.utils`
- **External / 外部**: `itertools`, `os`, `torch`, `triton`, `triton.testing`, `typing`, `vllm`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
