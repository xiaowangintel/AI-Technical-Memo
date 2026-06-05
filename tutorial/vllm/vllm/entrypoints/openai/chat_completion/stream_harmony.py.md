# stream_harmony.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/chat_completion/stream_harmony.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements streaming-specific Harmony formatting behavior. Scope: OpenAI-compatible chat completion. / 实现面向流式输出的 Harmony 格式化逻辑。 作用域：OpenAI 兼容 / 对话补全。

## Line-by-Line Analysis / 逐行分析
### Lines 3-8 — Module overview
```python
"""
Harmony-specific streaming delta extraction for chat completions.

This module handles the extraction of DeltaMessage objects from
harmony parser state during streaming chat completions.
"""
```
**EN:** The module header documents the main contract: Harmony-specific streaming delta extraction for chat completions.
**CN:** 模块开头的文档字符串说明了该文件的主要职责、使用边界或运行约束。

### Lines 10-23 — Imports and shared dependencies
```python
from typing import NamedTuple

from openai_harmony import StreamableParser

from vllm.entrypoints.chat_utils import make_tool_call_id
from vllm.entrypoints.openai.engine.protocol import (
    DeltaFunctionCall,
    DeltaMessage,
    DeltaToolCall,
)
from vllm.entrypoints.openai.parser.harmony_utils import (
    extract_function_from_recipient,
    is_function_recipient,
)
```
**EN:** This import block pulls in standard-library modules such as `typing`, uses third-party packages like `openai_harmony`, depends on internal helpers such as `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.parser.harmony_utils`.
**CN:** 该导入块引入 `typing` 等标准库模块，使用 `openai_harmony` 等第三方库，依赖 `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.parser.harmony_utils` 等 vLLM 内部模块。

### Lines 26-29 — Class `TokenState`
```python
class TokenState(NamedTuple):
    channel: str | None
    recipient: str | None
    text: str
```
**EN:** Class `TokenState` is defined here, extending `NamedTuple`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `TokenState`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 32-167 — Function `extract_harmony_streaming_delta`
```python
def extract_harmony_streaming_delta(
    harmony_parser: StreamableParser,
    token_states: list[TokenState],
    prev_recipient: str | None,
    include_reasoning: bool,
) -> tuple[DeltaMessage | None, bool]:
    """
    Extract a DeltaMessage from harmony parser state during streaming.

    Args:
        harmony_parser: The StreamableParser instance tracking parse state
        token_states: List of TokenState tuples for each token
        prev_recipient: Previous recipient for detecting tool call transitions
        include_reasoning: Whether to include reasoning content

    Returns:
        A tuple of (DeltaMessage or None, tools_streamed_flag)
    """

    if not token_states:
        return None, False

    tools_streamed = False

    # Group consecutive tokens with same channel/recipient
    groups: list[TokenState] = []

    current_channel = token_states[0].channel
...
        if tool_messages:
            delta_kwargs["tool_calls"] = tool_messages
            tools_streamed = True
        delta_message = DeltaMessage(**delta_kwargs)
    else:
        delta_message = None

    return delta_message, tools_streamed
```
**EN:** This function `extract_harmony_streaming_delta` is documented as: Extract a DeltaMessage from harmony parser state during streaming.
**CN:** 这里定义函数 `extract_harmony_streaming_delta`，其文档字符串说明了主要职责与调用约定。

## Key Concepts / 关键概念
- Tokenization or token-level processing / 分词或 token 级处理
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing`
- **Third-party / 第三方**: `openai_harmony`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.parser.harmony_utils`
