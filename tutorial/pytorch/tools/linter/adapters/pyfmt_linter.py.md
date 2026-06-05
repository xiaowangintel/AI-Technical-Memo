# pyfmt_linter.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/pyfmt_linter.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```python
# /// script
# requires-python = ">=3.10"
# dependencies = [
#   "usort==1.1.3",
#   "isort==6.0.1",
#   "ruff==0.14.4",
# ]
# ///
from __future__ import annotations
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__。

### Lines 11-22
```python
import argparse
import concurrent.futures
import json
import logging
import os
import re
import subprocess
import sys
from enum import Enum
from pathlib import Path
from typing import NamedTuple
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, concurrent.futures, json, and 8 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、concurrent.futures、json 等共 11 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 23-29
```python
import isort
import usort


IS_WINDOWS: bool = os.name == "nt"
REPO_ROOT = Path(__file__).absolute().parents[3]
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as isort, usort. Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 isort、usort。 REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 30-36
```python

class LintSeverity(str, Enum):
    ERROR = "error"
    WARNING = "warning"
    ADVICE = "advice"
    DISABLED = "disabled"
```
- **EN**: It introduces classes such as LintSeverity, which package state and behavior for this tooling task. This chunk continues `LintSeverity` and expands its internal control flow or data movement. Configuration constants such as ERROR, WARNING, ADVICE, and 1 more centralize defaults so later functions share the same policy knobs.
- **CN**: 它引入了 LintSeverity 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `LintSeverity`，进一步展开其内部控制流或数据流转。 ERROR、WARNING、ADVICE 等共 4 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 37-48
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

### Lines 49-54
```python

def as_posix(name: str) -> str:
    return name.replace("\\", "/") if IS_WINDOWS else name


def format_error_message(filename: str, err: Exception) -> LintMessage:
```
- **EN**: This chunk defines `format_error_message`, which implements a focused step inside the lint tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `format_error_message`，其作用是实现Lint 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 55-66
```python
    return LintMessage(
        path=filename,
        line=None,
        char=None,
        code="PYFMT",
        severity=LintSeverity.ADVICE,
        name="command-failed",
        original=None,
        replacement=None,
        description=(f"Failed due to {err.__class__.__name__}:\n{err}"),
    )
```
- **EN**: This chunk continues `format_error_message` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `format_error_message`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 67-72
```python

def run_isort(content: str, path: Path) -> str:
    isort_config = isort.Config(settings_path=str(REPO_ROOT))

    is_this_file = path.samefile(__file__)
    if not is_this_file:
```
- **EN**: This chunk defines `run_isort`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `run_isort`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 73-79
```python
        content = re.sub(r"(#.*\b)usort:\s*skip\b", r"\g<1>isort: split", content)

    content = isort.code(content, config=isort_config, file_path=path)

    if not is_this_file:
        content = re.sub(r"(#.*\b)isort: split\b", r"\g<1>usort: skip", content)
```
- **EN**: This chunk continues `run_isort` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `run_isort`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 80-85
```python
    return content


def run_usort(content: str, path: Path) -> str:
    usort_config = usort.Config.find(path)
```
- **EN**: This chunk defines `run_usort`, which orchestrates command-line execution and forwards parsed arguments into the core logic. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `run_usort`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 86-97
```python
    return usort.usort_string(content, path=path, config=usort_config)


def run_ruff_format(content: str, path: Path) -> str:
    try:
        return subprocess.check_output(
            [
                sys.executable,
                "-m",
                "ruff",
                "format",
                "--config",
```
- **EN**: This chunk defines `run_ruff_format`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `run_ruff_format`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 98-109
```python
                str(REPO_ROOT / "pyproject.toml"),
                "--stdin-filename",
                str(path),
                "-",
            ],
            input=content,
            stderr=subprocess.STDOUT,
            text=True,
            encoding="utf-8",
        )
    except subprocess.CalledProcessError as exc:
        raise ValueError(exc.output) from exc
```
- **EN**: This chunk continues `run_ruff_format` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `run_ruff_format`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 110-115
```python


def check_file(filename: str) -> list[LintMessage]:
    path = Path(filename).absolute()
    original = replacement = path.read_text(encoding="utf-8")
```
- **EN**: This chunk defines `check_file`, which validates invariants and reports policy violations early. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段定义了 `check_file`，其作用是校验不变量，并尽早报告策略违规。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 116-121
```python
    try:
        # NB: run isort first to enforce style for blank lines
        replacement = run_isort(replacement, path=path)
        replacement = run_usort(replacement, path=path)
        replacement = run_ruff_format(replacement, path=path)
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 122-133
```python
        if original == replacement:
            return []

        return [
            LintMessage(
                path=filename,
                line=None,
                char=None,
                code="PYFMT",
                severity=LintSeverity.WARNING,
                name="format",
                original=original,
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 134-140
```python
                replacement=replacement,
                description="Run `lintrunner -a` to apply this patch.",
            )
        ]
    except Exception as err:
        return [format_error_message(filename, err)]
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 141-152
```python

def main() -> None:
    parser = argparse.ArgumentParser(
        description="Format files with usort + ruff-format.",
        fromfile_prefix_chars="@",
    )
    parser.add_argument(
        "--verbose",
        action="store_true",
        help="verbose logging",
    )
    parser.add_argument(
```
- **EN**: This chunk defines `main`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段定义了 `main`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 153-158
```python
        "filenames",
        nargs="+",
        help="paths to lint",
    )
    args = parser.parse_args()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。

### Lines 159-164
```python
    logging.basicConfig(
        format="<%(processName)s:%(levelname)s> %(message)s",
        level=logging.NOTSET
        if args.verbose
        else logging.DEBUG
        if len(args.filenames) < 1000
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 165-173
```python
        else logging.INFO,
        stream=sys.stderr,
    )

    with concurrent.futures.ProcessPoolExecutor(
        max_workers=os.cpu_count(),
    ) as executor:
        futures = {executor.submit(check_file, x): x for x in args.filenames}
        for future in concurrent.futures.as_completed(futures):
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 174-180
```python
            try:
                for lint_message in future.result():
                    print(json.dumps(lint_message._asdict()), flush=True)
            except Exception:
                logging.critical('Failed at "%s".', futures[future])
                raise
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 181-183
```python

if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

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
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `argparse`, `concurrent.futures`, `json`, `logging`, `os`, `re`, `subprocess`, `sys`, `enum`, `pathlib`, `typing`
- **External packages / 外部依赖包**: `isort`, `usort`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `LintSeverity`, `LintMessage`, `as_posix`, `format_error_message`, `run_isort`, `run_usort`, `run_ruff_format`, `check_file`, `main`
