# bench_fused_qknorm_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_fused_qknorm_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-89: Imports and module setup
```python
"""
Benchmark: fused_qknorm_rope JIT vs AOT (sgl_kernel)

Measures throughput (µs) for fused_qk_norm_rope across typical
LLM configurations (head_dim × num_heads × num_tokens).

Run:
    python python/sglang/jit_kernel/benchmark/bench_fused_qknorm_rope.py
"""

import itertools

import torch
import triton
import triton.testing

from sglang.jit_kernel.benchmark.utils import get_benchmark_range, run_benchmark
from sglang.jit_kernel.fused_qknorm_rope import (
    fused_qk_norm_rope as fused_qk_norm_rope_jit,
)
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=6, suite="base-b-kernel-benchmark-1-gpu-large")

try:
    from sgl_kernel import fused_qk_norm_rope as fused_qk_norm_rope_aot

    AOT_AVAILABLE = True
except ImportError:
    fused_qk_norm_rope_aot = None
    AOT_AVAILABLE = False

# ---------------------------------------------------------------------------
# Benchmark configuration
# ---------------------------------------------------------------------------

NUM_TOKENS_RANGE = get_benchmark_range(
    full_range=[1, 64, 256, 1024, 4096],
    ci_range=[64, 512],
)

# (head_dim, num_heads_q, num_heads_k, num_heads_v) — typical MoE/dense configs
MODEL_CONFIGS = get_benchmark_range(
    full_range=[
        (64, 32, 8, 8),  # small
        (128, 32, 8, 8),  # typical (e.g. Qwen3-8B)
        (256, 16, 4, 4),  # large head_dim
    ],
    ci_range=[(128, 32, 8, 8)],
)

# Real production shapes (self-attention; num_heads_k == num_heads_v == num_heads_q).
# Format: (name, num_tokens, num_heads_q, num_heads_k, num_heads_v, head_dim, rotary_dim)
PRODUCTION_SHAPES = [
    ("flux_1024", 4096, 24, 24, 24, 128, 128),
    ("qwen_image_1024", 4096, 32, 32, 32, 128, 128),
    ("qwen_image_partial", 4096, 32, 32, 32, 128, 64),
    ("zimage_1024", 4096, 30, 30, 30, 128, 128),
    ("batch2_medium", 4096, 24, 24, 24, 128, 128),  # B=2, T=2048
]

LINE_VALS = ["jit", "aot"] if AOT_AVAILABLE else ["jit"]
LINE_NAMES = ["JIT (new)", "AOT sgl_kernel"] if AOT_AVAILABLE else ["JIT (new)"]
STYLES = [("blue", "--"), ("orange", "-")] if AOT_AVAILABLE else [("blue", "--")]


# ---------------------------------------------------------------------------
# Benchmark: fused_qk_norm_rope (interleave style, no YaRN)
# ---------------------------------------------------------------------------


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["num_tokens", "head_dim", "num_heads_q", "num_heads_k", "num_heads_v"],
        x_vals=[
            (nt, hd, nq, nk, nv)
            for nt, (hd, nq, nk, nv) in itertools.product(
                NUM_TOKENS_RANGE, MODEL_CONFIGS
            )
        ],
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 90-140: Function `bench_fused_qknorm_rope`
```python
def bench_fused_qknorm_rope(
    num_tokens: int,
    head_dim: int,
    num_heads_q: int,
    num_heads_k: int,
    num_heads_v: int,
    provider: str,
):
    device = "cuda"
    total_heads = num_heads_q + num_heads_k + num_heads_v

    qkv = torch.randn(
        (num_tokens, total_heads * head_dim), dtype=torch.bfloat16, device=device
    )
    q_weight = torch.ones(head_dim, dtype=torch.bfloat16, device=device)
    k_weight = torch.ones(head_dim, dtype=torch.bfloat16, device=device)
    position_ids = torch.arange(num_tokens, dtype=torch.int32, device=device)

    common_kwargs = dict(
        num_heads_q=num_heads_q,
        num_heads_k=num_heads_k,
        num_heads_v=num_heads_v,
        head_dim=head_dim,
        eps=1e-5,
        q_weight=q_weight,
        k_weight=k_weight,
        base=10000.0,
        is_neox=False,
        position_ids=position_ids,
        factor=1.0,
        low=1.0,
        high=32.0,
        attention_factor=1.0,
        rotary_dim=head_dim,
    )

    if provider == "jit":
        fn = lambda: fused_qk_norm_rope_jit(qkv.clone(), **common_kwargs)
    elif provider == "aot":
        fn = lambda: fused_qk_norm_rope_aot(qkv.clone(), **common_kwargs)
    else:
        raise ValueError(f"Unknown provider: {provider}")

    return run_benchmark(fn)


# ---------------------------------------------------------------------------
# Benchmark: fused_qk_norm_rope — real production shapes (with speedup column)
# ---------------------------------------------------------------------------
```
**EN:** This block defines `bench_fused_qknorm_rope`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `bench_fused_qknorm_rope`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 141-209: Function `bench_fused_qknorm_rope_production`
```python
def bench_fused_qknorm_rope_production():
    device = "cuda"
    header = f"{'name':<22} {'tokens':>6} {'nq':>4} {'nk':>4} {'nv':>4} {'hd':>4} {'rdim':>5}  {'JIT(us)':>9}  {'AOT(us)':>9}  {'speedup':>8}"
    sep = "-" * len(header)
    print("\nfused-qknorm-rope-production-shapes:")
    print(sep)
    print(header)
    print(sep)

    for (
        name,
        num_tokens,
        num_heads_q,
        num_heads_k,
        num_heads_v,
        head_dim,
        rotary_dim,
    ) in PRODUCTION_SHAPES:
        total_heads = num_heads_q + num_heads_k + num_heads_v
        qkv = torch.randn(
            (num_tokens, total_heads * head_dim), dtype=torch.bfloat16, device=device
        )
        q_weight = torch.ones(head_dim, dtype=torch.bfloat16, device=device)
        k_weight = torch.ones(head_dim, dtype=torch.bfloat16, device=device)
        position_ids = torch.arange(num_tokens, dtype=torch.int32, device=device)

        common_kwargs = dict(
            num_heads_q=num_heads_q,
            num_heads_k=num_heads_k,
            num_heads_v=num_heads_v,
            head_dim=head_dim,
            eps=1e-5,
            q_weight=q_weight,
            k_weight=k_weight,
            base=10000.0,
            is_neox=False,
            position_ids=position_ids,
            factor=1.0,
            low=1.0,
            high=32.0,
            attention_factor=1.0,
            rotary_dim=rotary_dim,
        )

        jit_us, _, _ = run_benchmark(
            lambda: fused_qk_norm_rope_jit(qkv.clone(), **common_kwargs)
        )
        if AOT_AVAILABLE:
            aot_us, _, _ = run_benchmark(
                lambda: fused_qk_norm_rope_aot(qkv.clone(), **common_kwargs)
            )
            speedup = f"{aot_us / jit_us:.2f}x"
            aot_str = f"{aot_us:9.3f}"
        else:
            aot_str = f"{'N/A':>9}"
            speedup = "N/A"

        print(
            f"{name:<22} {num_tokens:>6} {num_heads_q:>4} {num_heads_k:>4} {num_heads_v:>4}"
            f" {head_dim:>4} {rotary_dim:>5}  {jit_us:9.3f}  {aot_str}  {speedup:>8}"
        )
    print(sep)


# ---------------------------------------------------------------------------
# Quick correctness diff
# ---------------------------------------------------------------------------
```
**EN:** This block defines `bench_fused_qknorm_rope_production`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `bench_fused_qknorm_rope_production`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 210-267: Function `calculate_diff`
```python
def calculate_diff():
    if not AOT_AVAILABLE:
        print("sgl_kernel not available — skipping AOT diff check")
        return

    device = "cuda"
    print("Correctness diff (JIT vs AOT):")

    for head_dim, is_neox in [(64, False), (128, False), (128, True), (256, False)]:
        num_tokens = 32
        num_heads_q, num_heads_k, num_heads_v = 4, 2, 2
        total_heads = num_heads_q + num_heads_k + num_heads_v

        qkv = torch.randn(
            (num_tokens, total_heads * head_dim), dtype=torch.bfloat16, device=device
        )
        q_weight = torch.ones(head_dim, dtype=torch.bfloat16, device=device)
        k_weight = torch.ones(head_dim, dtype=torch.bfloat16, device=device)
        position_ids = torch.arange(num_tokens, dtype=torch.int32, device=device)

        common = dict(
            num_heads_q=num_heads_q,
            num_heads_k=num_heads_k,
            num_heads_v=num_heads_v,
            head_dim=head_dim,
            eps=1e-5,
            q_weight=q_weight,
            k_weight=k_weight,
            base=10000.0,
            is_neox=is_neox,
            position_ids=position_ids,
            factor=1.0,
            low=1.0,
            high=32.0,
            attention_factor=1.0,
            rotary_dim=head_dim,
        )

        qkv_jit = qkv.clone()
        fused_qk_norm_rope_jit(qkv_jit, **common)
        qkv_aot = qkv.clone()
        fused_qk_norm_rope_aot(qkv_aot, **common)

        match = torch.allclose(qkv_jit.float(), qkv_aot.float(), atol=1e-2, rtol=1e-2)
        status = "OK" if match else "MISMATCH"
        max_err = (qkv_jit.float() - qkv_aot.float()).abs().max().item()
        print(
            f"  head_dim={head_dim:3d} is_neox={str(is_neox):5s}  "
            f"max_err={max_err:.2e}  [{status}]"
        )


if __name__ == "__main__":
    calculate_diff()
    print()
    bench_fused_qknorm_rope.run(print_data=True)
    print()
    bench_fused_qknorm_rope_production()
```
**EN:** This block defines `calculate_diff`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `calculate_diff`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `itertools`
- `torch`
- `triton`
- `triton.testing`
- `sglang.jit_kernel.benchmark.utils -> get_benchmark_range`
- `sglang.jit_kernel.fused_qknorm_rope -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sgl_kernel -> fused_qk_norm_rope`
