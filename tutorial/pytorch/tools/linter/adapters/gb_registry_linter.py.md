# gb_registry_linter.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/gb_registry_linter.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```python
# mypy: ignore-errors

from __future__ import annotations

import argparse
import ast
import functools
import json
import random
import sys
from enum import Enum
from pathlib import Path
from typing import Any, NamedTuple
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, argparse, ast, and 7 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、argparse、ast 等共 10 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 15-25
```python

# Patch ast._splitlines_no_ff with caching to avoid O(n²) re-splitting.
# get_source_segment() calls _splitlines_no_ff() for every keyword argument,
# but the same source string is passed repeatedly for the same file.
if hasattr(ast, "_splitlines_no_ff"):
    ast._splitlines_no_ff = functools.lru_cache(maxsize=128)(ast._splitlines_no_ff)


REPO_ROOT = Path(__file__).resolve().parents[3]
sys.path.insert(0, str(REPO_ROOT))
```
- **EN**: Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 26-35
```python

from tools.dynamo.gb_id_mapping import (
    find_unimplemented_calls,
    load_registry,
    next_gb_id,
)


LINTER_CODE = "GB_REGISTRY"
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.dynamo.gb_id_mapping. Configuration constants such as LINTER_CODE centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.dynamo.gb_id_mapping。 LINTER_CODE 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 36-44
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

### Lines 45-54
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

### Lines 55-64
```python

def _is_noqa_suppressed(source_lines: list[str], lineno: int) -> bool:
    if lineno <= 0 or lineno > len(source_lines):
        return False
    if source_lines[lineno - 1].rstrip().endswith(f"# noqa: {LINTER_CODE}"):
        return True
    if lineno > 1 and source_lines[lineno - 2].strip() == f"# noqa: {LINTER_CODE}":
        return True
    return False
```
- **EN**: This chunk defines `_is_noqa_suppressed`, which implements a focused step inside the lint tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_is_noqa_suppressed`，其作用是实现Lint 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 65-74
```python

def _is_forbidden_raise(node: ast.Raise) -> bool:
    if not isinstance(node.exc, ast.Call):
        return False
    if isinstance(node.exc.func, ast.Name):
        return node.exc.func.id == "Unsupported"
    if isinstance(node.exc.func, ast.Attribute):
        return node.exc.func.attr == "Unsupported"
    return False
```
- **EN**: This chunk defines `_is_forbidden_raise`, which implements a focused step inside the lint tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_is_forbidden_raise`，其作用是实现Lint 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 75-84
```python

def _collect_forbidden_unsupported_raises(
    dynamo_dir: Path,
) -> list[tuple[Path, int, int]]:
    forbidden_raises: list[tuple[Path, int, int]] = []

    for py_file in dynamo_dir.rglob("*.py"):
        source = py_file.read_text(encoding="utf-8")
        source_lines = source.splitlines()
        try:
```
- **EN**: This chunk defines `_collect_forbidden_unsupported_raises`, which implements a focused step inside the lint tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `_collect_forbidden_unsupported_raises`，其作用是实现Lint 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 85-95
```python
            tree = ast.parse(source, filename=str(py_file))
        except SyntaxError:
            continue

        for node in ast.walk(tree):
            if not isinstance(node, ast.Raise) or not _is_forbidden_raise(node):
                continue
            if _is_noqa_suppressed(source_lines, node.lineno):
                continue
            forbidden_raises.append((py_file, node.lineno, node.col_offset + 1))
```
- **EN**: This chunk continues `_collect_forbidden_unsupported_raises` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `_collect_forbidden_unsupported_raises`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 96-104
```python
    return forbidden_raises


def _collect_all_calls(
    dynamo_dir: Path,
) -> dict[str, list[tuple[dict[str, Any], Path]]]:
    """Return mapping *gb_type → list[(call_info, file_path)]* for all occurrences."""
    gb_type_calls: dict[str, list[tuple[dict[str, Any], Path]]] = {}
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_collect_all_calls`, which implements a focused step inside the lint tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_collect_all_calls`，其作用是实现Lint 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 105-113
```python
    for py_file in dynamo_dir.rglob("*.py"):
        for call in find_unimplemented_calls(py_file, dynamo_dir):
            gb_type = call["gb_type"]
            if gb_type not in gb_type_calls:
                gb_type_calls[gb_type] = []
            gb_type_calls[gb_type].append((call, py_file))

    return gb_type_calls
```
- **EN**: This chunk continues `_collect_all_calls` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_collect_all_calls`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 114-125
```python

def _create_registry_entry(
    gb_type: str, context: str, explanation: str, hints: list[str]
) -> dict[str, Any]:
    """Create a registry entry with consistent format."""
    return {
        "Gb_type": gb_type,
        "Context": context,
        "Explanation": explanation,
        "Hints": hints or [],
    }
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_create_registry_entry`, which implements a focused step inside the lint tooling pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_create_registry_entry`，其作用是实现Lint 工具链流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 126-135
```python

def _update_registry_with_changes(
    registry: dict,
    calls: dict[str, tuple[dict[str, Any], Path]],
    renames: dict[str, str] | None = None,
) -> dict:
    """Calculate what the updated registry should look like."""
    renames = renames or {}
    updated_registry = dict(registry)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_update_registry_with_changes`, which implements a focused step inside the lint tooling pipeline.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_update_registry_with_changes`，其作用是实现Lint 工具链流水线中的一个关键步骤。

### Lines 136-147
```python
    latest_entry: dict[str, Any] = {
        entries[0]["Gb_type"]: entries[0] for entries in registry.values()
    }
    gb_type_to_key: dict[str, str] = {
        entries[0]["Gb_type"]: key for key, entries in registry.items()
    }

    # Method for determining add vs. update:
    # - If gb_type exists in registry but content differs: UPDATE (append new entry to preserve history)
    # - If gb_type is new but content matches existing entry: RENAME (append new entry with new gb_type)
    # - If gb_type is completely new: ADD (create new registry entry with a new GBID)
```
- **EN**: This chunk continues `_update_registry_with_changes` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `_update_registry_with_changes`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 148-159
```python
    for old_gb_type, new_gb_type in renames.items():
        registry_key = gb_type_to_key[old_gb_type]
        old_entry = updated_registry[registry_key][0]

        new_entry = _create_registry_entry(
            new_gb_type,
            old_entry["Context"],
            old_entry["Explanation"],
            old_entry["Hints"],
        )
        updated_registry[registry_key] = [new_entry] + updated_registry[registry_key]
```
- **EN**: This chunk continues `_update_registry_with_changes` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `_update_registry_with_changes`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 160-168
```python
        latest_entry[new_gb_type] = new_entry
        gb_type_to_key[new_gb_type] = registry_key
        del latest_entry[old_gb_type]
        del gb_type_to_key[old_gb_type]

    # Collect new entries separately to insert them all at once
    new_entries: list[tuple[str, list[dict[str, Any]]]] = []

    for gb_type, (call, file_path) in calls.items():
```
- **EN**: This chunk continues `_update_registry_with_changes` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `_update_registry_with_changes`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 169-186
```python
        if gb_type in latest_entry:
            existing_entry = latest_entry[gb_type]

            if not (
                call["context"] == existing_entry["Context"]
                and call["explanation"] == existing_entry["Explanation"]
                and sorted(call["hints"]) == sorted(existing_entry["Hints"])
            ):
                registry_key = gb_type_to_key[gb_type]
                new_entry = _create_registry_entry(
                    gb_type, call["context"], call["explanation"], call["hints"]
                )
                updated_registry[registry_key] = [new_entry] + updated_registry[
                    registry_key
                ]
        else:
            # Collect new entries to add later
            new_key = next_gb_id(updated_registry)
```
- **EN**: This chunk continues `_update_registry_with_changes` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_update_registry_with_changes`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 187-195
```python
            new_entry = _create_registry_entry(
                gb_type, call["context"], call["explanation"], call["hints"]
            )
            new_entries.append((new_key, [new_entry]))
            # Temporarily add to updated_registry so next_gb_id works correctly
            updated_registry[new_key] = [new_entry]

    # Insert all new entries at the same random position to reduce merge conflicts
    if new_entries:
```
- **EN**: This chunk continues `_update_registry_with_changes` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_update_registry_with_changes`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 196-205
```python
        # Remove temporarily added entries
        for new_key, _ in new_entries:
            del updated_registry[new_key]

        registry_items = list(updated_registry.items())
        if registry_items:
            # Pick one random position for all new entries
            insert_pos = random.randint(0, len(registry_items))
            # Insert all new entries at the same position
            for new_key, new_entry in new_entries:
```
- **EN**: This chunk continues `_update_registry_with_changes` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_update_registry_with_changes`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 206-215
```python
                registry_items.insert(insert_pos, (new_key, new_entry))
                insert_pos += 1  # Keep them together
            updated_registry = dict(registry_items)
        else:
            # Empty registry, just add all entries
            for new_key, new_entry in new_entries:
                updated_registry[new_key] = new_entry

    return updated_registry
```
- **EN**: This chunk continues `_update_registry_with_changes` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_update_registry_with_changes`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 216-233
```python

def check_registry_sync(dynamo_dir: Path, registry_path: Path) -> list[LintMessage]:
    """Check registry sync and return lint messages."""
    lint_messages = []

    forbidden_raises = _collect_forbidden_unsupported_raises(dynamo_dir)
    for path, line, char in forbidden_raises:
        lint_messages.append(
            LintMessage(
                path=str(path),
                line=line,
                char=char,
                code=LINTER_CODE,
                severity=LintSeverity.ERROR,
                name="Direct raise Unsupported",
                original=None,
                replacement=None,
                description=(
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `check_registry_sync`, which validates invariants and reports policy violations early. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `check_registry_sync`，其作用是校验不变量，并尽早报告策略违规。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 234-242
```python
                    "Do not directly `raise Unsupported(...)` in `torch/_dynamo`. "
                    "Use `unimplemented(...)` for graph breaks, or add `# noqa: GB_REGISTRY` "
                    "for infra-only exceptions."
                ),
            )
        )

    all_calls = _collect_all_calls(dynamo_dir)
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 243-255
```python
    duplicates = []
    for gb_type, call_list in all_calls.items():
        if len(call_list) > 1:
            first_call = call_list[0][0]
            for call, file_path in call_list[1:]:
                if (
                    call["context"] != first_call["context"]
                    or call["explanation"] != first_call["explanation"]
                    or sorted(call["hints"]) != sorted(first_call["hints"])
                ):
                    duplicates.append({"gb_type": gb_type, "calls": call_list})
                    break
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 256-273
```python
    for dup in duplicates:
        gb_type = dup["gb_type"]
        calls = dup["calls"]

        description = f"The gb_type '{gb_type}' is used {len(calls)} times with different content. "
        description += "Each gb_type must be unique across your entire codebase."

        lint_messages.append(
            LintMessage(
                path=str(calls[0][1]),
                line=None,
                char=None,
                code=LINTER_CODE,
                severity=LintSeverity.ERROR,
                name="Duplicate gb_type",
                original=None,
                replacement=None,
                description=description,
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 274-283
```python
            )
        )

    if duplicates:
        return lint_messages

    calls = {gb_type: calls[0] for gb_type, calls in all_calls.items()}

    registry = load_registry(registry_path)
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 284-295
```python
    # Check for duplicate gb_types across different GB IDs in the registry
    gb_type_to_ids: dict[str, list[str]] = {}
    for gb_id, entries in registry.items():
        gb_type = entries[0]["Gb_type"]
        if gb_type not in gb_type_to_ids:
            gb_type_to_ids[gb_type] = []
        gb_type_to_ids[gb_type].append(gb_id)

    duplicate_gb_types_in_registry = [
        (gb_type, ids) for gb_type, ids in gb_type_to_ids.items() if len(ids) > 1
    ]
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 296-313
```python
    if duplicate_gb_types_in_registry:
        for gb_type, ids in duplicate_gb_types_in_registry:
            description = (
                f"The gb_type '{gb_type}' appears in multiple GB IDs: {', '.join(sorted(ids))}. "
                f"Each gb_type must map to exactly one GB ID. Please manually fix the registry."
            )
            lint_messages.append(
                LintMessage(
                    path=str(registry_path),
                    line=None,
                    char=None,
                    code=LINTER_CODE,
                    severity=LintSeverity.ERROR,
                    name="Duplicate gb_type in registry",
                    original=None,
                    replacement=None,
                    description=description,
                )
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 314-323
```python
            )
        return lint_messages

    latest_entry: dict[str, Any] = {
        entries[0]["Gb_type"]: entries[0] for entries in registry.values()
    }

    renames: dict[str, str] = {}
    remaining_calls = dict(calls)
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 324-335
```python
    for gb_type, (call, file_path) in calls.items():
        if gb_type not in latest_entry:
            for existing_gb_type, existing_entry in latest_entry.items():
                if (
                    call["context"] == existing_entry["Context"]
                    and call["explanation"] == existing_entry["Explanation"]
                    and sorted(call["hints"]) == sorted(existing_entry["Hints"])
                ):
                    renames[existing_gb_type] = gb_type
                    del remaining_calls[gb_type]
                    break
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 336-352
```python
    needs_update = bool(renames)

    for gb_type, (call, file_path) in remaining_calls.items():
        if gb_type in latest_entry:
            existing_entry = latest_entry[gb_type]

            if not (
                call["context"] == existing_entry["Context"]
                and call["explanation"] == existing_entry["Explanation"]
                and sorted(call["hints"] or []) == sorted(existing_entry["Hints"] or [])
            ):
                needs_update = True
                break
        else:
            needs_update = True
            break
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 353-363
```python
    if needs_update:
        updated_registry = _update_registry_with_changes(
            registry, remaining_calls, renames
        )

        original_content = registry_path.read_text(encoding="utf-8")

        replacement_content = (
            json.dumps(updated_registry, indent=2, ensure_ascii=False) + "\n"
        )
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 364-372
```python
        changes = []
        if renames:
            for old, new in renames.items():
                changes.append(f"renamed '{old}' → '{new}'")
        if remaining_calls:
            new_count = sum(
                1 for gb_type in remaining_calls if gb_type not in latest_entry
            )
            if new_count:
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 373-390
```python
                changes.append(f"added {new_count} new gb_types")

        description = f"Registry sync needed ({', '.join(changes)}). Run `lintrunner -a` to apply changes."

        lint_messages.append(
            LintMessage(
                path=str(registry_path),
                line=None,
                char=None,
                code=LINTER_CODE,
                severity=LintSeverity.WARNING,
                name="Registry sync needed",
                original=original_content,
                replacement=replacement_content,
                description=description,
            )
        )
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。

### Lines 391-400
```python
    return lint_messages


if __name__ == "__main__":
    script_dir = Path(__file__).resolve()
    repo_root = script_dir.parents[3]
    default_registry_path = (
        repo_root / "torch" / "_dynamo" / "graph_break_registry.json"
    )
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 401-418
```python
    default_dynamo_dir = repo_root / "torch" / "_dynamo"

    parser = argparse.ArgumentParser(
        description="Auto-sync graph break registry with source code"
    )
    parser.add_argument(
        "--dynamo-dir",
        type=Path,
        default=default_dynamo_dir,
        help=f"Path to the dynamo directory (default: {default_dynamo_dir})",
    )
    parser.add_argument(
        "--registry-path",
        type=Path,
        default=default_registry_path,
        help=f"Path to the registry file (default: {default_registry_path})",
    )
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 419-426
```python
    args = parser.parse_args()

    lint_messages = check_registry_sync(
        dynamo_dir=args.dynamo_dir, registry_path=args.registry_path
    )

    for lint_message in lint_messages:
        print(json.dumps(lint_message._asdict()), flush=True)
```
- **EN**: This chunk continues `check_registry_sync` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `check_registry_sync`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

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

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.dynamo.gb_id_mapping`
- **Python standard library / Python 标准库**: `__future__`, `argparse`, `ast`, `functools`, `json`, `random`, `sys`, `enum`, `pathlib`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `LINTER_CODE`, `LintSeverity`, `LintMessage`, `_is_noqa_suppressed`, `_is_forbidden_raise`, `_collect_forbidden_unsupported_raises`, `_collect_all_calls`, `_create_registry_entry`, `_update_registry_with_changes`, `check_registry_sync`
