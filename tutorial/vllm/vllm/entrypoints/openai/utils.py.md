# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/utils.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Provides reusable helper utilities shared across related entrypoints. Scope: OpenAI-compatible. / 提供在相关入口之间复用的辅助工具函数。 作用域：OpenAI 兼容。

## Line-by-Line Analysis / 逐行分析
### Lines 3-12 — Imports and shared dependencies
```python
from typing import TypeVar

from fastapi import Request
from fastapi.exceptions import RequestValidationError

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
    ChatCompletionResponseChoice,
    ChatCompletionResponseStreamChoice,
)
```
**EN:** This import block pulls in standard-library modules such as `typing`, uses third-party packages like `fastapi`, depends on internal helpers such as `vllm.entrypoints.openai.chat_completion.protocol`.
**CN:** 该导入块引入 `typing` 等标准库模块，使用 `fastapi` 等第三方库，依赖 `vllm.entrypoints.openai.chat_completion.protocol` 等 vLLM 内部模块。

### Lines 15-19 — Module constants and state
```python
_ChatCompletionResponseChoiceT = TypeVar(
    "_ChatCompletionResponseChoiceT",
    ChatCompletionResponseChoice,
    ChatCompletionResponseStreamChoice,
)
```
**EN:** This block initializes `_ChatCompletionResponseChoiceT`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `_ChatCompletionResponseChoiceT`，为后续逻辑准备模块级常量或共享状态。

### Lines 22-40 — Function `maybe_filter_parallel_tool_calls`
```python
def maybe_filter_parallel_tool_calls(
    choice: _ChatCompletionResponseChoiceT, request: ChatCompletionRequest
) -> _ChatCompletionResponseChoiceT:
    """Filter to first tool call only when parallel_tool_calls is False."""

    if request.parallel_tool_calls:
        return choice

    if isinstance(choice, ChatCompletionResponseChoice) and choice.message.tool_calls:
        choice.message.tool_calls = choice.message.tool_calls[:1]
    elif (
        isinstance(choice, ChatCompletionResponseStreamChoice)
        and choice.delta.tool_calls
    ):
        choice.delta.tool_calls = [
            tool_call for tool_call in choice.delta.tool_calls if tool_call.index == 0
        ]

    return choice
```
**EN:** This function `maybe_filter_parallel_tool_calls` is documented as: Filter to first tool call only when parallel_tool_calls is False.
**CN:** 这里定义函数 `maybe_filter_parallel_tool_calls`，其文档字符串说明了主要职责与调用约定。

### Lines 43-49 — Function `validate_json_request`
```python
async def validate_json_request(raw_request: Request):
    content_type = raw_request.headers.get("content-type", "").lower()
    media_type = content_type.split(";", maxsplit=1)[0]
    if media_type != "application/json":
        raise RequestValidationError(
            errors=["Unsupported Media Type: Only 'application/json' is allowed"]
        )
```
**EN:** This async function `validate_json_request` checks or normalizes inputs.
**CN:** 该异步函数 `validate_json_request` 检查或规范化输入。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Async request handling / 异步请求处理
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing`
- **Third-party / 第三方**: `fastapi`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.openai.chat_completion.protocol`
