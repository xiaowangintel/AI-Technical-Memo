# bench_per_tensor_quant_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_per_tensor_quant_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from typing import Optional, Tuple". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from typing import Optional, Tuple”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: Imports and module setup
```python
from typing import Optional, Tuple

import torch
import triton
import triton.testing

from sglang.jit_kernel.benchmark.utils import get_benchmark_range, run_benchmark
from sglang.jit_kernel.per_tensor_quant_fp8 import per_tensor_quant_fp8
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=5, suite="base-b-kernel-benchmark-1-gpu-large")

try:
    from vllm import _custom_ops as ops

    VLLM_AVAILABLE = True
except ImportError:
    ops = None
    VLLM_AVAILABLE = False

try:
    from sglang.srt.utils import is_hip

    _is_hip = is_hip()
except ImportError:
    _is_hip = False

fp8_type_ = torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 31-39: Function `vllm_scaled_fp8_quant`
```python
def vllm_scaled_fp8_quant(
    input: torch.Tensor,
    scale: Optional[torch.Tensor] = None,
) -> Tuple[torch.Tensor, torch.Tensor]:
    if not VLLM_AVAILABLE:
        return sglang_scaled_fp8_quant(input, scale)
    return ops.scaled_fp8_quant(input, scale)
```
**EN:** This block defines `vllm_scaled_fp8_quant`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `vllm_scaled_fp8_quant`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 40-54: Function `sglang_scaled_fp8_quant`
```python
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
```
**EN:** This block defines `sglang_scaled_fp8_quant`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sglang_scaled_fp8_quant`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 55-101: Function `calculate_diff`
```python
def calculate_diff(batch_size: int, seq_len: int):
    device = torch.device("cuda")
    x = torch.rand((batch_size, seq_len), dtype=torch.bfloat16, device=device)

    if not VLLM_AVAILABLE:
        print("vLLM not available, skipping comparison")
        return

    vllm_out, vllm_scale = vllm_scaled_fp8_quant(x)
    sglang_out, sglang_scale = sglang_scaled_fp8_quant(x)

    vllm_out = vllm_out.to(torch.float32)
    sglang_out = sglang_out.to(torch.float32)

    triton.testing.assert_close(vllm_out, sglang_out, rtol=1e-3, atol=1e-3)
    triton.testing.assert_close(vllm_scale, sglang_scale, rtol=1e-3, atol=1e-3)


# Benchmark configuration
element_range = get_benchmark_range(
    full_range=[2**n for n in range(10, 20)],
    ci_range=[16384],
)

if VLLM_AVAILABLE:
    line_vals = ["vllm", "sglang"]
    line_names = ["VLLM", "SGL Kernel"]
    styles = [("blue", "-"), ("green", "-")]
else:
    line_vals = ["sglang"]
    line_names = ["SGL Kernel"]
    styles = [("green", "-")]


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["element_count"],
        x_vals=element_range,
        line_arg="provider",
        line_vals=line_vals,
        line_names=line_names,
        styles=styles,
        ylabel="us",
        plot_name="per-tensor-quant-fp8-performance",
        args={},
    )
)
```
**EN:** This block defines `calculate_diff`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `calculate_diff`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 102-120: Function `benchmark`
```python
def benchmark(element_count, provider):
    dtype = torch.float16
    device = torch.device("cuda")

    x = torch.randn(element_count, 4096, device=device, dtype=dtype)

    if provider == "vllm":
        fn = lambda: vllm_scaled_fp8_quant(x.clone())
    elif provider == "sglang":
        fn = lambda: sglang_scaled_fp8_quant(x.clone())
    else:
        raise ValueError(f"Unknown provider: {provider}")

    return run_benchmark(fn)


if __name__ == "__main__":
    calculate_diff(batch_size=4, seq_len=4096)
    benchmark.run(print_data=True)
```
**EN:** This block defines `benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `typing -> Optional`
- `torch`
- `triton`
- `triton.testing`
- `sglang.jit_kernel.benchmark.utils -> get_benchmark_range`
- `sglang.jit_kernel.per_tensor_quant_fp8 -> per_tensor_quant_fp8`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `vllm -> _custom_ops`
- `sglang.srt.utils -> is_hip`
