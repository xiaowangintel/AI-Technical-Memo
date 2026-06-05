# hunyuan_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/hunyuan_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a hunyuan detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 混元 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup
```python
import json
import logging
import re
from typing import Any, Dict, List, Optional, Set

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.environ import envs
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    StructureInfo,
    ToolCallItem,
    _GetInfoFunc,
)

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for hunyuan style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与混元风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 19-19: Class `HunyuanDetector` declaration
```python
class HunyuanDetector(BaseFormatDetector):
```
**EN:** This block declares the class `HunyuanDetector` and establishes its responsibility inside hunyuan style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `HunyuanDetector`.
**CN:** 这一段声明了类 `HunyuanDetector`，并说明它在混元风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `HunyuanDetector`。

### Lines 20-57: Supporting state inside `HunyuanDetector`
```python
    """
    Detector for Hunyuan (HYV3) tool call format.

    Format:
        <tool_calls>
        <tool_call>function_name<tool_sep>
        <arg_key>key1</arg_key>
        <arg_value>value1</arg_value>
        </tool_call>
        </tool_calls>

    Streaming behavior:
      * Phase 1 emits the tool name once <tool_sep> is seen.
      * Phase 2 streams argument JSON incrementally. Closed <arg_value>
        pairs are parsed with schema-aware type coercion; pure-string
        args may be streamed char-by-char (with JSON escaping). The
        closing "}" is withheld until </tool_call> arrives.
    """

    _TYPE_ALIASES: Dict[str, str] = {
        "str": "string",
        "text": "string",
        "varchar": "string",
        "char": "string",
        "enum": "string",
        "bool": "boolean",
        "binary": "boolean",
        "int": "integer",
        "float": "number",
        "double": "number",
        "list": "array",
        "dict": "object",
        "map": "object",
    }

    _INTEGER_PREFIXES = ("int", "uint", "long", "short", "unsigned")
    _NUMBER_PREFIXES = ("num", "float")

```
**EN:** This block adds supporting state or helper logic inside `HunyuanDetector`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `Hunyuan`, `char`.
**CN:** 这一段为 `HunyuanDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `Hunyuan`、`char`。

### Lines 58-60: Method `__init__` signature and setup
```python
    def __init__(self):
        super().__init__()

```
**EN:** This block defines the method `__init__` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 61-84: Method `__init__` logic (part 1)
```python
        self.bot_token = "<tool_calls>"
        self.eot_token = "</tool_calls>"

        self.tool_call_start_token = "<tool_call>"
        self.tool_call_end_token = "</tool_call>"
        self.tool_sep_token = "<tool_sep>"

        self.arg_key_start_token = "<arg_key>"
        self.arg_key_end_token = "</arg_key>"
        self.arg_value_start_token = "<arg_value>"
        self.arg_value_end_token = "</arg_value>"

        self.tool_call_regex = re.compile(
            r"<tool_call>(.*?)<tool_sep>(.*?)</tool_call>", re.DOTALL
        )
        self.func_args_regex = re.compile(
            r"<arg_key>(.*?)</arg_key>\s*<arg_value>(.*?)</arg_value>", re.DOTALL
        )

        # Streaming state
        self._in_tool_calls: bool = False
        self._streaming_tool_name: Optional[str] = None
        self._completed_args: Dict[str, Any] = {}
        self._streamed_json_len: int = 0
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `compile`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `compile`。

### Lines 85-89: Supporting state inside `HunyuanDetector`
```python

    # ------------------------------------------------------------------
    # Type-normalization helpers
    # ------------------------------------------------------------------

```
**EN:** This block adds supporting state or helper logic inside `HunyuanDetector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `HunyuanDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 90-100: Method `_normalize_type`
```python
    @staticmethod
    def _normalize_type(raw_type: str) -> str:
        exact = HunyuanDetector._TYPE_ALIASES.get(raw_type)
        if exact is not None:
            return exact
        lower = raw_type.lower()
        if any(lower.startswith(p) for p in HunyuanDetector._INTEGER_PREFIXES):
            return "integer"
        if any(lower.startswith(p) for p in HunyuanDetector._NUMBER_PREFIXES):
            return "number"
        return raw_type
```
**EN:** This block defines the method `_normalize_type` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_normalize_type`. Notable operations include `get`, `lower`, `startswith`.
**CN:** 这一段定义了method `_normalize_type`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_normalize_type`。 值得注意的操作包括 `get`、`lower`、`startswith`。

### Lines 102-113: Method `_get_arg_schema`
```python
    @staticmethod
    def _get_arg_schema(
        function_name: str, arg_key: str, tools: Optional[List[Tool]]
    ) -> dict:
        if not tools:
            return {}
        for tool in tools:
            if tool.function.name == function_name:
                if tool.function.parameters is None:
                    return {}
                return tool.function.parameters.get("properties", {}).get(arg_key, {})
        return {}
```
**EN:** This block defines the method `_get_arg_schema` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_get_arg_schema`. Notable operations include `get`.
**CN:** 这一段定义了method `_get_arg_schema`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_get_arg_schema`。 值得注意的操作包括 `get`。

### Lines 115-124: Method `_get_schema_options`
```python
    @staticmethod
    def _get_schema_options(arg_schema: dict) -> List[dict]:
        """Priority: single ``type`` > ``anyOf`` > ``oneOf``; else default string."""
        if "type" in arg_schema:
            return [arg_schema]
        if "anyOf" in arg_schema:
            return arg_schema["anyOf"]
        if "oneOf" in arg_schema:
            return arg_schema["oneOf"]
        return [{"type": "string"}]
```
**EN:** This block defines the method `_get_schema_options` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_get_schema_options`.
**CN:** 这一段定义了method `_get_schema_options`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_get_schema_options`。

### Lines 126-131: Method `_get_types`
```python
    @staticmethod
    def _get_types(arg_schema: dict) -> Set[str]:
        schemas = HunyuanDetector._get_schema_options(arg_schema)
        return {
            HunyuanDetector._normalize_type(s.get("type", "string")) for s in schemas
        } - {"null"}
```
**EN:** This block defines the method `_get_types` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_get_types`. Notable operations include `_get_schema_options`, `_normalize_type`, `get`.
**CN:** 这一段定义了method `_get_types`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_get_types`。 值得注意的操作包括 `_get_schema_options`、`_normalize_type`、`get`。

### Lines 133-141: Method `_is_only_string_type`
```python
    @staticmethod
    def _is_only_string_type(
        function_name: str, arg_key: str, tools: Optional[List[Tool]]
    ) -> bool:
        """Only pure-string args get char-by-char value streaming; compound
        types like anyOf(string | array) might resolve to a JSON array or
        object, so we can't safely stream them as open JSON strings."""
        arg_schema = HunyuanDetector._get_arg_schema(function_name, arg_key, tools)
        return HunyuanDetector._get_types(arg_schema) == {"string"}
```
**EN:** This block defines the method `_is_only_string_type` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_is_only_string_type`. Notable operations include `anyOf`, `_get_arg_schema`, `_get_types`.
**CN:** 这一段定义了method `_is_only_string_type`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_is_only_string_type`。 值得注意的操作包括 `anyOf`、`_get_arg_schema`、`_get_types`。

### Lines 143-150: Method `_try_parse_bool`
```python
    @staticmethod
    def _try_parse_bool(value: str) -> Optional[bool]:
        lower = value.lower()
        if lower == "true":
            return True
        if lower == "false":
            return False
        return None
```
**EN:** This block defines the method `_try_parse_bool` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_try_parse_bool`. Notable operations include `lower`.
**CN:** 这一段定义了method `_try_parse_bool`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_try_parse_bool`。 值得注意的操作包括 `lower`。

### Lines 152-157: Method `_try_parse_int`
```python
    @staticmethod
    def _try_parse_int(value: str) -> Optional[int]:
        try:
            return int(value)
        except (ValueError, TypeError):
            return None
```
**EN:** This block defines the method `_try_parse_int` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_try_parse_int`.
**CN:** 这一段定义了method `_try_parse_int`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_try_parse_int`。

### Lines 159-167: Method `_try_parse_number`
```python
    @staticmethod
    def _try_parse_number(value: str):
        """int if no '.'/'e'/'E', else float."""
        try:
            if "." in value or "e" in value or "E" in value:
                return float(value)
            return int(value)
        except (ValueError, TypeError):
            return None
```
**EN:** This block defines the method `_try_parse_number` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_try_parse_number`.
**CN:** 这一段定义了method `_try_parse_number`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_try_parse_number`。

### Lines 169-174: Method `_deserialize`
```python
    @staticmethod
    def _deserialize(value: str) -> Any:
        try:
            return json.loads(value)
        except (json.JSONDecodeError, ValueError):
            return value
```
**EN:** This block defines the method `_deserialize` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_deserialize`. Notable operations include `loads`.
**CN:** 这一段定义了method `_deserialize`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_deserialize`。 值得注意的操作包括 `loads`。

### Lines 176-183: Method `_parse_value` signature and setup
```python
    @staticmethod
    def _parse_value(
        value: str,
        function_name: str,
        arg_key: str,
        tools: Optional[List[Tool]],
    ) -> Any:
        """Unified value parser: bool → int → number → json (array/obj) → string."""
```
**EN:** This block defines the method `_parse_value` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_parse_value`. Notable operations include `json`.
**CN:** 这一段定义了method `_parse_value`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_value`。 值得注意的操作包括 `json`。

### Lines 184-206: Method `_parse_value` logic (part 1)
```python
        arg_schema = HunyuanDetector._get_arg_schema(function_name, arg_key, tools)
        types = HunyuanDetector._get_types(arg_schema)

        if "boolean" in types:
            r = HunyuanDetector._try_parse_bool(value)
            if r is not None:
                return r

        if "integer" in types:
            r = HunyuanDetector._try_parse_int(value)
            if r is not None:
                return r

        if "number" in types:
            r = HunyuanDetector._try_parse_number(value)
            if r is not None:
                return r

        if types - {"string", "boolean", "integer", "number"}:
            try:
                return json.loads(value)
            except (json.JSONDecodeError, ValueError):
                pass
```
**EN:** This block continues `_parse_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `_get_arg_schema`, `_get_types`, `_try_parse_bool`, `_try_parse_int`.
**CN:** 这一段延续了 `_parse_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_get_arg_schema`、`_get_types`、`_try_parse_bool`、`_try_parse_int`。

### Lines 207-211: Method `_parse_value` logic (part 2)
```python

        if "string" in types:
            return value

        return HunyuanDetector._deserialize(value)
```
**EN:** This block continues `_parse_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `_deserialize`.
**CN:** 这一段延续了 `_parse_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_deserialize`。

### Lines 212-216: Supporting state inside `HunyuanDetector`
```python

    # ------------------------------------------------------------------
    # Non-streaming
    # ------------------------------------------------------------------

```
**EN:** This block adds supporting state or helper logic inside `HunyuanDetector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `HunyuanDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 217-218: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        return self.bot_token in text
```
**EN:** This block defines the method `has_tool_call` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 220-222: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        if self.bot_token not in text:
            return StreamingParseResult(normal_text=text, calls=[])
```
**EN:** This block defines the method `detect_and_parse` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`. Notable operations include `StreamingParseResult`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `StreamingParseResult`。

### Lines 223-243: Method `detect_and_parse` logic (part 1)
```python

        idx = text.find(self.bot_token)
        normal_text = text[:idx].strip() if idx > 0 else ""

        tool_indices = self._get_tool_indices(tools)
        forward_unknown = envs.SGLANG_FORWARD_UNKNOWN_TOOLS.get()

        calls: List[ToolCallItem] = []
        try:
            for function_name, function_args in self.tool_call_regex.findall(text):
                function_name = function_name.strip()
                if function_name not in tool_indices and not forward_unknown:
                    logger.warning(
                        "Model attempted to call undefined function: %s", function_name
                    )
                    continue

                arg_dict: Dict[str, Any] = {}
                for key, value in self.func_args_regex.findall(function_args):
                    key = key.strip()
                    arg_dict[key] = self._parse_value(value, function_name, key, tools)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `find`, `strip`, `_get_tool_indices`, `get`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`strip`、`_get_tool_indices`、`get`。

### Lines 244-255: Method `detect_and_parse` logic (part 2)
```python

                calls.append(
                    ToolCallItem(
                        tool_index=tool_indices.get(function_name, -1),
                        name=function_name,
                        parameters=json.dumps(arg_dict, ensure_ascii=False),
                    )
                )
            return StreamingParseResult(normal_text=normal_text, calls=calls)
        except Exception as e:
            logger.error(f"Error in detect_and_parse: {e}", exc_info=True)
            return StreamingParseResult(normal_text=text)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `get`, `dumps`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`get`、`dumps`。

### Lines 256-260: Supporting state inside `HunyuanDetector`
```python

    # ------------------------------------------------------------------
    # Streaming
    # ------------------------------------------------------------------

```
**EN:** This block adds supporting state or helper logic inside `HunyuanDetector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `HunyuanDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 261-264: Method `_reset_streaming_tool_state`
```python
    def _reset_streaming_tool_state(self):
        self._streaming_tool_name = None
        self._completed_args = {}
        self._streamed_json_len = 0
```
**EN:** This block defines the method `_reset_streaming_tool_state` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_reset_streaming_tool_state`.
**CN:** 这一段定义了method `_reset_streaming_tool_state`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_reset_streaming_tool_state`。

### Lines 266-273: Method `parse_streaming_increment`
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        try:
            return self._parse_streaming_increment_impl(new_text, tools)
        except Exception as e:
            logger.error(f"Error in parse_streaming_increment: {e}", exc_info=True)
            return StreamingParseResult()
```
**EN:** This block defines the method `parse_streaming_increment` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`. Notable operations include `_parse_streaming_increment_impl`, `error`, `StreamingParseResult`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。 值得注意的操作包括 `_parse_streaming_increment_impl`、`error`、`StreamingParseResult`。

### Lines 275-277: Method `_parse_streaming_increment_impl` signature and setup
```python
    def _parse_streaming_increment_impl(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
```
**EN:** This block defines the method `_parse_streaming_increment_impl` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_parse_streaming_increment_impl`.
**CN:** 这一段定义了method `_parse_streaming_increment_impl`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_streaming_increment_impl`。

### Lines 278-299: Method `_parse_streaming_increment_impl` logic (part 1)
```python
        if not hasattr(self, "_tool_indices"):
            self._tool_indices = self._get_tool_indices(tools)

        # Not yet inside <tool_calls>: emit normal text or buffer partial bot_token.
        if not self._in_tool_calls:
            combined = self._buffer + new_text
            if self.bot_token in combined:
                bot_pos = combined.find(self.bot_token)
                normal_text = combined[:bot_pos]
                self._buffer = combined[bot_pos + len(self.bot_token) :]
                self._in_tool_calls = True
                return self._continue_streaming(tools, leading_normal=normal_text)

            partial_len = self._ends_with_partial_token(combined, self.bot_token)
            if partial_len:
                self._buffer = combined[-partial_len:]
                return StreamingParseResult(normal_text=combined[:-partial_len])
            self._buffer = ""
            return StreamingParseResult(normal_text=combined)

        self._buffer += new_text
        return self._continue_streaming(tools)
```
**EN:** This block continues `_parse_streaming_increment_impl` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `_get_tool_indices`, `find`, `_continue_streaming`, `_ends_with_partial_token`.
**CN:** 这一段延续了 `_parse_streaming_increment_impl` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_get_tool_indices`、`find`、`_continue_streaming`、`_ends_with_partial_token`。

### Lines 301-304: Method `_continue_streaming` signature and setup
```python
    def _continue_streaming(
        self, tools: List[Tool], leading_normal: str = ""
    ) -> StreamingParseResult:
        """Drive the state machine after <tool_calls> is open."""
```
**EN:** This block defines the method `_continue_streaming` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_continue_streaming`.
**CN:** 这一段定义了method `_continue_streaming`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_continue_streaming`。

### Lines 305-325: Method `_continue_streaming` logic (part 1)
```python
        calls: List[ToolCallItem] = []

        while True:
            if self._streaming_tool_name is None:
                # Phase 1: wait for <tool_call>..<tool_sep>.
                tc_start = self._buffer.find(self.tool_call_start_token)
                if tc_start == -1:
                    if self.eot_token in self._buffer:
                        eot_pos = self._buffer.find(self.eot_token)
                        self._buffer = self._buffer[eot_pos + len(self.eot_token) :]
                        self._in_tool_calls = False
                    break

                sep_pos = self._buffer.find(self.tool_sep_token, tc_start)
                if sep_pos == -1:
                    self._buffer = self._buffer[tc_start:]
                    break

                tool_name = self._buffer[
                    tc_start + len(self.tool_call_start_token) : sep_pos
                ].strip()
```
**EN:** This block continues `_continue_streaming` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `find`, `strip`.
**CN:** 这一段延续了 `_continue_streaming` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`strip`。

### Lines 326-348: Method `_continue_streaming` logic (part 2)
```python

                if (
                    tool_name not in self._tool_indices
                    and not envs.SGLANG_FORWARD_UNKNOWN_TOOLS.get()
                ):
                    logger.warning(
                        "Model attempted to call undefined function: %s", tool_name
                    )

                self._streaming_tool_name = tool_name
                self.current_tool_id += 1
                while len(self.streamed_args_for_tool) <= self.current_tool_id:
                    self.streamed_args_for_tool.append("")

                calls.append(
                    ToolCallItem(
                        tool_index=self.current_tool_id,
                        name=tool_name,
                        parameters="",
                    )
                )

                self._buffer = self._buffer[sep_pos + len(self.tool_sep_token) :]
```
**EN:** This block continues `_continue_streaming` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `get`, `warning`, `append`, `ToolCallItem`.
**CN:** 这一段延续了 `_continue_streaming` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `get`、`warning`、`append`、`ToolCallItem`。

### Lines 349-358: Method `_continue_streaming` logic (part 3)
```python

            # Phase 2: stream argument JSON of the current tool.
            before_name = self._streaming_tool_name
            calls.extend(self._stream_args(tools))
            if self._streaming_tool_name is not None:
                break  # current tool still open; need more data.
            if self._streaming_tool_name == before_name:
                break  # safety: avoid infinite loop if state didn't advance.

        return StreamingParseResult(normal_text=leading_normal, calls=calls)
```
**EN:** This block continues `_continue_streaming` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `extend`, `_stream_args`, `StreamingParseResult`.
**CN:** 这一段延续了 `_continue_streaming` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `extend`、`_stream_args`、`StreamingParseResult`。

### Lines 360-361: Method `_stream_args` signature and setup
```python
    def _stream_args(self, tools: List[Tool]) -> List[ToolCallItem]:
        """Emit argument-JSON deltas for the currently-open tool call."""
```
**EN:** This block defines the method `_stream_args` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `_stream_args`.
**CN:** 这一段定义了method `_stream_args`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_stream_args`。

### Lines 362-399: Method `_stream_args` logic (part 1)
```python
        is_complete = self.tool_call_end_token in self._buffer

        if is_complete:
            end_idx = self._buffer.find(self.tool_call_end_token)
            args_text = self._buffer[:end_idx]
        else:
            args_text = self._buffer

        # 1. Absorb closed <arg_key>..<arg_value> pairs.
        last_closed_end = 0
        for m in self.func_args_regex.finditer(args_text):
            key, value = m.groups()
            key = key.strip()
            if key not in self._completed_args:
                self._completed_args[key] = self._parse_value(
                    value, self._streaming_tool_name or "", key, tools
                )
            last_closed_end = m.end()

        # 2. Detect a partial (unclosed) kv pair at the tail.
        tail = args_text[last_closed_end:]
        partial_key: Optional[str] = None
        partial_value: Optional[str] = None

        ak_start = tail.find(self.arg_key_start_token)
        if ak_start != -1:
            ak_end = tail.find(
                self.arg_key_end_token, ak_start + len(self.arg_key_start_token)
            )
            if ak_end != -1:
                partial_key = tail[
                    ak_start + len(self.arg_key_start_token) : ak_end
                ].strip()
                av_start = tail.find(self.arg_value_start_token, ak_end)
                if av_start != -1 and self._is_only_string_type(
                    self._streaming_tool_name or "", partial_key, tools
                ):
                    partial_value = tail[av_start + len(self.arg_value_start_token) :]
```
**EN:** This block continues `_stream_args` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `find`, `finditer`, `groups`, `strip`.
**CN:** 这一段延续了 `_stream_args` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`finditer`、`groups`、`strip`。

### Lines 400-429: Method `_stream_args` logic (part 2)
```python

        # Avoid emitting a lone "{" before any arg content is knowable.
        if not is_complete and not self._completed_args and partial_value is None:
            return []

        # 3. Build the JSON snapshot manually to control streaming boundaries.
        snapshot_parts: List[str] = []
        for k, v in self._completed_args.items():
            k_json = json.dumps(k, ensure_ascii=False)
            v_json = json.dumps(v, ensure_ascii=False)
            snapshot_parts.append(f"{k_json}: {v_json}")

        if partial_key is not None and partial_value is not None:
            # Hold back chars that could be a partial </arg_value> marker so
            # that a `<` starting the end-tag doesn't leak into the streamed
            # JSON string value.
            hold = self._ends_with_partial_token(
                partial_value, self.arg_value_end_token
            )
            safe_value = partial_value[:-hold] if hold else partial_value
            k_json = json.dumps(partial_key, ensure_ascii=False)
            escaped = (
                safe_value.replace("\\", "\\\\")
                .replace('"', '\\"')
                .replace("\n", "\\n")
                .replace("\r", "\\r")
                .replace("\t", "\\t")
            )
            # No closing `"` here — it's appended when the value closes.
            snapshot_parts.append(f'{k_json}: "{escaped}')
```
**EN:** This block continues `_stream_args` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `items`, `dumps`, `append`, `_ends_with_partial_token`.
**CN:** 这一段延续了 `_stream_args` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `items`、`dumps`、`append`、`_ends_with_partial_token`。

### Lines 430-456: Method `_stream_args` logic (part 3)
```python

        snapshot = "{" + ", ".join(snapshot_parts) + "}"

        argument_diff: Optional[str] = None

        if is_complete:
            final_json = json.dumps(self._completed_args, ensure_ascii=False)
            if self._streamed_json_len < len(final_json):
                argument_diff = final_json[self._streamed_json_len :]
            self._streamed_json_len = len(final_json)

            while len(self.prev_tool_call_arr) <= self.current_tool_id:
                self.prev_tool_call_arr.append({})
            self.prev_tool_call_arr[self.current_tool_id] = {
                "name": self._streaming_tool_name,
                "arguments": dict(self._completed_args),
            }

            end_idx = self._buffer.find(self.tool_call_end_token)
            self._buffer = self._buffer[end_idx + len(self.tool_call_end_token) :]
            self._reset_streaming_tool_state()
        else:
            # Withhold the trailing "}" while the tool call is still open.
            end = len(snapshot) - 1
            if end > self._streamed_json_len:
                argument_diff = snapshot[self._streamed_json_len : end]
                self._streamed_json_len = end
```
**EN:** This block continues `_stream_args` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `join`, `dumps`, `append`, `find`.
**CN:** 这一段延续了 `_stream_args` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `join`、`dumps`、`append`、`find`。

### Lines 457-466: Method `_stream_args` logic (part 4)
```python

        if argument_diff:
            self.streamed_args_for_tool[self.current_tool_id] += argument_diff
            return [
                ToolCallItem(
                    tool_index=self.current_tool_id,
                    parameters=argument_diff,
                )
            ]
        return []
```
**EN:** This block continues `_stream_args` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hunyuan style tool-call detection and normalization workflow. Notable operations include `ToolCallItem`.
**CN:** 这一段延续了 `_stream_args` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的混元风格的工具调用检测与规范化工作流。 值得注意的操作包括 `ToolCallItem`。

### Lines 468-473: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        return lambda name: StructureInfo(
            begin=f"<tool_calls>\n<tool_call>{name}<tool_sep>",
            end="</tool_call>\n</tool_calls>",
            trigger="<tool_calls>",
        )
```
**EN:** This block defines the method `structure_info` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `StructureInfo`。

### Lines 475-476: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        return False
```
**EN:** This block defines the method `supports_structural_tag` on `HunyuanDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hunyuan style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `HunyuanDetector`），介绍了参数、初始化步骤，以及这部分混元风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。

## Key Concepts / 关键概念
- `HunyuanDetector`: Class that encapsulates hunyuan detector behavior in this module. / `HunyuanDetector`：封装与“混元检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.environ`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
