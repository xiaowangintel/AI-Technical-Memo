# benchmark_torch_compile_fused_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/fused_moe_triton/benchmark_torch_compile_fused_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels fused moe triton torch. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. / 该 Python 模块聚焦于 kernels fused moe triton torch 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

## Line-by-Line Analysis / 逐行分析
### Lines 2-13: Imports and setup / 导入与初始化
```python
import argparse

import torch
import triton
from torch.nn import functional as F
from transformers import AutoConfig

from sglang.benchmark.bench_utils import run_bench
from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import (
    fused_moe as fused_moe_triton,
)
from sglang.srt.model_executor.cuda_graph_runner import set_torch_compile_config
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

### Lines 16-74: Function `get_model_config` / 函数 `get_model_config`
```python
def get_model_config(model_name: str, tp_size: int):
    """Get model configuration parameters"""
    config = AutoConfig.from_pretrained(model_name, trust_remote_code=True)

    if config.architectures[0] == "DbrxForCausalLM":
        E = config.ffn_config.moe_num_experts
        topk = config.ffn_config.moe_top_k
        intermediate_size = config.ffn_config.ffn_hidden_size
        shard_intermediate_size = 2 * intermediate_size // tp_size
    elif config.architectures[0] == "JambaForCausalLM":
        E = config.num_experts
        topk = config.num_experts_per_tok
        intermediate_size = config.intermediate_size
        shard_intermediate_size = 2 * intermediate_size // tp_size
    elif config.architectures[0] == "Qwen2MoeForCausalLM":
        E = config.num_experts
        topk = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size
        shard_intermediate_size = 2 * intermediate_size // tp_size
    elif config.architectures[0] == "Qwen3MoeForCausalLM":
        E = config.n_routed_experts
        topk = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size
        shard_intermediate_size = 2 * intermediate_size // tp_size
    elif config.architectures[0] in ["DeepseekV2ForCausalLM", "DeepseekV3ForCausalLM"]:
        E = config.n_routed_experts
        topk = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size
        shard_intermediate_size = 2 * intermediate_size // tp_size
    elif config.architectures[0] == "Llama4ForConditionalGeneration":
        E = config.text_config.num_local_experts
        topk = config.text_config.num_experts_per_tok
        intermediate_size = config.text_config.intermediate_size
        shard_intermediate_size = 2 * intermediate_size // tp_size
    elif config.architectures[0] in [
        "Grok1ForCausalLM",
        "Grok1ImgGen",
        "Grok1AForCausalLM",
    ]:
        E = config.num_local_experts
        topk = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size
        shard_intermediate_size = 2 * intermediate_size // tp_size
    else:
        # Default: Mixtral
        E = config.num_local_experts
        topk = config.num_experts_per_tok
        intermediate_size = config.intermediate_size
        shard_intermediate_size = 2 * intermediate_size // tp_size

    shape_configs = {
        "num_experts": E,
        "topk": topk,
        "hidden_size": config.hidden_size,
        "shard_intermediate_size": shard_intermediate_size,
        "dtype": config.torch_dtype,
    }
    print(f"{shape_configs=}")
    return shape_configs
```
**EN:** `get_model_config` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Get model configuration parameters It returns `shape_configs` to the caller. Notable calls include `AutoConfig.from_pretrained`, `print`.
**CN:** `get_model_config` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `shape_configs`。其中较关键的调用包括 `AutoConfig.from_pretrained`, `print`。

### Lines 77-93: Function `fused_topk_native` / 函数 `fused_topk_native`
```python
def fused_topk_native(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
):
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"
    M, _ = hidden_states.shape
    topk_weights = torch.empty(
        M, topk, dtype=torch.float32, device=hidden_states.device
    )
    topk_ids = torch.empty(M, topk, dtype=torch.int32, device=hidden_states.device)
    topk_weights = F.softmax(gating_output.float(), dim=-1)
    topk_weights, topk_ids = torch.topk(topk_weights, topk, dim=-1)
    if renormalize:
        topk_weights = topk_weights / topk_weights.sum(dim=-1, keepdim=True)
    return topk_weights, topk_ids
```
**EN:** `fused_topk_native` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `(topk_weights, topk_ids)` to the caller. Notable calls include `torch.empty`, `F.softmax`, `torch.topk`.
**CN:** `fused_topk_native` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(topk_weights, topk_ids)`。其中较关键的调用包括 `torch.empty`, `F.softmax`, `torch.topk`。

### Lines 97-124: Function `fused_moe_torch` / 函数 `fused_moe_torch`
```python
def fused_moe_torch(
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
) -> torch.Tensor:
    assert not use_fp8_w8a8, "Fp8_w8a8 fused_moe is not supported for torch compile"

    topk_weights, topk_ids = fused_topk_native(
        hidden_states=x,
        gating_output=input_gating,
        topk=topk,
        renormalize=True,
    )
    w13_weights = w1[topk_ids]
    w1_weights, w3_weights = torch.chunk(w13_weights, 2, dim=2)
    w2_weights = w2[topk_ids]
    x1 = torch.einsum("ti,taoi -> tao", x, w1_weights)
    x1 = F.silu(x1)
    x3 = torch.einsum("ti, taoi -> tao", x, w3_weights)
    expert_outs = torch.einsum("tao, taio -> tai", (x1 * x3), w2_weights)
    return torch.einsum("tai,ta -> ti", expert_outs, topk_weights.to(expert_outs.dtype))
```
**EN:** `fused_moe_torch` is a function that prepares tensors and invokes GPU kernels. It returns `torch.einsum('tai,ta -> ti', expert_outs, topk_weights.to(expert_outs.dtype))` to the caller. Notable calls include `torch.compile`, `fused_topk_native`, `torch.chunk`.
**CN:** `fused_moe_torch` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `torch.einsum('tai,ta -> ti', expert_outs, topk_weights.to(expert_outs.dtype))`。其中较关键的调用包括 `torch.compile`, `fused_topk_native`, `torch.chunk`。

### Lines 127-150: Function `fused_moe_torch_compile` / 函数 `fused_moe_torch_compile`
```python
def fused_moe_torch_compile(
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
):
    return fused_moe_torch(
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
    )
```
**EN:** `fused_moe_torch_compile` is a function that prepares tensors and invokes GPU kernels. It returns `fused_moe_torch(x, w1, w2, input_gating, topk, use_fp8_w8a8=use_fp8_w8a8, w1_scale=w1_scale, w2_s...` to the caller. Notable calls include `fused_moe_torch`.
**CN:** `fused_moe_torch_compile` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `fused_moe_torch(x, w1, w2, input_gating, topk, use_fp8_w8a8=use_fp8_w8a8, w1_scale=w1_scale, w2_s...`。其中较关键的调用包括 `fused_moe_torch`。

### Lines 153-178: Function `fused_moe_sglang_api` / 函数 `fused_moe_sglang_api`
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
):
    return fused_moe_triton(
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
**EN:** `fused_moe_sglang_api` is a function that prepares tensors and invokes GPU kernels. It returns `fused_moe_triton(x, w1, w2, input_gating, topk, renormalize=True, inplace=True, use_fp8_w8a8=use_...` to the caller. Notable calls include `fused_moe_triton`.
**CN:** `fused_moe_sglang_api` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `fused_moe_triton(x, w1, w2, input_gating, topk, renormalize=True, inplace=True, use_fp8_w8a8=use_...`。其中较关键的调用包括 `fused_moe_triton`。

### Lines 203-278: Function `benchmark` / 函数 `benchmark`
```python
def benchmark(batch_size, provider, model_config, use_fp8_w8a8=False):
    print(f"benchmark {provider} with batch_size={batch_size}")
    torch.set_default_device("cuda")
    torch.cuda.manual_seed_all(0)
    set_torch_compile_config()

    num_tokens = batch_size
    num_experts = model_config["num_experts"]
    hidden_size = model_config["hidden_size"]
    shard_intermediate_size = model_config["shard_intermediate_size"]
    topk = model_config["topk"]
    dtype = model_config["dtype"]

    x = torch.randn(num_tokens, hidden_size, dtype=dtype)

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
        w1_scale = torch.randn(num_experts, dtype=torch.float32)
        w2_scale = torch.randn(num_experts, dtype=torch.float32)
        a1_scale = torch.randn(1, dtype=torch.float32)
        a2_scale = torch.randn(1, dtype=torch.float32)
    else:
        w1 = torch.randn(num_experts, shard_intermediate_size, hidden_size, dtype=dtype)
        w2 = torch.randn(
            num_experts, hidden_size, shard_intermediate_size // 2, dtype=dtype
        )
        w1_scale = w2_scale = a1_scale = a2_scale = None

    input_gating = torch.randn(num_tokens, num_experts, dtype=torch.float32)

    # Warmup
    api_func = (
        fused_moe_torch_compile
        if provider == "fused_moe_torch_compile"
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
        )[0],
        quantiles=quantiles,
    )
    return ms, min_ms, max_ms
```
**EN:** `benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `(ms, min_ms, max_ms)` to the caller. Notable calls include `triton.testing.perf_report`, `print`, `torch.set_default_device`.
**CN:** `benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(ms, min_ms, max_ms)`。其中较关键的调用包括 `triton.testing.perf_report`, `print`, `torch.set_default_device`。

### Lines 281-302: Function `main` / 函数 `main`
```python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--model", type=str, default="mistralai/Mixtral-8x7B-Instruct-v0.1"
    )
    parser.add_argument("--tp-size", type=int, default=2)
    parser.add_argument("--use-fp8-w8a8", action="store_true")
    parser.add_argument(
        "--save-path",
        type=str,
        default="./configs/benchmark_ops/fused_moe_torch_compile/",
    )
    args = parser.parse_args()

    model_config = get_model_config(args.model, args.tp_size)
    benchmark.run(
        show_plots=True,
        print_data=True,
        save_path=args.save_path,
        model_config=model_config,
        use_fp8_w8a8=args.use_fp8_w8a8,
    )
```
**EN:** `main` is a function that builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`.
**CN:** `main` 是一个函数，用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`。

### Lines 305-306: CLI entry point / 命令行入口
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
- **Third-party / 第三方依赖**: `torch`, `triton`, `torch.nn`, `transformers`
- **Internal / 项目内部依赖**: `sglang.benchmark.bench_utils`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe`, `sglang.srt.model_executor.cuda_graph_runner`
