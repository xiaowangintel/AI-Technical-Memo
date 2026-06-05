# map_ec2_to_arc.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/map_ec2_to_arc.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11 / 第 1-11 行

````python
#!/usr/bin/env python3
"""Map EC2 runner labels to ARC equivalents using .github/arc.yaml.

Takes a GitHub Actions test matrix, replaces each runner with its ARC
equivalent, and prints the updated matrix as JSON.

Usage:
    python map_ec2_to_arc.py --prefix mt- '{ include: [
      { config: "default", shard: 1, num_shards: 5, runner: "mt-linux.4xlarge" },
    ]}'
"""
````

- EN: This block touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 13-19 / 第 13-19 行

````python
import argparse
import json
import os
import sys
from pathlib import Path

import yaml
````

- EN: This block imports dependencies such as `argparse`, `json`, `os`, `sys`; parses command-line arguments for script entry points; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；为脚本入口解析命令行参数；序列化或解析结构化数据文件。

### Lines 22-35 / 第 22-35 行

````python
def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="Map EC2 runner labels to ARC runner labels in a test matrix"
    )
    parser.add_argument(
        "matrix",
        help="GitHub Actions test matrix string to transform",
    )
    parser.add_argument(
        "--prefix",
        default="",
        help="Runner prefix to strip from labels (e.g. 'mt-')",
    )
    return parser.parse_args()
````

- EN: This block defines callable units such as `parse_args`; parses command-line arguments for script entry points; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 38-47 / 第 38-47 行

````python
def strip_prefix(label: str, prefix: str) -> str:
    if prefix and label.startswith(prefix):
        return label[len(prefix) :]
    return label


def load_mapping(arc_yaml: Path) -> dict[str, str]:
    with open(arc_yaml) as f:
        data = yaml.safe_load(f)
    return data["runner_mapping"]
````

- EN: This block defines callable units such as `strip_prefix`, `load_mapping`; branches on runtime conditions or configuration choices; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 50-61 / 第 50-61 行

````python
def set_output(name: str, val: str) -> None:
    print(f"Setting {name}={val}")
    github_output = os.getenv("GITHUB_OUTPUT")
    if github_output:
        with open(github_output, "a") as f:
            print(f"{name}={val}", file=f)


def main() -> None:
    args = parse_args()
    arc_yaml = Path(__file__).resolve().parent.parent / "arc.yaml"
    mapping = load_mapping(arc_yaml)
````

- EN: This block defines callable units such as `set_output`, `main`; branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 63-71 / 第 63-71 行

````python
    matrix = yaml.safe_load(args.matrix)
    if not matrix:
        set_output("test-matrix", args.matrix)
        return

    entries = matrix.get("include", [])
    if not entries:
        set_output("test-matrix", json.dumps(matrix))
        return
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 73-85 / 第 73-85 行

````python
    # TODO(huydo): onnxruntime uses hardware_concurrency() to size its thread
    # pool, which sees all host CPUs (e.g., 192) on ARC k8s instead of the
    # container's cpuset (e.g., 16). This causes pthread_setaffinity_np errors.
    # Skip onnx tests on ARC until the onnxruntime session options are fixed to
    # use cgroup-aware CPU counts.
    excluded_configs = {"onnx"}
    filtered = []
    for entry in entries:
        if entry.get("config") in excluded_configs:
            print(f"Excluding config '{entry['config']}' from ARC test matrix")
            continue
        filtered.append(entry)
    matrix["include"] = filtered
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 87-99 / 第 87-99 行

````python
    for entry in filtered:
        if "runner" not in entry:
            continue
        clean = strip_prefix(entry["runner"].strip(), args.prefix)
        if clean not in mapping:
            print(f"error: no ARC runner found for '{clean}'", file=sys.stderr)
            sys.exit(1)
        mapped = mapping[clean]
        # Passthrough runners (e.g. linux.rocm.gpu.2, linux.idc.xpu) are not
        # OSDC-managed so they keep their original label without the prefix.
        entry["runner"] = mapped if mapped == clean else args.prefix + mapped

    set_output("test-matrix", json.dumps(matrix))
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 102-103 / 第 102-103 行

````python
if __name__ == "__main__":
    main()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。
- EN: CLI entry points — the module exposes command-line arguments so developers or CI jobs can drive automation predictably.
  CN: 命令行入口——该模块暴露命令行参数，使开发者或 CI 作业能够以可预测方式驱动自动化逻辑。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `argparse`, `json`, `os`, `sys`, `pathlib`
- Internal imports / 内部导入: none
- External imports / 外部导入: `yaml`
- Classes / 类: none
- Functions / 函数: `parse_args`, `strip_prefix`, `load_mapping`, `set_output`, `main`
- Test entry points / 测试入口: none
