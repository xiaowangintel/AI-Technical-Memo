# responses_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/parser/responses_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Parses response payloads into internal structured representations. Scope: OpenAI-compatible parser. / 将响应载荷解析为内部结构化表示。 作用域：OpenAI 兼容 / 解析器。

## Line-by-Line Analysis / 逐行分析
### Lines 3-28 — Imports and shared dependencies
```python
import logging
from typing import Any

from openai.types.responses import ResponseFunctionToolCall, ResponseOutputItem
from openai.types.responses.response_function_tool_call_output_item import (
    ResponseFunctionToolCallOutputItem,
)
from openai.types.responses.response_output_item import McpCall
from openai.types.responses.response_output_message import ResponseOutputMessage
from openai.types.responses.response_output_text import ResponseOutputText
from openai.types.responses.response_reasoning_item import (
    Content,
    ResponseReasoningItem,
)

from vllm.entrypoints.chat_utils import ChatTemplateContentFormatOption
from vllm.entrypoints.constants import MCP_PREFIX
from vllm.entrypoints.openai.responses.protocol import (
    ResponseInputOutputItem,
    ResponsesRequest,
)
from vllm.outputs import CompletionOutput
from vllm.reasoning.abs_reasoning_parsers import ReasoningParser
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import ToolParser
from vllm.utils import random_uuid
```
**EN:** This import block pulls in standard-library modules such as `logging`, `typing`, uses third-party packages like `openai`, depends on internal helpers such as `vllm.entrypoints.chat_utils`, `vllm.entrypoints.constants`, `vllm.entrypoints.openai.responses.protocol`, `vllm.outputs`, `vllm.reasoning.abs_reasoning_parsers`, `vllm.tokenizers`.
**CN:** 该导入块引入 `logging`, `typing` 等标准库模块，使用 `openai` 等第三方库，依赖 `vllm.entrypoints.chat_utils`, `vllm.entrypoints.constants`, `vllm.entrypoints.openai.responses.protocol`, `vllm.outputs`, `vllm.reasoning.abs_reasoning_parsers`, `vllm.tokenizers` 等 vLLM 内部模块。

### Lines 30-30 — Shared module state
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 33-166 — Class `ResponsesParser`
```python
class ResponsesParser:
    """Incremental parser over completion tokens with reasoning support."""

    def __init__(
        self,
        *,
        tokenizer: TokenizerLike,
        reasoning_parser_cls: type[ReasoningParser],
        response_messages: list[ResponseInputOutputItem],
        request: ResponsesRequest,
        tool_parser_cls: type[ToolParser] | None,
        chat_template: str | None,
        chat_template_content_format: ChatTemplateContentFormatOption,
    ):
        self.response_messages: list[ResponseInputOutputItem] = (
            # TODO: initial messages may not be properly typed
            response_messages
        )
        self.num_init_messages = len(response_messages)
        self.tokenizer = tokenizer
        self.request = request

        self.reasoning_parser_instance = reasoning_parser_cls(
            tokenizer,
            chat_template_kwargs=_effective_chat_template_kwargs(
                request,
                chat_template=chat_template,
                chat_template_content_format=chat_template_content_format,
...
                        type="mcp_call",
                        status="completed",
                        output=message.output,
                        # TODO: support error output
                    )
                    output_messages[-1] = mcp_message

        return output_messages
```
**EN:** Class `ResponsesParser` is introduced here. Its docstring describes the intent as: Incremental parser over completion tokens with reasoning support.
**CN:** 这里定义类 `ResponsesParser`。其文档字符串说明了该类的职责与使用方式。

### Lines 169-193 — Function `get_responses_parser_for_simple_context`
```python
def get_responses_parser_for_simple_context(
    *,
    tokenizer: TokenizerLike,
    reasoning_parser_cls: type[ReasoningParser],
    response_messages: list[ResponseInputOutputItem],
    request: ResponsesRequest,
    tool_parser_cls,
    chat_template: str | None,
    chat_template_content_format: ChatTemplateContentFormatOption,
) -> ResponsesParser:
    """Factory function to create a ResponsesParser with
    optional reasoning parser.

    Returns:
        ResponsesParser instance configured with the provided parser
    """
    return ResponsesParser(
        tokenizer=tokenizer,
        reasoning_parser_cls=reasoning_parser_cls,
        response_messages=response_messages,
        request=request,
        tool_parser_cls=tool_parser_cls,
        chat_template=chat_template,
        chat_template_content_format=chat_template_content_format,
    )
```
**EN:** This function `get_responses_parser_for_simple_context` is documented as: Factory function to create a ResponsesParser with optional reasoning parser.
**CN:** 这里定义函数 `get_responses_parser_for_simple_context`，其文档字符串说明了主要职责与调用约定。

### Lines 196-204 — Function `_effective_chat_template_kwargs`
```python
def _effective_chat_template_kwargs(
    request: ResponsesRequest,
    chat_template: str | None,
    chat_template_content_format: ChatTemplateContentFormatOption,
) -> dict[str, Any]:
    return request.build_chat_params(
        default_template=chat_template,
        default_template_content_format=chat_template_content_format,
    ).chat_template_kwargs
```
**EN:** This function `_effective_chat_template_kwargs` implements the `effective chat template kwargs` step within the module flow.
**CN:** 该函数 `_effective_chat_template_kwargs` 实现了模块流程中的“effective对话templatekwargs”步骤。

## Key Concepts / 关键概念
- Tokenization or token-level processing / 分词或 token 级处理
- MCP tool integration / MCP 工具集成
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `logging`, `typing`
- **Third-party / 第三方**: `openai`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.constants`, `vllm.entrypoints.openai.responses.protocol`, `vllm.outputs`, `vllm.reasoning.abs_reasoning_parsers`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.utils`
