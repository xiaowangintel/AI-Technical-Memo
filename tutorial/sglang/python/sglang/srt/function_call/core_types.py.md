# core_types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/core_types.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file declares the core dataclasses, call signatures, and result containers used across the function-call subsystem. These types let detectors and parsers exchange structured information safely. / 该文件声明了函数调用子系统使用的核心数据类、可调用签名与结果容器，使检测器与解析器能够安全地交换结构化信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Imports and module setup
```python
from dataclasses import dataclass
from typing import Callable, List, Optional

from pydantic import BaseModel


```
**EN:** This block gathers the imports and module-level setup for shared function-call data structures. The imported modules show which runtime services, schemas, or backend components this file depends on.
**CN:** 这一段汇集了与共享函数调用数据结构相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。

### Lines 7-7: Class `ToolCallItem` declaration
```python
class ToolCallItem(BaseModel):
```
**EN:** This block declares the class `ToolCallItem` and establishes its responsibility inside shared function-call data structures. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `ToolCallItem`.
**CN:** 这一段声明了类 `ToolCallItem`，并说明它在共享函数调用数据结构中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `ToolCallItem`。

### Lines 8-12: Supporting state inside `ToolCallItem`
```python
    """Simple encapsulation of the parsed ToolCall result for easier usage in streaming contexts."""

    tool_index: int
    name: Optional[str] = None
    parameters: str  # JSON string
```
**EN:** This block adds supporting state or helper logic inside `ToolCallItem`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `ToolCallItem` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 15-15: Class `StreamingParseResult` declaration
```python
class StreamingParseResult(BaseModel):
```
**EN:** This block declares the class `StreamingParseResult` and establishes its responsibility inside shared function-call data structures. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `StreamingParseResult`.
**CN:** 这一段声明了类 `StreamingParseResult`，并说明它在共享函数调用数据结构中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `StreamingParseResult`。

### Lines 16-19: Supporting state inside `StreamingParseResult`
```python
    """Result of streaming incremental parsing."""

    normal_text: str = ""
    calls: List[ToolCallItem] = []
```
**EN:** This block adds supporting state or helper logic inside `StreamingParseResult`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `StreamingParseResult` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 22-23: Class `StructureInfo` declaration
```python
@dataclass
class StructureInfo:
```
**EN:** This block declares the class `StructureInfo` and establishes its responsibility inside shared function-call data structures. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `StructureInfo`.
**CN:** 这一段声明了类 `StructureInfo`，并说明它在共享函数调用数据结构中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `StructureInfo`。

### Lines 24-26: Supporting state inside `StructureInfo`
```python
    begin: str
    end: str
    trigger: str
```
**EN:** This block adds supporting state or helper logic inside `StructureInfo`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `StructureInfo` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 27-34: Module-level constants and helper logic
```python


"""
Helper alias of function
Usually it is a function that takes a name string and returns a StructureInfo object,
which can be used to construct a structural_tag object
"""
_GetInfoFunc = Callable[[str], StructureInfo]
```
**EN:** This block contains module-level constants, helpers, or documentation for shared function-call data structures. It prepares shared state that later classes and functions build on.
**CN:** 这一段包含与共享函数调用数据结构相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。

## Key Concepts / 关键概念
- `ToolCallItem`: Class that encapsulates tool call item behavior in this module. / `ToolCallItem`：封装与“工具调用项”相关行为的类。
- `StreamingParseResult`: Class that encapsulates streaming parse result behavior in this module. / `StreamingParseResult`：封装与“流式解析result”相关行为的类。
- `StructureInfo`: Class that encapsulates structure info behavior in this module. / `StructureInfo`：封装与“structureinfo”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **External packages / 外部依赖**: `pydantic`
