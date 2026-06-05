# bench_mrope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_mrope.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `multimodal RoPE` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `multimodal RoPE` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Comments and local context
````python
# Adapted from vLLM benchmark_mrope.py

# This script benchmarks the mrope kernel (mainly for Qwen2VL and Qwen2.5VL models).
# It generates test data, runs benchmarks, and saves results to a CSV file.
#
# The CSV file (named with current date/time) contains these columns:
# model_name, tp_size, num_tokens, num_heads, num_kv_heads, head_dim, max_position,
# rope_theta, is_neox_style, rope_scaling, dtype, torch_mean, torch_median, torch_p99,
# torch_min, torch_max, triton_mean, triton_median, triton_p99, triton_min, triton_max,
# speedup
#
# == Usage Examples ==
#
# Single model benchmark:
# python3 benchmark_mrope.py --model-name Qwen/Qwen2.5-VL-7B-Instruct --tp-size 8 \
#   --warmup-iter 10 --benchmark-iter 100 --dtype bfloat16 --seed 0 --num-tokens 1024

````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 18-26: Imports and module setup
````python
import argparse
import time
from typing import Any

import numpy as np
import torch
from transformers import AutoConfig

from sglang.srt.layers.rotary_embedding import get_rope
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 28-28: Constants and configuration
````python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
````
**EN:** This block defines shared constants or configuration values such as `device`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `device`），供后续函数或控制流程复用。

### Lines 31-34: `get_model_config` definition
````python
def get_model_config(model_name: str):
    """Get model configuration parameters"""
    config = AutoConfig.from_pretrained(model_name, trust_remote_code=True)
    return config
````
**EN:** This section defines `get_model_config`. It retrieves or computes the `model config` path used by the module. Docstring summary: Get model configuration parameters
**CN:** 该部分定义 `get_model_config`。它负责获取或计算模块中与 `model config` 相关的处理路径。 文档字符串摘要：Get model configuration parameters

### Lines 37-56: `generate_test_data` definition
````python
def generate_test_data(
    num_tokens: int,
    num_q_heads: int,
    num_kv_heads: int,
    head_size: int,
    max_position_embeddings: int,
    dtype: torch.dtype,
    device: torch.device,
):
    """Generate test data for given configuration."""
    # Create 2D positions (3, num_tokens) for multimodal case
    positions = torch.randint(
        0, max_position_embeddings // 4, (3, num_tokens), device=device
    )

    # Create query and key tensors
    query = torch.randn(num_tokens, num_q_heads * head_size, dtype=dtype, device=device)
    key = torch.randn(num_tokens, num_kv_heads * head_size, dtype=dtype, device=device)

    return positions, query, key
````
**EN:** This section defines `generate_test_data` and implements the core logic associated with generate test data. Docstring summary: Generate test data for given configuration.
**CN:** 该部分定义 `generate_test_data`，并实现与 generate test data 相关的核心逻辑。 文档字符串摘要：Generate test data for given configuration.

### Lines 59-68: `calculate_stats` definition
````python
def calculate_stats(times: list[float]) -> dict[str, float]:
    """Calculate statistics from a list of times."""
    times_array = np.array(times)
    return {
        "mean": np.mean(times_array),
        "median": np.median(times_array),
        "p99": np.percentile(times_array, 99),
        "min": np.min(times_array),
        "max": np.max(times_array),
    }
````
**EN:** This section defines `calculate_stats`. It calculates the `stats` path used by the module. Docstring summary: Calculate statistics from a list of times.
**CN:** 该部分定义 `calculate_stats`。它负责计算模块中与 `stats` 相关的处理路径。 文档字符串摘要：Calculate statistics from a list of times.

### Lines 71-185: `benchmark_mrope` definition
````python
def benchmark_mrope(
    model_name: str,
    num_tokens: int,
    head_dim: int,
    tp_size: int,
    num_heads: int,
    num_kv_heads: int,
    max_position: int = 8192,
    rope_theta: float = 10000,
    is_neox_style: bool = True,
    rope_scaling: dict[str, Any] = None,
    dtype: torch.dtype = torch.bfloat16,
    seed: int = 0,
    warmup_iter: int = 10,
    benchmark_iter: int = 100,
):
    torch.manual_seed(seed)
    torch.set_default_device(device)
    # the parameters to compute the q k v size based on tp_size
    mrope_helper_class = get_rope(
        head_size=head_dim,
        rotary_dim=head_dim,
        max_position=max_position,
        base=rope_theta,
        is_neox_style=is_neox_style,
        rope_scaling=rope_scaling,
        dtype=dtype,
    ).to(device=device)

    print(80 * "=")
    print(
        f"Evaluating model: {model_name} "
        f"with tp_size: {tp_size} "
        f"and num_tokens: {num_tokens}, "
        f"dtype: {dtype}"
    )

    # create q k v input tensors
    # create rotary pos emb input tensors
    positions, query, key = generate_test_data(
        num_tokens, num_heads, num_kv_heads, head_dim, max_position, dtype, device
    )

    # Warm up
    for _ in range(warmup_iter):
        mrope_helper_class.forward_native(
            positions,
            query.clone(),
            key.clone(),
        )

        mrope_helper_class.forward(
            positions,
            query.clone(),
            key.clone(),
        )

    torch.cuda.synchronize()

    # Time reference implementation
    torch_times = []
    for _ in range(benchmark_iter):
        query_clone = query.clone()
        key_clone = key.clone()
        torch.cuda.synchronize()
        start_time = time.time()

        mrope_helper_class.forward_native(
            positions,
            query_clone,
            key_clone,
        )

        torch.cuda.synchronize()
        torch_times.append(time.time() - start_time)

    # Time triton kernel implementation
    triton_times = []
    for _ in range(benchmark_iter):
        query_clone = query.clone()
        key_clone = key.clone()
        torch.cuda.synchronize()
        start_time = time.time()
        mrope_helper_class.forward(
            positions,
            query_clone,
            key_clone,
        )
        torch.cuda.synchronize()
        triton_times.append(time.time() - start_time)

    # Calculate statistics
    torch_stats = calculate_stats(torch_times)
    triton_stats = calculate_stats(triton_times)
    print(f"\nPerformance for config ({num_tokens}, {num_heads}, {num_kv_heads}):")

    print(
        f"Torch implementation: "
        f"mean={torch_stats['mean']:.8f}s, "
        f"median={torch_stats['median']:.8f}s, "
        f"p99={torch_stats['p99']:.8f}s"
    )

    print(
        f"Triton implementation: "
        f"mean={triton_stats['mean']:.8f}s, "
        f"median={triton_stats['median']:.8f}s, "
        f"p99={triton_stats['p99']:.8f}s"
    )

    print(
        f"Triton Speedup over Torch: {torch_stats['mean'] / triton_stats['mean']:.8f}x"
    )

    return torch_stats, triton_stats
````
**EN:** This section defines `benchmark_mrope` and implements the core logic associated with benchmark multimodal RoPE.
**CN:** 该部分定义 `benchmark_mrope`，并实现与 benchmark multimodal RoPE 相关的核心逻辑。

### Lines 188-250: Command-line entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Benchmark the rotary embedding kernels."
    )
    parser.add_argument("--model-name", type=str, default="")
    parser.add_argument("--tp-size", type=int, default=1)
    parser.add_argument("--warmup-iter", type=int, default=10)
    parser.add_argument("--benchmark-iter", type=int, default=100)
    parser.add_argument("--dtype", type=str, choices=["bfloat16"], default="bfloat16")
    parser.add_argument("--seed", type=int, default=0)
    parser.add_argument("--num-tokens", type=int, nargs="+", required=False)
    parser.add_argument("--trust-remote-code", action="store_true")
    args = parser.parse_args()
    print(args)

    model_tp_dict = {}
    if args.model_name == "":
        model_tp_dict = {
            "Qwen/Qwen2-VL-2B-Instruct": [1],
            "Qwen/Qwen2-VL-7B-Instruct": [1],
            "Qwen/Qwen2-VL-72B-Instruct": [2, 4, 8],
            "Qwen/Qwen2.5-VL-3B-Instruct": [1, 2, 4, 8],
            "Qwen/Qwen2.5-VL-7B-Instruct": [1, 2, 4, 8],
            "Qwen/Qwen2.5-VL-72B-Instruct": [2, 4, 8],
        }
    else:
        model_tp_dict[args.model_name] = [args.tp_size]

    if args.num_tokens is None:
        num_tokens_list = [2**i for i in range(0, 18)]
    else:
        num_tokens_list = args.num_tokens

    for model_name, tp_list in model_tp_dict.items():
        for tp_size in tp_list:
            config = get_model_config(model_name)
            # get the model config
            total_num_kv_heads = config.num_key_value_heads
            total_num_heads = config.num_attention_heads
            num_heads = total_num_heads // tp_size
            num_kv_heads = max(1, total_num_kv_heads // tp_size)
            head_dim = config.hidden_size // total_num_heads
            is_neox_style = True
            rope_theta = config.rope_theta
            max_position = config.max_position_embeddings

            for num_tokens in num_tokens_list:
                benchmark_mrope(
                    model_name=model_name,
                    num_tokens=num_tokens,
                    head_dim=head_dim,
                    tp_size=tp_size,
                    num_heads=num_heads,
                    num_kv_heads=num_kv_heads,
                    max_position=max_position,
                    rope_theta=rope_theta,
                    is_neox_style=is_neox_style,
                    rope_scaling=config.rope_scaling,
                    dtype=getattr(torch, args.dtype),
                    seed=args.seed,
                    warmup_iter=args.warmup_iter,
                    benchmark_iter=args.benchmark_iter,
                )
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `get_model_config`, `generate_test_data`, `calculate_stats`, `benchmark_mrope`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具, CLI argument parsing / 命令行参数解析
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.srt.layers.rotary_embedding`
- **External / 外部**: `argparse`, `numpy`, `time`, `torch`, `transformers`, `typing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
