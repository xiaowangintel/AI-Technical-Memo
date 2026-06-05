# benchmark_hash.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/benchmark_hash.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, hashing and cache-key experiments, centered around `_generate_test_data`, `_benchmark_func`, `_run_benchmarks`, `builtin_hash`. / 实现与基准测试编排、哈希与缓存键实验相关的逻辑，核心符号包括 `_generate_test_data`, `_benchmark_func`, `_run_benchmarks`, `builtin_hash`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-11)
```python
"""
Micro benchmark comparing built-in hash(), SHA-256, and xxHash.

This focuses on a single test payload shaped like the prefix-cache hash input:
    (32-byte bytes object, 32-int tuple)

Usage:
    python benchmarks/hash_micro_benchmark.py --iterations 20000
"""
```
**EN:** The module docstring introduces Micro benchmark comparing built-in hash(), SHA-256, and xxHash. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Micro benchmark comparing built-in hash(), SHA-256, and xxHash 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 13-21)
```python
from __future__ import annotations

import argparse
import random
import statistics
import time
from collections.abc import Callable, Iterable

from vllm.utils.hashing import sha256, xxhash
```
**EN:** This block gathers standard-library helpers such as `__future__`, `argparse`, `random`, `statistics`, `time`, `collections.abc`; project-local modules such as `vllm.utils.hashing`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `__future__`, `argparse`, `random`, `statistics`, `time`, `collections.abc`；项目内部模块，如 `vllm.utils.hashing`。这些依赖构成了后续基准测试逻辑的基础。

### Function `_generate_test_data` (lines 24-29)
```python
def _generate_test_data(seed: int) -> tuple[bytes, tuple[int, ...]]:
    """Generate a deterministic test payload."""
    random.seed(seed)
    bytes_data = bytes(random.getrandbits(8) for _ in range(32))
    int_tuple = tuple(random.randint(1, 1_000_000) for _ in range(32))
    return (bytes_data, int_tuple)
```
**EN:** `_generate_test_data` Generate a deterministic test payload. It mainly works with `seed` and relies on `random.seed`, `bytes`, `random.getrandbits`, `range`, `tuple`, `random.randint` plus value production to move data through this part of the benchmark pipeline.
**CN:** `_generate_test_data` 的职责是：Generate a deterministic test payload。 它主要处理 `seed`，并结合 `random.seed`, `bytes`, `random.getrandbits`, `range`, `tuple`, `random.randint` 以及 结果返回 来完成这一段基准测试流程。

### Function `_benchmark_func` (lines 32-48)
```python
def _benchmark_func(func: Callable[[tuple], object], data: tuple, iterations: int):
    """Return (avg_seconds, std_seconds) for hashing `data` `iterations` times."""
    times: list[float] = []

    # Warm-up to avoid first-run noise.
    for _ in range(200):
        func(data)

    for _ in range(iterations):
        start = time.perf_counter()
        func(data)
        end = time.perf_counter()
        times.append(end - start)

    avg = statistics.mean(times)
    std = statistics.stdev(times) if len(times) > 1 else 0.0
    return avg, std
```
**EN:** `_benchmark_func` Return (avg_seconds, std_seconds) for hashing `data` `iterations` times. It mainly works with `func`, `data`, `iterations` and relies on `range`, `func`, `time.perf_counter`, `times.append`, `statistics.mean`, `len` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `_benchmark_func` 的职责是：Return (avg_seconds, std_seconds) for hashing `data` `iterations` times。 它主要处理 `func`, `data`, `iterations`，并结合 `range`, `func`, `time.perf_counter`, `times.append`, `statistics.mean`, `len` 以及 循环迭代 来完成这一段基准测试流程。

### Function `_run_benchmarks` (lines 51-63)
```python
def _run_benchmarks(
    benchmarks: Iterable[tuple[str, Callable[[tuple], object]]],
    data: tuple,
    iterations: int,
):
    """Yield (name, avg, std) for each benchmark, skipping unavailable ones."""
    for name, func in benchmarks:
        try:
            avg, std = _benchmark_func(func, data, iterations)
        except ModuleNotFoundError as exc:
            print(f"Skipping {name}: {exc}")
            continue
        yield name, avg, std
```
**EN:** `_run_benchmarks` Yield (name, avg, std) for each benchmark, skipping unavailable ones. It mainly works with `benchmarks`, `data`, `iterations` and relies on `_benchmark_func`, `print` plus iteration and error handling to move data through this part of the benchmark pipeline.
**CN:** `_run_benchmarks` 的职责是：Yield (name, avg, std) for each benchmark, skipping unavailable ones。 它主要处理 `benchmarks`, `data`, `iterations`，并结合 `_benchmark_func`, `print` 以及 循环迭代、异常处理 来完成这一段基准测试流程。

### Function `builtin_hash` (lines 66-68)
```python
def builtin_hash(data: tuple) -> int:
    """Wrapper for Python's built-in hash()."""
    return hash(data)
```
**EN:** `builtin_hash` Wrapper for Python's built-in hash(). It mainly works with `data` and relies on `hash` plus value production to move data through this part of the benchmark pipeline.
**CN:** `builtin_hash` 的职责是：Wrapper for Python's built-in hash()。 它主要处理 `data`，并结合 `hash` 以及 结果返回 来完成这一段基准测试流程。

### Function `main` (lines 71-116)
```python
def main() -> None:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument(
        "--iterations",
        type=int,
        default=10_000,
        help="Number of measured iterations per hash function.",
    )
    parser.add_argument(
        "--seed", type=int, default=42, help="Random seed for test payload."
    )
    args = parser.parse_args()

    data = _generate_test_data(args.seed)
    benchmarks = (
        ("SHA256 (pickle)", sha256),
        ("xxHash (pickle)", xxhash),
        ("built-in hash()", builtin_hash),
    )

    print("=" * 60)
    print("HASH FUNCTION MICRO BENCHMARK")
    print("=" * 60)
    print("Test data: (32-byte bytes object, 32-int tuple)")
    print(f"Iterations: {args.iterations:,}")
    print("=" * 60)

    results = list(_run_benchmarks(benchmarks, data, args.iterations))
    builtin_entry = next((r for r in results if r[0] == "built-in hash()"), None)

    print("\nResults:")
    for name, avg, std in results:
        print(f"  {name:16s}: {avg * 1e6:8.2f} ± {std * 1e6:6.2f} μs")

    if builtin_entry:
        _, builtin_avg, _ = builtin_entry
        print("\n" + "=" * 60)
        print("SUMMARY (relative to built-in hash())")
        print("=" * 60)
        for name, avg, _ in results:
            if name == "built-in hash()":
                continue
            speed_ratio = avg / builtin_avg
            print(f"• {name} is {speed_ratio:.1f}x slower than built-in hash()")
    else:
        print("\nBuilt-in hash() result missing; cannot compute speed ratios.")
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `_generate_test_data`, `print`, `list` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `_generate_test_data`, `print`, `list` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 119-120)
```python
if __name__ == "__main__":
    main()
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。

## Dependencies / 依赖关系
- **EN:** Standard library: `__future__`, `argparse`, `random`, `statistics`, `time`, `collections.abc`.
- **CN:** 标准库依赖：`__future__`, `argparse`, `random`, `statistics`, `time`, `collections.abc`。
- **EN:** Third-party packages: none.
- **CN:** 第三方依赖：无。
- **EN:** Internal modules: `vllm.utils.hashing`.
- **CN:** 内部模块：`vllm.utils.hashing`。
