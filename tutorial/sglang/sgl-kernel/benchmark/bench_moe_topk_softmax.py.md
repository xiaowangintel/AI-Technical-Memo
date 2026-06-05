# bench_moe_topk_softmax.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_moe_topk_softmax.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `MoE topk softmax` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `MoE topk softmax` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and module setup
````python
import itertools
import os

import pytest
import torch
import triton
from sgl_kernel import topk_softmax

from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 10-11: Comments and local context
````python

# Optional vLLM import
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 12-18: Optional dependency handling
````python
try:
    from vllm import _custom_ops as vllm_custom_ops

    VLLM_AVAILABLE = True
except ImportError:
    vllm_custom_ops = None
    VLLM_AVAILABLE = False
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 19-20: Comments and local context
````python

# Optional MUSA import
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 21-35: Optional dependency handling
````python
try:
    from sglang.srt.utils import is_musa

    if is_musa():
        from sglang.srt.hardware_backend.musa.kernels.topk import (
            topk_softmax as musa_topk_softmax,
        )

        MUSA_AVAILABLE = True
    else:
        musa_topk_softmax = None
        MUSA_AVAILABLE = False
except ImportError:
    musa_topk_softmax = None
    MUSA_AVAILABLE = False
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 37-37: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 40-59: `vllm_topk_softmax` definition
````python
def vllm_topk_softmax(gating_output, topk):
    if not VLLM_AVAILABLE:
        # Fallback to SGLang implementation if vLLM is not available
        return sglang_topk_softmax(gating_output, topk)

    num_tokens, num_experts = gating_output.shape

    topk_weights = torch.empty(
        (num_tokens, topk), device=gating_output.device, dtype=torch.float32
    )
    topk_indices = torch.empty(
        (num_tokens, topk), dtype=torch.int32, device=gating_output.device
    )
    token_expert_indices = torch.empty(
        (num_tokens, topk), dtype=torch.int32, device=gating_output.device
    )
    torch.ops._moe_C.topk_softmax(
        topk_weights, topk_indices, token_expert_indices, gating_output
    )
    return topk_weights, topk_indices
````
**EN:** This section defines `vllm_topk_softmax` and implements the core logic associated with vllm topk softmax.
**CN:** 该部分定义 `vllm_topk_softmax`，并实现与 vllm topk softmax 相关的核心逻辑。

### Lines 62-78: `sglang_topk_softmax` definition
````python
def sglang_topk_softmax(gating_output, topk):
    num_tokens, num_experts = gating_output.shape

    topk_weights = torch.empty(
        (num_tokens, topk), device=gating_output.device, dtype=torch.float32
    )
    topk_indices = torch.empty(
        (num_tokens, topk), dtype=torch.int32, device=gating_output.device
    )

    topk_softmax(
        topk_weights=topk_weights,
        topk_ids=topk_indices,
        gating_output=gating_output,
    )

    return topk_weights, topk_indices
````
**EN:** This section defines `sglang_topk_softmax` and implements the core logic associated with sglang topk softmax.
**CN:** 该部分定义 `sglang_topk_softmax`，并实现与 sglang topk softmax 相关的核心逻辑。

### Lines 81-97: `musa_topk_softmax_fn` definition
````python
def musa_topk_softmax_fn(gating_output, topk):
    num_tokens, num_experts = gating_output.shape

    topk_weights = torch.empty(
        (num_tokens, topk), device=gating_output.device, dtype=torch.float32
    )
    topk_indices = torch.empty(
        (num_tokens, topk), dtype=torch.int32, device=gating_output.device
    )

    musa_topk_softmax(
        topk_weights,
        topk_indices,
        gating_output,
    )

    return topk_weights, topk_indices
````
**EN:** This section defines `musa_topk_softmax_fn` and implements the core logic associated with MUSA topk softmax fn.
**CN:** 该部分定义 `musa_topk_softmax_fn`，并实现与 MUSA topk softmax fn 相关的核心逻辑。

### Lines 100-136: `calculate_diff` definition
````python
def calculate_diff(num_tokens, num_experts, topk):
    gating_output = torch.randn(
        (num_tokens, num_experts), device="cuda", dtype=torch.float32
    )
    weights_sglang, indices_sglang = sglang_topk_softmax(gating_output.clone(), topk)

    if MUSA_AVAILABLE:
        weights_musa, indices_musa = musa_topk_softmax_fn(gating_output.clone(), topk)
        weights_diff = torch.abs(weights_sglang - weights_musa).mean().item()
        indices_match = torch.equal(indices_sglang, indices_musa)

        if (
            torch.allclose(weights_sglang, weights_musa, atol=1e-3, rtol=1e-3)
            and indices_match
        ):
            print("✅ SGLang and MUSA topk_softmax implementations match")
        else:
            print(
                f"❌ Implementations differ: Weights diff={weights_diff}, Indices match={indices_match}"
            )
    else:
        print("⚠️ MUSA not available, skipping MUSA comparison")

    if VLLM_AVAILABLE:
        weights_vllm, indices_vllm = vllm_topk_softmax(gating_output.clone(), topk)
        weights_diff_vllm = torch.abs(weights_vllm - weights_sglang).mean().item()
        indices_match_vllm = torch.equal(indices_vllm, indices_sglang)

        if (
            torch.allclose(weights_vllm, weights_sglang, atol=1e-3, rtol=1e-3)
            and indices_match_vllm
        ):
            print("✅ VLLM and SGLang topk_softmax implementations match")
        else:
            print(
                f"❌ VLLM vs SGLang differ: Weights diff={weights_diff_vllm}, Indices match={indices_match_vllm}"
            )
````
**EN:** This section defines `calculate_diff`. It calculates the `diff` path used by the module.
**CN:** 该部分定义 `calculate_diff`。它负责计算模块中与 `diff` 相关的处理路径。

### Lines 137-139: Comments and local context
````python


# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 140-147: Conditional logic
````python
if IS_CI:
    num_tokens_range = [128]  # Single value for CI
    num_experts_range = [32]  # Single value for CI
    topk_range = [2]  # Single value for CI
else:
    num_tokens_range = [128, 512, 1024, 2048, 4096, 8192, 16384, 32768]
    num_experts_range = [32, 64, 128, 256, 12, 512]
    topk_range = [1, 2, 4, 8]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 149-149: Constants and configuration
````python
configs = list(itertools.product(num_tokens_range, num_experts_range, topk_range))
````
**EN:** This block defines shared constants or configuration values such as `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `configs`），供后续函数或控制流程复用。

### Lines 150-152: Comments and local context
````python


# Filter providers based on availability
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 153-155: Constants and configuration
````python
line_vals = ["sglang"]
line_names = ["SGLang"]
styles = [("blue", "-")]
````
**EN:** This block defines shared constants or configuration values such as `line_vals`, `line_names`, `styles`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `line_vals`, `line_names`, `styles`），供后续函数或控制流程复用。

### Lines 157-160: Conditional logic
````python
if VLLM_AVAILABLE:
    line_vals.append("vllm")
    line_names.append("VLLM")
    styles.append(("green", "-"))
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 162-165: Conditional logic
````python
if MUSA_AVAILABLE:
    line_vals.append("musa")
    line_names.append("MUSA")
    styles.append(("red", "-"))
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 168-201: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["num_tokens", "num_experts", "topk"],
        x_vals=configs,
        line_arg="provider",
        line_vals=line_vals,
        line_names=line_names,
        styles=styles,
        ylabel="Latency (us)",
        plot_name="topk-softmax-performance",
        args={},
    )
)
def benchmark(num_tokens, num_experts, topk, provider):

    gating_output = torch.randn(
        (num_tokens, num_experts), device="cuda", dtype=torch.float32
    )

    if provider == "vllm" or provider == "vllm1":
        if not VLLM_AVAILABLE:
            return (0, 0, 0)
        fn = lambda: vllm_topk_softmax(gating_output, topk)
    elif provider == "sglang" or provider == "sglang1":
        fn = lambda: sglang_topk_softmax(gating_output, topk)
    elif provider == "musa" or provider == "musa1":
        if not MUSA_AVAILABLE:
            return (0, 0, 0)
        fn = lambda: musa_topk_softmax_fn(gating_output, topk)

    quantiles = [0.5, 0.2, 0.8]
    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(fn, quantiles=quantiles)

    return 1000 * ms, 1000 * max_ms, 1000 * min_ms
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 204-220: Command-line entry point
````python
if __name__ == "__main__":
    # Simplify configs for CI environment
    if IS_CI:
        test_configs = [(20, 32, 2)]  # Single config for CI
    else:
        test_configs = [
            (20, 256, 4),
            (20, 256, 8),
            (20, 12, 4),
            (20, 12, 1),
            (20, 512, 4),
            (20, 512, 1),
        ]

    for num_tokens, num_experts, topk in test_configs:
        calculate_diff(num_tokens, num_experts, topk)
    benchmark.run(print_data=True)
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `vllm_topk_softmax`, `sglang_topk_softmax`, `musa_topk_softmax_fn`, `calculate_diff`, `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.srt.hardware_backend.musa.kernels.topk`, `sglang.srt.utils`, `sglang.utils`
- **External / 外部**: `itertools`, `os`, `pytest`, `torch`, `triton`, `vllm`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
