# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/target_determination/heuristics/utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements target-determination heuristics used to decide which tests should run for a change.
- **Purpose (CN)**: 实现目标判定启发式逻辑，用于决定某次变更应运行哪些测试。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
from __future__ import annotations

import json
import os
import re
import subprocess
from collections import defaultdict
from functools import cache
from pathlib import Path
from typing import cast, TYPE_CHECKING
from urllib.request import Request, urlopen
from warnings import warn
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, json, os, and 8 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、json、os 等共 11 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 13-18
```python


if TYPE_CHECKING:
    from tools.testing.test_run import TestRun
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.testing.test_run. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.testing.test_run。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 19-26
```python
REPO_ROOT = Path(__file__).resolve().parents[4]


def python_test_file_to_test_name(tests: set[str]) -> set[str]:
    prefix = f"test{os.path.sep}"
    valid_tests = {f for f in tests if f.startswith(prefix) and f.endswith(".py")}
    valid_tests = {f[len(prefix) : -len(".py")] for f in valid_tests}
```
- **EN**: This chunk defines `python_test_file_to_test_name`, which implements a focused step inside the testing infrastructure pipeline. Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `python_test_file_to_test_name`，其作用是实现测试基础设施流水线中的一个关键步骤。 REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 27-33
```python
    return valid_tests


@cache
def get_pr_number() -> int | None:
    pr_number = os.environ.get("PR_NUMBER", "")
    if pr_number == "":
```
- **EN**: This chunk defines `get_pr_number`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_pr_number`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 34-40
```python
        re_match = re.match(r"^refs/tags/.*/(\d+)$", os.environ.get("GITHUB_REF", ""))
        if re_match is not None:
            pr_number = re_match.group(1)
    if pr_number != "":
        return int(pr_number)
    return None
```
- **EN**: This chunk continues `get_pr_number` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_pr_number`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 41-52
```python

@cache
def get_merge_base() -> str:
    pr_number = get_pr_number()
    if pr_number is not None:
        github_token = os.environ.get("GITHUB_TOKEN")
        headers = {
            "Accept": "application/vnd.github.v3+json",
            "Authorization": f"token {github_token}",
        }
        url = f"https://api.github.com/repos/pytorch/pytorch/pulls/{pr_number}"
        with urlopen(Request(url, headers=headers)) as conn:
```
- **EN**: This chunk defines `get_merge_base`, which implements a focused step inside the testing infrastructure pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `get_merge_base`，其作用是实现测试基础设施流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 53-64
```python
            pr_info = json.loads(conn.read().decode())
            base = f"origin/{pr_info['base']['ref']}"
        merge_base = (
            subprocess.check_output(["git", "merge-base", base, "HEAD"])
            .decode()
            .strip()
        )
        return merge_base
    default_branch = f"origin/{os.environ.get('GIT_DEFAULT_BRANCH', 'main')}"
    merge_base = (
        subprocess.check_output(["git", "merge-base", default_branch, "HEAD"])
        .decode()
```
- **EN**: This chunk continues `get_merge_base` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_merge_base`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 65-70
```python
        .strip()
    )

    head = subprocess.check_output(["git", "rev-parse", "HEAD"]).decode().strip()

    if merge_base == head:
```
- **EN**: This chunk continues `get_merge_base` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_merge_base`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 71-76
```python
        # We are on the default branch, so check for changes since the last commit
        merge_base = "HEAD^"
    return merge_base


def query_changed_files() -> list[str]:
```
- **EN**: This chunk defines `query_changed_files`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `query_changed_files`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 77-85
```python
    base_commit = get_merge_base()

    proc = subprocess.run(
        ["git", "diff", "--name-only", base_commit, "HEAD"],
        capture_output=True,
        check=False,
    )
    print(f"base_commit: {base_commit}")
```
- **EN**: This chunk continues `query_changed_files` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段延续了 `query_changed_files`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 86-93
```python
    if proc.returncode != 0:
        raise RuntimeError("Unable to get changed files")

    lines = proc.stdout.decode().strip().split("\n")
    lines = [line.strip() for line in lines]
    print(f"Changed files: {lines}")
    return lines
```
- **EN**: This chunk continues `query_changed_files` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `query_changed_files`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 94-99
```python

# File extensions that are documentation-only and don't require running tests
DOCS_ONLY_EXTENSIONS = frozenset({".rst", ".md"})


def is_docs_only_change(changed_files: list[str]) -> bool:
```
- **EN**: This chunk defines `is_docs_only_change`, which implements a focused step inside the testing infrastructure pipeline. Configuration constants such as DOCS_ONLY_EXTENSIONS centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段定义了 `is_docs_only_change`，其作用是实现测试基础设施流水线中的一个关键步骤。 DOCS_ONLY_EXTENSIONS 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 100-106
```python
    """
    Returns True if all changed files are documentation-only files
    (e.g., .rst, .md files) that don't require running tests.
    """
    if not changed_files:
        return False
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `is_docs_only_change` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `is_docs_only_change`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 107-113
```python
    for f in changed_files:
        # Skip empty strings that might come from git diff output
        if not f:
            continue
        # Check if the file extension is in the docs-only set
        ext = os.path.splitext(f)[1].lower()
        if ext not in DOCS_ONLY_EXTENSIONS:
```
- **EN**: This chunk continues `is_docs_only_change` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `is_docs_only_change`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 114-119
```python
            return False

    return True


@cache
```
- **EN**: This chunk continues `is_docs_only_change` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `is_docs_only_change`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 120-131
```python
def get_git_commit_info() -> str:
    """Gets the commit info since the last commit on the default branch."""
    base_commit = get_merge_base()

    return (
        subprocess.check_output(
            ["git", "log", f"{base_commit}..HEAD"],
        )
        .decode()
        .strip()
    )
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_git_commit_info`, which implements a focused step inside the testing infrastructure pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_git_commit_info`，其作用是实现测试基础设施流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 132-143
```python

@cache
def get_issue_or_pr_body(number: int) -> str:
    """Gets the body of an issue or PR"""
    github_token = os.environ.get("GITHUB_TOKEN")
    headers = {
        "Accept": "application/vnd.github.v3+json",
        "Authorization": f"token {github_token}",
    }
    # Despite the 'issues' in the link, this also works for PRs
    url = f"https://api.github.com/repos/pytorch/pytorch/issues/{number}"
    with urlopen(Request(url, headers=headers)) as conn:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_issue_or_pr_body`, which implements a focused step inside the testing infrastructure pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_issue_or_pr_body`，其作用是实现测试基础设施流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 144-155
```python
        body: str = json.loads(conn.read().decode())["body"] or ""
        return body


def normalize_ratings(
    ratings: dict[TestRun, float], max_value: float, min_value: float = 0
) -> dict[TestRun, float]:
    # Takse the ratings, makes the max value into max_value, and proportionally
    # distributes the rest of the ratings.
    # Ex [1,2,3,4] and max_value 8 gets converted to [2,4,6,8]
    # Assumes all rankings are >= 0
    # min_value is what 0 gets mapped to and shifts the values accordingly.  Ex
```
- **EN**: This chunk defines `normalize_ratings`, which implements a focused step inside the testing infrastructure pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `normalize_ratings`，其作用是实现测试基础设施流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 156-161
```python
    # [1,2,3,4], min_value 1, max_value 5 gets converted to [2,3,4,5]
    # Don't modify in place
    if len(ratings) == 0:
        return ratings
    min_rating = min(ratings.values())
    if min_rating <= 0:
```
- **EN**: This chunk continues `normalize_ratings` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `normalize_ratings`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 162-167
```python
        raise AssertionError(f"min_rating must be > 0, got {min_rating}")
    max_rating = max(ratings.values())
    if max_rating <= 0:
        raise AssertionError(f"max_rating must be > 0, got {max_rating}")
    normalized_ratings = {}
    for tf, rank in ratings.items():
```
- **EN**: This chunk continues `normalize_ratings` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `normalize_ratings`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 168-174
```python
        normalized_ratings[tf] = rank / max_rating * (max_value - min_value) + min_value
    return normalized_ratings


def get_ratings_for_tests(file: str | Path) -> dict[str, float]:
    path = REPO_ROOT / file
    if not os.path.exists(path):
```
- **EN**: This chunk defines `get_ratings_for_tests`, which implements a focused step inside the testing infrastructure pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_ratings_for_tests`，其作用是实现测试基础设施流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 175-185
```python
        print(f"could not find path {path}")
        return {}
    with open(path) as f:
        test_file_ratings = cast(dict[str, dict[str, float]], json.load(f))
    try:
        changed_files = query_changed_files()
    except Exception as e:
        warn(f"Can't query changed test files due to {e}")
        return {}
    ratings: dict[str, float] = defaultdict(float)
    for file in changed_files:
```
- **EN**: This chunk continues `get_ratings_for_tests` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_ratings_for_tests`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 186-191
```python
        for test_file, score in test_file_ratings.get(file, {}).items():
            ratings[test_file] += score
    return ratings


def get_correlated_tests(file: str | Path) -> list[str]:
```
- **EN**: This chunk defines `get_correlated_tests`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_correlated_tests`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 192-194
```python
    ratings = get_ratings_for_tests(file)
    prioritize = sorted(ratings, key=lambda x: -ratings[x])
    return prioritize
```
- **EN**: This chunk continues `get_correlated_tests` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_correlated_tests`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

## Key Concepts / 关键概念

- **Testing infrastructure**
  - EN: This file belongs to the testing infrastructure layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于测试基础设施层，应结合同一子目录中的相邻脚本一起理解。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Heuristic decision-making**
  - EN: The code uses lightweight rules to prioritize tests, files, or workflow actions.
  - CN: 代码使用轻量规则来排序测试、文件或工作流动作。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.testing.test_run`
- **Python standard library / Python 标准库**: `__future__`, `json`, `os`, `re`, `subprocess`, `collections`, `functools`, `pathlib`, `typing`, `urllib.request`, `warnings`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `python_test_file_to_test_name`, `get_pr_number`, `get_merge_base`, `query_changed_files`, `DOCS_ONLY_EXTENSIONS`, `is_docs_only_change`, `get_git_commit_info`, `get_issue_or_pr_body`, `normalize_ratings`, `get_ratings_for_tests`, `get_correlated_tests`
