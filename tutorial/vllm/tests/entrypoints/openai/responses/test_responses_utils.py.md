# test_responses_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_responses_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 34 test(s), 0 fixture(s), and 10 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 34 个测试、0 个 fixture，以及 10 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L25)
```python
from unittest.mock import patch

import pytest
from openai.types.responses.response_function_tool_call import ResponseFunctionToolCall
from openai.types.responses.response_function_tool_call_output_item import (
    ResponseFunctionToolCallOutputItem,
)
from openai.types.responses.response_output_message import ResponseOutputMessage
from openai.types.responses.response_output_text import ResponseOutputText
from openai.types.responses.response_reasoning_item import (
    Content,
    ResponseReasoningItem,
    Summary,
)

from vllm.entrypoints.openai.responses.utils import (
    _construct_message_from_response_item,
    construct_chat_messages_with_tool_call,
    construct_input_messages,
    convert_tool_responses_to_completions_format,
    should_continue_final_message,
)
```
**EN:** Imports standard-library modules such as `unittest.mock.patch`, third-party packages like `openai.types.responses.response_function_tool_call.ResponseFunctionToolCall`, `openai.types.responses.response_function_tool_call_output_item.ResponseFunctionToolCallOutputItem`, `openai.types.responses.response_output_message.ResponseOutputMessage`, project helpers such as `vllm.entrypoints.openai.responses.utils._construct_message_from_response_item`, `vllm.entrypoints.openai.responses.utils.construct_chat_messages_with_tool_call`, `vllm.entrypoints.openai.responses.utils.construct_input_messages`.
**CN:** 导入标准库模块（如 `unittest.mock.patch`）、第三方包（如 `openai.types.responses.response_function_tool_call.ResponseFunctionToolCall`、`openai.types.responses.response_function_tool_call_output_item.ResponseFunctionToolCallOutputItem`、`openai.types.responses.response_output_message.ResponseOutputMessage`）、项目内辅助模块（如 `vllm.entrypoints.openai.responses.utils._construct_message_from_response_item`、`vllm.entrypoints.openai.responses.utils.construct_chat_messages_with_tool_call`、`vllm.entrypoints.openai.responses.utils.construct_input_messages`）。

### Helper / 辅助函数: _single_chat_message (L28-L31)
```python
def _single_chat_message(item):
    message = _construct_message_from_response_item(item)
    assert message is not None
    return message
```
**EN:** This helper encapsulates reusable logic in `_single_chat_message`. Key inputs are `item`. It returns computed state or helper objects back to the caller. The main assertion is `message is not None`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_single_chat_message` 中。 关键输入包括 `item`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `message is not None`。

### Helper / 辅助函数: make_output_message (L34-L53)
```python
def make_output_message(
    text: str,
    *,
    id: str = "msg_1",
    status: str = "completed",
) -> ResponseOutputMessage:
    return ResponseOutputMessage(
        id=id,
        content=[
            ResponseOutputText(
                annotations=[],
                text=text,
                type="output_text",
                logprobs=None,
            )
        ],
        role="assistant",
        status=status,
        type="message",
    )
```
**EN:** This helper encapsulates reusable logic in `make_output_message`. Key inputs are `text`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `make_output_message` 中。 关键输入包括 `text`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: make_reasoning_item (L56-L78)
```python
def make_reasoning_item(
    *,
    content_text: str | None = None,
    summary_text: str | None = None,
    content: list[Content] | None = None,
    summary: list[Summary] | None = None,
    encrypted_content: str | None = None,
    id: str = "reasoning_1",
    status: str | None = None,
) -> ResponseReasoningItem:
    if content is None and content_text is not None:
        content = [Content(text=content_text, type="reasoning_text")]
    if summary is None and summary_text is not None:
        summary = [Summary(text=summary_text, type="summary_text")]

    return ResponseReasoningItem(
        id=id,
        summary=[] if summary is None else summary,
        type="reasoning",
        content=content,
        encrypted_content=encrypted_content,
        status=status,
    )
```
**EN:** This helper encapsulates reusable logic in `make_reasoning_item`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `make_reasoning_item` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: make_function_call (L81-L99)
```python
def make_function_call(
    *,
    call_id: str,
    name: str = "test_function",
    arguments: str = "{}",
    id: str = "tool_id",
    status: str | None = None,
) -> ResponseFunctionToolCall:
    kwargs = {
        "type": "function_call",
        "id": id,
        "call_id": call_id,
        "name": name,
        "arguments": arguments,
    }
    if status is not None:
        kwargs["status"] = status

    return ResponseFunctionToolCall(**kwargs)
```
**EN:** This helper encapsulates reusable logic in `make_function_call`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `make_function_call` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: make_function_call_output (L102-L115)
```python
def make_function_call_output(
    *,
    call_id: str,
    output: str = "42",
    id: str = "output_1",
    status: str = "completed",
) -> ResponseFunctionToolCallOutputItem:
    return ResponseFunctionToolCallOutputItem(
        id=id,
        type="function_call_output",
        call_id=call_id,
        output=output,
        status=status,
    )
```
**EN:** This helper encapsulates reusable logic in `make_function_call_output`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `make_function_call_output` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: TestResponsesUtils (L118-L267)
```python
class TestResponsesUtils:
    """Tests for convert_tool_responses_to_completions_format function."""

    def test_convert_tool_responses_to_completions_format(self):
        """Test basic conversion of a flat tool schema to nested format."""
        input_tool = {
            "type": "function",
            "name": "get_weather",
            "description": "Get the current weather in a given location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"},
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
# ... 128 lines omitted for brevity ...
            role="assistant",
            status="completed",
            type="message",
        )

        formatted_item = _single_chat_message(output_item)
        assert formatted_item["role"] == "assistant"
        assert formatted_item["content"] == "dongyi"
```
**EN:** This class groups related scenarios in `TestResponsesUtils`. It contains 3 test method(s) and 0 supporting method(s). Representative methods include `test_convert_tool_responses_to_completions_format`, `test_construct_chat_messages_with_tool_call`, `test_construct_chat_messages_preserves_single_item_conversions`.
**CN:** 该类将与 `TestResponsesUtils` 相关的场景组织在一起。 它包含 3 个测试方法和 0 个辅助方法。 代表性方法包括 `test_convert_tool_responses_to_completions_format`、`test_construct_chat_messages_with_tool_call`、`test_construct_chat_messages_preserves_single_item_conversions`。

### Test method / 测试方法: TestResponsesUtils.test_convert_tool_responses_to_completions_format (L121-L139)
```python
    def test_convert_tool_responses_to_completions_format(self):
        """Test basic conversion of a flat tool schema to nested format."""
        input_tool = {
            "type": "function",
            "name": "get_weather",
            "description": "Get the current weather in a given location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"},
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                },
                "required": ["location", "unit"],
            },
        }

        result = convert_tool_responses_to_completions_format(input_tool)

        assert result == {"type": "function", "function": input_tool}
```
**EN:** This test validates `TestResponsesUtils.test_convert_tool_responses_to_completions_format`. The main assertion is `result == {'type': 'function', 'function': input_tool}`.
**CN:** 这个测试验证 `TestResponsesUtils.test_convert_tool_responses_to_completions_format`。 核心断言是 `result == {'type': 'function', 'function': input_tool}`。

### Test method / 测试方法: TestResponsesUtils.test_construct_chat_messages_with_tool_call (L141-L175)
```python
    def test_construct_chat_messages_with_tool_call(self):
        """Test construction of chat messages with tool calls."""
        reasoning_item = ResponseReasoningItem(
            id="lol",
            summary=[],
            type="reasoning",
            content=[
                Content(
                    text="Leroy Jenkins",
                    type="reasoning_text",
                )
            ],
            encrypted_content=None,
            status=None,
        )
        mcp_tool_item = ResponseFunctionToolCall(
            id="mcp_123",
            call_id="call_123",
# ... 9 lines omitted for brevity ...
        message = messages[0]
        assert message["role"] == "assistant"
        assert message["reasoning"] == "Leroy Jenkins"
        assert message["tool_calls"][0]["id"] == "call_123"
        assert message["tool_calls"][0]["function"]["name"] == "python"
        assert (
            message["tool_calls"][0]["function"]["arguments"] == '{"code": "123+456"}'
        )
```
**EN:** This test validates `TestResponsesUtils.test_construct_chat_messages_with_tool_call`. The main assertion is `len(messages) == 1` and `message['role'] == 'assistant'`.
**CN:** 这个测试验证 `TestResponsesUtils.test_construct_chat_messages_with_tool_call`。 核心断言是 `len(messages) == 1` and `message['role'] == 'assistant'`。

### Test method / 测试方法: TestResponsesUtils.test_construct_chat_messages_preserves_single_item_conversions (L177-L267)
```python
    def test_construct_chat_messages_preserves_single_item_conversions(self):
        item = ResponseReasoningItem(
            id="lol",
            summary=[],
            type="reasoning",
            content=[
                Content(
                    text="Leroy Jenkins",
                    type="reasoning_text",
                )
            ],
            encrypted_content=None,
            status=None,
        )
        formatted_item = _single_chat_message(item)
        assert formatted_item["role"] == "assistant"
        assert formatted_item["reasoning"] == "Leroy Jenkins"

# ... 65 lines omitted for brevity ...
            role="assistant",
            status="completed",
            type="message",
        )

        formatted_item = _single_chat_message(output_item)
        assert formatted_item["role"] == "assistant"
        assert formatted_item["content"] == "dongyi"
```
**EN:** This test validates `TestResponsesUtils.test_construct_chat_messages_preserves_single_item_conversions`. It checks an expected failure path with `pytest.raises`. The main assertion is `formatted_item['role'] == 'assistant'` and `formatted_item['reasoning'] == 'Leroy Jenkins'`.
**CN:** 这个测试验证 `TestResponsesUtils.test_construct_chat_messages_preserves_single_item_conversions`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `formatted_item['role'] == 'assistant'` and `formatted_item['reasoning'] == 'Leroy Jenkins'`。

### Class / 类: TestReasoningItemContentPriority (L270-L445)
```python
class TestReasoningItemContentPriority:
    """Tests that content is prioritized over summary for reasoning items."""

    def test_content_preferred_over_summary(self):
        """When both content and summary are present, content should win."""
        item = ResponseReasoningItem(
            id="reasoning_1",
            summary=[
                Summary(
                    text="This is a summary",
                    type="summary_text",
                )
            ],
            type="reasoning",
# ... 154 lines omitted for brevity ...
                    type="reasoning_text",
                )
            ],
            encrypted_content=None,
            status=None,
        )
        construct_chat_messages_with_tool_call([item])
        mock_logger.warning.assert_not_called()
```
**EN:** This class groups related scenarios in `TestReasoningItemContentPriority`. It contains 8 test method(s) and 0 supporting method(s). Representative methods include `test_content_preferred_over_summary`, `test_content_only`, `test_summary_fallback_when_no_content`.
**CN:** 该类将与 `TestReasoningItemContentPriority` 相关的场景组织在一起。 它包含 8 个测试方法和 0 个辅助方法。 代表性方法包括 `test_content_preferred_over_summary`、`test_content_only`、`test_summary_fallback_when_no_content`。

### Test method / 测试方法: TestReasoningItemContentPriority.test_content_preferred_over_summary (L273-L294)
```python
    def test_content_preferred_over_summary(self):
        """When both content and summary are present, content should win."""
        item = ResponseReasoningItem(
            id="reasoning_1",
            summary=[
                Summary(
                    text="This is a summary",
                    type="summary_text",
                )
            ],
            type="reasoning",
            content=[
                Content(
                    text="This is the actual content",
                    type="reasoning_text",
                )
            ],
            encrypted_content=None,
            status=None,
        )
        formatted = _single_chat_message(item)
        assert formatted["reasoning"] == "This is the actual content"
```
**EN:** This test validates `TestReasoningItemContentPriority.test_content_preferred_over_summary`. The main assertion is `formatted['reasoning'] == 'This is the actual content'`.
**CN:** 这个测试验证 `TestReasoningItemContentPriority.test_content_preferred_over_summary`。 核心断言是 `formatted['reasoning'] == 'This is the actual content'`。

### Test method / 测试方法: TestReasoningItemContentPriority.test_content_only (L296-L312)
```python
    def test_content_only(self):
        """When only content is present (no summary), content is used."""
        item = ResponseReasoningItem(
            id="reasoning_2",
            summary=[],
            type="reasoning",
            content=[
                Content(
                    text="Content without summary",
                    type="reasoning_text",
                )
            ],
            encrypted_content=None,
            status=None,
        )
        formatted = _single_chat_message(item)
        assert formatted["reasoning"] == "Content without summary"
```
**EN:** This test validates `TestReasoningItemContentPriority.test_content_only`. The main assertion is `formatted['reasoning'] == 'Content without summary'`.
**CN:** 这个测试验证 `TestReasoningItemContentPriority.test_content_only`。 核心断言是 `formatted['reasoning'] == 'Content without summary'`。

### Test method / 测试方法: TestReasoningItemContentPriority.test_summary_fallback_when_no_content (L314-L335)
```python
    @patch("vllm.entrypoints.openai.responses.utils.logger")
    def test_summary_fallback_when_no_content(self, mock_logger):
        """When content is absent, summary is used as fallback with warning."""
        item = ResponseReasoningItem(
            id="reasoning_3",
            summary=[
                Summary(
                    text="Fallback summary text",
                    type="summary_text",
                )
            ],
            type="reasoning",
            content=None,
            encrypted_content=None,
            status=None,
        )
        formatted = _single_chat_message(item)
        assert formatted["reasoning"] == "Fallback summary text"
        mock_logger.warning.assert_called_once()
        assert (
            "summary text as reasoning content" in mock_logger.warning.call_args[0][0]
        )
```
**EN:** This test validates `TestReasoningItemContentPriority.test_summary_fallback_when_no_content`. Key inputs are `mock_logger`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `formatted['reasoning'] == 'Fallback summary text'` and `'summary text as reasoning content' in mock_logger.warning.call_args[0][0]`.
**CN:** 这个测试验证 `TestReasoningItemContentPriority.test_summary_fallback_when_no_content`。 关键输入包括 `mock_logger`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `formatted['reasoning'] == 'Fallback summary text'` and `'summary text as reasoning content' in mock_logger.warning.call_args[0][0]`。

### Test method / 测试方法: TestReasoningItemContentPriority.test_summary_fallback_when_content_empty (L337-L358)
```python
    @patch("vllm.entrypoints.openai.responses.utils.logger")
    def test_summary_fallback_when_content_empty(self, mock_logger):
        """When content is an empty list, summary is used as fallback."""
        item = ResponseReasoningItem(
            id="reasoning_4",
            summary=[
                Summary(
                    text="Summary when content empty",
                    type="summary_text",
                )
            ],
            type="reasoning",
            content=[],
            encrypted_content=None,
            status=None,
        )
        formatted = _single_chat_message(item)
        assert formatted["reasoning"] == "Summary when content empty"
        mock_logger.warning.assert_called_once()
        assert (
            "summary text as reasoning content" in mock_logger.warning.call_args[0][0]
        )
```
**EN:** This test validates `TestReasoningItemContentPriority.test_summary_fallback_when_content_empty`. Key inputs are `mock_logger`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `formatted['reasoning'] == 'Summary when content empty'` and `'summary text as reasoning content' in mock_logger.warning.call_args[0][0]`.
**CN:** 这个测试验证 `TestReasoningItemContentPriority.test_summary_fallback_when_content_empty`。 关键输入包括 `mock_logger`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `formatted['reasoning'] == 'Summary when content empty'` and `'summary text as reasoning content' in mock_logger.warning.call_args[0][0]`。

### Test method / 测试方法: TestReasoningItemContentPriority.test_neither_content_nor_summary (L360-L371)
```python
    def test_neither_content_nor_summary(self):
        """When neither content nor summary is present, reasoning is empty."""
        item = ResponseReasoningItem(
            id="reasoning_5",
            summary=[],
            type="reasoning",
            content=None,
            encrypted_content=None,
            status=None,
        )
        formatted = _single_chat_message(item)
        assert formatted["reasoning"] == ""
```
**EN:** This test validates `TestReasoningItemContentPriority.test_neither_content_nor_summary`. The main assertion is `formatted['reasoning'] == ''`.
**CN:** 这个测试验证 `TestReasoningItemContentPriority.test_neither_content_nor_summary`。 核心断言是 `formatted['reasoning'] == ''`。

### Test method / 测试方法: TestReasoningItemContentPriority.test_encrypted_content_raises (L373-L394)
```python
    def test_encrypted_content_raises(self):
        """Encrypted content should still raise ValueError."""
        item = ResponseReasoningItem(
            id="reasoning_6",
            summary=[
                Summary(
                    text="Some summary",
                    type="summary_text",
                )
            ],
            type="reasoning",
            content=[
                Content(
                    text="Some content",
                    type="reasoning_text",
                )
            ],
            encrypted_content="ENCRYPTED",
            status=None,
        )
        with pytest.raises(ValueError):
            construct_chat_messages_with_tool_call([item])
```
**EN:** This test validates `TestReasoningItemContentPriority.test_encrypted_content_raises`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `TestReasoningItemContentPriority.test_encrypted_content_raises`。 它使用 `pytest.raises` 检查预期失败路径。

### Test method / 测试方法: TestReasoningItemContentPriority.test_summary_with_multiple_entries_uses_first (L396-L421)
```python
    @patch("vllm.entrypoints.openai.responses.utils.logger")
    def test_summary_with_multiple_entries_uses_first(self, mock_logger):
        """When multiple summary entries exist, the first one is used."""
        item = ResponseReasoningItem(
            id="reasoning_7",
            summary=[
                Summary(
                    text="First summary",
                    type="summary_text",
                ),
                Summary(
                    text="Second summary",
                    type="summary_text",
                ),
            ],
            type="reasoning",
            content=None,
            encrypted_content=None,
            status=None,
        )
        formatted = _single_chat_message(item)
        assert formatted["reasoning"] == "First summary"
        mock_logger.warning.assert_called_once()
        assert (
            "summary text as reasoning content" in mock_logger.warning.call_args[0][0]
        )
```
**EN:** This test validates `TestReasoningItemContentPriority.test_summary_with_multiple_entries_uses_first`. Key inputs are `mock_logger`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `formatted['reasoning'] == 'First summary'` and `'summary text as reasoning content' in mock_logger.warning.call_args[0][0]`.
**CN:** 这个测试验证 `TestReasoningItemContentPriority.test_summary_with_multiple_entries_uses_first`。 关键输入包括 `mock_logger`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `formatted['reasoning'] == 'First summary'` and `'summary text as reasoning content' in mock_logger.warning.call_args[0][0]`。

### Test method / 测试方法: TestReasoningItemContentPriority.test_no_warning_when_content_used (L423-L445)
```python
    @patch("vllm.entrypoints.openai.responses.utils.logger")
    def test_no_warning_when_content_used(self, mock_logger):
        """No warning should be emitted when content is available."""
        item = ResponseReasoningItem(
            id="reasoning_8",
            summary=[
                Summary(
                    text="Summary text",
                    type="summary_text",
                )
            ],
            type="reasoning",
            content=[
                Content(
                    text="Content text",
                    type="reasoning_text",
                )
            ],
            encrypted_content=None,
            status=None,
        )
        construct_chat_messages_with_tool_call([item])
        mock_logger.warning.assert_not_called()
```
**EN:** This test validates `TestReasoningItemContentPriority.test_no_warning_when_content_used`. Key inputs are `mock_logger`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `TestReasoningItemContentPriority.test_no_warning_when_content_used`。 关键输入包括 `mock_logger`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Class / 类: TestShouldContinueFinalMessage (L448-L724)
```python
class TestShouldContinueFinalMessage:
    """Tests for should_continue_final_message function.

    This function enables Anthropic-style partial message completion, where
    users can provide an incomplete assistant message and have the model
    continue from where it left off.
    """

    def test_string_input_returns_false(self):
        """String input is always a user message, so should not continue."""
        assert should_continue_final_message("Hello, world!") is False

    def test_empty_list_returns_false(self):
        """Empty list should not continue."""
# ... 255 lines omitted for brevity ...
        dict_item = {
            "id": "msg_123",
            "type": "message",
            "role": "assistant",
            "status": None,
            "content": [{"type": "output_text", "text": "Some text"}],
        }
        assert should_continue_final_message([dict_item]) is False
```
**EN:** This class groups related scenarios in `TestShouldContinueFinalMessage`. It contains 17 test method(s) and 0 supporting method(s). Representative methods include `test_string_input_returns_false`, `test_empty_list_returns_false`, `test_completed_message_returns_false`.
**CN:** 该类将与 `TestShouldContinueFinalMessage` 相关的场景组织在一起。 它包含 17 个测试方法和 0 个辅助方法。 代表性方法包括 `test_string_input_returns_false`、`test_empty_list_returns_false`、`test_completed_message_returns_false`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_string_input_returns_false (L456-L458)
```python
    def test_string_input_returns_false(self):
        """String input is always a user message, so should not continue."""
        assert should_continue_final_message("Hello, world!") is False
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_string_input_returns_false`. The main assertion is `should_continue_final_message('Hello, world!') is False`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_string_input_returns_false`。 核心断言是 `should_continue_final_message('Hello, world!') is False`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_empty_list_returns_false (L460-L462)
```python
    def test_empty_list_returns_false(self):
        """Empty list should not continue."""
        assert should_continue_final_message([]) is False
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_empty_list_returns_false`. The main assertion is `should_continue_final_message([]) is False`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_empty_list_returns_false`。 核心断言是 `should_continue_final_message([]) is False`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_completed_message_returns_false (L464-L480)
```python
    def test_completed_message_returns_false(self):
        """Completed message should not be continued."""
        output_item = ResponseOutputMessage(
            id="msg_123",
            content=[
                ResponseOutputText(
                    annotations=[],
                    text="The answer is 42.",
                    type="output_text",
                    logprobs=None,
                )
            ],
            role="assistant",
            status="completed",
            type="message",
        )
        assert should_continue_final_message([output_item]) is False
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_completed_message_returns_false`. The main assertion is `should_continue_final_message([output_item]) is False`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_completed_message_returns_false`。 核心断言是 `should_continue_final_message([output_item]) is False`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_in_progress_message_returns_true (L482-L503)
```python
    def test_in_progress_message_returns_true(self):
        """In-progress message should be continued.

        This is the key use case for partial message completion.
        Example: The user provides "The best answer is (" and wants
        the model to continue from there.
        """
        output_item = ResponseOutputMessage(
            id="msg_123",
            content=[
                ResponseOutputText(
                    annotations=[],
                    text="The best answer is (",
                    type="output_text",
                    logprobs=None,
                )
            ],
            role="assistant",
            status="in_progress",
            type="message",
        )
        assert should_continue_final_message([output_item]) is True
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_in_progress_message_returns_true`. The main assertion is `should_continue_final_message([output_item]) is True`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_in_progress_message_returns_true`。 核心断言是 `should_continue_final_message([output_item]) is True`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_incomplete_message_returns_true (L505-L521)
```python
    def test_incomplete_message_returns_true(self):
        """Incomplete message should be continued."""
        output_item = ResponseOutputMessage(
            id="msg_123",
            content=[
                ResponseOutputText(
                    annotations=[],
                    text="The answer",
                    type="output_text",
                    logprobs=None,
                )
            ],
            role="assistant",
            status="incomplete",
            type="message",
        )
        assert should_continue_final_message([output_item]) is True
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_incomplete_message_returns_true`. The main assertion is `should_continue_final_message([output_item]) is True`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_incomplete_message_returns_true`。 核心断言是 `should_continue_final_message([output_item]) is True`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_in_progress_reasoning_returns_true (L523-L538)
```python
    def test_in_progress_reasoning_returns_true(self):
        """In-progress reasoning should be continued."""
        reasoning_item = ResponseReasoningItem(
            id="reasoning_123",
            summary=[],
            type="reasoning",
            content=[
                Content(
                    text="Let me think about this...",
                    type="reasoning_text",
                )
            ],
            encrypted_content=None,
            status="in_progress",
        )
        assert should_continue_final_message([reasoning_item]) is True
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_in_progress_reasoning_returns_true`. The main assertion is `should_continue_final_message([reasoning_item]) is True`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_in_progress_reasoning_returns_true`。 核心断言是 `should_continue_final_message([reasoning_item]) is True`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_incomplete_reasoning_returns_true (L540-L564)
```python
    def test_incomplete_reasoning_returns_true(self):
        """Incomplete reasoning should be continued."""
        reasoning_item = ResponseReasoningItem(
            id="reasoning_123",
            summary=[],
            type="reasoning",
            content=[
                Content(
                    text="Let me think",
                    type="reasoning_text",
                )
            ],
            encrypted_content=None,
            status="incomplete",
        )
        assert should_continue_final_message([reasoning_item]) is True

        reasoning_item = {
            "id": "reasoning_123",
            "summary": [],
            "type": "reasoning",
            "content": [],
            "status": "incomplete",
        }
        assert should_continue_final_message([reasoning_item]) is True
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_incomplete_reasoning_returns_true`. The main assertion is `should_continue_final_message([reasoning_item]) is True` and `should_continue_final_message([reasoning_item]) is True`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_incomplete_reasoning_returns_true`。 核心断言是 `should_continue_final_message([reasoning_item]) is True` and `should_continue_final_message([reasoning_item]) is True`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_completed_reasoning_returns_false (L566-L581)
```python
    def test_completed_reasoning_returns_false(self):
        """Completed reasoning should not be continued."""
        reasoning_item = ResponseReasoningItem(
            id="reasoning_123",
            summary=[],
            type="reasoning",
            content=[
                Content(
                    text="I have thought about this.",
                    type="reasoning_text",
                )
            ],
            encrypted_content=None,
            status="completed",
        )
        assert should_continue_final_message([reasoning_item]) is False
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_completed_reasoning_returns_false`. The main assertion is `should_continue_final_message([reasoning_item]) is False`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_completed_reasoning_returns_false`。 核心断言是 `should_continue_final_message([reasoning_item]) is False`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_reasoning_with_none_status_returns_false (L583-L598)
```python
    def test_reasoning_with_none_status_returns_false(self):
        """Reasoning with None status should not be continued."""
        reasoning_item = ResponseReasoningItem(
            id="reasoning_123",
            summary=[],
            type="reasoning",
            content=[
                Content(
                    text="Some reasoning",
                    type="reasoning_text",
                )
            ],
            encrypted_content=None,
            status=None,
        )
        assert should_continue_final_message([reasoning_item]) is False
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_reasoning_with_none_status_returns_false`. The main assertion is `should_continue_final_message([reasoning_item]) is False`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_reasoning_with_none_status_returns_false`。 核心断言是 `should_continue_final_message([reasoning_item]) is False`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_only_last_item_matters (L600-L637)
```python
    def test_only_last_item_matters(self):
        """Only the last item in the list determines continuation."""
        completed_item = ResponseOutputMessage(
            id="msg_1",
            content=[
                ResponseOutputText(
                    annotations=[],
                    text="Complete message.",
                    type="output_text",
                    logprobs=None,
                )
            ],
            role="assistant",
            status="completed",
            type="message",
        )
        in_progress_item = ResponseOutputMessage(
            id="msg_2",
# ... 12 lines omitted for brevity ...

        # In-progress as last item -> should continue
        assert should_continue_final_message([completed_item, in_progress_item]) is True

        # Completed as last item -> should not continue
        assert (
            should_continue_final_message([in_progress_item, completed_item]) is False
        )
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_only_last_item_matters`. The main assertion is `should_continue_final_message([completed_item, in_progress_item]) is True` and `should_continue_final_message([in_progress_item, completed_item]) is False`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_only_last_item_matters`。 核心断言是 `should_continue_final_message([completed_item, in_progress_item]) is True` and `should_continue_final_message([in_progress_item, completed_item]) is False`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_tool_call_returns_false (L639-L659)
```python
    def test_tool_call_returns_false(self):
        """Tool calls should not trigger continuation."""
        tool_call = ResponseFunctionToolCall(
            id="fc_123",
            call_id="call_123",
            type="function_call",
            status="in_progress",
            name="get_weather",
            arguments='{"location": "NYC"}',
        )
        assert should_continue_final_message([tool_call]) is False

        tool_call = {
            "id": "msg_123",
            "call_id": "call_123",
            "type": "function_call",
            "status": "in_progress",
            "name": "get_weather",
            "arguments": '{"location": "NYC"}',
        }
        assert should_continue_final_message([tool_call]) is False
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_tool_call_returns_false`. The main assertion is `should_continue_final_message([tool_call]) is False` and `should_continue_final_message([tool_call]) is False`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_tool_call_returns_false`。 核心断言是 `should_continue_final_message([tool_call]) is False` and `should_continue_final_message([tool_call]) is False`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_dict_in_progress_message_returns_true (L662-L671)
```python
    def test_dict_in_progress_message_returns_true(self):
        """Dict with in_progress status should be continued (curl input)."""
        dict_item = {
            "id": "msg_123",
            "type": "message",
            "role": "assistant",
            "status": "in_progress",
            "content": [{"type": "output_text", "text": "The answer is ("}],
        }
        assert should_continue_final_message([dict_item]) is True
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_dict_in_progress_message_returns_true`. The main assertion is `should_continue_final_message([dict_item]) is True`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_dict_in_progress_message_returns_true`。 核心断言是 `should_continue_final_message([dict_item]) is True`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_dict_incomplete_message_returns_true (L673-L682)
```python
    def test_dict_incomplete_message_returns_true(self):
        """Dict with incomplete status should be continued (curl input)."""
        dict_item = {
            "id": "msg_123",
            "type": "message",
            "role": "assistant",
            "status": "incomplete",
            "content": [{"type": "output_text", "text": "Partial answer"}],
        }
        assert should_continue_final_message([dict_item]) is True
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_dict_incomplete_message_returns_true`. The main assertion is `should_continue_final_message([dict_item]) is True`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_dict_incomplete_message_returns_true`。 核心断言是 `should_continue_final_message([dict_item]) is True`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_dict_completed_message_returns_false (L684-L693)
```python
    def test_dict_completed_message_returns_false(self):
        """Dict with completed status should not be continued (curl input)."""
        dict_item = {
            "id": "msg_123",
            "type": "message",
            "role": "assistant",
            "status": "completed",
            "content": [{"type": "output_text", "text": "Complete answer."}],
        }
        assert should_continue_final_message([dict_item]) is False
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_dict_completed_message_returns_false`. The main assertion is `should_continue_final_message([dict_item]) is False`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_dict_completed_message_returns_false`。 核心断言是 `should_continue_final_message([dict_item]) is False`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_dict_reasoning_in_progress_returns_true (L695-L703)
```python
    def test_dict_reasoning_in_progress_returns_true(self):
        """Dict reasoning item with in_progress status should be continued."""
        dict_item = {
            "id": "reasoning_123",
            "type": "reasoning",
            "status": "in_progress",
            "content": [{"type": "reasoning_text", "text": "Let me think..."}],
        }
        assert should_continue_final_message([dict_item]) is True
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_dict_reasoning_in_progress_returns_true`. The main assertion is `should_continue_final_message([dict_item]) is True`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_dict_reasoning_in_progress_returns_true`。 核心断言是 `should_continue_final_message([dict_item]) is True`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_dict_without_status_returns_false (L705-L713)
```python
    def test_dict_without_status_returns_false(self):
        """Dict without status field should not be continued."""
        dict_item = {
            "id": "msg_123",
            "type": "message",
            "role": "assistant",
            "content": [{"type": "output_text", "text": "Some text"}],
        }
        assert should_continue_final_message([dict_item]) is False
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_dict_without_status_returns_false`. The main assertion is `should_continue_final_message([dict_item]) is False`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_dict_without_status_returns_false`。 核心断言是 `should_continue_final_message([dict_item]) is False`。

### Test method / 测试方法: TestShouldContinueFinalMessage.test_dict_with_none_status_returns_false (L715-L724)
```python
    def test_dict_with_none_status_returns_false(self):
        """Dict with None status should not be continued."""
        dict_item = {
            "id": "msg_123",
            "type": "message",
            "role": "assistant",
            "status": None,
            "content": [{"type": "output_text", "text": "Some text"}],
        }
        assert should_continue_final_message([dict_item]) is False
```
**EN:** This test validates `TestShouldContinueFinalMessage.test_dict_with_none_status_returns_false`. The main assertion is `should_continue_final_message([dict_item]) is False`.
**CN:** 这个测试验证 `TestShouldContinueFinalMessage.test_dict_with_none_status_returns_false`。 核心断言是 `should_continue_final_message([dict_item]) is False`。

### Class / 类: TestConstructChatMessagesCombinePolicy (L727-L845)
```python
class TestConstructChatMessagesCombinePolicy:
    """Tests for contiguous assistant-side merging."""

    @pytest.mark.parametrize(
        ("items", "expected_content", "expected_reasoning", "expected_tool_call_ids"),
        [
            pytest.param(
                [
                    make_reasoning_item(content_text="Let me think"),
                    make_output_message("Hello"),
                ],
                "Hello",
                "Let me think",
                None,
# ... 97 lines omitted for brevity ...
                3,
                id="interrupted-by-non-assistant-item",
            ),
        ],
    )
    def test_merge_chain_breaks(self, items, num_expected_messages):
        messages = construct_chat_messages_with_tool_call(items)
        assert len(messages) == num_expected_messages
```
**EN:** This class groups related scenarios in `TestConstructChatMessagesCombinePolicy`. It contains 2 test method(s) and 0 supporting method(s). Representative methods include `test_assistant_side_items_merge_until_tool_output`, `test_merge_chain_breaks`.
**CN:** 该类将与 `TestConstructChatMessagesCombinePolicy` 相关的场景组织在一起。 它包含 2 个测试方法和 0 个辅助方法。 代表性方法包括 `test_assistant_side_items_merge_until_tool_output`、`test_merge_chain_breaks`。

### Test method / 测试方法: TestConstructChatMessagesCombinePolicy.test_assistant_side_items_merge_until_tool_output (L730-L811)
```python
    @pytest.mark.parametrize(
        ("items", "expected_content", "expected_reasoning", "expected_tool_call_ids"),
        [
            pytest.param(
                [
                    make_reasoning_item(content_text="Let me think"),
                    make_output_message("Hello"),
                ],
                "Hello",
                "Let me think",
                None,
                id="reasoning-output-messages",
            ),
            pytest.param(
                [
                    make_function_call(call_id="call_123"),
                    make_function_call(call_id="call_456"),
                ],
# ... 56 lines omitted for brevity ...
        else:
            assert messages[0]["reasoning"] == expected_reasoning
        if expected_tool_call_ids is None:
            assert "tool_calls" not in messages[0]
        else:
            assert [tool_call["id"] for tool_call in messages[0]["tool_calls"]] == (
                expected_tool_call_ids
            )
```
**EN:** This test validates `TestConstructChatMessagesCombinePolicy.test_assistant_side_items_merge_until_tool_output`. It uses parameterization over `items`, `expected_content`, `expected_reasoning`, `expected_tool_call_ids`. Key inputs are `items`, `expected_content`, `expected_reasoning`, `expected_tool_call_ids`. The main assertion is `len(messages) == 1` and `messages[0]['role'] == 'assistant'`.
**CN:** 这个测试验证 `TestConstructChatMessagesCombinePolicy.test_assistant_side_items_merge_until_tool_output`。 它通过参数化组合 `items`、`expected_content`、`expected_reasoning`、`expected_tool_call_ids`。 关键输入包括 `items`、`expected_content`、`expected_reasoning`、`expected_tool_call_ids`。 核心断言是 `len(messages) == 1` and `messages[0]['role'] == 'assistant'`。

### Test method / 测试方法: TestConstructChatMessagesCombinePolicy.test_merge_chain_breaks (L813-L845)
```python
    @pytest.mark.parametrize(
        ("items", "num_expected_messages"),
        [
            pytest.param(
                [
                    make_output_message("Hello"),
                    make_output_message("World"),
                ],
                2,
                id="consecutive-output-messages",
            ),
            pytest.param(
                [
                    make_reasoning_item(content_text="Let me think"),
                    make_reasoning_item(content_text="Let me think more"),
                ],
                2,
                id="consecutive-reasoning-messages",
# ... 7 lines omitted for brevity ...
                3,
                id="interrupted-by-non-assistant-item",
            ),
        ],
    )
    def test_merge_chain_breaks(self, items, num_expected_messages):
        messages = construct_chat_messages_with_tool_call(items)
        assert len(messages) == num_expected_messages
```
**EN:** This test validates `TestConstructChatMessagesCombinePolicy.test_merge_chain_breaks`. It uses parameterization over `items`, `num_expected_messages`. Key inputs are `items`, `num_expected_messages`. The main assertion is `len(messages) == num_expected_messages`.
**CN:** 这个测试验证 `TestConstructChatMessagesCombinePolicy.test_merge_chain_breaks`。 它通过参数化组合 `items`、`num_expected_messages`。 关键输入包括 `items`、`num_expected_messages`。 核心断言是 `len(messages) == num_expected_messages`。

### Class / 类: TestConstructInputMessagesInstructionsLeak (L848-L913)
```python
class TestConstructInputMessagesInstructionsLeak:
    """Regression tests for #37697: instructions from a prior response
    should NOT leak through previous_response_id."""

    def test_old_instructions_stripped_from_prev_msg(self):
        """System message in prev_msg must be dropped so the new request's
        instructions are the only system message in the conversation."""
        prev = [
            {"role": "system", "content": "old instructions"},
            {"role": "user", "content": "What is 2+2?"},
            {"role": "assistant", "content": "4"},
        ]
        msgs = construct_input_messages(
            request_instructions="new instructions",
# ... 44 lines omitted for brevity ...
        msgs = construct_input_messages(
            request_instructions="be helpful",
            request_input="hello",
            prev_msg=None,
        )
        assert len(msgs) == 2
        assert msgs[0] == {"role": "system", "content": "be helpful"}
        assert msgs[1] == {"role": "user", "content": "hello"}
```
**EN:** This class groups related scenarios in `TestConstructInputMessagesInstructionsLeak`. It contains 4 test method(s) and 0 supporting method(s). Representative methods include `test_old_instructions_stripped_from_prev_msg`, `test_no_instructions_in_new_request`, `test_non_system_messages_preserved`.
**CN:** 该类将与 `TestConstructInputMessagesInstructionsLeak` 相关的场景组织在一起。 它包含 4 个测试方法和 0 个辅助方法。 代表性方法包括 `test_old_instructions_stripped_from_prev_msg`、`test_no_instructions_in_new_request`、`test_non_system_messages_preserved`。

### Test method / 测试方法: TestConstructInputMessagesInstructionsLeak.test_old_instructions_stripped_from_prev_msg (L852-L867)
```python
    def test_old_instructions_stripped_from_prev_msg(self):
        """System message in prev_msg must be dropped so the new request's
        instructions are the only system message in the conversation."""
        prev = [
            {"role": "system", "content": "old instructions"},
            {"role": "user", "content": "What is 2+2?"},
            {"role": "assistant", "content": "4"},
        ]
        msgs = construct_input_messages(
            request_instructions="new instructions",
            request_input="What is 3+3?",
            prev_msg=prev,
        )
        system_msgs = [m for m in msgs if m.get("role") == "system"]
        assert len(system_msgs) == 1
        assert system_msgs[0]["content"] == "new instructions"
```
**EN:** This test validates `TestConstructInputMessagesInstructionsLeak.test_old_instructions_stripped_from_prev_msg`. The main assertion is `len(system_msgs) == 1` and `system_msgs[0]['content'] == 'new instructions'`.
**CN:** 这个测试验证 `TestConstructInputMessagesInstructionsLeak.test_old_instructions_stripped_from_prev_msg`。 核心断言是 `len(system_msgs) == 1` and `system_msgs[0]['content'] == 'new instructions'`。

### Test method / 测试方法: TestConstructInputMessagesInstructionsLeak.test_no_instructions_in_new_request (L869-L883)
```python
    def test_no_instructions_in_new_request(self):
        """If the new request has no instructions, old ones should still
        be stripped -- they must not carry over."""
        prev = [
            {"role": "system", "content": "old instructions"},
            {"role": "user", "content": "Hi"},
            {"role": "assistant", "content": "Hello"},
        ]
        msgs = construct_input_messages(
            request_instructions=None,
            request_input="What is 3+3?",
            prev_msg=prev,
        )
        system_msgs = [m for m in msgs if m.get("role") == "system"]
        assert len(system_msgs) == 0
```
**EN:** This test validates `TestConstructInputMessagesInstructionsLeak.test_no_instructions_in_new_request`. The main assertion is `len(system_msgs) == 0`.
**CN:** 这个测试验证 `TestConstructInputMessagesInstructionsLeak.test_no_instructions_in_new_request`。 核心断言是 `len(system_msgs) == 0`。

### Test method / 测试方法: TestConstructInputMessagesInstructionsLeak.test_non_system_messages_preserved (L885-L902)
```python
    def test_non_system_messages_preserved(self):
        """User/assistant messages from prev_msg must remain intact."""
        prev = [
            {"role": "system", "content": "old instructions"},
            {"role": "user", "content": "Hi"},
            {"role": "assistant", "content": "Hello"},
        ]
        msgs = construct_input_messages(
            request_instructions="new instructions",
            request_input="Follow up",
            prev_msg=prev,
        )
        roles = [m["role"] for m in msgs]
        assert roles == ["system", "user", "assistant", "user"]
        assert msgs[0]["content"] == "new instructions"
        assert msgs[1]["content"] == "Hi"
        assert msgs[2]["content"] == "Hello"
        assert msgs[3]["content"] == "Follow up"
```
**EN:** This test validates `TestConstructInputMessagesInstructionsLeak.test_non_system_messages_preserved`. The main assertion is `roles == ['system', 'user', 'assistant', 'user']` and `msgs[0]['content'] == 'new instructions'`.
**CN:** 这个测试验证 `TestConstructInputMessagesInstructionsLeak.test_non_system_messages_preserved`。 核心断言是 `roles == ['system', 'user', 'assistant', 'user']` and `msgs[0]['content'] == 'new instructions'`。

### Test method / 测试方法: TestConstructInputMessagesInstructionsLeak.test_no_prev_msg (L904-L913)
```python
    def test_no_prev_msg(self):
        """Baseline: when there's no prev_msg, instructions work normally."""
        msgs = construct_input_messages(
            request_instructions="be helpful",
            request_input="hello",
            prev_msg=None,
        )
        assert len(msgs) == 2
        assert msgs[0] == {"role": "system", "content": "be helpful"}
        assert msgs[1] == {"role": "user", "content": "hello"}
```
**EN:** This test validates `TestConstructInputMessagesInstructionsLeak.test_no_prev_msg`. The main assertion is `len(msgs) == 2` and `msgs[0] == {'role': 'system', 'content': 'be helpful'}`.
**CN:** 这个测试验证 `TestConstructInputMessagesInstructionsLeak.test_no_prev_msg`。 核心断言是 `len(msgs) == 2` and `msgs[0] == {'role': 'system', 'content': 'be helpful'}`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `unittest.mock.patch`
- **Third-party / 第三方**: `openai.types.responses.response_function_tool_call.ResponseFunctionToolCall`, `openai.types.responses.response_function_tool_call_output_item.ResponseFunctionToolCallOutputItem`, `openai.types.responses.response_output_message.ResponseOutputMessage`, `openai.types.responses.response_output_text.ResponseOutputText`, `openai.types.responses.response_reasoning_item.Content`, `openai.types.responses.response_reasoning_item.ResponseReasoningItem`, `openai.types.responses.response_reasoning_item.Summary`, `pytest`
- **Project / 项目内**: `vllm.entrypoints.openai.responses.utils._construct_message_from_response_item`, `vllm.entrypoints.openai.responses.utils.construct_chat_messages_with_tool_call`, `vllm.entrypoints.openai.responses.utils.construct_input_messages`, `vllm.entrypoints.openai.responses.utils.convert_tool_responses_to_completions_format`, `vllm.entrypoints.openai.responses.utils.should_continue_final_message`
