# results.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/benchmarks/results.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises results behavior in the benchmark suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端基准测试 中与 results 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module docstring
```python
"""Benchmark result dataclasses for parsing genai-bench and GPU monitor output."""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 3-7: Imports and dependencies
```python
from __future__ import annotations

import json
from dataclasses import dataclass
from pathlib import Path
```
**EN:** This block imports `__future__`, `json`, `dataclasses`, `pathlib`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 8-77: Class `BenchmarkResult`
```python


@dataclass
class BenchmarkResult:
    """Parsed benchmark metrics from genai-bench output."""

    ttft_mean: float
    e2e_latency_mean: float
    input_throughput_mean: float
    output_throughput_mean: float
    file_name: str

    @classmethod
    def from_json(cls, path: Path) -> "BenchmarkResult":
        """Parse benchmark results from JSON file."""
        with path.open() as f:
            data = json.load(f)
        stats = data.get("aggregated_metrics", {}).get("stats", {})
        return cls(
            ttft_mean=float(stats.get("ttft", {}).get("mean", float("inf"))),
            e2e_latency_mean=float(
                stats.get("e2e_latency", {}).get("mean", float("inf"))
            ),
            input_throughput_mean=float(
                stats.get("input_throughput", {}).get("mean", 0.0)
            ),
            output_throughput_mean=float(
                stats.get("output_throughput", {}).get("mean", 0.0)
            ),
            file_name=path.name,
        )

    def log(self, experiment: str, logger) -> None:
        """Log benchmark results."""
        logger.info(
            "genai-bench[%s] %s ttft=%.3fs e2e=%.3fs input=%.1f tok/s output=%.1f tok/s",
            experiment,
            self.file_name,
            self.ttft_mean,
            self.e2e_latency_mean,
            self.input_throughput_mean,
            self.output_throughput_mean,
        )

    def validate(self, thresholds: dict) -> None:
        """Validate metrics against thresholds."""
        checks = [
            ("ttft_mean_max", self.ttft_mean, "<=", "TTFT"),
            ("e2e_latency_mean_max", self.e2e_latency_mean, "<=", "E2E latency"),
            (
                "input_throughput_mean_min",
                self.input_throughput_mean,
                ">=",
                "Input throughput",
            ),
            (
                "output_throughput_mean_min",
                self.output_throughput_mean,
                ">=",
                "Output throughput",
            ),
        ]
        for key, value, op, name in checks:
            if key not in thresholds:
                continue
            threshold = thresholds[key]
            if op == "<=" and value > threshold:
                raise AssertionError(f"{name}: {value:.2f} > {threshold}")
            if op == ">=" and value < threshold:
                raise AssertionError(f"{name}: {value:.2f} < {threshold}")
```
**EN:** Class `BenchmarkResult` groups related state and behavior. It exposes 3 method(s) that implement the module's primary abstraction.
**CN:** 类 `BenchmarkResult` 将相关状态与行为封装在一起，提供 3 个方法来实现本模块的核心抽象。

### Lines 78-98: Class `GPUUtilization`
```python


@dataclass
class GPUUtilization:
    """Parsed GPU utilization metrics from gpu_monitor output."""

    overall_mean: float
    per_gpu: dict[str, dict[str, float]]

    @classmethod
    def from_json(cls, path: Path) -> "GPUUtilization | None":
        """Parse GPU utilization from JSON file."""
        try:
            with path.open() as f:
                data = json.load(f)
            return cls(
                overall_mean=float(data.get("overall", {}).get("mean", 0)),
                per_gpu=data.get("per_gpu", {}),
            )
        except Exception:
            return None
```
**EN:** Class `GPUUtilization` groups related state and behavior. It exposes 1 method(s) that implement the module's primary abstraction.
**CN:** 类 `GPUUtilization` 将相关状态与行为封装在一起，提供 1 个方法来实现本模块的核心抽象。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Performance measurement and comparison / 性能测量与对比
- Primary classes: `BenchmarkResult`, `GPUUtilization` / 主要类：`BenchmarkResult`, `GPUUtilization`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `json`, `pathlib`
- **Third-party / 第三方**: None explicitly imported / 未显式导入
- **Internal / 内部模块**: None explicitly imported / 未显式导入
