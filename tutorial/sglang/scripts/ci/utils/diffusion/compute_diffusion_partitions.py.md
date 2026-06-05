# compute_diffusion_partitions.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/diffusion/compute_diffusion_partitions.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `compute_diffusion_partitions` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `compute_diffusion_partitions` 流程，主要负责CI 编排、测试执行。它属于 `diffusion` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-7: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Compute dynamic partitions for diffusion CI tests.

This script runs on lightweight CI runners without sglang dependencies and uses
AST parsing to extract parametrized cases plus standalone files from source.
"""
```
**EN:** Compute dynamic partitions for diffusion CI tests.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 9-23: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import importlib.util
import json
import math
import os
import sys
from pathlib import Path

from diffusion_case_parser import (
    BASELINE_REL_PATH,
    RUN_SUITE_REL_PATH,
    DiffusionSuiteInfo,
    collect_diffusion_suites,
    resolve_case_config_path,
)
```
**EN:** This block loads argparse, importlib.util, json, math, os, sys, pathlib, diffusion_case_parser. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, importlib.util, json, math, os, sys, pathlib, diffusion_case_parser。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 26-35: Defines the `_load_partitioning_helpers` routine / 定义 `_load_partitioning_helpers` 例程
```python
def _load_partitioning_helpers():
    repo_root = Path(__file__).resolve().parents[4]
    helper_path = repo_root / "python/sglang/multimodal_gen/test/partitioning.py"
    spec = importlib.util.spec_from_file_location(
        "diffusion_test_partitioning", helper_path
    )
    module = importlib.util.module_from_spec(spec)
    sys.modules[spec.name] = module
    spec.loader.exec_module(module)
    return module.PartitionItem, module.partition_items_by_lpt
```
**EN:** This block defines `_load_partitioning_helpers`. It takes no explicit parameters at the top level. Internally it touches filesystem paths.
**CN:** 该代码块定义了 `_load_partitioning_helpers`。它在顶层定义中不接收显式参数。内部会处理文件系统路径。

### Lines 38-41: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
PartitionItem, partition_items_by_lpt = _load_partitioning_helpers()

SUITE_OUTPUT_NAMES = {"1-gpu": "1gpu", "2-gpu": "2gpu", "1-gpu-b200": "b200"}
DEFAULT_STANDALONE_EST_TIME_SECONDS = 300.0
```
**EN:** This section defines SUITE_OUTPUT_NAMES, DEFAULT_STANDALONE_EST_TIME_SECONDS, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 SUITE_OUTPUT_NAMES, DEFAULT_STANDALONE_EST_TIME_SECONDS，用于把可复用的默认值集中在模块顶部。

### Lines 44-64: Defines the `validate_suite_case_coverage` routine / 定义 `validate_suite_case_coverage` 例程
```python
def validate_suite_case_coverage(suites: dict[str, DiffusionSuiteInfo]) -> None:
    """
    Guardrail: dynamic diffusion suites must contain parametrized cases.
    """
    suites_with_no_cases = []
    for suite_name in SUITE_OUTPUT_NAMES:
        suite_info = suites.get(suite_name)
        if suite_info is None:
            print(f"Error: Required suite '{suite_name}' not found in parsed suites.")
            sys.exit(1)
        if len(suite_info.cases) == 0:
            suites_with_no_cases.append(suite_name)

    if suites_with_no_cases:
        joined = ", ".join(suites_with_no_cases)
        print(
            "Error: Parsed zero parametrized cases for diffusion suites: "
            f"{joined}. This usually means run_suite case imports changed but "
            "diffusion parser logic was not updated."
        )
        sys.exit(1)
```
**EN:** This block defines `validate_suite_case_coverage`. Guardrail: dynamic diffusion suites must contain parametrized cases. It accepts 1 parameter(s): suites. Internally it emits status messages, controls process exit status.
**CN:** 该代码块定义了 `validate_suite_case_coverage`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：suites。内部会输出状态信息、控制进程退出状态。

### Lines 67-89: Defines the `compute_partition_count` routine / 定义 `compute_partition_count` 例程
```python
def compute_partition_count(
    total_time_seconds: float,
    min_time_seconds: float,
    target_time_seconds: float,
    max_time_seconds: float,
    max_partitions: int,
) -> int:
    if total_time_seconds <= 0:
        return 0

    min_partition_count = max(1, math.ceil(total_time_seconds / max_time_seconds))
    max_partition_count = max(1, math.floor(total_time_seconds / min_time_seconds))

    min_partition_count = min(min_partition_count, max_partitions)
    max_partition_count = min(max_partition_count, max_partitions)

    if max_partition_count < min_partition_count:
        fallback_count = math.ceil(total_time_seconds / target_time_seconds)
        return max(1, min(fallback_count, max_partitions))

    preferred_count = math.ceil(total_time_seconds / target_time_seconds)
    preferred_count = max(1, min(preferred_count, max_partitions))
    return max(min_partition_count, min(preferred_count, max_partition_count))
```
**EN:** This block defines `compute_partition_count`. It accepts 5 parameter(s): total_time_seconds, min_time_seconds, target_time_seconds, max_time_seconds, max_partitions. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `compute_partition_count`。它接收 5 个参数：total_time_seconds, min_time_seconds, target_time_seconds, max_time_seconds, max_partitions。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 92-115: Defines the `build_partition_items` routine / 定义 `build_partition_items` 例程
```python
def build_partition_items(
    suite_info: DiffusionSuiteInfo, include_standalone: bool = True
) -> list[PartitionItem]:
    items = [
        PartitionItem(kind="case", item_id=case.case_id, est_time=case.est_time)
        for case in suite_info.cases
    ]
    if not include_standalone:
        return items

    items.extend(
        PartitionItem(
            kind="standalone",
            item_id=standalone_file,
            est_time=suite_info.standalone_est_times.get(
                standalone_file, DEFAULT_STANDALONE_EST_TIME_SECONDS
            ),
            used_fallback_estimate=(
                standalone_file in suite_info.missing_standalone_estimates
            ),
        )
        for standalone_file in suite_info.standalone_files
    )
    return items
```
**EN:** This block defines `build_partition_items`. It accepts 2 parameter(s): suite_info, include_standalone. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `build_partition_items`。它接收 2 个参数：suite_info, include_standalone。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 118-121: Defines the `build_matrix` routine / 定义 `build_matrix` 例程
```python
def build_matrix(partition_count: int) -> dict:
    if partition_count <= 0:
        return {"include": []}
    return {"include": [{"part": i} for i in range(partition_count)]}
```
**EN:** This block defines `build_matrix`. It accepts 1 parameter(s): partition_count. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `build_matrix`。它接收 1 个参数：partition_count。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 124-147: Defines the `build_partition_plan` routine / 定义 `build_partition_plan` 例程
```python
def build_partition_plan(
    suite_name: str,
    partitions: list[list[PartitionItem]],
) -> dict:
    return {
        "suite": suite_name,
        "partition_count": len(partitions),
        "partitions": [
            {
                "part": idx,
                "case_ids": [item.item_id for item in partition if item.kind == "case"],
                "standalone_files": [
                    item.item_id for item in partition if item.kind == "standalone"
                ],
                "missing_standalone_estimates": [
                    item.item_id
                    for item in partition
                    if item.kind == "standalone" and item.used_fallback_estimate
                ],
                "estimated_time": round(sum(item.est_time for item in partition), 1),
            }
            for idx, partition in enumerate(partitions)
        ],
    }
```
**EN:** This block defines `build_partition_plan`. It accepts 2 parameter(s): suite_name, partitions. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `build_partition_plan`。它接收 2 个参数：suite_name, partitions。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 150-156: Defines the `output_github_value` routine / 定义 `output_github_value` 例程
```python
def output_github_value(name: str, value: dict) -> None:
    value_json = json.dumps(value, separators=(",", ":"))
    github_output = os.environ.get("GITHUB_OUTPUT")
    if github_output:
        with open(github_output, "a", encoding="utf-8") as f:
            f.write(f"{name}={value_json}\n")
    print(f"{name}={value_json}")
```
**EN:** This block defines `output_github_value`. It accepts 2 parameter(s): name, value. Internally it reads or writes files, emits status messages.
**CN:** 该代码块定义了 `output_github_value`。它接收 2 个参数：name, value。内部会读写文件、输出状态信息。

### Lines 159-164: Defines the `output_github_scalar` routine / 定义 `output_github_scalar` 例程
```python
def output_github_scalar(name: str, value: str) -> None:
    github_output = os.environ.get("GITHUB_OUTPUT")
    if github_output:
        with open(github_output, "a", encoding="utf-8") as f:
            f.write(f"{name}={value}\n")
    print(f"{name}={value}")
```
**EN:** This block defines `output_github_scalar`. It accepts 2 parameter(s): name, value. Internally it reads or writes files, emits status messages.
**CN:** 该代码块定义了 `output_github_scalar`。它接收 2 个参数：name, value。内部会读写文件、输出状态信息。

### Lines 167-216: Defines the `print_suite_summary` routine / 定义 `print_suite_summary` 例程
```python
def print_suite_summary(
    suite_name: str,
    suite_info: DiffusionSuiteInfo,
    partitions: list[list[PartitionItem]],
    include_standalone: bool = True,
) -> None:
    total_time = sum(
        item.est_time
        for item in build_partition_items(
            suite_info, include_standalone=include_standalone
        )
    )
    print(f"{suite_name.upper()} suite:")
    print(f"  Cases: {len(suite_info.cases)}")
    standalone_label = "Standalone files"
    if not include_standalone:
        standalone_label = "Standalone files ignored"
    print(f"  {standalone_label}: {len(suite_info.standalone_files)}")
    print(
        f"  Missing standalone estimates: {len(suite_info.missing_standalone_estimates)}"
    )
    if suite_info.missing_standalone_estimates:
        print(
            f"  Fallback standalone estimate: "
            f"{DEFAULT_STANDALONE_EST_TIME_SECONDS:.1f}s"
        )
        for standalone_file in suite_info.missing_standalone_estimates:
            print(f"    - {standalone_file}")
    print(f"  Total estimated time: {total_time:.1f}s ({total_time/60:.1f} min)")
    print(f"  Selected partitions: {len(partitions)}")
    print()

    print("  Partition assignments:")
    for idx, partition in enumerate(partitions):
        partition_time = sum(item.est_time for item in partition)
        print(f"    Partition {idx}:")
        print(
            f"      Estimated time: {partition_time:.1f}s ({partition_time/60:.1f} min)"
        )
        for item in partition:
            fallback_suffix = (
                ", fallback estimate"
                if item.kind == "standalone" and item.used_fallback_estimate
                else ""
            )
            print(
                f"      - {item.kind}: {item.item_id} "
                f"({item.est_time:.1f}s{fallback_suffix})"
            )
    print()
```
**EN:** This block defines `print_suite_summary`. It accepts 4 parameter(s): suite_name, suite_info, partitions, include_standalone. Internally it emits status messages.
**CN:** 该代码块定义了 `print_suite_summary`。它接收 4 个参数：suite_name, suite_info, partitions, include_standalone。内部会输出状态信息。

### Lines 219-311: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(
        description="Compute diffusion test partitions for CI"
    )
    parser.add_argument(
        "--min-time",
        type=float,
        default=1200.0,
        help="Minimum desired partition time in seconds (default: 1200 = 20 minutes)",
    )
    parser.add_argument(
        "--target-time",
        type=float,
        default=1800.0,
        help="Preferred partition time in seconds (default: 1800 = 30 minutes)",
    )
    parser.add_argument(
        "--max-time",
        type=float,
        default=2400.0,
        help="Maximum desired partition time in seconds (default: 2400 = 40 minutes)",
    )
    parser.add_argument(
        "--max-partitions",
        type=int,
        default=10,
        help="Maximum number of partitions (default: 10)",
    )
    parser.add_argument(
        "--parametrized-only",
        action="store_true",
        help="Only partition DiffusionTestCase parametrized cases.",
    )
    args = parser.parse_args()

    script_dir = Path(__file__).resolve().parent
    repo_root = script_dir.parent.parent.parent.parent

    baseline_path = repo_root / BASELINE_REL_PATH
    run_suite_path = repo_root / RUN_SUITE_REL_PATH

    if not run_suite_path.exists():
        print(f"Error: Run suite not found: {run_suite_path}")
        sys.exit(1)
    try:
        case_config_path = resolve_case_config_path(repo_root, run_suite_path)
    except (RuntimeError, FileNotFoundError) as exc:
        print(f"Error: {exc}")
        sys.exit(1)

    suites = collect_diffusion_suites(
        case_config_path,
        run_suite_path,
        baseline_path,
    )
    validate_suite_case_coverage(suites)

    print("=== Diffusion Partition Computation ===")
    print(f"Min partition time: {args.min_time}s ({args.min_time/60:.1f} min)")
    print(f"Target partition time: {args.target_time}s ({args.target_time/60:.1f} min)")
    print(f"Max partition time: {args.max_time}s ({args.max_time/60:.1f} min)")
    print()

    for suite_name, suite_info in suites.items():
        if suite_name not in SUITE_OUTPUT_NAMES:
            continue

        items = build_partition_items(
            suite_info, include_standalone=not args.parametrized_only
        )
        total_time = sum(item.est_time for item in items)
        partition_count = compute_partition_count(
            total_time_seconds=total_time,
            min_time_seconds=args.min_time,
            target_time_seconds=args.target_time,
            max_time_seconds=args.max_time,
            max_partitions=args.max_partitions,
        )
        partitions = partition_items_by_lpt(items, partition_count)

        print_suite_summary(
            suite_name,
            suite_info,
            partitions,
            include_standalone=not args.parametrized_only,
        )

        output_name = SUITE_OUTPUT_NAMES[suite_name]
        output_github_value(f"matrix-{output_name}", build_matrix(partition_count))
        output_github_scalar(f"partition-count-{output_name}", str(partition_count))
        output_github_value(
            f"plan-{output_name}", build_partition_plan(suite_name, partitions)
        )
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it touches filesystem paths, parses CLI arguments, emits status messages, checks file existence.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会处理文件系统路径、解析命令行参数、输出状态信息、检查文件是否存在。

### Lines 314-315: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `importlib`, `json`, `math`, `os`, `pathlib`, `sys`
- **Third-party modules / 第三方模块**: `diffusion_case_parser`
