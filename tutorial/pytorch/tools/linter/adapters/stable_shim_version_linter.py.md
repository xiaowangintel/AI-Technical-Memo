# stable_shim_version_linter.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/stable_shim_version_linter.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```python
#!/usr/bin/env python3
"""
STABLE_SHIM_VERSION: Ensures that function declarations in stable/c/shim.h
are properly wrapped in TORCH_FEATURE_VERSION macros corresponding to the
current TORCH_ABI_VERSION.
"""

from __future__ import annotations
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。

### Lines 10-18
```python
import argparse
import json
import logging
import re
import sys
from enum import Enum
from pathlib import Path
from typing import NamedTuple
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, json, logging, and 5 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、json、logging 等共 8 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 19-28
```python

# Add repo root to sys.path so we can import from tools.setup_helpers
REPO_ROOT = Path(__file__).resolve().parents[3]
sys.path.insert(0, str(REPO_ROOT))

from tools.setup_helpers.gen_version_header import parse_version


LINTER_CODE = "STABLE_SHIM_VERSION"
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.setup_helpers.gen_version_header. Configuration constants such as REPO_ROOT, LINTER_CODE centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.setup_helpers.gen_version_header。 REPO_ROOT、LINTER_CODE 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 29-37
```python

class PreprocessorTracker:
    """
    Helper class to track preprocessor directives and version blocks.

    This class maintains state as it processes C/C++ preprocessor directives
    (#if, #elif, #else, #endif) and tracks which code is inside version blocks.
    """
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as PreprocessorTracker, which package state and behavior for this tooling task. This chunk continues `PreprocessorTracker` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 PreprocessorTracker 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `PreprocessorTracker`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 38-47
```python
    def __init__(self):
        """Initialize the preprocessor tracker."""
        # Stack of (is_version_block, version_tuple) tuples
        # is_version_block: True if this is a TORCH_FEATURE_VERSION >= TORCH_VERSION_X_Y_0 block
        # version_tuple: (major, minor) if is_version_block is True, else None
        self.preprocessor_stack: list[tuple[bool, tuple[int, int] | None]] = []

        # Current version requirement (if inside a version block)
        self.version_of_block: tuple[int, int] | None = None
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `__init__`, which implements a focused step inside the lint tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `__init__`，其作用是实现Lint 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 48-56
```python
        # Track if we're inside a block comment
        self.in_block_comment: bool = False

        # Regex to match version conditions in #if or #elif
        self.version_pattern = re.compile(
            r"#(?:if|elif)\s+TORCH_FEATURE_VERSION\s*>=\s*TORCH_VERSION_(\d+)_(\d+)_\d+"
        )

    def process_line(self, line: str) -> bool:
```
- **EN**: This chunk defines `process_line`, which implements a focused step inside the lint tooling pipeline. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `process_line`，其作用是实现Lint 工具链流水线中的一个关键步骤。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 57-68
```python
        """
        Process a line and update the preprocessor state.

        Args:
            line: The line to process

        Returns:
            True if the line was processed (is a preprocessor directive or comment),
            False if it's a regular code line that should be further analyzed.
        """
        stripped = line.strip()
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `process_line` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `process_line`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 69-79
```python
        # Handle block comments (/* ... */)
        # Check if we're entering a block comment
        if "/*" in line:
            self.in_block_comment = True

        # If we're in a block comment, check if we're exiting
        if self.in_block_comment:
            if "*/" in line:
                self.in_block_comment = False
            return True  # Skip the line if we're in a block comment
```
- **EN**: This chunk continues `process_line` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `process_line`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 80-97
```python
        # Skip line comments - they're not active code
        if stripped.startswith("//"):
            return True

        # Track #if directives
        if stripped.startswith("#if"):
            version_match = self.version_pattern.match(stripped)
            if version_match:
                major = int(version_match.group(1))
                minor = int(version_match.group(2))
                version_tuple = (major, minor)
                self.preprocessor_stack.append((True, version_tuple))
                self.version_of_block = version_tuple
            else:
                # Regular #if (not a version block)
                self.preprocessor_stack.append((False, None))
            return True
```
- **EN**: This chunk continues `process_line` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `process_line`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 98-107
```python
        # Track #ifdef and #ifndef directives (not version blocks)
        if stripped.startswith(("#ifdef", "#ifndef")):
            self.preprocessor_stack.append((False, None))
            return True

        # Track #endif directives
        if stripped.startswith("#endif"):
            if self.preprocessor_stack:
                is_version_block, _ = self.preprocessor_stack.pop()
                if is_version_block:
```
- **EN**: This chunk continues `process_line` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `process_line`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 108-118
```python
                    # Restore previous version block if any
                    self.version_of_block = None
                    for i in range(len(self.preprocessor_stack) - 1, -1, -1):
                        if self.preprocessor_stack[i][0]:
                            self.version_of_block = self.preprocessor_stack[i][1]
                            break
            return True

        # Track #else directives
        # #else replaces the previous #if or #elif, so we pop and push
        if stripped.startswith("#else"):
```
- **EN**: This chunk continues `process_line` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `process_line`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 119-128
```python
            if self.preprocessor_stack:
                self.preprocessor_stack.pop()
            # #else is never versioned, so push (False, None)
            self.preprocessor_stack.append((False, None))
            self.version_of_block = None
            return True

        # Track #elif directives
        # #elif replaces the previous #if or #elif, so we pop and push
        if stripped.startswith("#elif"):
```
- **EN**: This chunk continues `process_line` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `process_line`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 129-146
```python
            if self.preprocessor_stack:
                self.preprocessor_stack.pop()

            self.version_of_block = None

            # Check if this #elif has a version condition
            version_match_elif = self.version_pattern.match(stripped)
            if version_match_elif:
                major = int(version_match_elif.group(1))
                minor = int(version_match_elif.group(2))
                version_tuple = (major, minor)
                self.preprocessor_stack.append((True, version_tuple))
                self.version_of_block = version_tuple
            else:
                # Not a version elif, treat as regular conditional
                self.preprocessor_stack.append((False, None))
            return True
```
- **EN**: This chunk continues `process_line` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `process_line`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 147-157
```python
        # Not a preprocessor directive or comment
        return False

    def is_in_version_block(self) -> bool:
        """Check if currently inside any version block."""
        return self.version_of_block is not None

    def get_version_of_block(self) -> tuple[int, int] | None:
        """Get the current version requirement, or None if not in a version block."""
        return self.version_of_block
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_version_of_block`, which implements a focused step inside the lint tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_version_of_block`，其作用是实现Lint 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 158-166
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

### Lines 167-176
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

### Lines 177-188
```python

def get_current_version() -> tuple[int, int]:
    """
    Get the current PyTorch version from version.txt.
    This uses the same logic as tools/setup_helpers/gen_version_header.py
    which is used to generate torch/headeronly/version.h from version.h.in.

    Returns (major, minor) tuple or None if not found.
    """
    repo_root = Path(__file__).resolve().parents[3]
    version_file = repo_root / "version.txt"
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_current_version`, which implements a focused step inside the lint tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_current_version`，其作用是实现Lint 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 189-197
```python
    if not version_file.exists():
        raise RuntimeError(
            "Could not find version.txt. This linter requires version.txt to run"
        )

    with open(version_file) as f:
        version = f.read().strip()
        major, minor, patch = parse_version(version)
```
- **EN**: This chunk continues `get_current_version` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `get_current_version`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 198-206
```python
    return (major, minor)


def get_added_lines(filename: str) -> set[int]:
    """
    Get the line numbers of added lines in:
    1. Current uncommitted changes (git diff HEAD)
    2. All commits in the current PR (git diff merge-base..HEAD)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_added_lines`, which implements a focused step inside the lint tooling pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_added_lines`，其作用是实现Lint 工具链流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 207-215
```python
    This ensures that in CI we catch version macro issues across all PR commits.

    Returns:
        Set of line numbers (1-indexed) that are new additions.
    """
    import subprocess

    added_lines = set()
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as subprocess. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 subprocess。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 216-224
```python
    def parse_diff(diff_output: str) -> set[int]:
        """Parse git diff output and return line numbers of added lines."""
        lines = set()
        current_line = 0
        for line in diff_output.split("\n"):
            # Unified diff format: @@ -old_start,old_count +new_start,new_count @@
            if line.startswith("@@"):
                match = re.search(r"\+(\d+)", line)
                if match:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `parse_diff`, which parses or loads structured input into tool-friendly data structures. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `parse_diff`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 225-234
```python
                    current_line = int(match.group(1))
            elif line.startswith("+") and not line.startswith("+++"):
                # This is an added line
                lines.add(current_line)
                current_line += 1
            elif not line.startswith("-"):
                # Context line or unchanged line
                current_line += 1
        return lines
```
- **EN**: This chunk continues `parse_diff` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `parse_diff`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 235-243
```python
    try:
        # Check uncommitted changes (working directory vs HEAD)
        result = subprocess.run(
            ["git", "diff", "HEAD", filename],
            capture_output=True,
            text=True,
            timeout=5,
        )
        if result.returncode == 0:
```
- **EN**: This chunk continues `parse_diff` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `parse_diff`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 244-253
```python
            added_lines.update(parse_diff(result.stdout))

        # Get merge-base with origin/main to check all PR commits
        result = subprocess.run(
            ["git", "fetch", "origin", "main"],
            capture_output=True,
            text=True,
            timeout=600,
        )
        if result.returncode != 0:
```
- **EN**: This chunk continues `parse_diff` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `parse_diff`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 254-264
```python
            raise RuntimeError(
                f"Failed to fetch origin. Error: {result.stderr.strip()}"
            )

        result = subprocess.run(
            ["git", "merge-base", "HEAD", "origin/main"],
            capture_output=True,
            text=True,
            timeout=5,
        )
        if result.returncode != 0:
```
- **EN**: This chunk continues `parse_diff` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `parse_diff`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 265-278
```python
            raise RuntimeError(
                f"Failed to find merge-base with origin/main. "
                f"Make sure origin/main exists (run 'git fetch origin main'). "
                f"Error: {result.stderr.strip()}"
            )

        merge_base = result.stdout.strip()
        result = subprocess.run(
            ["git", "diff", f"{merge_base}..HEAD", filename],
            capture_output=True,
            text=True,
            timeout=5,
        )
        if result.returncode != 0:
```
- **EN**: This chunk continues `parse_diff` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `parse_diff`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 279-288
```python
            raise RuntimeError(
                f"Failed to get git diff information for {filename}. Error: {result.stderr}"
            )
        added_lines.update(parse_diff(result.stdout))

    except Exception as e:
        raise RuntimeError(
            f"Failed to get git diff information for {filename}. Error: {e}"
        ) from e
```
- **EN**: This chunk continues `parse_diff` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `parse_diff`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 289-297
```python
    return added_lines


def check_file(filename: str) -> list[LintMessage]:
    """
    Parse the stable/c/shim.h file and check that:
    1. All function declarations are within TORCH_FEATURE_VERSION blocks
    2. New functions added in this commit use the current version macro
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `check_file`, which validates invariants and reports policy violations early. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `check_file`，其作用是校验不变量，并尽早报告策略违规。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 298-306
```python
    For the AOTI shim (torch/csrc/inductor/aoti_torch/c/shim.h), we only
    enforce versioning on NEW function declarations, since existing functions
    are intentionally not version-guarded.
    """
    lint_messages: list[LintMessage] = []

    # Check if this is the AOTI shim - only enforce versioning on new lines
    is_aoti_shim = "torch/csrc/inductor/aoti_torch/c/shim.h" in filename
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 307-315
```python
    # Get current version
    current_version = get_current_version()
    major, minor = current_version
    expected_version_macro = f"TORCH_VERSION_{major}_{minor}_0"
    expected_version_check = f"#if TORCH_FEATURE_VERSION >= {expected_version_macro}"

    # Get lines that are uncommitted or added in the most recent commit
    added_lines = get_added_lines(filename)
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 316-324
```python
    with open(filename) as f:
        lines = f.readlines()

    # Use PreprocessorTracker to handle preprocessor directives
    tracker = PreprocessorTracker()

    # Track extern "C" blocks separately
    inside_extern_c = False
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 325-335
```python
    # Patterns for extern "C" blocks
    extern_c_pattern = re.compile(r'extern\s+"C"\s*{')
    extern_c_end_pattern = re.compile(r'}\s*//\s*extern\s+"C"')

    # Function declaration patterns - looking for AOTI_TORCH_EXPORT or typedef
    function_decl_patterns = [
        re.compile(r"^\s*AOTI_TORCH_EXPORT\s+\w+"),  # AOTI_TORCH_EXPORT functions
        re.compile(r"^\s*typedef\s+.*\(\*\w+\)"),  # typedef function pointers
        re.compile(r"^\s*using\s+\w+\s*="),  # using declarations
    ]
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 336-345
```python
    for line_num, line in enumerate(lines, 1):
        stripped = line.strip()

        # Skip empty lines
        if not stripped:
            continue

        # Let the tracker process preprocessor directives and comments
        is_directive_or_comment = tracker.process_line(line)
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 346-356
```python
        if is_directive_or_comment:
            continue

        # Track extern "C" blocks
        if extern_c_pattern.search(stripped):
            inside_extern_c = True
            continue
        if extern_c_end_pattern.search(stripped):
            inside_extern_c = False
            continue
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 357-366
```python
        # Check for function declarations
        if inside_extern_c:
            is_function_decl = any(
                pattern.match(stripped) for pattern in function_decl_patterns
            )

            if is_function_decl:
                # Check if this is a newly added line
                is_new_line = line_num in added_lines
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 367-375
```python
                # Get current version state from tracker
                inside_version_block = tracker.is_in_version_block()
                tracker_version = tracker.get_version_of_block()
                version_of_block_macro = (
                    f"TORCH_VERSION_{tracker_version[0]}_{tracker_version[1]}_0"
                    if tracker_version
                    else None
                )
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 376-393
```python
                if not inside_version_block:
                    # Function declaration outside of version block
                    if not is_new_line:
                        # Existing function declaration outside of version block in aoti shim is ignored
                        if is_aoti_shim:
                            continue
                        expected_version_macro_str = "TORCH_VERSION_X_Y_Z"
                        expected_version_check_str = (
                            f"#if TORCH_FEATURE_VERSION >= {expected_version_macro}"
                        )
                        additional_text = "\nX, Y, and Z correspond to the TORCH_ABI_VERSION when the function was added."
                    else:
                        expected_version_macro_str = expected_version_macro
                        expected_version_check_str = expected_version_check
                        additional_text = ""
                    lint_messages.append(
                        LintMessage(
                            path=filename,
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 394-411
```python
                            line=line_num,
                            char=None,
                            code=LINTER_CODE,
                            severity=LintSeverity.ERROR,
                            name="unversioned-function-declaration",
                            original=None,
                            replacement=None,
                            description=(
                                f"Function declaration found outside of TORCH_FEATURE_VERSION block. "
                                f"All function declarations must be wrapped in:\n"
                                f"{expected_version_check_str}\n"
                                f"// ... your declarations ...\n"
                                f"#endif // TORCH_FEATURE_VERSION >= {expected_version_macro_str}"
                                f"{additional_text}"
                            ),
                        )
                    )
                elif is_new_line and version_of_block_macro != expected_version_macro:
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 412-429
```python
                    # New function declaration using wrong version macro
                    lint_messages.append(
                        LintMessage(
                            path=filename,
                            line=line_num,
                            char=None,
                            code=LINTER_CODE,
                            severity=LintSeverity.ERROR,
                            name="wrong-version-for-new-function",
                            original=None,
                            replacement=None,
                            description=(
                                f"New function declaration should use {expected_version_macro}, "
                                f"but is wrapped in {version_of_block_macro}. "
                                f"New additions in this commit must use the current version:\n"
                                f"{expected_version_check}\n"
                                f"// ... your declarations ...\n"
                                f"#endif // TORCH_FEATURE_VERSION >= {expected_version_macro}"
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 430-447
```python
                            ),
                        )
                    )

    return lint_messages


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="stable shim version linter",
        fromfile_prefix_chars="@",
    )
    parser.add_argument(
        "--verbose",
        action="store_true",
    )
    parser.add_argument(
        "filenames",
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 448-457
```python
        nargs="+",
        help="paths to lint",
    )

    args = parser.parse_args()

    logging.basicConfig(
        format="<%(threadName)s:%(levelname)s> %(message)s",
        level=logging.NOTSET
        if args.verbose
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 458-467
```python
        else logging.DEBUG
        if len(args.filenames) < 1000
        else logging.INFO,
        stream=sys.stderr,
    )

    lint_messages = []
    for filename in args.filenames:
        lint_messages.extend(check_file(filename))
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 468-469
```python
    for lint_message in lint_messages:
        print(json.dumps(lint_message._asdict()), flush=True)
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

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

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.setup_helpers.gen_version_header`
- **Python standard library / Python 标准库**: `__future__`, `argparse`, `json`, `logging`, `re`, `sys`, `enum`, `pathlib`, `typing`, `subprocess`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `LINTER_CODE`, `PreprocessorTracker`, `LintSeverity`, `LintMessage`, `get_current_version`, `get_added_lines`, `check_file`
