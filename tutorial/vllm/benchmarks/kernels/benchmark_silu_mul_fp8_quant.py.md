# benchmark_silu_mul_fp8_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_silu_mul_fp8_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, activation/quantization benchmarks, FP8 kernel experiments, centered around `_silu_mul_fp8_quant_deep_gemm`, `silu_mul_fp8_quant_deep_gemm_triton`, `benchmark`, `create_comparison_plot`. / 实现与基准测试编排、激活与量化基准、FP8 内核实验相关的逻辑，核心符号包括 `_silu_mul_fp8_quant_deep_gemm`, `silu_mul_fp8_quant_deep_gemm_triton`, `benchmark`, `create_comparison_plot`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 4-15)
```python
"""
Comprehensive 3-way SiLU Benchmark Suite

This benchmark compares three SiLU implementations:
1. SiLU V2 (CUDA) - Optimized CUDA kernel implementation
2. Triton Kernel - Triton-based implementation

The suite generates detailed performance comparisons including:
- Memory bandwidth utilization
- Speedup ratios (baseline vs optimized implementations)
- Performance across different expert configurations and token distributions
"""
```
**EN:** The module docstring introduces Comprehensive 3-way SiLU Benchmark Suite This benchmark compares three SiLU implementations: 1. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Comprehensive 3-way SiLU Benchmark Suite This benchmark compares three SiLU implementations: 1 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 17-28)
```python
from collections.abc import Callable

import matplotlib.pyplot as plt
import numpy as np
import torch

from vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe import (
    persistent_masked_m_silu_mul_quant,
)
from vllm.triton_utils import tl, triton
from vllm.utils.deep_gemm import is_deep_gemm_e8m0_used
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block gathers standard-library helpers such as `collections.abc`; third-party packages such as `matplotlib.pyplot`, `numpy`, `torch`; project-local modules such as `vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `collections.abc`；第三方依赖，如 `matplotlib.pyplot`, `numpy`, `torch`；项目内部模块，如 `vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Function `_silu_mul_fp8_quant_deep_gemm` (lines 32-102)
```python
def _silu_mul_fp8_quant_deep_gemm(
    # Pointers ------------------------------------------------------------
    input_ptr,  # 16-bit activations (E, T, 2*H)
    y_q_ptr,  # fp8 quantized activations (E, T, H)
    y_s_ptr,  # 16-bit scales (E, T, G)
    counts_ptr,  # int32 num tokens per expert (E)
    # Sizes ---------------------------------------------------------------
    H: tl.constexpr,  # hidden dimension (per output)
    GROUP_SIZE: tl.constexpr,  # elements per group (usually 128)
    # Strides for input (elements) ---------------------------------------
    stride_i_e,
    stride_i_t,
    stride_i_h,
    # Strides for y_q (elements) -----------------------------------------
    stride_yq_e,
    stride_yq_t,
    stride_yq_h,
    # Strides for y_s (elements) -----------------------------------------
    stride_ys_e,
    stride_ys_t,
    stride_ys_g,
    # Stride for counts (elements)
    stride_counts_e,
    # Numeric params ------------------------------------------------------
    eps: tl.constexpr,
    fp8_min: tl.constexpr,
    fp8_max: tl.constexpr,
    use_ue8m0: tl.constexpr,
    # Meta ---------------------------------------------------------------
    BLOCK: tl.constexpr,
    NUM_STAGES: tl.constexpr,
):
    G = H // GROUP_SIZE

    # ... omitted for brevity ...
        y_s = tl.maximum(tl.max(tl.abs(y)), eps) / fp8_max
        if use_ue8m0:
            y_s = tl.exp2(tl.ceil(tl.log2(y_s)))

        y_q = tl.clamp(y / y_s, fp8_min, fp8_max).to(y_q_ptr.dtype.element_ty)

        tl.store(y_q_ptr + base_yq_offset + t * stride_yq_t, y_q, mask=mask)
        tl.store(y_s_ptr + base_ys_offset + t * stride_ys_t, y_s)
```
**EN:** `_silu_mul_fp8_quant_deep_gemm` coordinates or measures FP8 kernel experiments. It mainly works with `input_ptr`, `y_q_ptr`, `y_s_ptr`, `counts_ptr`, `H`, ... and relies on `tl.program_id`, `e.to`, `g.to`, `tl.load.to`, `tl.load`, `tl.arange.to` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `_silu_mul_fp8_quant_deep_gemm` 负责协调或测量FP8 内核实验。 它主要处理 `input_ptr`, `y_q_ptr`, `y_s_ptr`, `counts_ptr`, `H`, ...，并结合 `tl.program_id`, `e.to`, `g.to`, `tl.load.to`, `tl.load`, `tl.arange.to` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `silu_mul_fp8_quant_deep_gemm_triton` (lines 105-188)
```python
def silu_mul_fp8_quant_deep_gemm_triton(
    y: torch.Tensor,  # (E, T, 2*H)
    tokens_per_expert: torch.Tensor,  # (E,) number of valid tokens per expert
    num_parallel_tokens,
    group_size: int = 128,
    eps: float = 1e-10,
    expert_offsets: torch.Tensor = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Quantize silu(y[..., :H]) * y[..., H:] to FP8 with group per-token scales

    y has shape (E, T, 2*H). The first half of the last dimension is
    silu-activated, multiplied by the second half, then quantized into FP8.

    Returns `(y_q, y_s)` where
    * `y_q`: FP8 tensor, shape (E, T, H), same layout as y[..., :H]
    * `y_s`: FP32 tensor, shape (E, T, H // group_size), strides (T*G, 1, T)
    """
    assert y.ndim == 3, "y must be (E, T, 2*H)"
    E, T, H2 = y.shape
    assert H2 % 2 == 0, "last dim of y must be even (2*H)"
    H = H2 // 2
    G = (H + group_size - 1) // group_size
    assert H % group_size == 0, "H must be divisible by group_size"
    assert tokens_per_expert.ndim == 1 and tokens_per_expert.shape[0] == E, (
        "tokens_per_expert must be shape (E,)"
    )
    tokens_per_expert = tokens_per_expert.to(device=y.device, dtype=torch.int32)

    # allocate outputs
    fp8_dtype = torch.float8_e4m3fn
    y_q = torch.empty((E, T, H), dtype=fp8_dtype, device=y.device)

    # strides (elements)
    stride_i_e, stride_i_t, stride_i_h = y.stride()
    # ... omitted for brevity ...
        fp8_max,
        is_deep_gemm_e8m0_used(),
        BLOCK=group_size,
        NUM_STAGES=4,
        num_warps=1,
    )

    return y_q, y_s
```
**EN:** `silu_mul_fp8_quant_deep_gemm_triton` Quantize silu(y[..., :H]) * y[..., H:] to FP8 with group per-token scales y has shape (E, T, 2*H). It mainly works with `y`, `tokens_per_expert`, `num_parallel_tokens`, `group_size`, `eps`, ... and relies on `tokens_per_expert.to`, `torch.empty`, `y.stride`, `y_q.stride`, `torch.empty_strided`, `tokens_per_expert.stride` plus value production to move data through this part of the benchmark pipeline.
**CN:** `silu_mul_fp8_quant_deep_gemm_triton` 的职责是：Quantize silu(y[..., :H]) * y[..., H:] to FP8 with group per-token scales y has shape (E, T, 2*H)。 它主要处理 `y`, `tokens_per_expert`, `num_parallel_tokens`, `group_size`, `eps`, ...，并结合 `tokens_per_expert.to`, `torch.empty`, `y.stride`, `y_q.stride`, `torch.empty_strided`, `tokens_per_expert.stride` 以及 结果返回 来完成这一段基准测试流程。

### Top-level setup (lines 192-192)
```python
strategies = ["random_imbalanced", "uniform", "max_t"]
```
**EN:** This top-level block prepares shared state such as `strategies`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `strategies`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `benchmark` (lines 195-307)
```python
def benchmark(
    kernel: Callable,
    E: int,
    T: int,
    H: int,
    total_tokens: int,
    num_parallel_tokens: int = 64,
    G: int = 128,
    runs: int = 200,
    num_warmups: int = 20,
    gen_strategy: str = "default",
    iterations_per_run: int = 20,
):
    def generate_data(seed_offset=0):
        """Generate input data with given seed offset"""
        set_random_seed(42 + seed_offset)
        y = torch.rand((E, T, 2 * H), dtype=torch.bfloat16, device="cuda").contiguous()

        if gen_strategy == "random_imbalanced":

            def generate_expert_loads(n_e, total_tokens, ratio, device="cuda"):
                mean = total_tokens // n_e
                min_max = mean // ratio
                e = torch.ones(size=(E,), dtype=torch.int64, device=device) * mean
                e[0] = min_max
                r = torch.rand(size=(E - 1,))
                r /= r.sum()
                r *= total_tokens - min_max
                r = r.round().long()
                e[1:] = r.to(device=device)
                return e

            tokens_per_expert = generate_expert_loads(E, total_tokens, 0.7, "cuda")
        elif gen_strategy == "uniform":
    # ... omitted for brevity ...

    HOPPER_BANDWIDTH_TBPS = 3.35
    return (
        median_time_ms,
        gflops,
        memory_bw,
        (memory_bw / (HOPPER_BANDWIDTH_TBPS * 1024)) * 100,
    )
```
**EN:** `benchmark` coordinates or measures benchmark orchestration. It mainly works with `kernel`, `E`, `T`, `H`, `total_tokens`, ... and relies on `set_random_seed`, `torch.rand.contiguous`, `torch.rand`, `torch.ones`, `r.sum`, `r.round.long` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark` 负责协调或测量基准测试编排。 它主要处理 `kernel`, `E`, `T`, `H`, `total_tokens`, ...，并结合 `set_random_seed`, `torch.rand.contiguous`, `torch.rand`, `torch.ones`, `r.sum`, `r.round.long` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `create_comparison_plot` (lines 310-352)
```python
def create_comparison_plot(
    ratios, silu_v2_times, triton_times, config_labels, strategy_name, id
):
    fig, ax = plt.subplots(1, 1, figsize=(18, 6))

    # Configure x-axis positions
    x = np.arange(len(config_labels))
    width = 0.25

    # Execution Time plot (lower is better)
    ax.bar(x, silu_v2_times, width, label="SiLU V2 (CUDA)", alpha=0.8, color="blue")
    ax.bar(
        x + width, triton_times, width, label="Triton Kernel", alpha=0.8, color="green"
    )

    # Add speedup labels over each bar trio
    for i in range(len(x)):
        triton_v2_speedup = ratios[i][1]  # triton/v2
        max_height = max(silu_v2_times[i], triton_times[i])

        # Triton/V2 speedup
        ax.text(
            x[i] + width / 2,
            max_height + max_height * 0.02,
            f"{triton_v2_speedup:.2f}x",
            ha="center",
            va="bottom",
            fontweight="bold",
            fontsize=8,
        )

    ax.set_xlabel("Configuration")
    ax.set_ylabel("% Utilization")
    ax.set_title(
        f"Memory Bandwidth Utilization (%) - {strategy_name}\n(Higher is Better)"
    )
    ax.set_xticks(x)
    ax.set_xticklabels(config_labels, rotation=45, ha="right")
    ax.legend()
    ax.grid(True, alpha=0.3)

    plt.tight_layout()
    return fig, ax
```
**EN:** `create_comparison_plot` implements plotting and result visualization. It mainly works with `ratios`, `silu_v2_times`, `triton_times`, `config_labels`, `strategy_name`, ... and relies on `plt.subplots`, `np.arange`, `len`, `ax.bar`, `range`, `max` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `create_comparison_plot` 负责实现绘图与结果可视化。 它主要处理 `ratios`, `silu_v2_times`, `triton_times`, `config_labels`, `strategy_name`, ...，并结合 `plt.subplots`, `np.arange`, `len`, `ax.bar`, `range`, `max` 以及 循环迭代 来完成这一段基准测试流程。

### Function `create_combined_plot` (lines 355-442)
```python
def create_combined_plot(all_results):
    num_strategies = len(all_results)
    fig, axes = plt.subplots(num_strategies, 1, figsize=(22, 7 * num_strategies))

    if num_strategies == 1:
        axes = [axes]

    for idx, (
        strategy_name,
        all_ratios,
        all_silu_v2_results,
        all_triton_results,
        config_labels,
        config_x_axis,
    ) in enumerate(all_results):
        ax = axes[idx]

        # Flatten the nested results to get bandwidth percentages for plotting
        silu_v2_bandwidths = []
        triton_bandwidths = []
        flat_ratios = []

        for config_results in all_silu_v2_results:
            for result in config_results:
                silu_v2_bandwidths.append(result[3])  # bandwidth percentage

        for config_results in all_triton_results:
            for result in config_results:
                triton_bandwidths.append(result[3])  # bandwidth percentage

        for config_ratios in all_ratios:
            for ratio in config_ratios:
                flat_ratios.append(ratio)

    # ... omitted for brevity ...
        ax.grid(True, alpha=0.3)

    plt.tight_layout()
    filename = "silu_benchmark_combined_3way.png"
    plt.savefig(filename, dpi=300, bbox_inches="tight")
    plt.show()

    return filename
```
**EN:** `create_combined_plot` implements plotting and result visualization. It mainly works with `all_results` and relies on `len`, `plt.subplots`, `enumerate`, `silu_v2_bandwidths.append`, `triton_bandwidths.append`, `flat_ratios.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `create_combined_plot` 负责实现绘图与结果可视化。 它主要处理 `all_results`，并结合 `len`, `plt.subplots`, `enumerate`, `silu_v2_bandwidths.append`, `triton_bandwidths.append`, `flat_ratios.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Top-level setup (lines 445-567)
```python
outer_dim = 7168
configs = [
    # DeepSeekV3 Configs
    # (1, 56, 7168),
    (8, 1024, 7168),
    # (32, 56, 7168),
    # DeepSeekV3 Configs
    (32, 1024, 7168),
    # DeepSeekV3 Configs
    (256, 1024, 7168),
]

runs = 100
num_warmups = 20

strategy_descriptions = {
    "uniform": "Uniform Random",
    "random_imbalanced": "Imbalanced Random",
    "max_t": "Even Assignment",
    "first_t": "experts[0] = T, experts[1:] = 0",
}

print(f"GPU: {torch.cuda.get_device_name()}")
print(f"Testing strategies: {', '.join(strategies)}")
print(f"Configurations: {len(configs)} configs")

all_results = []

# Run benchmarks for each strategy
for id, strategy in enumerate(strategies):
    print(f"\n{'=' * 60}")
    print(f"Testing strategy: {strategy_descriptions[strategy]}")
    print(f"{'=' * 60}")

# ... omitted for brevity ...
        v2_time = silu_v2_results[i][0]
        triton_time = triton_results[i][0]
        triton_v2_speedup = triton_time / v2_time
        config_label = f"E={E:3d},T={T:4d},H={H:4d}"
        print(
            f"{config_label:<20} {v2_time:8.5f} {triton_time:10.5f} "
            f"{triton_v2_speedup:8.2f}x"
        )
```
**EN:** This top-level block prepares shared state such as `outer_dim`, `configs`, `runs`, `num_warmups`, `strategy_descriptions`, `all_results`. It uses `print`, `torch.cuda.get_device_name`, `join`, `len`, `enumerate`, `total_tokens_config.append` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `outer_dim`, `configs`, `runs`, `num_warmups`, `strategy_descriptions`, `all_results`。它借助 `print`, `torch.cuda.get_device_name`, `join`, `len`, `enumerate`, `total_tokens_config.append` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `create_total_tokens_plot` (lines 570-708)
```python
def create_total_tokens_plot(all_results):
    num_strategies = len(all_results)
    num_configs = len(configs)

    fig, axs = plt.subplots(
        num_strategies, num_configs * 2, figsize=(32, 8 * num_strategies)
    )

    # Add main title to the entire figure
    fig.suptitle(
        "Performance Analysis: Speedup vs Bandwidth Utilization (SiLU V2, and Triton)",
        fontsize=18,
        fontweight="bold",
        y=0.98,
    )

    # Handle single strategy case
    if num_strategies == 1:
        axs = axs.reshape(1, -1)

    # Handle single config case
    if num_configs == 1:
        axs = axs.reshape(-1, 2)

    for strategy_idx, result in enumerate(all_results):
        (
            strategy_name,
            all_ratios,
            all_silu_v2_results,
            all_triton_results,
            config_labels,
            config_x_axis,
        ) = result

    # ... omitted for brevity ...

    plt.tight_layout()
    plt.subplots_adjust(top=0.93)  # Make room for main title
    filename = "silu_benchmark_total_tokens_3way.png"
    plt.savefig(filename, dpi=300, bbox_inches="tight")
    plt.show()

    return filename
```
**EN:** `create_total_tokens_plot` implements plotting and result visualization. It mainly works with `all_results` and relies on `len`, `plt.subplots`, `fig.suptitle`, `axs.reshape`, `enumerate`, `range` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `create_total_tokens_plot` 负责实现绘图与结果可视化。 它主要处理 `all_results`，并结合 `len`, `plt.subplots`, `fig.suptitle`, `axs.reshape`, `enumerate`, `range` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Top-level setup (lines 712-720)
```python
combined_plot_filename = create_combined_plot(all_results)
total_tokens_plot_filename = create_total_tokens_plot(all_results)

print(f"\n{'=' * 80}")
print("3-Way Benchmark Suite Complete!")
print(f"Generated combined comparison plot: {combined_plot_filename}")
print(f"Generated total tokens analysis plot: {total_tokens_plot_filename}")
print("Compared: SiLU V2 (CUDA), and Triton implementations")
print(f"{'=' * 80}")
```
**EN:** This top-level block prepares shared state such as `combined_plot_filename`, `total_tokens_plot_filename`. It uses `create_combined_plot`, `create_total_tokens_plot`, `print` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `combined_plot_filename`, `total_tokens_plot_filename`。它借助 `create_combined_plot`, `create_total_tokens_plot`, `print` 建立默认值、常量或可复用对象，为后续基准入口做准备。

## Key Concepts / 关键概念
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `collections.abc`.
- **CN:** 标准库依赖：`collections.abc`。
- **EN:** Third-party packages: `matplotlib.pyplot`, `numpy`, `torch`.
- **CN:** 第三方依赖：`matplotlib.pyplot`, `numpy`, `torch`。
- **EN:** Internal modules: `vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`。
