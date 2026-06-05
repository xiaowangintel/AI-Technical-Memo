# run_benchmarks.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/scripts/run_benchmarks.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This utility script automates run benchmarks for the gateway utility scripts. It packages repeatable operational steps into a standalone command-line entry point. / 该工具脚本为 脚本工具 自动化 run benchmarks 流程，并将可重复执行的操作步骤封装成独立的命令行入口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module header and comments
```python
#!/usr/bin/env python3
```
**EN:** These leading lines typically contain a shebang, encoding note, or comments that frame the rest of the file.
**CN:** 这些起始行通常包含 shebang、编码说明或注释，用于为后续代码提供背景。

### Lines 2-7: Module docstring
```python
"""
SGLang Router Benchmark Runner

A Python script to run Rust benchmarks with various options and modes.
Replaces the shell script for better maintainability and integration.
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 9-15: Imports and dependencies
```python
import argparse
import os
import subprocess
import sys
import time
from pathlib import Path
from typing import Dict, List, Optional
```
**EN:** This block imports `argparse`, `os`, `subprocess`, `sys`, and 3 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 18-188: Class `BenchmarkRunner`
```python
class BenchmarkRunner:
    """Handles running Rust benchmarks for the SGLang router."""

    def __init__(self, project_root: str):
        self.project_root = Path(project_root)
        self.timestamp = time.strftime("%a %b %d %H:%M:%S UTC %Y", time.gmtime())

    def run_command(
        self, cmd: List[str], capture_output: bool = False
    ) -> subprocess.CompletedProcess:
        """Run a command and handle errors."""
        try:
            if capture_output:
                result = subprocess.run(
                    cmd, capture_output=True, text=True, cwd=self.project_root
                )
            else:
                result = subprocess.run(cmd, cwd=self.project_root)
            return result
        except subprocess.CalledProcessError as e:
            print(f"Error running command: {' '.join(cmd)}")
            print(f"Exit code: {e.returncode}")
            sys.exit(1)

    def print_header(self):
        """Print the benchmark runner header."""
        print("SGLang Router Benchmark Runner")
        print("=" * 30)
        print(f"Project: {self.project_root.absolute()}")
        print(f"Timestamp: {self.timestamp}")
        print()

    def build_release(self):
        """Build the project in release mode."""
        print("Building in release mode...")
        result = self.run_command(["cargo", "build", "--release", "--quiet"])
        if result.returncode != 0:
            print("Failed to build in release mode")
            sys.exit(1)

    def run_benchmarks(
        self,
        quick_mode: bool = False,
        save_baseline: Optional[str] = None,
        compare_baseline: Optional[str] = None,
    ) -> str:
        """Run benchmarks with specified options."""
        bench_args = ["cargo", "bench", "--bench", "request_processing"]

        if quick_mode:
            bench_args.append("benchmark_summary")
            print("Running quick benchmarks...")
        else:
            print("Running full benchmark suite...")

        # Note: Criterion baselines are handled via target directory structure
        # For now, we'll implement baseline functionality via file copying
        if save_baseline:
            print(f"Will save results as baseline: {save_baseline}")

        if compare_baseline:
            print(f"Will compare with baseline: {compare_baseline}")

        print(f"Executing: {' '.join(bench_args)}")
        result = self.run_command(bench_args, capture_output=True)

        if result.returncode != 0:
            print("Benchmark execution failed!")
            print("STDOUT:", result.stdout)
            print("STDERR:", result.stderr)
            sys.exit(1)

        # Handle baseline saving after successful run
        if save_baseline:
            self._save_baseline(save_baseline, result.stdout)

        return result.stdout

    def _save_baseline(self, filename: str, output: str):
        """Save benchmark results to a file as baseline."""
        filepath = self.project_root / filename
        with open(filepath, "w") as f:
            f.write(output)
        print(f"Baseline saved to: {filepath}")

    def parse_benchmark_results(self, output: str) -> Dict[str, str]:
        """Parse benchmark output to extract performance metrics."""
        results = {}

        # Look for performance overview section
        lines = output.split("\n")
        parsing_overview = False

        for line in lines:
            line = line.strip()

            if "Quick Performance Overview:" in line:
                parsing_overview = True
                continue

            if parsing_overview and line.startswith("* "):
                # Parse lines like "* Serialization (avg):          481 ns/req"
                if "Serialization (avg):" in line:
                    results["serialization_time"] = self._extract_time(line)
                elif "Deserialization (avg):" in line:
                    results["deserialization_time"] = self._extract_time(line)
                elif "Bootstrap Injection (avg):" in line:
                    results["bootstrap_injection_time"] = self._extract_time(line)
                elif "Total Pipeline (avg):" in line:
                    results["total_time"] = self._extract_time(line)

            # Stop parsing after the overview section
            if parsing_overview and line.startswith("Performance Insights:"):
                break

        return results

    def _extract_time(self, line: str) -> str:
        """Extract time value from a benchmark line."""
        # Extract number followed by ns/req
        import re

        match = re.search(r"(\d+)\s*ns/req", line)
        return match.group(1) if match else "N/A"

    def validate_thresholds(self, results: Dict[str, str]) -> bool:
        """Validate benchmark results against performance thresholds."""
        thresholds = {
            "serialization_time": 2000,  # 2μs max
            "deserialization_time": 2000,  # 2μs max
            "bootstrap_injection_time": 5000,  # 5μs max
            "total_time": 10000,  # 10μs max
        }

        all_passed = True
        print("\nPerformance Threshold Validation:")
        print("=" * 35)

        for metric, threshold in thresholds.items():
            if metric in results and results[metric] != "N/A":
                try:
                    value = int(results[metric])
                    passed = value <= threshold
                    status = "✓ PASS" if passed else "✗ FAIL"
                    print(f"{metric:20}: {value:>6}ns <= {threshold:>6}ns {status}")
                    if not passed:
                        all_passed = False
                except ValueError:
                    print(f"{metric:20}: Invalid value: {results[metric]}")
                    all_passed = False
            else:
                print(f"{metric:20}: No data available")
                all_passed = False

        print()
        if all_passed:
            print("All performance thresholds passed!")
        else:
            print("Some performance thresholds failed!")

        return all_passed

    def save_results_to_file(
        self, results: Dict[str, str], filename: str = "benchmark_results.env"
    ):
        """Save benchmark results to a file for CI consumption."""
        filepath = self.project_root / filename
        with open(filepath, "w") as f:
            for key, value in results.items():
                f.write(f"{key}={value}\n")
        print(f"Results saved to: {filepath}")
```
**EN:** Class `BenchmarkRunner` groups related state and behavior. It exposes 10 method(s) that implement the module's primary abstraction.
**CN:** 类 `BenchmarkRunner` 将相关状态与行为封装在一起，提供 10 个方法来实现本模块的核心抽象。

### Lines 191-246: function `main`
```python
def main():
    parser = argparse.ArgumentParser(description="Run SGLang router benchmarks")
    parser.add_argument(
        "--quick", action="store_true", help="Run quick benchmarks (summary only)"
    )
    parser.add_argument(
        "--save-baseline", type=str, help="Save benchmark results as baseline"
    )
    parser.add_argument(
        "--compare-baseline", type=str, help="Compare with saved baseline"
    )
    parser.add_argument(
        "--validate-thresholds",
        action="store_true",
        help="Validate results against performance thresholds",
    )
    parser.add_argument(
        "--save-results", action="store_true", help="Save results to file for CI"
    )

    args = parser.parse_args()

    # Determine project root (script is in scripts/ subdirectory)
    script_dir = Path(__file__).parent
    project_root = script_dir.parent

    runner = BenchmarkRunner(str(project_root))
    runner.print_header()

    # Build in release mode
    runner.build_release()

    # Run benchmarks
    output = runner.run_benchmarks(
        quick_mode=args.quick,
        save_baseline=args.save_baseline,
        compare_baseline=args.compare_baseline,
    )

    # Print the raw output
    print(output)

    # Parse and validate results if requested
    if args.validate_thresholds or args.save_results:
        results = runner.parse_benchmark_results(output)

        if args.save_results:
            runner.save_results_to_file(results)

        if args.validate_thresholds:
            passed = runner.validate_thresholds(results)
            if not passed:
                print("Validation failed - performance regression detected!")
                sys.exit(1)

    print("\nBenchmark run completed successfully!")
```
**EN:** function `main` encapsulates `main` logic for callers in this module.
**CN:** 函数 `main` 封装了与 `main` 相关的逻辑，供本模块中的调用方复用。

### Lines 249-250: Command-line entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This main-guard block makes the file executable as a script by invoking the module's top-level routine only when run directly.
**CN:** 这个主入口保护块让文件可以作为脚本直接执行，并仅在被直接运行时调用模块的顶层流程。

## Key Concepts / 关键概念
- Performance measurement and comparison / 性能测量与对比
- Standalone operational automation / 独立运维自动化脚本
- Command-line argument parsing / 命令行参数解析
- Primary classes: `BenchmarkRunner` / 主要类：`BenchmarkRunner`
- Reusable functions: `main` / 可复用函数：`main`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `os`, `pathlib`, `re`, `subprocess`, `sys`, `time`, `typing`
- **Third-party / 第三方**: None explicitly imported / 未显式导入
- **Internal / 内部模块**: None explicitly imported / 未显式导入
