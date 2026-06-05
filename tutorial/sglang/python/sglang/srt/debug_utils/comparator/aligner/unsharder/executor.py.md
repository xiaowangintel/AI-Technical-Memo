# executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/unsharder/executor.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on unsharding and tensor-parallel merging. It mainly applies previously prepared plans to tensors, records, or intermediate debug objects. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于反分片与张量并行合并。它主要用于将预先生成的计划应用到张量、记录或中间调试对象上。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Import dependencies and shared types / 导入依赖与共享类型
```python
from dataclasses import dataclass, field
from typing import Optional

import torch

from sglang.srt.debug_utils.comparator.aligner.unsharder.types import (
    ConcatParams,
    CpThdConcatParams,
    PickParams,
    ReduceSumParams,
    UnsharderParams,
    UnsharderPlan,
)
from sglang.srt.debug_utils.comparator.dims_spec import (
    ParallelAxis,
    resolve_dim_by_name,
)
from sglang.srt.debug_utils.comparator.output_types import ReplicatedCheckResult
from sglang.srt.debug_utils.comparator.tensor_comparator.comparator import compute_diff
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 21-21: Declare module-level symbols such as `_REPLICATED_ATOL` / 声明模块级符号，例如 `_REPLICATED_ATOL`
```python
_REPLICATED_ATOL: float = 1e-6
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 25-25: Define class `UnsharderResult` and class context / 定义类 `UnsharderResult`及类上下文
```python
class UnsharderResult:
```
**EN:** This section introduces `UnsharderResult`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `UnsharderResult`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 26-27: Declare fields for `UnsharderResult` such as `tensors`, `replicated_checks` / 为 `UnsharderResult` 声明字段，例如 `tensors`, `replicated_checks`
```python
    tensors: list[torch.Tensor]
    replicated_checks: list[ReplicatedCheckResult] = field(default_factory=list)
```
**EN:** These lines declare the state carried by `UnsharderResult`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `UnsharderResult` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 30-48: Implement function `execute_unsharder_plan` / 实现函数 `execute_unsharder_plan`
```python
def execute_unsharder_plan(
    plan: UnsharderPlan,
    tensors: list[torch.Tensor],
) -> UnsharderResult:
    result_tensors: list[torch.Tensor] = []
    all_checks: list[ReplicatedCheckResult] = []

    for group_idx, group in enumerate(plan.groups):
        group_tensors = [tensors[i] for i in group]
        tensor, checks = _apply_unshard(
            plan.params,
            group_tensors,
            axis=plan.axis,
            group_index=group_idx,
        )
        result_tensors.append(tensor)
        all_checks.extend(checks)

    return UnsharderResult(tensors=result_tensors, replicated_checks=all_checks)
```
**EN:** Function `execute_unsharder_plan` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `execute_unsharder_plan` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 51-89: Implement helper `_apply_unshard` / 实现辅助函数 `_apply_unshard`
```python
def _apply_unshard(
    params: UnsharderParams,
    ordered_tensors: list[torch.Tensor],
    *,
    axis: ParallelAxis,
    group_index: int,
) -> tuple[torch.Tensor, list[ReplicatedCheckResult]]:
    if isinstance(params, PickParams):
        checks: list[ReplicatedCheckResult] = _verify_replicated_group(
            ordered_tensors,
            axis=axis,
            group_index=group_index,
        )
        return ordered_tensors[0], checks

    if isinstance(params, ConcatParams):
        dim: int = resolve_dim_by_name(ordered_tensors[0], params.dim_name)
        return torch.cat(ordered_tensors, dim=dim), []

    if isinstance(params, CpThdConcatParams):
        thd_dim: int = resolve_dim_by_name(ordered_tensors[0], params.dim_name)
        return (
            _thd_concat(
                ordered_tensors,
                dim=thd_dim,
                seq_lens_per_rank=params.seq_lens_per_rank,
            ),
            [],
        )

    if isinstance(params, ReduceSumParams):
        stripped: list[torch.Tensor] = [t.rename(None) for t in ordered_tensors]
        result: torch.Tensor = torch.stack(stripped).sum(dim=0)
        names: tuple[Optional[str], ...] = ordered_tensors[0].names
        if names[0] is not None:
            result = result.refine_names(*names)
        return result, []

    raise ValueError(f"Unsupported unshard operation: {type(params).__name__}")
```
**EN:** Function `_apply_unshard` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_apply_unshard` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 92-109: Implement helper `_verify_replicated_group` / 实现辅助函数 `_verify_replicated_group`
```python
def _verify_replicated_group(
    ordered_tensors: list[torch.Tensor],
    *,
    axis: ParallelAxis,
    group_index: int,
) -> list[ReplicatedCheckResult]:
    baseline: torch.Tensor = ordered_tensors[0].rename(None).float()

    return [
        _check_replicated_pair(
            baseline=baseline,
            other=ordered_tensors[i],
            axis=axis,
            group_index=group_index,
            compared_index=i,
        )
        for i in range(1, len(ordered_tensors))
    ]
```
**EN:** Function `_verify_replicated_group` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_verify_replicated_group` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 112-141: Implement helper `_check_replicated_pair` / 实现辅助函数 `_check_replicated_pair`
```python
def _check_replicated_pair(
    *,
    baseline: torch.Tensor,
    other: torch.Tensor,
    axis: ParallelAxis,
    group_index: int,
    compared_index: int,
) -> ReplicatedCheckResult:
    other_float: torch.Tensor = other.rename(None).float()

    if baseline.shape != other_float.shape:
        passed = False
        diff_info = None
    else:
        diff_info = compute_diff(
            x_baseline=baseline,
            x_target=other_float,
            diff_threshold=_REPLICATED_ATOL,
        )
        passed = diff_info.max_abs_diff <= _REPLICATED_ATOL

    return ReplicatedCheckResult(
        axis=axis.value,
        group_index=group_index,
        compared_index=compared_index,
        baseline_index=0,
        passed=passed,
        atol=_REPLICATED_ATOL,
        diff=diff_info,
    )
```
**EN:** Function `_check_replicated_pair` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_check_replicated_pair` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 144-183: Implement helper `_thd_concat` / 实现辅助函数 `_thd_concat`
```python
def _thd_concat(
    ordered_tensors: list[torch.Tensor],
    *,
    dim: int,
    seq_lens_per_rank: list[int],
) -> torch.Tensor:
    """Per-seq concat across ranks for THD format.

    Each rank holds segments of each seq packed contiguously:
      rank_data = [seq0_tokens | seq1_tokens | ... | pad_tokens]

    This function splits each rank by seq_lens, then interleaves across ranks
    per-seq: [seqA_r0 + seqA_r1 + ... | seqB_r0 + seqB_r1 + ... | tail_pad].
    """
    names: tuple[Optional[str], ...] = ordered_tensors[0].names
    stripped: list[torch.Tensor] = [t.rename(None) for t in ordered_tensors]

    # Split each rank into [seq0, seq1, ..., tail_remainder]
    split_sizes: list[int] = list(seq_lens_per_rank)
    remainder: int = stripped[0].shape[dim] - sum(split_sizes)
    if remainder < 0:
        raise ValueError(
            f"sum(seq_lens_per_rank)={sum(split_sizes)} exceeds tensor dim size "
            f"{stripped[0].shape[dim]} along dim={dim}"
        )
    if remainder > 0:
        split_sizes.append(remainder)
    per_rank_splits: list[tuple[torch.Tensor, ...]] = [
        t.split(split_sizes, dim=dim) for t in stripped
    ]

    # Per-seq concat across ranks, then concatenate all seqs
    result: torch.Tensor = torch.cat(
        [torch.cat(rank_parts, dim=dim) for rank_parts in zip(*per_rank_splits)],
        dim=dim,
    )

    if names[0] is not None:
        result = result.refine_names(*names)
    return result
```
**EN:** Function `_thd_concat` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_thd_concat` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `UnsharderResult`, `execute_unsharder_plan`, `_apply_unshard`, `_verify_replicated_group`, `_check_replicated_pair`, `_thd_concat`
- **Module role / 模块角色**: Unsharding and tensor-parallel merging / 反分片与张量并行合并
- **Implementation focus / 实现重点**: Applies previously prepared plans to tensors, records, or intermediate debug objects / 将预先生成的计划应用到张量、记录或中间调试对象上

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.tensor_comparator.comparator`
