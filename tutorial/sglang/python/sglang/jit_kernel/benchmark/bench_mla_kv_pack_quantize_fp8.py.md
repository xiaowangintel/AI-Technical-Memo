# bench_mla_kv_pack_quantize_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_mla_kv_pack_quantize_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"Bench the hybrid ``mla_kv_pack_quantize_fp8`` against an inlined naive Triton baseline."""". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""Bench the hybrid ``mla_kv_pack_quantize_fp8`` against an inlined naive Triton baseline."""”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports and module setup
```python
"""Bench the hybrid ``mla_kv_pack_quantize_fp8`` against an inlined naive Triton baseline."""

import itertools
from typing import Tuple

import torch
import triton
import triton.language as tl
import triton.testing

from sglang.jit_kernel.benchmark.utils import (
    DEFAULT_DEVICE,
    DEFAULT_DTYPE,
    DEFAULT_QUANTILES,
    get_benchmark_range,
)
from sglang.jit_kernel.mla_kv_pack_quantize_fp8 import (
    mla_kv_pack_quantize_fp8 as hybrid_pack,
)
from sglang.jit_kernel.utils import is_arch_support_pdl
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=15, suite="base-b-kernel-benchmark-1-gpu-large")


@triton.jit
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 27-88: Function `_triton_mla_kv_pack_quantize_fp8_kernel`
```python
def _triton_mla_kv_pack_quantize_fp8_kernel(
    k_nope_ptr,
    k_pe_ptr,
    v_ptr,
    k_out_ptr,
    v_out_ptr,
    k_scale_inv,
    v_scale_inv,
    s_total,
    k_nope_stride_t,
    k_nope_stride_h,
    k_pe_stride_t,
    v_stride_t,
    v_stride_h,
    k_out_stride_t,
    k_out_stride_h,
    v_out_stride_t,
    v_out_stride_h,
    QK_NOPE: tl.constexpr,
    QK_ROPE: tl.constexpr,
    V_HEAD: tl.constexpr,
    FP8_DTYPE: tl.constexpr,
    BLOCK_S: tl.constexpr,
    ENABLE_PDL: tl.constexpr,
):
    pid_s = tl.program_id(0)
    pid_h = tl.program_id(1)
    t_idx = pid_s * BLOCK_S + tl.arange(0, BLOCK_S)
    t_mask = t_idx < s_total
    nope_idx = tl.arange(0, QK_NOPE)
    rope_idx = tl.arange(0, QK_ROPE)
    v_idx = tl.arange(0, V_HEAD)
    if ENABLE_PDL:
        tl.extra.cuda.gdc_wait()
    nope_off = (
        t_idx[:, None] * k_nope_stride_t + pid_h * k_nope_stride_h + nope_idx[None, :]
    )
    k_nope = tl.load(k_nope_ptr + nope_off, mask=t_mask[:, None])
    pe_off = t_idx[:, None] * k_pe_stride_t + rope_idx[None, :]
    k_pe = tl.load(k_pe_ptr + pe_off, mask=t_mask[:, None])
    v_off = t_idx[:, None] * v_stride_t + pid_h * v_stride_h + v_idx[None, :]
    v = tl.load(v_ptr + v_off, mask=t_mask[:, None])
    k_nope_fp8 = (k_nope.to(tl.float32) * k_scale_inv).to(FP8_DTYPE)
    k_pe_fp8 = (k_pe.to(tl.float32) * k_scale_inv).to(FP8_DTYPE)
    v_fp8 = (v.to(tl.float32) * v_scale_inv).to(FP8_DTYPE)
    k_out_base = t_idx[:, None] * k_out_stride_t + pid_h * k_out_stride_h
    tl.store(
        k_out_ptr + k_out_base + nope_idx[None, :], k_nope_fp8, mask=t_mask[:, None]
    )
    tl.store(
        k_out_ptr + k_out_base + QK_NOPE + rope_idx[None, :],
        k_pe_fp8,
        mask=t_mask[:, None],
    )
    v_out_off = (
        t_idx[:, None] * v_out_stride_t + pid_h * v_out_stride_h + v_idx[None, :]
    )
    tl.store(v_out_ptr + v_out_off, v_fp8, mask=t_mask[:, None])
    if ENABLE_PDL:
        tl.extra.cuda.gdc_launch_dependents()
```
**EN:** This block defines `_triton_mla_kv_pack_quantize_fp8_kernel`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_triton_mla_kv_pack_quantize_fp8_kernel`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 89-162: Function `_triton_pack`
```python
def _triton_pack(k_nope, k_pe, v, k_out, v_out):
    s, num_heads, qk_nope = k_nope.shape
    qk_rope = k_pe.shape[-1]
    v_head = v.shape[-1]
    k_pe_2d = k_pe.squeeze(1) if k_pe.dim() == 3 else k_pe
    enable_pdl = is_arch_support_pdl()
    if s < 512:
        block_s, num_warps, num_stages = 1, 1, 2
    elif s < 2048:
        block_s, num_warps, num_stages = 4, 2, 3
    else:
        block_s, num_warps, num_stages = 16, 4, 3
    extra = {"launch_pdl": True} if enable_pdl else {}
    grid = (triton.cdiv(s, block_s), num_heads)
    _triton_mla_kv_pack_quantize_fp8_kernel[grid](
        k_nope,
        k_pe_2d,
        v,
        k_out,
        v_out,
        1.0,
        1.0,
        s,
        k_nope.stride(0),
        k_nope.stride(1),
        k_pe_2d.stride(0),
        v.stride(0),
        v.stride(1),
        k_out.stride(0),
        k_out.stride(1),
        v_out.stride(0),
        v_out.stride(1),
        QK_NOPE=qk_nope,
        QK_ROPE=qk_rope,
        V_HEAD=v_head,
        FP8_DTYPE=tl.float8e4nv,
        BLOCK_S=block_s,
        ENABLE_PDL=enable_pdl,
        num_warps=num_warps,
        num_stages=num_stages,
        **extra,
    )


QK_NOPE = 128
QK_ROPE = 64
V_HEAD = 128
NUM_HEADS = 32
NUM_LAYERS = 8

BS_RANGE = get_benchmark_range(
    full_range=[1, 4, 16, 64, 256, 1024, 4096, 8192, 16384],
    ci_range=[1, 64, 1024, 4096, 16384],
)

LINE_VALS = ["hybrid", "triton"]
LINE_NAMES = ["hybrid (v0+v1_flat)", "naive Triton"]
STYLES = [("green", "-"), ("red", "--")]
CONFIGS = list(itertools.product(BS_RANGE))


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size"],
        x_vals=CONFIGS,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="mla-kv-pack-quantize-fp8-performance",
        args={},
    )
)
```
**EN:** This block defines `_triton_pack`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_triton_pack`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 163-214: Function `benchmark`
```python
def benchmark(batch_size: int, provider: str) -> Tuple[float, float, float]:
    k_nope = torch.randn(
        (NUM_LAYERS, batch_size, NUM_HEADS, QK_NOPE),
        dtype=DEFAULT_DTYPE,
        device=DEFAULT_DEVICE,
    )
    k_pe = torch.randn(
        (NUM_LAYERS, batch_size, 1, QK_ROPE),
        dtype=DEFAULT_DTYPE,
        device=DEFAULT_DEVICE,
    )
    v = torch.randn(
        (NUM_LAYERS, batch_size, NUM_HEADS, V_HEAD),
        dtype=DEFAULT_DTYPE,
        device=DEFAULT_DEVICE,
    )
    k_out = torch.empty(
        (NUM_LAYERS, batch_size, NUM_HEADS, QK_NOPE + QK_ROPE),
        dtype=torch.float8_e4m3fn,
        device=DEFAULT_DEVICE,
    )
    v_out = torch.empty(
        (NUM_LAYERS, batch_size, NUM_HEADS, V_HEAD),
        dtype=torch.float8_e4m3fn,
        device=DEFAULT_DEVICE,
    )
    torch.cuda.synchronize()

    if provider == "hybrid":

        def fn():
            for i in range(NUM_LAYERS):
                hybrid_pack(k_nope[i], k_pe[i], v[i], k_out=k_out[i], v_out=v_out[i])

    else:

        def fn():
            for i in range(NUM_LAYERS):
                _triton_pack(k_nope[i], k_pe[i], v[i], k_out[i], v_out[i])

    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
        fn, quantiles=DEFAULT_QUANTILES
    )
    return (
        1000 * ms / NUM_LAYERS,
        1000 * max_ms / NUM_LAYERS,
        1000 * min_ms / NUM_LAYERS,
    )


if __name__ == "__main__":
    benchmark.run(print_data=True)
```
**EN:** This block defines `benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `itertools`
- `typing -> Tuple`
- `torch`
- `triton`
- `triton.language as tl`
- `triton.testing`
- `sglang.jit_kernel.benchmark.utils -> (`
- `sglang.jit_kernel.mla_kv_pack_quantize_fp8 -> (`
- `sglang.jit_kernel.utils -> is_arch_support_pdl`
- `sglang.test.ci.ci_register -> register_cuda_ci`
