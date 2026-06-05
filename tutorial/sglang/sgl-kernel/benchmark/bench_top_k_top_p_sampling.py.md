# bench_top_k_top_p_sampling.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_top_k_top_p_sampling.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `top k top p sampling` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `top k top p sampling` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and module setup
````python
import itertools
import os

import flashinfer.sampling
import sgl_kernel
import torch
import triton
import triton.testing

from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 12-12: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 15-48: `torch_top_k_top_p_joint_sampling_from_probs` definition
````python
def torch_top_k_top_p_joint_sampling_from_probs(
    normalized_prob, top_k, top_p, eps=1e-4
):
    """Reference PyTorch implementation of joint top-k top-p sampling."""
    batch_size, vocab_size = normalized_prob.shape
    samples = torch.empty(batch_size, dtype=torch.int64, device=normalized_prob.device)

    for i in range(batch_size):
        p_val = top_p[i].item()
        k_val = top_k[i].item()

        # top-p mask
        sorted_prob, indices = torch.sort(normalized_prob[i], descending=False)
        cdf = torch.cumsum(sorted_prob, dim=-1)
        mask_top_p = torch.zeros(
            vocab_size, dtype=torch.int32, device=normalized_prob.device
        )
        mask_top_p.scatter_add_(0, indices, (cdf > (1 - p_val) - eps).int())

        # top-k mask
        sorted_prob_desc, _ = torch.sort(normalized_prob[i], descending=True)
        pivot = sorted_prob_desc[k_val - 1]
        mask_top_k = (normalized_prob[i] >= pivot).int()

        # joint mask
        mask = torch.minimum(mask_top_p, mask_top_k).bool()

        # sample from masked probs
        masked_probs = normalized_prob[i] * mask
        masked_probs = masked_probs / masked_probs.sum()
        idx = torch.multinomial(masked_probs, 1)
        samples[i] = idx

    return samples
````
**EN:** This section defines `torch_top_k_top_p_joint_sampling_from_probs` and implements the core logic associated with torch top k top p joint sampling from probs. Docstring summary: Reference PyTorch implementation of joint top-k top-p sampling.
**CN:** 该部分定义 `torch_top_k_top_p_joint_sampling_from_probs`，并实现与 torch top k top p joint sampling from probs 相关的核心逻辑。 文档字符串摘要：Reference PyTorch implementation of joint top-k top-p sampling.

### Lines 51-73: `calculate_diff` definition
````python
def calculate_diff(batch_size, vocab_size, p):
    """Compare Torch reference and SGLang kernel for correctness."""
    torch.manual_seed(42)
    if p == 0.1:
        k = int(vocab_size * 0.5)
    elif p == 0.5:
        k = int(vocab_size * 0.1)
    else:
        raise ValueError("p not recognized")

    device = torch.device("cuda")
    pre_norm_prob = torch.rand(batch_size, vocab_size, device=device)
    normalized_prob = pre_norm_prob / pre_norm_prob.sum(dim=-1, keepdim=True)

    top_p_tensor = torch.full((batch_size,), p, device=device)
    top_k_tensor = torch.full((batch_size,), k, device=device)

    torch_samples = torch_top_k_top_p_joint_sampling_from_probs(
        normalized_prob, top_k_tensor, top_p_tensor
    )
    sglang_samples = flashinfer.sampling.top_k_top_p_sampling_from_probs(
        normalized_prob, top_k_tensor, top_p_tensor, filter_apply_order="joint"
    )
````
**EN:** This section defines `calculate_diff`. It calculates the `diff` path used by the module. Docstring summary: Compare Torch reference and SGLang kernel for correctness.
**CN:** 该部分定义 `calculate_diff`。它负责计算模块中与 `diff` 相关的处理路径。 文档字符串摘要：Compare Torch reference and SGLang kernel for correctness.

### Lines 74-76: Comments and local context
````python


# parameter space - simplified for CI
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 77-84: Conditional logic
````python
if IS_CI:
    batch_size_range = [16]  # Single batch size for CI
    vocab_size_range = [111]  # Single vocab size for CI
    p_range = [0.1]  # Single p value for CI
else:
    batch_size_range = [16, 64, 128]
    vocab_size_range = [111, 32000]
    p_range = [0.1, 0.5]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 86-86: Constants and configuration
````python
configs = list(itertools.product(batch_size_range, vocab_size_range, p_range))
````
**EN:** This block defines shared constants or configuration values such as `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `configs`），供后续函数或控制流程复用。

### Lines 89-130: `benchmark_sampling` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size", "vocab_size", "p"],
        x_vals=configs,
        line_arg="provider",
        line_vals=["torch", "sglang"],
        line_names=["Torch Reference", "SGL Kernel"],
        styles=[("red", "-"), ("green", "-")],
        ylabel="us",
        plot_name="top-k-top-p-joint-sampling-performance",
        args={},
    )
)
def benchmark_sampling(batch_size, vocab_size, p, provider):
    torch.manual_seed(42)
    if p == 0.1:
        k = int(vocab_size * 0.5)
    elif p == 0.5:
        k = int(vocab_size * 0.1)
    else:
        raise ValueError("p not recognized")

    device = torch.device("cuda")
    pre_norm_prob = torch.rand(batch_size, vocab_size, device=device)
    normalized_prob = pre_norm_prob / pre_norm_prob.sum(dim=-1, keepdim=True)
    top_p_tensor = torch.full((batch_size,), p, device=device)
    top_k_tensor = torch.full((batch_size,), k, device=device)

    if provider == "torch":
        fn = lambda: torch_top_k_top_p_joint_sampling_from_probs(
            normalized_prob.clone(), top_k_tensor, top_p_tensor
        )
    elif provider == "sglang":
        fn = lambda: flashinfer.sampling.top_k_top_p_sampling_from_probs(
            normalized_prob.clone(),
            top_k_tensor,
            top_p_tensor,
            filter_apply_order="joint",
        )

    ms, min_ms, max_ms = triton.testing.do_bench(fn, quantiles=[0.5, 0.2, 0.8])
    return 1000 * ms, 1000 * max_ms, 1000 * min_ms
````
**EN:** This section defines `benchmark_sampling` and implements the core logic associated with benchmark sampling. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark_sampling`，并实现与 benchmark sampling 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 133-146: Command-line entry point
````python
if __name__ == "__main__":
    # Correctness check - simplified for CI
    if IS_CI:
        # Only test one configuration in CI
        test_configs = [configs[0]] if configs else [(16, 111, 0.1)]
    else:
        test_configs = configs

    for cfg in test_configs:
        calculate_diff(*cfg)

    print("\n" + "=" * 60)
    print("Starting performance benchmark...")
    benchmark_sampling.run(print_data=True)
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `torch_top_k_top_p_joint_sampling_from_probs`, `calculate_diff`, `benchmark_sampling`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.utils`
- **External / 外部**: `flashinfer.sampling`, `itertools`, `os`, `torch`, `triton`, `triton.testing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
