# deepseekv4_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/deepseekv4_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a deepseekv 4 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 deepseekv4 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Module-level constants and helper logic
```python
import logging

from sglang.srt.function_call.deepseekv32_detector import DeepSeekV32Detector

logger = logging.getLogger(__name__)


```
**EN:** This block contains module-level constants, helpers, or documentation for deepseekv 4 style tool-call detection and normalization. It prepares shared state that later classes and functions build on. Notable operations include `getLogger`.
**CN:** 这一段包含与deepseekv4风格的工具调用检测与规范化相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `getLogger`。

### Lines 8-8: Class `DeepSeekV4Detector` declaration
```python
class DeepSeekV4Detector(DeepSeekV32Detector):
```
**EN:** This block declares the class `DeepSeekV4Detector` and establishes its responsibility inside deepseekv 4 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `DeepSeekV4Detector`.
**CN:** 这一段声明了类 `DeepSeekV4Detector`，并说明它在deepseekv4风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `DeepSeekV4Detector`。

### Lines 9-34: Supporting state inside `DeepSeekV4Detector`
```python
    """
    Detector for DeepSeek V4 model function call format.

    The DeepSeek V4 format uses XML-like DSML tags to delimit function calls.
    Supports two parameter formats:

    Format 1 - XML Parameter Tags:
    ```
    <｜DSML｜tool_calls>
        <｜DSML｜invoke name="function_name">
        <｜DSML｜parameter name="param_name" string="true">value</｜DSML｜parameter>
        ...
    </｜DSML｜invoke>
    </｜DSML｜tool_calls>
    ```

    Format 2 - Direct JSON:
    ```
    <｜DSML｜tool_calls>
        <｜DSML｜invoke name="function_name">
        {
            "param_name": "value"
        }
    </｜DSML｜invoke>
    </｜DSML｜tool_calls>
    ```
```
**EN:** This block adds supporting state or helper logic inside `DeepSeekV4Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `DeepSeekV4Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 35-55: Supporting state inside `DeepSeekV4Detector`
```python

    Examples:
    ```
    <｜DSML｜tool_calls>
        <｜DSML｜invoke name="get_favorite_tourist_spot">
        <｜DSML｜parameter name="city" string="true">San Francisco</｜DSML｜parameter>
    </｜DSML｜invoke>
    </｜DSML｜tool_calls>

    <｜DSML｜tool_calls>
        <｜DSML｜invoke name="get_favorite_tourist_spot">
        { "city": "San Francisco" }
    </｜DSML｜invoke>
    </｜DSML｜tool_calls>
    ```

    Key Components:
    - Tool Calls Section: Wrapped between `<｜DSML｜tool_calls>` and `</｜DSML｜tool_calls>`
    - Individual Tool Call: Wrapped between `<｜DSML｜invoke name="...">` and `</｜DSML｜invoke>`
    - Parameters: Either XML tags or direct JSON format
    - Supports multiple tool calls
```
**EN:** This block adds supporting state or helper logic inside `DeepSeekV4Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `DeepSeekV4Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 56-59: Supporting state inside `DeepSeekV4Detector`
```python

    Reference: DeepSeek V4 format specification
    """

```
**EN:** This block adds supporting state or helper logic inside `DeepSeekV4Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `DeepSeekV4Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 60-64: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.bot_token = "<｜DSML｜tool_calls>"
        self.eot_token = "</｜DSML｜tool_calls>"
        self.function_calls_regex = r"<｜DSML｜tool_calls>(.*?)</｜DSML｜tool_calls>"
```
**EN:** This block defines the method `__init__` on `DeepSeekV4Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 4 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `DeepSeekV4Detector`），介绍了参数、初始化步骤，以及这部分deepseekv4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 66-67: Method `get_structural_tag_name`
```python
    def get_structural_tag_name(self) -> str:
        return "deepseek_v4"
```
**EN:** This block defines the method `get_structural_tag_name` on `DeepSeekV4Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 4 style tool-call detection and normalization. Definitions introduced here include `get_structural_tag_name`.
**CN:** 这一段定义了method `get_structural_tag_name`（属于 `DeepSeekV4Detector`），介绍了参数、初始化步骤，以及这部分deepseekv4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `get_structural_tag_name`。

## Key Concepts / 关键概念
- `DeepSeekV4Detector`: Class that encapsulates deep seek v 4 detector behavior in this module. / `DeepSeekV4Detector`：封装与“deepseekv4检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`
- **Internal modules / 内部模块**: `sglang.srt.function_call.deepseekv32_detector`
