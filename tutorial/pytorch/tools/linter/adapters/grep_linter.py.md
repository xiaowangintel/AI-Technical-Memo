# grep_linter.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/grep_linter.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
"""
Generic linter that greps for a pattern and optionally suggests replacements.
"""

from __future__ import annotations

import argparse
import json
import logging
import os
import subprocess
import sys
import time
from enum import Enum
from typing import NamedTuple
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, argparse, json, and 7 more. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、argparse、json 等共 10 项。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 17-25
```python

IS_WINDOWS: bool = os.name == "nt"
MAX_FILE_SIZE: int = 1024 * 1024 * 1024  # 1GB in bytes
MAX_MATCHES_PER_FILE: int = 100  # Maximum number of matches to report per file
MAX_ORIGINAL_SIZE: int = (
    512 * 1024
)  # 512KB - don't compute replacement if original is larger
```
- **EN**: Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 26-35
```python
class LintSeverity(str, Enum):
    ERROR = "error"
    WARNING = "warning"
    ADVICE = "advice"
    DISABLED = "disabled"


LINTER_NAME: str = ""
ERROR_DESCRIPTION: str | None = None
```
- **EN**: It introduces classes such as LintSeverity, which package state and behavior for this tooling task. This chunk continues `LintSeverity` and expands its internal control flow or data movement. Configuration constants such as ERROR, WARNING, ADVICE, and 1 more centralize defaults so later functions share the same policy knobs.
- **CN**: 它引入了 LintSeverity 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `LintSeverity`，进一步展开其内部控制流或数据流转。 ERROR、WARNING、ADVICE 等共 4 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 36-47
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

### Lines 48-58
```python

def as_posix(name: str) -> str:
    return name.replace("\\", "/") if IS_WINDOWS else name


def run_command(
    args: list[str],
) -> subprocess.CompletedProcess[bytes]:
    logging.debug("$ %s", " ".join(args))
    start_time = time.monotonic()
    try:
```
- **EN**: This chunk defines `run_command`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `run_command`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 59-67
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

### Lines 68-79
```python
def print_lint_message(
    name: str,
    severity: LintSeverity = LintSeverity.ERROR,
    path: str | None = None,
    line: int | None = None,
    original: str | None = None,
    replacement: str | None = None,
    description: str | None = None,
) -> None:
    """
    Create a LintMessage and print it as JSON.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `print_lint_message`, which validates invariants and reports policy violations early. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `print_lint_message`，其作用是校验不变量，并尽早报告策略违规。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 80-89
```python
    Accepts the same arguments as LintMessage constructor.
    """
    char = None
    code = LINTER_NAME
    description = description or ERROR_DESCRIPTION
    lint_message = LintMessage(
        path, line, char, code, severity, name, original, replacement, description
    )
    print(json.dumps(lint_message._asdict()), flush=True)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `print_lint_message` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `print_lint_message`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 90-102
```python

def group_lines_by_file(lines: list[str]) -> dict[str, list[str]]:
    """
    Group matching lines by filename.

    Args:
        lines: List of grep output lines in format "filename:line:content"

    Returns:
        Dictionary mapping filename to list of line remainders (without filename prefix)
    """
    grouped: dict[str, list[str]] = {}
    for line in lines:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `group_lines_by_file`, which implements a focused step inside the lint tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `group_lines_by_file`，其作用是实现Lint 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 103-113
```python
        if not line:
            continue
        # Extract filename and remainder from "filename:line:content" format
        parts = line.split(":", 1)
        filename = parts[0]
        remainder = parts[1] if len(parts) > 1 else ""
        if filename not in grouped:
            grouped[filename] = []
        grouped[filename].append(remainder)
    return grouped
```
- **EN**: This chunk continues `group_lines_by_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `group_lines_by_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 114-125
```python

def check_allowlist(
    filename: str,
    allowlist_pattern: str,
) -> bool:
    """
    Check if a file matches the allowlist pattern.

    Args:
        filename: Path to the file to check
        allowlist_pattern: Pattern to grep for in the file
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `check_allowlist`, which validates invariants and reports policy violations early. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `check_allowlist`，其作用是校验不变量，并尽早报告策略违规。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 126-140
```python
    Returns:
        True if the file should be skipped (allowlist pattern matched), False otherwise.
        Prints error message and returns False if there was an error running grep.
    """
    if not allowlist_pattern:
        return False

    try:
        proc = run_command(["grep", "-nEHI", allowlist_pattern, filename])
    except Exception as err:
        print_lint_message(
            name="command-failed",
            description=(
                f"Failed due to {err.__class__.__name__}:\n{err}"
                if not isinstance(err, subprocess.CalledProcessError)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `check_allowlist` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `check_allowlist`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 141-155
```python
                else (
                    "COMMAND (exit code {returncode})\n"
                    "{command}\n\n"
                    "STDERR\n{stderr}\n\n"
                    "STDOUT\n{stdout}"
                ).format(
                    returncode=err.returncode,
                    command=" ".join(as_posix(x) for x in err.cmd),
                    stderr=err.stderr.decode("utf-8").strip() or "(empty)",
                    stdout=err.stdout.decode("utf-8").strip() or "(empty)",
                )
            ),
        )
        return False
```
- **EN**: This chunk continues `check_allowlist` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_allowlist`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 156-172
```python
    # allowlist pattern was found, abort lint
    if proc.returncode == 0:
        return True

    return False


def lint_file(
    filename: str,
    line_remainders: list[str],
    allowlist_pattern: str,
    replace_pattern: str,
    error_name: str,
) -> None:
    """
    Lint a file with one or more pattern matches, printing LintMessages as they're created.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `lint_file`, which validates invariants and reports policy violations early. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `lint_file`，其作用是校验不变量，并尽早报告策略违规。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 173-182
```python
    Args:
        filename: Path to the file being linted
        line_remainders: List of line remainders (format: "line:content" without filename prefix)
        allowlist_pattern: Pattern to check for allowlisting
        replace_pattern: Pattern for sed replacement
        error_name: Human-readable error name
    """
    if not line_remainders:
        return
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `lint_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `lint_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 183-194
```python
    should_skip = check_allowlist(filename, allowlist_pattern)
    if should_skip:
        return

    # Check if file is too large to compute replacement
    file_size = os.path.getsize(filename)
    compute_replacement = replace_pattern and file_size <= MAX_ORIGINAL_SIZE

    # Apply replacement to entire file if pattern is specified and file is not too large
    original = None
    replacement = None
    if compute_replacement:
```
- **EN**: This chunk continues `lint_file` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `lint_file`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 195-207
```python
        # When we have a replacement, report a single message with line=None
        try:
            with open(filename) as f:
                original = f.read()

            proc = run_command(["sed", "-r", replace_pattern, filename])
            replacement = proc.stdout.decode("utf-8")
        except Exception as err:
            print_lint_message(
                name="command-failed",
                description=(
                    f"Failed due to {err.__class__.__name__}:\n{err}"
                    if not isinstance(err, subprocess.CalledProcessError)
```
- **EN**: This chunk continues `lint_file` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `lint_file`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 208-222
```python
                    else (
                        "COMMAND (exit code {returncode})\n"
                        "{command}\n\n"
                        "STDERR\n{stderr}\n\n"
                        "STDOUT\n{stdout}"
                    ).format(
                        returncode=err.returncode,
                        command=" ".join(as_posix(x) for x in err.cmd),
                        stderr=err.stderr.decode("utf-8").strip() or "(empty)",
                        stdout=err.stdout.decode("utf-8").strip() or "(empty)",
                    )
                ),
            )
            return
```
- **EN**: This chunk continues `lint_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `lint_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 223-233
```python
        print_lint_message(
            path=filename,
            name=error_name,
            original=original,
            replacement=replacement,
        )
    else:
        # When no replacement, report each matching line (up to MAX_MATCHES_PER_FILE)
        total_matches = len(line_remainders)
        matches_to_report = min(total_matches, MAX_MATCHES_PER_FILE)
```
- **EN**: This chunk continues `lint_file` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `lint_file`，进一步展开其内部控制流或数据流转。

### Lines 234-243
```python
        for line_remainder in line_remainders[:matches_to_report]:
            # line_remainder format: "line_number:content"
            split = line_remainder.split(":", 1)
            line_number = int(split[0]) if split[0] else None
            print_lint_message(
                path=filename,
                line=line_number,
                name=error_name,
            )
```
- **EN**: This chunk continues `lint_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `lint_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 244-252
```python
        # If there are more matches than the limit, print an error
        if total_matches > MAX_MATCHES_PER_FILE:
            print_lint_message(
                path=filename,
                name="too-many-matches",
                description=f"File has {total_matches} matches, only showing first {MAX_MATCHES_PER_FILE}",
            )
```
- **EN**: This chunk continues `lint_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `lint_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 253-270
```python
def main() -> None:
    parser = argparse.ArgumentParser(
        description="grep wrapper linter.",
        fromfile_prefix_chars="@",
    )
    parser.add_argument(
        "--pattern",
        required=True,
        help="pattern to grep for",
    )
    parser.add_argument(
        "--allowlist-pattern",
        help="if this pattern is true in the file, we don't grep for pattern",
    )
    parser.add_argument(
        "--linter-name",
        required=True,
        help="name of the linter",
```
- **EN**: This chunk defines `main`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `main`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 271-288
```python
    )
    parser.add_argument(
        "--match-first-only",
        action="store_true",
        help="only match the first hit in the file",
    )
    parser.add_argument(
        "--error-name",
        required=True,
        help="human-readable description of what the error is",
    )
    parser.add_argument(
        "--error-description",
        required=True,
        help="message to display when the pattern is found",
    )
    parser.add_argument(
        "--replace-pattern",
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 289-304
```python
        help=(
            "the form of a pattern passed to `sed -r`. "
            "If specified, this will become proposed replacement text."
        ),
    )
    parser.add_argument(
        "--verbose",
        action="store_true",
        help="verbose logging",
    )
    parser.add_argument(
        "filenames",
        nargs="+",
        help="paths to lint",
    )
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 305-315
```python
    # Check for duplicate arguments before parsing
    seen_args = set()
    for arg in sys.argv[1:]:
        if arg.startswith("--"):
            arg_name = arg.split("=")[0]
            if arg_name in seen_args:
                parser.error(
                    f"argument {arg_name}: not allowed to be specified multiple times"
                )
            seen_args.add(arg_name)
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 316-325
```python
    args = parser.parse_args()

    global LINTER_NAME, ERROR_DESCRIPTION
    LINTER_NAME = args.linter_name
    ERROR_DESCRIPTION = args.error_description

    logging.basicConfig(
        format="<%(threadName)s:%(levelname)s> %(message)s",
        level=logging.NOTSET
        if args.verbose
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Configuration constants such as LINTER_NAME, ERROR_DESCRIPTION centralize defaults so later functions share the same policy knobs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 LINTER_NAME、ERROR_DESCRIPTION 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 326-334
```python
        else logging.DEBUG
        if len(args.filenames) < 1000
        else logging.INFO,
        stream=sys.stderr,
    )

    # Filter out files that are too large before running grep
    filtered_filenames = []
    for filename in args.filenames:
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 335-352
```python
        try:
            file_size = os.path.getsize(filename)
            if file_size > MAX_FILE_SIZE:
                print_lint_message(
                    path=filename,
                    severity=LintSeverity.WARNING,
                    name="file-too-large",
                    description=f"File size ({file_size} bytes) exceeds {MAX_FILE_SIZE} bytes limit, skipping",
                )
            else:
                filtered_filenames.append(filename)
        except OSError as err:
            print_lint_message(
                path=filename,
                name="file-access-error",
                description=f"Failed to get file size: {err}",
            )
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 353-362
```python
    # If all files were filtered out, nothing to do
    if not filtered_filenames:
        return

    files_with_matches = []
    if args.match_first_only:
        files_with_matches = ["--files-with-matches"]

    lines = []
    try:
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 363-380
```python
        # Split the grep command into multiple batches to avoid hitting the
        # command line length limit of ~1M on my machine
        arg_length = sum(len(x) for x in filtered_filenames)
        batches = arg_length // 750000 + 1
        batch_size = len(filtered_filenames) // batches
        for i in range(0, len(filtered_filenames), batch_size):
            proc = run_command(
                [
                    "grep",
                    "-nEHI",
                    *files_with_matches,
                    args.pattern,
                    *filtered_filenames[i : i + batch_size],
                ]
            )
            lines.extend(proc.stdout.decode().splitlines())
    except Exception as err:
        print_lint_message(
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 381-398
```python
            name="command-failed",
            description=(
                f"Failed due to {err.__class__.__name__}:\n{err}"
                if not isinstance(err, subprocess.CalledProcessError)
                else (
                    "COMMAND (exit code {returncode})\n"
                    "{command}\n\n"
                    "STDERR\n{stderr}\n\n"
                    "STDOUT\n{stdout}"
                ).format(
                    returncode=err.returncode,
                    command=" ".join(as_posix(x) for x in err.cmd),
                    stderr=err.stderr.decode("utf-8").strip() or "(empty)",
                    stdout=err.stdout.decode("utf-8").strip() or "(empty)",
                )
            ),
        )
        sys.exit(0)
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 399-411
```python

    # Group lines by file to call lint_file once per file
    grouped_lines = group_lines_by_file(lines)

    for filename, line_remainders in grouped_lines.items():
        lint_file(
            filename,
            line_remainders,
            args.allowlist_pattern,
            args.replace_pattern,
            args.error_name,
        )
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 412-414
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
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **LintSeverity**
  - EN: `LintSeverity` is one of the main local symbols exposed or implemented here.
  - CN: `LintSeverity` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `argparse`, `json`, `logging`, `os`, `subprocess`, `sys`, `time`, `enum`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `LintSeverity`, `LintMessage`, `as_posix`, `run_command`, `print_lint_message`, `group_lines_by_file`, `check_allowlist`, `lint_file`, `main`
