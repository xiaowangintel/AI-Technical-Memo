# benchmark_tokenizer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/benchmark_batch/benchmark_tokenizer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on batch tokenizer. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 batch tokenizer 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and setup / 导入与初始化
```python
import argparse
import random
import time
from statistics import mean

from transformers import AutoTokenizer

from sglang.srt.utils.patch_tokenizer import patch_tokenizer
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and processes tokenized prompts or decoded outputs.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、处理分词后的提示词或解码后的输出。

### Lines 11-62: Function `main` / 函数 `main`
```python
def main():
    args = parse_args()

    print("Tokenizer Benchmark: Sequential vs Batch Processing")
    print("-" * 60)
    print(f"Tokenizer: {args.tokenizer}")
    print(f"Functions: {', '.join(args.function)}")
    print(f"Tokens per prompt: {args.num_tokens}")
    print(f"Number of runs per batch size: {args.num_runs}")
    print(f"Batch mode: {', '.join(args.batch_mode)}")
    print("-" * 60)

    tokenizer = AutoTokenizer.from_pretrained(args.tokenizer, trust_remote_code=True)
    tokenizer = patch_tokenizer(tokenizer)
    max_batch_size = max(args.batch_sizes)

    token_ids = generate_random_token_ids(
        num_prompts=max_batch_size, num_tokens=args.num_tokens, tokenizer=tokenizer
    )

    if "encode" in args.function:
        prompts = [
            tokenizer.decode(ids, clean_up_tokenization_spaces=True)
            for ids in token_ids
        ]
        run_benchmark(
            name="encode",
            data=prompts,
            sequential_fn=lambda batch: [tokenizer.encode(p) for p in batch],
            batch_fn=lambda batch: tokenizer(batch),
            batch_sizes=args.batch_sizes,
            num_runs=args.num_runs,
            batch_mode=args.batch_mode,
        )

    if "decode" in args.function:
        # mimic DetokenizerManager's usual case
        decode_kwargs = dict(
            skip_special_tokens=True,
            spaces_between_special_tokens=True,
        )
        run_benchmark(
            name="decode",
            data=token_ids,
            sequential_fn=lambda batch: [
                tokenizer.decode(ids, **decode_kwargs) for ids in batch
            ],
            batch_fn=lambda batch: tokenizer.batch_decode(batch, **decode_kwargs),
            batch_sizes=args.batch_sizes,
            num_runs=args.num_runs,
            batch_mode=args.batch_mode,
        )
```
**EN:** `main` is a function that builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `parse_args`, `print`, `AutoTokenizer.from_pretrained`.
**CN:** `main` 是一个函数，用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `parse_args`, `print`, `AutoTokenizer.from_pretrained`。

### Lines 65-83: Function `run_benchmark` / 函数 `run_benchmark`
```python
def run_benchmark(
    *, name, data, sequential_fn, batch_fn, batch_sizes, num_runs, batch_mode
):
    print("\n" + "=" * 60)
    print(f"{name.upper()} BENCHMARK")
    print("=" * 60)

    results = [
        benchmark(
            data=data,
            batch_size=bs,
            sequential_fn=sequential_fn,
            batch_fn=batch_fn,
            num_runs=num_runs,
            batch_mode=batch_mode,
        )
        for bs in batch_sizes
    ]
    print_results(results=results, func_name=name, batch_mode=batch_mode)
```
**EN:** `run_benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics. Notable calls include `print`, `print_results`, `benchmark`.
**CN:** `run_benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。其中较关键的调用包括 `print`, `print_results`, `benchmark`。

### Lines 86-116: Function `benchmark` / 函数 `benchmark`
```python
def benchmark(*, data, batch_size, sequential_fn, batch_fn, num_runs, batch_mode):
    batch_data = data[:batch_size]
    run_single = "single" in batch_mode
    run_batch = "batch" in batch_mode

    out = {"batch_size": batch_size}

    if run_single:
        sequential_times = measure_times(
            fn=lambda: sequential_fn(batch_data), num_runs=num_runs
        )
        out |= {
            "avg_sequential_ms": mean(sequential_times),
            "sequential_runs": sequential_times,
        }

    if run_batch:
        batch_times = measure_times(fn=lambda: batch_fn(batch_data), num_runs=num_runs)
        out |= {
            "avg_batch_ms": mean(batch_times),
            "batch_runs": batch_times,
        }

    if run_single and run_batch:
        out["speedup_factor"] = (
            out["avg_sequential_ms"] / out["avg_batch_ms"]
            if out["avg_batch_ms"] > 0
            else 0
        )

    return out
```
**EN:** `benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics. It returns `out` to the caller. Notable calls include `measure_times`, `mean`, `sequential_fn`.
**CN:** `benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `out`。其中较关键的调用包括 `measure_times`, `mean`, `sequential_fn`。

### Lines 119-160: Function `print_results` / 函数 `print_results`
```python
def print_results(*, results, func_name, batch_mode):
    run_single = "single" in batch_mode
    run_batch = "batch" in batch_mode

    for r in results:
        print(f"\nBatch size: {r['batch_size']}")
        if run_single:
            print_runs(
                label=f"Sequential {func_name}",
                runs=r["sequential_runs"],
                avg=r["avg_sequential_ms"],
            )
        if run_batch:
            print_runs(
                label=f"Batch {func_name}", runs=r["batch_runs"], avg=r["avg_batch_ms"]
            )
        if run_single and run_batch:
            print(f"  Speedup factor: {r['speedup_factor']:.2f}x")

    print("\n" + "=" * 60)
    print(f"SUMMARY: {func_name.upper()}")
    print("=" * 60)

    headers = ["Batch Size"]
    if run_single:
        headers.append("Sequential (ms)")
    if run_batch:
        headers.append("Batch (ms)")
    if run_single and run_batch:
        headers.append("Speedup")
    print("".join(f"{h:<18}" for h in headers))
    print("-" * (18 * len(headers)))

    for r in results:
        row = [f"{r['batch_size']}"]
        if run_single:
            row.append(f"{r['avg_sequential_ms']:.2f} ms")
        if run_batch:
            row.append(f"{r['avg_batch_ms']:.2f} ms")
        if run_single and run_batch:
            row.append(f"{r['speedup_factor']:.2f}x")
        print("".join(f"{v:<18}" for v in row))
```
**EN:** `print_results` is a function that implements the core logic for this scope. Notable calls include `print`, `headers.append`, `''.join`.
**CN:** `print_results` 是一个函数，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `print`, `headers.append`, `''.join`。

### Lines 163-167: Function `print_runs` / 函数 `print_runs`
```python
def print_runs(*, label, runs, avg):
    print(f"  {label}:")
    for i, t in enumerate(runs):
        print(f"    Run {i+1}: {t:.2f} ms")
    print(f"    Average: {avg:.2f} ms")
```
**EN:** `print_runs` is a function that implements the core logic for this scope. Notable calls include `print`, `enumerate`.
**CN:** `print_runs` 是一个函数，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `print`, `enumerate`。

### Lines 170-176: Function `measure_times` / 函数 `measure_times`
```python
def measure_times(*, fn, num_runs):
    times = []
    for _ in range(num_runs):
        start = time.perf_counter()
        fn()
        times.append((time.perf_counter() - start) * 1000)
    return times
```
**EN:** `measure_times` is a function that measures runtime latency, throughput, or other benchmark metrics. It returns `times` to the caller. Notable calls include `range`, `time.perf_counter`, `fn`.
**CN:** `measure_times` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `times`。其中较关键的调用包括 `range`, `time.perf_counter`, `fn`。

### Lines 179-185: Function `generate_random_token_ids` / 函数 `generate_random_token_ids`
```python
def generate_random_token_ids(*, num_prompts, num_tokens, tokenizer):
    vocab_size = tokenizer.vocab_size
    print(f"Generating {num_prompts} random sequences with {num_tokens} tokens each...")
    return [
        [random.randint(0, vocab_size - 1) for _ in range(num_tokens)]
        for _ in range(num_prompts)
    ]
```
**EN:** `generate_random_token_ids` is a function that processes tokenized prompts or decoded outputs. It returns `[[random.randint(0, vocab_size - 1) for _ in range(num_tokens)] for _ in range(num_prompts)]` to the caller. Notable calls include `print`, `random.randint`, `range`.
**CN:** `generate_random_token_ids` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `[[random.randint(0, vocab_size - 1) for _ in range(num_tokens)] for _ in range(num_prompts)]`。其中较关键的调用包括 `print`, `random.randint`, `range`。

### Lines 188-232: Function `parse_args` / 函数 `parse_args`
```python
def parse_args():
    parser = argparse.ArgumentParser(
        description="Tokenizer Benchmark: Sequential vs Batch Processing"
    )
    parser.add_argument(
        "--tokenizer",
        type=str,
        required=True,
        help="Tokenizer name or path (e.g. nvidia/Kimi-K2-Thinking-NVFP4)",
    )
    parser.add_argument(
        "--function",
        type=str,
        nargs="+",
        choices=["encode", "decode"],
        default=["encode", "decode"],
        help="Functions to benchmark (default: encode decode)",
    )
    parser.add_argument(
        "--num-tokens",
        type=int,
        default=20000,
        help="Number of tokens per prompt (default: 20000)",
    )
    parser.add_argument(
        "--batch-sizes",
        type=int,
        nargs="+",
        default=[1, 2, 4, 8],
        help="Batch sizes to test (default: 1 2 4 8)",
    )
    parser.add_argument(
        "--batch-mode",
        nargs="+",
        choices=["single", "batch"],
        default=["single", "batch"],
        help="Benchmark modes to run (default: single batch)",
    )
    parser.add_argument(
        "--num-runs",
        type=int,
        default=5,
        help="Number of runs per batch size (default: 5)",
    )
    return parser.parse_args()
```
**EN:** `parse_args` is a function that builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. It returns `parser.parse_args()` to the caller. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`.
**CN:** `parse_args` 是一个函数，用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。它会向调用方返回 `parser.parse_args()`。其中较关键的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`。

### Lines 235-237: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    random.seed(0)
    main()
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `random`, `time`, `statistics`
- **Third-party / 第三方依赖**: `transformers`
- **Internal / 项目内部依赖**: `sglang.srt.utils.patch_tokenizer`
