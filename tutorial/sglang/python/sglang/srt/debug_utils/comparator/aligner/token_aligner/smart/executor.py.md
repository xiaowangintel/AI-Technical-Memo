# executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/token_aligner/smart/executor.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on heuristic token alignment. It mainly applies previously prepared plans to tensors, records, or intermediate debug objects. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于启发式 token 对齐。它主要用于将预先生成的计划应用到张量、记录或中间调试对象上。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

import torch
from einops import rearrange

from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    TokenAlignerPlan,
    TokenLocator,
)
from sglang.srt.debug_utils.comparator.dims_spec import (
    BATCH_DIM_NAME,
    SEQ_DIM_NAME,
    TOKEN_DIM_NAME,
    TokenLayout,
    resolve_dim_by_name,
    strip_dim_names,
)
from sglang.srt.debug_utils.comparator.utils import Pair
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 20-20: Declare module-level symbols such as `_UNNAMED_TOKEN_DIM_FALLBACK` / 声明模块级符号，例如 `_UNNAMED_TOKEN_DIM_FALLBACK`
```python
_UNNAMED_TOKEN_DIM_FALLBACK: int = 0
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 23-49: Implement function `execute_token_aligner` / 实现函数 `execute_token_aligner`
```python
def execute_token_aligner(
    plan: TokenAlignerPlan,
    tensor_of_step_pair: Pair[dict[int, torch.Tensor]],
) -> Pair[torch.Tensor]:
    flat_pair: Pair[dict[int, torch.Tensor]] = Pair(
        x=_collapse_bs_to_t(
            tensor_of_step=tensor_of_step_pair.x, layout=plan.layouts.x
        ),
        y=_collapse_bs_to_t(
            tensor_of_step=tensor_of_step_pair.y, layout=plan.layouts.y
        ),
    )

    if not plan.locators.x.steps:
        return Pair(
            x=_make_empty(tensor_of_step=flat_pair.x),
            y=_make_empty(tensor_of_step=flat_pair.y),
        )

    return Pair(
        x=_extract_and_stack_tokens(
            tensor_of_step=flat_pair.x, locator=plan.locators.x
        ),
        y=_extract_and_stack_tokens(
            tensor_of_step=flat_pair.y, locator=plan.locators.y
        ),
    )
```
**EN:** Function `execute_token_aligner` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `execute_token_aligner` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 55-91: Implement helper `_collapse_bs_to_t` / 实现辅助函数 `_collapse_bs_to_t`
```python
def _collapse_bs_to_t(
    *,
    tensor_of_step: dict[int, torch.Tensor],
    layout: TokenLayout,
) -> dict[int, torch.Tensor]:
    """Collapse B and S dims into a single flat token dim (always batch-major).

    Handles both ``b s`` and ``s b`` orderings correctly via einops rearrange.
    Returns the original tensors unchanged if layout is T.
    """
    if layout != TokenLayout.BS:
        return tensor_of_step

    some_tensor: torch.Tensor = next(iter(tensor_of_step.values()))
    batch_dim: int = _resolve_dim_or_fallback(some_tensor, BATCH_DIM_NAME)
    seq_dim: int = _resolve_dim_or_fallback(some_tensor, SEQ_DIM_NAME)

    if abs(batch_dim - seq_dim) != 1:
        raise ValueError(
            f"BS dims must be adjacent: "
            f"{BATCH_DIM_NAME}={batch_dim}, "
            f"{SEQ_DIM_NAME}={seq_dim}"
        )

    lhs_pattern, rhs_pattern, new_names = _build_bs_collapse_pattern(
        names=list(some_tensor.names),
        batch_dim=batch_dim,
        seq_dim=seq_dim,
    )

    result: dict[int, torch.Tensor] = {}
    for step, tensor in tensor_of_step.items():
        plain: torch.Tensor = strip_dim_names(tensor)
        collapsed: torch.Tensor = rearrange(plain, f"{lhs_pattern} -> {rhs_pattern}")
        result[step] = collapsed.refine_names(*new_names)

    return result
```
**EN:** Function `_collapse_bs_to_t` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_collapse_bs_to_t` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 94-117: Implement helper `_build_bs_collapse_pattern` / 实现辅助函数 `_build_bs_collapse_pattern`
```python
def _build_bs_collapse_pattern(
    *,
    names: list[str | None],
    batch_dim: int,
    seq_dim: int,
) -> tuple[str, str, list[str | None]]:
    """Build einops lhs/rhs patterns and output dim names for BS→T collapse.

    Always produces batch-major order ``(b s)`` regardless of input ordering.
    Uses the tensor's own dim names as einops axis names.
    """
    lo: int = min(batch_dim, seq_dim)
    hi: int = max(batch_dim, seq_dim)

    lhs: str = " ".join(names)  # type: ignore[arg-type]

    rhs_names: list[str] = list(names[:lo]) + [f"({BATCH_DIM_NAME} {SEQ_DIM_NAME})"] + list(names[hi + 1 :])  # type: ignore[misc]
    rhs: str = " ".join(rhs_names)

    new_names: list[str | None] = (
        list(names[:lo]) + [TOKEN_DIM_NAME] + list(names[hi + 1 :])
    )

    return lhs, rhs, new_names
```
**EN:** Function `_build_bs_collapse_pattern` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_build_bs_collapse_pattern` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 123-126: Implement helper `_resolve_dim_or_fallback` / 实现辅助函数 `_resolve_dim_or_fallback`
```python
def _resolve_dim_or_fallback(tensor: torch.Tensor, name: str) -> int:
    if tensor.names[0] is None:
        return _UNNAMED_TOKEN_DIM_FALLBACK
    return resolve_dim_by_name(tensor, name)
```
**EN:** Function `_resolve_dim_or_fallback` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_resolve_dim_or_fallback` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 129-134: Implement helper `_make_empty` / 实现辅助函数 `_make_empty`
```python
def _make_empty(*, tensor_of_step: dict[int, torch.Tensor]) -> torch.Tensor:
    dummy: torch.Tensor = next(iter(tensor_of_step.values()))
    token_dim: int = _resolve_dim_or_fallback(dummy, TOKEN_DIM_NAME)
    shape: list[int] = list(dummy.shape)
    shape[token_dim] = 0
    return torch.empty(shape, dtype=dummy.dtype)
```
**EN:** Function `_make_empty` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_make_empty` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 137-149: Implement helper `_extract_and_stack_tokens` / 实现辅助函数 `_extract_and_stack_tokens`
```python
def _extract_and_stack_tokens(
    *,
    tensor_of_step: dict[int, torch.Tensor],
    locator: TokenLocator,
) -> torch.Tensor:
    some_tensor: torch.Tensor = next(iter(tensor_of_step.values()))
    token_dim: int = _resolve_dim_or_fallback(some_tensor, TOKEN_DIM_NAME)

    tokens: list[torch.Tensor] = [
        strip_dim_names(tensor_of_step[s]).select(dim=token_dim, index=i)
        for s, i in zip(locator.steps, locator.token_index_in_step)
    ]
    return torch.stack(tokens, dim=token_dim)
```
**EN:** Function `_extract_and_stack_tokens` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_extract_and_stack_tokens` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `execute_token_aligner`, `_collapse_bs_to_t`, `_build_bs_collapse_pattern`, `_resolve_dim_or_fallback`, `_make_empty`, `_extract_and_stack_tokens`
- **Module role / 模块角色**: Heuristic token alignment / 启发式 token 对齐
- **Implementation focus / 实现重点**: Applies previously prepared plans to tensors, records, or intermediate debug objects / 将预先生成的计划应用到张量、记录或中间调试对象上

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`
- **Third-party / 第三方**: `torch`, `einops`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.utils`
