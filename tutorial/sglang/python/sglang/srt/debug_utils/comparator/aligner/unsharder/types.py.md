# types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/unsharder/types.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on unsharding and tensor-parallel merging. It mainly defines structured types that make the debugging pipeline explicit and validated. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于反分片与张量并行合并。它主要用于定义结构化类型，使调试流水线更清晰且可校验。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from typing import Annotated, Literal, Union

from pydantic import Field, model_validator

from sglang.srt.debug_utils.comparator.dims_spec import ParallelAxis
from sglang.srt.debug_utils.comparator.utils import _FrozenBase
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 11-11: Define class `AxisInfo` and class context / 定义类 `AxisInfo`及类上下文
```python
class AxisInfo(_FrozenBase):
```
**EN:** This section introduces `AxisInfo`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `AxisInfo`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 12-13: Declare fields for `AxisInfo` such as `axis_rank`, `axis_size` / 为 `AxisInfo` 声明字段，例如 `axis_rank`, `axis_size`
```python
    axis_rank: int
    axis_size: int
```
**EN:** These lines declare the state carried by `AxisInfo`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `AxisInfo` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 16-23: Implement method `_validate_bounds` for `AxisInfo` / 为 `AxisInfo` 实现方法 `_validate_bounds`
```python
    def _validate_bounds(self) -> AxisInfo:
        if self.axis_size <= 0:
            raise ValueError(f"axis_size must be > 0, got {self.axis_size}")
        if not (0 <= self.axis_rank < self.axis_size):
            raise ValueError(
                f"axis_rank must be in [0, {self.axis_size}), got {self.axis_rank}"
            )
        return self
```
**EN:** Method `_validate_bounds` implements behavior on `AxisInfo`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_validate_bounds` 为 `AxisInfo` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 26-26: Define class `ConcatParams` and class context / 定义类 `ConcatParams`及类上下文
```python
class ConcatParams(_FrozenBase):
```
**EN:** This section introduces `ConcatParams`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ConcatParams`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 27-28: Declare fields for `ConcatParams` such as `op`, `dim_name` / 为 `ConcatParams` 声明字段，例如 `op`, `dim_name`
```python
    op: Literal["concat"] = "concat"
    dim_name: str
```
**EN:** These lines declare the state carried by `ConcatParams`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ConcatParams` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 31-31: Define class `CpThdConcatParams` and class context / 定义类 `CpThdConcatParams`及类上下文
```python
class CpThdConcatParams(_FrozenBase):
```
**EN:** This section introduces `CpThdConcatParams`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `CpThdConcatParams`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 32-34: Declare fields for `CpThdConcatParams` such as `op`, `dim_name`, `seq_lens_per_rank` / 为 `CpThdConcatParams` 声明字段，例如 `op`, `dim_name`, `seq_lens_per_rank`
```python
    op: Literal["cp_thd_concat"] = "cp_thd_concat"
    dim_name: str
    seq_lens_per_rank: list[int]  # per-seq token count on each rank, e.g. [50, 32, 46]
```
**EN:** These lines declare the state carried by `CpThdConcatParams`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `CpThdConcatParams` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 37-37: Define class `PickParams` and class context / 定义类 `PickParams`及类上下文
```python
class PickParams(_FrozenBase):
```
**EN:** This section introduces `PickParams`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `PickParams`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 38-38: Declare fields for `PickParams` such as `op` / 为 `PickParams` 声明字段，例如 `op`
```python
    op: Literal["pick"] = "pick"
```
**EN:** These lines declare the state carried by `PickParams`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `PickParams` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 41-41: Define class `ReduceSumParams` and class context / 定义类 `ReduceSumParams`及类上下文
```python
class ReduceSumParams(_FrozenBase):
```
**EN:** This section introduces `ReduceSumParams`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ReduceSumParams`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 42-42: Declare fields for `ReduceSumParams` such as `op` / 为 `ReduceSumParams` 声明字段，例如 `op`
```python
    op: Literal["reduce_sum"] = "reduce_sum"
```
**EN:** These lines declare the state carried by `ReduceSumParams`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ReduceSumParams` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 45-48: Declare module-level symbols such as `UnsharderParams` / 声明模块级符号，例如 `UnsharderParams`
```python
UnsharderParams = Annotated[
    Union[ConcatParams, CpThdConcatParams, PickParams, ReduceSumParams],
    Field(discriminator="op"),
]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 51-51: Define class `UnsharderPlan` and class context / 定义类 `UnsharderPlan`及类上下文
```python
class UnsharderPlan(_FrozenBase):
```
**EN:** This section introduces `UnsharderPlan`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `UnsharderPlan`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 52-60: Declare fields for `UnsharderPlan` such as `type`, `axis`, `params`, `groups` / 为 `UnsharderPlan` 声明字段，例如 `type`, `axis`, `params`, `groups`
```python
    type: Literal["unsharder"] = "unsharder"
    axis: ParallelAxis
    params: UnsharderParams
    # groups[i] = indices in the input tensor list, which will be operated (e.g. concat) into i-th output tensor.
    #
    # Multistep example (CP=2, TP=2, 4 input tensors):
    #   plan[0] (CP): groups=[[0,2],[1,3]]  — 4 tensors → 2 tensors
    #   plan[1] (TP): groups=[[0,1]]        — 2 tensors → 1 tensor
    groups: list[list[int]]
```
**EN:** These lines declare the state carried by `UnsharderPlan`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `UnsharderPlan` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `AxisInfo`, `ConcatParams`, `CpThdConcatParams`, `PickParams`, `ReduceSumParams`, `UnsharderPlan`
- **Module role / 模块角色**: Unsharding and tensor-parallel merging / 反分片与张量并行合并
- **Implementation focus / 实现重点**: Defines structured types that make the debugging pipeline explicit and validated / 定义结构化类型，使调试流水线更清晰且可校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`
- **Third-party / 第三方**: `pydantic`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.utils`
