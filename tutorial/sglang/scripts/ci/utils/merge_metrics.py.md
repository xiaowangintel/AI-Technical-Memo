# merge_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/merge_metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `merge_metrics` workflow in SGLang. It mainly handles CI orchestration. / 该Python 模块用于支撑 SGLang 中的 `merge_metrics` 流程，主要负责CI 编排。它属于 `utils` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-13: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Merge per-partition metrics into a consolidated metrics file.

This script reads all per-partition metric JSON files and consolidates them
into a single JSON file with run-level metadata.

Usage:
    python3 scripts/ci/utils/merge_metrics.py \
        --input-dir metrics/ \
        --output consolidated-metrics-12345678.json \
        --run-id 12345678 \
        --commit-sha abc123def456
"""
```
**EN:** Merge per-partition metrics into a consolidated metrics file.
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

### Lines 23-33: Defines the `find_partition_files` routine / 定义 `find_partition_files` 例程
```python
def find_partition_files(input_dir: str) -> list[str]:
    """Find all partition metric files in the input directory."""
    patterns = [
        os.path.join(input_dir, "**/metrics-*.json"),
        os.path.join(input_dir, "**/diffusion-metrics-*.json"),
        os.path.join(input_dir, "**/comparison-metrics-*.json"),
    ]
    files = set()
    for pattern in patterns:
        files.update(glob.glob(pattern, recursive=True))
    return list(files)
```
**EN:** This block defines `find_partition_files`. Find all partition metric files in the input directory. It accepts 1 parameter(s): input_dir. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `find_partition_files`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：input_dir。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 36-43: Defines the `load_partition_metrics` routine / 定义 `load_partition_metrics` 例程
```python
def load_partition_metrics(filepath: str) -> dict | None:
    """Load a partition metrics file."""
    try:
        with open(filepath, "r", encoding="utf-8") as f:
            return json.load(f)
    except (json.JSONDecodeError, OSError) as e:
        print(f"Warning: Failed to load {filepath}: {e}")
        return None
```
**EN:** This block defines `load_partition_metrics`. Load a partition metrics file. It accepts 1 parameter(s): filepath. Internally it loads structured data, reads or writes files, emits status messages.
**CN:** 该代码块定义了 `load_partition_metrics`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：filepath。内部会加载结构化数据、读写文件、输出状态信息。

### Lines 46-94: Defines the `merge_metrics` routine / 定义 `merge_metrics` 例程
```python
def merge_metrics(
    input_dir: str,
    output_file: str,
    run_id: str,
    commit_sha: str,
    branch: str | None = None,
) -> bool:
    """Merge all partition metrics into a consolidated file."""
    run_date = datetime.now(timezone.utc).isoformat()

    # Find all partition files
    partition_files = find_partition_files(input_dir)
    print(f"Found {len(partition_files)} partition file(s)")

    all_results = []
    if not partition_files:
        print("No partition metrics files found")
    else:
        # Load all partition files
        for filepath in sorted(partition_files):
            print(f"  Reading: {filepath}")
            metrics = load_partition_metrics(filepath)
            if metrics and "results" in metrics:
                all_results.extend(metrics["results"])
        print(f"Total results collected: {len(all_results)}")

    # Create consolidated structure
    consolidated = {
        "run_id": run_id,
        "run_date": run_date,
        "commit_sha": commit_sha,
        "branch": branch,
        "results": all_results,
    }

    # Ensure output directory exists and write output
    try:
        os.makedirs(os.path.dirname(output_file) or ".", exist_ok=True)
        with open(output_file, "w", encoding="utf-8") as f:
            json.dump(consolidated, f, indent=2)

        if not partition_files:
            print(f"Created empty consolidated file: {output_file}")
        else:
            print(f"Saved consolidated metrics to: {output_file}")
        return True
    except OSError as e:
        print(f"Error writing consolidated file: {e}")
        return False
```
**EN:** This block defines `merge_metrics`. Merge all partition metrics into a consolidated file. It accepts 5 parameter(s): input_dir, output_file, run_id, commit_sha, branch. Internally it reads or writes files, emits status messages.
**CN:** 该代码块定义了 `merge_metrics`。其文档字符串说明了该逻辑的预期职责。它接收 5 个参数：input_dir, output_file, run_id, commit_sha, branch。内部会读写文件、输出状态信息。

### Lines 97-137: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(
        description="Merge per-partition metrics into consolidated file"
    )
    parser.add_argument(
        "--input-dir",
        required=True,
        help="Directory containing partition metric files",
    )
    parser.add_argument(
        "--output",
        required=True,
        help="Output file path for consolidated metrics JSON",
    )
    parser.add_argument(
        "--run-id",
        required=True,
        help="GitHub Actions run ID",
    )
    parser.add_argument(
        "--commit-sha",
        required=True,
        help="Git commit SHA",
    )
    parser.add_argument(
        "--branch",
        default=None,
        help="Git branch name (optional)",
    )

    args = parser.parse_args()

    success = merge_metrics(
        input_dir=args.input_dir,
        output_file=args.output,
        run_id=args.run_id,
        commit_sha=args.commit_sha,
        branch=args.branch,
    )

    sys.exit(0 if success else 1)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, controls process exit status.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、控制进程退出状态。

### Lines 140-141: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
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
