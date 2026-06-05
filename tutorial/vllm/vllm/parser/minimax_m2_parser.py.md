# minimax_m2_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/parser/minimax_m2_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MiniMax M2 Parser - A unified parser for MiniMax M2 models / 该模块围绕 `minimax_m2_parser` 相关逻辑组织实现，提供与源文件相匹配的核心功能。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-21)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

"""
MiniMax M2 Parser - A unified parser for MiniMax M2 models.

This parser combines the existing MiniMaxM2ReasoningParser and
MinimaxM2ToolParser into a single unified interface by delegating
to those implementations.
"""

from vllm.logger import init_logger
from vllm.parser.abstract_parser import DelegatingParser
from vllm.reasoning.minimax_m2_reasoning_parser import MiniMaxM2ReasoningParser
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
)
from vllm.tool_parsers.minimax_m2_tool_parser import MinimaxM2ToolParser

logger = init_logger(__name__)
```
**EN:** Sets up the module with vLLM modules such as `vllm.logger`, `vllm.parser.abstract_parser`, `vllm.reasoning.minimax_m2_reasoning_parser`. It prepares the symbols later used by `MiniMaxM2Parser`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.logger`, `vllm.parser.abstract_parser`, `vllm.reasoning.minimax_m2_reasoning_parser` 等 vLLM 内部依赖。 这些准备工作为后续的 `MiniMaxM2Parser` 提供上下文。

### MiniMaxM2Parser (lines 24-61)
```python
class MiniMaxM2Parser(DelegatingParser):
    """
    Unified parser for MiniMax M2 models that handles both reasoning
    extraction and tool call parsing.

    This parser delegates to the existing implementations:
    - MiniMaxM2ReasoningParser for reasoning extraction
    - MinimaxM2ToolParser for tool call parsing

    MiniMax M2 models have two special behaviors:
    1. Reasoning: They don't generate <think> start token, only </think> end
       token. All content before </think> is reasoning, content after is the
       actual response.
    2. Tool Calls: They use <minimax:tool_call>...</minimax:tool_call> tags
       with <invoke name="...">...</invoke> and <parameter name="...">...</parameter>
       syntax.
    """

    # Class-level parser classes for compatibility
    reasoning_parser_cls = MiniMaxM2ReasoningParser
    tool_parser_cls = MinimaxM2ToolParser

    def __init__(
        self,
        tokenizer: TokenizerLike,
        tools: list[Tool] | None = None,
        *args,
        **kwargs,
    # ...
        self._reasoning_parser = MiniMaxM2ReasoningParser(tokenizer, *args, **kwargs)
        self._tool_parser = MinimaxM2ToolParser(tokenizer, tools)

        logger.debug(
            "vLLM Successfully initialized parser %s!", self.__class__.__name__
        )
```
**EN:** `MiniMaxM2Parser`: Unified parser for MiniMax M2 models that handles both reasoning extraction and tool call parsing. It extends `DelegatingParser`. Key methods include `__init__`.
**CN:** `MiniMaxM2Parser` 是该文件中的核心类，用于封装与 `MiniMaxM2Parser` 相关的状态和行为。 它继承自 `DelegatingParser`。 关键方法包括 `__init__`。

## Key Concepts / 关键概念
- **`MiniMaxM2Parser`**: Core class that organizes module behavior. / **`MiniMaxM2Parser`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: None / 无
- **Third-party / 第三方**: None / 无
- **Internal vLLM / vLLM 内部依赖**: vllm.logger, vllm.parser.abstract_parser, vllm.reasoning.minimax_m2_reasoning_parser, vllm.tokenizers, vllm.tool_parsers.abstract_tool_parser, vllm.tool_parsers.minimax_m2_tool_parser
