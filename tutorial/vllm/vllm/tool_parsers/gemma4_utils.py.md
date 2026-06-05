# gemma4_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/gemma4_utils.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Collects Gemma 4 specific helper routines shared by the tool parser implementation. / 收集 Gemma 4 工具解析器实现共用的辅助函数。

## Line-by-Line Analysis / 逐行分析
### Docstring (lines 5-36)
```python
"""Gemma4 tool call parsing utilities for offline inference.

Standalone functions that parse decoded model text to extract tool calls
from Gemma4 models. These are pure-Python utilities with zero heavy
dependencies — they work on raw decoded strings from any inference
backend (vLLM, HuggingFace, TGI, etc.).

For the OpenAI-compatible API server tool parser (streaming +
non-streaming), see ``vllm.tool_parsers.gemma4_tool_parser``.
For thinking/reasoning output parsing, see
``vllm.reasoning.gemma4_utils``.

Usage with vLLM offline inference::

    from vllm import LLM, SamplingParams
    from vllm.tool_parsers.gemma4_utils import (
        parse_tool_calls,
        has_tool_response_tag,
    )

    llm = LLM(model="google/gemma-4-it")
    outputs = llm.generate(prompt, SamplingParams(...))
    text = tokenizer.decode(outputs[0].outputs[0].token_ids, skip_special_tokens=False)

    # Extract tool calls
    tool_calls = parse_tool_calls(text)
    for tc in tool_calls:
        print(f"{tc['name']}({tc['arguments']})")

Ported from ``transformers.models.gemma4.utils_gemma4`` so that vLLM users
do not need a transformers dependency for output parsing.
"""
```
**EN:** Gemma4 tool call parsing utilities for offline inference.
**CN:** 模块文档字符串直接说明了该文件的用途。

### Imports and setup (lines 38-40)
```python
import json

import regex as re
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 44-46)
```python
_TOOL_CALL_START_TAG = "<|tool_call>"
_TOOL_CALL_END_TAG = "<tool_call|>"
_TOOL_RESPONSE_START_TAG = "<|tool_response>"
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Module constants/configuration 2 (lines 49-49)
```python
_ESCAPE_TOKEN = '<|"|>'
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Function `_parse_tool_arguments` (lines 52-90)
```python
def _parse_tool_arguments(args_str: str) -> dict[str, str]:
    """Parse tool call arguments from the Gemma4 compact format.

    Handles the ``key:<|"|>value<|"|>`` format used by Gemma4, with fallback
    to heuristic key-value extraction. Also tolerates the slightly different
    ``key: "value"`` format (space + plain quotes) that some chat templates
    produce.

    Args:
        args_str: Raw argument string from inside ``call:name{...}``.

    Returns:
        Dictionary of argument name → value.
    """
    if not args_str or not args_str.strip():
        return {}

    # Replace Gemma4 escape tokens with standard quotes.
    cleaned = args_str.replace(_ESCAPE_TOKEN, '"')

    # Try JSON parsing first (handles nested values, arrays, etc.).
    try:
        parsed = json.loads("{" + cleaned + "}")
        # Ensure all values are strings for consistency.
        return {k: str(v) if not isinstance(v, str) else v for k, v in parsed.items()}
    except (json.JSONDecodeError, ValueError):
        pass

    # Fallback: extract key:"value" pairs (allow optional space after colon).
    arguments = {}
    for key, value in re.findall(r'(\w+):\s*"([^"]*)"', cleaned):
        arguments[key] = value

    if not arguments:
        # Last resort: extract key:value pairs (unquoted).
        for key, value in re.findall(r"(\w+):\s*([^,}]+)", args_str):
            arguments[key] = value.strip().strip('"').replace(_ESCAPE_TOKEN, "")

    return arguments
```
**EN:** Parse tool call arguments from the Gemma4 compact format.
**CN:** `_parse_tool_arguments` 函数实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Function `parse_tool_calls` (lines 93-157)
```python
def parse_tool_calls(text: str, *, strict: bool = False) -> list[dict]:
    """Parse tool calls from decoded Gemma4 model output.

    Uses a tiered parsing strategy to handle known output variations in
    Gemma4 models, which may emit
    non-standard tool call formats.

    Parsing tiers:
        1. **Standard**: ``<|tool_call>call:name{args}<tool_call|>``
           (special token IDs 48/49 in decoded text)
        2. **Fallback** (when ``strict=False``): bare ``call:name{args}``
           patterns, including ``<call>name{args}`` (fragmented tokens from
           multimodal inputs)

    Args:
        text: Decoded model output text (from ``tokenizer.decode(...,
            skip_special_tokens=False)``).
        strict: If ``True``, only match the standard ``<|tool_call>`` format.
            If ``False`` (default), also try fallback patterns for
            known Gemma4 output variations.

    Returns:
        A list of dicts, each with keys:
            - ``"name"``: The tool function name (e.g. ``"get_weather"``).
            - ``"arguments"``: A dict of argument name → value.

    Example::

        >>> from vllm.tool_parsers.gemma4_utils import parse_tool_calls
        >>> output = tokenizer.decode(outputs[0], skip_special_tokens=False)
        >>> tool_calls = parse_tool_calls(output)
        >>> for tc in tool_calls:
        ...     print(f"Call: {tc['name']}({tc['arguments']})")
    """
    results = []

    # Tier 1: Standard format with special tokens.
    # <|tool_call>call:name{args}<tool_call|>
    # Note: Some Gemma4 models emit <turn|> instead of <tool_call|>.
    standard_pattern = r"<\|tool_call\>call:(\w+)\{(.*?)\}(?:<tool_call\|>|<turn\|>)"
    for match in re.finditer(standard_pattern, text, re.DOTALL):
        name, args_str = match.group(1), match.group(2)
        results.append(
            {
                "name": name,
                "arguments": _parse_tool_arguments(args_str),
            }
        )

    if results or strict:
        return results

    # Tier 2: Fallback for known Gemma4 output variations.
    # Matches: <call>name{args}, call:name{args}, or bare call:name{args}<eos>
    fallback_pattern = r"(?:<call>|(?:^|\s)call:)(\w+)\{(.*?)\}"
    for match in re.finditer(fallback_pattern, text, re.DOTALL):
        name, args_str = match.group(1), match.group(2)
        results.append(
            {
                "name": name,
                "arguments": _parse_tool_arguments(args_str),
            }
        )

    return results
```
**EN:** Parse tool calls from decoded Gemma4 model output.
**CN:** `parse_tool_calls` 函数实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Function `has_tool_response_tag` (lines 160-183)
```python
def has_tool_response_tag(text: str) -> bool:
    """Check if model output properly ends with a tool response tag.

    Some Gemma4 models sometimes emit ``<eos>`` instead of
    ``<|tool_response>`` after a tool call. This helper detects
    whether the model used the proper termination, so callers can
    decide whether to inject ``<|tool_response>`` into the next prompt.

    Args:
        text: Decoded model output text.

    Returns:
        ``True`` if the output ends with ``<|tool_response>``
        (proper behavior), ``False`` otherwise.

    Example::

        >>> from vllm.tool_parsers.gemma4_utils import has_tool_response_tag
        >>> if not has_tool_response_tag(model_output):
        ...     # Model used <eos> instead — inject <|tool_response> manually
        ...     next_prompt = "<|tool_response>" + tool_result
    """
    stripped = text.rstrip()
    return stripped.endswith(_TOOL_RESPONSE_START_TAG)
```
**EN:** Check if model output properly ends with a tool response tag.
**CN:** `has_tool_response_tag` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: None / 无
