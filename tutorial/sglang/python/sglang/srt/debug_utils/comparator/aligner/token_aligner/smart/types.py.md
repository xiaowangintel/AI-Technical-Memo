# types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/token_aligner/smart/types.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on heuristic token alignment. It mainly defines structured types that make the debugging pipeline explicit and validated. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于启发式 token 对齐。它主要用于定义结构化类型，使调试流水线更清晰且可校验。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from dataclasses import dataclass, field
from typing import NamedTuple, Optional, Union

from pydantic import model_validator

from sglang.srt.debug_utils.comparator.dims_spec import TokenLayout
from sglang.srt.debug_utils.comparator.utils import (
    Pair,
    _check_equal_lengths,
    _FrozenBase,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 16-16: Define class `SGLangSeqId` and class context / 定义类 `SGLangSeqId`及类上下文
```python
class SGLangSeqId(NamedTuple):
```
**EN:** This section introduces `SGLangSeqId`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `SGLangSeqId`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 17-17: Declare fields for `SGLangSeqId` such as `rid` / 为 `SGLangSeqId` 声明字段，例如 `rid`
```python
    rid: str
```
**EN:** These lines declare the state carried by `SGLangSeqId`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `SGLangSeqId` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 20-20: Define class `PositionalSeqId` and class context / 定义类 `PositionalSeqId`及类上下文
```python
class PositionalSeqId(NamedTuple):
```
**EN:** This section introduces `PositionalSeqId`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `PositionalSeqId`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 21-22: Declare fields for `PositionalSeqId` such as `step`, `seq_index` / 为 `PositionalSeqId` 声明字段，例如 `step`, `seq_index`
```python
    step: int
    seq_index: int
```
**EN:** These lines declare the state carried by `PositionalSeqId`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `PositionalSeqId` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 25-25: Declare module-level symbols such as `SeqId` / 声明模块级符号，例如 `SeqId`
```python
SeqId = Union[SGLangSeqId, PositionalSeqId]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 29-29: Define class `TokenAlignerStepAux` and class context / 定义类 `TokenAlignerStepAux`及类上下文
```python
class TokenAlignerStepAux:
```
**EN:** This section introduces `TokenAlignerStepAux`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TokenAlignerStepAux`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 30-30: Document the module intent / 说明模块意图
```python
    """Normalized auxiliary tensors for a single step (framework-agnostic)."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 32-35: Declare fields for `TokenAlignerStepAux` such as `input_ids`, `positions`, `seq_lens`, `seq_ids` / 为 `TokenAlignerStepAux` 声明字段，例如 `input_ids`, `positions`, `seq_lens`, `seq_ids`
```python
    input_ids: list[int]  # [num_tokens]
    positions: list[int]  # [num_tokens]
    seq_lens: list[int]  # [num_seqs]
    seq_ids: list[SeqId]  # [num_seqs] — sequence identity
```
**EN:** These lines declare the state carried by `TokenAlignerStepAux`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TokenAlignerStepAux` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 37-45: Implement method `__post_init__` for `TokenAlignerStepAux` / 为 `TokenAlignerStepAux` 实现方法 `__post_init__`
```python
    def __post_init__(self) -> None:
        _check_equal_lengths(input_ids=self.input_ids, positions=self.positions)
        _check_equal_lengths(seq_lens=self.seq_lens, seq_ids=self.seq_ids)

        token_count: int = sum(self.seq_lens)
        if token_count != len(self.input_ids):
            raise ValueError(
                f"sum(seq_lens)={token_count} != len(input_ids)={len(self.input_ids)}"
            )
```
**EN:** Method `__post_init__` implements behavior on `TokenAlignerStepAux`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__post_init__` 为 `TokenAlignerStepAux` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 49-49: Define class `TokenAlignerGlobalAux` and class context / 定义类 `TokenAlignerGlobalAux`及类上下文
```python
class TokenAlignerGlobalAux:
```
**EN:** This section introduces `TokenAlignerGlobalAux`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TokenAlignerGlobalAux`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 50-50: Document the module intent / 说明模块意图
```python
    """Auxiliary tensors for one side across all steps + side-level metadata."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 52-55: Declare fields for `TokenAlignerGlobalAux` such as `step_auxs`, `framework`, `layout`, `thd_seq_lens_by_step` / 为 `TokenAlignerGlobalAux` 声明字段，例如 `step_auxs`, `framework`, `layout`, `thd_seq_lens_by_step`
```python
    step_auxs: dict[int, TokenAlignerStepAux]
    framework: str  # "sglang" | "megatron"
    layout: TokenLayout
    thd_seq_lens_by_step: Optional[dict[int, list[int]]] = field(default=None)
```
**EN:** These lines declare the state carried by `TokenAlignerGlobalAux`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TokenAlignerGlobalAux` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 58-58: Define class `TokenLocator` and class context / 定义类 `TokenLocator`及类上下文
```python
class TokenLocator(_FrozenBase):
```
**EN:** This section introduces `TokenLocator`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TokenLocator`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 59-62: Document the module intent / 说明模块意图
```python
    """Locates tokens within a multi-step tensor store.

    token i is at tensor_of_step[steps[i]][token_index_in_step[i]].
    """
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 64-65: Declare fields for `TokenLocator` such as `steps`, `token_index_in_step` / 为 `TokenLocator` 声明字段，例如 `steps`, `token_index_in_step`
```python
    steps: list[int]
    token_index_in_step: list[int]
```
**EN:** These lines declare the state carried by `TokenLocator`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TokenLocator` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 67-71: Implement method `__add__` for `TokenLocator` / 为 `TokenLocator` 实现方法 `__add__`
```python
    def __add__(self, other: TokenLocator) -> TokenLocator:
        return TokenLocator(
            steps=self.steps + other.steps,
            token_index_in_step=self.token_index_in_step + other.token_index_in_step,
        )
```
**EN:** Method `__add__` implements behavior on `TokenLocator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__add__` 为 `TokenLocator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 74-74: Define class `TokenAlignerSeqInfo` and class context / 定义类 `TokenAlignerSeqInfo`及类上下文
```python
class TokenAlignerSeqInfo(_FrozenBase):
```
**EN:** This section introduces `TokenAlignerSeqInfo`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TokenAlignerSeqInfo`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 75-75: Document the module intent / 说明模块意图
```python
    """Information for a sequence, containing information to locate all the tokens inside the sequence."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 78-80: Declare fields for `TokenAlignerSeqInfo` such as `input_ids`, `positions`, `locator` / 为 `TokenAlignerSeqInfo` 声明字段，例如 `input_ids`, `positions`, `locator`
```python
    input_ids: list[int]
    positions: list[int]
    locator: TokenLocator
```
**EN:** These lines declare the state carried by `TokenAlignerSeqInfo`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TokenAlignerSeqInfo` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 83-97: Implement method `_validate_fields` for `TokenAlignerSeqInfo` / 为 `TokenAlignerSeqInfo` 实现方法 `_validate_fields`
```python
    def _validate_fields(self) -> TokenAlignerSeqInfo:
        n: int = len(self.input_ids)
        _check_equal_lengths(
            input_ids=self.input_ids,
            positions=self.positions,
            locator_steps=self.locator.steps,
            locator_token_index_in_step=self.locator.token_index_in_step,
        )

        if self.positions != list(range(n)):
            raise ValueError(
                f"positions must be [0, 1, ..., {n - 1}], got {self.positions}"
            )

        return self
```
**EN:** Method `_validate_fields` implements behavior on `TokenAlignerSeqInfo`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_validate_fields` 为 `TokenAlignerSeqInfo` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 99-104: Implement method `__add__` for `TokenAlignerSeqInfo` / 为 `TokenAlignerSeqInfo` 实现方法 `__add__`
```python
    def __add__(self, other: TokenAlignerSeqInfo) -> TokenAlignerSeqInfo:
        return TokenAlignerSeqInfo(
            input_ids=self.input_ids + other.input_ids,
            positions=self.positions + other.positions,
            locator=self.locator + other.locator,
        )
```
**EN:** Method `__add__` implements behavior on `TokenAlignerSeqInfo`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__add__` 为 `TokenAlignerSeqInfo` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 107-107: Define class `TokenAlignerSeqsInfo` and class context / 定义类 `TokenAlignerSeqsInfo`及类上下文
```python
class TokenAlignerSeqsInfo(_FrozenBase):
```
**EN:** This section introduces `TokenAlignerSeqsInfo`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TokenAlignerSeqsInfo`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 108-108: Document the module intent / 说明模块意图
```python
    """All sequences for one side across all steps."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 110-111: Declare fields for `TokenAlignerSeqsInfo` such as `sequences`, `layout` / 为 `TokenAlignerSeqsInfo` 声明字段，例如 `sequences`, `layout`
```python
    sequences: dict[SeqId, TokenAlignerSeqInfo]
    layout: TokenLayout
```
**EN:** These lines declare the state carried by `TokenAlignerSeqsInfo`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TokenAlignerSeqsInfo` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 114-114: Define class `TokenAlignerPlan` and class context / 定义类 `TokenAlignerPlan`及类上下文
```python
class TokenAlignerPlan(_FrozenBase):
```
**EN:** This section introduces `TokenAlignerPlan`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TokenAlignerPlan`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 115-115: Document the module intent / 说明模块意图
```python
    """Token alignment plan. locators.x[i] and locators.y[i] correspond to the same logical token."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 117-118: Declare fields for `TokenAlignerPlan` such as `locators`, `layouts` / 为 `TokenAlignerPlan` 声明字段，例如 `locators`, `layouts`
```python
    locators: Pair[TokenLocator]
    layouts: Pair[TokenLayout]
```
**EN:** These lines declare the state carried by `TokenAlignerPlan`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TokenAlignerPlan` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 121-128: Implement method `_validate_fields` for `TokenAlignerPlan` / 为 `TokenAlignerPlan` 实现方法 `_validate_fields`
```python
    def _validate_fields(self) -> TokenAlignerPlan:
        _check_equal_lengths(
            locators_x_steps=self.locators.x.steps,
            locators_x_token_index_in_step=self.locators.x.token_index_in_step,
            locators_y_steps=self.locators.y.steps,
            locators_y_token_index_in_step=self.locators.y.token_index_in_step,
        )
        return self
```
**EN:** Method `_validate_fields` implements behavior on `TokenAlignerPlan`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_validate_fields` 为 `TokenAlignerPlan` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `SGLangSeqId`, `PositionalSeqId`, `TokenAlignerStepAux`, `TokenAlignerGlobalAux`, `TokenLocator`, `TokenAlignerSeqInfo`, `TokenAlignerSeqsInfo`, `TokenAlignerPlan`
- **Module role / 模块角色**: Heuristic token alignment / 启发式 token 对齐
- **Implementation focus / 实现重点**: Defines structured types that make the debugging pipeline explicit and validated / 定义结构化类型，使调试流水线更清晰且可校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`
- **Third-party / 第三方**: `pydantic`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.utils`
