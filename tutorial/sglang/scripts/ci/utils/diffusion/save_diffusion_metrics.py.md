# save_diffusion_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/diffusion/save_diffusion_metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `save_diffusion_metrics` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `save_diffusion_metrics` 流程，主要负责CI 编排、测试执行。它属于 `diffusion` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-13: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Collect and save diffusion performance metrics for artifact collection in CI.

This script reads diffusion test results from the pytest stash and saves them
with metadata for the performance dashboard.

Usage:
    python3 scripts/ci/utils/diffusion/save_diffusion_metrics.py \
        --gpu-config 1-gpu-h100 \
        --run-id 12345678 \
        --output test/diffusion-metrics-1gpu.json \
        --results-json test/diffusion-results.json
"""
```
**EN:** Collect and save diffusion performance metrics for artifact collection in CI.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 15-19: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import json
import os
import sys
from datetime import datetime, timezone
```
**EN:** This block loads argparse, json, os, sys, datetime. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, json, os, sys, datetime。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 22-34: Defines the `load_diffusion_results` routine / 定义 `load_diffusion_results` 例程
```python
def load_diffusion_results(results_file: str) -> list[dict]:
    """Load diffusion performance results from JSON file."""
    if not os.path.exists(results_file):
        print(f"Warning: Results file not found: {results_file}")
        return []

    try:
        with open(results_file, "r", encoding="utf-8") as f:
            data = json.load(f)
        return data if isinstance(data, list) else [data]
    except (json.JSONDecodeError, OSError) as e:
        print(f"Warning: Failed to parse {results_file}: {e}")
        return []
```
**EN:** This block defines `load_diffusion_results`. Load diffusion performance results from JSON file. It accepts 1 parameter(s): results_file. Internally it loads structured data, reads or writes files, checks file existence, emits status messages.
**CN:** 该代码块定义了 `load_diffusion_results`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：results_file。内部会加载结构化数据、读写文件、检查文件是否存在、输出状态信息。

### Lines 37-58: Defines the `transform_diffusion_result` routine / 定义 `transform_diffusion_result` 例程
```python
def transform_diffusion_result(result: dict, gpu_config: str) -> dict:
    """Transform a diffusion result to match dashboard expectations.

    Dashboard expects:
    - Separate test_name, class_name
    - Numeric metrics in consistent units
    - Optional modality field
    """
    return {
        "test_name": result.get("test_name"),
        "class_name": result.get("class_name"),
        "modality": result.get("modality", "image"),
        "e2e_ms": result.get("e2e_ms"),
        "avg_denoise_ms": result.get("avg_denoise_ms"),
        "median_denoise_ms": result.get("median_denoise_ms"),
        "stage_metrics": result.get("stage_metrics", {}),
        "sampled_steps": result.get("sampled_steps", {}),
        # Video-specific metrics (if present)
        "frames_per_second": result.get("frames_per_second"),
        "total_frames": result.get("total_frames"),
        "avg_frame_time_ms": result.get("avg_frame_time_ms"),
    }
```
**EN:** This block defines `transform_diffusion_result`. Transform a diffusion result to match dashboard expectations. It accepts 2 parameter(s): result, gpu_config. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `transform_diffusion_result`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：result, gpu_config。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 61-81: Defines the `group_results_by_class` routine / 定义 `group_results_by_class` 例程
```python
def group_results_by_class(results: list[dict], gpu_config: str) -> list[dict]:
    """Group diffusion results by test class (suite).

    Returns list with one entry per test class, containing all tests in that class.
    """
    groups = {}

    for result in results:
        class_name = result.get("class_name", "unknown")

        if class_name not in groups:
            groups[class_name] = {
                "gpu_config": gpu_config,
                "test_suite": class_name,
                "tests": [],
            }

        transformed = transform_diffusion_result(result, gpu_config)
        groups[class_name]["tests"].append(transformed)

    return list(groups.values())
```
**EN:** This block defines `group_results_by_class`. Group diffusion results by test class (suite). It accepts 2 parameter(s): results, gpu_config. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `group_results_by_class`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：results, gpu_config。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 84-122: Defines the `save_metrics` routine / 定义 `save_metrics` 例程
```python
def save_metrics(
    gpu_config: str,
    run_id: str,
    output_file: str,
    results_file: str,
) -> bool:
    """Collect diffusion metrics and save to output file."""
    timestamp = datetime.now(timezone.utc).isoformat()

    # Load diffusion results
    raw_results = load_diffusion_results(results_file)
    print(f"Loaded {len(raw_results)} diffusion test result(s)")

    # Group by test class
    grouped = group_results_by_class(raw_results, gpu_config)

    # Create metrics structure
    metrics = {
        "run_id": run_id,
        "timestamp": timestamp,
        "gpu_config": gpu_config,
        "test_type": "diffusion",
        "results": grouped,
    }

    # Ensure output directory exists and write output
    try:
        os.makedirs(os.path.dirname(output_file) or ".", exist_ok=True)
        with open(output_file, "w", encoding="utf-8") as f:
            json.dump(metrics, f, indent=2)

        if not raw_results:
            print(f"Created empty metrics file: {output_file}")
        else:
            print(f"Saved diffusion metrics to: {output_file}")
        return True
    except OSError as e:
        print(f"Error writing metrics file: {e}")
        return False
```
**EN:** This block defines `save_metrics`. Collect diffusion metrics and save to output file. It accepts 4 parameter(s): gpu_config, run_id, output_file, results_file. Internally it reads or writes files, emits status messages.
**CN:** 该代码块定义了 `save_metrics`。其文档字符串说明了该逻辑的预期职责。它接收 4 个参数：gpu_config, run_id, output_file, results_file。内部会读写文件、输出状态信息。

### Lines 125-159: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(
        description="Collect diffusion performance metrics from test results"
    )
    parser.add_argument(
        "--gpu-config",
        required=True,
        help="GPU configuration (e.g., 1-gpu-h100, 2-gpu-h100)",
    )
    parser.add_argument(
        "--run-id",
        required=True,
        help="GitHub Actions run ID",
    )
    parser.add_argument(
        "--output",
        required=True,
        help="Output file path for metrics JSON",
    )
    parser.add_argument(
        "--results-json",
        required=True,
        help="Path to diffusion results JSON file",
    )

    args = parser.parse_args()

    success = save_metrics(
        gpu_config=args.gpu_config,
        run_id=args.run_id,
        output_file=args.output,
        results_file=args.results_json,
    )

    sys.exit(0 if success else 1)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, controls process exit status.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、控制进程退出状态。

### Lines 162-163: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `datetime`, `json`, `os`, `sys`
