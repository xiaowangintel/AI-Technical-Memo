# bench_mxfp4_sm90_kernels.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/layers/moe/bench_mxfp4_sm90_kernels.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module supports the `bench mxfp4 sm90 kernels` workflow in `test/manual/layers/moe`. It organizes scenario-specific logic used by SGLang's manual validation suite. / 该 Python 模块支撑 `test/manual/layers/moe` 中的 `bench mxfp4 sm90 kernels` 流程。它组织了 SGLang 手动验证套件所需的场景化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-50: Constants and scenario settings / 常量与场景配置
```python
"""Benchmark MXFP4 MoE kernels on H100/H200: SGLang Marlin vs FlashInfer cutlass.

Compares per-call latency of:

  * Marlin path  :  ``fused_marlin_moe(...)`` after Marlin weight repack
  * FlashInfer   :  ``cutlass_fused_moe(use_w4_group_scaling=True, ...)``
                    (PR #3084's SM90 mixed-input path)

Both run on the same random MXFP4 weights/scales (semantics differ slightly --
Marlin uses a scalar swiglu clamp + no bias, FlashInfer fuses per-expert
SwiGLU with bias -- so the timing comparison reports kernel cost for
*equivalent compute volume*, not bit-exact numerics).

Run on H100/H200:

    cd /sgl-workspace/sglang_dev3 && \\
    PYTHONPATH=python:/sgl-workspace/flashinfer FLASHINFER_DISABLE_VERSION_CHECK=1 \\
    python python/sglang/test/bench_mxfp4_sm90_kernels.py
"""

from __future__ import annotations

import argparse
from dataclasses import dataclass
from typing import Callable, List, Tuple

import torch
from flashinfer.autotuner import autotune

# ---- FlashInfer ----
from flashinfer.fused_moe import (
    cutlass_fused_moe,
    interleave_moe_scales_for_sm90_mixed_gemm,
    interleave_moe_weights_for_sm90_mixed_gemm,
)
from flashinfer.fused_moe.core import ActivationType

# ---- SGLang Marlin ----
from sglang.jit_kernel.gptq_marlin_repack import gptq_marlin_repack
from sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe import fused_marlin_moe
from sglang.srt.layers.quantization.marlin_utils import (
    marlin_make_workspace,
    marlin_permute_scales,
)
from sglang.srt.layers.quantization.marlin_utils_fp4 import mxfp4_marlin_process_scales

GROUP_SIZE = 32


@dataclass
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. The code also ties this block to evaluation or benchmark execution. Representative call sites include `fused_marlin_moe`, `cutlass_fused_moe` and `scales`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 51-57: Class definition for Shape / 类定义
```python
class Shape:
    tokens: int
    hidden: int
    inter: int
    num_experts: int
    top_k: int
```
**EN:** This range declares `Shape`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 58-62: Helper routines around label / 辅助例程
```python
    def label(self) -> str:
        return (
            f"m={self.tokens:>4} h={self.hidden} i={self.inter} "
            f"E={self.num_experts} k={self.top_k}"
        )
```
**EN:** This range implements helper routine(s) `label` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 63-81: Scenario logic / 场景逻辑
```python


# Sweep tokens at a fixed GPT-OSS-like body (hidden=4096, inter=2048, E=256,
# topk=6 -- matches PR #3084's headline shape so the small-batch numbers stay
# directly comparable). Token range covers decode (4-256) and prefill chunks
# (1024-8192).
_BODY = dict(hidden=4096, inter=2048, num_experts=256, top_k=6)
DEFAULT_SHAPES: List[Shape] = [
    Shape(tokens=4, **_BODY),
    Shape(tokens=16, **_BODY),
    Shape(tokens=64, **_BODY),
    Shape(tokens=256, **_BODY),
    Shape(tokens=1024, **_BODY),
    Shape(tokens=2048, **_BODY),
    Shape(tokens=4096, **_BODY),
    Shape(tokens=8192, **_BODY),
]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `body`, `decode` and `Shape`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 82-106: Helper routines around _make_random_mxfp4 / 辅助例程
```python
def _make_random_mxfp4(shape: Shape, seed: int = 0):
    g = torch.Generator(device="cuda").manual_seed(seed)
    e = shape.num_experts
    n = shape.inter
    k = shape.hidden
    w13 = torch.randint(
        0, 256, (e, 2 * n, k // 2), dtype=torch.uint8, device="cuda", generator=g
    )
    w2 = torch.randint(
        0, 256, (e, k, n // 2), dtype=torch.uint8, device="cuda", generator=g
    )
    # Narrow E8M0 band so dequant magnitudes stay sane.
    w13_s = torch.randint(
        125,
        130,
        (e, 2 * n, k // GROUP_SIZE),
        dtype=torch.uint8,
        device="cuda",
        generator=g,
    )
    w2_s = torch.randint(
        125,
        130,
        (e, k, n // GROUP_SIZE),
        dtype=torch.uint8,
```
**EN:** This range implements helper routine(s) `_make_random_mxfp4` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `Generator`, `manual_seed` and `randint`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 107-122: Scenario logic / 场景逻辑
```python
        device="cuda",
        generator=g,
    )
    w13_b = (
        torch.randn(e, 2 * n, dtype=torch.float32, device="cuda", generator=g).to(
            torch.bfloat16
        )
        * 0.01
    )
    w2_b = (
        torch.randn(e, k, dtype=torch.float32, device="cuda", generator=g).to(
            torch.bfloat16
        )
        * 0.01
    )
    return w13, w2, w13_s, w2_s, w13_b, w2_b
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 125-136: Helper routines around _make_topk / 辅助例程
```python
def _make_topk(shape: Shape, seed: int = 1):
    g = torch.Generator(device="cuda").manual_seed(seed)
    logits = torch.randn(
        shape.tokens,
        shape.num_experts,
        dtype=torch.float32,
        device="cuda",
        generator=g,
    )
    weights, ids = torch.topk(torch.softmax(logits, dim=-1), shape.top_k, dim=-1)
    weights = weights / weights.sum(dim=-1, keepdim=True)
    return logits, weights.to(torch.float32), ids.to(torch.int32)
```
**EN:** This range implements helper routine(s) `_make_topk` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `Generator`, `manual_seed`, `randn` and `topk`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 137-143: Scenario logic / 场景逻辑
```python


# ---------------------------------------------------------------------------
# FlashInfer cutlass path
# ---------------------------------------------------------------------------
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 144-162: Helper routines around build_flashinfer_inputs / 辅助例程
```python
def build_flashinfer_inputs(shape: Shape, w13, w2, w13_s, w2_s, w13_b, w2_b):
    w13_il = interleave_moe_weights_for_sm90_mixed_gemm(w13, "fp4")
    w2_il = interleave_moe_weights_for_sm90_mixed_gemm(w2, "fp4")
    w13_s_il = interleave_moe_scales_for_sm90_mixed_gemm(w13_s, group_size=GROUP_SIZE)
    w2_s_il = interleave_moe_scales_for_sm90_mixed_gemm(w2_s, group_size=GROUP_SIZE)
    e = shape.num_experts
    swiglu_alpha = torch.full((e,), 1.702, dtype=torch.float32, device="cuda")
    swiglu_beta = torch.full((e,), 1.0, dtype=torch.float32, device="cuda")
    swiglu_limit = torch.full((e,), 7.0, dtype=torch.float32, device="cuda")
    return {
        "w13": w13_il,
        "w2": w2_il,
        "quant_scales": [w13_s_il.view(torch.int32), w2_s_il.view(torch.int32)],
        "w13_b": w13_b,
        "w2_b": w2_b,
        "swiglu_alpha": swiglu_alpha,
        "swiglu_beta": swiglu_beta,
        "swiglu_limit": swiglu_limit,
    }
```
**EN:** This range implements helper routine(s) `build_flashinfer_inputs` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `interleave_moe_weights_for_sm90_mixed_gemm`, `interleave_moe_scales_for_sm90_mixed_gemm`, `full` and `view`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 165-189: Helper routines around make_flashinfer_runner / 辅助例程
```python
def make_flashinfer_runner(
    shape: Shape, prep, x, topk_w, topk_i, autotuned: bool, with_bias: bool = True
):
    out = torch.empty(shape.tokens, shape.hidden, dtype=torch.bfloat16, device="cuda")
    fc1_b = prep["w13_b"] if with_bias else None
    fc2_b = prep["w2_b"] if with_bias else None

    def _call():
        cutlass_fused_moe(
            input=x,
            token_selected_experts=topk_i,
            token_final_scales=topk_w,
            fc1_expert_weights=prep["w13"],
            fc2_expert_weights=prep["w2"],
            output_dtype=torch.bfloat16,
            quant_scales=prep["quant_scales"],
            fc1_expert_biases=fc1_b,
            fc2_expert_biases=fc2_b,
            swiglu_alpha=prep["swiglu_alpha"],
            swiglu_beta=prep["swiglu_beta"],
            swiglu_limit=prep["swiglu_limit"],
            use_w4_group_scaling=True,
            activation_type=ActivationType.Swiglu,
            output=out,
        )
```
**EN:** This range implements helper routine(s) `make_flashinfer_runner` and `_call` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `empty` and `cutlass_fused_moe`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 190-196: Scenario logic / 场景逻辑
```python

    if autotuned:
        # Populate FlashInfer's tactic cache once before timing.
        with autotune(True):
            _call()

    return _call
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `autotune` and `_call`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 197-203: Scenario logic / 场景逻辑
```python


# ---------------------------------------------------------------------------
# SGLang Marlin path
# ---------------------------------------------------------------------------
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 204-228: Helper routines around build_marlin_inputs / 辅助例程
```python
def build_marlin_inputs(shape: Shape, w13, w2, w13_s, w2_s):
    """Repack MXFP4 weights into Marlin layout. Mirrors
    ``prepare_moe_mxfp4_layer_for_marlin`` but does not require a layer object."""
    e = shape.num_experts
    n = shape.inter
    k = shape.hidden
    device = w13.device
    perm = torch.empty(0, dtype=torch.int, device=device)

    def _repack(weight, size_n, size_k):
        out_list = []
        for i in range(e):
            qweight = weight[i].view(torch.int32).T.contiguous()
            out_list.append(
                gptq_marlin_repack(
                    b_q_weight=qweight,
                    perm=perm,
                    size_k=size_k,
                    size_n=size_n,
                    num_bits=4,
                )
            )
        return torch.stack(out_list)

    def _scales_for(scales, size_n, size_k):
```
**EN:** This range implements helper routine(s) `build_marlin_inputs`, `_repack` and `_scales_for` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `empty`, `view`, `contiguous` and `append`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 229-252: Scenario logic / 场景逻辑
```python
        out_list = []
        # Reinterpret uint8 E8M0 byte as float8_e8m0fnu, then to bf16 numerical.
        scales_bf16 = scales.view(torch.float8_e8m0fnu).to(torch.bfloat16)
        for i in range(e):
            s = scales_bf16[i].T.contiguous()
            ms = marlin_permute_scales(
                s=s, size_k=size_k, size_n=size_n, group_size=GROUP_SIZE
            )
            out_list.append(mxfp4_marlin_process_scales(ms, input_dtype=torch.bfloat16))
        return torch.stack(out_list)

    w13_marlin = _repack(w13, size_n=2 * n, size_k=k)
    w2_marlin = _repack(w2, size_n=k, size_k=n)
    w13_s_marlin = _scales_for(w13_s, size_n=2 * n, size_k=k)
    w2_s_marlin = _scales_for(w2_s, size_n=k, size_k=n)

    workspace = marlin_make_workspace(device, 4)
    return {
        "w13": w13_marlin,
        "w2": w2_marlin,
        "w13_s": w13_s_marlin,
        "w2_s": w2_s_marlin,
        "workspace": workspace,
    }
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `view`, `to`, `contiguous` and `marlin_permute_scales`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 255-273: Helper routines around make_marlin_runner / 辅助例程
```python
def make_marlin_runner(shape: Shape, prep, x_bf16, router_logits, topk_w, topk_i):
    def _call():
        fused_marlin_moe(
            hidden_states=x_bf16,
            w1=prep["w13"],
            w2=prep["w2"],
            w1_scale=prep["w13_s"],
            w2_scale=prep["w2_s"],
            gating_output=router_logits,
            topk_weights=topk_w,
            topk_ids=topk_i,
            workspace=prep["workspace"],
            num_bits=4,
            is_k_full=True,
            inplace=False,
            clamp_limit=7.0,
        )

    return _call
```
**EN:** This range implements helper routine(s) `make_marlin_runner` and `_call` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `fused_marlin_moe`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 274-280: Scenario logic / 场景逻辑
```python


# ---------------------------------------------------------------------------
# Timing harness
# ---------------------------------------------------------------------------
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 281-295: Helper routines around time_call / 辅助例程
```python
def time_call(fn: Callable, warmup: int = 5, iters: int = 30) -> Tuple[float, float]:
    """Returns (median_ms, min_ms) across ``iters`` calls after ``warmup``."""
    for _ in range(warmup):
        fn()
    torch.cuda.synchronize()

    starts = [torch.cuda.Event(enable_timing=True) for _ in range(iters)]
    ends = [torch.cuda.Event(enable_timing=True) for _ in range(iters)]
    for s, e in zip(starts, ends):
        s.record()
        fn()
        e.record()
    torch.cuda.synchronize()
    times = sorted(s.elapsed_time(e) for s, e in zip(starts, ends))
    return times[len(times) // 2], times[0]
```
**EN:** This range implements helper routine(s) `time_call` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `Returns`, `fn`, `synchronize` and `Event`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 298-322: Helper routines around run_one_shape / 辅助例程
```python
def run_one_shape(shape: Shape, run_marlin: bool):
    print(f"\n=== {shape.label()} ===")
    w13, w2, w13_s, w2_s, w13_b, w2_b = _make_random_mxfp4(shape, seed=0)
    router_logits, topk_w, topk_i = _make_topk(shape, seed=1)
    x = (
        torch.randn(shape.tokens, shape.hidden, dtype=torch.bfloat16, device="cuda")
        * 0.1
    )

    # FlashInfer cutlass (autotune ON, with bias).
    fi_prep = build_flashinfer_inputs(shape, w13, w2, w13_s, w2_s, w13_b, w2_b)
    fi_at_call = make_flashinfer_runner(
        shape, fi_prep, x, topk_w, topk_i, autotuned=True, with_bias=True
    )
    fi_at_med, fi_at_min = time_call(fi_at_call)
    print(
        f"  FlashInfer cutlass (autotune):     median={fi_at_med:.3f} ms  "
        f"min={fi_at_min:.3f} ms"
    )

    # FlashInfer cutlass (autotune ON, no bias) -- isolate bias epilogue cost.
    fi_at_nb_call = make_flashinfer_runner(
        shape, fi_prep, x, topk_w, topk_i, autotuned=True, with_bias=False
    )
    fi_at_nb_med, fi_at_nb_min = time_call(fi_at_nb_call)
```
**EN:** This range implements helper routine(s) `run_one_shape` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `label`, `_make_random_mxfp4`, `_make_topk` and `randn`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 323-346: Scenario logic / 场景逻辑
```python
    print(
        f"  FlashInfer cutlass (AT, no-bias):  median={fi_at_nb_med:.3f} ms  "
        f"min={fi_at_nb_min:.3f} ms  "
        f"(bias overhead = {fi_at_med - fi_at_nb_med:+.3f} ms / "
        f"{(fi_at_med / fi_at_nb_med - 1) * 100:+.1f}%)"
    )
    fi_med = fi_at_med  # alias for downstream speedup print

    # Marlin
    if run_marlin:
        try:
            ml_prep = build_marlin_inputs(shape, w13, w2, w13_s, w2_s)
            ml_call = make_marlin_runner(
                shape, ml_prep, x, router_logits, topk_w, topk_i
            )
            ml_med, ml_min = time_call(ml_call)
            print(
                f"  SGLang Marlin:                     median={ml_med:.3f} ms  "
                f"min={ml_min:.3f} ms"
            )
            print(f"  speedup (Marlin / FI autotune):    {ml_med / fi_at_med:.2f}x")
            print(f"  speedup (Marlin / FI AT no-bias):  {ml_med / fi_at_nb_med:.2f}x")
        except Exception as exc:  # pylint: disable=broad-except
            print(f"  SGLang Marlin: SKIPPED ({type(exc).__name__}: {exc})")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `cutlass`, `build_marlin_inputs`, `make_marlin_runner` and `time_call`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 349-362: Helper routines around main / 辅助例程
```python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--no-marlin", action="store_true", help="Skip Marlin path.")
    args = parser.parse_args()

    if not torch.cuda.is_available():
        raise SystemExit("CUDA required.")
    cap = torch.cuda.get_device_capability()
    if cap[0] != 9:
        print(f"WARNING: device cap {cap} is not SM90; SM90-specific kernel may fail.")

    print(f"Device: {torch.cuda.get_device_name()} (cap {cap[0]}.{cap[1]})")
    for shape in DEFAULT_SHAPES:
        run_one_shape(shape, run_marlin=not args.no_marlin)
```
**EN:** This range implements helper routine(s) `main` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `ArgumentParser`, `add_argument`, `parse_args` and `is_available`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 363-366: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Streaming responses / 流式响应
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `__future__`, `argparse`, `dataclasses`, `typing`
- **Third-party / 第三方库**: `flashinfer.autotuner`, `flashinfer.fused_moe`, `flashinfer.fused_moe.core`, `torch`
- **Project Modules / 项目模块**: `sglang.jit_kernel.gptq_marlin_repack`, `sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe`, `sglang.srt.layers.quantization.marlin_utils`, `sglang.srt.layers.quantization.marlin_utils_fp4`
