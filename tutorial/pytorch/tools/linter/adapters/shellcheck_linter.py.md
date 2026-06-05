# shellcheck_linter.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/shellcheck_linter.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```python
# /// script
# requires-python = ">=3.10"
# dependencies = [
#   "shellcheck-py==0.7.2.1; platform_machine == 'x86_64'",
# ]
# ///
from __future__ import annotations
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__。

### Lines 9-19
```python
import argparse
import json
import logging
import os
import platform
import subprocess
import sys
import time
from enum import Enum
from typing import NamedTuple
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, json, logging, and 7 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、json、logging 等共 10 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 20-29
```python

LINTER_CODE = "SHELLCHECK"


class LintSeverity(str, Enum):
    ERROR = "error"
    WARNING = "warning"
    ADVICE = "advice"
    DISABLED = "disabled"
```
- **EN**: It introduces classes such as LintSeverity, which package state and behavior for this tooling task. This chunk continues `LintSeverity` and expands its internal control flow or data movement. Configuration constants such as LINTER_CODE, ERROR, WARNING, and 2 more centralize defaults so later functions share the same policy knobs.
- **CN**: 它引入了 LintSeverity 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `LintSeverity`，进一步展开其内部控制流或数据流转。 LINTER_CODE、ERROR、WARNING 等共 5 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 30-41
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

### Lines 42-48
```python

def run_command(
    args: list[str],
) -> subprocess.CompletedProcess[bytes]:
    logging.debug("$ %s", " ".join(args))
    start_time = time.monotonic()
    try:
```
- **EN**: This chunk defines `run_command`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `run_command`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 49-56
```python
        return subprocess.run(
            args,
            capture_output=True,
        )
    finally:
        end_time = time.monotonic()
        logging.debug("took %dms", (end_time - start_time) * 1000)
```
- **EN**: This chunk continues `run_command` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `run_command`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 57-62
```python

def _is_x86_64() -> bool:
    return platform.machine() == "x86_64"


def _shellcheck_candidates() -> list[str]:
```
- **EN**: This chunk defines `_shellcheck_candidates`, which validates invariants and reports policy violations early. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_shellcheck_candidates`，其作用是校验不变量，并尽早报告策略违规。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 63-69
```python
    path_env = os.environ.get("PATH", "")
    candidates: list[str] = []
    for directory in path_env.split(os.pathsep):
        if not directory:
            continue
        candidate = os.path.join(directory, "shellcheck")
        if os.path.isfile(candidate) and os.access(candidate, os.X_OK):
```
- **EN**: This chunk continues `_shellcheck_candidates` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_shellcheck_candidates`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 70-78
```python
            candidates.append(candidate)
    return candidates


def check_files(
    files: list[str],
) -> list[LintMessage]:
    args = ["--external-sources", "--format=json1"] + files
```
- **EN**: This chunk defines `check_files`, which validates invariants and reports policy violations early. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `check_files`，其作用是校验不变量，并尽早报告策略违规。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 79-88
```python
    proc: subprocess.CompletedProcess[bytes] | None = None
    last_error: OSError | None = None

    for shellcheck in _shellcheck_candidates():
        try:
            proc = run_command([shellcheck] + args)
            break
        except OSError as err:
            last_error = err
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 89-100
```python
    if proc is None:
        if last_error is not None and last_error.errno == 8:
            return []
        if not _is_x86_64():
            return []
        return [
            LintMessage(
                path=None,
                line=None,
                char=None,
                code=LINTER_CODE,
                severity=LintSeverity.ERROR,
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 101-106
```python
                name="command-failed",
                original=None,
                replacement=None,
                description=(
                    f"Failed to execute shellcheck.\n{last_error.__class__.__name__}: {last_error}"
                    if last_error is not None
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 107-118
```python
                    else "Failed to find a usable shellcheck executable."
                ),
            )
        ]
    stdout = str(proc.stdout, "utf-8").strip()
    results = json.loads(stdout)["comments"]
    return [
        LintMessage(
            path=result["file"],
            name=f"SC{result['code']}",
            description=result["message"],
            line=result["line"],
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 119-125
```python
            char=result["column"],
            code=LINTER_CODE,
            severity=LintSeverity.ERROR,
            original=None,
            replacement=None,
        )
        for result in results
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 126-137
```python
    ]


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="shellcheck runner",
        fromfile_prefix_chars="@",
    )
    parser.add_argument(
        "filenames",
        nargs="+",
        help="paths to lint",
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 138-149
```python
    )

    if not _shellcheck_candidates():
        if not _is_x86_64():
            sys.exit(0)
        err_msg = LintMessage(
            path="<none>",
            line=None,
            char=None,
            code=LINTER_CODE,
            severity=LintSeverity.ERROR,
            name="command-failed",
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 150-156
```python
            original=None,
            replacement=None,
            description="shellcheck is not installed, did you forget to run `lintrunner init`?",
        )
        print(json.dumps(err_msg._asdict()), flush=True)
        sys.exit(0)
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 157-161
```python
    args = parser.parse_args()

    lint_messages = check_files(args.filenames)
    for lint_message in lint_messages:
        print(json.dumps(lint_message._asdict()), flush=True)
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

## Key Concepts / 关键概念

- **Lint tooling**
  - EN: This file belongs to the lint tooling layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于Lint 工具链层，应结合同一子目录中的相邻脚本一起理解。
- **Lint enforcement**
  - EN: The code applies policy checks to source files and reports actionable diagnostics.
  - CN: 代码对源码施加策略检查，并产出可执行的诊断信息。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **LINTER_CODE**
  - EN: `LINTER_CODE` is one of the main local symbols exposed or implemented here.
  - CN: `LINTER_CODE` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `argparse`, `json`, `logging`, `os`, `platform`, `subprocess`, `sys`, `time`, `enum`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `LINTER_CODE`, `LintSeverity`, `LintMessage`, `run_command`, `_is_x86_64`, `_shellcheck_candidates`, `check_files`
