# benchmark_concat_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/elementwise/benchmark_concat_mla.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels elementwise concat mla. It primarily measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. / 该 Python 模块聚焦于 kernels elementwise concat mla 相关流程。它主要用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and setup / 导入与初始化
```python
import torch
import triton
import triton.language as tl
from sgl_kernel import concat_mla_k as concat_mla_k_cuda

from sglang.benchmark.bench_utils import run_bench

DEVICE = triton.runtime.driver.active.get_active_torch_device()

num_local_heads = 128
qk_nope_head_dim = 128
qk_rope_head_dim = 64
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 15-33: Function `create_data` / 函数 `create_data`
```python
def create_data(num_tokens):
    k_nope_container = torch.randn(
        (num_tokens, num_local_heads, qk_nope_head_dim + 128),
        dtype=torch.bfloat16,
        device="cuda",
    )
    k_nope = k_nope_container[:, :, :qk_nope_head_dim]

    k_rope_container = torch.randn(
        (num_tokens, 1, 128 + qk_rope_head_dim), dtype=torch.bfloat16, device="cuda"
    )
    k_rope = k_rope_container[:, :, -qk_rope_head_dim:]

    k = torch.empty(
        (num_tokens, num_local_heads, qk_nope_head_dim + qk_rope_head_dim),
        dtype=torch.bfloat16,
        device="cuda",
    )
    return dict(k=k, k_nope=k_nope, k_rope=k_rope)
```
**EN:** `create_data` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `dict(k=k, k_nope=k_nope, k_rope=k_rope)` to the caller. Notable calls include `torch.randn`, `torch.empty`, `dict`.
**CN:** `create_data` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `dict(k=k, k_nope=k_nope, k_rope=k_rope)`。其中较关键的调用包括 `torch.randn`, `torch.empty`, `dict`。

### Lines 36-38: Function `fn_torch` / 函数 `fn_torch`
```python
def fn_torch(k, k_nope, k_rope):
    k[..., :qk_nope_head_dim] = k_nope
    k[..., qk_nope_head_dim:] = k_rope
```
**EN:** `fn_torch` is a function that prepares tensors and invokes GPU kernels.
**CN:** `fn_torch` 是一个函数，用于准备张量并调用 GPU 内核。

### Lines 41-46: Function `fn_hack_non_strided` / 函数 `fn_hack_non_strided`
```python
def fn_hack_non_strided(k, k_nope, k_rope):
    k_flatten_view = k.flatten()
    k_flatten_view[: k_nope.numel()] = k_nope.flatten()

    k2 = k_flatten_view[k_nope.numel() :].view(k_rope.numel(), -1)
    k2 = k_rope.flatten()[:, None]
```
**EN:** `fn_hack_non_strided` is a function that implements the core logic for this scope. Notable calls include `k.flatten`, `k_nope.flatten`, `k_flatten_view[k_nope.numel():].view`.
**CN:** `fn_hack_non_strided` 是一个函数，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `k.flatten`, `k_nope.flatten`, `k_flatten_view[k_nope.numel():].view`。

### Lines 50-51: Function `fn_torch_compiled` / 函数 `fn_torch_compiled`
```python
def fn_torch_compiled(k, k_nope, k_rope):
    return fn_torch(k, k_nope, k_rope)
```
**EN:** `fn_torch_compiled` is a function that prepares tensors and invokes GPU kernels. It returns `fn_torch(k, k_nope, k_rope)` to the caller. Notable calls include `torch.compile`, `fn_torch`.
**CN:** `fn_torch_compiled` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `fn_torch(k, k_nope, k_rope)`。其中较关键的调用包括 `torch.compile`, `fn_torch`。

### Lines 54-55: Function `fn_cuda` / 函数 `fn_cuda`
```python
def fn_cuda(k, k_nope, k_rope):
    concat_mla_k_cuda(k, k_nope, k_rope)
```
**EN:** `fn_cuda` is a function that prepares tensors and invokes GPU kernels. Notable calls include `concat_mla_k_cuda`.
**CN:** `fn_cuda` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `concat_mla_k_cuda`。

### Lines 59-106: Function `fn_triton_kernel` / 函数 `fn_triton_kernel`
```python
def fn_triton_kernel(
    k_ptr,
    k_nope_ptr,
    k_rope_ptr,
    num_tokens,
    QK_NOPE_HEAD_DIM: tl.constexpr,
    QK_ROPE_HEAD_DIM: tl.constexpr,
    NUM_LOCAL_HEADS: tl.constexpr,
    K_NOPE_STRIDE_0: tl.constexpr,
    K_NOPE_STRIDE_1: tl.constexpr,
    K_STRIDE_0: tl.constexpr,
    K_STRIDE_1: tl.constexpr,
    K_ROPE_STRIDE_0: tl.constexpr,
    BLOCK_ROWS: tl.constexpr,
):
    pid = tl.program_id(axis=0)

    token_id = pid * BLOCK_ROWS + tl.arange(0, BLOCK_ROWS)
    token_mask = token_id < num_tokens

    head_id = tl.arange(0, NUM_LOCAL_HEADS)

    # nope
    nope_sub_id = tl.arange(0, QK_NOPE_HEAD_DIM)
    offs_nope = (
        token_id[:, None, None] * K_NOPE_STRIDE_0
        + head_id[None, :, None] * K_NOPE_STRIDE_1
        + nope_sub_id[None, None, :]
    )
    offs_k = (
        token_id[:, None, None] * K_STRIDE_0
        + head_id[None, :, None] * K_STRIDE_1
        + nope_sub_id[None, None, :]
    )
    vals_nope = tl.load(k_nope_ptr + offs_nope, mask=token_mask[:, None, None])
    tl.store(k_ptr + offs_k, vals_nope, mask=token_mask[:, None, None])

    # rope
    rope_sub_id = tl.arange(0, QK_ROPE_HEAD_DIM)
    offs_rope = token_id[:, None, None] * K_ROPE_STRIDE_0 + rope_sub_id[None, None, :]
    offs_k = (
        token_id[:, None, None] * K_STRIDE_0
        + head_id[None, :, None] * K_STRIDE_1
        + rope_sub_id[None, None, :]
        + QK_NOPE_HEAD_DIM
    )
    vals_rope = tl.load(k_rope_ptr + offs_rope, mask=token_mask[:, None, None])
    tl.store(k_ptr + offs_k, vals_rope, mask=token_mask[:, None, None])
```
**EN:** `fn_triton_kernel` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. Notable calls include `tl.program_id`, `tl.arange`, `tl.load`.
**CN:** `fn_triton_kernel` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `tl.program_id`, `tl.arange`, `tl.load`。

### Lines 109-127: Function `fn_triton` / 函数 `fn_triton`
```python
def fn_triton(k, k_nope, k_rope):
    assert k.device == DEVICE and k_nope.device == DEVICE and k_rope.device == DEVICE
    num_tokens, _, _ = k.shape
    grid = lambda meta: (triton.cdiv(num_tokens, meta["BLOCK_ROWS"]),)
    fn_triton_kernel[grid](
        k,
        k_nope,
        k_rope,
        num_tokens,
        QK_NOPE_HEAD_DIM=qk_nope_head_dim,
        QK_ROPE_HEAD_DIM=qk_rope_head_dim,
        NUM_LOCAL_HEADS=num_local_heads,
        K_NOPE_STRIDE_0=k_nope.stride(0),
        K_NOPE_STRIDE_1=k_nope.stride(1),
        K_STRIDE_0=k.stride(0),
        K_STRIDE_1=k.stride(1),
        K_ROPE_STRIDE_0=k_rope.stride(0),
        BLOCK_ROWS=16,
    )
```
**EN:** `fn_triton` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. Notable calls include `fn_triton_kernel[grid]`, `triton.cdiv`, `k_nope.stride`.
**CN:** `fn_triton` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `fn_triton_kernel[grid]`, `triton.cdiv`, `k_nope.stride`。

### Lines 130-134: Function `execute_and_get_output` / 函数 `execute_and_get_output`
```python
def execute_and_get_output(f, data):
    data["k"].zero_()
    f(**data)
    assert data["k"].sum().item() != 0
    return data["k"].clone()
```
**EN:** `execute_and_get_output` is a function that implements the core logic for this scope. It returns `data['k'].clone()` to the caller. Notable calls include `data['k'].zero_`, `f`, `data['k'].clone`.
**CN:** `execute_and_get_output` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `data['k'].clone()`。其中较关键的调用包括 `data['k'].zero_`, `f`, `data['k'].clone`。

### Lines 137-149: Top-level execution logic / 顶层执行逻辑
```python
torch.manual_seed(0)
data = create_data(num_tokens=32768)
output_ref = execute_and_get_output(fn_torch, data)
output_exp = execute_and_get_output(fn_cuda, data)
# print(output_ref)
# print(output_exp)
if not torch.all(output_ref == output_exp):
    abs_delta = torch.abs(output_ref - output_exp)
    raise AssertionError(
        f"{output_ref=} {output_exp=} "
        f"{abs_delta=} "
        f"{torch.argwhere(abs_delta != 0.0)=} "
    )
```
**EN:** This block performs top-level coordination such as preparing inputs, selecting modes, or invoking previously defined helpers. In practice it prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs.
**CN:** 该代码块负责顶层协调，例如准备输入、选择模式或调用前面定义的辅助函数。 从实现上看，它会准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。

### Lines 182-193: Function `benchmark` / 函数 `benchmark`
```python
def benchmark(num_tokens, provider):
    data = create_data(num_tokens=num_tokens)
    quantiles = (0.5, 0.2, 0.8)
    fn = {
        "torch": fn_torch,
        "torch_compiled": fn_torch_compiled,
        "triton": fn_triton,
        "hack_non_strided": fn_hack_non_strided,
        "cuda": fn_cuda,
    }[provider]
    ms, min_ms, max_ms = run_bench(lambda: fn(**data), quantiles=quantiles)
    return ms, min_ms, max_ms
```
**EN:** `benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `(ms, min_ms, max_ms)` to the caller. Notable calls include `triton.testing.perf_report`, `create_data`, `run_bench`.
**CN:** `benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(ms, min_ms, max_ms)`。其中较关键的调用包括 `triton.testing.perf_report`, `create_data`, `run_bench`。

### Lines 196-198: Top-level execution logic / 顶层执行逻辑
```python
torch.cuda.cudart().cudaProfilerStart()
benchmark.run(print_data=True, show_plots=True)
torch.cuda.cudart().cudaProfilerStop()
```
**EN:** This block performs top-level coordination such as preparing inputs, selecting modes, or invoking previously defined helpers. In practice it measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels.
**CN:** 该代码块负责顶层协调，例如准备输入、选择模式或调用前面定义的辅助函数。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`, `triton`, `triton.language`, `sgl_kernel`
- **Internal / 项目内部依赖**: `sglang.benchmark.bench_utils`
