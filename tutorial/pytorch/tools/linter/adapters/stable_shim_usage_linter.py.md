# stable_shim_usage_linter.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/stable_shim_usage_linter.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```python
#!/usr/bin/env python3
"""
STABLE_SHIM_USAGE: Ensures that calls to versioned shim functions from
torch/csrc/stable/c/shim.h in torch/csrc/stable are properly wrapped in
TORCH_FEATURE_VERSION macros corresponding to the version when those
functions were introduced.
"""

from __future__ import annotations
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。

### Lines 11-19
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

### Lines 20-29
```python

# Add repo root to sys.path so we can import from tools
REPO_ROOT = Path(__file__).resolve().parents[3]
sys.path.insert(0, str(REPO_ROOT))

from tools.linter.adapters.stable_shim_version_linter import PreprocessorTracker


LINTER_CODE = "STABLE_SHIM_USAGE"
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.linter.adapters.stable_shim_version_linter. Configuration constants such as REPO_ROOT, LINTER_CODE centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.linter.adapters.stable_shim_version_linter。 REPO_ROOT、LINTER_CODE 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 30-38
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

### Lines 39-48
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

### Lines 49-59
```python

def get_shim_functions(
    shim_files: list[Path | str] | None = None,
) -> dict[str, tuple[int, int]]:
    """
    Extract function names from shim header files and their required version.
    Returns a dict mapping function name to (major, minor) version tuple.

    Only functions defined inside TORCH_FEATURE_VERSION blocks are extracted.
    Functions without version guards are ignored.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_shim_functions`, which implements a focused step inside the lint tooling pipeline.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_shim_functions`，其作用是实现Lint 工具链流水线中的一个关键步骤。

### Lines 60-73
```python
    Args:
        shim_files: List of paths to shim header files. If None, will use the default
                    paths to torch/csrc/stable/c/shim.h and
                    torch/csrc/inductor/aoti_torch/c/shim.h based on the repository root.
    """
    if shim_files is None:
        repo_root = Path(__file__).resolve().parents[3]
        shim_files_to_check = [
            repo_root / "torch/csrc/stable/c/shim.h",
            repo_root / "torch/csrc/inductor/aoti_torch/c/shim.h",
        ]
    else:
        shim_files_to_check = [Path(f) for f in shim_files]
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `get_shim_functions` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `get_shim_functions`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 74-83
```python
    # Assert that all shim files exist
    missing_files = [f for f in shim_files_to_check if not f.exists()]
    if missing_files:
        raise RuntimeError(
            f"The following shim files do not exist: {missing_files}. "
            "Ensure all shim header files exist in the repository."
        )

    functions: dict[str, tuple[int, int]] = {}
```
- **EN**: This chunk continues `get_shim_functions` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `get_shim_functions`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 84-92
```python
    # Match function declarations like: AOTI_TORCH_EXPORT ... function_name(
    function_pattern = re.compile(r"AOTI_TORCH_EXPORT.+?(\w+)\s*\(")
    # Also match typedef function pointers
    typedef_pattern = re.compile(r"typedef\s+.*\(\*(\w+)\)")
    # Match using declarations like: using TypeName = ...
    using_pattern = re.compile(r"using\s+(\w+)\s*=")
    # Match struct/class declarations like: struct StructName or class ClassName
    struct_class_pattern = re.compile(r"(?:struct|class)\s+(\w+)")
```
- **EN**: This chunk continues `get_shim_functions` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs.
- **CN**: 这一段延续了 `get_shim_functions`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。

### Lines 93-101
```python
    for shim_file in shim_files_to_check:
        with open(shim_file) as f:
            lines = f.readlines()

        tracker = PreprocessorTracker()

        for line in lines:
            is_directive_or_comment = tracker.process_line(line)
```
- **EN**: This chunk continues `get_shim_functions` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_shim_functions`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 102-112
```python
            # Only look for function declarations if not a comment/directive and inside a version block
            if not is_directive_or_comment:
                version_of_block = tracker.get_version_of_block()
                if version_of_block:
                    stripped = line.strip()
                    func_match = function_pattern.search(stripped)
                    if func_match:
                        func_name = func_match.group(1)
                        functions[func_name] = version_of_block
                        continue
```
- **EN**: This chunk continues `get_shim_functions` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_shim_functions`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 113-124
```python
                    typedef_match = typedef_pattern.search(stripped)
                    if typedef_match:
                        func_name = typedef_match.group(1)
                        functions[func_name] = version_of_block
                        continue

                    using_match = using_pattern.search(stripped)
                    if using_match:
                        type_name = using_match.group(1)
                        functions[type_name] = version_of_block
                        continue
```
- **EN**: This chunk continues `get_shim_functions` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_shim_functions`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 125-136
```python
                    struct_class_match = struct_class_pattern.search(stripped)
                    if struct_class_match:
                        type_name = struct_class_match.group(1)
                        functions[type_name] = version_of_block
                        continue

    if not functions:
        raise RuntimeError(
            "Could not find any versioned shim functions. "
            "Ensure at least one of the shim files exists and contains versioned functions."
        )
```
- **EN**: This chunk continues `get_shim_functions` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `get_shim_functions`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 137-146
```python
    return functions


def write_shim_function_versions(
    functions: dict[str, tuple[int, int]],
    output_file: Path | str | None = None,
) -> None:
    """
    Write the shim function versions to a text file.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `write_shim_function_versions`, which generates derived source text, templates, or metadata outputs. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `write_shim_function_versions`，其作用是生成派生源码文本、模板或元数据输出。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 147-157
```python
    Args:
        functions: Dictionary mapping function name to (major, minor) version tuple.
        output_file: Path to the output file. If None, will write to
                     torch/csrc/stable/c/shim_function_versions.txt in the repository.
    """
    if output_file is None:
        repo_root = Path(__file__).resolve().parents[3]
        output_file = repo_root / "torch/csrc/stable/c/shim_function_versions.txt"
    else:
        output_file = Path(output_file)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `write_shim_function_versions` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `write_shim_function_versions`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 158-174
```python
    # Sort functions by version, then by name for consistency
    sorted_functions = sorted(functions.items(), key=lambda x: (x[1], x[0]))

    with open(output_file, "w") as f:
        f.write(
            "# Auto-generated file listing shim functions and their minimum required versions\n"
        )
        f.write("# Format: function_name: TORCH_VERSION_MAJOR_MINOR_PATCH\n")
        f.write("#\n")
        f.write(
            "# This file is automatically updated by the stable_shim_usage_linter.\n"
        )
        f.write(
            "# If a function is not in this file, it was available before 2.10.0.\n"
        )
        f.write("# DO NOT EDIT MANUALLY.\n\n")
```
- **EN**: The text behaves like a template or generated scaffold, so the main value is the declared expansion points and emitted structure. This chunk continues `write_shim_function_versions` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这段内容表现为模板或生成脚手架，因此重点在于声明的展开点和输出结构。 这一段延续了 `write_shim_function_versions`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 175-184
```python
        for func_name, (major, minor) in sorted_functions:
            f.write(f"{func_name}: TORCH_VERSION_{major}_{minor}_0\n")


def check_file(
    filename: str, shim_functions: dict[str, tuple[int, int]]
) -> list[LintMessage]:
    """
    Check the input file for proper usage of versioned shim functions.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `check_file`, which validates invariants and reports policy violations early. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `check_file`，其作用是校验不变量，并尽早报告策略违规。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 185-193
```python
    Args:
        filename: File in torch/csrc/stable that calls functions from shim.
        shim_functions: Dictionary mapping function name to (major, minor) version tuple.
    """
    lint_messages: list[LintMessage] = []

    with open(filename) as f:
        lines = f.readlines()
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `check_file` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 194-203
```python
    tracker = PreprocessorTracker()

    for line_num, line in enumerate(lines, 1):
        is_directive_or_comment = tracker.process_line(line)

        if is_directive_or_comment:
            continue

        version_of_block = tracker.get_version_of_block()
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 204-213
```python
        for func_name, required_version in shim_functions.items():
            # Look for:
            # 1. Function calls like: func_name(
            # 2. Type usage like: func_name variable_name
            # Use word boundaries to avoid matching partial names

            if re.search(rf"\b{re.escape(func_name)}\b", line):
                major, minor = required_version
                required_macro = f"TORCH_VERSION_{major}_{minor}_0"
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 214-231
```python
                if version_of_block is None:
                    # Not inside any version block
                    lint_messages.append(
                        LintMessage(
                            path=filename,
                            line=line_num,
                            char=None,
                            code=LINTER_CODE,
                            severity=LintSeverity.ERROR,
                            name="unversioned-shim-call",
                            original=None,
                            replacement=None,
                            description=(
                                f"Usage '{func_name}' from shim.h is not wrapped "
                                f"in a TORCH_FEATURE_VERSION block. This function requires at least:\n"
                                f"#if TORCH_FEATURE_VERSION >= {required_macro}\n"
                                f"  // ... your code calling {func_name} ...\n"
                                f"#endif // TORCH_FEATURE_VERSION >= {required_macro}"
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 232-249
```python
                            ),
                        )
                    )
                elif version_of_block < required_version:
                    # Inside a version block, but version is too old
                    current_major, current_minor = version_of_block
                    current_macro = f"TORCH_VERSION_{current_major}_{current_minor}_0"
                    lint_messages.append(
                        LintMessage(
                            path=filename,
                            line=line_num,
                            char=None,
                            code=LINTER_CODE,
                            severity=LintSeverity.ERROR,
                            name="insufficient-version-for-shim-call",
                            original=None,
                            replacement=None,
                            description=(
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 250-259
```python
                                f"Use of '{func_name}' is wrapped in {current_macro}, "
                                f"but this function requires at least {required_macro}. "
                                f"The version guard must be at least the required version:\n"
                                f"#if TORCH_FEATURE_VERSION >= {required_macro}\n"
                                f"  // ... your code calling {func_name} ...\n"
                                f"#endif // TORCH_FEATURE_VERSION >= {required_macro}"
                            ),
                        )
                    )
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 260-277
```python
    return lint_messages


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="stable shim usage linter",
        fromfile_prefix_chars="@",
    )
    parser.add_argument(
        "--verbose",
        action="store_true",
    )
    parser.add_argument(
        "filenames",
        nargs="+",
        help="paths to lint",
    )
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 278-289
```python
    args = parser.parse_args()

    logging.basicConfig(
        format="<%(threadName)s:%(levelname)s> %(message)s",
        level=logging.NOTSET if args.verbose else logging.DEBUG,
        stream=sys.stderr,
    )

    # Update the shim function versions file
    shim_functions = get_shim_functions()
    write_shim_function_versions(shim_functions)
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 290-295
```python
    lint_messages = []
    for filename in args.filenames:
        lint_messages.extend(check_file(filename, shim_functions))

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
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **REPO_ROOT**
  - EN: `REPO_ROOT` is one of the main local symbols exposed or implemented here.
  - CN: `REPO_ROOT` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.linter.adapters.stable_shim_version_linter`
- **Python standard library / Python 标准库**: `__future__`, `argparse`, `json`, `logging`, `re`, `sys`, `enum`, `pathlib`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `LINTER_CODE`, `LintSeverity`, `LintMessage`, `get_shim_functions`, `write_shim_function_versions`, `check_file`
