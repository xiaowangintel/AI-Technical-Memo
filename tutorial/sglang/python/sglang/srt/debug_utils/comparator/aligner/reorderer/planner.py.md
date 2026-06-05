# planner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/reorderer/planner.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on tensor reorder planning and execution. It mainly computes reusable plans that later stages can execute deterministically. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于张量重排规划与执行。它主要用于计算可复用的计划对象，供后续阶段确定性执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Import dependencies and shared types / 导入依赖与共享类型
```python
from typing import Optional

from sglang.srt.debug_utils.comparator.aligner.reorderer.types import (
    ReordererPlan,
    ZigzagToNaturalParams,
    ZigzagToNaturalThdParams,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import AxisInfo
from sglang.srt.debug_utils.comparator.dims_spec import (
    SEQ_DIM_NAME,
    TOKEN_DIM_NAME,
    DimSpec,
    Ordering,
    ParallelAxis,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 17-17: Declare module-level symbols such as `_ALLOWED_ZIGZAG_DIM_NAMES` / 声明模块级符号，例如 `_ALLOWED_ZIGZAG_DIM_NAMES`
```python
_ALLOWED_ZIGZAG_DIM_NAMES: set[str] = {SEQ_DIM_NAME, TOKEN_DIM_NAME}
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 20-67: Implement function `compute_reorderer_plans` / 实现函数 `compute_reorderer_plans`
```python
def compute_reorderer_plans(
    dim_specs: list[DimSpec],
    parallel_infos: list[dict[ParallelAxis, AxisInfo]],
    *,
    thd_global_seq_lens: Optional[list[int]] = None,
) -> list[ReordererPlan]:
    plans: list[ReordererPlan] = []

    for spec in dim_specs:
        for modifier in spec.parallel_modifiers:
            if modifier.ordering is None or modifier.ordering == Ordering.NATURAL:
                continue

            if spec.name not in _ALLOWED_ZIGZAG_DIM_NAMES:
                raise ValueError(
                    f"Zigzag ordering is only supported on sequence dims "
                    f"(dim name must be one of "
                    f"{sorted(_ALLOWED_ZIGZAG_DIM_NAMES)}), "
                    f"but got dim name {spec.name!r} in {spec}"
                )

            if modifier.ordering != Ordering.ZIGZAG:
                raise ValueError(
                    f"Unsupported ordering {modifier.ordering!r} for dim {spec.name!r}"
                )
            axis_size: int = parallel_infos[0][modifier.axis].axis_size

            if spec.name == TOKEN_DIM_NAME:
                if thd_global_seq_lens is None:
                    raise ValueError(
                        "thd_global_seq_lens is required for zigzag reorder on 't' dimension"
                    )
                params = ZigzagToNaturalThdParams(
                    dim_name=spec.name,
                    cp_size=axis_size,
                    seq_lens=thd_global_seq_lens,
                )
            elif spec.name == SEQ_DIM_NAME:
                params = ZigzagToNaturalParams(dim_name=spec.name, cp_size=axis_size)
            else:
                raise ValueError(
                    f"Unsupported zigzag dim name {spec.name!r}, "
                    f"expected one of {sorted(_ALLOWED_ZIGZAG_DIM_NAMES)}"
                )

            plans.append(ReordererPlan(params=params))

    return plans
```
**EN:** Function `compute_reorderer_plans` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `compute_reorderer_plans` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `compute_reorderer_plans`
- **Module role / 模块角色**: Tensor reorder planning and execution / 张量重排规划与执行
- **Implementation focus / 实现重点**: Computes reusable plans that later stages can execute deterministically / 计算可复用的计划对象，供后续阶段确定性执行

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.reorderer.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`
