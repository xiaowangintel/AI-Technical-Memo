# bench_attention_sink_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/bench_attention_sink/bench_attention_sink_triton.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on attention sink triton. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 attention sink triton 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and setup / 导入与初始化
```python
import argparse

import torch
import triton

from sglang.srt.layers.attention.triton_ops.decode_attention import (
    decode_attention_fwd_grouped,
)
from sglang.srt.layers.attention.triton_ops.extend_attention import extend_attention_fwd

# gpt oss
head_num = 64
head_dim = 64
head_kv_num = 8
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。

### Lines 36-117: Function `benchmark_decode` / 函数 `benchmark_decode`
```python
def benchmark_decode(B, S, H_Q, H_KV, D):
    D_V = D
    dtype = torch.bfloat16
    seq_len = S
    total_tokens = B * seq_len
    device = torch.device("cuda")
    sm_scale = 1.0 / (D**0.5)
    max_kv_splits = 8
    num_kv_splits = torch.full((B,), 4, dtype=torch.int32, device="cuda")

    # q represents the new token being generated, one per batch
    q = torch.randn(B, H_Q, D, dtype=dtype, device="cuda")

    # k_buffer and v_buffer represent all previous tokens
    k_buffer = torch.randn(total_tokens, H_KV, D, dtype=dtype, device="cuda")
    v_buffer = torch.randn(total_tokens, H_KV, D, dtype=dtype, device="cuda")

    o = torch.zeros(B, H_Q, D_V, dtype=dtype, device="cuda")

    b_seq_len = torch.full((B,), seq_len, device="cuda")

    kv_indptr = torch.zeros((B + 1,), dtype=torch.int32, device="cuda")
    kv_indptr[1 : B + 1] = torch.cumsum(b_seq_len, dim=0)
    kv_indices = torch.arange(total_tokens, device="cuda")

    attn_logits1 = torch.empty(
        (B, H_Q, max_kv_splits, D_V),
        dtype=torch.float32,
        device="cuda",
    )
    attn_lse1 = torch.empty(
        (B, H_Q, max_kv_splits, D_V),
        dtype=torch.float32,
        device="cuda",
    )
    sink = torch.randn(H_Q, device=device, dtype=torch.float32)

    # warmup
    for _ in range(5):
        decode_attention_fwd_grouped(
            q,
            k_buffer,
            v_buffer,
            o,
            kv_indptr,
            kv_indices,
            attn_logits1,
            attn_lse1,
            num_kv_splits,
            max_kv_splits,
            sm_scale,
            logit_cap=0.0,
            sinks=sink,
        )

    # benchmark
    run_step = 500
    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)
    start_event.record()
    for _ in range(run_step):
        decode_attention_fwd_grouped(
            q,
            k_buffer,
            v_buffer,
            o,
            kv_indptr,
            kv_indices,
            attn_logits1,
            attn_lse1,
            num_kv_splits,
            max_kv_splits,
            sm_scale,
            logit_cap=0.0,
            sinks=sink,
        )
    end_event.record()
    end_event.synchronize()
    torch.cuda.synchronize()
    ms = start_event.elapsed_time(end_event) / run_step
    tflops = lambda ms: (2 * B * S * H_Q * D) * 1e-9 / ms  # must be causal
    return tflops(ms)
```
**EN:** `benchmark_decode` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `tflops(ms)` to the caller. Notable calls include `triton.testing.perf_report`, `torch.device`, `torch.full`.
**CN:** `benchmark_decode` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `tflops(ms)`。其中较关键的调用包括 `triton.testing.perf_report`, `torch.device`, `torch.full`。

### Lines 139-230: Function `benchmark_extend` / 函数 `benchmark_extend`
```python
def benchmark_extend(B, S, H_Q, H_KV, D):
    # S here represents N_CTX from the test
    dtype = torch.bfloat16
    device = "cuda"

    # Split S into prefix and extend lengths
    prefill_len = S // 2  # Similar to test's N_CTX // 2
    extend_len = S // 4  # Make extend length smaller than prefix

    # Calculate total tokens and extend tokens
    total_extend_tokens = B * extend_len
    total_prefix_tokens = B * prefill_len

    # Create query, key, value tensors for extension
    q_extend = torch.randn(total_extend_tokens, H_Q, D, dtype=dtype, device=device)
    k_extend = torch.randn(total_extend_tokens, H_KV, D, dtype=dtype, device=device)
    v_extend = torch.randn(total_extend_tokens, H_KV, D, dtype=dtype, device=device)
    o_extend = torch.empty_like(q_extend)

    # Create key-value buffers for prefix
    k_buffer = torch.randn(total_prefix_tokens, H_KV, D, dtype=dtype, device=device)
    v_buffer = torch.randn(total_prefix_tokens, H_KV, D, dtype=dtype, device=device)

    # Create index pointers
    qo_indptr = torch.arange(0, (B + 1) * extend_len, extend_len, device=device).to(
        torch.int32
    )
    kv_indptr = torch.arange(0, (B + 1) * prefill_len, prefill_len, device=device).to(
        torch.int32
    )
    kv_indices = torch.arange(0, total_prefix_tokens, device=device).to(torch.int32)

    sm_scale = 1.0 / (D**0.5)
    # sliding_window = 128  # From GPT-OSS config, skip for now
    sliding_window = -1

    sink = torch.randn(H_Q, device=device, dtype=torch.float32)

    # warmup
    for _ in range(5):
        extend_attention_fwd(
            q_extend,
            k_extend,
            v_extend,
            o_extend,
            k_buffer,
            v_buffer,
            qo_indptr,
            kv_indptr,
            kv_indices,
            custom_mask=None,
            is_causal=True,
            mask_indptr=None,
            max_len_extend=extend_len,
            sm_scale=sm_scale,
            sliding_window_size=sliding_window,
            sinks=sink,
        )

    # benchmark
    run_step = 500
    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)
    start_event.record()
    for _ in range(run_step):
        extend_attention_fwd(
            q_extend,
            k_extend,
            v_extend,
            o_extend,
            k_buffer,
            v_buffer,
            qo_indptr,
            kv_indptr,
            kv_indices,
            custom_mask=None,
            is_causal=True,
            mask_indptr=None,
            max_len_extend=extend_len,
            sm_scale=sm_scale,
            sliding_window_size=sliding_window,
            sinks=sink,
        )
    end_event.record()
    end_event.synchronize()
    torch.cuda.synchronize()
    ms = start_event.elapsed_time(end_event) / run_step

    # FLOPS calculation: each attention operation requires 2 multiplications per element
    total_flops = 2 * total_extend_tokens * H_Q * (prefill_len + extend_len / 2) * D
    tflops = lambda ms: total_flops * 1e-12 / (ms * 1e-3)  # convert to TFLOPS
    return tflops(ms)
```
**EN:** `benchmark_extend` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `tflops(ms)` to the caller. Notable calls include `triton.testing.perf_report`, `torch.randn`, `torch.empty_like`.
**CN:** `benchmark_extend` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `tflops(ms)`。其中较关键的调用包括 `triton.testing.perf_report`, `torch.randn`, `torch.empty_like`。

### Lines 233-250: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--bench", type=str, default="all", help="all, extend, decode")
    args = parser.parse_args()

    kwargs = {
        "H_Q": head_num,
        "H_KV": head_kv_num,
        "D": head_dim,
    }

    if args.bench in ["all", "decode"]:
        benchmark_decode.run(print_data=True, show_plots=False, **kwargs)

    if args.bench in ["all", "extend"]:
        benchmark_extend.run(print_data=True, show_plots=False, **kwargs)

    print("Benchmark finished!")
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`
- **Third-party / 第三方依赖**: `torch`, `triton`
- **Internal / 项目内部依赖**: `sglang.srt.layers.attention.triton_ops.decode_attention`, `sglang.srt.layers.attention.triton_ops.extend_attention`
