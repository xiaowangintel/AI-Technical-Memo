# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file collects shared helper functions for schema handling, JSON validation, and text-level parsing utilities. The detector implementations reuse these helpers to stay consistent. / 该文件汇总了模式处理、JSON 校验以及文本级解析相关的共享辅助函数，各检测器实现通过复用这些工具来保持一致性。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and module setup
```python
from json import JSONDecodeError, JSONDecoder
from json.decoder import WHITESPACE
from typing import Any, Dict, List, Literal, Optional, Tuple, Union

import orjson
import partial_json_parser
from partial_json_parser.core.options import Allow

from sglang.srt.entrypoints.openai.protocol import Tool, ToolChoice


```
**EN:** This block gathers the imports and module-level setup for function-call schema and JSON helpers. The imported modules show which runtime services, schemas, or backend components this file depends on.
**CN:** 这一段汇集了与函数调用模式与 JSON 辅助工具相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。

### Lines 12-20: Function `_find_common_prefix`
```python
def _find_common_prefix(s1: str, s2: str) -> str:
    prefix = ""
    min_length = min(len(s1), len(s2))
    for i in range(0, min_length):
        if s1[i] == s2[i]:
            prefix += s1[i]
        else:
            break
    return prefix
```
**EN:** This block defines the function `_find_common_prefix`. It introduces the parameters, setup steps, and the main entry point for this piece of function-call schema and JSON helpers. Definitions introduced here include `_find_common_prefix`.
**CN:** 这一段定义了function `_find_common_prefix`，介绍了参数、初始化步骤，以及这部分函数调用模式与 JSON 辅助工具逻辑的主要入口。 此处引入的定义包括 `_find_common_prefix`。

### Lines 23-40: Function `_partial_json_loads` signature and setup
```python
def _partial_json_loads(input_str: str, flags: Allow) -> Tuple[Any, int]:
    """
    Parse incomplete or partial JSON strings commonly encountered during streaming.

    Args:
        input_str (str): The potentially incomplete JSON string to parse.
        flags (Allow): Bitwise flags controlling what types of partial data are allowed.
            Common flags include:
            - Allow.STR: Allow partial strings (e.g., '"hello wo' -> 'hello wo')
            - Allow.OBJ: Allow partial objects (e.g., '{"key":' -> {'key': None})
            - Allow.ARR: Allow partial arrays (e.g., '[1, 2,' -> [1, 2])
            - Allow.ALL: Allow all types of partial data

    Returns:
        Tuple[Any, int]: A tuple containing:
            - parsed_object: The Python object parsed from the JSON
            - consumed_length: Number of characters consumed from input_str
    """
```
**EN:** This block defines the function `_partial_json_loads`. It introduces the parameters, setup steps, and the main entry point for this piece of function-call schema and JSON helpers. Definitions introduced here include `_partial_json_loads`. Notable operations include `input_str`, `flags`, `strings`, `objects`.
**CN:** 这一段定义了function `_partial_json_loads`，介绍了参数、初始化步骤，以及这部分函数调用模式与 JSON 辅助工具逻辑的主要入口。 此处引入的定义包括 `_partial_json_loads`。 值得注意的操作包括 `input_str`、`flags`、`strings`、`objects`。

### Lines 41-49: Function `_partial_json_loads` logic (part 1)
```python
    try:
        return (partial_json_parser.loads(input_str, flags), len(input_str))
    except (JSONDecodeError, IndexError) as e:
        msg = getattr(e, "msg", str(e))
        if "Extra data" in msg or "pop from empty list" in msg:
            start = WHITESPACE.match(input_str, 0).end()
            obj, end = JSONDecoder().raw_decode(input_str, start)
            return obj, end
        raise
```
**EN:** This block continues `_partial_json_loads` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding function-call schema and JSON helpers workflow. Notable operations include `loads`, `match`, `end`, `JSONDecoder`.
**CN:** 这一段延续了 `_partial_json_loads` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的函数调用模式与 JSON 辅助工具工作流。 值得注意的操作包括 `loads`、`match`、`end`、`JSONDecoder`。

### Lines 52-57: Function `_is_complete_json`
```python
def _is_complete_json(input_str: str) -> bool:
    try:
        orjson.loads(input_str)
        return True
    except JSONDecodeError:
        return False
```
**EN:** This block defines the function `_is_complete_json`. It introduces the parameters, setup steps, and the main entry point for this piece of function-call schema and JSON helpers. Definitions introduced here include `_is_complete_json`. Notable operations include `loads`.
**CN:** 这一段定义了function `_is_complete_json`，介绍了参数、初始化步骤，以及这部分函数调用模式与 JSON 辅助工具逻辑的主要入口。 此处引入的定义包括 `_is_complete_json`。 值得注意的操作包括 `loads`。

### Lines 60-72: Function `_get_tool_schema_defs` signature and setup
```python
def _get_tool_schema_defs(tools: List[Tool]) -> dict:
    """
    Get consolidated $defs from all tools, validating for conflicts.

    Args:
        tools: List of tools to process

    Returns:
        Dictionary of consolidated $defs from all tools

    Raises:
        ValueError: If conflicting $defs are found
    """
```
**EN:** This block defines the function `_get_tool_schema_defs`. It introduces the parameters, setup steps, and the main entry point for this piece of function-call schema and JSON helpers. Definitions introduced here include `_get_tool_schema_defs`.
**CN:** 这一段定义了function `_get_tool_schema_defs`，介绍了参数、初始化步骤，以及这部分函数调用模式与 JSON 辅助工具逻辑的主要入口。 此处引入的定义包括 `_get_tool_schema_defs`。

### Lines 73-87: Function `_get_tool_schema_defs` logic (part 1)
```python
    all_defs = {}
    for tool in tools:
        if tool.function.parameters is None:
            continue
        defs = tool.function.parameters.get("$defs", {})
        for def_name, def_schema in defs.items():
            if def_name in all_defs and all_defs[def_name] != def_schema:
                raise ValueError(
                    f"Tool definition '{def_name}' has "
                    "multiple schemas, which is not "
                    "supported."
                )
            else:
                all_defs[def_name] = def_schema
    return all_defs
```
**EN:** This block continues `_get_tool_schema_defs` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding function-call schema and JSON helpers workflow. Notable operations include `get`, `items`, `ValueError`.
**CN:** 这一段延续了 `_get_tool_schema_defs` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的函数调用模式与 JSON 辅助工具工作流。 值得注意的操作包括 `get`、`items`、`ValueError`。

### Lines 90-101: Function `_get_tool_schema`
```python
def _get_tool_schema(tool: Tool) -> dict:
    return {
        "properties": {
            "name": {"type": "string", "enum": [tool.function.name]},
            "parameters": (
                tool.function.parameters
                if tool.function.parameters
                else {"type": "object", "properties": {}}
            ),
        },
        "required": ["name", "parameters"],
    }
```
**EN:** This block defines the function `_get_tool_schema`. It introduces the parameters, setup steps, and the main entry point for this piece of function-call schema and JSON helpers. Definitions introduced here include `_get_tool_schema`.
**CN:** 这一段定义了function `_get_tool_schema`，介绍了参数、初始化步骤，以及这部分函数调用模式与 JSON 辅助工具逻辑的主要入口。 此处引入的定义包括 `_get_tool_schema`。

### Lines 104-121: Function `infer_type_from_json_schema` signature and setup
```python
def infer_type_from_json_schema(schema: Dict[str, Any]) -> Optional[str]:
    """
    Infer the primary type of a parameter from JSON Schema.

    Supports complex JSON Schema structures including:
    - Direct type field (including type arrays)
    - anyOf/oneOf: parameter can be any of multiple types
    - enum: parameter must be one of enum values
    - allOf: parameter must satisfy all type definitions
    - properties: inferred as object type
    - items: inferred as array type

    Args:
        schema: JSON Schema definition

    Returns:
        Inferred type ('string', 'number', 'object', 'array', etc.) or None
    """
```
**EN:** This block defines the function `infer_type_from_json_schema`. It introduces the parameters, setup steps, and the main entry point for this piece of function-call schema and JSON helpers. Definitions introduced here include `infer_type_from_json_schema`. Notable operations include `field`.
**CN:** 这一段定义了function `infer_type_from_json_schema`，介绍了参数、初始化步骤，以及这部分函数调用模式与 JSON 辅助工具逻辑的主要入口。 此处引入的定义包括 `infer_type_from_json_schema`。 值得注意的操作包括 `field`。

### Lines 122-156: Function `infer_type_from_json_schema` logic (part 1)
```python
    if not isinstance(schema, dict):
        return None

    # Priority 1: Direct type field (including type arrays)
    if "type" in schema:
        type_value = schema["type"]
        if isinstance(type_value, str):
            return type_value
        elif isinstance(type_value, list) and type_value:
            # Handle type arrays: return first non-null type
            non_null_types = [t for t in type_value if t != "null"]
            if non_null_types:
                return non_null_types[0]
            return "string"  # If only null, default to string

    # Priority 2: Handle anyOf/oneOf
    if "anyOf" in schema or "oneOf" in schema:
        schemas = schema.get("anyOf") or schema.get("oneOf")
        types = []

        if isinstance(schemas, list):
            for sub_schema in schemas:
                inferred_type = infer_type_from_json_schema(sub_schema)
                if inferred_type:
                    types.append(inferred_type)

            if types:
                # If all types are the same, return unified type
                if len(set(types)) == 1:
                    return types[0]
                # When types differ, prioritize string (safest)
                if "string" in types:
                    return "string"
                # Otherwise return first type
                return types[0]
```
**EN:** This block continues `infer_type_from_json_schema` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding function-call schema and JSON helpers workflow. Notable operations include `field`, `get`, `infer_type_from_json_schema`, `append`.
**CN:** 这一段延续了 `infer_type_from_json_schema` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的函数调用模式与 JSON 辅助工具工作流。 值得注意的操作包括 `field`、`get`、`infer_type_from_json_schema`、`append`。

### Lines 157-185: Function `infer_type_from_json_schema` logic (part 2)
```python

    # Priority 3: Handle enum (infer type from enum values)
    if "enum" in schema and isinstance(schema["enum"], list):
        if not schema["enum"]:
            return "string"

        # Infer type from enum values
        enum_types = set()
        for value in schema["enum"]:
            if value is None:
                enum_types.add("null")
            elif isinstance(value, bool):
                enum_types.add("boolean")
            elif isinstance(value, int):
                enum_types.add("integer")
            elif isinstance(value, float):
                enum_types.add("number")
            elif isinstance(value, str):
                enum_types.add("string")
            elif isinstance(value, list):
                enum_types.add("array")
            elif isinstance(value, dict):
                enum_types.add("object")

        # If type is uniform, return that type
        if len(enum_types) == 1:
            return enum_types.pop()
        # Mixed types, prioritize string
        return "string"
```
**EN:** This block continues `infer_type_from_json_schema` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding function-call schema and JSON helpers workflow. Notable operations include `enum`, `add`, `pop`.
**CN:** 这一段延续了 `infer_type_from_json_schema` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的函数调用模式与 JSON 辅助工具工作流。 值得注意的操作包括 `enum`、`add`、`pop`。

### Lines 186-204: Function `infer_type_from_json_schema` logic (part 3)
```python

    # Priority 4: Handle allOf (must satisfy all types)
    if "allOf" in schema and isinstance(schema["allOf"], list):
        schemas = schema["allOf"]
        for sub_schema in schemas:
            inferred_type = infer_type_from_json_schema(sub_schema)
            if inferred_type and inferred_type != "string":
                return inferred_type
        return "string"

    # Priority 5: Infer object type
    if "properties" in schema:
        return "object"

    # Priority 6: Infer array type
    if "items" in schema:
        return "array"

    return None
```
**EN:** This block continues `infer_type_from_json_schema` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding function-call schema and JSON helpers workflow. Notable operations include `allOf`, `infer_type_from_json_schema`.
**CN:** 这一段延续了 `infer_type_from_json_schema` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的函数调用模式与 JSON 辅助工具工作流。 值得注意的操作包括 `allOf`、`infer_type_from_json_schema`。

### Lines 207-221: Function `get_json_schema_constraint` signature and setup
```python
def get_json_schema_constraint(
    tools: List[Tool],
    tool_choice: Union[ToolChoice, Literal["required"]],
    parallel_tool_calls: bool = True,
) -> Optional[dict]:
    """
    Get the JSON schema constraint for the specified tool choice.

    Args:
        tool_choice: The tool choice specification
        parallel_tool_calls: If False, constrain to exactly one tool call (maxItems=1)

    Returns:
        JSON schema dict, or None if no valid tools found
    """
```
**EN:** This block defines the function `get_json_schema_constraint`. It introduces the parameters, setup steps, and the main entry point for this piece of function-call schema and JSON helpers. Definitions introduced here include `get_json_schema_constraint`. Notable operations include `call`.
**CN:** 这一段定义了function `get_json_schema_constraint`，介绍了参数、初始化步骤，以及这部分函数调用模式与 JSON 辅助工具逻辑的主要入口。 此处引入的定义包括 `get_json_schema_constraint`。 值得注意的操作包括 `call`。

### Lines 222-251: Function `get_json_schema_constraint` logic (part 1)
```python

    if isinstance(tool_choice, ToolChoice):
        # For specific function choice, return the user's parameters schema directly
        fn_name = tool_choice.function.name
        for tool in tools:
            if tool.function.name == fn_name:
                schema = {
                    "type": "array",
                    "minItems": 1,
                    "items": _get_tool_schema(tool),
                }
                if not parallel_tool_calls:
                    schema["maxItems"] = 1
                return schema
        return None
    elif tool_choice == "required":
        json_schema = {
            "type": "array",
            "minItems": 1,
            "items": {
                "type": "object",
                "anyOf": [_get_tool_schema(tool) for tool in tools],
            },
        }
        if not parallel_tool_calls:
            json_schema["maxItems"] = 1
        json_schema_defs = _get_tool_schema_defs(tools)
        if json_schema_defs:
            json_schema["$defs"] = json_schema_defs
        return json_schema
```
**EN:** This block continues `get_json_schema_constraint` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding function-call schema and JSON helpers workflow. Notable operations include `_get_tool_schema`, `_get_tool_schema_defs`.
**CN:** 这一段延续了 `get_json_schema_constraint` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的函数调用模式与 JSON 辅助工具工作流。 值得注意的操作包括 `_get_tool_schema`、`_get_tool_schema_defs`。

### Lines 252-253: Function `get_json_schema_constraint` logic (part 2)
```python

    return None
```
**EN:** This block continues `get_json_schema_constraint` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding function-call schema and JSON helpers workflow.
**CN:** 这一段延续了 `get_json_schema_constraint` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的函数调用模式与 JSON 辅助工具工作流。

## Key Concepts / 关键概念
- `_find_common_prefix`: Function that performs find common prefix for the surrounding workflow. / `_find_common_prefix`：在周边工作流中执行“find通用prefix”相关任务的函数。
- `_partial_json_loads`: Function that performs partial json loads for the surrounding workflow. / `_partial_json_loads`：在周边工作流中执行“partialJSONloads”相关任务的函数。
- `_is_complete_json`: Function that performs is complete json for the surrounding workflow. / `_is_complete_json`：在周边工作流中执行“iscompleteJSON”相关任务的函数。
- `_get_tool_schema_defs`: Function that performs get tool schema defs for the surrounding workflow. / `_get_tool_schema_defs`：在周边工作流中执行“get工具模式defs”相关任务的函数。
- `_get_tool_schema`: Function that performs get tool schema for the surrounding workflow. / `_get_tool_schema`：在周边工作流中执行“get工具模式”相关任务的函数。
- `infer_type_from_json_schema`: Function that performs infer type from json schema for the surrounding workflow. / `infer_type_from_json_schema`：在周边工作流中执行“infer类型fromJSON模式”相关任务的函数。
- `get_json_schema_constraint`: Function that performs get json schema constraint for the surrounding workflow. / `get_json_schema_constraint`：在周边工作流中执行“getJSON模式约束”相关任务的函数。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `typing`
- **External packages / 外部依赖**: `orjson`, `partial_json_parser`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`
