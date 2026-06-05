# code_patcher.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/source_patcher/code_patcher.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on source-code instrumentation and patching. It mainly rewrites source text safely so extra debugging hooks can be inserted. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于源码插桩与补丁处理。它主要用于安全地重写源码文本，以便插入额外的调试钩子。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Import dependencies and shared types / 导入依赖与共享类型
```python
import __future__

import importlib
import inspect
import textwrap
import types
from collections.abc import Callable
from typing import Any, Optional

import yaml

from sglang.srt.debug_utils.source_patcher.source_editor import apply_edits
from sglang.srt.debug_utils.source_patcher.types import (
    EditSpec,
    PatchConfig,
    PatchSpec,
    PatchState,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 21-40: Implement function `apply_patches_from_config` / 实现函数 `apply_patches_from_config`
```python
def apply_patches_from_config(
    yaml_content: str,
    *,
    extra_imports: Optional[list[str]] = None,
) -> list[PatchState]:
    """Parse a YAML config string and apply all patches.

    Args:
        yaml_content: YAML string with patch specifications.
        extra_imports: Import lines inserted once at the top of each patched
            function body (e.g. ["from pkg import foo"]).  The caller (dumper)
            uses this so users don't have to write boilerplate in YAML.
    """
    raw: dict[str, Any] = yaml.safe_load(yaml_content)
    config: PatchConfig = PatchConfig(**raw)

    if extra_imports:
        config = _inject_preamble(config=config, extra_imports=extra_imports)

    return _apply_specs(config.patches)
```
**EN:** Function `apply_patches_from_config` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `apply_patches_from_config` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 43-43: Define class `CodePatcher` and class context / 定义类 `CodePatcher`及类上下文
```python
class CodePatcher:
```
**EN:** This section introduces `CodePatcher`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `CodePatcher`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 44-44: Document the module intent / 说明模块意图
```python
    """Context manager that patches functions on enter and restores on exit."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 46-48: Implement method `__init__` for `CodePatcher` / 为 `CodePatcher` 实现方法 `__init__`
```python
    def __init__(self, *, patches: list[PatchSpec]) -> None:
        self._patches = patches
        self._states: list[PatchState] = []
```
**EN:** Method `__init__` implements behavior on `CodePatcher`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `CodePatcher` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 50-52: Implement method `__enter__` for `CodePatcher` / 为 `CodePatcher` 实现方法 `__enter__`
```python
    def __enter__(self) -> "CodePatcher":
        self._states = _apply_specs(self._patches)
        return self
```
**EN:** Method `__enter__` implements behavior on `CodePatcher`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__enter__` 为 `CodePatcher` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 54-62: Implement method `__exit__` for `CodePatcher` / 为 `CodePatcher` 实现方法 `__exit__`
```python
    def __exit__(
        self,
        exc_type: Optional[type],
        exc_val: Optional[BaseException],
        exc_tb: Optional[Any],
    ) -> None:
        for state in reversed(self._states):
            state.restore()
        self._states.clear()
```
**EN:** Method `__exit__` implements behavior on `CodePatcher`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__exit__` 为 `CodePatcher` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 65-102: Implement function `patch_function` / 实现函数 `patch_function`
```python
def patch_function(
    *,
    target: Callable[..., Any],
    edits: list[EditSpec],
    preamble: str = "",
) -> PatchState:
    """Patch a function by modifying its source and replacing __code__.

    1. inspect.getsource -> get original source
    2. apply_edits -> modify source text
    3. optionally prepend preamble (e.g. import lines) inside the function body
    4. compile + exec -> get new code object
    5. replace target.__code__

    Returns PatchState that can restore the original code.
    """
    original_code: types.CodeType = target.__code__

    source: str = inspect.getsource(target)
    modified_source: str = apply_edits(source=source, edits=edits)
    modified_source = textwrap.dedent(modified_source)

    if preamble.strip():
        modified_source = _insert_preamble(source=modified_source, preamble=preamble)

    code: types.CodeType = compile(
        modified_source,
        inspect.getfile(target),
        "exec",
        flags=__future__.annotations.compiler_flag,
    )
    temp_namespace: dict[str, Any] = {}
    exec(code, target.__globals__, temp_namespace)

    new_fn: Any = temp_namespace[target.__name__]
    target.__code__ = new_fn.__code__

    return PatchState(target_fn=target, original_code=original_code)
```
**EN:** Function `patch_function` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `patch_function` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 108-117: Implement helper `_apply_specs` / 实现辅助函数 `_apply_specs`
```python
def _apply_specs(specs: list[PatchSpec]) -> list[PatchState]:
    states: list[PatchState] = []
    for spec in specs:
        target_fn: Callable[..., Any] = _resolve_target(spec.target)
        print(f"[source_patcher] patching {spec.target}")
        state: PatchState = patch_function(
            target=target_fn, edits=spec.edits, preamble=spec.preamble
        )
        states.append(state)
    return states
```
**EN:** Function `_apply_specs` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_apply_specs` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 120-134: Implement helper `_inject_preamble` / 实现辅助函数 `_inject_preamble`
```python
def _inject_preamble(*, config: PatchConfig, extra_imports: list[str]) -> PatchConfig:
    """Set preamble on every PatchSpec so imports are inserted once at function top."""
    import_block: str = "\n".join(extra_imports)
    new_patches: list[PatchSpec] = []

    for spec in config.patches:
        existing: str = spec.preamble
        combined: str = (
            import_block + "\n" + existing if existing.strip() else import_block
        )
        new_patches.append(
            PatchSpec(target=spec.target, edits=spec.edits, preamble=combined)
        )

    return PatchConfig(patches=new_patches)
```
**EN:** Function `_inject_preamble` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_inject_preamble` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 137-154: Implement helper `_insert_preamble` / 实现辅助函数 `_insert_preamble`
```python
def _insert_preamble(*, source: str, preamble: str) -> str:
    """Insert preamble lines right after the function signature (and optional docstring)."""
    lines: list[str] = source.splitlines()

    signature_end: int = _find_signature_end(lines)

    body_start: int = signature_end + 1
    body_indent: str = ""
    for i in range(body_start, len(lines)):
        if lines[i].strip():
            body_indent = " " * (len(lines[i]) - len(lines[i].lstrip()))
            body_start = i
            break

    preamble_lines: list[str] = [
        body_indent + pl for pl in preamble.strip().splitlines()
    ]
    return "\n".join(lines[:body_start] + preamble_lines + lines[body_start:])
```
**EN:** Function `_insert_preamble` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_insert_preamble` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 157-162: Implement helper `_find_signature_end` / 实现辅助函数 `_find_signature_end`
```python
def _find_signature_end(lines: list[str]) -> int:
    """Find the line index where the function signature ends (the line with trailing colon)."""
    for i, line in enumerate(lines):
        if line.rstrip().endswith(":"):
            return i
    return 0
```
**EN:** Function `_find_signature_end` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_find_signature_end` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 165-195: Implement helper `_resolve_target` / 实现辅助函数 `_resolve_target`
```python
def _resolve_target(qualified_name: str) -> Callable[..., Any]:
    """Resolve 'pkg.mod.Class.method' to the actual function object.

    Tries progressively shorter module paths from right to left,
    then uses getattr for the remaining attribute chain.
    """
    parts: list[str] = qualified_name.split(".")

    target: Any = None
    for split_idx in range(len(parts), 0, -1):
        module_path: str = ".".join(parts[:split_idx])
        try:
            target = importlib.import_module(module_path)
            attr_parts: list[str] = parts[split_idx:]
            break
        except ImportError:
            continue
    else:
        raise ImportError(f"could not import any module prefix of '{qualified_name}'")

    for attr_name in attr_parts:
        target = getattr(target, attr_name)

    if isinstance(target, classmethod):
        target = target.__func__
    if not callable(target):
        raise TypeError(
            f"resolved target '{qualified_name}' is not callable: {type(target)}"
        )

    return target
```
**EN:** Function `_resolve_target` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_resolve_target` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `CodePatcher`, `apply_patches_from_config`, `patch_function`, `_apply_specs`, `_inject_preamble`, `_insert_preamble`, `_find_signature_end`, `_resolve_target`
- **Module role / 模块角色**: Source-code instrumentation and patching / 源码插桩与补丁处理
- **Implementation focus / 实现重点**: Rewrites source text safely so extra debugging hooks can be inserted / 安全地重写源码文本，以便插入额外的调试钩子

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `importlib`, `inspect`, `textwrap`, `types`, `collections`, `typing`
- **Third-party / 第三方**: `yaml`
- **Internal / 内部**: `sglang.srt.debug_utils.source_patcher.source_editor`, `sglang.srt.debug_utils.source_patcher.types`
