# planner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/entrypoint/planner.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on end-to-end alignment orchestration. It mainly computes reusable plans that later stages can execute deterministically. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于端到端对齐编排。它主要用于计算可复用的计划对象，供后续阶段确定性执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from typing import Any, Optional

from sglang.srt.debug_utils.comparator.aligner.axis_aligner import (
    AxisAlignerPlan,
    compute_axis_aligner_plan,
)
from sglang.srt.debug_utils.comparator.aligner.entrypoint.types import (
    AlignerPerStepPlan,
    AlignerPerStepSubPlan,
    AlignerPlan,
)
from sglang.srt.debug_utils.comparator.aligner.reorderer.planner import (
    compute_reorderer_plans,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    TokenAlignerPlan,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.parallel_info import (
    normalize_parallel_info,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.planner import (
    compute_unsharder_plan,
)
from sglang.srt.debug_utils.comparator.dims_spec import (
    DimSpec,
    DimsSpec,
    ParallelAxis,
    _SingletonDimUtil,
    parse_dims,
)
from sglang.srt.debug_utils.comparator.utils import Pair
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 36-66: Implement function `compute_aligner_plan` / 实现函数 `compute_aligner_plan`
```python
def compute_aligner_plan(
    *,
    metas_pair: Pair[list[dict[str, Any]]],
    token_aligner_mode: Optional[str],
    token_aligner_plan: Optional[TokenAlignerPlan],
    thd_seq_lens_by_step_pair: Pair[Optional[dict[int, list[int]]]] = Pair(
        x=None, y=None
    ),
) -> AlignerPlan:
    dims_str_pair: Pair[Optional[str]] = metas_pair.map(
        lambda metas: metas[0].get("dims") if metas else None
    )
    axis_aligner_plan: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
        dims_str_pair=dims_str_pair
    )

    return AlignerPlan(
        per_step_plans=Pair(
            x=_compute_per_step_plans(
                metas=metas_pair.x,
                thd_seq_lens_by_step=thd_seq_lens_by_step_pair.x,
            ),
            y=_compute_per_step_plans(
                metas=metas_pair.y,
                thd_seq_lens_by_step=thd_seq_lens_by_step_pair.y,
            ),
        ),
        token_aligner_mode=token_aligner_mode,
        token_aligner_plan=token_aligner_plan,
        axis_aligner_plan=axis_aligner_plan,
    )
```
**EN:** Function `compute_aligner_plan` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `compute_aligner_plan` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 69-96: Implement helper `_compute_per_step_plans` / 实现辅助函数 `_compute_per_step_plans`
```python
def _compute_per_step_plans(
    metas: list[dict[str, Any]],
    *,
    thd_seq_lens_by_step: Optional[dict[int, list[int]]] = None,
) -> list[AlignerPerStepPlan]:
    step_to_input_indices: dict[int, list[int]] = {}
    for i, meta in enumerate(metas):
        step: int = int(meta["step"])
        step_to_input_indices.setdefault(step, []).append(i)

    result: list[AlignerPerStepPlan] = []
    for step in sorted(step_to_input_indices):
        input_indices: list[int] = step_to_input_indices[step]
        step_metas: list[dict[str, Any]] = [metas[idx] for idx in input_indices]
        step_seq_lens: Optional[list[int]] = (
            thd_seq_lens_by_step.get(step) if thd_seq_lens_by_step is not None else None
        )
        plans: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(
            metas=step_metas,
            thd_global_seq_lens=step_seq_lens,
        )
        result.append(
            AlignerPerStepPlan(
                step=step, input_object_indices=input_indices, sub_plans=plans
            )
        )

    return result
```
**EN:** Function `_compute_per_step_plans` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compute_per_step_plans` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 99-134: Implement function `compute_per_step_sub_plans` / 实现函数 `compute_per_step_sub_plans`
```python
def compute_per_step_sub_plans(
    metas: list[dict[str, Any]],
    *,
    thd_global_seq_lens: Optional[list[int]] = None,
) -> list[AlignerPerStepSubPlan]:
    if not metas or len(metas) == 1:
        return []

    dims_str = metas[0].get("dims")
    if dims_str is None:
        return []

    dims_spec: DimsSpec = parse_dims(dims_str)
    dim_specs: list[DimSpec] = _SingletonDimUtil.filter_out(dims_spec.dims)
    replicated_axes: frozenset[ParallelAxis] = dims_spec.replicated_axes
    parallel_infos = [normalize_parallel_info(meta) for meta in metas]

    dp_axis: ParallelAxis = (
        ParallelAxis(dims_spec.dp_group_alias)
        if dims_spec.dp_group_alias
        else ParallelAxis.DP
    )

    unsharder_plans = compute_unsharder_plan(
        dim_specs=dim_specs,
        parallel_infos=parallel_infos,
        explicit_replicated_axes=replicated_axes,
        thd_global_seq_lens=thd_global_seq_lens,
        dp_filtered_axis=dims_spec.dp_axis,
    )
    reorderer_plans = compute_reorderer_plans(
        dim_specs=dim_specs,
        parallel_infos=parallel_infos,
        thd_global_seq_lens=thd_global_seq_lens,
    )
    return [*unsharder_plans, *reorderer_plans]
```
**EN:** Function `compute_per_step_sub_plans` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `compute_per_step_sub_plans` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `compute_aligner_plan`, `_compute_per_step_plans`, `compute_per_step_sub_plans`
- **Module role / 模块角色**: End-to-end alignment orchestration / 端到端对齐编排
- **Implementation focus / 实现重点**: Computes reusable plans that later stages can execute deterministically / 计算可复用的计划对象，供后续阶段确定性执行

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.axis_aligner`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.types`, `sglang.srt.debug_utils.comparator.aligner.reorderer.planner`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.parallel_info`, `sglang.srt.debug_utils.comparator.aligner.unsharder.planner`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.utils`
