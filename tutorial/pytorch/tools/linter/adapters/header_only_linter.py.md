# header_only_linter.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/header_only_linter.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```python
#!/usr/bin/env python3
"""
Checks that all symbols in torch/header_only_apis.txt are tested in a .cpp
test file to ensure header-only-ness. The .cpp test file must be built
without linking libtorch.
"""
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。

### Lines 8-14
```python
import argparse
import json
import re
from enum import Enum
from pathlib import Path
from typing import NamedTuple
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, json, re, and 3 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、json、re 等共 6 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 15-24
```python

LINTER_CODE = "HEADER_ONLY_LINTER"


class LintSeverity(str, Enum):
    ERROR = "error"
    WARNING = "warning"
    ADVICE = "advice"
    DISABLED = "disabled"
```
- **EN**: It introduces classes such as LintSeverity, which package state and behavior for this tooling task. This chunk continues `LintSeverity` and expands its internal control flow or data movement. Configuration constants such as LINTER_CODE, ERROR, WARNING, and 2 more centralize defaults so later functions share the same policy knobs.
- **CN**: 它引入了 LintSeverity 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `LintSeverity`，进一步展开其内部控制流或数据流转。 LINTER_CODE、ERROR、WARNING 等共 5 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 25-36
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

### Lines 37-43
```python

CPP_TEST_GLOBS = [
    "test/cpp/aoti_abi_check/*.cpp",
]

REPO_ROOT = Path(__file__).parents[3]
```
- **EN**: This chunk continues `LintMessage` and expands its internal control flow or data movement. Configuration constants such as CPP_TEST_GLOBS, REPO_ROOT centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段延续了 `LintMessage`，进一步展开其内部控制流或数据流转。 CPP_TEST_GLOBS、REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 44-55
```python

def find_matched_symbols(
    symbols_regex: re.Pattern[str], test_globs: list[str] = CPP_TEST_GLOBS
) -> set[str]:
    """
    Goes through all lines not starting with // in the cpp files and
    accumulates a list of matches with the symbols_regex. Note that
    we expect symbols_regex to be sorted in reverse alphabetical
    order to allow superset regexes to get matched.
    """
    matched_symbols = set()
    # check noncommented out lines of the test files
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `find_matched_symbols`, which implements a focused step inside the lint tooling pipeline. Pattern-matching logic extracts structured facts from loosely formatted text inputs.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `find_matched_symbols`，其作用是实现Lint 工具链流水线中的一个关键步骤。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。

### Lines 56-61
```python
    for cpp_test_glob in test_globs:
        for test_file in REPO_ROOT.glob(cpp_test_glob):
            with open(test_file) as tf:
                for test_file_line in tf:
                    test_file_line = test_file_line.strip()
                    if test_file_line.startswith(("//", "#")) or test_file_line == "":
```
- **EN**: This chunk continues `find_matched_symbols` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `find_matched_symbols`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 62-68
```python
                        continue
                    matches = re.findall(symbols_regex, test_file_line)
                    for m in matches:
                        if m != "":
                            matched_symbols.add(m)
    return matched_symbols
```
- **EN**: This chunk continues `find_matched_symbols` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `find_matched_symbols`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 69-77
```python

def check_file(
    filename: str, test_globs: list[str] = CPP_TEST_GLOBS
) -> list[LintMessage]:
    """
    Goes through the header_only_apis.txt file and verifies that all symbols
    within the file can be found tested in an appropriately independent .cpp
    file.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `check_file`, which validates invariants and reports policy violations early.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `check_file`，其作用是校验不变量，并尽早报告策略违规。

### Lines 78-85
```python
    Note that we expect CPP_TEST_GLOBS to be passed in as test_globs--the
    only reason this is an argument at all is for ease of testing.
    """
    lint_messages: list[LintMessage] = []

    symbols: dict[str, int] = {}  # symbol -> lineno
    with open(filename) as f:
        for idx, line in enumerate(f):
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `check_file` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 86-94
```python
            # commented out lines should be skipped
            symbol = line.strip()
            if not symbol or symbol[0] == "#":
                continue

            # symbols can in fact be duplicated and come from different headers.
            # we are aware this is a flaw in using simple string matching.
            symbols[symbol] = idx + 1
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 95-101
```python
    # Why reverse the keys? To allow superset regexes to get matched first in
    # find_matched_symbols. For example, we want Float8_e5m2fnuz to match
    # before Float8_e5m2. Otherwise, both Float8_e5m2fnuz and Float8_e5m2 will
    # match Float8_e5m2
    symbols_regex = re.compile("|".join(sorted(symbols.keys(), reverse=True)))
    matched_symbols = find_matched_symbols(symbols_regex, test_globs)
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。

### Lines 102-113
```python
    for s, lineno in symbols.items():
        if s not in matched_symbols:
            lint_messages.append(
                LintMessage(
                    path=filename,
                    line=lineno,
                    char=None,
                    code=LINTER_CODE,
                    severity=LintSeverity.ERROR,
                    name="[untested-symbol]",
                    original=None,
                    replacement=None,
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 114-125
```python
                    description=(
                        f"{s} has been included as a header-only API "
                        "but is not tested in any of CPP_TEST_GLOBS, which "
                        f"contains {CPP_TEST_GLOBS}.\n"
                        "Please add a .cpp test using the symbol without "
                        "linking anything to verify that the symbol is in "
                        "fact header-only. If you already have a test but it's"
                        " not found, please add the .cpp file to CPP_TEST_GLOBS"
                        " in tools/linters/adapters/header_only_linter.py."
                    ),
                )
            )
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。

### Lines 126-136
```python

    return lint_messages


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="header only APIs linter",
        fromfile_prefix_chars="@",
    )
    args = parser.parse_args()
```
- **EN**: This chunk continues `check_file` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_file`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 137-140
```python
    for lint_message in check_file(
        str(REPO_ROOT) + "/torch/header_only_apis.txt", CPP_TEST_GLOBS
    ):
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
- **LINTER_CODE**
  - EN: `LINTER_CODE` is one of the main local symbols exposed or implemented here.
  - CN: `LINTER_CODE` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `argparse`, `json`, `re`, `enum`, `pathlib`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `LINTER_CODE`, `LintSeverity`, `LintMessage`, `CPP_TEST_GLOBS`, `REPO_ROOT`, `find_matched_symbols`, `check_file`
