# bench_triton_swa_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/sliding_window_attention_triton/bench_triton_swa_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels sliding window attention triton. It primarily measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. / 该 Python 模块聚焦于 kernels sliding window attention triton 相关流程。它主要用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and setup / 导入与初始化
```python
import itertools

import torch
import torch.nn.functional as F
import triton.testing as tt

from sglang.benchmark.bench_utils import run_bench
from sglang.srt.layers.attention.triton_ops.extend_attention import extend_attention_fwd
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 11-82: Function `extend_attention_fwd_torch` / 函数 `extend_attention_fwd_torch`
```python
def extend_attention_fwd_torch(
    q: torch.Tensor,  # [extend_tokens, H_Q, D]
    k: torch.Tensor,  # [extend_tokens, H_KV, D]
    v: torch.Tensor,  # [extend_tokens, H_KV, D]
    o: torch.Tensor,  # [extend_tokens, H_Q, D]
    k_cache: torch.Tensor,  # [total_tokens, H_KV, D]
    v_cache: torch.Tensor,  # [total_tokens, H_KV, D]
    qo_indptr: torch.Tensor,  # [B+1]
    kv_indptr: torch.Tensor,  # [B+1]
    kv_indices: torch.Tensor,  # [prefix_tokens]
    sliding_window_size: int,
):
    B = qo_indptr.size(0) - 1
    _, H_Q, D = q.shape
    _, H_KV, _ = k.shape

    group_size = H_Q // H_KV
    scale = 1.0 / D**0.5

    for i in range(B):
        q_start = int(qo_indptr[i].item())
        q_end = int(qo_indptr[i + 1].item())
        kv_start = int(kv_indptr[i].item())
        kv_end = int(kv_indptr[i + 1].item())

        prefix_indices = kv_indices[kv_start:kv_end]
        k_prefix = k_cache[prefix_indices]  # [prefix_len, H_KV, D]
        v_prefix = v_cache[prefix_indices]  # [prefix_len, H_KV, D]

        k_extend = k[q_start:q_end]  # [extend_len, H_KV, D]
        v_extend = v[q_start:q_end]  # [extend_len, H_KV, D]
        q_extend = q[q_start:q_end]  # [extend_len, H_Q,  D]

        k_full = torch.cat([k_prefix, k_extend], dim=0)  # [total_len, H_KV, D]
        v_full = torch.cat([v_prefix, v_extend], dim=0)  # [total_len, H_KV, D]

        if group_size != 1:
            k_full_hq = k_full.repeat_interleave(
                group_size, dim=1
            )  # [total_len, H_Q, D]
            v_full_hq = v_full.repeat_interleave(
                group_size, dim=1
            )  # [total_len, H_Q, D]
        else:
            k_full_hq = k_full
            v_full_hq = v_full

        prefix_len = k_prefix.size(0)
        extend_len = k_extend.size(0)
        total_len = prefix_len + extend_len

        # causal
        pos_keys = torch.arange(total_len, device=q.device)
        t = prefix_len + torch.arange(extend_len, device=q.device)  # [extend_len]
        causal_mask = pos_keys.unsqueeze(0) <= t.unsqueeze(1)

        # sliding window
        if sliding_window_size is not None and sliding_window_size > 0:
            start = (t - (sliding_window_size)).clamp_min(0)  # [extend_len]
        else:
            start = torch.zeros_like(t)
        window_mask = pos_keys.unsqueeze(0) >= start.unsqueeze(1)

        final_mask = causal_mask & window_mask

        attn_scores = (
            torch.einsum("qhd,khd->qhk", q_extend, k_full_hq) * scale
        )  # [extend_len, H_Q, total_len]
        attn_scores = attn_scores.masked_fill(~final_mask.unsqueeze(1), float("-inf"))

        attn_weights = F.softmax(attn_scores, dim=-1)
        o[q_start:q_end] = torch.einsum("qhk,khd->qhd", attn_weights, v_full_hq)
```
**EN:** `extend_attention_fwd_torch` is a function that prepares tensors and invokes GPU kernels, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. Notable calls include `range`, `qo_indptr.size`, `int`.
**CN:** `extend_attention_fwd_torch` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。其中较关键的调用包括 `range`, `qo_indptr.size`, `int`。

### Lines 85-174: Function `_build_batch` / 函数 `_build_batch`
```python
def _build_batch(
    B, N_CTX, H_Q, H_KV, D, WINDOW_SIZE, dtype=torch.bfloat16, device="cuda"
):
    b_seq_len_prefix = torch.randint(
        1, max(2, N_CTX // 2), (B,), dtype=torch.int32, device=device
    )
    b_seq_len_extend = torch.randint(
        1, max(2, N_CTX // 2), (B,), dtype=torch.int32, device=device
    )
    b_seq_len = b_seq_len_prefix + b_seq_len_extend

    b_start_loc = torch.zeros((B,), dtype=torch.int32, device=device)
    b_start_loc[1:] = torch.cumsum(b_seq_len[:-1], 0)
    b_start_loc_extend = torch.zeros((B,), dtype=torch.int32, device=device)
    b_start_loc_extend[1:] = torch.cumsum(b_seq_len_extend[:-1], 0)

    kv_indptr = torch.zeros((B + 1,), dtype=torch.int32, device=device)
    kv_indptr[1 : B + 1] = torch.cumsum(b_seq_len_prefix[:B], dim=0)

    kv_indices = torch.zeros(
        (int(b_seq_len_prefix.sum().item()),), dtype=torch.int32, device=device
    )
    for i in range(B):
        s = kv_indptr[i].item()
        e = kv_indptr[i + 1].item()
        kv_indices[s:e] = torch.arange(
            b_start_loc[i],
            b_start_loc[i] + b_seq_len_prefix[i],
            dtype=torch.int32,
            device=device,
        )

    total_token_num = int(torch.sum(b_seq_len).item())
    extend_token_num = int(torch.sum(b_seq_len_extend).item())

    k_buffer = torch.empty(
        (total_token_num, H_KV, D), dtype=dtype, device=device
    ).normal_(mean=0.1, std=0.2)
    v_buffer = torch.empty(
        (total_token_num, H_KV, D), dtype=dtype, device=device
    ).normal_(mean=0.1, std=0.2)

    k_extend = torch.empty((extend_token_num, H_KV, D), dtype=dtype, device=device)
    v_extend = torch.empty((extend_token_num, H_KV, D), dtype=dtype, device=device)
    q_extend = torch.empty((extend_token_num, H_Q, D), dtype=dtype, device=device)

    for i in range(B):
        extend_start_in_buffer = b_start_loc[i] + b_seq_len_prefix[i]
        extend_end_in_buffer = b_start_loc[i] + b_seq_len[i]
        extend_start = b_start_loc_extend[i]
        extend_end = b_start_loc_extend[i] + b_seq_len_extend[i]

        k_extend[extend_start:extend_end] = k_buffer[
            extend_start_in_buffer:extend_end_in_buffer
        ]
        v_extend[extend_start:extend_end] = v_buffer[
            extend_start_in_buffer:extend_end_in_buffer
        ]
        q_extend[extend_start:extend_end] = torch.empty(
            (int(b_seq_len_extend[i].item()), H_Q, D), dtype=dtype, device=device
        ).normal_(mean=0.1, std=0.2)

    o_extend_triton = torch.empty(
        (extend_token_num, H_Q, D), dtype=dtype, device=device
    )
    o_extend_torch = torch.empty((extend_token_num, H_Q, D), dtype=dtype, device=device)

    b_seq_len_extend = b_seq_len - b_seq_len_prefix
    max_len_extend = int(torch.max(b_seq_len_extend, 0)[0].item())
    qo_indptr = torch.zeros((B + 1,), dtype=torch.int32, device=device)
    qo_indptr[1 : B + 1] = torch.cumsum(b_seq_len_extend[:B], dim=0)

    inputs = dict(
        q_extend=q_extend,
        k_extend=k_extend,
        v_extend=v_extend,
        k_buffer=k_buffer,
        v_buffer=v_buffer,
        o_extend_triton=o_extend_triton,
        o_extend_torch=o_extend_torch,
        qo_indptr=qo_indptr,
        kv_indptr=kv_indptr,
        kv_indices=kv_indices,
        max_len_extend=max_len_extend,
        WINDOW_SIZE=WINDOW_SIZE,
    )
    meta = dict(
        B=B, N_CTX=N_CTX, H_Q=H_Q, H_KV=H_KV, D=D, extend_token_num=extend_token_num
    )
    return inputs, meta
```
**EN:** `_build_batch` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `(inputs, meta)` to the caller. Notable calls include `torch.randint`, `torch.zeros`, `torch.cumsum`.
**CN:** `_build_batch` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(inputs, meta)`。其中较关键的调用包括 `torch.randint`, `torch.zeros`, `torch.cumsum`。

### Lines 177-193: Function `_run_triton` / 函数 `_run_triton`
```python
def _run_triton(inputs):
    extend_attention_fwd(
        inputs["q_extend"],
        inputs["k_extend"],
        inputs["v_extend"],
        inputs["o_extend_triton"],
        inputs["k_buffer"],
        inputs["v_buffer"],
        inputs["qo_indptr"],
        inputs["kv_indptr"],
        inputs["kv_indices"],
        custom_mask=None,
        is_causal=True,
        mask_indptr=None,
        max_len_extend=inputs["max_len_extend"],
        sliding_window_size=inputs["WINDOW_SIZE"],
    )
```
**EN:** `_run_triton` is a function that prepares tensors and invokes GPU kernels. Notable calls include `extend_attention_fwd`.
**CN:** `_run_triton` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `extend_attention_fwd`。

### Lines 196-208: Function `_run_torch_ref` / 函数 `_run_torch_ref`
```python
def _run_torch_ref(inputs):
    extend_attention_fwd_torch(
        inputs["q_extend"],
        inputs["k_extend"],
        inputs["v_extend"],
        inputs["o_extend_torch"],
        inputs["k_buffer"],
        inputs["v_buffer"],
        inputs["qo_indptr"],
        inputs["kv_indptr"],
        inputs["kv_indices"],
        inputs["WINDOW_SIZE"],
    )
```
**EN:** `_run_torch_ref` is a function that prepares tensors and invokes GPU kernels. Notable calls include `extend_attention_fwd_torch`.
**CN:** `_run_torch_ref` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `extend_attention_fwd_torch`。

### Lines 211-216: Top-level execution logic / 顶层执行逻辑
```python
N_CTXS = [1024, 2048, 4096, 8192]
WINDOW_SIZES = [-1, 127, 256, 512]

CONFIGS = list(itertools.product(N_CTXS, WINDOW_SIZES))

PROVIDERS = ["torch", "triton"]
```
**EN:** This block performs top-level coordination such as preparing inputs, selecting modes, or invoking previously defined helpers. In practice it prepares tensors and invokes GPU kernels.
**CN:** 该代码块负责顶层协调，例如准备输入、选择模式或调用前面定义的辅助函数。 从实现上看，它会准备张量并调用 GPU 内核。

### Lines 241-290: Function `bench` / 函数 `bench`
```python
def bench(
    N_CTX,
    provider,
    B,
    H_Q,
    H_KV,
    D,
    dtype,
    device,
    WINDOW_SIZE,
    check_correctness,
    warmup,
    rep,
):
    torch.manual_seed(0)
    torch.cuda.manual_seed(0)
    dtype_map = {"bf16": torch.bfloat16, "fp16": torch.float16, "fp32": torch.float32}
    dt = dtype_map[dtype]

    inputs, _ = _build_batch(
        B, N_CTX, H_Q, H_KV, D, WINDOW_SIZE, dtype=dt, device=device
    )

    if check_correctness and provider == "triton":
        _run_triton(inputs)
        _run_torch_ref(inputs)
        torch.cuda.synchronize()
        if not torch.allclose(
            inputs["o_extend_triton"], inputs["o_extend_torch"], rtol=1e-3, atol=1e-3
        ):
            raise AssertionError("Mismatch between triton and torch reference.")

    if provider == "triton":
        ms = run_bench(
            lambda: _run_triton(inputs),
            quantiles=None,
            warmup_ms=warmup,
            rep_ms=rep,
        )[0]
    elif provider == "torch":
        ms = run_bench(
            lambda: _run_torch_ref(inputs),
            quantiles=None,
            warmup_ms=warmup,
            rep_ms=rep,
        )[0]
    else:
        raise ValueError(provider)

    return ms
```
**EN:** `bench` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and computes evaluation scores and aggregate statistics. It returns `ms` to the caller. Notable calls include `tt.perf_report`, `torch.manual_seed`, `torch.cuda.manual_seed`.
**CN:** `bench` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、计算评测分数与聚合统计结果。它会向调用方返回 `ms`。其中较关键的调用包括 `tt.perf_report`, `torch.manual_seed`, `torch.cuda.manual_seed`。

### Lines 293-294: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    bench.run(print_data=True, show_plots=False)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`
- **Third-party / 第三方依赖**: `torch`, `torch.nn.functional`, `triton.testing`
- **Internal / 项目内部依赖**: `sglang.benchmark.bench_utils`, `sglang.srt.layers.attention.triton_ops.extend_attention`
