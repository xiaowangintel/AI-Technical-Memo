# bench_int8_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/quantization/bench_int8_quant.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels quantization int8 quant. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 kernels quantization int8 quant 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and setup / 导入与初始化
```python
import argparse

import torch
import triton
from vllm._custom_ops import scaled_int8_quant as vllm_scaled_int8_quant

from sglang.benchmark.bench_utils import run_bench
from sglang.srt.layers.quantization.int8_kernel import per_token_quant_int8
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 12-20: Function `torch_int8_quant` / 函数 `torch_int8_quant`
```python
def torch_int8_quant(x):
    int8_max = torch.iinfo(torch.int8).max

    abs_max = x.abs().max(dim=-1, keepdim=True).values
    scales = abs_max.to(torch.float32) / float(int8_max)

    q_x = (x / scales).round().to(torch.int8)

    return q_x, scales
```
**EN:** `torch_int8_quant` is a function that prepares tensors and invokes GPU kernels. It returns `(q_x, scales)` to the caller. Notable calls include `torch.compile`, `(x / scales).round().to`, `torch.iinfo`.
**CN:** `torch_int8_quant` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `(q_x, scales)`。其中较关键的调用包括 `torch.compile`, `(x / scales).round().to`, `torch.iinfo`。

### Lines 23-32: Function `_test_accuracy_once` / 函数 `_test_accuracy_once`
```python
def _test_accuracy_once(M, K, input_dtype, device):
    x = torch.randn(M, K, dtype=input_dtype, device=device) * 5000
    out, scales, _ = vllm_scaled_int8_quant(x, symmetric=True)
    out1, scales1 = per_token_quant_int8(x)
    out2, scales2 = torch_int8_quant(x)
    torch.testing.assert_close(out, out2, atol=1, rtol=0)
    torch.testing.assert_close(out, out1, atol=1, rtol=0)
    torch.testing.assert_close(scales, scales2)
    torch.testing.assert_close(scales1, scales2)
    print(f"M: {M}, K: {K}, type: {input_dtype} OK")
```
**EN:** `_test_accuracy_once` is a function that prepares tensors and invokes GPU kernels, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. Notable calls include `vllm_scaled_int8_quant`, `per_token_quant_int8`, `torch_int8_quant`.
**CN:** `_test_accuracy_once` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。其中较关键的调用包括 `vllm_scaled_int8_quant`, `per_token_quant_int8`, `torch_int8_quant`。

### Lines 35-42: Function `test_accuracy` / 函数 `test_accuracy`
```python
def test_accuracy():
    Ms = [1, 13, 128, 1024, 2048, 4096]
    Ks = [512, 1024, 2048, 8192]
    input_dtypes = [torch.float16, torch.bfloat16]
    for M in Ms:
        for K in Ks:
            for input_dtype in input_dtypes:
                _test_accuracy_once(M, K, input_dtype, "cuda")
```
**EN:** `test_accuracy` is a function that prepares tensors and invokes GPU kernels and computes evaluation scores and aggregate statistics. Notable calls include `_test_accuracy_once`.
**CN:** `test_accuracy` 是一个函数，用于准备张量并调用 GPU 内核、计算评测分数与聚合统计结果。其中较关键的调用包括 `_test_accuracy_once`。

### Lines 59-80: Function `benchmark` / 函数 `benchmark`
```python
def benchmark(batch_size, provider):
    M, K = batch_size, 16384
    x = torch.randn(M, K, dtype=torch.float16, device="cuda") * 1000

    quantiles = (0.5, 0.2, 0.8)
    if provider == "vllm op":
        ms, min_ms, max_ms = run_bench(
            lambda: vllm_scaled_int8_quant(x, symmetric=True),
            quantiles=quantiles,
        )
    if provider == "triton":
        ms, min_ms, max_ms = run_bench(
            lambda: per_token_quant_int8(x),
            quantiles=quantiles,
        )
    if provider == "torch.compile":
        ms, min_ms, max_ms = run_bench(
            lambda: torch_int8_quant(x),
            quantiles=quantiles,
        )

    return ms, min_ms, max_ms
```
**EN:** `benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `(ms, min_ms, max_ms)` to the caller. Notable calls include `triton.testing.perf_report`, `triton.testing.Benchmark`, `torch.randn`.
**CN:** `benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(ms, min_ms, max_ms)`。其中较关键的调用包括 `triton.testing.perf_report`, `triton.testing.Benchmark`, `torch.randn`。

### Lines 83-95: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--save_path",
        type=str,
        default="./bench_int8_quant_res",
        help="Path to save int8 quant benchmark results",
    )
    args = parser.parse_args()

    test_accuracy()

    benchmark.run(print_data=True, show_plots=True, save_path=args.save_path)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and computes evaluation scores and aggregate statistics.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、计算评测分数与聚合统计结果。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`
- **Third-party / 第三方依赖**: `torch`, `triton`, `vllm._custom_ops`
- **Internal / 项目内部依赖**: `sglang.benchmark.bench_utils`, `sglang.srt.layers.quantization.int8_kernel`
