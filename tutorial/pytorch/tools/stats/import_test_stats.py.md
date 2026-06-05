# import_test_stats.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stats/import_test_stats.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements statistics collection and reporting helpers for build, CI, or repository health signals.
- **Purpose (CN)**: 实现统计采集与报告辅助逻辑，用于构建、CI 或仓库健康度信号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
#!/usr/bin/env python3

from __future__ import annotations

import datetime
import json
import os
import shutil
from pathlib import Path
from typing import Any, cast, TYPE_CHECKING
from urllib.request import urlopen
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, datetime, json, and 5 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、datetime、json 等共 8 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 13-19
```python

if TYPE_CHECKING:
    from collections.abc import Callable


REPO_ROOT = Path(__file__).resolve().parents[2]
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as collections.abc. Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 collections.abc。 REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 20-26
```python

def get_disabled_issues() -> list[str]:
    reenabled_issues = os.getenv("REENABLED_ISSUES", "")
    issue_numbers = reenabled_issues.split(",")
    print("Ignoring disabled issues: ", issue_numbers)
    return issue_numbers
```
- **EN**: This chunk defines `get_disabled_issues`, which implements a focused step inside the ci statistics pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_disabled_issues`，其作用是实现CI 统计流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 27-38
```python

DISABLED_TESTS_FILE = ".pytorch-disabled-tests.json"
ADDITIONAL_CI_FILES_FOLDER = Path(".additional_ci_files")
TEST_TIMES_FILE = "test-times.json"
TEST_CLASS_TIMES_FILE = "test-class-times.json"
TEST_FILE_RATINGS_FILE = "test-file-ratings.json"
TEST_CLASS_RATINGS_FILE = "test-class-ratings.json"
TD_HEURISTIC_PROFILING_FILE = "td_heuristic_profiling.json"
TD_HEURISTIC_HISTORICAL_EDITED_FILES = "td_heuristic_historical_edited_files.json"
TD_HEURISTIC_PREVIOUSLY_FAILED = "previous_failures.json"
TD_HEURISTIC_PREVIOUSLY_FAILED_ADDITIONAL = "previous_failures_additional.json"
```
- **EN**: This chunk continues `get_disabled_issues` and expands its internal control flow or data movement. Configuration constants such as DISABLED_TESTS_FILE, ADDITIONAL_CI_FILES_FOLDER, TEST_TIMES_FILE, and 7 more centralize defaults so later functions share the same policy knobs. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `get_disabled_issues`，进一步展开其内部控制流或数据流转。 DISABLED_TESTS_FILE、ADDITIONAL_CI_FILES_FOLDER、TEST_TIMES_FILE 等共 10 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 39-50
```python
FILE_CACHE_LIFESPAN_SECONDS = datetime.timedelta(hours=3).seconds


def fetch_and_cache(
    dirpath: str | Path,
    name: str,
    url: str,
    process_fn: Callable[[dict[str, Any]], dict[str, Any]],
) -> dict[str, Any]:
    """
    This fetch and cache utils allows sharing between different process.
    """
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `fetch_and_cache`, which implements a focused step inside the ci statistics pipeline. Configuration constants such as FILE_CACHE_LIFESPAN_SECONDS centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `fetch_and_cache`，其作用是实现CI 统计流水线中的一个关键步骤。 FILE_CACHE_LIFESPAN_SECONDS 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 51-56
```python
    Path(dirpath).mkdir(exist_ok=True)

    path = os.path.join(dirpath, name)
    print(f"Downloading {url} to {path}")

    def is_cached_file_valid() -> bool:
```
- **EN**: This chunk defines `is_cached_file_valid`, which implements a focused step inside the ci statistics pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段定义了 `is_cached_file_valid`，其作用是实现CI 统计流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 57-64
```python
        # Check if the file is new enough (see: FILE_CACHE_LIFESPAN_SECONDS). A real check
        # could make a HEAD request and check/store the file's ETag
        fname = Path(path)
        now = datetime.datetime.now()
        mtime = datetime.datetime.fromtimestamp(fname.stat().st_mtime)
        diff = now - mtime
        return diff.total_seconds() < FILE_CACHE_LIFESPAN_SECONDS
```
- **EN**: This chunk continues `is_cached_file_valid` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `is_cached_file_valid`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 65-70
```python
    if os.path.exists(path) and is_cached_file_valid():
        # Another test process already download the file, so don't re-do it
        with open(path) as f:
            return cast(dict[str, Any], json.load(f))

    for _ in range(3):
```
- **EN**: This chunk continues `is_cached_file_valid` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `is_cached_file_valid`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 71-81
```python
        try:
            contents = urlopen(url, timeout=5).read().decode("utf-8")
            processed_contents = process_fn(json.loads(contents))
            with open(path, "w") as f:
                f.write(json.dumps(processed_contents))
            return processed_contents
        except Exception as e:
            print(f"Could not download {url} because: {e}.")
    print(f"All retries exhausted, downloading {url} failed.")
    return {}
```
- **EN**: This chunk continues `is_cached_file_valid` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `is_cached_file_valid`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 82-89
```python

def get_test_times() -> dict[str, dict[str, float]]:
    return get_from_test_infra_generated_stats(
        "test-times.json",
        TEST_TIMES_FILE,
        "Couldn't download test times...",
    )
```
- **EN**: This chunk defines `get_test_times`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_test_times`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 90-97
```python

def get_test_class_times() -> dict[str, dict[str, float]]:
    return get_from_test_infra_generated_stats(
        "test-class-times.json",
        TEST_CLASS_TIMES_FILE,
        "Couldn't download test times...",
    )
```
- **EN**: This chunk defines `get_test_class_times`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_test_class_times`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 98-106
```python

def get_disabled_tests(
    dirpath: str, filename: str = DISABLED_TESTS_FILE
) -> dict[str, Any] | None:
    def process_disabled_test(the_response: dict[str, Any]) -> dict[str, Any]:
        # remove re-enabled tests and condense even further by getting rid of pr_num
        disabled_issues = get_disabled_issues()
        disabled_test_from_issues = {}
        for test_name, (pr_num, link, platforms) in the_response.items():
```
- **EN**: This chunk defines `process_disabled_test`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `process_disabled_test`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 107-113
```python
            if pr_num not in disabled_issues:
                disabled_test_from_issues[test_name] = (
                    link,
                    platforms,
                )
        return disabled_test_from_issues
```
- **EN**: This chunk continues `process_disabled_test` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `process_disabled_test`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 114-120
```python
    try:
        url = "https://ossci-metrics.s3.amazonaws.com/disabled-tests-condensed.json"
        return fetch_and_cache(dirpath, filename, url, process_disabled_test)
    except Exception:
        print("Couldn't download test skip set, leaving all tests enabled...")
        return {}
```
- **EN**: This chunk continues `process_disabled_test` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `process_disabled_test`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 121-128
```python

def get_test_file_ratings() -> dict[str, Any]:
    return get_from_test_infra_generated_stats(
        "file_test_rating.json",
        TEST_FILE_RATINGS_FILE,
        "Couldn't download test file ratings file, not reordering...",
    )
```
- **EN**: This chunk defines `get_test_file_ratings`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_test_file_ratings`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 129-136
```python

def get_test_class_ratings() -> dict[str, Any]:
    return get_from_test_infra_generated_stats(
        "file_test_class_rating.json",
        TEST_CLASS_RATINGS_FILE,
        "Couldn't download test class ratings file, not reordering...",
    )
```
- **EN**: This chunk defines `get_test_class_ratings`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_test_class_ratings`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 137-144
```python

def get_td_heuristic_historial_edited_files_json() -> dict[str, Any]:
    return get_from_test_infra_generated_stats(
        "td_heuristic_historical_edited_files.json",
        TD_HEURISTIC_HISTORICAL_EDITED_FILES,
        "Couldn't download td_heuristic_historical_edited_files.json, not reordering...",
    )
```
- **EN**: This chunk defines `get_td_heuristic_historial_edited_files_json`, which narrows a larger candidate set down to the items relevant for the current workflow. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_td_heuristic_historial_edited_files_json`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 145-152
```python

def get_td_heuristic_profiling_json() -> dict[str, Any]:
    return get_from_test_infra_generated_stats(
        "td_heuristic_profiling.json",
        TD_HEURISTIC_PROFILING_FILE,
        "Couldn't download td_heuristic_profiling.json not reordering...",
    )
```
- **EN**: This chunk defines `get_td_heuristic_profiling_json`, which narrows a larger candidate set down to the items relevant for the current workflow. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_td_heuristic_profiling_json`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 153-162
```python

def copy_pytest_cache() -> None:
    original_path = REPO_ROOT / ".pytest_cache/v/cache/lastfailed"
    if not original_path.exists():
        return
    shutil.copyfile(
        original_path,
        REPO_ROOT / ADDITIONAL_CI_FILES_FOLDER / TD_HEURISTIC_PREVIOUSLY_FAILED,
    )
```
- **EN**: This chunk defines `copy_pytest_cache`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `copy_pytest_cache`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 163-168
```python

def copy_additional_previous_failures() -> None:
    original_path = (
        REPO_ROOT / ".pytest_cache" / TD_HEURISTIC_PREVIOUSLY_FAILED_ADDITIONAL
    )
    if not original_path.exists():
```
- **EN**: This chunk defines `copy_additional_previous_failures`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `copy_additional_previous_failures`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 169-176
```python
        return
    shutil.copyfile(
        original_path,
        REPO_ROOT
        / ADDITIONAL_CI_FILES_FOLDER
        / TD_HEURISTIC_PREVIOUSLY_FAILED_ADDITIONAL,
    )
```
- **EN**: This chunk continues `copy_additional_previous_failures` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `copy_additional_previous_failures`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 177-182
```python

def get_from_test_infra_generated_stats(
    from_file: str, to_file: str, failure_explanation: str
) -> dict[str, Any]:
    url = f"https://raw.githubusercontent.com/pytorch/test-infra/generated-stats/stats/{from_file}"
    try:
```
- **EN**: This chunk defines `get_from_test_infra_generated_stats`, which generates derived source text, templates, or metadata outputs. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `get_from_test_infra_generated_stats`，其作用是生成派生源码文本、模板或元数据输出。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 183-188
```python
        return fetch_and_cache(
            REPO_ROOT / ADDITIONAL_CI_FILES_FOLDER, to_file, url, lambda x: x
        )
    except Exception:
        print(failure_explanation)
        return {}
```
- **EN**: This chunk continues `get_from_test_infra_generated_stats` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_from_test_infra_generated_stats`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

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
- **Heuristic decision-making**
  - EN: The code uses lightweight rules to prioritize tests, files, or workflow actions.
  - CN: 代码使用轻量规则来排序测试、文件或工作流动作。
- **Pytest integration**
  - EN: The file plugs into pytest collection, execution, or reporting behavior.
  - CN: 该文件接入 pytest 的收集、执行或报告行为。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `datetime`, `json`, `os`, `shutil`, `pathlib`, `typing`, `urllib.request`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `get_disabled_issues`, `DISABLED_TESTS_FILE`, `ADDITIONAL_CI_FILES_FOLDER`, `TEST_TIMES_FILE`, `TEST_CLASS_TIMES_FILE`, `TEST_FILE_RATINGS_FILE`, `TEST_CLASS_RATINGS_FILE`, `TD_HEURISTIC_PROFILING_FILE`, `TD_HEURISTIC_HISTORICAL_EDITED_FILES`, `TD_HEURISTIC_PREVIOUSLY_FAILED`, `TD_HEURISTIC_PREVIOUSLY_FAILED_ADDITIONAL`
