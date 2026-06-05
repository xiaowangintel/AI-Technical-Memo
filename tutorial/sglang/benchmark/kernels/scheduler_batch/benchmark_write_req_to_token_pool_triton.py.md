# benchmark_write_req_to_token_pool_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/scheduler_batch/benchmark_write_req_to_token_pool_triton.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels scheduler batch write req. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 kernels scheduler batch write req 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and setup / 导入与初始化
```python
import itertools
import os

import torch
import triton
import triton.language as tl

from sglang.benchmark.bench_utils import run_bench
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 12-45: Function `write_req_to_token_pool_triton` / 函数 `write_req_to_token_pool_triton`
```python
def write_req_to_token_pool_triton(
    req_to_token_ptr,  # [max_batch, max_context_len]
    req_pool_indices,
    pre_lens,
    seq_lens,
    extend_lens,
    out_cache_loc,
    req_to_token_ptr_stride: tl.constexpr,
):
    BLOCK_SIZE: tl.constexpr = 512
    pid = tl.program_id(0)

    req_pool_index = tl.load(req_pool_indices + pid)
    pre_len = tl.load(pre_lens + pid)
    seq_len = tl.load(seq_lens + pid)

    # TODO: optimize this?
    cumsum_start = 0
    for i in range(pid):
        cumsum_start += tl.load(extend_lens + i)

    num_loop = tl.cdiv(seq_len - pre_len, BLOCK_SIZE)
    for i in range(num_loop):
        offset = tl.arange(0, BLOCK_SIZE) + i * BLOCK_SIZE
        mask = offset < (seq_len - pre_len)
        value = tl.load(out_cache_loc + cumsum_start + offset, mask=mask)
        tl.store(
            req_to_token_ptr
            + req_pool_index * req_to_token_ptr_stride
            + offset
            + pre_len,
            value,
            mask=mask,
        )
```
**EN:** `write_req_to_token_pool_triton` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. Notable calls include `tl.program_id`, `tl.load`, `range`.
**CN:** `write_req_to_token_pool_triton` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `tl.program_id`, `tl.load`, `range`。

### Lines 49-88: Function `write_req_to_token_pool_triton_optimize` / 函数 `write_req_to_token_pool_triton_optimize`
```python
def write_req_to_token_pool_triton_optimize(
    req_to_token_ptr,  # [max_batch, max_context_len]
    req_pool_indices,
    pre_lens,
    seq_lens,
    extend_lens,
    out_cache_loc,
    req_to_token_ptr_stride: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    pid_batch = tl.program_id(0)
    pid_token = tl.program_id(1)

    req_pool_index = tl.load(req_pool_indices + pid_batch)
    pre_len = tl.load(pre_lens + pid_batch)
    seq_len = tl.load(seq_lens + pid_batch)
    extend_len = seq_len - pre_len

    cumsum_start = 0
    for i in range(pid_batch):
        cumsum_start += tl.load(extend_lens + i)

    token_start = pid_token * BLOCK_SIZE

    offset = tl.arange(0, BLOCK_SIZE)
    actual_offset = token_start + offset
    mask = actual_offset < extend_len

    src_ptr = out_cache_loc + cumsum_start + actual_offset
    src_ptr = tl.max_contiguous(tl.multiple_of(src_ptr, BLOCK_SIZE), BLOCK_SIZE)
    value = tl.load(src_ptr, mask=mask)
    dst_ptr = (
        req_to_token_ptr
        + req_pool_index * req_to_token_ptr_stride
        + actual_offset
        + pre_len
    )
    dst_ptr = tl.max_contiguous(tl.multiple_of(dst_ptr, BLOCK_SIZE), BLOCK_SIZE)

    tl.store(dst_ptr, value, mask=mask)
```
**EN:** `write_req_to_token_pool_triton_optimize` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. Notable calls include `tl.program_id`, `tl.load`, `range`.
**CN:** `write_req_to_token_pool_triton_optimize` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `tl.program_id`, `tl.load`, `range`。

### Lines 91-111: Function `write_req_to_token_pool_reference` / 函数 `write_req_to_token_pool_reference`
```python
def write_req_to_token_pool_reference(
    req_to_token: torch.Tensor,
    req_pool_indices: torch.Tensor,
    pre_lens: torch.Tensor,
    seq_lens: torch.Tensor,
    extend_lens: torch.Tensor,
    out_cache_loc: torch.Tensor,
) -> None:
    """Reference implementation using PyTorch"""
    for i in range(len(req_pool_indices)):
        req_pool_idx = req_pool_indices[i].item()
        pre_len = pre_lens[i].item()
        seq_len = seq_lens[i].item()
        extend_len = extend_lens[i].item()

        cumsum_start = sum(extend_lens[:i].tolist())

        # Copy values from out_cache_loc to req_to_token
        req_to_token[req_pool_idx, pre_len:seq_len] = out_cache_loc[
            cumsum_start : cumsum_start + extend_len
        ]
```
**EN:** `write_req_to_token_pool_reference` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. The docstring frames it as: Reference implementation using PyTorch Notable calls include `range`, `len`, `req_pool_indices[i].item`.
**CN:** `write_req_to_token_pool_reference` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `range`, `len`, `req_pool_indices[i].item`。

### Lines 114-228: Function `test_write_req_to_token_pool` / 函数 `test_write_req_to_token_pool`
```python
def test_write_req_to_token_pool():
    max_batch = 4097
    max_context_len = 6148
    batch_size = 1
    extend_len = 14

    # Initialize input tensors
    req_to_token = torch.zeros(
        (max_batch, max_context_len), dtype=torch.int32, device="cuda"
    )
    req_pool_indices = torch.tensor([42], dtype=torch.int32, device="cuda")
    pre_lens = torch.tensor([8], dtype=torch.int32, device="cuda")
    seq_lens = torch.tensor([22], dtype=torch.int32, device="cuda")
    extend_lens = torch.tensor([extend_len], dtype=torch.int32, device="cuda")
    out_cache_loc = torch.arange(extend_len, dtype=torch.int32, device="cuda")

    # Create copies for reference implementation
    req_to_token_ref = req_to_token.clone()
    req_to_token_opt = req_to_token.clone()

    # Run original triton kernel
    write_req_to_token_pool_triton[(batch_size,)](
        req_to_token,
        req_pool_indices,
        pre_lens,
        seq_lens,
        extend_lens,
        out_cache_loc,
        max_context_len,
    )

    # Run optimized triton kernel
    def grid(batch_size, extend_len):
        num_token_blocks = triton.cdiv(extend_len, 512)
        return (batch_size, num_token_blocks)

    write_req_to_token_pool_triton_optimize[grid(batch_size, extend_len)](
        req_to_token_opt,
        req_pool_indices,
        pre_lens,
        seq_lens,
        extend_lens,
        out_cache_loc,
        max_context_len,
        BLOCK_SIZE=512,
    )

    # Run reference implementation
    write_req_to_token_pool_reference(
        req_to_token_ref,
        req_pool_indices,
        pre_lens,
        seq_lens,
        extend_lens,
        out_cache_loc,
    )

    # Compare results
    torch.testing.assert_close(req_to_token, req_to_token_ref)
    torch.testing.assert_close(req_to_token_opt, req_to_token_ref)

    # Test case 2: batch size > 1
    batch_size = 3
    extend_lens_list = [14, 20, 30]
    total_extend_len = sum(extend_lens_list)

    req_to_token = torch.zeros(
        (max_batch, max_context_len), dtype=torch.int32, device="cuda"
    )
    req_pool_indices = torch.tensor([42, 100, 200], dtype=torch.int32, device="cuda")
    pre_lens = torch.tensor([8, 10, 15], dtype=torch.int32, device="cuda")
    seq_lens = torch.tensor([22, 30, 45], dtype=torch.int32, device="cuda")
    extend_lens = torch.tensor(extend_lens_list, dtype=torch.int32, device="cuda")
    out_cache_loc = torch.arange(total_extend_len, dtype=torch.int32, device="cuda")

    req_to_token_ref = req_to_token.clone()
    req_to_token_opt = req_to_token.clone()

    # Run original triton kernel
    write_req_to_token_pool_triton[(batch_size,)](
        req_to_token,
        req_pool_indices,
        pre_lens,
        seq_lens,
        extend_lens,
        out_cache_loc,
        max_context_len,
    )

    # Run optimized triton kernel
    max_extend_len = max(extend_lens_list)
    write_req_to_token_pool_triton_optimize[grid(batch_size, max_extend_len)](
        req_to_token_opt,
        req_pool_indices,
        pre_lens,
        seq_lens,
        extend_lens,
        out_cache_loc,
        max_context_len,
        BLOCK_SIZE=512,
    )

    # Run reference implementation
    write_req_to_token_pool_reference(
        req_to_token_ref,
        req_pool_indices,
        pre_lens,
        seq_lens,
        extend_lens,
        out_cache_loc,
    )

    # Compare results
    torch.testing.assert_close(req_to_token, req_to_token_ref)
    torch.testing.assert_close(req_to_token_opt, req_to_token_ref)
```
**EN:** `test_write_req_to_token_pool` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `(batch_size, num_token_blocks)` to the caller. Notable calls include `torch.zeros`, `torch.tensor`, `torch.arange`.
**CN:** `test_write_req_to_token_pool` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(batch_size, num_token_blocks)`。其中较关键的调用包括 `torch.zeros`, `torch.tensor`, `torch.arange`。

### Lines 231-312: Function `get_benchmark` / 函数 `get_benchmark`
```python
def get_benchmark():
    batch_sizes = [1, 2, 4, 8, 16, 32, 64, 128]
    extend_lens = [32, 64, 128, 256, 512, 1024, 2048, 4096, 8192]
    configs = list(itertools.product(batch_sizes, extend_lens))

    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["batch_size", "extend_len"],
            x_vals=configs,
            line_arg="provider",
            line_vals=["reference", "triton", "triton_optimize"],
            line_names=["PyTorch", "Triton", "Triton Optimized"],
            styles=[("blue", "-"), ("green", "-"), ("red", "-")],
            ylabel="us",
            plot_name="write-req-to-token-pool-performance",
            args={},
        )
    )
    def benchmark(batch_size, extend_len, provider):
        max_batch = 256
        max_context_len = 16384

        extend_lens_list = [extend_len] * batch_size
        total_extend_len = sum(extend_lens_list)

        req_to_token = torch.zeros(
            (max_batch, max_context_len), dtype=torch.int32, device="cuda"
        )
        req_pool_indices = torch.arange(batch_size, dtype=torch.int32, device="cuda")
        pre_lens = torch.ones(batch_size, dtype=torch.int32, device="cuda") * 8
        seq_lens = pre_lens + extend_len
        extend_lens = torch.tensor(extend_lens_list, dtype=torch.int32, device="cuda")
        out_cache_loc = torch.arange(total_extend_len, dtype=torch.int32, device="cuda")

        quantiles = [0.5, 0.2, 0.8]

        if provider == "reference":
            ms, min_ms, max_ms = run_bench(
                lambda: write_req_to_token_pool_reference(
                    req_to_token.clone(),
                    req_pool_indices,
                    pre_lens,
                    seq_lens,
                    extend_lens,
                    out_cache_loc,
                ),
                quantiles=tuple(quantiles),
            )
        elif provider == "triton":
            ms, min_ms, max_ms = run_bench(
                lambda: write_req_to_token_pool_triton[(batch_size,)](
                    req_to_token.clone(),
                    req_pool_indices,
                    pre_lens,
                    seq_lens,
                    extend_lens,
                    out_cache_loc,
                    max_context_len,
                ),
                quantiles=tuple(quantiles),
            )
        else:

            def run_optimized():
                block_size = 128 if extend_len <= 1024 else 512
                grid_config = (batch_size, triton.cdiv(extend_len, block_size))
                write_req_to_token_pool_triton_optimize[grid_config](
                    req_to_token.clone(),
                    req_pool_indices,
                    pre_lens,
                    seq_lens,
                    extend_lens,
                    out_cache_loc,
                    max_context_len,
                    BLOCK_SIZE=block_size,
                )

            ms, min_ms, max_ms = run_bench(run_optimized, quantiles=tuple(quantiles))

        return 1000 * ms, 1000 * max_ms, 1000 * min_ms

    return benchmark
```
**EN:** `get_benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `benchmark` to the caller. Notable calls include `list`, `triton.testing.perf_report`, `itertools.product`.
**CN:** `get_benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `benchmark`。其中较关键的调用包括 `list`, `triton.testing.perf_report`, `itertools.product`。

### Lines 315-327: Function `run_benchmark` / 函数 `run_benchmark`
```python
def run_benchmark(save_path: str = "./configs/benchmark_ops/write_req_to_token_pool/"):
    """Run benchmark and save results"""

    # Ensure save path exists
    os.makedirs(save_path, exist_ok=True)

    # Run correctness test
    test_write_req_to_token_pool()
    print("Correctness test passed!")

    # Run performance test
    benchmark = get_benchmark()
    benchmark.run(print_data=True, save_path=save_path)
```
**EN:** `run_benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics and processes tokenized prompts or decoded outputs. The docstring frames it as: Run benchmark and save results Notable calls include `os.makedirs`, `test_write_req_to_token_pool`, `print`.
**CN:** `run_benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `os.makedirs`, `test_write_req_to_token_pool`, `print`。

### Lines 330-342: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--save_path",
        type=str,
        default="./configs/benchmark_ops/write_req_to_token_pool/",
        help="Path to save benchmark results",
    )
    args = parser.parse_args()

    run_benchmark(args.save_path)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `os`, `argparse`
- **Third-party / 第三方依赖**: `torch`, `triton`, `triton.language`
- **Internal / 项目内部依赖**: `sglang.benchmark.bench_utils`
