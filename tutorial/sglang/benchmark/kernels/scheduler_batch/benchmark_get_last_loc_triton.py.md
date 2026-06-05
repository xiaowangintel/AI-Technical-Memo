# benchmark_get_last_loc_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/scheduler_batch/benchmark_get_last_loc_triton.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels scheduler batch get last. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 kernels scheduler batch get last 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and setup / 导入与初始化
```python
import os

import torch
import triton
import triton.language as tl

from sglang.benchmark.bench_utils import run_bench
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 11-20: Function `get_last_loc_torch` / 函数 `get_last_loc_torch`
```python
def get_last_loc_torch(
    req_to_token: torch.Tensor,
    req_pool_indices_tensor: torch.Tensor,
    prefix_lens_tensor: torch.Tensor,
) -> torch.Tensor:
    return torch.where(
        prefix_lens_tensor > 0,
        req_to_token[req_pool_indices_tensor, prefix_lens_tensor - 1],
        torch.full_like(prefix_lens_tensor, -1),
    )
```
**EN:** `get_last_loc_torch` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `torch.where(prefix_lens_tensor > 0, req_to_token[req_pool_indices_tensor, prefix_lens_tensor - 1]...` to the caller. Notable calls include `torch.compile`, `torch.where`, `torch.full_like`.
**CN:** `get_last_loc_torch` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `torch.where(prefix_lens_tensor > 0, req_to_token[req_pool_indices_tensor, prefix_lens_tensor - 1]...`。其中较关键的调用包括 `torch.compile`, `torch.where`, `torch.full_like`。

### Lines 24-44: Function `get_last_loc_kernel` / 函数 `get_last_loc_kernel`
```python
def get_last_loc_kernel(
    req_to_token,
    req_pool_indices_tensor,
    prefix_lens_tensor,
    result,
    num_tokens,
    req_to_token_stride,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(0)
    offset = tl.arange(0, BLOCK_SIZE) + pid * BLOCK_SIZE
    mask = offset < num_tokens

    prefix_lens = tl.load(prefix_lens_tensor + offset, mask=mask, other=0)
    req_pool_indices = tl.load(req_pool_indices_tensor + offset, mask=mask, other=0)

    token_mask = prefix_lens > 0
    token_index = req_pool_indices * req_to_token_stride + (prefix_lens - 1)
    tokens = tl.load(req_to_token + token_index, mask=token_mask, other=-1)

    tl.store(result + offset, tokens, mask=mask)
```
**EN:** `get_last_loc_kernel` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. Notable calls include `tl.program_id`, `tl.load`, `tl.store`.
**CN:** `get_last_loc_kernel` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `tl.program_id`, `tl.load`, `tl.store`。

### Lines 47-66: Function `get_last_loc_triton` / 函数 `get_last_loc_triton`
```python
def get_last_loc_triton(
    req_to_token: torch.Tensor,
    req_pool_indices_tensor: torch.Tensor,
    prefix_lens_tensor: torch.Tensor,
) -> torch.Tensor:
    BLOCK_SIZE = 256
    num_tokens = prefix_lens_tensor.shape[0]
    result = torch.empty_like(prefix_lens_tensor)
    grid = (triton.cdiv(num_tokens, BLOCK_SIZE),)

    get_last_loc_kernel[grid](
        req_to_token,
        req_pool_indices_tensor,
        prefix_lens_tensor,
        result,
        num_tokens,
        req_to_token.stride(0),
        BLOCK_SIZE,
    )
    return result
```
**EN:** `get_last_loc_triton` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `result` to the caller. Notable calls include `torch.empty_like`, `get_last_loc_kernel[grid]`, `triton.cdiv`.
**CN:** `get_last_loc_triton` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `result`。其中较关键的调用包括 `torch.empty_like`, `get_last_loc_kernel[grid]`, `triton.cdiv`。

### Lines 69-91: Function `test_get_last_loc` / 函数 `test_get_last_loc`
```python
def test_get_last_loc():
    max_batch = 4097
    max_context_len = 6148
    batch_size = 20

    # Initialize input tensors
    req_to_token = torch.zeros(
        (max_batch, max_context_len), dtype=torch.int32, device="cuda"
    )
    req_pool_indices = torch.arange(batch_size, dtype=torch.int64, device="cuda")
    pre_lens = torch.randint(
        -max_context_len // 2,
        max_context_len,
        (batch_size,),
        dtype=torch.int64,
        device="cuda",
    )

    last_loc_res = get_last_loc_triton(req_to_token, req_pool_indices, pre_lens)
    last_loc_ref = get_last_loc_torch(req_to_token, req_pool_indices, pre_lens)

    # Compare results
    torch.testing.assert_close(last_loc_res, last_loc_ref)
```
**EN:** `test_get_last_loc` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. Notable calls include `torch.zeros`, `torch.arange`, `torch.randint`.
**CN:** `test_get_last_loc` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `torch.zeros`, `torch.arange`, `torch.randint`。

### Lines 94-141: Function `get_benchmark` / 函数 `get_benchmark`
```python
def get_benchmark():
    batch_sizes = [1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024]

    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["batch_size"],
            x_vals=batch_sizes,
            line_arg="provider",
            line_vals=["reference", "triton"],
            line_names=["PyTorch", "Triton"],
            styles=[("blue", "-"), ("green", "-")],
            ylabel="us",
            plot_name="get-last-loc-performance",
            args={},
        )
    )
    def benchmark(batch_size, provider):
        max_batch = 2048
        max_context_len = 16384

        req_to_token = torch.zeros(
            (max_batch, max_context_len), dtype=torch.int32, device="cuda"
        )
        req_pool_indices = torch.arange(batch_size, dtype=torch.int64, device="cuda")
        pre_lens = torch.randint(
            -max_context_len // 2,
            max_context_len,
            (batch_size,),
            dtype=torch.int64,
            device="cuda",
        )

        quantiles = [0.5, 0.2, 0.8]

        if provider == "reference":
            ms, min_ms, max_ms = run_bench(
                lambda: get_last_loc_torch(req_to_token, req_pool_indices, pre_lens),
                quantiles=tuple(quantiles),
            )
        elif provider == "triton":
            ms, min_ms, max_ms = run_bench(
                lambda: get_last_loc_triton(req_to_token, req_pool_indices, pre_lens),
                quantiles=tuple(quantiles),
            )

        return 1000 * ms, 1000 * max_ms, 1000 * min_ms

    return benchmark
```
**EN:** `get_benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `benchmark` to the caller. Notable calls include `triton.testing.perf_report`, `torch.zeros`, `torch.arange`.
**CN:** `get_benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `benchmark`。其中较关键的调用包括 `triton.testing.perf_report`, `torch.zeros`, `torch.arange`。

### Lines 144-156: Function `run_benchmark` / 函数 `run_benchmark`
```python
def run_benchmark(save_path: str = "./configs/benchmark_ops/get_last_loc/"):
    """Run benchmark and save results"""

    # Ensure save path exists
    os.makedirs(save_path, exist_ok=True)

    # Run correctness test
    test_get_last_loc()
    print("Correctness test passed!")

    # Run performance test
    benchmark = get_benchmark()
    benchmark.run(print_data=True, save_path=save_path)
```
**EN:** `run_benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics. The docstring frames it as: Run benchmark and save results Notable calls include `os.makedirs`, `test_get_last_loc`, `print`.
**CN:** `run_benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `os.makedirs`, `test_get_last_loc`, `print`。

### Lines 159-171: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--save_path",
        type=str,
        default="./configs/benchmark_ops/get_last_loc/",
        help="Path to save benchmark results",
    )
    args = parser.parse_args()

    run_benchmark(args.save_path)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration and measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `argparse`
- **Third-party / 第三方依赖**: `torch`, `triton`, `triton.language`
- **Internal / 项目内部依赖**: `sglang.benchmark.bench_utils`
