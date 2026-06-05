# types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/entrypoint/types.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on end-to-end alignment orchestration. It mainly defines structured types that make the debugging pipeline explicit and validated. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于端到端对齐编排。它主要用于定义结构化类型，使调试流水线更清晰且可校验。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from typing import Annotated, Optional, Union

from pydantic import Discriminator

from sglang.srt.debug_utils.comparator.aligner.axis_aligner import AxisAlignerPlan
from sglang.srt.debug_utils.comparator.aligner.reorderer.types import ReordererPlan
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    TokenAlignerPlan,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import UnsharderPlan
from sglang.srt.debug_utils.comparator.utils import Pair, _FrozenBase
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 15-18: Declare module-level symbols such as `AlignerPerStepSubPlan` / 声明模块级符号，例如 `AlignerPerStepSubPlan`
```python
AlignerPerStepSubPlan = Annotated[
    Union[UnsharderPlan, ReordererPlan],
    Discriminator("type"),
]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 21-21: Define class `AlignerPerStepPlan` and class context / 定义类 `AlignerPerStepPlan`及类上下文
```python
class AlignerPerStepPlan(_FrozenBase):
```
**EN:** This section introduces `AlignerPerStepPlan`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `AlignerPerStepPlan`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 22-24: Declare fields for `AlignerPerStepPlan` such as `step`, `input_object_indices`, `sub_plans` / 为 `AlignerPerStepPlan` 声明字段，例如 `step`, `input_object_indices`, `sub_plans`
```python
    step: int
    input_object_indices: list[int]
    sub_plans: list[AlignerPerStepSubPlan]
```
**EN:** These lines declare the state carried by `AlignerPerStepPlan`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `AlignerPerStepPlan` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 27-27: Define class `AlignerPlan` and class context / 定义类 `AlignerPlan`及类上下文
```python
class AlignerPlan(_FrozenBase):
```
**EN:** This section introduces `AlignerPlan`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `AlignerPlan`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 28-31: Declare fields for `AlignerPlan` such as `per_step_plans`, `token_aligner_mode`, `token_aligner_plan`, `axis_aligner_plan` / 为 `AlignerPlan` 声明字段，例如 `per_step_plans`, `token_aligner_mode`, `token_aligner_plan`, `axis_aligner_plan`
```python
    per_step_plans: Pair[list[AlignerPerStepPlan]]
    token_aligner_mode: Optional[str] = None  # "concat_steps" | "smart" | None
    token_aligner_plan: Optional[TokenAlignerPlan] = None
    axis_aligner_plan: Optional[AxisAlignerPlan] = None
```
**EN:** These lines declare the state carried by `AlignerPlan`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `AlignerPlan` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `AlignerPerStepPlan`, `AlignerPlan`
- **Module role / 模块角色**: End-to-end alignment orchestration / 端到端对齐编排
- **Implementation focus / 实现重点**: Defines structured types that make the debugging pipeline explicit and validated / 定义结构化类型，使调试流水线更清晰且可校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`
- **Third-party / 第三方**: `pydantic`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.axis_aligner`, `sglang.srt.debug_utils.comparator.aligner.reorderer.types`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.utils`
