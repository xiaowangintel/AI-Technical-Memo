# executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/entrypoint/executor.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on end-to-end alignment orchestration. It mainly applies previously prepared plans to tensors, records, or intermediate debug objects. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于端到端对齐编排。它主要用于将预先生成的计划应用到张量、记录或中间调试对象上。

## Line-by-Line Analysis / 逐行分析
### Lines 1-41: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from dataclasses import dataclass, field
from typing import NamedTuple, Optional

import torch

from sglang.srt.debug_utils.comparator.aligner.axis_aligner import (
    execute_axis_aligner_plan,
)
from sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types import (
    TracedAlignerPlan,
    TracedSidePlan,
    TracedStepPlan,
    TracedSubPlan,
)
from sglang.srt.debug_utils.comparator.aligner.entrypoint.types import (
    AlignerPerStepPlan,
    AlignerPerStepSubPlan,
    AlignerPlan,
)
from sglang.srt.debug_utils.comparator.aligner.reorderer.executor import (
    execute_reorderer_plan,
)
from sglang.srt.debug_utils.comparator.aligner.reorderer.types import ReordererPlan
from sglang.srt.debug_utils.comparator.aligner.token_aligner.concat_steps import (
    execute_token_aligner_concat_steps,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.executor import (
    execute_token_aligner,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.executor import (
    UnsharderResult,
    execute_unsharder_plan,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import UnsharderPlan
from sglang.srt.debug_utils.comparator.output_types import (
    ReplicatedCheckResult,
    ShapeSnapshot,
)
from sglang.srt.debug_utils.comparator.utils import Pair
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 44-44: Define class `StepPlansResult` and class context / 定义类 `StepPlansResult`及类上下文
```python
class StepPlansResult(NamedTuple):
```
**EN:** This section introduces `StepPlansResult`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `StepPlansResult`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 45-47: Declare fields for `StepPlansResult` such as `tensors`, `checks`, `traced_side` / 为 `StepPlansResult` 声明字段，例如 `tensors`, `checks`, `traced_side`
```python
    tensors: dict[int, torch.Tensor]
    checks: list[ReplicatedCheckResult]
    traced_side: TracedSidePlan
```
**EN:** These lines declare the state carried by `StepPlansResult`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `StepPlansResult` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 50-50: Define class `SubPlansResult` and class context / 定义类 `SubPlansResult`及类上下文
```python
class SubPlansResult(NamedTuple):
```
**EN:** This section introduces `SubPlansResult`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `SubPlansResult`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 51-53: Declare fields for `SubPlansResult` such as `tensor`, `checks`, `snapshots` / 为 `SubPlansResult` 声明字段，例如 `tensor`, `checks`, `snapshots`
```python
    tensor: Optional[torch.Tensor]
    checks: list[ReplicatedCheckResult]
    snapshots: list[ShapeSnapshot]
```
**EN:** These lines declare the state carried by `SubPlansResult`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `SubPlansResult` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 57-57: Define class `AlignerResult` and class context / 定义类 `AlignerResult`及类上下文
```python
class AlignerResult:
```
**EN:** This section introduces `AlignerResult`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `AlignerResult`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 58-61: Declare fields for `AlignerResult` such as `tensors`, `failed_side_xy`, `replicated_checks`, `traced_plan` / 为 `AlignerResult` 声明字段，例如 `tensors`, `failed_side_xy`, `replicated_checks`, `traced_plan`
```python
    tensors: Optional[Pair[torch.Tensor]]
    failed_side_xy: Optional[str]  # "x" or "y"; None if success
    replicated_checks: list[ReplicatedCheckResult] = field(default_factory=list)
    traced_plan: Optional[TracedAlignerPlan] = None
```
**EN:** These lines declare the state carried by `AlignerResult`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `AlignerResult` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 64-129: Implement function `execute_aligner_plan` / 实现函数 `execute_aligner_plan`
```python
def execute_aligner_plan(
    *,
    tensors_pair: Pair[list[torch.Tensor]],
    plan: AlignerPlan,
) -> AlignerResult:
    """Execute unified unshard/reorder + token-align."""
    all_checks: list[ReplicatedCheckResult] = []

    # Per-side: unshard + reorder -> dict[step, tensor]
    result_x: StepPlansResult = _execute_step_plans(
        tensors=tensors_pair.x, step_plans=plan.per_step_plans.x
    )
    all_checks.extend(result_x.checks)

    result_y: StepPlansResult = _execute_step_plans(
        tensors=tensors_pair.y, step_plans=plan.per_step_plans.y
    )
    all_checks.extend(result_y.checks)

    traced_plan: TracedAlignerPlan = TracedAlignerPlan(
        plan=plan,
        per_side=Pair(x=result_x.traced_side, y=result_y.traced_side),
    )

    if not result_x.tensors or not result_y.tensors:
        failed_side_xy: str = "x" if not result_x.tensors else "y"
        return AlignerResult(
            tensors=None,
            failed_side_xy=failed_side_xy,
            replicated_checks=all_checks,
            traced_plan=traced_plan,
        )

    # Cross-side: token alignment (or direct extraction for single-step)
    step_pair: Pair[dict[int, torch.Tensor]] = Pair(
        x=result_x.tensors, y=result_y.tensors
    )
    combined: Pair[torch.Tensor]
    if plan.token_aligner_mode == "concat_steps":
        combined = execute_token_aligner_concat_steps(tensor_of_step_pair=step_pair)
    elif plan.token_aligner_mode == "smart":
        assert plan.token_aligner_plan is not None
        combined = execute_token_aligner(
            plan=plan.token_aligner_plan,
            tensor_of_step_pair=step_pair,
        )
    else:
        assert len(result_x.tensors) == 1 and len(result_y.tensors) == 1
        combined = Pair(
            x=list(result_x.tensors.values())[0],
            y=list(result_y.tensors.values())[0],
        )

    # Cross-side: axis alignment (squeeze singletons + rearrange dim order)
    if (aligner_plan := plan.axis_aligner_plan) is not None:
        combined = Pair(
            x=execute_axis_aligner_plan(tensor=combined.x, plan=aligner_plan, side="x"),
            y=execute_axis_aligner_plan(tensor=combined.y, plan=aligner_plan, side="y"),
        )

    return AlignerResult(
        tensors=combined,
        failed_side_xy=None,
        replicated_checks=all_checks,
        traced_plan=traced_plan,
    )
```
**EN:** Function `execute_aligner_plan` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `execute_aligner_plan` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 132-168: Implement helper `_execute_step_plans` / 实现辅助函数 `_execute_step_plans`
```python
def _execute_step_plans(
    tensors: list[torch.Tensor],
    step_plans: list[AlignerPerStepPlan],
) -> StepPlansResult:
    result: dict[int, torch.Tensor] = {}
    all_checks: list[ReplicatedCheckResult] = []
    traced_steps: list[TracedStepPlan] = []

    for step_plan in step_plans:
        step_tensors: list[torch.Tensor] = [
            tensors[i] for i in step_plan.input_object_indices
        ]
        sub_result: SubPlansResult = execute_sub_plans(
            tensors=step_tensors, plans=step_plan.sub_plans
        )
        all_checks.extend(sub_result.checks)

        traced_subs: list[TracedSubPlan] = [
            TracedSubPlan(plan=sub_plan, snapshot=snapshot)
            for sub_plan, snapshot in zip(step_plan.sub_plans, sub_result.snapshots)
        ]
        traced_steps.append(
            TracedStepPlan(
                step=step_plan.step,
                input_object_indices=step_plan.input_object_indices,
                sub_plans=traced_subs,
            )
        )

        if sub_result.tensor is not None:
            result[step_plan.step] = sub_result.tensor

    return StepPlansResult(
        tensors=result,
        checks=all_checks,
        traced_side=TracedSidePlan(step_plans=traced_steps),
    )
```
**EN:** Function `_execute_step_plans` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_execute_step_plans` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 171-199: Implement function `execute_sub_plans` / 实现函数 `execute_sub_plans`
```python
def execute_sub_plans(
    tensors: list[torch.Tensor],
    plans: list[AlignerPerStepSubPlan],
) -> SubPlansResult:
    if not tensors:
        return SubPlansResult(tensor=None, checks=[], snapshots=[])

    if not plans:
        if len(tensors) != 1:
            return SubPlansResult(tensor=None, checks=[], snapshots=[])
        return SubPlansResult(tensor=tensors[0], checks=[], snapshots=[])

    current: list[torch.Tensor] = tensors
    all_checks: list[ReplicatedCheckResult] = []
    all_snapshots: list[ShapeSnapshot] = []
    for plan in plans:
        input_shapes: list[list[int]] = [list(t.shape) for t in current]
        current, checks = execute_sub_plan(tensors=current, plan=plan)
        output_shapes: list[list[int]] = [list(t.shape) for t in current]
        all_checks.extend(checks)
        all_snapshots.append(
            ShapeSnapshot(
                input_shapes=input_shapes,
                output_shapes=output_shapes,
            )
        )

    assert len(current) == 1
    return SubPlansResult(tensor=current[0], checks=all_checks, snapshots=all_snapshots)
```
**EN:** Function `execute_sub_plans` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `execute_sub_plans` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 202-212: Implement function `execute_sub_plan` / 实现函数 `execute_sub_plan`
```python
def execute_sub_plan(
    tensors: list[torch.Tensor],
    plan: AlignerPerStepSubPlan,
) -> tuple[list[torch.Tensor], list[ReplicatedCheckResult]]:
    if isinstance(plan, UnsharderPlan):
        unsharder_result: UnsharderResult = execute_unsharder_plan(plan, tensors)
        return unsharder_result.tensors, unsharder_result.replicated_checks
    elif isinstance(plan, ReordererPlan):
        return execute_reorderer_plan(plan, tensors), []
    else:
        raise NotImplementedError(f"Unknown {plan=}")
```
**EN:** Function `execute_sub_plan` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `execute_sub_plan` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `StepPlansResult`, `SubPlansResult`, `AlignerResult`, `execute_aligner_plan`, `_execute_step_plans`, `execute_sub_plans`, `execute_sub_plan`
- **Module role / 模块角色**: End-to-end alignment orchestration / 端到端对齐编排
- **Implementation focus / 实现重点**: Applies previously prepared plans to tensors, records, or intermediate debug objects / 将预先生成的计划应用到张量、记录或中间调试对象上

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.axis_aligner`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.types`, `sglang.srt.debug_utils.comparator.aligner.reorderer.executor`, `sglang.srt.debug_utils.comparator.aligner.reorderer.types`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.concat_steps`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.executor`, `sglang.srt.debug_utils.comparator.aligner.unsharder.executor`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.utils`
