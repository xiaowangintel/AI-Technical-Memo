# types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/tensor_comparator/types.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on tensor comparison rules and metrics. It mainly defines structured types that make the debugging pipeline explicit and validated. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于张量比较规则与度量。它主要用于定义结构化类型，使调试流水线更清晰且可校验。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Import dependencies and shared types / 导入依赖与共享类型
```python
from typing import Optional

from sglang.srt.debug_utils.comparator.utils import _StrictBase
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 5-5: Declare module-level symbols such as `DEFAULT_PERCENTILES` / 声明模块级符号，例如 `DEFAULT_PERCENTILES`
```python
DEFAULT_PERCENTILES: tuple[int, ...] = (1, 5, 50, 95, 99)
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 8-8: Define class `TensorStats` and class context / 定义类 `TensorStats`及类上下文
```python
class TensorStats(_StrictBase):
```
**EN:** This section introduces `TensorStats`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TensorStats`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 9-14: Declare fields for `TensorStats` such as `mean`, `abs_mean`, `std`, `min`, `max` / 为 `TensorStats` 声明字段，例如 `mean`, `abs_mean`, `std`, `min`, `max`
```python
    mean: float
    abs_mean: float
    std: float
    min: float
    max: float
    percentiles: dict[int, float] = {}
```
**EN:** These lines declare the state carried by `TensorStats`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TensorStats` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 17-17: Define class `TensorInfo` and class context / 定义类 `TensorInfo`及类上下文
```python
class TensorInfo(_StrictBase):
```
**EN:** This section introduces `TensorInfo`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TensorInfo`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 18-21: Declare fields for `TensorInfo` such as `shape`, `dtype`, `stats`, `sample` / 为 `TensorInfo` 声明字段，例如 `shape`, `dtype`, `stats`, `sample`
```python
    shape: list[int]
    dtype: str
    stats: TensorStats
    sample: Optional[str] = None
```
**EN:** These lines declare the state carried by `TensorInfo`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TensorInfo` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 24-24: Define class `DiffInfo` and class context / 定义类 `DiffInfo`及类上下文
```python
class DiffInfo(_StrictBase):
```
**EN:** This section introduces `DiffInfo`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `DiffInfo`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 25-34: Declare fields for `DiffInfo` such as `rel_diff`, `max_abs_diff`, `mean_abs_diff`, `abs_diff_percentiles`, `max_diff_coord` / 为 `DiffInfo` 声明字段，例如 `rel_diff`, `max_abs_diff`, `mean_abs_diff`, `abs_diff_percentiles`, `max_diff_coord`
```python
    rel_diff: float
    max_abs_diff: float
    mean_abs_diff: float
    abs_diff_percentiles: dict[int, float] = {}
    max_diff_coord: list[int]
    baseline_at_max: float
    target_at_max: float
    diff_threshold: float
    passed: bool
    per_token_rel_diff: Optional[list[float]] = None
```
**EN:** These lines declare the state carried by `DiffInfo`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `DiffInfo` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 37-37: Define class `TensorComparisonInfo` and class context / 定义类 `TensorComparisonInfo`及类上下文
```python
class TensorComparisonInfo(_StrictBase):
```
**EN:** This section introduces `TensorComparisonInfo`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TensorComparisonInfo`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 38-45: Declare fields for `TensorComparisonInfo` such as `name`, `baseline`, `target`, `unified_shape`, `shape_mismatch` / 为 `TensorComparisonInfo` 声明字段，例如 `name`, `baseline`, `target`, `unified_shape`, `shape_mismatch`
```python
    name: str
    baseline: TensorInfo
    target: TensorInfo
    unified_shape: Optional[list[int]]
    shape_mismatch: bool
    diff: Optional[DiffInfo] = None
    diff_downcast: Optional[DiffInfo] = None
    downcast_dtype: Optional[str] = None
```
**EN:** These lines declare the state carried by `TensorComparisonInfo`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TensorComparisonInfo` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `TensorStats`, `TensorInfo`, `DiffInfo`, `TensorComparisonInfo`
- **Module role / 模块角色**: Tensor comparison rules and metrics / 张量比较规则与度量
- **Implementation focus / 实现重点**: Defines structured types that make the debugging pipeline explicit and validated / 定义结构化类型，使调试流水线更清晰且可校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.utils`
