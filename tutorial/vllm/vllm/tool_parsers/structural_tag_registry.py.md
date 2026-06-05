# structural_tag_registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/structural_tag_registry.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Maintains model-specific structural-tag builders used to constrain tool-calling output formats during decoding. / 维护模型专用的结构标签构建器，用于在解码期间约束工具调用输出格式。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 8-25)
```python
from collections.abc import Callable
from typing import Any, Literal

from xgrammar import StructuralTag
from xgrammar.structural_tag import (
    AnyTextFormat,
    ConstStringFormat,
    JSONSchemaFormat,
    SequenceFormat,
    TagFormat,
    TagsWithSeparatorFormat,
    TriggeredTagsFormat,
)

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionNamedToolChoiceParam,
    ChatCompletionToolsParam,
)
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 27-36)
```python
SimplifiedToolChoice = Literal["auto", "required", "forced"]
ToolChoice = (
    Literal["none", "auto", "required"] | ChatCompletionNamedToolChoiceParam | None
)
StructuralTagBuilder = Callable[
    [list[ChatCompletionToolsParam], SimplifiedToolChoice, bool],
    StructuralTag,
]

_structural_tag_registry: dict[str, StructuralTagBuilder] = {}
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Function `register_model_structural_tag` (lines 39-46)
```python
def register_model_structural_tag(name: str):
    """Register a vLLM-owned model-specific structural tag builder."""

    def decorator(func: StructuralTagBuilder) -> StructuralTagBuilder:
        _structural_tag_registry[name] = func
        return func

    return decorator
```
**EN:** Register a vLLM-owned model-specific structural tag builder.
**CN:** `register_model_structural_tag` 函数把实现注册到共享注册表中，便于服务层按模型或格式名称查找。

### Function `get_model_structural_tag` (lines 49-69)
```python
def get_model_structural_tag(
    model: str,
    tools: list[ChatCompletionToolsParam] | None,
    tool_choice: ToolChoice,
    reasoning: bool,
) -> StructuralTag | None:
    """Build a structural tag from vLLM-owned model-specific builders."""

    builder = _structural_tag_registry.get(model)
    if builder is None:
        supported = list(_structural_tag_registry.keys())
        raise ValueError(f"Unknown format type: {model}, supported types: {supported}")

    normalized_tools, simplified_tool_choice = _normalize_tool_choice(
        tools=tools,
        tool_choice=tool_choice,
    )
    if not normalized_tools:
        return None

    return builder(normalized_tools, simplified_tool_choice, reasoning)
```
**EN:** Build a structural tag from vLLM-owned model-specific builders.
**CN:** `get_model_structural_tag` 函数构建或返回解析器与上层服务需要的派生元数据。

### Function `_normalize_tool_choice` (lines 72-99)
```python
def _normalize_tool_choice(
    tools: list[ChatCompletionToolsParam] | None,
    tool_choice: ToolChoice,
) -> tuple[list[ChatCompletionToolsParam], SimplifiedToolChoice]:
    """Normalize vLLM ChatCompletion tool_choice for structural tag builders."""

    if not tools:
        return [], "auto"

    if tool_choice is None or tool_choice == "none":
        return [], "auto"

    if tool_choice == "auto":
        return tools, "auto"

    if tool_choice == "required":
        return tools, "required"

    if isinstance(tool_choice, ChatCompletionNamedToolChoiceParam):
        tool_name = tool_choice.function.name
        filtered_tools = [tool for tool in tools if tool.function.name == tool_name]
        if not filtered_tools:
            raise ValueError(
                f"The tool with name '{tool_name}' is not found in the tools list."
            )
        return filtered_tools, "forced"

    raise ValueError(f"Unsupported tool_choice for structural tag: {tool_choice}")
```
**EN:** Normalize vLLM ChatCompletion tool_choice for structural tag builders.
**CN:** `_normalize_tool_choice` 函数实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Function `_get_function_parameters` (lines 102-109)
```python
def _get_function_parameters(function: Any) -> dict[str, Any] | bool:
    """Return the JSON schema used for constrained tool arguments."""

    if getattr(function, "strict", None) is False:
        return True
    if function.parameters is None:
        return True
    return function.parameters
```
**EN:** Return the JSON schema used for constrained tool arguments.
**CN:** `_get_function_parameters` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Module constants/configuration 2 (lines 112-112)
```python
_enable_structured_outputs_in_reasoning: bool = False
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Function `set_enable_structured_outputs_in_reasoning` (lines 115-123)
```python
def set_enable_structured_outputs_in_reasoning(enabled: bool) -> None:
    """Publish the engine's ``enable_in_reasoning`` flag to tool parsers.

    Called once during APIServer startup so request-time parsers can read
    it without going through the EngineCore-only contextvar.
    """

    global _enable_structured_outputs_in_reasoning
    _enable_structured_outputs_in_reasoning = bool(enabled)
```
**EN:** Publish the engine's ``enable_in_reasoning`` flag to tool parsers.
**CN:** `set_enable_structured_outputs_in_reasoning` 函数更新共享配置，供后续解析流程中的其他组件读取。

### Function `get_enable_structured_outputs_in_reasoning` (lines 126-134)
```python
def get_enable_structured_outputs_in_reasoning() -> bool:
    """Whether structured outputs are active during the reasoning phase.

    When ``True``, the structural tag will cover the reasoning part:
    ``<think>...</think>`` prefix (if available); when ``False`` (default), the tag only
    constrains the post-reasoning suffix.
    """

    return _enable_structured_outputs_in_reasoning
```
**EN:** Whether structured outputs are active during the reasoning phase.
**CN:** `get_enable_structured_outputs_in_reasoning` 函数构建或返回解析器与上层服务需要的派生元数据。

### Function `get_deepseek_v4_structural_tag` (lines 138-243)
```python
def get_deepseek_v4_structural_tag(
    tools: list[ChatCompletionToolsParam],
    tool_choice: SimplifiedToolChoice,
    reasoning: bool,
) -> StructuralTag:
    """Build DeepSeek V4 structural tags."""

    invoke_begin_prefix = '<｜DSML｜invoke name="'
    invoke_begin_suffix = '">\n'
    invoke_end = "</｜DSML｜invoke>\n"
    tool_calls_prefix = "\n\n"
    function_calls_begin = "<｜DSML｜tool_calls>\n"
    function_calls_end = "</｜DSML｜tool_calls>"
    function_calls_trigger = "<｜DSML｜tool_calls>"
    think_tag_end = "</think>"
    think_exclude_tokens = ["<think>", "</think>"]
    xml_style = "deepseek_xml"

    if tool_choice == "auto":
        tags = []
        for tool in tools:
            function = tool.function
            parameters = _get_function_parameters(function)
            tags.append(
                TagFormat(
                    begin=invoke_begin_prefix + function.name + invoke_begin_suffix,
                    content=JSONSchemaFormat(
                        json_schema=parameters,
                        style=xml_style,
                    ),
                    end=invoke_end,
                )
            )

        if tags:
            function_calling_tags = TagsWithSeparatorFormat(
                tags=tags,
                separator="\n",
                at_least_one=True,
            )
            suffix_tag = TriggeredTagsFormat(
                triggers=[function_calls_trigger],
                tags=[
                    TagFormat(
                        begin=function_calls_begin,
                        content=function_calling_tags,
                        end=function_calls_end,
                    )
                ],
                excludes=think_exclude_tokens,
# ... omitted for brevity / 为简洁省略 ...
                    separator="\n",
                    at_least_one=True,
                ),
                ConstStringFormat(value=function_calls_end),
            ]
        )

    if not reasoning:
        return StructuralTag(format=suffix_tag)

    prefix_tag = TagFormat(begin="", content=AnyTextFormat(), end=think_tag_end)
    return StructuralTag(format=SequenceFormat(elements=[prefix_tag, suffix_tag]))
```
**EN:** Build DeepSeek V4 structural tags.
**CN:** `get_deepseek_v4_structural_tag` 函数构建或返回解析器与上层服务需要的派生元数据。

### Function `get_qwen_3_5_structural_tag` (lines 247-330)
```python
def get_qwen_3_5_structural_tag(
    tools: list[ChatCompletionToolsParam],
    tool_choice: SimplifiedToolChoice,
    reasoning: bool,
) -> StructuralTag:
    """Build Qwen XML structural tags.

    This format is used for Qwen3-Coder/Qwen3.5/Qwen3.6 and is compatible with
    Qwen variants that use the same XML tool-call format.
    """
    tool_call_begin_prefix = "<tool_call>\n<function="
    tool_call_begin_suffix = ">\n"
    tool_call_end = "\n</function>\n</tool_call>"
    tool_call_trigger = "<tool_call>\n<function="
    think_tag_end = "</think>"
    think_suffix = "\n\n"
    think_exclude_tokens = ["<think>", "</think>"]

    if tool_choice == "auto":
        tags = []
        for tool in tools:
            function = tool.function
            parameters = _get_function_parameters(function)
            tags.append(
                TagFormat(
                    begin=f"{tool_call_begin_prefix}{function.name}{tool_call_begin_suffix}",
                    content=JSONSchemaFormat(json_schema=parameters, style="qwen_xml"),
                    end=tool_call_end,
                )
            )

        if tags:
            suffix_tag = TriggeredTagsFormat(
                triggers=[tool_call_trigger],
                tags=tags,
                excludes=think_exclude_tokens,
            )
        else:
            suffix_tag = AnyTextFormat(excludes=think_exclude_tokens)

    elif tool_choice == "forced":
        if not tools:
            raise ValueError("Forced tool choice must resolve to exactly one tool.")
        function = tools[0].function
        suffix_tag = TagFormat(
            begin=f"{tool_call_begin_prefix}{function.name}{tool_call_begin_suffix}",
            content=JSONSchemaFormat(
                json_schema=_get_function_parameters(function),
                style="qwen_xml",
            ),
# ... omitted for brevity / 为简洁省略 ...
    if not reasoning:
        result = StructuralTag(format=suffix_tag)
    else:
        prefix_tag = SequenceFormat(
            elements=[
                TagFormat(begin="", content=AnyTextFormat(), end=think_tag_end),
                ConstStringFormat(value=think_suffix),
            ]
        )
        result = StructuralTag(format=SequenceFormat(elements=[prefix_tag, suffix_tag]))

    return result
```
**EN:** Build Qwen XML structural tags.
**CN:** `get_qwen_3_5_structural_tag` 函数构建或返回解析器与上层服务需要的派生元数据。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Structured decoding constraints**: Structured decoding constraints. / 用结构标签约束模型输出，减少无效工具调用格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方**: `xgrammar`, `xgrammar.structural_tag`
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`
