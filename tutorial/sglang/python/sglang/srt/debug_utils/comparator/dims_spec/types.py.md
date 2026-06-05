# types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/dims_spec/types.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on tensor dimension specification handling. It mainly defines structured types that make the debugging pipeline explicit and validated. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于张量维度规格处理。它主要用于定义结构化类型，使调试流水线更清晰且可校验。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from enum import Enum
from typing import Optional

from sglang.srt.debug_utils.comparator.utils import _FrozenBase
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 8-11: Declare module-level symbols such as `TOKEN_DIM_NAME`, `BATCH_DIM_NAME`, `SEQ_DIM_NAME`, `SQUEEZE_DIM_NAME` / 声明模块级符号，例如 `TOKEN_DIM_NAME`, `BATCH_DIM_NAME`, `SEQ_DIM_NAME`, `SQUEEZE_DIM_NAME`
```python
TOKEN_DIM_NAME: str = "t"
BATCH_DIM_NAME: str = "b"
SEQ_DIM_NAME: str = "s"
SQUEEZE_DIM_NAME: str = "1"
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 14-14: Define class `TokenLayout` and class context / 定义类 `TokenLayout`及类上下文
```python
class TokenLayout(Enum):
```
**EN:** This section introduces `TokenLayout`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TokenLayout`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 15-16: Declare fields for `TokenLayout` such as `T`, `BS` / 为 `TokenLayout` 声明字段，例如 `T`, `BS`
```python
    T = "t"  # single flat token dim
    BS = "bs"  # separate batch + seq dims, need collapse
```
**EN:** These lines declare the state carried by `TokenLayout`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TokenLayout` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 20-20: Define class `ParallelAxis` and class context / 定义类 `ParallelAxis`及类上下文
```python
class ParallelAxis(Enum):
```
**EN:** This section introduces `ParallelAxis`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ParallelAxis`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 21-33: Declare fields for `ParallelAxis` such as `TP`, `CP`, `EP`, `SP`, `DP` / 为 `ParallelAxis` 声明字段，例如 `TP`, `CP`, `EP`, `SP`, `DP`
```python
    TP = "tp"
    CP = "cp"
    EP = "ep"
    SP = "sp"
    DP = "dp"
    ETP = "etp"
    EDP = "edp"
    ATTN_TP = "attn_tp"
    ATTN_DP = "attn_dp"
    MOE_EP = "moe_ep"
    MOE_TP = "moe_tp"
    MOE_DP = "moe_dp"
    RECOMPUTE_PSEUDO = "recompute_pseudo"
```
**EN:** These lines declare the state carried by `ParallelAxis`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ParallelAxis` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 36-36: Define class `Ordering` and class context / 定义类 `Ordering`及类上下文
```python
class Ordering(Enum):
```
**EN:** This section introduces `Ordering`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `Ordering`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 37-38: Declare fields for `Ordering` such as `ZIGZAG`, `NATURAL` / 为 `Ordering` 声明字段，例如 `ZIGZAG`, `NATURAL`
```python
    ZIGZAG = "zigzag"
    NATURAL = "natural"
```
**EN:** These lines declare the state carried by `Ordering`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `Ordering` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 41-41: Define class `Reduction` and class context / 定义类 `Reduction`及类上下文
```python
class Reduction(Enum):
```
**EN:** This section introduces `Reduction`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `Reduction`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 42-42: Declare fields for `Reduction` such as `PARTIAL` / 为 `Reduction` 声明字段，例如 `PARTIAL`
```python
    PARTIAL = "partial"
```
**EN:** These lines declare the state carried by `Reduction`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `Reduction` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 45-45: Define class `ParallelModifier` and class context / 定义类 `ParallelModifier`及类上下文
```python
class ParallelModifier(_FrozenBase):
```
**EN:** This section introduces `ParallelModifier`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ParallelModifier`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 46-48: Declare fields for `ParallelModifier` such as `axis`, `ordering`, `reduction` / 为 `ParallelModifier` 声明字段，例如 `axis`, `ordering`, `reduction`
```python
    axis: ParallelAxis
    ordering: Optional[Ordering] = None
    reduction: Optional[Reduction] = None
```
**EN:** These lines declare the state carried by `ParallelModifier`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ParallelModifier` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 51-57: Declare module-level symbols such as `_AXIS_LOOKUP`, `_QUALIFIER_LOOKUP`, `_FUSED_NAME_SEP` / 声明模块级符号，例如 `_AXIS_LOOKUP`, `_QUALIFIER_LOOKUP`, `_FUSED_NAME_SEP`
```python
_AXIS_LOOKUP: dict[str, ParallelAxis] = {m.value: m for m in ParallelAxis}
_QUALIFIER_LOOKUP: dict[str, Ordering | Reduction] = {
    **{m.value: m for m in Ordering},
    **{m.value: m for m in Reduction},
}

_FUSED_NAME_SEP: str = "___"
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 60-60: Define class `DimSpec` and class context / 定义类 `DimSpec`及类上下文
```python
class DimSpec(_FrozenBase):
```
**EN:** This section introduces `DimSpec`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `DimSpec`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 61-62: Declare fields for `DimSpec` such as `name`, `parallel_modifiers` / 为 `DimSpec` 声明字段，例如 `name`, `parallel_modifiers`
```python
    name: str
    parallel_modifiers: list[ParallelModifier] = []
```
**EN:** These lines declare the state carried by `DimSpec`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `DimSpec` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 65-67: Implement method `sub_dims` for `DimSpec` / 为 `DimSpec` 实现方法 `sub_dims`
```python
    def sub_dims(self) -> list[str]:
        """Sub-dim names. Fused: ``["num_heads", "head_dim"]``; plain: ``["h"]``."""
        return self.name.split("*")
```
**EN:** Method `sub_dims` implements behavior on `DimSpec`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `sub_dims` 为 `DimSpec` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 70-71: Implement method `is_fused` for `DimSpec` / 为 `DimSpec` 实现方法 `is_fused`
```python
    def is_fused(self) -> bool:
        return len(self.sub_dims) > 1
```
**EN:** Method `is_fused` implements behavior on `DimSpec`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `is_fused` 为 `DimSpec` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 74-78: Implement method `sanitized_name` for `DimSpec` / 为 `DimSpec` 实现方法 `sanitized_name`
```python
    def sanitized_name(self) -> str:
        """Name safe for PyTorch named tensors (``*`` → ``___``)."""
        if self.is_fused:
            return _FUSED_NAME_SEP.join(self.sub_dims)
        return self.name
```
**EN:** Method `sanitized_name` implements behavior on `DimSpec`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `sanitized_name` 为 `DimSpec` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 81-81: Define class `DimsSpec` and class context / 定义类 `DimsSpec`及类上下文
```python
class DimsSpec(_FrozenBase):
```
**EN:** This section introduces `DimsSpec`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `DimsSpec`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 82-82: Document the module intent / 说明模块意图
```python
    """Parsed result of a full dims string like ``"b s h[tp] # dp:=moe_dp"``."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 84-86: Declare fields for `DimsSpec` such as `dims`, `dp_group_alias`, `replicated_axes` / 为 `DimsSpec` 声明字段，例如 `dims`, `dp_group_alias`, `replicated_axes`
```python
    dims: list[DimSpec]
    dp_group_alias: Optional[str] = None
    replicated_axes: frozenset[ParallelAxis] = frozenset()
```
**EN:** These lines declare the state carried by `DimsSpec`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `DimsSpec` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 89-94: Implement method `dp_axis` for `DimsSpec` / 为 `DimsSpec` 实现方法 `dp_axis`
```python
    def dp_axis(self) -> ParallelAxis:
        return (
            ParallelAxis(self.dp_group_alias)
            if self.dp_group_alias
            else ParallelAxis.DP
        )
```
**EN:** Method `dp_axis` implements behavior on `DimsSpec`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `dp_axis` 为 `DimsSpec` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `TokenLayout`, `ParallelAxis`, `Ordering`, `Reduction`, `ParallelModifier`, `DimSpec`, `DimsSpec`
- **Module role / 模块角色**: Tensor dimension specification handling / 张量维度规格处理
- **Implementation focus / 实现重点**: Defines structured types that make the debugging pipeline explicit and validated / 定义结构化类型，使调试流水线更清晰且可校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `enum`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.utils`
