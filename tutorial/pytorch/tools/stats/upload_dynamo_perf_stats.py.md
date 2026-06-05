# upload_dynamo_perf_stats.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stats/upload_dynamo_perf_stats.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements statistics collection and reporting helpers for build, CI, or repository health signals.
- **Purpose (CN)**: 实现统计采集与报告辅助逻辑，用于构建、CI 或仓库健康度信号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
from __future__ import annotations

import argparse
import csv
import hashlib
import json
import os
import re
from pathlib import Path
from tempfile import TemporaryDirectory
from typing import Any
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, argparse, csv, and 7 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、argparse、csv 等共 10 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 13-18
```python
from tools.stats.upload_stats_lib import (
    download_s3_artifacts,
    unzip,
    upload_to_dynamodb,
)
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.upload_stats_lib.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.upload_stats_lib。

### Lines 19-26
```python

ARTIFACTS = [
    "test-reports",
]
ARTIFACT_REGEX = re.compile(
    r"test-reports-test-(?P<name>[\w\-]+)-\d+-\d+-(?P<runner>[\w\.-]+)_(?P<job>\d+).zip"
)
```
- **EN**: Configuration constants such as ARTIFACTS, ARTIFACT_REGEX centralize defaults so later functions share the same policy knobs. Pattern-matching logic extracts structured facts from loosely formatted text inputs.
- **CN**: ARTIFACTS、ARTIFACT_REGEX 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。

### Lines 27-38
```python

def get_perf_stats(
    repo: str,
    workflow_run_id: int,
    workflow_run_attempt: int,
    head_branch: str,
    match_filename: str,
) -> list[dict[str, Any]]:
    match_filename_regex = re.compile(match_filename)
    perf_stats = []
    with TemporaryDirectory() as temp_dir:
        print("Using temporary directory:", temp_dir)
```
- **EN**: This chunk defines `get_perf_stats`, which aggregates signals and turns them into summaries, metrics, or alerts. Pattern-matching logic extracts structured facts from loosely formatted text inputs.
- **CN**: 这一段定义了 `get_perf_stats`，其作用是聚合信号，并将其转化为摘要、指标或告警。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。

### Lines 39-45
```python
        os.chdir(temp_dir)

        for artifact in ARTIFACTS:
            artifact_paths = download_s3_artifacts(
                artifact, workflow_run_id, workflow_run_attempt
            )
```
- **EN**: This chunk continues `get_perf_stats` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_perf_stats`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 46-52
```python
            # Unzip to get perf stats csv files
            for path in artifact_paths:
                m = ARTIFACT_REGEX.match(str(path))
                if not m:
                    print(f"Test report {path} has an invalid name. Skipping")
                    continue
```
- **EN**: This chunk continues `get_perf_stats` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_perf_stats`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 53-59
```python
                test_name = m.group("name")
                runner = m.group("runner")
                job_id = m.group("job")

                # Extract all files
                unzip(path)
```
- **EN**: This chunk continues `get_perf_stats` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_perf_stats`，进一步展开其内部控制流或数据流转。

### Lines 60-65
```python
                for csv_file in Path(".").glob("**/*.csv"):
                    filename = os.path.splitext(os.path.basename(csv_file))[0]
                    if not re.match(match_filename_regex, filename):
                        continue
                    print(f"Processing {filename} from {path}")
```
- **EN**: This chunk continues `get_perf_stats` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_perf_stats`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 66-77
```python
                    with open(csv_file) as csvfile:
                        reader = csv.DictReader(csvfile, delimiter=",")

                        for row in reader:
                            row.update(
                                {
                                    "workflow_id": workflow_run_id,  # type: ignore[dict-item]
                                    "run_attempt": workflow_run_attempt,  # type: ignore[dict-item]
                                    "test_name": test_name,
                                    "runner": runner,
                                    "job_id": job_id,
                                    "filename": filename,
```
- **EN**: This chunk continues `get_perf_stats` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_perf_stats`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 78-85
```python
                                    "head_branch": head_branch,
                                }
                            )
                            perf_stats.append(row)

                    # Done processing the file, removing it
                    os.remove(csv_file)
```
- **EN**: This chunk continues `get_perf_stats` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_perf_stats`，进一步展开其内部控制流或数据流转。

### Lines 86-97
```python
    return perf_stats


def generate_partition_key(repo: str, doc: dict[str, Any]) -> str:
    """
    Generate an unique partition key for the document on DynamoDB
    """
    workflow_id = doc["workflow_id"]
    job_id = doc["job_id"]
    test_name = doc["test_name"]
    filename = doc["filename"]
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `generate_partition_key`, which generates derived source text, templates, or metadata outputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `generate_partition_key`，其作用是生成派生源码文本、模板或元数据输出。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 98-103
```python
    hash_content = hashlib.md5(
        json.dumps(doc).encode("utf-8"), usedforsecurity=False
    ).hexdigest()
    return f"{repo}/{workflow_id}/{job_id}/{test_name}/{filename}/{hash_content}"
```
- **EN**: This chunk continues `generate_partition_key` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `generate_partition_key`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 104-115
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Upload dynamo perf stats from S3 to DynamoDB"
    )
    parser.add_argument(
        "--workflow-run-id",
        type=int,
        required=True,
        help="id of the workflow to get perf stats from",
    )
    parser.add_argument(
        "--workflow-run-attempt",
```
- **EN**: This chunk continues `generate_partition_key` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `generate_partition_key`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 116-127
```python
        type=int,
        required=True,
        help="which retry of the workflow this is",
    )
    parser.add_argument(
        "--repo",
        type=str,
        required=True,
        help="which GitHub repo this workflow run belongs to",
    )
    parser.add_argument(
        "--head-branch",
```
- **EN**: This chunk continues `generate_partition_key` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `generate_partition_key`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 128-139
```python
        type=str,
        required=True,
        help="head branch of the workflow",
    )
    parser.add_argument(
        "--dynamodb-table",
        type=str,
        required=True,
        help="the name of the DynamoDB table to store the stats",
    )
    parser.add_argument(
        "--match-filename",
```
- **EN**: This chunk continues `generate_partition_key` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `generate_partition_key`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 140-151
```python
        type=str,
        default="",
        help="the regex to filter the list of CSV files containing the records to upload",
    )
    args = parser.parse_args()
    perf_stats = get_perf_stats(
        args.repo,
        args.workflow_run_id,
        args.workflow_run_attempt,
        args.head_branch,
        args.match_filename,
    )
```
- **EN**: This chunk continues `generate_partition_key` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs.
- **CN**: 这一段延续了 `generate_partition_key`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。

### Lines 152-157
```python
    upload_to_dynamodb(
        dynamodb_table=args.dynamodb_table,
        repo=args.repo,
        docs=perf_stats,
        generate_partition_key=generate_partition_key,
    )
```
- **EN**: This chunk continues `generate_partition_key` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `generate_partition_key`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **CI statistics**
  - EN: This file belongs to the ci statistics layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于CI 统计层，应结合同一子目录中的相邻脚本一起理解。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.upload_stats_lib`
- **Python standard library / Python 标准库**: `__future__`, `argparse`, `csv`, `hashlib`, `json`, `os`, `re`, `pathlib`, `tempfile`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `ARTIFACTS`, `ARTIFACT_REGEX`, `get_perf_stats`, `generate_partition_key`
