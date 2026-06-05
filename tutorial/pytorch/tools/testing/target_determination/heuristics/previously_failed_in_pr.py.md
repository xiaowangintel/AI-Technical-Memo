# previously_failed_in_pr.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/target_determination/heuristics/previously_failed_in_pr.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements target-determination heuristics used to decide which tests should run for a change.
- **Purpose (CN)**: 实现目标判定启发式逻辑，用于决定某次变更应运行哪些测试。
## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```python
from __future__ import annotations

import json
import os
from pathlib import Path
from typing import Any
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, json, os, and 2 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、json、os 等共 5 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 8-19
```python
from tools.stats.import_test_stats import (
    ADDITIONAL_CI_FILES_FOLDER,
    TD_HEURISTIC_PREVIOUSLY_FAILED,
    TD_HEURISTIC_PREVIOUSLY_FAILED_ADDITIONAL,
)
from tools.testing.target_determination.heuristics.interface import (
    HeuristicInterface,
    TestPrioritizations,
)
from tools.testing.target_determination.heuristics.utils import (
    python_test_file_to_test_name,
)
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.import_test_stats, tools.testing.target_determination.heuristics.interface, tools.testing.target_determination.heuristics.utils.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.import_test_stats、tools.testing.target_determination.heuristics.interface、tools.testing.target_determination.heuristics.utils。

### Lines 20-25
```python
from tools.testing.test_run import TestRun


REPO_ROOT = Path(__file__).resolve().parents[4]
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.testing.test_run. Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.testing.test_run。 REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 26-35
```python
class PreviouslyFailedInPR(HeuristicInterface):
    def __init__(self, **kwargs: dict[str, Any]) -> None:
        super().__init__(**kwargs)

    def get_prediction_confidence(self, tests: list[str]) -> TestPrioritizations:
        critical_tests = get_previous_failures() | read_additional_test_failures_file()
        return TestPrioritizations(
            tests, {TestRun(test): 1 for test in critical_tests if test in tests}
        )
```
- **EN**: It introduces classes such as PreviouslyFailedInPR, which package state and behavior for this tooling task. This chunk defines `get_prediction_confidence`, which implements a focused step inside the testing infrastructure pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 它引入了 PreviouslyFailedInPR 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `get_prediction_confidence`，其作用是实现测试基础设施流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 36-46
```python

def get_previous_failures() -> set[str]:
    path = REPO_ROOT / ADDITIONAL_CI_FILES_FOLDER / TD_HEURISTIC_PREVIOUSLY_FAILED
    if not os.path.exists(path):
        print(f"could not find path {path}")
        return set()
    with open(path) as f:
        return python_test_file_to_test_name(
            _parse_prev_failing_test_files(json.load(f))
        )
```
- **EN**: This chunk defines `get_previous_failures`, which implements a focused step inside the testing infrastructure pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_previous_failures`，其作用是实现测试基础设施流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 47-53
```python

def _parse_prev_failing_test_files(last_failed_tests: dict[str, bool]) -> set[str]:
    prioritized_tests = set()

    # The keys are formatted as "test_file.py::test_class::test_method[params]"
    # We just need the test_file part
    for test in last_failed_tests:
```
- **EN**: This chunk defines `_parse_prev_failing_test_files`, which parses or loads structured input into tool-friendly data structures. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `_parse_prev_failing_test_files`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 54-60
```python
        parts = test.split("::")
        if len(parts) > 1:
            test_file = parts[0]
            prioritized_tests.add(test_file)

    return prioritized_tests
```
- **EN**: This chunk continues `_parse_prev_failing_test_files` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_parse_prev_failing_test_files`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 61-68
```python

def gen_additional_test_failures_file(tests: list[str]) -> None:
    # Segfaults usually result in no xml and some tests don't run through pytest
    # (ex doctests).  In these cases, there will be no entry in the pytest
    # cache, so we should generate a separate file for them and upload it to s3
    # along with the pytest cache
    pytest_cache_dir = REPO_ROOT / ".pytest_cache"
    if not os.path.exists(pytest_cache_dir):
```
- **EN**: This chunk defines `gen_additional_test_failures_file`, which generates derived source text, templates, or metadata outputs. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `gen_additional_test_failures_file`，其作用是生成派生源码文本、模板或元数据输出。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 69-74
```python
        os.makedirs(pytest_cache_dir)
    with open(pytest_cache_dir / TD_HEURISTIC_PREVIOUSLY_FAILED_ADDITIONAL, "w") as f:
        json.dump(tests, f, indent=2)


def read_additional_test_failures_file() -> set[str]:
```
- **EN**: This chunk defines `read_additional_test_failures_file`, which parses or loads structured input into tool-friendly data structures. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段定义了 `read_additional_test_failures_file`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 75-80
```python
    path = (
        REPO_ROOT
        / ADDITIONAL_CI_FILES_FOLDER
        / TD_HEURISTIC_PREVIOUSLY_FAILED_ADDITIONAL
    )
    if not os.path.exists(path):
```
- **EN**: This chunk continues `read_additional_test_failures_file` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `read_additional_test_failures_file`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 81-86
```python
        print(f"could not find path {path}")
        return set()
    with open(path) as f:
        s = set(json.load(f))
        print(f"additional failures: {s}")
        return s
```
- **EN**: This chunk continues `read_additional_test_failures_file` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `read_additional_test_failures_file`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

## Key Concepts / 关键概念

- **Testing infrastructure**
  - EN: This file belongs to the testing infrastructure layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于测试基础设施层，应结合同一子目录中的相邻脚本一起理解。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **Heuristic decision-making**
  - EN: The code uses lightweight rules to prioritize tests, files, or workflow actions.
  - CN: 代码使用轻量规则来排序测试、文件或工作流动作。
- **Pytest integration**
  - EN: The file plugs into pytest collection, execution, or reporting behavior.
  - CN: 该文件接入 pytest 的收集、执行或报告行为。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.import_test_stats`, `tools.testing.target_determination.heuristics.interface`, `tools.testing.target_determination.heuristics.utils`, `tools.testing.test_run`
- **Python standard library / Python 标准库**: `__future__`, `json`, `os`, `pathlib`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `PreviouslyFailedInPR`, `get_previous_failures`, `_parse_prev_failing_test_files`, `gen_additional_test_failures_file`, `read_additional_test_failures_file`
