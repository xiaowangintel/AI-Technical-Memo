# bench_fused_norm_scale_shift.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/diffusion/bench_fused_norm_scale_shift.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "Benchmarks SGLang fused layernorm/rmsnorm scale shift kernels". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“Benchmarks SGLang fused layernorm/rmsnorm scale shift kernels”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44: Imports and module setup
```python
# Benchmarks SGLang fused layernorm/rmsnorm scale shift kernels
# 1. fused_norm_scale_shift
# 2. fused_scale_residual_norm_scale_shift
import itertools
from typing import Tuple

import torch
import triton
import triton.testing

from sglang.jit_kernel.benchmark.utils import run_benchmark_no_cudagraph
from sglang.multimodal_gen.runtime.layers.layernorm import (
    LayerNormScaleShift,
    RMSNormScaleShift,
    ScaleResidualLayerNormScaleShift,
    ScaleResidualRMSNormScaleShift,
)
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.utils import is_in_ci

register_cuda_ci(
    est_time=17,
    suite="base-b-kernel-benchmark-1-gpu-large",
    disabled="Temporarily skipped to unblock flashinfer upgrade. Ref: https://github.com/sgl-project/sglang/actions/runs/23735552939/job/69139238979?pr=21422",
)

if is_in_ci():
    B_RANGE, S_RANGE, D_RANGE = [1], [128], [1024]
else:
    B_RANGE, S_RANGE, D_RANGE = [1], [128, 1024, 4096], [1024, 3072, 4096]

NORM_TYPE_RANGE = ["layer", "rms"]
AFFINE_RANGE = [True, False]
DTYPE = torch.bfloat16
DEVICE = "cuda"
EPS = 1e-5
LINE_VALS = ["native", "cuda"]
LINE_NAMES = ["SGLang Native", "SGLang Fused"]
STYLES = [("red", "-"), ("blue", "--")]
config = list(
    itertools.product(B_RANGE, S_RANGE, D_RANGE, NORM_TYPE_RANGE, AFFINE_RANGE)
)
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 45-72: Function `preprocess_layer`
```python
def preprocess_layer(layer, affine: bool, D: int, DTYPE: torch.dtype):
    if affine:
        weight = torch.randn(D, dtype=DTYPE, device=DEVICE)
        bias = torch.randn(D, dtype=DTYPE, device=DEVICE)
        with torch.no_grad():
            layer.norm.weight.copy_(weight)
            if hasattr(layer.norm, "bias"):
                layer.norm.bias.copy_(bias)
    layer.requires_grad_(False)
    return layer.to(DEVICE)


# ============================================================================
# Benchmark 1: fused_norm_scale_shift
# ============================================================================
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["B", "S", "D", "norm_type", "affine"],
        x_vals=config,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="fused_norm_scale_shift",
        args={},
    )
)
```
**EN:** This block defines `preprocess_layer`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `preprocess_layer`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 73-107: Function `bench_fused_norm_scale_shift`
```python
def bench_fused_norm_scale_shift(
    B: int, S: int, D: int, norm_type, affine: bool, provider: str
) -> Tuple[float, float, float]:
    x = torch.randn(B, S, D, dtype=DTYPE, device=DEVICE)
    scale = torch.randn(B, S, D, dtype=DTYPE, device=DEVICE)
    shift = torch.randn(B, S, D, dtype=DTYPE, device=DEVICE)
    if norm_type == "layer":
        layer = LayerNormScaleShift(D, EPS, affine, dtype=DTYPE)
    else:
        layer = RMSNormScaleShift(D, EPS, affine, dtype=DTYPE)
    layer = preprocess_layer(layer, affine, D, DTYPE)
    if provider == "native":
        fn = lambda: layer.forward_native(x, shift, scale)
    else:
        fn = lambda: layer.forward_cuda(x, shift, scale)

    return run_benchmark_no_cudagraph(fn)


# ============================================================================
# Benchmark 2: fused_scale_residual_norm_scale_shift
# ============================================================================
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["B", "S", "D", "norm_type", "affine"],
        x_vals=config,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="fused_scale_residual_norm_scale_shift",
        args={},
    )
)
```
**EN:** This block defines `bench_fused_norm_scale_shift`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `bench_fused_norm_scale_shift`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 108-138: Function `bench_fused_scale_residual_norm_scale_shift`
```python
def bench_fused_scale_residual_norm_scale_shift(
    B: int, S: int, D: int, norm_type, affine: bool, provider: str
) -> Tuple[float, float, float]:
    residual = torch.randn(B, S, D, dtype=DTYPE, device=DEVICE)
    x = torch.randn(B, S, D, dtype=DTYPE, device=DEVICE)
    scale = torch.randn(B, S, D, dtype=DTYPE, device=DEVICE)
    shift = torch.randn(B, S, D, dtype=DTYPE, device=DEVICE)
    gate = torch.randn(B, 1, D, dtype=DTYPE, device=DEVICE)
    if norm_type == "layer":
        layer = ScaleResidualLayerNormScaleShift(D, EPS, affine, dtype=DTYPE).to(DEVICE)
    else:
        layer = ScaleResidualRMSNormScaleShift(D, EPS, affine, dtype=DTYPE).to(DEVICE)
    layer = preprocess_layer(layer, affine, D, DTYPE)
    if provider == "native":
        fn = lambda: layer.forward_native(residual, x, gate, shift, scale)
    else:
        fn = lambda: layer.forward_cuda(residual, x, gate, shift, scale)

    return run_benchmark_no_cudagraph(fn)


if __name__ == "__main__":
    print(f"\n{'='*80}")
    print("Benchmark: fused_norm_scale_shift")
    print(f"{'='*80}\n")
    bench_fused_norm_scale_shift.run(print_data=True)

    print(f"\n{'='*80}")
    print("Benchmark: fused_scale_residual_norm_scale_shift")
    print(f"{'='*80}\n")
    bench_fused_scale_residual_norm_scale_shift.run(print_data=True)
```
**EN:** This block defines `bench_fused_scale_residual_norm_scale_shift`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `bench_fused_scale_residual_norm_scale_shift`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `itertools`
- `typing -> Tuple`
- `torch`
- `triton`
- `triton.testing`
- `sglang.jit_kernel.benchmark.utils -> run_benchmark_no_cudagraph`
- `sglang.multimodal_gen.runtime.layers.layernorm -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sglang.utils -> is_in_ci`
