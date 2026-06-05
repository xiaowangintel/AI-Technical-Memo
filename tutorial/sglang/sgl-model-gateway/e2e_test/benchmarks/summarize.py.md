# summarize.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/benchmarks/summarize.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises summarize behavior in the benchmark suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端基准测试 中与 summarize 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module docstring
```python
"""Generate benchmark summary for GitHub Actions."""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 3-9: Imports and dependencies
```python
from __future__ import annotations

import os
import sys
from pathlib import Path

from results import BenchmarkResult, GPUUtilization
```
**EN:** This block imports `__future__`, `os`, `sys`, `pathlib`, and 1 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 12-31: Helper function `discover_benchmarks`
```python
def discover_benchmarks(base_dir: Path) -> list[tuple[Path, str]]:
    """Auto-discover benchmark folders and their result JSON files.

    Returns list of (json_path, label) tuples sorted by folder name.
    """
    results = []
    for folder in base_dir.rglob("benchmark_*"):
        if not folder.is_dir():
            continue
        # Find result JSON (exclude metadata and gpu files)
        for json_file in folder.glob("*.json"):
            if (
                "experiment_metadata" not in json_file.name
                and "gpu_utilization" not in json_file.name
            ):
                # Generate label from folder name: benchmark_cache_aware_pd_grpc -> cache_aware pd grpc
                label = folder.name.replace("benchmark_", "").replace("_", " ")
                results.append((json_file, label))
                break  # One JSON per folder
    return sorted(results, key=lambda x: x[0].parent.name)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 34-37: Helper function `find_gpu_utilization`
```python
def find_gpu_utilization(result_path: Path) -> Path | None:
    """Find GPU utilization JSON in same folder as result."""
    gpu_json = result_path.parent / "gpu_utilization.json"
    return gpu_json if gpu_json.exists() else None
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 40-99: Helper function `generate_summary`
```python
def generate_summary(base_dir: Path) -> str:
    """Generate markdown summary."""
    benchmarks = discover_benchmarks(base_dir)

    if not benchmarks:
        return (
            "## Gateway E2E Genai-Bench Results Summary\n\nNo benchmark results found."
        )

    lines = [
        "## Gateway E2E Genai-Bench Results Summary",
        "",
        "| Scenario | Status | TTFT (s) | E2E Latency (s) | Input Throughput (tok/s) | Output Throughput (tok/s) |",
        "|----------|--------|----------|-----------------|--------------------------|---------------------------|",
    ]

    gpu_sections = []

    for result_path, label in benchmarks:
        try:
            result = BenchmarkResult.from_json(result_path)
        except Exception as e:
            print(f"Warning: Failed to parse {result_path}: {e}", file=sys.stderr)
            lines.append(f"| {label} | ❌ Failed | - | - | - | - |")
            continue

        lines.append(
            f"| {label} | ✅ Success | "
            f"{result.ttft_mean:.2f} | "
            f"{result.e2e_latency_mean:.2f} | "
            f"{result.input_throughput_mean:.0f} | "
            f"{result.output_throughput_mean:.0f} |"
        )

        # GPU utilization
        gpu_path = find_gpu_utilization(result_path)
        if gpu_path:
            gpu = GPUUtilization.from_json(gpu_path)
            if gpu and gpu.per_gpu:
                gpu_lines = [
                    f"### GPU Utilization — {label}",
                    "",
                    f"Overall mean: {gpu.overall_mean:.2f}%",
                    "",
                    "| GPU | Mean (%) | p5 | p10 | p25 | p50 | p75 | p90 | p95 |",
                    "|-----|----------|----|-----|-----|-----|-----|-----|-----|",
                ]
                for gpu_id, stats in sorted(
                    gpu.per_gpu.items(), key=lambda x: int(x[0])
                ):
                    gpu_lines.append(
                        f"| {gpu_id} | {stats.get('mean', 0):.2f} | "
                        f"{stats.get('p5', 0):.2f} | {stats.get('p10', 0):.2f} | "
                        f"{stats.get('p25', 0):.2f} | {stats.get('p50', 0):.2f} | "
                        f"{stats.get('p75', 0):.2f} | {stats.get('p90', 0):.2f} | "
                        f"{stats.get('p95', 0):.2f} |"
                    )
                gpu_sections.append("\n".join(gpu_lines))

    return "\n".join(lines) + "\n" + "\n\n".join(gpu_sections)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 102-115: Helper function `main`
```python
def main() -> None:
    """Main entry point."""
    base_dir = Path(sys.argv[1]) if len(sys.argv) > 1 else Path.cwd()
    summary = generate_summary(base_dir)

    # Write to GITHUB_STEP_SUMMARY if available
    summary_file = os.environ.get("GITHUB_STEP_SUMMARY")
    if summary_file:
        with open(summary_file, "a") as f:
            f.write(summary)
            f.write("\n")
        print(f"Summary written to {summary_file}")
    else:
        print(summary)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 118-119: Command-line entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This main-guard block makes the file executable as a script by invoking the module's top-level routine only when run directly.
**CN:** 这个主入口保护块让文件可以作为脚本直接执行，并仅在被直接运行时调用模块的顶层流程。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Performance measurement and comparison / 性能测量与对比
- Reusable functions: `discover_benchmarks`, `find_gpu_utilization`, `generate_summary`, `main` / 可复用函数：`discover_benchmarks`, `find_gpu_utilization`, `generate_summary`, `main`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `os`, `pathlib`, `sys`
- **Third-party / 第三方**: `results`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
