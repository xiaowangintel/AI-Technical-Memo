# pyrefly_linter.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/pyrefly_linter.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
# /// script
# requires-python = ">=3.10"
# dependencies = [
#   "numpy==1.26.4 ; python_version >= '3.10' and python_version <= '3.11'",
#   "numpy==2.1.0 ; python_version >= '3.12' and python_version <= '3.13'",
#   "numpy==2.3.4 ; python_version >= '3.14'",
#   "expecttest==0.3.0",
#   "pyrefly==0.58.0",
#   "sympy==1.13.3",
#   "types-requests==2.27.25",
#   "types-pyyaml==6.0.2",
#   "types-tabulate==0.8.8",
#   "types-protobuf==5.29.1.20250403",
#   "types-setuptools==79.0.0.20250422",
#   "types-jinja2==2.11.9",
#   "types-colorama==0.4.6",
#   "filelock==3.18.0",
#   "junitparser==2.1.1",
```
- **EN**: This comment block records design intent, generation notes, or local caveats that frame the code below. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段注释记录了设计意图、生成说明或局部注意事项，为下方代码提供背景。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 19-30
```python
#   "rich==14.1.0",
#   "optree==0.17.0",
#   "types-openpyxl==3.1.5.20250919",
#   "types-python-dateutil==2.9.0.20251008",
#   "packaging",
#   "libcst",
#   "isort",
#   "usort",
# ]
# ///
from __future__ import annotations
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__。

### Lines 31-41
```python
import argparse
import json
import logging
import os
import re
import subprocess
import sys
import time
from enum import Enum
from typing import NamedTuple
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, json, logging, and 7 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、json、logging 等共 10 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 42-50
```python

class LintSeverity(str, Enum):
    ERROR = "error"
    WARNING = "warning"
    ADVICE = "advice"
    DISABLED = "disabled"


class LintMessage(NamedTuple):
```
- **EN**: It introduces classes such as LintSeverity, LintMessage, which package state and behavior for this tooling task. This chunk continues `LintMessage` and expands its internal control flow or data movement. Configuration constants such as ERROR, WARNING, ADVICE, and 1 more centralize defaults so later functions share the same policy knobs.
- **CN**: 它引入了 LintSeverity、LintMessage 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `LintMessage`，进一步展开其内部控制流或数据流转。 ERROR、WARNING、ADVICE 等共 4 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 51-60
```python
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
- **EN**: This chunk continues `LintMessage` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `LintMessage`，进一步展开其内部控制流或数据流转。

### Lines 61-75
```python

# Note: This regex pattern is kept for reference but not used for pyrefly JSON parsing
RESULTS_RE: re.Pattern[str] = re.compile(
    r"""(?mx)
    ^
    (?P<file>.*?):
    (?P<line>\d+):
    (?:(?P<column>-?\d+):)?
    \s(?P<severity>\S+?):?
    \s(?P<message>.*)
    \s(?P<code>\[.*\])
    $
    """
)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `LintMessage` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `LintMessage`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 76-87
```python
# torch/_dynamo/variables/tensor.py:363: error: INTERNAL ERROR
INTERNAL_ERROR_RE: re.Pattern[str] = re.compile(
    r"""(?mx)
    ^
    (?P<file>.*?):
    (?P<line>\d+):
    \s(?P<severity>\S+?):?
    \s(?P<message>INTERNAL\sERROR.*)
    $
    """
)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `LintMessage` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `LintMessage`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。

### Lines 88-97
```python

def run_command(
    args: list[str],
    *,
    extra_env: dict[str, str] | None,
    retries: int,
) -> subprocess.CompletedProcess[bytes]:
    logging.debug("$ %s", " ".join(args))
    start_time = time.monotonic()
    try:
```
- **EN**: This chunk defines `run_command`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `run_command`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 98-106
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

### Lines 107-115
```python
# Severity mapping (currently only used for stderr internal errors)
# Pyrefly JSON output doesn't include severity, so all errors default to ERROR
severities = {
    "error": LintSeverity.ERROR,
    "note": LintSeverity.ADVICE,
}


def check_pyrefly_installed(code: str) -> list[LintMessage]:
```
- **EN**: This chunk defines `check_pyrefly_installed`, which validates invariants and reports policy violations early. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `check_pyrefly_installed`，其作用是校验不变量，并尽早报告策略违规。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 116-133
```python
    cmd = ["pyrefly", "--version"]
    try:
        subprocess.run(cmd, check=True, capture_output=True)
        return []
    except subprocess.CalledProcessError as e:
        msg = e.stderr.decode(errors="replace")
        return [
            LintMessage(
                path=None,
                line=None,
                char=None,
                code=code,
                severity=LintSeverity.ERROR,
                name="command-failed",
                original=None,
                replacement=None,
                description=f"Could not run '{' '.join(cmd)}': {msg}",
            )
```
- **EN**: This chunk continues `check_pyrefly_installed` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_pyrefly_installed`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 134-144
```python
        ]


def in_github_actions() -> bool:
    return bool(os.getenv("GITHUB_ACTIONS"))


def check_files(
    code: str, config: str, remove_unused_ignores: bool, suppress: bool
) -> list[LintMessage]:
    try:
```
- **EN**: This chunk defines `check_files`, which validates invariants and reports policy violations early. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `check_files`，其作用是校验不变量，并尽早报告策略违规。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 145-154
```python
        pyrefly_commands = [
            "pyrefly",
            "check",
            "--config",
            config,
            "--output-format=json",
        ]
        if remove_unused_ignores:
            pyrefly_commands.append("--remove-unused-ignores")
        if suppress:
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 155-172
```python
            pyrefly_commands.append("--suppress-errors")
        proc = run_command(
            [*pyrefly_commands],
            extra_env={},
            retries=0,
        )
    except OSError as err:
        return [
            LintMessage(
                path=None,
                line=None,
                char=None,
                code=code,
                severity=LintSeverity.ERROR,
                name="command-failed",
                original=None,
                replacement=None,
                description=(f"Failed due to {err.__class__.__name__}:\n{err}"),
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 173-190
```python
            )
        ]
    stdout = str(proc.stdout, "utf-8").strip()
    stderr = str(proc.stderr, "utf-8").strip()
    if proc.returncode not in (0, 1):
        return [
            LintMessage(
                path=None,
                line=None,
                char=None,
                code=code,
                severity=LintSeverity.ERROR,
                name="command-failed",
                original=None,
                replacement=None,
                description=stderr,
            )
        ]
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 191-208
```python

    # Parse JSON output from pyrefly. In GitHub Actions, pyrefly appends
    # ::error commands to stdout after the JSON, so use raw_decode to parse
    # only the first JSON object and ignore trailing output.
    try:
        if stdout:
            result, _ = json.JSONDecoder().raw_decode(stdout)
            errors = result.get("errors", [])
        else:
            errors = []
        errors = [error for error in errors if error["name"] != "deprecated"]
        rc = [
            LintMessage(
                path=error["path"],
                name=error["name"],
                description=error.get(
                    "description", error.get("concise_description", "")
                ),
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 209-218
```python
                line=error["line"],
                char=error["column"],
                code=code,
                severity=LintSeverity.ADVICE
                if error["name"] == "deprecated"
                else LintSeverity.ERROR,
                original=None,
                replacement=None,
            )
            for error in errors
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 219-234
```python
        ]
    except (json.JSONDecodeError, KeyError, TypeError) as e:
        return [
            LintMessage(
                path=None,
                line=None,
                char=None,
                code=code,
                severity=LintSeverity.ERROR,
                name="json-parse-error",
                original=None,
                replacement=None,
                description=f"Failed to parse pyrefly JSON output: {e}",
            )
        ]
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 235-248
```python
    # Still check stderr for internal errors
    rc += [
        LintMessage(
            path=match["file"],
            name="INTERNAL ERROR",
            description=match["message"],
            line=int(match["line"]),
            char=None,
            code=code,
            severity=severities.get(match["severity"], LintSeverity.ERROR),
            original=None,
            replacement=None,
        )
        for match in INTERNAL_ERROR_RE.finditer(stderr)
```
- **EN**: This chunk continues `check_files` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `check_files`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 249-266
```python
    ]
    return rc


def main() -> None:
    parser = argparse.ArgumentParser(
        description="pyrefly wrapper linter.",
        fromfile_prefix_chars="@",
    )
    parser.add_argument(
        "--code",
        default="PYREFLY",
        help="the code this lint should report as",
    )
    parser.add_argument(
        "--verbose",
        action="store_true",
        help="verbose logging",
```
- **EN**: This chunk defines `main`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `main`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 267-284
```python
    )
    parser.add_argument(
        "--config",
        required=True,
        help="path to an mypy .ini config file",
    )
    parser.add_argument(
        "--remove-unused-ignores",
        action="store_true",
        help="clean up unused ignores",
    )
    parser.add_argument(
        "--suppress",
        action="store_true",
        help="add suppressions",
    )
    args = parser.parse_args()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 285-294
```python
    logging.basicConfig(
        format="<%(threadName)s:%(levelname)s> %(message)s",
        level=logging.INFO,
        stream=sys.stderr,
    )

    lint_messages = check_pyrefly_installed(args.code) + check_files(
        args.code, args.config, args.remove_unused_ignores, args.suppress
    )
    for lint_message in lint_messages:
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 295-299
```python
        print(json.dumps(lint_message._asdict()), flush=True)


if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

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
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `argparse`, `json`, `logging`, `os`, `re`, `subprocess`, `sys`, `time`, `enum`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `LintSeverity`, `LintMessage`, `run_command`, `check_pyrefly_installed`, `in_github_actions`, `check_files`, `main`
