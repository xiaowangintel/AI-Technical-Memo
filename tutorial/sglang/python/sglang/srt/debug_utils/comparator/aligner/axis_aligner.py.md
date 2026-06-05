# axis_aligner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/axis_aligner.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on tensor axis normalization. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于张量轴归一化。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from typing import Optional

import torch
from einops import rearrange

from sglang.srt.debug_utils.comparator.dims_spec import (
    _FUSED_NAME_SEP,
    SEQ_DIM_NAME,
    TOKEN_DIM_NAME,
    DimSpec,
    _SingletonDimUtil,
    parse_dims,
)
from sglang.srt.debug_utils.comparator.log_sink import log_sink
from sglang.srt.debug_utils.comparator.utils import Pair, _FrozenBase
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 22-22: Define class `AxisAlignerPlan` and class context / 定义类 `AxisAlignerPlan`及类上下文
```python
class AxisAlignerPlan(_FrozenBase):
```
**EN:** This section introduces `AxisAlignerPlan`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `AxisAlignerPlan`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 23-23: Declare fields for `AxisAlignerPlan` such as `pattern` / 为 `AxisAlignerPlan` 声明字段，例如 `pattern`
```python
    pattern: Pair[Optional[str]]  # einops pattern per side, None = no-op
```
**EN:** These lines declare the state carried by `AxisAlignerPlan`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `AxisAlignerPlan` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 29-53: Implement function `compute_axis_aligner_plan` / 实现函数 `compute_axis_aligner_plan`
```python
def compute_axis_aligner_plan(
    dims_str_pair: Pair[Optional[str]],
) -> Optional[AxisAlignerPlan]:
    if dims_str_pair.x is None or dims_str_pair.y is None:
        return None

    dims_pair: Pair[str] = Pair(x=dims_str_pair.x, y=dims_str_pair.y)
    specs_pair: Pair[list[DimSpec]] = dims_pair.map(lambda s: parse_dims(s).dims)

    if not _semantic_names_match(specs_pair):
        return None

    # Canonical dim order follows y; fused groups stay fused (flatten, not unflatten).
    canonical_order: Optional[list[str]] = _build_canonical_order(specs_pair)
    if canonical_order is None:
        return None

    pattern: Pair[Optional[str]] = specs_pair.map(
        lambda specs: _build_side_pattern(specs=specs, canonical_order=canonical_order)
    )

    if pattern.x is None and pattern.y is None:
        return None

    return AxisAlignerPlan(pattern=pattern)
```
**EN:** Function `compute_axis_aligner_plan` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `compute_axis_aligner_plan` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 56-60: Declare module-level symbols such as `_SEQ_DIM_EQUIVALENCES` / 声明模块级符号，例如 `_SEQ_DIM_EQUIVALENCES`
```python
_SEQ_DIM_EQUIVALENCES: frozenset[frozenset[str]] = frozenset(
    {
        frozenset({SEQ_DIM_NAME, TOKEN_DIM_NAME}),  # s ≡ t
    }
)
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 63-67: Implement helper `_normalize_dim_name` / 实现辅助函数 `_normalize_dim_name`
```python
def _normalize_dim_name(name: str) -> str:
    for equiv_set in _SEQ_DIM_EQUIVALENCES:
        if name in equiv_set:
            return min(equiv_set)
    return name
```
**EN:** Function `_normalize_dim_name` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_normalize_dim_name` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 70-92: Implement helper `_semantic_names_match` / 实现辅助函数 `_semantic_names_match`
```python
def _semantic_names_match(specs_pair: Pair[list[DimSpec]]) -> bool:
    """Check that both sides share the same semantic name set (ignoring squeeze dims)."""
    names_pair: Pair[list[str]] = specs_pair.map(_expand_and_skip_squeeze)

    if set(map(_normalize_dim_name, names_pair.x)) == set(
        map(_normalize_dim_name, names_pair.y)
    ):
        return True

    # Local import to avoid circular dependency:
    # output_types -> aligner/entrypoint/types -> axis_aligner -> output_types
    from sglang.srt.debug_utils.comparator.output_types import ErrorLog

    log_sink.add(
        ErrorLog(
            category="axis_aligner_dim_mismatch",
            message=(
                f"AxisAligner: dim name sets differ (x={names_pair.x}, y={names_pair.y}), "
                f"skipping axis swap"
            ),
        )
    )
    return False
```
**EN:** Function `_semantic_names_match` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_semantic_names_match` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 95-102: Implement helper `_expand_and_skip_squeeze` / 实现辅助函数 `_expand_and_skip_squeeze`
```python
def _expand_and_skip_squeeze(specs: list[DimSpec]) -> list[str]:
    """Expand DimSpecs to flat semantic names, skipping squeeze dims."""
    return [
        name
        for spec in specs
        if not _SingletonDimUtil.is_squeeze(spec)
        for name in spec.sub_dims
    ]
```
**EN:** Function `_expand_and_skip_squeeze` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_expand_and_skip_squeeze` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 105-160: Implement helper `_build_canonical_order` / 实现辅助函数 `_build_canonical_order`
```python
def _build_canonical_order(specs_pair: Pair[list[DimSpec]]) -> Optional[list[str]]:
    """Build canonical dim order following y, preferring fused representation.

    Each element is either a plain name (``"c"``) or a fused placeholder (``"a___b"``).
    Fused groups from *either* side are merged — the separate side must flatten.
    Squeeze dims are excluded.

    Returns ``None`` if the two sides have overlapping but incompatible fused groups
    (e.g. x fuses ``(a*b)`` while y fuses ``(b*c)``).
    """
    # Map each sub-dim name → (placeholder, siblings) from both sides
    fused_lookup: dict[str, tuple[str, frozenset[str]]] = {}
    for spec in (*specs_pair.x, *specs_pair.y):
        if spec.is_fused:
            placeholder: str = spec.sanitized_name
            siblings: frozenset[str] = frozenset(spec.sub_dims)
            for sub_name in spec.sub_dims:
                existing: Optional[tuple[str, frozenset[str]]] = fused_lookup.get(
                    sub_name
                )
                if existing is not None and existing[1] != siblings:
                    from sglang.srt.debug_utils.comparator.output_types import ErrorLog

                    log_sink.add(
                        ErrorLog(
                            category="axis_aligner_fused_conflict",
                            message=(
                                f"AxisAligner: overlapping fused groups for sub-dim {sub_name!r} "
                                f"({existing[0]} vs {placeholder}), skipping axis alignment"
                            ),
                        )
                    )
                    return None
                fused_lookup.setdefault(sub_name, (placeholder, siblings))

    result: list[str] = []
    consumed: set[str] = set()

    for spec in specs_pair.y:
        if _SingletonDimUtil.is_squeeze(spec):
            continue

        names: list[str] = spec.sub_dims
        if any(n in consumed for n in names):
            continue

        entry: Optional[tuple[str, frozenset[str]]] = fused_lookup.get(names[0])
        if entry is not None:
            fused_placeholder, sibs = entry
            result.append(fused_placeholder)
            consumed.update(sibs)
        else:
            result.append(_normalize_dim_name(spec.name))
            consumed.update(names)

    return result
```
**EN:** Function `_build_canonical_order` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_build_canonical_order` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 163-201: Implement helper `_build_side_pattern` / 实现辅助函数 `_build_side_pattern`
```python
def _build_side_pattern(
    *, specs: list[DimSpec], canonical_order: list[str]
) -> Optional[str]:
    """Build an einops pattern for one side to reach ``canonical_order``.

    Fused specs become their placeholder; separate specs that belong to a fused group
    stay as individual names on the LHS and become ``(a b)`` on the RHS (einops flatten).
    Squeeze dims (``1``) appear on the LHS but are dropped from the RHS.
    """
    source_tokens: list[str] = [spec.sanitized_name for spec in specs]

    # Map normalized dim names back to this side's original names so that
    # einops patterns use consistent identifiers on LHS and RHS.
    norm_to_original: dict[str, str] = {
        _normalize_dim_name(spec.name): spec.name for spec in specs
    }

    def _to_side_name(token: str) -> str:
        return norm_to_original.get(token, token)

    # Build per-side target: replace fused placeholders with ``(a b)`` only if this side
    # has the sub-dims as separate (non-fused) names in the source
    fused_placeholders: set[str] = {
        spec.sanitized_name for spec in specs if spec.is_fused
    }
    translated_order: list[str] = [_to_side_name(t) for t in canonical_order]
    target_tokens: list[str] = [
        (
            f"({t.replace(_FUSED_NAME_SEP, ' ')})"
            if _FUSED_NAME_SEP in t and t not in fused_placeholders
            else t
        )
        for t in translated_order
    ]

    if source_tokens == target_tokens:
        return None

    return f"{' '.join(source_tokens)} -> {' '.join(target_tokens)}"
```
**EN:** Function `_build_side_pattern` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_build_side_pattern` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 207-218: Implement function `execute_axis_aligner_plan` / 实现函数 `execute_axis_aligner_plan`
```python
def execute_axis_aligner_plan(
    tensor: torch.Tensor, plan: AxisAlignerPlan, *, side: str
) -> torch.Tensor:
    if side not in ("x", "y"):
        raise ValueError(f"side must be 'x' or 'y', got {side!r}")

    pattern: Optional[str] = plan.pattern.x if side == "x" else plan.pattern.y

    if pattern is not None:
        tensor = rearrange(tensor.rename(None), pattern)

    return tensor
```
**EN:** Function `execute_axis_aligner_plan` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `execute_axis_aligner_plan` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `AxisAlignerPlan`, `compute_axis_aligner_plan`, `_normalize_dim_name`, `_semantic_names_match`, `_expand_and_skip_squeeze`, `_build_canonical_order`, `_build_side_pattern`, `execute_axis_aligner_plan`
- **Module role / 模块角色**: Tensor axis normalization / 张量轴归一化
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`
- **Third-party / 第三方**: `torch`, `einops`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.log_sink`, `sglang.srt.debug_utils.comparator.utils`, `sglang.srt.debug_utils.comparator.output_types`
