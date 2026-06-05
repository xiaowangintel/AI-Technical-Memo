# output_types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/output_types.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on typed data models for debugging. It mainly defines structured types that make the debugging pipeline explicit and validated. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于调试用类型化数据模型。它主要用于定义结构化类型，使调试流水线更清晰且可校验。

## Line-by-Line Analysis / 逐行分析
### Lines 1-37: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from abc import abstractmethod
from typing import TYPE_CHECKING, Annotated, Any, Literal, Optional, Union

from pydantic import ConfigDict, Discriminator, Field, TypeAdapter, model_validator
from rich.console import Group, RenderableType
from rich.markup import escape

from sglang.srt.debug_utils.comparator.output_formatter import (  # noqa: F401 — re-export
    _format_aligner_plan as _format_aligner_plan,
)
from sglang.srt.debug_utils.comparator.output_formatter import (
    _format_config_body,
    _format_config_rich_body,
    _format_error_body,
    _format_error_rich_body,
    _format_log_body,
    _format_non_tensor_body,
    _format_non_tensor_rich_body,
    _format_skip_body,
    _format_skip_rich_body,
    _format_summary_body,
    _format_summary_rich_body,
    _format_table_body,
    _format_table_rich_body,
    _format_tensor_comparison_body,
    _format_tensor_comparison_rich_body,
    _render_record_rich,
    _render_record_text,
)
from sglang.srt.debug_utils.comparator.tensor_comparator.types import (
    DiffInfo,
    TensorComparisonInfo,
    TensorInfo,
)
from sglang.srt.debug_utils.comparator.utils import Pair, _StrictBase
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 39-43: Handle conditional module logic / 处理条件模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types import (
        TracedAlignerPlan,
    )
    from sglang.srt.debug_utils.comparator.report_sink import Verbosity
```
**EN:** This conditional branch selects behavior based on runtime state, optional inputs, or developer-facing entry conditions.
**CN:** 该条件分支会根据运行时状态、可选输入或面向开发者的入口条件来选择不同的行为。

### Lines 46-46: Define class `BaseLog` and class context / 定义类 `BaseLog`及类上下文
```python
class BaseLog(_StrictBase):
```
**EN:** This section introduces `BaseLog`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `BaseLog`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 47-48: Declare fields for `BaseLog` such as `category`, `message` / 为 `BaseLog` 声明字段，例如 `category`, `message`
```python
    category: str
    message: str
```
**EN:** These lines declare the state carried by `BaseLog`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `BaseLog` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 50-51: Implement method `to_text` for `BaseLog` / 为 `BaseLog` 实现方法 `to_text`
```python
    def to_text(self) -> str:
        return self.message
```
**EN:** Method `to_text` implements behavior on `BaseLog`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `to_text` 为 `BaseLog` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 54-54: Define class `ErrorLog` and class context / 定义类 `ErrorLog`及类上下文
```python
class ErrorLog(BaseLog):
```
**EN:** This section introduces `ErrorLog`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ErrorLog`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 55-55: Declare fields for `ErrorLog` such as `kind` / 为 `ErrorLog` 声明字段，例如 `kind`
```python
    kind: Literal["error"] = "error"
```
**EN:** These lines declare the state carried by `ErrorLog`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ErrorLog` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 58-58: Define class `InfoLog` and class context / 定义类 `InfoLog`及类上下文
```python
class InfoLog(BaseLog):
```
**EN:** This section introduces `InfoLog`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `InfoLog`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 59-59: Declare fields for `InfoLog` such as `kind` / 为 `InfoLog` 声明字段，例如 `kind`
```python
    kind: Literal["info"] = "info"
```
**EN:** These lines declare the state carried by `InfoLog`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `InfoLog` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 62-62: Declare module-level symbols such as `AnyLog` / 声明模块级符号，例如 `AnyLog`
```python
AnyLog = Annotated[Union[ErrorLog, InfoLog], Discriminator("kind")]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 65-68: Implement helper `_split_logs` / 实现辅助函数 `_split_logs`
```python
def _split_logs(logs: list[BaseLog]) -> tuple[list[ErrorLog], list[InfoLog]]:
    errors: list[ErrorLog] = [log for log in logs if isinstance(log, ErrorLog)]
    infos: list[InfoLog] = [log for log in logs if isinstance(log, InfoLog)]
    return errors, infos
```
**EN:** Function `_split_logs` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_split_logs` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 71-71: Define class `ReplicatedCheckResult` and class context / 定义类 `ReplicatedCheckResult`及类上下文
```python
class ReplicatedCheckResult(_StrictBase):
```
**EN:** This section introduces `ReplicatedCheckResult`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ReplicatedCheckResult`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 72-78: Declare fields for `ReplicatedCheckResult` such as `axis`, `group_index`, `compared_index`, `baseline_index`, `passed` / 为 `ReplicatedCheckResult` 声明字段，例如 `axis`, `group_index`, `compared_index`, `baseline_index`, `passed`
```python
    axis: str
    group_index: int
    compared_index: int
    baseline_index: int
    passed: bool
    atol: float
    diff: Optional[DiffInfo] = None
```
**EN:** These lines declare the state carried by `ReplicatedCheckResult`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ReplicatedCheckResult` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 81-81: Define class `BundleFileInfo` and class context / 定义类 `BundleFileInfo`及类上下文
```python
class BundleFileInfo(_StrictBase):
```
**EN:** This section introduces `BundleFileInfo`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `BundleFileInfo`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 82-82: Document the module intent / 说明模块意图
```python
    """Per-file info within a bundle (one rank's raw tensor)."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 84-88: Declare fields for `BundleFileInfo` such as `shape`, `dtype`, `rank`, `parallel_info`, `filename` / 为 `BundleFileInfo` 声明字段，例如 `shape`, `dtype`, `rank`, `parallel_info`, `filename`
```python
    shape: list[int]
    dtype: str
    rank: Optional[int] = None
    parallel_info: Optional[dict[str, str]] = None  # e.g. {"tp": "0/4", "ep": "1/2"}
    filename: Optional[str] = None
```
**EN:** These lines declare the state carried by `BundleFileInfo`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `BundleFileInfo` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 91-91: Define class `BundleSideInfo` and class context / 定义类 `BundleSideInfo`及类上下文
```python
class BundleSideInfo(_StrictBase):
```
**EN:** This section introduces `BundleSideInfo`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `BundleSideInfo`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 92-94: Declare fields for `BundleSideInfo` such as `num_files`, `files`, `dims` / 为 `BundleSideInfo` 声明字段，例如 `num_files`, `files`, `dims`
```python
    num_files: int
    files: list[BundleFileInfo]
    dims: Optional[str] = None  # e.g. "b s h(tp) d"
```
**EN:** These lines declare the state carried by `BundleSideInfo`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `BundleSideInfo` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 97-97: Define class `ShapeSnapshot` and class context / 定义类 `ShapeSnapshot`及类上下文
```python
class ShapeSnapshot(_StrictBase):
```
**EN:** This section introduces `ShapeSnapshot`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ShapeSnapshot`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 98-99: Declare fields for `ShapeSnapshot` such as `input_shapes`, `output_shapes` / 为 `ShapeSnapshot` 声明字段，例如 `input_shapes`, `output_shapes`
```python
    input_shapes: list[list[int]]
    output_shapes: list[list[int]]
```
**EN:** These lines declare the state carried by `ShapeSnapshot`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ShapeSnapshot` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 102-102: Define class `_OutputRecord` and class context / 定义类 `_OutputRecord`及类上下文
```python
class _OutputRecord(_StrictBase):
```
**EN:** This section introduces `_OutputRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_OutputRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 103-104: Declare fields for `_OutputRecord` such as `errors`, `infos` / 为 `_OutputRecord` 声明字段，例如 `errors`, `infos`
```python
    errors: list[ErrorLog] = Field(default_factory=list)
    infos: list[InfoLog] = Field(default_factory=list)
```
**EN:** These lines declare the state carried by `_OutputRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_OutputRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 107-107: Implement method `_format_body` for `_OutputRecord` / 为 `_OutputRecord` 实现方法 `_format_body`
```python
    def _format_body(self) -> str: ...
```
**EN:** Method `_format_body` implements behavior on `_OutputRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_body` 为 `_OutputRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 109-110: Implement method `_format_rich_body` for `_OutputRecord` / 为 `_OutputRecord` 实现方法 `_format_rich_body`
```python
    def _format_rich_body(self, verbosity: Verbosity = "normal") -> RenderableType:
        return self._format_body()
```
**EN:** Method `_format_rich_body` implements behavior on `_OutputRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_rich_body` 为 `_OutputRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 112-113: Implement method `to_rich` for `_OutputRecord` / 为 `_OutputRecord` 实现方法 `to_rich`
```python
    def to_rich(self, verbosity: Verbosity = "normal") -> RenderableType:
        return _render_record_rich(self, verbosity=verbosity)
```
**EN:** Method `to_rich` implements behavior on `_OutputRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `to_rich` 为 `_OutputRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 115-116: Implement method `to_text` for `_OutputRecord` / 为 `_OutputRecord` 实现方法 `to_text`
```python
    def to_text(self) -> str:
        return _render_record_text(self)
```
**EN:** Method `to_text` implements behavior on `_OutputRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `to_text` 为 `_OutputRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 119-119: Define class `RecordLocation` and class context / 定义类 `RecordLocation`及类上下文
```python
class RecordLocation(_StrictBase):
```
**EN:** This section introduces `RecordLocation`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `RecordLocation`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 120-120: Declare fields for `RecordLocation` such as `step` / 为 `RecordLocation` 声明字段，例如 `step`
```python
    step: Optional[int] = None
```
**EN:** These lines declare the state carried by `RecordLocation`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `RecordLocation` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 123-123: Define class `_BaseComparisonRecord` and class context / 定义类 `_BaseComparisonRecord`及类上下文
```python
class _BaseComparisonRecord(_OutputRecord):
```
**EN:** This section introduces `_BaseComparisonRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_BaseComparisonRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 124-124: Declare fields for `_BaseComparisonRecord` such as `location` / 为 `_BaseComparisonRecord` 声明字段，例如 `location`
```python
    location: RecordLocation = Field(default_factory=RecordLocation)
```
**EN:** These lines declare the state carried by `_BaseComparisonRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_BaseComparisonRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 126-130: Implement method `to_rich` for `_BaseComparisonRecord` / 为 `_BaseComparisonRecord` 实现方法 `to_rich`
```python
    def to_rich(self, verbosity: Verbosity = "normal") -> RenderableType:
        result = _render_record_rich(self, verbosity=verbosity)
        if isinstance(result, str):
            return result + "\n"
        return Group(result, "")
```
**EN:** Method `to_rich` implements behavior on `_BaseComparisonRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `to_rich` 为 `_BaseComparisonRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 132-135: Implement method `_format_location_prefix` for `_BaseComparisonRecord` / 为 `_BaseComparisonRecord` 实现方法 `_format_location_prefix`
```python
    def _format_location_prefix(self) -> str:
        if self.location.step is not None:
            return f"[step={self.location.step}] "
        return ""
```
**EN:** Method `_format_location_prefix` implements behavior on `_BaseComparisonRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_location_prefix` 为 `_BaseComparisonRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 137-140: Implement method `_format_location_prefix_rich` for `_BaseComparisonRecord` / 为 `_BaseComparisonRecord` 实现方法 `_format_location_prefix_rich`
```python
    def _format_location_prefix_rich(self) -> str:
        if self.location.step is not None:
            return escape(f"[step={self.location.step}]") + " "
        return ""
```
**EN:** Method `_format_location_prefix_rich` implements behavior on `_BaseComparisonRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_location_prefix_rich` 为 `_BaseComparisonRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 142-145: Implement method `_format_location_suffix` for `_BaseComparisonRecord` / 为 `_BaseComparisonRecord` 实现方法 `_format_location_suffix`
```python
    def _format_location_suffix(self) -> str:
        if self.location.step is not None:
            return f" (step={self.location.step})"
        return ""
```
**EN:** Method `_format_location_suffix` implements behavior on `_BaseComparisonRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_location_suffix` 为 `_BaseComparisonRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 148-148: Define class `ConfigRecord` and class context / 定义类 `ConfigRecord`及类上下文
```python
class ConfigRecord(_OutputRecord):
```
**EN:** This section introduces `ConfigRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ConfigRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 149-150: Declare fields for `ConfigRecord` such as `type`, `config` / 为 `ConfigRecord` 声明字段，例如 `type`, `config`
```python
    type: Literal["config"] = "config"
    config: dict[str, Any]
```
**EN:** These lines declare the state carried by `ConfigRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ConfigRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 152-153: Implement method `_format_body` for `ConfigRecord` / 为 `ConfigRecord` 实现方法 `_format_body`
```python
    def _format_body(self) -> str:
        return _format_config_body(self)
```
**EN:** Method `_format_body` implements behavior on `ConfigRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_body` 为 `ConfigRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 155-156: Implement method `_format_rich_body` for `ConfigRecord` / 为 `ConfigRecord` 实现方法 `_format_rich_body`
```python
    def _format_rich_body(self, verbosity: Verbosity = "normal") -> RenderableType:
        return _format_config_rich_body(self, verbosity=verbosity)
```
**EN:** Method `_format_rich_body` implements behavior on `ConfigRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_rich_body` 为 `ConfigRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 159-159: Define class `ComparisonSkipRecord` and class context / 定义类 `ComparisonSkipRecord`及类上下文
```python
class ComparisonSkipRecord(_BaseComparisonRecord):
```
**EN:** This section introduces `ComparisonSkipRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ComparisonSkipRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 160-165: Declare fields for `ComparisonSkipRecord` such as `type`, `name`, `reason`, `available_side`, `available_tensor_info` / 为 `ComparisonSkipRecord` 声明字段，例如 `type`, `name`, `reason`, `available_side`, `available_tensor_info`
```python
    type: Literal["comparison_skip"] = "comparison_skip"
    name: str
    reason: str
    available_side: Optional[Literal["baseline", "target"]] = None
    available_tensor_info: Optional[TensorInfo] = None
    available_bundle_info: Optional[BundleSideInfo] = None
```
**EN:** These lines declare the state carried by `ComparisonSkipRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ComparisonSkipRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 168-171: Implement method `category` for `ComparisonSkipRecord` / 为 `ComparisonSkipRecord` 实现方法 `category`
```python
    def category(self) -> str:
        if self.errors:
            return "failed"
        return "skipped"
```
**EN:** Method `category` implements behavior on `ComparisonSkipRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `category` 为 `ComparisonSkipRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 173-174: Implement method `_format_body` for `ComparisonSkipRecord` / 为 `ComparisonSkipRecord` 实现方法 `_format_body`
```python
    def _format_body(self) -> str:
        return _format_skip_body(self)
```
**EN:** Method `_format_body` implements behavior on `ComparisonSkipRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_body` 为 `ComparisonSkipRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 176-177: Implement method `_format_rich_body` for `ComparisonSkipRecord` / 为 `ComparisonSkipRecord` 实现方法 `_format_rich_body`
```python
    def _format_rich_body(self, verbosity: Verbosity = "normal") -> RenderableType:
        return _format_skip_rich_body(self, verbosity=verbosity)
```
**EN:** Method `_format_rich_body` implements behavior on `ComparisonSkipRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_rich_body` 为 `ComparisonSkipRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 180-180: Define class `ComparisonErrorRecord` and class context / 定义类 `ComparisonErrorRecord`及类上下文
```python
class ComparisonErrorRecord(_BaseComparisonRecord):
```
**EN:** This section introduces `ComparisonErrorRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ComparisonErrorRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 181-185: Declare fields for `ComparisonErrorRecord` such as `type`, `name`, `exception_type`, `exception_message`, `traceback_str` / 为 `ComparisonErrorRecord` 声明字段，例如 `type`, `name`, `exception_type`, `exception_message`, `traceback_str`
```python
    type: Literal["comparison_error"] = "comparison_error"
    name: str
    exception_type: str
    exception_message: str
    traceback_str: str
```
**EN:** These lines declare the state carried by `ComparisonErrorRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ComparisonErrorRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 188-189: Implement method `category` for `ComparisonErrorRecord` / 为 `ComparisonErrorRecord` 实现方法 `category`
```python
    def category(self) -> str:
        return "errored"
```
**EN:** Method `category` implements behavior on `ComparisonErrorRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `category` 为 `ComparisonErrorRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 191-192: Implement method `_format_body` for `ComparisonErrorRecord` / 为 `ComparisonErrorRecord` 实现方法 `_format_body`
```python
    def _format_body(self) -> str:
        return _format_error_body(self)
```
**EN:** Method `_format_body` implements behavior on `ComparisonErrorRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_body` 为 `ComparisonErrorRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 194-195: Implement method `_format_rich_body` for `ComparisonErrorRecord` / 为 `ComparisonErrorRecord` 实现方法 `_format_rich_body`
```python
    def _format_rich_body(self, verbosity: Verbosity = "normal") -> RenderableType:
        return _format_error_rich_body(self, verbosity=verbosity)
```
**EN:** Method `_format_rich_body` implements behavior on `ComparisonErrorRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_rich_body` 为 `ComparisonErrorRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 198-198: Define class `_TableRecord` and class context / 定义类 `_TableRecord`及类上下文
```python
class _TableRecord(_OutputRecord):
```
**EN:** This section introduces `_TableRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_TableRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 199-200: Declare fields for `_TableRecord` such as `label`, `rows` / 为 `_TableRecord` 声明字段，例如 `label`, `rows`
```python
    label: str
    rows: list[dict[str, Any]]
```
**EN:** These lines declare the state carried by `_TableRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_TableRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 203-203: Implement method `_table_title` for `_TableRecord` / 为 `_TableRecord` 实现方法 `_table_title`
```python
    def _table_title(self) -> str: ...
```
**EN:** Method `_table_title` implements behavior on `_TableRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_table_title` 为 `_TableRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 205-206: Implement method `_format_body` for `_TableRecord` / 为 `_TableRecord` 实现方法 `_format_body`
```python
    def _format_body(self) -> str:
        return _format_table_body(self)
```
**EN:** Method `_format_body` implements behavior on `_TableRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_body` 为 `_TableRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 208-209: Implement method `_format_rich_body` for `_TableRecord` / 为 `_TableRecord` 实现方法 `_format_rich_body`
```python
    def _format_rich_body(self, verbosity: Verbosity = "normal") -> RenderableType:
        return _format_table_rich_body(self, verbosity=verbosity)
```
**EN:** Method `_format_rich_body` implements behavior on `_TableRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_rich_body` 为 `_TableRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 212-212: Define class `RankInfoRecord` and class context / 定义类 `RankInfoRecord`及类上下文
```python
class RankInfoRecord(_TableRecord):
```
**EN:** This section introduces `RankInfoRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `RankInfoRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 213-213: Declare fields for `RankInfoRecord` such as `type` / 为 `RankInfoRecord` 声明字段，例如 `type`
```python
    type: Literal["rank_info"] = "rank_info"
```
**EN:** These lines declare the state carried by `RankInfoRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `RankInfoRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 215-216: Implement method `_table_title` for `RankInfoRecord` / 为 `RankInfoRecord` 实现方法 `_table_title`
```python
    def _table_title(self) -> str:
        return f"{self.label} ranks"
```
**EN:** Method `_table_title` implements behavior on `RankInfoRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_table_title` 为 `RankInfoRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 219-219: Define class `InputIdsRecord` and class context / 定义类 `InputIdsRecord`及类上下文
```python
class InputIdsRecord(_TableRecord):
```
**EN:** This section introduces `InputIdsRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `InputIdsRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 220-220: Declare fields for `InputIdsRecord` such as `type` / 为 `InputIdsRecord` 声明字段，例如 `type`
```python
    type: Literal["input_ids"] = "input_ids"
```
**EN:** These lines declare the state carried by `InputIdsRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `InputIdsRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 222-223: Implement method `_table_title` for `InputIdsRecord` / 为 `InputIdsRecord` 实现方法 `_table_title`
```python
    def _table_title(self) -> str:
        return f"{self.label} input_ids & positions"
```
**EN:** Method `_table_title` implements behavior on `InputIdsRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_table_title` 为 `InputIdsRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 226-226: Define class `ComparisonTensorRecord` and class context / 定义类 `ComparisonTensorRecord`及类上下文
```python
class ComparisonTensorRecord(TensorComparisonInfo, _BaseComparisonRecord):
```
**EN:** This section introduces `ComparisonTensorRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ComparisonTensorRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 227-232: Declare fields for `ComparisonTensorRecord` such as `model_config`, `type`, `traced_plan`, `replicated_checks`, `raw_bundle_info` / 为 `ComparisonTensorRecord` 声明字段，例如 `model_config`, `type`, `traced_plan`, `replicated_checks`, `raw_bundle_info`
```python
    model_config = ConfigDict(extra="forbid", defer_build=True)

    type: Literal["comparison_tensor"] = "comparison_tensor"
    traced_plan: Optional[TracedAlignerPlan] = None
    replicated_checks: list[ReplicatedCheckResult] = Field(default_factory=list)
    raw_bundle_info: Optional[Pair[BundleSideInfo]] = None
```
**EN:** These lines declare the state carried by `ComparisonTensorRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ComparisonTensorRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 235-240: Implement method `category` for `ComparisonTensorRecord` / 为 `ComparisonTensorRecord` 实现方法 `category`
```python
    def category(self) -> str:
        if self.errors:
            return "failed"
        if any(not check.passed for check in self.replicated_checks):
            return "failed"
        return "passed" if self.diff is not None and self.diff.passed else "failed"
```
**EN:** Method `category` implements behavior on `ComparisonTensorRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `category` 为 `ComparisonTensorRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 242-243: Implement method `_format_body` for `ComparisonTensorRecord` / 为 `ComparisonTensorRecord` 实现方法 `_format_body`
```python
    def _format_body(self) -> str:
        return _format_tensor_comparison_body(self)
```
**EN:** Method `_format_body` implements behavior on `ComparisonTensorRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_body` 为 `ComparisonTensorRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 245-246: Implement method `_format_rich_body` for `ComparisonTensorRecord` / 为 `ComparisonTensorRecord` 实现方法 `_format_rich_body`
```python
    def _format_rich_body(self, verbosity: Verbosity = "normal") -> RenderableType:
        return _format_tensor_comparison_rich_body(self, verbosity=verbosity)
```
**EN:** Method `_format_rich_body` implements behavior on `ComparisonTensorRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_rich_body` 为 `ComparisonTensorRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 249-249: Define class `ComparisonNonTensorRecord` and class context / 定义类 `ComparisonNonTensorRecord`及类上下文
```python
class ComparisonNonTensorRecord(_BaseComparisonRecord):
```
**EN:** This section introduces `ComparisonNonTensorRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ComparisonNonTensorRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 250-256: Declare fields for `ComparisonNonTensorRecord` such as `type`, `name`, `baseline_value`, `target_value`, `baseline_type` / 为 `ComparisonNonTensorRecord` 声明字段，例如 `type`, `name`, `baseline_value`, `target_value`, `baseline_type`
```python
    type: Literal["comparison_non_tensor"] = "comparison_non_tensor"
    name: str
    baseline_value: str
    target_value: str
    baseline_type: str
    target_type: str
    values_equal: bool
```
**EN:** These lines declare the state carried by `ComparisonNonTensorRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ComparisonNonTensorRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 259-262: Implement method `category` for `ComparisonNonTensorRecord` / 为 `ComparisonNonTensorRecord` 实现方法 `category`
```python
    def category(self) -> str:
        if self.errors:
            return "failed"
        return "passed" if self.values_equal else "failed"
```
**EN:** Method `category` implements behavior on `ComparisonNonTensorRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `category` 为 `ComparisonNonTensorRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 264-265: Implement method `_format_body` for `ComparisonNonTensorRecord` / 为 `ComparisonNonTensorRecord` 实现方法 `_format_body`
```python
    def _format_body(self) -> str:
        return _format_non_tensor_body(self)
```
**EN:** Method `_format_body` implements behavior on `ComparisonNonTensorRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_body` 为 `ComparisonNonTensorRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 267-268: Implement method `_format_rich_body` for `ComparisonNonTensorRecord` / 为 `ComparisonNonTensorRecord` 实现方法 `_format_rich_body`
```python
    def _format_rich_body(self, verbosity: Verbosity = "normal") -> RenderableType:
        return _format_non_tensor_rich_body(self, verbosity=verbosity)
```
**EN:** Method `_format_rich_body` implements behavior on `ComparisonNonTensorRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_rich_body` 为 `ComparisonNonTensorRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 271-271: Define class `SummaryRecord` and class context / 定义类 `SummaryRecord`及类上下文
```python
class SummaryRecord(_OutputRecord):
```
**EN:** This section introduces `SummaryRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `SummaryRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 272-277: Declare fields for `SummaryRecord` such as `type`, `total`, `passed`, `failed`, `skipped` / 为 `SummaryRecord` 声明字段，例如 `type`, `total`, `passed`, `failed`, `skipped`
```python
    type: Literal["summary"] = "summary"
    total: int
    passed: int
    failed: int
    skipped: int
    errored: int = 0
```
**EN:** These lines declare the state carried by `SummaryRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `SummaryRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 280-287: Implement method `_validate_totals` for `SummaryRecord` / 为 `SummaryRecord` 实现方法 `_validate_totals`
```python
    def _validate_totals(self) -> "SummaryRecord":
        expected: int = self.passed + self.failed + self.skipped + self.errored
        if self.total != expected:
            raise ValueError(
                f"total={self.total} != passed({self.passed}) + failed({self.failed}) "
                f"+ skipped({self.skipped}) + errored({self.errored}) = {expected}"
            )
        return self
```
**EN:** Method `_validate_totals` implements behavior on `SummaryRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_validate_totals` 为 `SummaryRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 289-290: Implement method `_format_body` for `SummaryRecord` / 为 `SummaryRecord` 实现方法 `_format_body`
```python
    def _format_body(self) -> str:
        return _format_summary_body(self)
```
**EN:** Method `_format_body` implements behavior on `SummaryRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_body` 为 `SummaryRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 292-293: Implement method `_format_rich_body` for `SummaryRecord` / 为 `SummaryRecord` 实现方法 `_format_rich_body`
```python
    def _format_rich_body(self, verbosity: Verbosity = "normal") -> RenderableType:
        return _format_summary_rich_body(self, verbosity=verbosity)
```
**EN:** Method `_format_rich_body` implements behavior on `SummaryRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_rich_body` 为 `SummaryRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 296-296: Define class `LogRecord` and class context / 定义类 `LogRecord`及类上下文
```python
class LogRecord(_OutputRecord):
```
**EN:** This section introduces `LogRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `LogRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 297-297: Declare fields for `LogRecord` such as `type` / 为 `LogRecord` 声明字段，例如 `type`
```python
    type: Literal["log"] = "log"
```
**EN:** These lines declare the state carried by `LogRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `LogRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 299-300: Implement method `_format_body` for `LogRecord` / 为 `LogRecord` 实现方法 `_format_body`
```python
    def _format_body(self) -> str:
        return _format_log_body(self)
```
**EN:** Method `_format_body` implements behavior on `LogRecord`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_format_body` 为 `LogRecord` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 303-316: Declare module-level symbols such as `AnyRecord` / 声明模块级符号，例如 `AnyRecord`
```python
AnyRecord = Annotated[
    Union[
        ConfigRecord,
        RankInfoRecord,
        InputIdsRecord,
        ComparisonSkipRecord,
        ComparisonErrorRecord,
        ComparisonTensorRecord,
        ComparisonNonTensorRecord,
        SummaryRecord,
        LogRecord,
    ],
    Discriminator("type"),
]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 319-320: Implement helper `_get_any_record_adapter` / 实现辅助函数 `_get_any_record_adapter`
```python
def _get_any_record_adapter() -> TypeAdapter:
    return TypeAdapter(AnyRecord)
```
**EN:** Function `_get_any_record_adapter` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_get_any_record_adapter` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 323-324: Implement function `parse_record_json` / 实现函数 `parse_record_json`
```python
def parse_record_json(json_str: str | bytes) -> AnyRecord:
    return _get_any_record_adapter().validate_json(json_str)
```
**EN:** Function `parse_record_json` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `parse_record_json` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `BaseLog`, `ErrorLog`, `InfoLog`, `ReplicatedCheckResult`, `BundleFileInfo`, `BundleSideInfo`, `ShapeSnapshot`, `_OutputRecord`, `RecordLocation`, `_BaseComparisonRecord`, `ConfigRecord`, `ComparisonSkipRecord`
- **Module role / 模块角色**: Typed data models for debugging / 调试用类型化数据模型
- **Implementation focus / 实现重点**: Defines structured types that make the debugging pipeline explicit and validated / 定义结构化类型，使调试流水线更清晰且可校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `abc`, `typing`
- **Third-party / 第三方**: `pydantic`, `rich`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.output_formatter`, `sglang.srt.debug_utils.comparator.tensor_comparator.types`, `sglang.srt.debug_utils.comparator.utils`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types`, `sglang.srt.debug_utils.comparator.report_sink`
