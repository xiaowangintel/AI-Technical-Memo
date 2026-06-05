# bench_nvfp4_blockwise_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_nvfp4_blockwise_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and module setup
```python
from __future__ import annotations

import sys
from typing import Any

import torch
import triton

from sglang.jit_kernel.benchmark.utils import get_benchmark_range, run_benchmark
from sglang.jit_kernel.nvfp4 import (
    cutlass_fp4_group_mm,
    scaled_fp4_experts_quant,
    scaled_fp4_quant,
)
from sglang.srt.utils import is_sm100_supported
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=5, suite="base-b-kernel-benchmark-1-gpu-large")

FLOAT4_E2M1_MAX = 6.0
FLOAT8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max
_NVFP4_SUPPORTED = is_sm100_supported()
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 25-28: Function `_round_up`
```python
def _round_up(x: int, y: int) -> int:
    return ((x + y - 1) // y) * y
```
**EN:** This block defines `_round_up`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_round_up`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 29-35: Function `_expert_offsets`
```python
def _expert_offsets(m_per_expert: list[int], device: torch.device) -> torch.Tensor:
    offsets = [0]
    for m in m_per_expert:
        offsets.append(offsets[-1] + m)
    return torch.tensor(offsets, dtype=torch.int32, device=device)
```
**EN:** This block defines `_expert_offsets`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_expert_offsets`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 36-42: Function `_blockscale_offsets`
```python
def _blockscale_offsets(m_per_expert: list[int], device: torch.device) -> torch.Tensor:
    offsets = [0]
    for m in m_per_expert:
        offsets.append(offsets[-1] + _round_up(m, 128))
    return torch.tensor(offsets, dtype=torch.int32, device=device)
```
**EN:** This block defines `_blockscale_offsets`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_blockscale_offsets`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 43-132: Function `_prepare_case`
```python
def _prepare_case(
    total_tokens: int, n: int, k: int, num_experts: int, dtype: torch.dtype
) -> dict[str, Any]:
    device = torch.device("cuda")
    base = total_tokens // num_experts
    rem = total_tokens % num_experts
    m_per_expert = [base + (1 if i < rem else 0) for i in range(num_experts)]

    expert_offsets_full = _expert_offsets(m_per_expert, device)
    blockscale_offsets_full = _blockscale_offsets(m_per_expert, device)

    a = torch.randn((total_tokens, k), device=device, dtype=dtype) * 0.1
    b = torch.randn((num_experts, n, k), device=device, dtype=dtype) * 0.1

    a_global_scale = torch.empty((num_experts,), device=device, dtype=torch.float32)
    for i in range(num_experts):
        start = int(expert_offsets_full[i].item())
        end = int(expert_offsets_full[i + 1].item())
        a_global_scale[i] = (
            FLOAT8_E4M3_MAX
            * FLOAT4_E2M1_MAX
            / a[start:end].abs().max().to(torch.float32)
        )

    b_global_scale = torch.empty((num_experts,), device=device, dtype=torch.float32)
    for i in range(num_experts):
        b_global_scale[i] = (
            FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / b[i].abs().max().to(torch.float32)
        )

    a_fp4, a_blockscale = scaled_fp4_experts_quant(
        a,
        a_global_scale,
        expert_offsets_full,
        blockscale_offsets_full,
        topk=1,
    )

    b_fp4 = torch.empty((num_experts, n, k // 2), device=device, dtype=torch.uint8)
    b_blockscale = torch.empty(
        (num_experts, _round_up(n, 128), _round_up(k // 16, 4)),
        device=device,
        dtype=torch.float8_e4m3fn,
    )
    for i in range(num_experts):
        b_fp4_i, b_scale_i = scaled_fp4_quant(b[i], b_global_scale[i])
        b_fp4[i].copy_(b_fp4_i)
        b_blockscale[i].copy_(b_scale_i)

    alphas = (1.0 / (a_global_scale * b_global_scale)).to(torch.float32)
    params = {
        "ab_strides": torch.full((num_experts,), k, dtype=torch.int64, device=device),
        "c_strides": torch.full((num_experts,), n, dtype=torch.int64, device=device),
        "problem_sizes": torch.tensor(
            [[m, n, k] for m in m_per_expert], dtype=torch.int32, device=device
        ),
        "expert_offsets": expert_offsets_full[:-1].contiguous(),
        "blockscale_offsets": blockscale_offsets_full[:-1].contiguous(),
        "a_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "b_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "out_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "a_scales_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "b_scales_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "alpha_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "layout_sfa": torch.empty((num_experts, 5), dtype=torch.int64, device=device),
        "layout_sfb": torch.empty((num_experts, 5), dtype=torch.int64, device=device),
    }

    expert_ranges: list[tuple[int, int]] = []
    start = 0
    for m in m_per_expert:
        end = start + m
        expert_ranges.append((start, end))
        start = end

    return {
        "a": a,
        "b": b,
        "a_fp4": a_fp4,
        "b_fp4": b_fp4,
# ...
```
**EN:** This block defines `_prepare_case`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_prepare_case`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 133-145: Function `_torch_ref_group_mm`
```python
def _torch_ref_group_mm(case: dict[str, Any]) -> torch.Tensor:
    a = case["a"]
    b = case["b"]
    dtype = case["dtype"]
    expert_ranges = case["expert_ranges"]
    total_tokens = a.shape[0]
    n = b.shape[1]
    out = torch.empty((total_tokens, n), device=a.device, dtype=dtype)
    for i, (start, end) in enumerate(expert_ranges):
        out[start:end] = torch.matmul(a[start:end], b[i].t())
    return out
```
**EN:** This block defines `_torch_ref_group_mm`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_torch_ref_group_mm`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 146-173: Function `_aot_cutlass_fp4_group_mm`
```python
def _aot_cutlass_fp4_group_mm(case: dict[str, Any]) -> torch.Tensor:
    a_fp4 = case["a_fp4"]
    b_fp4 = case["b_fp4"]
    a_blockscale = case["a_blockscale"]
    b_blockscale = case["b_blockscale"]
    alphas = case["alphas"]
    params = case["params"]
    out_dtype = case["dtype"]

    out = torch.empty(
        (a_fp4.shape[0], b_fp4.shape[1]), device=a_fp4.device, dtype=out_dtype
    )
    torch.ops.sgl_kernel.cutlass_fp4_group_mm.default(
        out,
        a_fp4,
        b_fp4,
        a_blockscale,
        b_blockscale,
        alphas,
        params["ab_strides"],
        params["c_strides"],
        params["problem_sizes"],
        params["expert_offsets"],
        params["blockscale_offsets"],
    )
    return out
```
**EN:** This block defines `_aot_cutlass_fp4_group_mm`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_aot_cutlass_fp4_group_mm`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 174-229: Imports and module setup
```python
def _probe_legacy_aot_group_mm() -> tuple[bool, str]:
    if not torch.cuda.is_available():
        return False, "CUDA is not available."
    if not _NVFP4_SUPPORTED:
        return False, "NVFP4 benchmarks require sm100+ with CUDA 12.8+."
    try:
        import sgl_kernel  # noqa: F401
    except Exception as e:
        return False, f"import sgl_kernel failed: {e}"
    if not hasattr(torch.ops, "sgl_kernel"):
        return False, "torch.ops.sgl_kernel is not registered."
    op = getattr(torch.ops.sgl_kernel, "cutlass_fp4_group_mm", None)
    if op is None or not hasattr(op, "default"):
        return False, "torch.ops.sgl_kernel.cutlass_fp4_group_mm.default is missing."
    try:
        case = _prepare_case(64, 256, 128, 4, torch.bfloat16)
        _aot_cutlass_fp4_group_mm(case)
        torch.cuda.synchronize()
    except Exception as e:
        return False, f"calling AOT grouped_mm op failed: {e}"
    return True, ""


_AOT_GROUP_MM_AVAILABLE, _AOT_GROUP_MM_REASON = _probe_legacy_aot_group_mm()

shape_range = get_benchmark_range(
    full_range=[(128, 256, 128, 4), (256, 512, 128, 8), (512, 512, 256, 8)],
    ci_range=[(128, 256, 128, 4)],
)

line_vals = ["jit"]
line_names = ["JIT NVFP4 MoE GroupMM"]
styles = [("green", "-")]
if _AOT_GROUP_MM_AVAILABLE:
    line_vals.append("aot_sgl_kernel")
    line_names.append("AOT NVFP4 MoE GroupMM")
    styles.append(("orange", "-"))
line_vals.append("torch_ref")
line_names.append("Torch Ref")
styles.append(("blue", "-"))


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["total_tokens", "n", "k", "num_experts"],
        x_vals=shape_range,
        x_log=False,
        line_arg="provider",
        line_vals=line_vals,
        line_names=line_names,
        styles=styles,
        ylabel="us",
        plot_name="nvfp4-blockwise-moe-groupmm-performance",
        args={},
    )
)
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 230-261: Function `benchmark`
```python
def benchmark(total_tokens, n, k, num_experts, provider):
    case = _prepare_case(total_tokens, n, k, num_experts, torch.bfloat16)

    if provider == "jit":
        fn = lambda: cutlass_fp4_group_mm(
            case["a_fp4"],
            case["b_fp4"],
            case["a_blockscale"],
            case["b_blockscale"],
            case["alphas"],
            case["dtype"],
            case["params"],
        )
    elif provider == "aot_sgl_kernel":
        fn = lambda: _aot_cutlass_fp4_group_mm(case)
    elif provider == "torch_ref":
        fn = lambda: _torch_ref_group_mm(case)
    else:
        raise ValueError(f"Unknown provider: {provider}")

    return run_benchmark(fn)


if __name__ == "__main__":
    if not _NVFP4_SUPPORTED:
        print("[skip] NVFP4 blockwise MoE benchmark requires sm100+ with CUDA 12.8+.")
        sys.exit(0)
    if not _AOT_GROUP_MM_AVAILABLE:
        print(
            f"[info] legacy AOT grouped_mm baseline unavailable: {_AOT_GROUP_MM_REASON}"
        )
    benchmark.run(print_data=True)
```
**EN:** This block defines `benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `sys`
- `typing -> Any`
- `torch`
- `triton`
- `sglang.jit_kernel.benchmark.utils -> get_benchmark_range`
- `sglang.jit_kernel.nvfp4 -> (`
- `sglang.srt.utils -> is_sm100_supported`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sgl_kernel`
