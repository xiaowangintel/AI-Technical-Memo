# executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/reorderer/executor.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on tensor reorder planning and execution. It mainly applies previously prepared plans to tensors, records, or intermediate debug objects. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于张量重排规划与执行。它主要用于将预先生成的计划应用到张量、记录或中间调试对象上。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies and shared types / 导入依赖与共享类型
```python
from typing import Optional

import torch

from sglang.srt.debug_utils.comparator.aligner.reorderer.types import (
    ReordererPlan,
    ZigzagToNaturalParams,
    ZigzagToNaturalThdParams,
)
from sglang.srt.debug_utils.comparator.dims_spec import (
    resolve_dim_by_name,
    strip_dim_names,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 16-39: Implement function `execute_reorderer_plan` / 实现函数 `execute_reorderer_plan`
```python
def execute_reorderer_plan(
    plan: ReordererPlan,
    tensors: list[torch.Tensor],
) -> list[torch.Tensor]:
    if isinstance(plan.params, ZigzagToNaturalThdParams):
        thd_dim: int = resolve_dim_by_name(tensors[0], plan.params.dim_name)
        return [
            _reorder_zigzag_to_natural_thd(
                tensor,
                dim=thd_dim,
                cp_size=plan.params.cp_size,
                seq_lens=plan.params.seq_lens,
            )
            for tensor in tensors
        ]

    if isinstance(plan.params, ZigzagToNaturalParams):
        dim: int = resolve_dim_by_name(tensors[0], plan.params.dim_name)
        return [
            _reorder_zigzag_to_natural(tensor, dim=dim, cp_size=plan.params.cp_size)
            for tensor in tensors
        ]

    raise ValueError(f"Unsupported reorderer params type: {type(plan.params).__name__}")
```
**EN:** Function `execute_reorderer_plan` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `execute_reorderer_plan` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 42-78: Implement helper `_reorder_zigzag_to_natural_thd` / 实现辅助函数 `_reorder_zigzag_to_natural_thd`
```python
def _reorder_zigzag_to_natural_thd(
    tensor: torch.Tensor, *, dim: int, cp_size: int, seq_lens: list[int]
) -> torch.Tensor:
    """Undo CP zigzag interleaving for THD (packed-seq) format.

    Each seq in seq_lens is independently reordered from zigzag to natural order
    along the given dim.
    """
    stripped: torch.Tensor = strip_dim_names(tensor)
    names: tuple[Optional[str], ...] = tensor.names

    split_sizes: list[int] = list(seq_lens)
    remainder: int = stripped.shape[dim] - sum(split_sizes)
    if remainder < 0:
        raise ValueError(
            f"sum(seq_lens)={sum(split_sizes)} exceeds tensor dim size "
            f"{stripped.shape[dim]} along dim={dim}"
        )
    if remainder > 0:
        split_sizes.append(remainder)

    segments: list[torch.Tensor] = list(stripped.split(split_sizes, dim=dim))

    reordered_segments: list[torch.Tensor] = [
        _reorder_zigzag_to_natural(seg, dim=dim, cp_size=cp_size)
        for seg in segments[: len(seq_lens)]
    ]

    # Tail padding — pass through unchanged
    if remainder > 0:
        reordered_segments.append(segments[-1])

    result: torch.Tensor = torch.cat(reordered_segments, dim=dim)

    if names[0] is not None:
        result = result.refine_names(*names)
    return result
```
**EN:** Function `_reorder_zigzag_to_natural_thd` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_reorder_zigzag_to_natural_thd` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 81-101: Implement helper `_reorder_zigzag_to_natural` / 实现辅助函数 `_reorder_zigzag_to_natural`
```python
def _reorder_zigzag_to_natural(
    tensor: torch.Tensor, *, dim: int, cp_size: int
) -> torch.Tensor:
    """Undo CP zigzag interleaving, restoring natural chunk order.

    Generalized from Megatron-LM _undo_attention_load_balancing
    (megatron/core/ssm/mamba_context_parallel.py:360-373).
    """
    stripped: torch.Tensor = strip_dim_names(tensor)
    names: tuple[Optional[str], ...] = tensor.names

    num_chunks: int = cp_size * 2
    chunks: tuple[torch.Tensor, ...] = stripped.chunk(num_chunks, dim=dim)
    order: list[int] = [2 * i for i in range(cp_size)] + [
        num_chunks - 2 * i - 1 for i in range(cp_size)
    ]
    result: torch.Tensor = torch.cat([chunks[i] for i in order], dim=dim)

    if names[0] is not None:
        result = result.refine_names(*names)
    return result
```
**EN:** Function `_reorder_zigzag_to_natural` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_reorder_zigzag_to_natural` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `execute_reorderer_plan`, `_reorder_zigzag_to_natural_thd`, `_reorder_zigzag_to_natural`
- **Module role / 模块角色**: Tensor reorder planning and execution / 张量重排规划与执行
- **Implementation focus / 实现重点**: Applies previously prepared plans to tensors, records, or intermediate debug objects / 将预先生成的计划应用到张量、记录或中间调试对象上

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: `torch`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.reorderer.types`, `sglang.srt.debug_utils.comparator.dims_spec`
