# benchmark_vllm_vs_sglang_fused_moe_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/fused_moe_triton/benchmark_vllm_vs_sglang_fused_moe_triton.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels fused moe triton vllm. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. / 该 Python 模块聚焦于 kernels fused moe triton vllm 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

## Line-by-Line Analysis / 逐行分析
### Lines 2-19: Imports and setup / 导入与初始化
```python
import argparse

import torch
import triton
from vllm.model_executor.layers.fused_moe.fused_moe import fused_moe as fused_moe_vllm

from sglang.benchmark.bench_utils import run_bench
from sglang.srt.distributed.parallel_state import (
    destroy_distributed_environment,
    destroy_model_parallel,
    init_distributed_environment,
    initialize_model_parallel,
)
from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import (
    fused_moe as fused_moe_sglang,
)

from .common_utils import get_model_config
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

### Lines 22-65: Function `fused_moe_vllm_api` / 函数 `fused_moe_vllm_api`
```python
def fused_moe_vllm_api(
    x,
    w1,
    w2,
    input_gating,
    topk,
    use_fp8_w8a8=False,
    w1_scale=None,
    w2_scale=None,
    a1_scale=None,
    a2_scale=None,
    block_shape=None,
):
    if block_shape is not None:
        return fused_moe_vllm(
            x,
            w1,
            w2,
            input_gating,
            topk,
            renormalize=True,
            inplace=True,
            use_fp8_w8a8=use_fp8_w8a8,
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            a1_scale=a1_scale,
            a2_scale=a2_scale,
            block_shape=block_shape,
        )
    else:
        return fused_moe_vllm(
            x,
            w1,
            w2,
            input_gating,
            topk,
            renormalize=True,
            inplace=True,
            use_fp8_w8a8=use_fp8_w8a8,
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            a1_scale=a1_scale,
            a2_scale=a2_scale,
        )
```
**EN:** `fused_moe_vllm_api` is a function that implements the core logic for this scope. It returns `fused_moe_vllm(x, w1, w2, input_gating, topk, renormalize=True, inplace=True, use_fp8_w8a8=use_fp...` to the caller. Notable calls include `fused_moe_vllm`.
**CN:** `fused_moe_vllm_api` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `fused_moe_vllm(x, w1, w2, input_gating, topk, renormalize=True, inplace=True, use_fp8_w8a8=use_fp...`。其中较关键的调用包括 `fused_moe_vllm`。

### Lines 68-95: Function `fused_moe_sglang_api` / 函数 `fused_moe_sglang_api`
```python
def fused_moe_sglang_api(
    x,
    w1,
    w2,
    input_gating,
    topk,
    use_fp8_w8a8=False,
    w1_scale=None,
    w2_scale=None,
    a1_scale=None,
    a2_scale=None,
    block_shape=None,
):
    return fused_moe_sglang(
        x,
        w1,
        w2,
        input_gating,
        topk,
        renormalize=True,
        inplace=True,
        use_fp8_w8a8=use_fp8_w8a8,
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        a1_scale=a1_scale,
        a2_scale=a2_scale,
        block_shape=block_shape,
    )
```
**EN:** `fused_moe_sglang_api` is a function that implements the core logic for this scope. It returns `fused_moe_sglang(x, w1, w2, input_gating, topk, renormalize=True, inplace=True, use_fp8_w8a8=use_...` to the caller. Notable calls include `fused_moe_sglang`.
**CN:** `fused_moe_sglang_api` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `fused_moe_sglang(x, w1, w2, input_gating, topk, renormalize=True, inplace=True, use_fp8_w8a8=use_...`。其中较关键的调用包括 `fused_moe_sglang`。

### Lines 120-211: Function `benchmark` / 函数 `benchmark`
```python
def benchmark(batch_size, provider, model_config, use_fp8_w8a8=False):
    print(f"benchmark {provider} with batch_size={batch_size}")
    torch.set_default_device("cuda")
    torch.cuda.manual_seed_all(0)

    num_tokens = batch_size
    num_experts = model_config["num_experts"]
    hidden_size = model_config["hidden_size"]
    shard_intermediate_size = model_config["shard_intermediate_size"]
    topk = model_config["topk"]
    dtype = model_config["dtype"]
    block_shape = model_config["block_shape"]

    x = torch.randn(num_tokens, hidden_size, dtype=dtype)
    w1_scale = w2_scale = a1_scale = a2_scale = None

    if use_fp8_w8a8:
        init_dtype = dtype
        w1 = torch.randn(
            num_experts, shard_intermediate_size, hidden_size, dtype=init_dtype
        )
        w2 = torch.randn(
            num_experts, hidden_size, shard_intermediate_size // 2, dtype=init_dtype
        )
        w1 = w1.to(torch.float8_e4m3fn)
        w2 = w2.to(torch.float8_e4m3fn)

        if block_shape is None:
            w1_scale = torch.randn(num_experts, dtype=torch.float32)
            w2_scale = torch.randn(num_experts, dtype=torch.float32)
            a1_scale = torch.randn(1, dtype=torch.float32)
            a2_scale = torch.randn(1, dtype=torch.float32)
        else:
            block_n, block_k = block_shape[0], block_shape[1]
            n_tiles_w1 = (shard_intermediate_size + block_n - 1) // block_n
            n_tiles_w2 = (hidden_size + block_n - 1) // block_n
            k_tiles_w1 = (hidden_size + block_k - 1) // block_k
            k_tiles_w2 = (shard_intermediate_size // 2 + block_k - 1) // block_k
            w1_scale = torch.rand(
                (num_experts, n_tiles_w1, k_tiles_w1), dtype=torch.float32
            )
            w2_scale = torch.rand(
                (num_experts, n_tiles_w2, k_tiles_w2), dtype=torch.float32
            )
    else:
        w1 = torch.randn(num_experts, shard_intermediate_size, hidden_size, dtype=dtype)
        w2 = torch.randn(
            num_experts, hidden_size, shard_intermediate_size // 2, dtype=dtype
        )

    input_gating = torch.randn(num_tokens, num_experts, dtype=torch.float32)

    # Warmup
    api_func = (
        fused_moe_vllm_api
        if provider == "vllm_fused_moe_triton"
        else fused_moe_sglang_api
    )
    for _ in range(10):
        y = api_func(
            x,
            w1,
            w2,
            input_gating,
            topk,
            use_fp8_w8a8=use_fp8_w8a8,
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            a1_scale=a1_scale,
            a2_scale=a2_scale,
            block_shape=block_shape,
        )
    torch.cuda.synchronize()

    quantiles = (0.5, 0.2, 0.8)
    ms, min_ms, max_ms = run_bench(
        lambda: api_func(
            x,
            w1,
            w2,
            input_gating,
            topk,
            use_fp8_w8a8=use_fp8_w8a8,
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            a1_scale=a1_scale,
            a2_scale=a2_scale,
            block_shape=block_shape,
        )[0],
        quantiles=quantiles,
    )
    return ms, min_ms, max_ms
```
**EN:** `benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `(ms, min_ms, max_ms)` to the caller. Notable calls include `triton.testing.perf_report`, `print`, `torch.set_default_device`.
**CN:** `benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(ms, min_ms, max_ms)`。其中较关键的调用包括 `triton.testing.perf_report`, `print`, `torch.set_default_device`。

### Lines 214-261: Function `main` / 函数 `main`
```python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--model", type=str, default="mistralai/Mixtral-8x7B-Instruct-v0.1"
    )
    parser.add_argument("--tp-size", "--tp", type=int, default=2)
    parser.add_argument("--ep-size", "--ep", type=int, default=1)
    parser.add_argument("--use-fp8-w8a8", action="store_true")
    parser.add_argument(
        "--save-path",
        type=str,
        default="./configs/benchmark_ops/vllm_sglang_fused_moe/",
    )
    args = parser.parse_args()

    try:
        if not torch.distributed.is_initialized():
            torch.distributed.init_process_group(
                backend="nccl" if torch.cuda.is_available() else "gloo",
                init_method="tcp://127.0.0.1:23456",
                world_size=1,
                rank=0,
            )

        init_distributed_environment(
            world_size=1,
            rank=0,
            distributed_init_method="tcp://127.0.0.1:23456",
            local_rank=0,
            backend="nccl" if torch.cuda.is_available() else "gloo",
        )

        initialize_model_parallel(
            tensor_model_parallel_size=1,
            pipeline_model_parallel_size=1,
        )

        shape_configs = get_model_config(args.model, args.tp_size, args.ep_size)
        benchmark.run(
            show_plots=True,
            print_data=True,
            save_path=args.save_path,
            model_config=shape_configs,
            use_fp8_w8a8=args.use_fp8_w8a8,
        )
    finally:
        destroy_model_parallel()
        destroy_distributed_environment()
```
**EN:** `main` is a function that builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`.
**CN:** `main` 是一个函数，用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`。

### Lines 264-265: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    main()
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Orchestration / 编排**: Launches tools, captures logs, or stitches multi-step experiments together. / 启动工具、抓取日志或拼接多步实验流程。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`
- **Third-party / 第三方依赖**: `torch`, `triton`, `vllm.model_executor.layers.fused_moe.fused_moe`
- **Internal / 项目内部依赖**: `sglang.benchmark.bench_utils`, `sglang.srt.distributed.parallel_state`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe`, `.common_utils`
