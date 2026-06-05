# base_format_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/base_format_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a base format detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 基础格式 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Imports and module setup
```python
import json
import logging
from abc import ABC, abstractmethod
from typing import Any, Dict, List, Literal, Optional, Union

import orjson
from partial_json_parser.core.exceptions import MalformedJSON
from partial_json_parser.core.options import Allow

try:
    from xgrammar import StructuralTag, get_model_structural_tag
except ImportError:
    StructuralTag = Any
    get_model_structural_tag = None

from sglang.srt.entrypoints.openai.protocol import Tool, ToolChoice
from sglang.srt.environ import envs
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    ToolCallItem,
    _GetInfoFunc,
)
from sglang.srt.function_call.utils import (
    _find_common_prefix,
    _is_complete_json,
    _partial_json_loads,
)

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for base format style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与基础格式风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 32-32: Class `BaseFormatDetector` declaration
```python
class BaseFormatDetector(ABC):
```
**EN:** This block declares the class `BaseFormatDetector` and establishes its responsibility inside base format style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `BaseFormatDetector`.
**CN:** 这一段声明了类 `BaseFormatDetector`，并说明它在基础格式风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `BaseFormatDetector`。

### Lines 33-34: Supporting state inside `BaseFormatDetector`
```python
    """Base class providing two sets of interfaces: one-time and streaming incremental."""

```
**EN:** This block adds supporting state or helper logic inside `BaseFormatDetector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `BaseFormatDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 35-57: Method `__init__`
```python
    def __init__(self):
        # Streaming state management
        # Buffer for accumulating incomplete patterns that arrive across multiple streaming chunks
        self._buffer = ""
        # Stores complete tool call info (name and arguments) for each tool being parsed.
        # Used by serving layer for completion handling when streaming ends.
        # Format: [{"name": str, "arguments": dict}, ...]
        self.prev_tool_call_arr: List[Dict] = []
        # Index of currently streaming tool call. Starts at -1 (no active tool),
        # increments as each tool completes. Tracks which tool's arguments are streaming.
        self.current_tool_id: int = -1
        # Flag for whether current tool's name has been sent to client.
        # Tool names sent first with empty parameters, then arguments stream incrementally.
        self.current_tool_name_sent: bool = False
        # Tracks raw JSON string content streamed to client for each tool's arguments.
        # Critical for serving layer to calculate remaining content when streaming ends.
        # Each index corresponds to a tool_id. Example: ['{"location": "San Francisco"', '{"temp": 72']
        self.streamed_args_for_tool: List[str] = []

        # Token configuration (override in subclasses)
        self.bot_token = ""
        self.eot_token = ""
        self.tool_call_separator = ", "
```
**EN:** This block defines the method `__init__` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `info`, `configuration`.
**CN:** 这一段定义了method `__init__`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `info`、`configuration`。

### Lines 59-75: Method `_get_tool_indices`
```python
    def _get_tool_indices(self, tools: List[Tool]) -> Dict[str, int]:
        """
        Get a mapping of tool names to their indices in the tools list.

        This utility method creates a dictionary mapping function names to their
        indices in the tools list, which is commonly needed for tool validation
        and ToolCallItem creation.

        Args:
            tools: List of available tools

        Returns:
            Dictionary mapping tool names to their indices
        """
        return {
            tool.function.name: i for i, tool in enumerate(tools) if tool.function.name
        }
```
**EN:** This block defines the method `_get_tool_indices` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `_get_tool_indices`.
**CN:** 这一段定义了method `_get_tool_indices`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_get_tool_indices`。

### Lines 77-79: Method `parse_base_json` signature and setup
```python
    def parse_base_json(self, action: Any, tools: List[Tool]) -> List[ToolCallItem]:
        tool_indices = self._get_tool_indices(tools)
        if not isinstance(action, list):
```
**EN:** This block defines the method `parse_base_json` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `parse_base_json`. Notable operations include `_get_tool_indices`.
**CN:** 这一段定义了method `parse_base_json`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_base_json`。 值得注意的操作包括 `_get_tool_indices`。

### Lines 80-101: Method `parse_base_json` logic (part 1)
```python
            action = [action]

        results = []
        for act in action:
            name = act.get("name")
            if not (name and name in tool_indices):
                logger.warning(f"Model attempted to call undefined function: {name}")
                if not envs.SGLANG_FORWARD_UNKNOWN_TOOLS.get():
                    continue  # Skip unknown tools (default legacy behavior)

            results.append(
                ToolCallItem(
                    tool_index=tool_indices.get(name, -1),
                    name=name,
                    parameters=json.dumps(
                        act.get("parameters") or act.get("arguments", {}),
                        ensure_ascii=False,
                    ),
                )
            )

        return results
```
**EN:** This block continues `parse_base_json` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding base format style tool-call detection and normalization workflow. Notable operations include `get`, `not`, `warning`, `tools`.
**CN:** 这一段延续了 `parse_base_json` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的基础格式风格的工具调用检测与规范化工作流。 值得注意的操作包括 `get`、`not`、`warning`、`tools`。

### Lines 103-110: Method `detect_and_parse`
```python
    @abstractmethod
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        Parses the text in one go. Returns success=True if the format matches, otherwise False.
        Note that leftover_text here represents "content that this parser will not consume further".
        """
        action = orjson.loads(text)
        return StreamingParseResult(calls=self.parse_base_json(action, tools))
```
**EN:** This block defines the method `detect_and_parse` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`. Notable operations include `loads`, `StreamingParseResult`, `parse_base_json`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `loads`、`StreamingParseResult`、`parse_base_json`。

### Lines 112-123: Method `_ends_with_partial_token`
```python
    def _ends_with_partial_token(self, buffer: str, bot_token: str) -> int:
        """
        Check if buffer ends with a partial bot_token.
        Return the length of the partial bot_token.

        For some format, the bot_token is not a token in model's vocabulary, such as
        `[TOOL_CALLS] [` in Mistral.
        """
        for i in range(1, min(len(buffer) + 1, len(bot_token))):
            if bot_token.startswith(buffer[-i:]):
                return i
        return 0
```
**EN:** This block defines the method `_ends_with_partial_token` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `_ends_with_partial_token`. Notable operations include `startswith`.
**CN:** 这一段定义了method `_ends_with_partial_token`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_ends_with_partial_token`。 值得注意的操作包括 `startswith`。

### Lines 125-142: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing with tool validation.

        This base implementation works best with formats where:
        1. bot_token is followed immediately by JSON (e.g., bot_token + JSON_array)
        2. JSON can be parsed incrementally using partial_json_loads
        3. Multiple tool calls are separated by "; " or ", "

        Examples of incompatible formats (need custom implementation, may reuse some logic from this class):
        - Each tool call is wrapped in a separate block: See Qwen25Detector
        - Multiple separate blocks: [TOOL_CALLS] [...] \n [TOOL_CALLS] [...]
        - Tool call is Pythonic style

        For incompatible formats, detectors should override this method with custom logic.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`. Notable operations include `JSON`, `formats`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。 值得注意的操作包括 `JSON`、`formats`。

### Lines 143-182: Method `parse_streaming_increment` logic (part 1)
```python
        # Append new text to buffer
        self._buffer += new_text
        current_text = self._buffer

        # The current_text has tool_call if it is the start of a new tool call sequence
        # or it is the start of a new tool call after a tool call separator, when there is a previous tool call
        if not (
            self.has_tool_call(current_text)
            or (
                self.current_tool_id > 0
                and current_text.startswith(self.tool_call_separator)
            )
        ):
            # Only clear buffer if we're sure no tool call is starting
            if not self._ends_with_partial_token(self._buffer, self.bot_token):
                normal_text = self._buffer
                self._buffer = ""
                if self.eot_token in normal_text:
                    normal_text = normal_text.replace(self.eot_token, "")
                return StreamingParseResult(normal_text=normal_text)
            else:
                # Might be partial bot_token, keep buffering
                return StreamingParseResult()

        # Build tool indices if not already built
        if not hasattr(self, "_tool_indices"):
            self._tool_indices = self._get_tool_indices(tools)

        flags = Allow.ALL if self.current_tool_name_sent else Allow.ALL & ~Allow.STR

        try:
            try:
                # Priority check: if we're processing a subsequent tool (current_tool_id > 0),
                # first check if text starts with the tool separator. This is critical for
                # parallel tool calls because the bot_token (e.g., '[') can also
                # appear inside array parameters of the current tool, and we must not
                # mistakenly identify that as the start of a new tool.
                used_separator_branch = False
                if self.current_tool_id > 0 and current_text.startswith(
                    self.tool_call_separator
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding base format style tool-call detection and normalization workflow. Notable operations include `not`, `has_tool_call`, `or`, `startswith`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的基础格式风格的工具调用检测与规范化工作流。 值得注意的操作包括 `not`、`has_tool_call`、`or`、`startswith`。

### Lines 183-216: Method `parse_streaming_increment` logic (part 2)
```python
                ):
                    start_idx = len(self.tool_call_separator)
                    used_separator_branch = True
                else:
                    tool_call_pos = current_text.find(self.bot_token)
                    if tool_call_pos != -1:
                        start_idx = tool_call_pos + len(self.bot_token)
                    else:
                        start_idx = 0

                if start_idx >= len(current_text):
                    return StreamingParseResult()

                try:
                    obj, end_idx = _partial_json_loads(current_text[start_idx:], flags)
                except (MalformedJSON, json.JSONDecodeError):
                    # Separator landed on non-JSON markup; fall back to
                    # bot_token which skips past all inter-object markup.
                    # e.g. Qwen25: separator "," matches between eot/bot tags.
                    if used_separator_branch and self.bot_token in current_text:
                        start_idx = current_text.find(self.bot_token) + len(
                            self.bot_token
                        )
                        if start_idx >= len(current_text):
                            return StreamingParseResult()
                        obj, end_idx = _partial_json_loads(
                            current_text[start_idx:], flags
                        )
                    else:
                        raise

                is_current_complete = _is_complete_json(
                    current_text[start_idx : start_idx + end_idx]
                )
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding base format style tool-call detection and normalization workflow. Notable operations include `find`, `StreamingParseResult`, `_partial_json_loads`, `_is_complete_json`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的基础格式风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`StreamingParseResult`、`_partial_json_loads`、`_is_complete_json`。

### Lines 217-257: Method `parse_streaming_increment` logic (part 3)
```python

                # Validate tool name if present
                if "name" in obj and obj["name"] not in self._tool_indices:
                    # Invalid tool name - reset state
                    self._buffer = ""
                    self.current_tool_id = -1
                    self.current_tool_name_sent = False
                    if self.streamed_args_for_tool:
                        self.streamed_args_for_tool.pop()
                    return StreamingParseResult()

                # Handle parameters/arguments consistency
                # NOTE: we assume here that the obj is always partial of a single tool call
                if "parameters" in obj:
                    assert (
                        "arguments" not in obj
                    ), "model generated both parameters and arguments"
                    obj["arguments"] = obj["parameters"]

                current_tool_call = obj

            except (MalformedJSON, json.JSONDecodeError):
                return StreamingParseResult()

            if not current_tool_call:
                return StreamingParseResult()

            # Case 1: Handle tool name streaming
            # This happens when we encounter a tool but haven't sent its name yet
            if not self.current_tool_name_sent:
                function_name = current_tool_call.get("name")

                if function_name and function_name in self._tool_indices:
                    # If this is a new tool (current_tool_id was -1), initialize it
                    if self.current_tool_id == -1:
                        self.current_tool_id = 0
                        self.streamed_args_for_tool.append("")
                    # If this is a subsequent tool, ensure streamed_args_for_tool is large enough
                    elif self.current_tool_id >= len(self.streamed_args_for_tool):
                        while len(self.streamed_args_for_tool) <= self.current_tool_id:
                            self.streamed_args_for_tool.append("")
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding base format style tool-call detection and normalization workflow. Notable operations include `pop`, `StreamingParseResult`, `get`, `tool`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的基础格式风格的工具调用检测与规范化工作流。 值得注意的操作包括 `pop`、`StreamingParseResult`、`get`、`tool`。

### Lines 258-296: Method `parse_streaming_increment` logic (part 4)
```python

                    # Send the tool name with empty parameters
                    res = StreamingParseResult(
                        calls=[
                            ToolCallItem(
                                tool_index=self.current_tool_id,
                                name=function_name,
                                parameters="",
                            )
                        ],
                    )
                    self.current_tool_name_sent = True
                else:
                    res = StreamingParseResult()

            # Case 2: Handle streaming arguments
            # This happens when we've already sent the tool name and now need to stream arguments incrementally
            else:
                cur_arguments = current_tool_call.get("arguments")
                res = StreamingParseResult()

                if cur_arguments is not None:
                    # Calculate how much of the arguments we've already streamed
                    sent = len(self.streamed_args_for_tool[self.current_tool_id])
                    cur_args_json = json.dumps(cur_arguments, ensure_ascii=False)
                    prev_arguments = None
                    if self.current_tool_id < len(self.prev_tool_call_arr):
                        prev_arguments = self.prev_tool_call_arr[
                            self.current_tool_id
                        ].get("arguments")

                    argument_diff = None

                    # If the current tool's JSON is complete, send all remaining arguments
                    if is_current_complete:
                        argument_diff = cur_args_json[sent:]
                        completing_tool_id = (
                            self.current_tool_id
                        )  # Save the ID of the tool that's completing
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding base format style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `ToolCallItem`, `get`, `dumps`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的基础格式风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`ToolCallItem`、`get`、`dumps`。

### Lines 297-338: Method `parse_streaming_increment` logic (part 5)
```python

                        # Only remove the processed portion, keep unprocessed content
                        self._buffer = current_text[start_idx + end_idx :]

                    # If the tool is still being parsed, send incremental changes
                    elif prev_arguments:
                        prev_args_json = json.dumps(prev_arguments, ensure_ascii=False)
                        if cur_args_json != prev_args_json:
                            prefix = _find_common_prefix(prev_args_json, cur_args_json)
                            argument_diff = prefix[sent:]

                    # Update prev_tool_call_arr with current state
                    if self.current_tool_id >= 0:
                        # Ensure prev_tool_call_arr is large enough
                        while len(self.prev_tool_call_arr) <= self.current_tool_id:
                            self.prev_tool_call_arr.append({})
                        self.prev_tool_call_arr[self.current_tool_id] = (
                            current_tool_call
                        )

                    # Advance to next tool if complete
                    if is_current_complete:
                        self.current_tool_name_sent = False
                        self.current_tool_id += 1

                    # Send the argument diff if there's something new
                    if argument_diff is not None:
                        # Use the correct tool_index: completing_tool_id for completed tools, current_tool_id for ongoing
                        tool_index_to_use = (
                            completing_tool_id
                            if is_current_complete
                            else self.current_tool_id
                        )
                        res = StreamingParseResult(
                            calls=[
                                ToolCallItem(
                                    tool_index=tool_index_to_use,
                                    parameters=argument_diff,
                                )
                            ],
                        )
                        self.streamed_args_for_tool[tool_index_to_use] += argument_diff
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding base format style tool-call detection and normalization workflow. Notable operations include `dumps`, `_find_common_prefix`, `append`, `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的基础格式风格的工具调用检测与规范化工作流。 值得注意的操作包括 `dumps`、`_find_common_prefix`、`append`、`StreamingParseResult`。

### Lines 339-344: Method `parse_streaming_increment` logic (part 6)
```python

            return res

        except Exception as e:
            logger.error(f"Error in parse_streaming_increment: {e}")
            return StreamingParseResult()
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding base format style tool-call detection and normalization workflow. Notable operations include `error`, `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的基础格式风格的工具调用检测与规范化工作流。 值得注意的操作包括 `error`、`StreamingParseResult`。

### Lines 346-351: Method `has_tool_call`
```python
    @abstractmethod
    def has_tool_call(self, text: str) -> bool:
        """
        Check if the given text contains function call markers specific to this format.
        """
        raise NotImplementedError()
```
**EN:** This block defines the method `has_tool_call` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `has_tool_call`. Notable operations include `NotImplementedError`.
**CN:** 这一段定义了method `has_tool_call`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。 值得注意的操作包括 `NotImplementedError`。

### Lines 353-355: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        """Return True if this detector supports structural tag format."""
        return True
```
**EN:** This block defines the method `supports_structural_tag` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。

### Lines 357-369: Method `structure_info`
```python
    @abstractmethod
    def structure_info(self) -> _GetInfoFunc:
        """
        Return a function that creates StructureInfo for constrained generation.

        The returned function takes a tool name and returns a StructureInfo object
        containing the begin/end patterns and trigger tokens needed for constrained
        generation of function calls in this format.

        Returns:
            A function that takes a tool name (str) and returns StructureInfo
        """
        raise NotImplementedError()
```
**EN:** This block defines the method `structure_info` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `name`, `NotImplementedError`.
**CN:** 这一段定义了method `structure_info`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `name`、`NotImplementedError`。

### Lines 371-373: Method `get_structural_tag_name`
```python
    def get_structural_tag_name(self) -> Optional[str]:
        """Return the XGrammar model name for native structural tags, if supported."""
        return None
```
**EN:** This block defines the method `get_structural_tag_name` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `get_structural_tag_name`.
**CN:** 这一段定义了method `get_structural_tag_name`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `get_structural_tag_name`。

### Lines 375-395: Method `get_structural_tag` signature and setup
```python
    def get_structural_tag(
        self,
        tools: Union[List[Tool], None] = None,
        tool_choice: Union[ToolChoice, Literal["auto", "required"]] = "auto",
        thinking_mode: bool = False,
    ) -> Optional[StructuralTag]:
        """
        Return a model-native XGrammar structural tag when supported.

        Args:
            tools: List of available tools
            tool_choice: The tool choice setting from the request
            thinking_mode: Whether to include the model's reasoning prefix in
                the returned structural tag. Pass False when SGLang's
                ReasonerGrammarBackend will own the <think>...</think> prefix
                (the typical case when --reasoning-parser is configured) so
                only one layer constrains the reasoning section.

        Returns:
            StructuralTag if this detector supports model-native tags, otherwise None
        """
```
**EN:** This block defines the method `get_structural_tag` on `BaseFormatDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of base format style tool-call detection and normalization. Definitions introduced here include `get_structural_tag`. Notable operations include `prefix`.
**CN:** 这一段定义了method `get_structural_tag`（属于 `BaseFormatDetector`），介绍了参数、初始化步骤，以及这部分基础格式风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `get_structural_tag`。 值得注意的操作包括 `prefix`。

### Lines 396-411: Method `get_structural_tag` logic (part 1)
```python
        structural_tag_name = self.get_structural_tag_name()
        if not structural_tag_name or get_model_structural_tag is None:
            return None

        converted_tools = [tool.model_dump() for tool in tools or []]
        converted_tool_choice = (
            tool_choice.model_dump()
            if isinstance(tool_choice, ToolChoice)
            else tool_choice
        )
        return get_model_structural_tag(
            model=structural_tag_name,
            tools=converted_tools,
            tool_choice=converted_tool_choice,
            reasoning=thinking_mode,
        )
```
**EN:** This block continues `get_structural_tag` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding base format style tool-call detection and normalization workflow. Notable operations include `get_structural_tag_name`, `model_dump`, `get_model_structural_tag`.
**CN:** 这一段延续了 `get_structural_tag` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的基础格式风格的工具调用检测与规范化工作流。 值得注意的操作包括 `get_structural_tag_name`、`model_dump`、`get_model_structural_tag`。

## Key Concepts / 关键概念
- `BaseFormatDetector`: Class that encapsulates base format detector behavior in this module. / `BaseFormatDetector`：封装与“基础格式检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `abc`, `typing`
- **External packages / 外部依赖**: `orjson`, `partial_json_parser`, `xgrammar`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.environ`, `sglang.srt.function_call.core_types`, `sglang.srt.function_call.utils`
