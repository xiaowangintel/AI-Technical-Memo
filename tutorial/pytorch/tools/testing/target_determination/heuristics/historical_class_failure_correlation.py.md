# historical_class_failure_correlation.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/target_determination/heuristics/historical_class_failure_correlation.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements target-determination heuristics used to decide which tests should run for a change.
- **Purpose (CN)**: 实现目标判定启发式逻辑，用于决定某次变更应运行哪些测试。
## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```python
from __future__ import annotations

import json
import os
from collections import defaultdict
from typing import Any, cast
from warnings import warn
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, json, os, and 3 more. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、json、os 等共 6 项。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 9-20
```python
from tools.stats.import_test_stats import (
    ADDITIONAL_CI_FILES_FOLDER,
    TEST_CLASS_RATINGS_FILE,
)
from tools.testing.target_determination.heuristics.interface import (
    HeuristicInterface,
    TestPrioritizations,
)
from tools.testing.target_determination.heuristics.utils import (
    normalize_ratings,
    query_changed_files,
    REPO_ROOT,
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.import_test_stats, tools.testing.target_determination.heuristics.interface, tools.testing.target_determination.heuristics.utils.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.import_test_stats、tools.testing.target_determination.heuristics.interface、tools.testing.target_determination.heuristics.utils。

### Lines 21-30
```python
)
from tools.testing.test_run import TestRun


class HistoricalClassFailurCorrelation(HeuristicInterface):
    """
    This heuristic prioritizes test classes that have historically tended to fail
    when the files edited by current PR were modified.
    """
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.testing.test_run. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as HistoricalClassFailurCorrelation, which package state and behavior for this tooling task.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.testing.test_run。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 HistoricalClassFailurCorrelation 等类，用来封装该工具任务所需的状态与行为。

### Lines 31-40
```python
    def __init__(self, **kwargs: Any) -> None:
        super().__init__(**kwargs)

    def get_prediction_confidence(self, tests: list[str]) -> TestPrioritizations:
        ratings = _get_ratings_for_tests(set(tests))
        test_ratings = {
            TestRun(k): v for (k, v) in ratings.items() if TestRun(k).test_file in tests
        }
        return TestPrioritizations(tests, normalize_ratings(test_ratings, 0.25))
```
- **EN**: This chunk defines `get_prediction_confidence`, which implements a focused step inside the testing infrastructure pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_prediction_confidence`，其作用是实现测试基础设施流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 41-50
```python

def _get_historical_test_class_correlations() -> dict[str, dict[str, float]]:
    path = REPO_ROOT / ADDITIONAL_CI_FILES_FOLDER / TEST_CLASS_RATINGS_FILE
    if not os.path.exists(path):
        print(f"could not find path {path}")
        return {}
    with open(path) as f:
        test_class_correlations = cast(dict[str, dict[str, float]], json.load(f))
        return test_class_correlations
```
- **EN**: This chunk defines `_get_historical_test_class_correlations`, which implements a focused step inside the testing infrastructure pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_get_historical_test_class_correlations`，其作用是实现测试基础设施流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 51-56
```python

def _get_ratings_for_tests(
    tests_to_run: set[str],
) -> dict[str, float]:
    # Get the files edited
    try:
```
- **EN**: This chunk defines `_get_ratings_for_tests`, which implements a focused step inside the testing infrastructure pipeline. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `_get_ratings_for_tests`，其作用是实现测试基础设施流水线中的一个关键步骤。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 57-63
```python
        changed_files = query_changed_files()
    except Exception as e:
        warn(f"Can't query changed test files due to {e}")
        return {}

    test_class_correlations = _get_historical_test_class_correlations()
    if not test_class_correlations:
```
- **EN**: This chunk continues `_get_ratings_for_tests` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_get_ratings_for_tests`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 64-69
```python
        return {}

    # Find the tests failures that are correlated with the edited files.
    # Filter the list to only include tests we want to run.
    ratings: dict[str, float] = defaultdict(float)
    for file in changed_files:
```
- **EN**: This chunk continues `_get_ratings_for_tests` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_get_ratings_for_tests`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 70-75
```python
        for qualified_test_class, score in test_class_correlations.get(
            file, {}
        ).items():
            # qualified_test_class looks like "test_file::test_class"
            test_file, test_class = qualified_test_class.split("::")
            if test_file in tests_to_run:
```
- **EN**: This chunk continues `_get_ratings_for_tests` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_get_ratings_for_tests`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 76-81
```python
                ratings[qualified_test_class] += score

    return ratings


def _rank_correlated_tests(
```
- **EN**: This chunk defines `_rank_correlated_tests`, which implements a focused step inside the testing infrastructure pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_rank_correlated_tests`，其作用是实现测试基础设施流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 82-91
```python
    tests_to_run: list[str],
) -> list[str]:
    # Find the tests failures that are correlated with the edited files.
    # Filter the list to only include tests we want to run.
    # pyrefly: ignore [bad-assignment]
    tests_to_run = set(tests_to_run)
    # pyrefly: ignore [bad-argument-type]
    ratings = _get_ratings_for_tests(tests_to_run)
    prioritize = sorted(ratings, key=lambda x: -ratings[x])
    return prioritize
```
- **EN**: This chunk continues `_rank_correlated_tests` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_rank_correlated_tests`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

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
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **HistoricalClassFailurCorrelation**
  - EN: `HistoricalClassFailurCorrelation` is one of the main local symbols exposed or implemented here.
  - CN: `HistoricalClassFailurCorrelation` 是此处暴露或实现的主要局部符号之一。
- **_get_historical_test_class_correlations**
  - EN: `_get_historical_test_class_correlations` is one of the main local symbols exposed or implemented here.
  - CN: `_get_historical_test_class_correlations` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.import_test_stats`, `tools.testing.target_determination.heuristics.interface`, `tools.testing.target_determination.heuristics.utils`, `tools.testing.test_run`
- **Python standard library / Python 标准库**: `__future__`, `json`, `os`, `collections`, `typing`, `warnings`
- **Primary symbols in this file / 本文件核心符号**: `HistoricalClassFailurCorrelation`, `_get_historical_test_class_correlations`, `_get_ratings_for_tests`, `_rank_correlated_tests`
