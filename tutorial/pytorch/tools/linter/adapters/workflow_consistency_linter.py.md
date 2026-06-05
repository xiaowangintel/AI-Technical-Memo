# workflow_consistency_linter.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/workflow_consistency_linter.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```python
# /// script
# requires-python = ">=3.10"
# dependencies = [
#   "pyyaml==6.0.2",
# ]
# ///
"""Checks for consistency of jobs between different GitHub workflows.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 9-20
```python
Any job with a specific `sync-tag` must match all other jobs with the same `sync-tag`.
"""

from __future__ import annotations

import argparse
import itertools
import json
from collections import defaultdict
from enum import Enum
from pathlib import Path
from typing import Any, NamedTuple, TYPE_CHECKING
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, argparse, itertools, and 5 more. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、argparse、itertools 等共 8 项。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 21-26
```python

from yaml import dump, load


REPO_ROOT = Path(__file__).resolve().parent.parent.parent.parent
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as yaml. Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 yaml。 REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 27-33
```python

if TYPE_CHECKING:
    from collections.abc import Iterable


# Safely load fast C Yaml loader/dumper if they are available
try:
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as collections.abc. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 collections.abc。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 34-39
```python
    from yaml import CSafeLoader as Loader
except ImportError:
    from yaml import SafeLoader as Loader  # type: ignore[assignment, misc]


class LintSeverity(str, Enum):
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as yaml. It introduces classes such as LintSeverity, which package state and behavior for this tooling task. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 yaml。 它引入了 LintSeverity 等类，用来封装该工具任务所需的状态与行为。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 40-45
```python
    ERROR = "error"
    WARNING = "warning"
    ADVICE = "advice"
    DISABLED = "disabled"
```
- **EN**: This chunk continues `LintSeverity` and expands its internal control flow or data movement. Configuration constants such as ERROR, WARNING, ADVICE, and 1 more centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段延续了 `LintSeverity`，进一步展开其内部控制流或数据流转。 ERROR、WARNING、ADVICE 等共 4 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 46-56
```python
class LintMessage(NamedTuple):
    path: str | None
    line: int | None
    char: int | None
    code: str
    severity: LintSeverity
    name: str
    original: str | None
    replacement: str | None
    description: str | None
```
- **EN**: It introduces classes such as LintMessage, which package state and behavior for this tooling task. This chunk continues `LintMessage` and expands its internal control flow or data movement.
- **CN**: 它引入了 LintMessage 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `LintMessage`，进一步展开其内部控制流或数据流转。

### Lines 57-62
```python

def glob_yamls(path: Path) -> Iterable[Path]:
    return itertools.chain(path.glob("**/*.yml"), path.glob("**/*.yaml"))


def load_yaml(path: Path) -> Any:
```
- **EN**: This chunk defines `load_yaml`, which parses or loads structured input into tool-friendly data structures. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `load_yaml`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 63-69
```python
    with open(path) as f:
        return load(f, Loader)


def is_workflow(yaml: Any) -> bool:
    return yaml.get("jobs") is not None
```
- **EN**: This chunk defines `is_workflow`, which implements a focused step inside the lint tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `is_workflow`，其作用是实现Lint 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 70-81
```python

def print_lint_message(
    path: Path,
    job: dict[str, Any],
    sync_tag: str,
    baseline_path: Path,
    baseline_job_id: str,
) -> None:
    job_id = next(iter(job.keys()))
    with open(path) as f:
        lines = f.readlines()
    for i, line in enumerate(lines):
```
- **EN**: This chunk defines `print_lint_message`, which validates invariants and reports policy violations early. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `print_lint_message`，其作用是校验不变量，并尽早报告策略违规。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 82-93
```python
        if f"{job_id}:" in line:
            line_number = i + 1

    lint_message = LintMessage(
        path=str(path),
        # pyrefly: ignore [unbound-name]
        line=line_number,
        char=None,
        code="WORKFLOWSYNC",
        severity=LintSeverity.ERROR,
        name="workflow-inconsistency",
        original=None,
```
- **EN**: This chunk continues `print_lint_message` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `print_lint_message`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 94-99
```python
        replacement=None,
        description=f"Job doesn't match other job {baseline_job_id} in file {baseline_path} with sync-tag: '{sync_tag}'",
    )
    print(json.dumps(lint_message._asdict()), flush=True)
```
- **EN**: This chunk continues `print_lint_message` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `print_lint_message`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 100-106
```python
def get_jobs_with_sync_tag(
    job: dict[str, Any],
) -> tuple[str, str, dict[str, Any]] | None:
    sync_tag = job.get("with", {}).get("sync-tag")
    if sync_tag is None:
        return None
```
- **EN**: This chunk defines `get_jobs_with_sync_tag`, which implements a focused step inside the lint tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_jobs_with_sync_tag`，其作用是实现Lint 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 107-112
```python
    # remove the "if" field, which we allow to be different between jobs
    # (since you might have different triggering conditions on pull vs.
    # trunk, say.)
    if "if" in job:
        del job["if"]
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 113-120
```python
    # same is true for ['with']['test-matrix']
    if "test-matrix" in job.get("with", {}):
        del job["with"]["test-matrix"]
    # and ['with']['tests-to-include'], since dispatch filters differ
    if "tests-to-include" in job.get("with", {}):
        del job["with"]["tests-to-include"]
    # and ['with']['build-environment'], since GPU-specific suffixes differ for ROCm
    if (
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 121-126
```python
        "build-environment" in job.get("with", {})
        and "rocm" in job["with"]["build-environment"]
    ):
        del job["with"]["build-environment"]
    # and ['name'], since ROCm jobs append a GPU-specific suffix to the job name
    if "name" in job and "rocm" in job.get("name", ""):
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 127-134
```python
        del job["name"]

    # normalize needs: remove helper job-filter so comparisons ignore it
    needs = job.get("needs")
    if needs:
        needs_list = [needs] if isinstance(needs, str) else list(needs)
        needs_list = [n for n in needs_list if n != "job-filter"]
        if not needs_list:
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 135-140
```python
            job.pop("needs", None)
        elif len(needs_list) == 1:
            job["needs"] = needs_list[0]
        else:
            job["needs"] = needs_list
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 141-152
```python
    return (sync_tag, job_id, job)


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="workflow consistency linter.",
        fromfile_prefix_chars="@",
    )
    parser.add_argument(
        "filenames",
        nargs="+",
        help="paths to lint",
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 153-158
```python
    )
    args = parser.parse_args()

    # Go through all files, aggregating jobs with the same sync tag
    tag_to_jobs = defaultdict(list)
    for path in REPO_ROOT.glob(".github/workflows/*"):
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 159-166
```python
        if not path.is_file() or path.suffix not in {".yml", ".yaml"}:
            continue
        workflow = load_yaml(path)
        if not is_workflow(workflow):
            continue
        clean_path = path.relative_to(REPO_ROOT)
        jobs = workflow.get("jobs", {})
        for job_id, job in jobs.items():
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 167-172
```python
            res = get_jobs_with_sync_tag(job)
            if res is None:
                continue
            sync_tag, job_id, job_dict = res
            tag_to_jobs[sync_tag].append((clean_path, job_id, job_dict))
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 173-179
```python
    # Check the files passed as arguments
    for path in args.filenames:
        workflow = load_yaml(Path(path))
        jobs = workflow["jobs"]
        for job_id, job in jobs.items():
            res = get_jobs_with_sync_tag(job)
            if res is None:
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 180-185
```python
                continue
            sync_tag, job_id, job_dict = res
            job_str = dump(job_dict)

            # For each sync tag, check that all the jobs have the same code.
            for baseline_path, baseline_job_id, baseline_dict in tag_to_jobs[sync_tag]:
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 186-191
```python
                baseline_str = dump(baseline_dict)

                if job_id != baseline_job_id or job_str != baseline_str:
                    print_lint_message(
                        path, job_dict, sync_tag, baseline_path, baseline_job_id
                    )
```
- **EN**: This chunk continues `get_jobs_with_sync_tag` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_jobs_with_sync_tag`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **Lint tooling**
  - EN: This file belongs to the lint tooling layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于Lint 工具链层，应结合同一子目录中的相邻脚本一起理解。
- **YAML-driven metadata**
  - EN: The logic reads declarative YAML metadata and converts it into executable build or codegen decisions.
  - CN: 该逻辑读取声明式 YAML 元数据，并把它转化为可执行的构建或代码生成决策。
- **Lint enforcement**
  - EN: The code applies policy checks to source files and reports actionable diagnostics.
  - CN: 代码对源码施加策略检查，并产出可执行的诊断信息。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `argparse`, `itertools`, `json`, `collections`, `enum`, `pathlib`, `typing`, `collections.abc`
- **External packages / 外部依赖包**: `yaml`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `LintSeverity`, `LintMessage`, `glob_yamls`, `load_yaml`, `is_workflow`, `print_lint_message`, `get_jobs_with_sync_tag`
