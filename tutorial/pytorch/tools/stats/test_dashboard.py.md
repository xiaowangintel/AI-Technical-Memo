# test_dashboard.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stats/test_dashboard.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements statistics collection and reporting helpers for build, CI, or repository health signals.
- **Purpose (CN)**: 实现统计采集与报告辅助逻辑，用于构建、CI 或仓库健康度信号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```python
from __future__ import annotations

import json
import os
import re
from collections import defaultdict
from functools import lru_cache
from pathlib import Path
from tempfile import TemporaryDirectory
from typing import Any, cast
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, json, os, and 6 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、json、os 等共 9 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 12-22
```python
import requests

from tools.stats.upload_stats_lib import (
    _get_request_headers,
    download_s3_artifacts,
    get_job_id,
    get_s3_resource,
    unzip,
    upload_workflow_stats_to_s3,
)
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.upload_stats_lib; external packages such as requests.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.upload_stats_lib；外部依赖包，如 requests。

### Lines 23-28
```python

REGEX_JOB_INFO = r"(.*) \/ .*test \(([^,]*), .*\)"


@lru_cache(maxsize=1000)
def get_job_name(job_id: int) -> str:
```
- **EN**: This chunk defines `get_job_name`, which implements a focused step inside the ci statistics pipeline. Configuration constants such as REGEX_JOB_INFO centralize defaults so later functions share the same policy knobs. Pattern-matching logic extracts structured facts from loosely formatted text inputs.
- **CN**: 这一段定义了 `get_job_name`，其作用是实现CI 统计流水线中的一个关键步骤。 REGEX_JOB_INFO 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。

### Lines 29-40
```python
    try:
        return cast(
            str,
            requests.get(
                f"https://api.github.com/repos/pytorch/pytorch/actions/jobs/{job_id}",
                headers=_get_request_headers(),
            ).json()["name"],
        )
    except Exception as e:
        print(f"Failed to get job name for job id {job_id}: {e}")
        return "NoJobName"
```
- **EN**: This chunk continues `get_job_name` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_job_name`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 41-49
```python

@lru_cache(maxsize=1000)
def get_build_name(job_name: str) -> str:
    try:
        return re.match(REGEX_JOB_INFO, job_name).group(1)  # type: ignore[union-attr]
    except AttributeError:
        print(f"Failed to match job name: {job_name}")
        return "NoBuildEnv"
```
- **EN**: This chunk defines `get_build_name`, which prepares build-system state and translates configuration into downstream tool invocations. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_build_name`，其作用是准备构建系统状态，并把配置转换为后续工具调用。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 50-58
```python

@lru_cache(maxsize=1000)
def get_test_config(job_name: str) -> str:
    try:
        return re.match(REGEX_JOB_INFO, job_name).group(2)  # type: ignore[union-attr]
    except AttributeError:
        print(f"Failed to match job name: {job_name}")
        return "NoTestConfig"
```
- **EN**: This chunk defines `get_test_config`, which implements a focused step inside the ci statistics pipeline. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_test_config`，其作用是实现CI 统计流水线中的一个关键步骤。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 59-66
```python

def get_td_exclusions(
    workflow_run_id: int, workflow_run_attempt: int
) -> dict[str, Any]:
    with TemporaryDirectory() as temp_dir:
        print("Using temporary directory:", temp_dir)
        os.chdir(temp_dir)
```
- **EN**: This chunk defines `get_td_exclusions`, which implements a focused step inside the ci statistics pipeline.
- **CN**: 这一段定义了 `get_td_exclusions`，其作用是实现CI 统计流水线中的一个关键步骤。

### Lines 67-73
```python
        # Download and extract all the reports (both GHA and S3)
        s3_paths = download_s3_artifacts(
            "test-jsons", workflow_run_id, workflow_run_attempt
        )
        for path in s3_paths:
            unzip(path)
```
- **EN**: This chunk continues `get_td_exclusions` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_td_exclusions`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 74-84
```python
        grouped_tests: dict[str, Any] = defaultdict(lambda: defaultdict(set))
        for td_exclusions in Path(".").glob("**/td_exclusions*.json"):
            with open(td_exclusions) as f:
                exclusions = json.load(f)
                for exclusion in exclusions["excluded"]:
                    job_id = get_job_id(td_exclusions)
                    job_name = get_job_name(job_id)
                    build_name = get_build_name(job_name)
                    test_config = get_test_config(job_name)
                    grouped_tests[build_name][test_config].add(exclusion["test_file"])
```
- **EN**: This chunk continues `get_td_exclusions` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_td_exclusions`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 85-90
```python
        for build_name, build in grouped_tests.items():
            for test_config, test_files in build.items():
                grouped_tests[build_name][test_config] = sorted(test_files)
        return grouped_tests
```
- **EN**: This chunk continues `get_td_exclusions` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_td_exclusions`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 91-98
```python
def get_all_run_attempts(workflow_run_id: int) -> list[int]:
    # Returns all run attempts for a given workflow run id that have test
    # artifacts
    bucket = get_s3_resource().Bucket("gha-artifacts")
    prefix = f"pytorch/pytorch/{workflow_run_id}/"
    objs = bucket.objects.filter(Prefix=prefix)
    run_attempts = set()
    for obj in objs:
```
- **EN**: This chunk defines `get_all_run_attempts`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `get_all_run_attempts`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 99-106
```python
        no_prefix = obj.key[len(prefix) :]
        try:
            run_attempt = int(no_prefix.split("/")[0])
            run_attempts.add(run_attempt)
        except ValueError:
            continue
    return sorted(run_attempts)
```
- **EN**: This chunk continues `get_all_run_attempts` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_all_run_attempts`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 107-116
```python

def upload_additional_info(workflow_run_id: int, workflow_run_attempt: int) -> None:
    exclusions = get_td_exclusions(workflow_run_id, workflow_run_attempt)

    upload_workflow_stats_to_s3(
        workflow_run_id,
        workflow_run_attempt,
        "additional_info/td_exclusions",
        [exclusions],
    )
```
- **EN**: This chunk defines `upload_additional_info`, which parses or loads structured input into tool-friendly data structures.
- **CN**: 这一段定义了 `upload_additional_info`，其作用是把结构化输入解析或加载为工具可处理的数据结构。

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
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **REGEX_JOB_INFO**
  - EN: `REGEX_JOB_INFO` is one of the main local symbols exposed or implemented here.
  - CN: `REGEX_JOB_INFO` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.upload_stats_lib`
- **Python standard library / Python 标准库**: `__future__`, `json`, `os`, `re`, `collections`, `functools`, `pathlib`, `tempfile`, `typing`
- **External packages / 外部依赖包**: `requests`
- **Primary symbols in this file / 本文件核心符号**: `REGEX_JOB_INFO`, `get_job_name`, `get_build_name`, `get_test_config`, `get_td_exclusions`, `get_all_run_attempts`, `upload_additional_info`
