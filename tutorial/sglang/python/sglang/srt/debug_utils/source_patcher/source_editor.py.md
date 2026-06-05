# source_editor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/source_patcher/source_editor.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on source-code instrumentation and patching. It mainly rewrites source text safely so extra debugging hooks can be inserted. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于源码插桩与补丁处理。它主要用于安全地重写源码文本，以便插入额外的调试钩子。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Import dependencies and shared types / 导入依赖与共享类型
```python
from sglang.srt.debug_utils.source_patcher.types import EditSpec, PatchApplicationError
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 4-12: Implement function `apply_edits` / 实现函数 `apply_edits`
```python
def apply_edits(*, source: str, edits: list[EditSpec]) -> str:
    """Apply a sequence of match/replacement edits to source text.

    Each edit is applied sequentially so later edits see the result of earlier ones.
    """
    result: str = source
    for edit in edits:
        result = _apply_single_edit(source=result, edit=edit)
    return result
```
**EN:** Function `apply_edits` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `apply_edits` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 15-41: Implement helper `_apply_single_edit` / 实现辅助函数 `_apply_single_edit`
```python
def _apply_single_edit(*, source: str, edit: EditSpec) -> str:
    """Apply a single match/replacement edit to the source text."""
    match_text: str = edit.match.strip()
    if not match_text:
        raise PatchApplicationError("empty match text")

    source_lines: list[str] = source.splitlines()
    match_lines: list[str] = match_text.splitlines()

    start_idx: int = _find_match(source_lines=source_lines, match_lines=match_lines)
    match_len: int = len(match_lines)

    original_indent: int = _leading_spaces(source_lines[start_idx])

    effective_replacement: str = _resolve_replacement(edit=edit, match_text=match_text)
    replacement_lines: list[str] = (
        effective_replacement.splitlines() if effective_replacement else []
    )
    aligned: list[str] = _realign_replacement(
        replacement_lines=replacement_lines, original_indent=original_indent
    )
    new_lines: list[str] = (
        source_lines[:start_idx] + aligned + source_lines[start_idx + match_len :]
    )

    trailing_newline: str = "\n" if source.endswith("\n") else ""
    return "\n".join(new_lines) + trailing_newline
```
**EN:** Function `_apply_single_edit` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_apply_single_edit` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 44-50: Implement helper `_resolve_replacement` / 实现辅助函数 `_resolve_replacement`
```python
def _resolve_replacement(*, edit: EditSpec, match_text: str) -> str:
    """Return the effective replacement text, handling replacement, prepend, and append modes."""
    if edit.prepend.strip():
        return edit.prepend.strip() + "\n" + match_text
    if edit.append.strip():
        return match_text + "\n" + edit.append.strip()
    return edit.replacement.strip()
```
**EN:** Function `_resolve_replacement` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_resolve_replacement` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 53-78: Implement helper `_find_match` / 实现辅助函数 `_find_match`
```python
def _find_match(*, source_lines: list[str], match_lines: list[str]) -> int:
    """Find the start index of match_lines in source_lines (strip-compared).

    Returns the index of the first matching line.
    Raises PatchApplicationError if not found or found multiple times.
    """
    stripped_source: list[str] = [line.strip() for line in source_lines]
    stripped_match: list[str] = [line.strip() for line in match_lines]
    match_len: int = len(stripped_match)

    found_indices: list[int] = [
        i
        for i in range(len(stripped_source) - match_len + 1)
        if stripped_source[i : i + match_len] == stripped_match
    ]

    if len(found_indices) == 0:
        preview: str = "\n".join(match_lines)
        raise PatchApplicationError(f"match text not found in source:\n{preview}")
    if len(found_indices) > 1:
        preview = "\n".join(match_lines)
        raise PatchApplicationError(
            f"match text found multiple times ({len(found_indices)} occurrences) in source:\n{preview}"
        )

    return found_indices[0]
```
**EN:** Function `_find_match` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_find_match` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 81-104: Implement helper `_realign_replacement` / 实现辅助函数 `_realign_replacement`
```python
def _realign_replacement(
    *, replacement_lines: list[str], original_indent: int
) -> list[str]:
    """Realign replacement lines to the original indentation level.

    Strategy:
    - Take the leading spaces of the first non-empty replacement line as base_indent
    - For each replacement line: remove base_indent, add original_indent
    """
    non_empty: list[str] = [line for line in replacement_lines if line.strip()]
    if not non_empty:
        return []

    base_indent: int = _leading_spaces(non_empty[0])
    result: list[str] = []

    for line in replacement_lines:
        if not line.strip():
            result.append("")
        else:
            stripped = line[min(base_indent, len(line) - len(line.lstrip())) :]
            result.append(" " * original_indent + stripped)

    return result
```
**EN:** Function `_realign_replacement` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_realign_replacement` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 107-108: Implement helper `_leading_spaces` / 实现辅助函数 `_leading_spaces`
```python
def _leading_spaces(line: str) -> int:
    return len(line) - len(line.lstrip(" "))
```
**EN:** Function `_leading_spaces` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_leading_spaces` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `apply_edits`, `_apply_single_edit`, `_resolve_replacement`, `_find_match`, `_realign_replacement`, `_leading_spaces`
- **Module role / 模块角色**: Source-code instrumentation and patching / 源码插桩与补丁处理
- **Implementation focus / 实现重点**: Rewrites source text safely so extra debugging hooks can be inserted / 安全地重写源码文本，以便插入额外的调试钩子

## Dependencies / 依赖关系
- **Standard library / 标准库**: None / 无
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.source_patcher.types`
