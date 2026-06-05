# save_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/save_metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `save_metrics` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `save_metrics` 流程，主要负责CI 编排、测试执行。它属于 `utils` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-13: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Collect and save performance metrics from nightly benchmark results.

This script reads benchmark result JSON files from performance profile directories
and saves them with metadata for artifact collection in CI.

Usage:
    python3 scripts/ci/utils/save_metrics.py \
        --gpu-config 8-gpu-h200 \
        --partition 0 \
        --run-id 12345678 \
        --output test/metrics-8gpu-h200-partition-0.json
"""
```
**EN:** Collect and save performance metrics from nightly benchmark results.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 15-20: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import glob
import json
import os
import sys
from datetime import datetime, timezone
```
**EN:** This block loads argparse, glob, json, os, sys, datetime. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, glob, json, os, sys, datetime。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 23-30: Defines the `find_result_files` routine / 定义 `find_result_files` 例程
```python
def find_result_files(search_dirs: list[str]) -> list[str]:
    """Find all results_*.json files in the given directories."""
    result_files = set()
    for search_dir in search_dirs:
        if os.path.exists(search_dir):
            pattern = os.path.join(search_dir, "**/results_*.json")
            result_files.update(glob.glob(pattern, recursive=True))
    return list(result_files)
```
**EN:** This block defines `find_result_files`. Find all results_*.json files in the given directories. It accepts 1 parameter(s): search_dirs. Internally it checks file existence.
**CN:** 该代码块定义了 `find_result_files`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：search_dirs。内部会检查文件是否存在。

### Lines 33-43: Defines the `parse_result_file` routine / 定义 `parse_result_file` 例程
```python
def parse_result_file(filepath: str) -> list[dict]:
    """Parse a benchmark result JSON file."""
    try:
        with open(filepath, "r", encoding="utf-8") as f:
            data = json.load(f)
        if isinstance(data, list):
            return data
        return [data]
    except (json.JSONDecodeError, OSError) as e:
        print(f"Warning: Failed to parse {filepath}: {e}")
        return []
```
**EN:** This block defines `parse_result_file`. Parse a benchmark result JSON file. It accepts 1 parameter(s): filepath. Internally it loads structured data, reads or writes files, emits status messages.
**CN:** 该代码块定义了 `parse_result_file`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：filepath。内部会加载结构化数据、读写文件、输出状态信息。

### Lines 46-66: Defines the `transform_benchmark_result` routine / 定义 `transform_benchmark_result` 例程
```python
def transform_benchmark_result(result: dict, gpu_config: str, partition: int) -> dict:
    """Transform a benchmark result to the metrics schema.

    Note: input_len and output_len are preserved here for the flat benchmarks list,
    but are also used as grouping keys in benchmarks_by_io_len.
    """
    # Handle None values safely for numeric conversions
    latency = result.get("latency")
    last_ttft = result.get("last_ttft")

    return {
        "batch_size": result.get("batch_size"),
        "input_len": result.get("input_len"),
        "output_len": result.get("output_len"),
        "latency_ms": latency * 1000 if latency is not None else None,
        "input_throughput": result.get("input_throughput"),
        "output_throughput": result.get("output_throughput"),
        "overall_throughput": result.get("overall_throughput"),
        "ttft_ms": last_ttft * 1000 if last_ttft is not None else None,
        "acc_length": result.get("acc_length"),
    }
```
**EN:** This block defines `transform_benchmark_result`. Transform a benchmark result to the metrics schema. It accepts 3 parameter(s): result, gpu_config, partition. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `transform_benchmark_result`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：result, gpu_config, partition。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 69-71: Defines the `get_io_len_key` routine / 定义 `get_io_len_key` 例程
```python
def get_io_len_key(input_len: int, output_len: int) -> str:
    """Generate a key for input/output length combination."""
    return f"{input_len}_{output_len}"
```
**EN:** This block defines `get_io_len_key`. Generate a key for input/output length combination. It accepts 2 parameter(s): input_len, output_len. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_io_len_key`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：input_len, output_len。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 74-132: Defines the `group_results_by_model` routine / 定义 `group_results_by_model` 例程
```python
def group_results_by_model(
    results: list[dict], gpu_config: str, partition: int
) -> list[dict]:
    """Group benchmark results by model, variant, and server_args.

    Results are organized with two benchmark structures:
    - benchmarks: flat list of all benchmarks (for backward compatibility)
    - benchmarks_by_io_len: nested structure grouped by input/output length combinations
    """
    groups = {}

    for result in results:
        model_path = result.get("model_path", "unknown")
        run_name = result.get("run_name", "default")
        variant = run_name if run_name != "default" else None
        server_args = result.get("server_args")
        # Convert server_args list to tuple for use as dict key (lists are not hashable)
        server_args_key = tuple(server_args) if server_args else None

        key = (model_path, variant, server_args_key)
        if key not in groups:
            groups[key] = {
                "gpu_config": gpu_config,
                "partition": partition,
                "model": model_path,
                "variant": variant,
                "server_args": server_args,
                "benchmarks": [],
                "benchmarks_by_io_len": {},
            }

        transformed = transform_benchmark_result(result, gpu_config, partition)

        # Add to flat benchmarks list (backward compatibility)
        groups[key]["benchmarks"].append(transformed)

        # Add to nested benchmarks_by_io_len structure
        input_len = result.get("input_len")
        output_len = result.get("output_len")
        if input_len is not None and output_len is not None:
            io_key = get_io_len_key(input_len, output_len)
            if io_key not in groups[key]["benchmarks_by_io_len"]:
                groups[key]["benchmarks_by_io_len"][io_key] = {
                    "input_len": input_len,
                    "output_len": output_len,
                    "benchmarks": [],
                }
            # For the nested structure, exclude input_len and output_len from individual benchmarks
            # since they're already in the parent
            nested_benchmark = {
                k: v
                for k, v in transformed.items()
                if k not in ("input_len", "output_len")
            }
            groups[key]["benchmarks_by_io_len"][io_key]["benchmarks"].append(
                nested_benchmark
            )

    return list(groups.values())
```
**EN:** This block defines `group_results_by_model`. Group benchmark results by model, variant, and server_args. It accepts 3 parameter(s): results, gpu_config, partition. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `group_results_by_model`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：results, gpu_config, partition。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 135-186: Defines the `save_metrics` routine / 定义 `save_metrics` 例程
```python
def save_metrics(
    gpu_config: str,
    partition: int,
    run_id: str,
    output_file: str,
    search_dirs: list[str],
) -> bool:
    """Collect metrics and save to output file."""
    timestamp = datetime.now(timezone.utc).isoformat()

    # Find all result files
    result_files = find_result_files(search_dirs)
    print(f"Found {len(result_files)} result file(s)")

    grouped = []
    if not result_files:
        print("No benchmark result files found")
    else:
        # Parse all result files
        all_results = []
        for filepath in sorted(result_files):
            print(f"  Reading: {filepath}")
            results = parse_result_file(filepath)
            all_results.extend(results)
        print(f"Total benchmark results: {len(all_results)}")

        # Group by model/variant
        grouped = group_results_by_model(all_results, gpu_config, partition)

    # Create metrics structure
    metrics = {
        "run_id": run_id,
        "timestamp": timestamp,
        "gpu_config": gpu_config,
        "partition": partition,
        "results": grouped,
    }

    # Ensure output directory exists and write output
    try:
        os.makedirs(os.path.dirname(output_file) or ".", exist_ok=True)
        with open(output_file, "w", encoding="utf-8") as f:
            json.dump(metrics, f, indent=2)

        if not result_files:
            print(f"Created empty metrics file: {output_file}")
        else:
            print(f"Saved metrics to: {output_file}")
        return True
    except OSError as e:
        print(f"Error writing metrics file: {e}")
        return False
```
**EN:** This block defines `save_metrics`. Collect metrics and save to output file. It accepts 5 parameter(s): gpu_config, partition, run_id, output_file, search_dirs. Internally it reads or writes files, emits status messages.
**CN:** 该代码块定义了 `save_metrics`。其文档字符串说明了该逻辑的预期职责。它接收 5 个参数：gpu_config, partition, run_id, output_file, search_dirs。内部会读写文件、输出状态信息。

### Lines 189-241: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(
        description="Collect performance metrics from benchmark results"
    )
    parser.add_argument(
        "--gpu-config",
        required=True,
        help="GPU configuration (e.g., 8-gpu-h200, 8-gpu-b200)",
    )
    parser.add_argument(
        "--partition",
        type=int,
        required=True,
        help="Partition number (0, 1, 2, etc.)",
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
        "--search-dir",
        action="append",
        default=[],
        dest="search_dirs",
        help="Directory to search for result files (can be specified multiple times)",
    )

    args = parser.parse_args()

    # Default search directories if none specified
    search_dirs = args.search_dirs or [
        "test/performance_profiles_8_gpu",
        "test/performance_profiles_text_models",
        "test/performance_profiles_vlms",
        "test",
        ".",
    ]

    success = save_metrics(
        gpu_config=args.gpu_config,
        partition=args.partition,
        run_id=args.run_id,
        output_file=args.output,
        search_dirs=search_dirs,
    )

    sys.exit(0 if success else 1)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, controls process exit status.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、控制进程退出状态。

### Lines 244-245: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
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
- **Standard library / 标准库**: `argparse`, `datetime`, `glob`, `json`, `os`, `sys`
