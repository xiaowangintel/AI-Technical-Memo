# poolside_v1_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/poolside_v1_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a poolside v 1 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 Poolsidev1 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and module setup
```python
import ast
import json
import re
from enum import Enum, auto
from typing import Any, List, Optional

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    StructureInfo,
    ToolCallItem,
    _GetInfoFunc,
)


```
**EN:** This block gathers the imports and module-level setup for poolside v 1 style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`.
**CN:** 这一段汇集了与Poolsidev1风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`。

### Lines 17-17: Class `_ParseState` declaration
```python
class _ParseState(Enum):
```
**EN:** This block declares the class `_ParseState` and establishes its responsibility inside poolside v 1 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `_ParseState`.
**CN:** 这一段声明了类 `_ParseState`，并说明它在Poolsidev1风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `_ParseState`。

### Lines 18-36: Supporting state inside `_ParseState`
```python
    """5 FSM states for the streaming parser.

    Entry guard: READING_VALUE is reachable only from READING_KEY, so the
    "stray <arg_value> before <tool_call>" bug class is structurally
    impossible.

    Exit guard: both READING_KEY and READING_VALUE recover on `</tool_call>`
    by closing the active call (orphan key dropped if any). READING_VALUE
    additionally recovers on `<arg_key>` by replacing the orphan pending key
    with the new one. Both guards match the (regex-tightened) non-streaming
    path. Without them, malformed inputs would leave the FSM stuck in
    READING_VALUE and mis-attribute subsequent values to stale state.
    """

    OUTSIDE = auto()
    READING_NAME = auto()
    READING_KEY = auto()
    READING_VALUE = auto()
    DRAINING = auto()
```
**EN:** This block adds supporting state or helper logic inside `_ParseState`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `call`, `the`, `auto`.
**CN:** 这一段为 `_ParseState` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `call`、`the`、`auto`。

### Lines 39-39: Class `PoolsideV1Detector` declaration
```python
class PoolsideV1Detector(BaseFormatDetector):
```
**EN:** This block declares the class `PoolsideV1Detector` and establishes its responsibility inside poolside v 1 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `PoolsideV1Detector`.
**CN:** 这一段声明了类 `PoolsideV1Detector`，并说明它在Poolsidev1风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `PoolsideV1Detector`。

### Lines 40-76: Supporting state inside `PoolsideV1Detector`
```python
    """
    Detector for poolside Laguna-XS.2 (poolside_v1 series) tool-call wire format.

    Wire format:
        <tool_call>{name}\\n
        <arg_key>{key}</arg_key>\\n
        <arg_value>{val}</arg_value>\\n
        ...
        </tool_call>

    String values are emitted as raw text; non-strings are JSON-encoded by
    the chat template. The parser does schema-based type coercion to round-trip
    them: schema type `string` keeps the raw value; other types attempt
    `json.loads` and fall back to `ast.literal_eval`, then to the raw string.
    """

    # Wire-format tag tokens — constants, not per-instance.
    tool_call_start_token = "<tool_call>"
    tool_call_end_token = "</tool_call>"
    arg_key_start = "<arg_key>"
    arg_key_end = "</arg_key>"
    arg_value_start = "<arg_value>"
    arg_value_end = "</arg_value>"

    tool_call_regex = re.compile(r"<tool_call>(.*?)</tool_call>", re.DOTALL)
    # Key uses [^<]*? to prevent the non-greedy `.*?` from backtracking
    # across an `</arg_key>` boundary on malformed inputs like
    # `<arg_key>K1</arg_key><arg_key>K2</arg_key><arg_value>V</arg_value>`
    # — without the `[^<]` constraint, the regex matches the entire orphan
    # span as a single key (`K1</arg_key><arg_key>K2`). Param names never
    # contain `<` in practice, so this is safe. The value side keeps `.*?`
    # because legitimate values can contain `<` (HTML, paths, etc.); the
    # `</arg_value>` boundary is anchored enough.
    arg_pair_regex = re.compile(
        r"<arg_key>([^<]*?)</arg_key>\s*<arg_value>(.*?)</arg_value>",
        re.DOTALL,
    )
```
**EN:** This block adds supporting state or helper logic inside `PoolsideV1Detector`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `compile`, `key`.
**CN:** 这一段为 `PoolsideV1Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `compile`、`key`。

### Lines 77-86: Supporting state inside `PoolsideV1Detector`
```python

    _partial_tag_prefixes = (
        tool_call_start_token,
        tool_call_end_token,
        arg_key_start,
        arg_key_end,
        arg_value_start,
        arg_value_end,
    )

```
**EN:** This block adds supporting state or helper logic inside `PoolsideV1Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `PoolsideV1Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 87-93: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.parsed_pos: int = 0
        self._state: _ParseState = _ParseState.OUTSIDE
        self.current_func_name: Optional[str] = None
        self.current_pending_key: Optional[str] = None
        self.json_started: bool = False
```
**EN:** This block defines the method `__init__` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 94-96: Supporting state inside `PoolsideV1Detector`
```python

    # ---------- Helpers ----------

```
**EN:** This block adds supporting state or helper logic inside `PoolsideV1Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `PoolsideV1Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 97-102: Method `_reset_call_state`
```python
    def _reset_call_state(self) -> None:
        """Reset per-call FSM scratch fields. Called when entering a new
        <tool_call> and on </tool_call> close."""
        self.current_func_name = None
        self.current_pending_key = None
        self.json_started = False
```
**EN:** This block defines the method `_reset_call_state` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `_reset_call_state`.
**CN:** 这一段定义了method `_reset_call_state`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_reset_call_state`。

### Lines 104-115: Method `_consume_arg_key`
```python
    def _consume_arg_key(self, slice_: str) -> bool:
        """Consume `<arg_key>K</arg_key>`, set `current_pending_key` to K.
        Returns True if consumed, False if `</arg_key>` hasn't arrived yet
        (caller should break to wait for more bytes). Shared by READING_KEY
        (well-formed: transitions to READING_VALUE) and READING_VALUE
        (orphan-key-replace: stays in READING_VALUE)."""
        end = slice_.find(self.arg_key_end)
        if end == -1:
            return False
        self.current_pending_key = slice_[len(self.arg_key_start) : end].strip()
        self.parsed_pos += end + len(self.arg_key_end)
        return True
```
**EN:** This block defines the method `_consume_arg_key` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `_consume_arg_key`. Notable operations include `yet`, `READING_KEY`, `READING_VALUE`, `find`.
**CN:** 这一段定义了method `_consume_arg_key`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_consume_arg_key`。 值得注意的操作包括 `yet`、`READING_KEY`、`READING_VALUE`、`find`。

### Lines 117-135: Method `_close_current_call`
```python
    def _close_current_call(self, calls: List[ToolCallItem]) -> None:
        """Emit the closing `}` (or `{}` for zero-arg) for the active call,
        advance past `</tool_call>`, return to OUTSIDE, and reset per-call
        state. Called from both READING_KEY (the well-formed close path) and
        READING_VALUE (malformed close: `<arg_key>...</arg_key></tool_call>`
        with no value — orphan key is discarded, matching the regex
        non-streaming path which drops unmatched <arg_key>...</arg_key>
        pairs)."""
        fragment = "}" if self.json_started else "{}"
        calls.append(
            ToolCallItem(
                tool_index=self.current_tool_id,
                parameters=fragment,
            )
        )
        self.streamed_args_for_tool[self.current_tool_id] += fragment
        self.parsed_pos += len(self.tool_call_end_token)
        self._state = _ParseState.OUTSIDE
        self._reset_call_state()
```
**EN:** This block defines the method `_close_current_call` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `_close_current_call`. Notable operations include `READING_KEY`, `READING_VALUE`, `append`, `ToolCallItem`.
**CN:** 这一段定义了method `_close_current_call`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_close_current_call`。 值得注意的操作包括 `READING_KEY`、`READING_VALUE`、`append`、`ToolCallItem`。

### Lines 137-138: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        return self.tool_call_start_token in text
```
**EN:** This block defines the method `has_tool_call` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 140-156: Method `_get_param_schema`
```python
    @staticmethod
    def _get_param_schema(
        func_name: Optional[str], tools: Optional[List[Tool]]
    ) -> dict:
        if not tools or not func_name:
            return {}
        for tool in tools:
            try:
                if (
                    tool.type == "function"
                    and tool.function.name == func_name
                    and isinstance(tool.function.parameters, dict)
                ):
                    return tool.function.parameters.get("properties", {})
            except AttributeError:
                continue
        return {}
```
**EN:** This block defines the method `_get_param_schema` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `_get_param_schema`. Notable operations include `get`.
**CN:** 这一段定义了method `_get_param_schema`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_get_param_schema`。 值得注意的操作包括 `get`。

### Lines 157-159: Supporting state inside `PoolsideV1Detector`
```python

    _STRING_TYPES = frozenset({"string", "str", "text", "enum"})

```
**EN:** This block adds supporting state or helper logic inside `PoolsideV1Detector`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `frozenset`.
**CN:** 这一段为 `PoolsideV1Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `frozenset`。

### Lines 160-176: Method `_convert_param_value` signature and setup
```python
    @staticmethod
    def _convert_param_value(raw: str, schema: dict, key: str) -> Any:
        """Coerce a raw arg_value string per schema; fall back to raw on failure.

        Decoder selection by schema type:
          - string-like types         → identity (raw text)
          - no schema entry           → json.loads only (conservative; don't
                                        ast-eval untyped values)
          - everything else (int,
            number, bool, object, …)  → json.loads, then ast.literal_eval

        Each decoder result is round-tripped through `json.dumps` before being
        returned; non-JSON-serializable values (sets / complex / bytes from
        `ast.literal_eval`) are rejected to the next decoder, ultimately
        falling through to the raw-string fallback rather than crashing the
        streaming JSON emission downstream.
        """
```
**EN:** This block defines the method `_convert_param_value` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `_convert_param_value`. Notable operations include `identity`, `only`, `else`, `values`.
**CN:** 这一段定义了method `_convert_param_value`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_convert_param_value`。 值得注意的操作包括 `identity`、`only`、`else`、`values`。

### Lines 177-193: Method `_convert_param_value` logic (part 1)
```python
        spec = schema.get(key) if isinstance(schema, dict) else None
        param_type = str(spec.get("type", "")).lower() if isinstance(spec, dict) else ""
        if param_type in PoolsideV1Detector._STRING_TYPES:
            return raw

        decoders = (json.loads,) if not param_type else (json.loads, ast.literal_eval)
        for decoder in decoders:
            try:
                result = decoder(raw)
                # ast.literal_eval can return non-JSON-serializable values
                # (sets, complex numbers); reject so json.dumps downstream
                # doesn't choke.
                json.dumps(result)
                return result
            except (ValueError, SyntaxError, TypeError):
                continue
        return raw
```
**EN:** This block continues `_convert_param_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding poolside v 1 style tool-call detection and normalization workflow. Notable operations include `get`, `lower`, `else`, `decoder`.
**CN:** 这一段延续了 `_convert_param_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Poolsidev1风格的工具调用检测与规范化工作流。 值得注意的操作包括 `get`、`lower`、`else`、`decoder`。

### Lines 195-203: Method `_find_name_boundary`
```python
    def _find_name_boundary(self, text: str) -> int:
        """Earliest of `\\n`, `<arg_key>`, `</tool_call>`. -1 if none."""
        hits = (
            text.find("\n"),
            text.find(self.arg_key_start),
            text.find(self.tool_call_end_token),
        )
        positive = [h for h in hits if h != -1]
        return min(positive) if positive else -1
```
**EN:** This block defines the method `_find_name_boundary` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `_find_name_boundary`. Notable operations include `find`.
**CN:** 这一段定义了method `_find_name_boundary`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_find_name_boundary`。 值得注意的操作包括 `find`。

### Lines 205-211: Method `_is_partial_tag`
```python
    def _is_partial_tag(self, slice_: str) -> bool:
        """True if slice_ is a strict prefix of any known tag — i.e. more
        bytes might complete it into a real tag."""
        return any(
            tag.startswith(slice_) and tag != slice_
            for tag in self._partial_tag_prefixes
        )
```
**EN:** This block defines the method `_is_partial_tag` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `_is_partial_tag`. Notable operations include `startswith`.
**CN:** 这一段定义了method `_is_partial_tag`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_is_partial_tag`。 值得注意的操作包括 `startswith`。

### Lines 212-214: Supporting state inside `PoolsideV1Detector`
```python

    # ---------- Non-streaming ----------

```
**EN:** This block adds supporting state or helper logic inside `PoolsideV1Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `PoolsideV1Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 215-217: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        if self.tool_call_start_token not in text:
            return StreamingParseResult(normal_text=text)
```
**EN:** This block defines the method `detect_and_parse` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`. Notable operations include `StreamingParseResult`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `StreamingParseResult`。

### Lines 218-243: Method `detect_and_parse` logic (part 1)
```python

        tool_indices = self._get_tool_indices(tools)
        first_idx = text.find(self.tool_call_start_token)
        normal_text = text[:first_idx] if first_idx > 0 else ""

        calls: List[ToolCallItem] = []
        for body in self.tool_call_regex.findall(text):
            # _find_name_boundary searches for `\n` / `<arg_key>` /
            # `</tool_call>`, but the regex already stripped `</tool_call>`,
            # so a no-arg call without a trailing newline
            # (`<tool_call>now</tool_call>`) gives boundary == -1. Treat
            # that case as "name == entire body".
            boundary = self._find_name_boundary(body)
            name = (body if boundary == -1 else body[:boundary]).strip()
            if not name or name not in tool_indices:
                continue

            schema = self._get_param_schema(name, tools)
            args: dict = {}
            for raw_key, raw_val in self.arg_pair_regex.findall(body):
                key = raw_key.strip()
                # Strip at most one wrapping `\n` on each side (template adds
                # them around the value); preserve newlines that are part of
                # the value itself.
                val = raw_val.removeprefix("\n").removesuffix("\n")
                args[key] = self._convert_param_value(val, schema, key)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding poolside v 1 style tool-call detection and normalization workflow. Notable operations include `_get_tool_indices`, `find`, `findall`, `_find_name_boundary`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Poolsidev1风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_get_tool_indices`、`find`、`findall`、`_find_name_boundary`。

### Lines 244-253: Method `detect_and_parse` logic (part 2)
```python

            calls.append(
                ToolCallItem(
                    tool_index=tool_indices[name],
                    name=name,
                    parameters=json.dumps(args, ensure_ascii=False),
                )
            )

        return StreamingParseResult(normal_text=normal_text, calls=calls)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding poolside v 1 style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `dumps`, `StreamingParseResult`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Poolsidev1风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`dumps`、`StreamingParseResult`。

### Lines 254-256: Supporting state inside `PoolsideV1Detector`
```python

    # ---------- Streaming ----------

```
**EN:** This block adds supporting state or helper logic inside `PoolsideV1Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `PoolsideV1Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 257-259: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
```
**EN:** This block defines the method `parse_streaming_increment` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 260-294: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text
        if not self._buffer:
            return StreamingParseResult()

        tool_indices = self._get_tool_indices(tools)
        calls: List[ToolCallItem] = []
        normal_text_chunks: List[str] = []

        # No try/except: the FSM's invariants make the prior masked-IndexError
        # class unreachable, and TypeError from json.dumps is prevented at the
        # source (_convert_param_value round-trips its decoder output). If a
        # real bug surfaces, let it surface.
        while True:
            slice_ = self._buffer[self.parsed_pos :]
            if not slice_:
                break
            state = self._state

            if state is _ParseState.OUTSIDE:
                if slice_.startswith(self.tool_call_start_token):
                    self.parsed_pos += len(self.tool_call_start_token)
                    self._state = _ParseState.READING_NAME
                    self._reset_call_state()
                    continue
                if slice_.startswith("<"):
                    if self._is_partial_tag(slice_):
                        break  # could be a partial <tool_call>
                    normal_text_chunks.append("<")
                    self.parsed_pos += 1
                    continue
                next_lt = slice_.find("<")
                segment = slice_ if next_lt == -1 else slice_[:next_lt]
                normal_text_chunks.append(segment)
                self.parsed_pos += len(segment)
                continue
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding poolside v 1 style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `_get_tool_indices`, `source`, `startswith`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Poolsidev1风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`_get_tool_indices`、`source`、`startswith`。

### Lines 295-332: Method `parse_streaming_increment` logic (part 2)
```python

            if state is _ParseState.READING_NAME:
                boundary = self._find_name_boundary(slice_)
                if boundary == -1:
                    break  # name still incoming
                name = slice_[:boundary].strip()
                # Consume the name and a single delimiting newline (if
                # present). The other boundary types (<arg_key>,
                # </tool_call>) are left for the next state. boundary may
                # be 0 for a malformed `<tool_call><arg_key>...` (no
                # name); the state transition below is the loop-progress
                # guarantee.
                consume = boundary
                if boundary < len(slice_) and slice_[boundary : boundary + 1] == "\n":
                    consume += 1
                self.parsed_pos += consume

                if name and name in tool_indices:
                    self.current_tool_id += 1
                    while len(self.streamed_args_for_tool) <= self.current_tool_id:
                        self.streamed_args_for_tool.append("")
                    self.current_func_name = name
                    # Per-response sequential index — OpenAI clients group
                    # chunks by tool_index, so the name event and later
                    # parameter fragments must share this value.
                    calls.append(
                        ToolCallItem(
                            tool_index=self.current_tool_id,
                            name=name,
                            parameters="",
                        )
                    )
                    self._state = _ParseState.READING_KEY
                else:
                    # Unknown / empty name — drain to </tool_call> with no
                    # client-visible emission.
                    self._state = _ParseState.DRAINING
                continue
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding poolside v 1 style tool-call detection and normalization workflow. Notable operations include `_find_name_boundary`, `strip`, `newline`, `types`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Poolsidev1风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_find_name_boundary`、`strip`、`newline`、`types`。

### Lines 333-364: Method `parse_streaming_increment` logic (part 3)
```python

            if state is _ParseState.READING_KEY:
                if slice_.startswith(self.tool_call_end_token):
                    self._close_current_call(calls)
                    continue
                if slice_.startswith(self.arg_key_start):
                    if not self._consume_arg_key(slice_):
                        break  # incomplete <arg_key>
                    self._state = _ParseState.READING_VALUE
                    continue
                if slice_.startswith("<"):
                    if self._is_partial_tag(slice_):
                        break
                    # Bare '<' that's not any known tag — discard silently
                    # (inside a tool call, this is not normal_text).
                    self.parsed_pos += 1
                    continue
                # Inter-tag whitespace / newline — discard.
                next_lt = slice_.find("<")
                self.parsed_pos += len(slice_) if next_lt == -1 else next_lt
                continue

            if state is _ParseState.READING_VALUE:
                # Recover from a malformed `<arg_key>K</arg_key></tool_call>`
                # (no <arg_value>) by closing the call here. Without this
                # branch the FSM would stay stuck in READING_VALUE and
                # mis-attribute the next call's <arg_value> to the orphan
                # `current_pending_key`, silently swallowing the next call's
                # name. Matches the regex non-streaming path, which drops
                # unmatched <arg_key>...</arg_key> pairs.
                if slice_.startswith(self.tool_call_end_token):
                    self._close_current_call(calls)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding poolside v 1 style tool-call detection and normalization workflow. Notable operations include `startswith`, `_close_current_call`, `_consume_arg_key`, `_is_partial_tag`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Poolsidev1风格的工具调用检测与规范化工作流。 值得注意的操作包括 `startswith`、`_close_current_call`、`_consume_arg_key`、`_is_partial_tag`。

### Lines 365-396: Method `parse_streaming_increment` logic (part 4)
```python
                    continue
                # Recover from a malformed `<arg_key>K1</arg_key><arg_key>K2`
                # (no value for K1, model went straight to a new key) by
                # replacing the orphan pending_key with the new one. Stays
                # in READING_VALUE so the next <arg_value> binds to K2.
                # Without this branch the FSM treats the second <arg_key>
                # as bare-`<` garbage and the next <arg_value> binds to
                # the stale K1 — wrong-argument corruption.
                if slice_.startswith(self.arg_key_start):
                    if not self._consume_arg_key(slice_):
                        break  # incomplete <arg_key>
                    continue  # stay in READING_VALUE: orphan replaced
                if slice_.startswith(self.arg_value_start):
                    end = slice_.find(self.arg_value_end)
                    if end == -1:
                        break  # incomplete <arg_value> — no partial emission
                    raw = (
                        slice_[len(self.arg_value_start) : end]
                        .removeprefix("\n")
                        .removesuffix("\n")
                    )
                    # READING_VALUE is reachable only via READING_KEY
                    # consuming an <arg_key>...</arg_key>, so
                    # current_pending_key is set by construction.
                    schema = self._get_param_schema(self.current_func_name, tools)
                    converted = self._convert_param_value(
                        raw, schema, self.current_pending_key
                    )
                    kv = (
                        f"{json.dumps(self.current_pending_key)}: "
                        f"{json.dumps(converted, ensure_ascii=False)}"
                    )
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding poolside v 1 style tool-call detection and normalization workflow. Notable operations include `startswith`, `_consume_arg_key`, `find`, `removeprefix`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Poolsidev1风格的工具调用检测与规范化工作流。 值得注意的操作包括 `startswith`、`_consume_arg_key`、`find`、`removeprefix`。

### Lines 397-431: Method `parse_streaming_increment` logic (part 5)
```python
                    fragment = "{" + kv if not self.json_started else ", " + kv
                    self.json_started = True
                    calls.append(
                        ToolCallItem(
                            tool_index=self.current_tool_id,
                            parameters=fragment,
                        )
                    )
                    self.streamed_args_for_tool[self.current_tool_id] += fragment
                    self.current_pending_key = None
                    self.parsed_pos += end + len(self.arg_value_end)
                    self._state = _ParseState.READING_KEY
                    continue
                if slice_.startswith("<"):
                    if self._is_partial_tag(slice_):
                        break
                    self.parsed_pos += 1
                    continue
                next_lt = slice_.find("<")
                self.parsed_pos += len(slice_) if next_lt == -1 else next_lt
                continue

            if state is _ParseState.DRAINING:
                end_idx = slice_.find(self.tool_call_end_token)
                if end_idx != -1:
                    self.parsed_pos += end_idx + len(self.tool_call_end_token)
                    self._state = _ParseState.OUTSIDE
                    continue
                # Hold back trailing bytes that could be a prefix of
                # </tool_call>; the next chunk extends the tail.
                holdback = self._ends_with_partial_token(
                    slice_, self.tool_call_end_token
                )
                self.parsed_pos += len(slice_) - holdback
                break
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding poolside v 1 style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `startswith`, `_is_partial_tag`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Poolsidev1风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`startswith`、`_is_partial_tag`。

### Lines 432-440: Method `parse_streaming_increment` logic (part 6)
```python

        if self.parsed_pos > 0:
            self._buffer = self._buffer[self.parsed_pos :]
            self.parsed_pos = 0

        return StreamingParseResult(
            calls=calls,
            normal_text="".join(normal_text_chunks),
        )
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding poolside v 1 style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `join`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Poolsidev1风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`join`。

### Lines 441-443: Supporting state inside `PoolsideV1Detector`
```python

    # ---------- Constrained generation ----------

```
**EN:** This block adds supporting state or helper logic inside `PoolsideV1Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `PoolsideV1Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 444-445: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        return False
```
**EN:** This block defines the method `supports_structural_tag` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。

### Lines 447-452: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        return lambda name: StructureInfo(
            begin=f"<tool_call>{name}\n",
            end="</tool_call>",
            trigger="<tool_call>",
        )
```
**EN:** This block defines the method `structure_info` on `PoolsideV1Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of poolside v 1 style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `PoolsideV1Detector`），介绍了参数、初始化步骤，以及这部分Poolsidev1风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `StructureInfo`。

## Key Concepts / 关键概念
- `_ParseState`: Class that encapsulates parse state behavior in this module. / `_ParseState`：封装与“解析状态”相关行为的类。
- `PoolsideV1Detector`: Class that encapsulates poolside v 1 detector behavior in this module. / `PoolsideV1Detector`：封装与“Poolsidev1检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ast`, `json`, `re`, `enum`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
