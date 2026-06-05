# test_serving_chat_stream_harmony.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_serving_chat_stream_harmony.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 19 test(s), 0 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 19 个测试、0 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L7-L15)
```python
from dataclasses import dataclass, field
from unittest.mock import patch

import pytest

from vllm.entrypoints.openai.chat_completion.stream_harmony import (
    TokenState,
    extract_harmony_streaming_delta,
)
```
**EN:** Imports standard-library modules such as `dataclasses.dataclass`, `dataclasses.field`, `unittest.mock.patch`, third-party packages like `pytest`, project helpers such as `vllm.entrypoints.openai.chat_completion.stream_harmony.TokenState`, `vllm.entrypoints.openai.chat_completion.stream_harmony.extract_harmony_streaming_delta`.
**CN:** 导入标准库模块（如 `dataclasses.dataclass`、`dataclasses.field`、`unittest.mock.patch`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.entrypoints.openai.chat_completion.stream_harmony.TokenState`、`vllm.entrypoints.openai.chat_completion.stream_harmony.extract_harmony_streaming_delta`）。

### Class / 类: MockMessage (L18-L23)
```python
@dataclass
class MockMessage:
    """Mock message object for testing."""

    channel: str | None = None
    recipient: str | None = None
```
**EN:** This class groups related scenarios in `MockMessage`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockMessage` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: MockStreamableParser (L26-L30)
```python
@dataclass
class MockStreamableParser:
    """Mock StreamableParser for testing without openai_harmony dependency."""

    messages: list[MockMessage] = field(default_factory=list)
```
**EN:** This class groups related scenarios in `MockStreamableParser`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockStreamableParser` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: TestExtractHarmonyStreamingDelta (L33-L445)
```python
class TestExtractHarmonyStreamingDelta:
    """Tests for extract_harmony_streaming_delta function."""

    @pytest.mark.parametrize(
        "delta_text,expected_content",
        [
            ("Hello, world!", "Hello, world!"),
            ("", ""),
        ],
    )
    def test_final_channel_returns_content_delta(self, delta_text, expected_content):
        """Test that final channel returns a DeltaMessage with content."""
        parser = MockStreamableParser()

# ... 391 lines omitted for brevity ...
        tool_c_start = next(t for t in delta_message.tool_calls if t.id == "id_c")
        assert tool_c_start.index == 3
        tool_c_args = next(
            t for t in delta_message.tool_calls if t.index == 3 and t.id is None
        )
        assert tool_c_args.function.arguments == '{"key_c": "val_c"}'

        assert delta_message.content == "Thinking... Thinking again..."
```
**EN:** This class groups related scenarios in `TestExtractHarmonyStreamingDelta`. It contains 15 test method(s) and 0 supporting method(s). Representative methods include `test_final_channel_returns_content_delta`, `test_analysis_channel_reasoning`, `test_new_tool_call`.
**CN:** 该类将与 `TestExtractHarmonyStreamingDelta` 相关的场景组织在一起。 它包含 15 个测试方法和 0 个辅助方法。 代表性方法包括 `test_final_channel_returns_content_delta`、`test_analysis_channel_reasoning`、`test_new_tool_call`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_final_channel_returns_content_delta (L36-L59)
```python
    @pytest.mark.parametrize(
        "delta_text,expected_content",
        [
            ("Hello, world!", "Hello, world!"),
            ("", ""),
        ],
    )
    def test_final_channel_returns_content_delta(self, delta_text, expected_content):
        """Test that final channel returns a DeltaMessage with content."""
        parser = MockStreamableParser()

        # Updated to use TokenState list
        token_states = [TokenState(channel="final", recipient=None, text=delta_text)]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient=None,
            include_reasoning=False,
        )

        assert delta_message is not None
        assert delta_message.content == expected_content
        assert tools_streamed is False
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_final_channel_returns_content_delta`. It uses parameterization over `delta_text`, `expected_content`. Key inputs are `delta_text`, `expected_content`. The main assertion is `delta_message is not None` and `delta_message.content == expected_content`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_final_channel_returns_content_delta`。 它通过参数化组合 `delta_text`、`expected_content`。 关键输入包括 `delta_text`、`expected_content`。 核心断言是 `delta_message is not None` and `delta_message.content == expected_content`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_analysis_channel_reasoning (L61-L86)
```python
    @pytest.mark.parametrize(
        "include_reasoning,expected_has_message",
        [
            (True, True),
            (False, False),
        ],
    )
    def test_analysis_channel_reasoning(self, include_reasoning, expected_has_message):
        """Test analysis channel respects include_reasoning flag."""
        parser = MockStreamableParser()
        text = "Let me think..."
        token_states = [TokenState(channel="analysis", recipient=None, text=text)]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient=None,
            include_reasoning=include_reasoning,
        )

        if expected_has_message:
            assert delta_message is not None
            assert delta_message.reasoning == text
        else:
            assert delta_message is None
        assert tools_streamed is False
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_analysis_channel_reasoning`. It uses parameterization over `include_reasoning`, `expected_has_message`. Key inputs are `include_reasoning`, `expected_has_message`. The main assertion is `tools_streamed is False` and `delta_message is not None`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_analysis_channel_reasoning`。 它通过参数化组合 `include_reasoning`、`expected_has_message`。 关键输入包括 `include_reasoning`、`expected_has_message`。 核心断言是 `tools_streamed is False` and `delta_message is not None`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_new_tool_call (L88-L114)
```python
    @pytest.mark.parametrize("channel", ["commentary", "analysis"])
    @patch("vllm.entrypoints.openai.chat_completion.stream_harmony.make_tool_call_id")
    def test_new_tool_call(self, mock_make_tool_call_id, channel):
        """Test new tool call creation when recipient changes."""
        mock_make_tool_call_id.return_value = "call_test123"
        parser = MockStreamableParser()

        token_states = [
            TokenState(channel=channel, recipient="functions.get_weather", text="")
        ]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient=None,
            include_reasoning=False,
        )

        assert delta_message is not None
        assert len(delta_message.tool_calls) == 1
        tool_call = delta_message.tool_calls[0]
        assert tool_call.id == "call_test123"
        assert tool_call.type == "function"
        assert tool_call.function.name == "get_weather"
        assert tool_call.function.arguments == ""
        assert tool_call.index == 0
        assert tools_streamed is True
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_new_tool_call`. It uses parameterization over `channel`. Key inputs are `mock_make_tool_call_id`, `channel`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `delta_message is not None` and `len(delta_message.tool_calls) == 1`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_new_tool_call`。 它通过参数化组合 `channel`。 关键输入包括 `mock_make_tool_call_id`、`channel`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `delta_message is not None` and `len(delta_message.tool_calls) == 1`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_tool_call_argument_streaming (L116-L140)
```python
    @pytest.mark.parametrize("channel", ["commentary", "analysis"])
    def test_tool_call_argument_streaming(self, channel):
        """Test streaming tool call arguments (same recipient)."""
        parser = MockStreamableParser()
        args_text = '{"location": "Paris"}'

        token_states = [
            TokenState(
                channel=channel, recipient="functions.get_weather", text=args_text
            )
        ]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient="functions.get_weather",
            include_reasoning=False,
        )

        assert delta_message is not None
        tool_call = delta_message.tool_calls[0]
        assert tool_call.id is None
        assert tool_call.function.arguments == args_text
        assert tool_call.index == 0
        assert tools_streamed is True
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_tool_call_argument_streaming`. It uses parameterization over `channel`. Key inputs are `channel`. The main assertion is `delta_message is not None` and `tool_call.id is None`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_tool_call_argument_streaming`。 它通过参数化组合 `channel`。 关键输入包括 `channel`。 核心断言是 `delta_message is not None` and `tool_call.id is None`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_tool_call_empty_arguments_returns_none (L142-L159)
```python
    @pytest.mark.parametrize("channel", ["commentary", "analysis"])
    def test_tool_call_empty_arguments_returns_none(self, channel):
        """Test empty delta_text with same recipient returns None."""
        parser = MockStreamableParser()

        token_states = [
            TokenState(channel=channel, recipient="functions.get_weather", text="")
        ]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient="functions.get_weather",
            include_reasoning=False,
        )

        assert delta_message is None
        assert tools_streamed is False
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_tool_call_empty_arguments_returns_none`. It uses parameterization over `channel`. Key inputs are `channel`. The main assertion is `delta_message is None` and `tools_streamed is False`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_tool_call_empty_arguments_returns_none`。 它通过参数化组合 `channel`。 关键输入包括 `channel`。 核心断言是 `delta_message is None` and `tools_streamed is False`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_tool_call_index_from_previous_messages (L161-L181)
```python
    def test_tool_call_index_from_previous_messages(self):
        """Test tool call index accounts for previous function messages."""
        messages = [
            MockMessage(channel="analysis", recipient=None),  # Not counted
            MockMessage(channel="commentary", recipient="functions.tool1"),  # Counted
            MockMessage(channel="final", recipient=None),  # Not counted
        ]
        parser = MockStreamableParser(messages=messages)

        token_states = [
            TokenState(channel="commentary", recipient="functions.tool2", text="args")
        ]

        delta_message, _ = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient="functions.tool2",
            include_reasoning=False,
        )

        assert delta_message.tool_calls[0].index == 1
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_tool_call_index_from_previous_messages`. The main assertion is `delta_message.tool_calls[0].index == 1`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_tool_call_index_from_previous_messages`。 核心断言是 `delta_message.tool_calls[0].index == 1`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_returns_preambles_as_content (L183-L200)
```python
    def test_returns_preambles_as_content(self):
        """Test that commentary with no recipient (preamble) is user content."""
        parser = MockStreamableParser()
        delta_text = "some text"

        token_states = [
            TokenState(channel="commentary", recipient=None, text=delta_text)
        ]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient=None,
            include_reasoning=True,
        )

        assert delta_message.content == delta_text
        assert tools_streamed is False
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_returns_preambles_as_content`. The main assertion is `delta_message.content == delta_text` and `tools_streamed is False`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_returns_preambles_as_content`。 核心断言是 `delta_message.content == delta_text` and `tools_streamed is False`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_new_tool_call_without_functions_prefix (L202-L227)
```python
    @pytest.mark.parametrize("channel", ["commentary", "analysis"])
    @patch("vllm.entrypoints.openai.chat_completion.stream_harmony.make_tool_call_id")
    def test_new_tool_call_without_functions_prefix(
        self, mock_make_tool_call_id, channel
    ):
        mock_make_tool_call_id.return_value = "call_bare123"
        parser = MockStreamableParser()

        token_states = [TokenState(channel=channel, recipient="get_weather", text="")]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient=None,
            include_reasoning=False,
        )

        assert delta_message is not None
        assert len(delta_message.tool_calls) == 1
        tool_call = delta_message.tool_calls[0]
        assert tool_call.id == "call_bare123"
        assert tool_call.type == "function"
        assert tool_call.function.name == "get_weather"
        assert tool_call.function.arguments == ""
        assert tool_call.index == 0
        assert tools_streamed is True
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_new_tool_call_without_functions_prefix`. It uses parameterization over `channel`. Key inputs are `mock_make_tool_call_id`, `channel`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `delta_message is not None` and `len(delta_message.tool_calls) == 1`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_new_tool_call_without_functions_prefix`。 它通过参数化组合 `channel`。 关键输入包括 `mock_make_tool_call_id`、`channel`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `delta_message is not None` and `len(delta_message.tool_calls) == 1`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_tool_call_argument_streaming_without_functions_prefix (L229-L250)
```python
    @pytest.mark.parametrize("channel", ["commentary", "analysis"])
    def test_tool_call_argument_streaming_without_functions_prefix(self, channel):
        parser = MockStreamableParser()
        args_text = '{"location": "Paris"}'

        token_states = [
            TokenState(channel=channel, recipient="get_weather", text=args_text)
        ]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient="get_weather",
            include_reasoning=False,
        )

        assert delta_message is not None
        tool_call = delta_message.tool_calls[0]
        assert tool_call.id is None
        assert tool_call.function.arguments == args_text
        assert tool_call.index == 0
        assert tools_streamed is True
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_tool_call_argument_streaming_without_functions_prefix`. It uses parameterization over `channel`. Key inputs are `channel`. The main assertion is `delta_message is not None` and `tool_call.id is None`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_tool_call_argument_streaming_without_functions_prefix`。 它通过参数化组合 `channel`。 关键输入包括 `channel`。 核心断言是 `delta_message is not None` and `tool_call.id is None`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_tool_call_index_from_previous_messages_without_functions_prefix (L252-L269)
```python
    def test_tool_call_index_from_previous_messages_without_functions_prefix(self):
        messages = [
            MockMessage(channel="commentary", recipient="tool1"),
        ]
        parser = MockStreamableParser(messages=messages)

        token_states = [
            TokenState(channel="commentary", recipient="tool2", text="args")
        ]

        delta_message, _ = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient="tool2",
            include_reasoning=False,
        )

        assert delta_message.tool_calls[0].index == 1
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_tool_call_index_from_previous_messages_without_functions_prefix`. The main assertion is `delta_message.tool_calls[0].index == 1`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_tool_call_index_from_previous_messages_without_functions_prefix`。 核心断言是 `delta_message.tool_calls[0].index == 1`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_new_tool_call_dotted_function_name (L271-L294)
```python
    @pytest.mark.parametrize("channel", ["commentary", "analysis"])
    @patch("vllm.entrypoints.openai.chat_completion.stream_harmony.make_tool_call_id")
    def test_new_tool_call_dotted_function_name(self, mock_make_tool_call_id, channel):
        mock_make_tool_call_id.return_value = "call_dotted123"
        parser = MockStreamableParser()

        token_states = [TokenState(channel=channel, recipient="math.sum", text="")]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient=None,
            include_reasoning=False,
        )

        assert delta_message is not None
        assert len(delta_message.tool_calls) == 1
        tool_call = delta_message.tool_calls[0]
        assert tool_call.id == "call_dotted123"
        assert tool_call.type == "function"
        assert tool_call.function.name == "math.sum"
        assert tool_call.function.arguments == ""
        assert tool_call.index == 0
        assert tools_streamed is True
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_new_tool_call_dotted_function_name`. It uses parameterization over `channel`. Key inputs are `mock_make_tool_call_id`, `channel`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `delta_message is not None` and `len(delta_message.tool_calls) == 1`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_new_tool_call_dotted_function_name`。 它通过参数化组合 `channel`。 关键输入包括 `mock_make_tool_call_id`、`channel`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `delta_message is not None` and `len(delta_message.tool_calls) == 1`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_returns_none_for_invalid_inputs (L296-L321)
```python
    @pytest.mark.parametrize(
        "channel,recipient",
        [
            (None, None),
            ("unknown_channel", None),
            ("commentary", "browser.search"),
            ("commentary", "assistant"),
        ],
    )
    def test_returns_none_for_invalid_inputs(self, channel, recipient):
        """Test that invalid channel/recipient combinations return None."""
        parser = MockStreamableParser()

        token_states = [
            TokenState(channel=channel, recipient=recipient, text="some text")
        ]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient=None,
            include_reasoning=True,
        )

        assert delta_message is None
        assert tools_streamed is False
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_returns_none_for_invalid_inputs`. It uses parameterization over `channel`, `recipient`. Key inputs are `channel`, `recipient`. The main assertion is `delta_message is None` and `tools_streamed is False`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_returns_none_for_invalid_inputs`。 它通过参数化组合 `channel`、`recipient`。 关键输入包括 `channel`、`recipient`。 核心断言是 `delta_message is None` and `tools_streamed is False`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_consecutive_token_grouping (L323-L345)
```python
    def test_consecutive_token_grouping(self):
        """
        Test that consecutive tokens with the same channel/recipient
        are merged into a single processing group.
        """
        parser = MockStreamableParser()
        token_states = [
            TokenState("final", None, "H"),
            TokenState("final", None, "el"),
            TokenState("final", None, "lo"),
            TokenState("final", None, ","),
            TokenState("final", None, " World"),
        ]

        delta_message, _ = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient=None,
            include_reasoning=False,
        )

        assert delta_message is not None
        assert delta_message.content == "Hello, World"
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_consecutive_token_grouping`. The main assertion is `delta_message is not None` and `delta_message.content == 'Hello, World'`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_consecutive_token_grouping`。 核心断言是 `delta_message is not None` and `delta_message.content == 'Hello, World'`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_complex_batch_permutation (L347-L394)
```python
    @patch("vllm.entrypoints.openai.chat_completion.stream_harmony.make_tool_call_id")
    def test_complex_batch_permutation(self, mock_make_id):
        """
        Test a complex permutation: Reasoning -> Tool Call -> Content.
        This verifies that multiple distinct actions in one batch
        are all captured in the single DeltaMessage.
        """
        mock_make_id.return_value = "call_batch_test"
        parser = MockStreamableParser()

        token_states = [
            # 1. Reasoning
            TokenState("analysis", None, "Reasoning about query..."),
            # 2. Tool Calling
            TokenState("commentary", "functions.search", '{"query":'),
            TokenState("commentary", "functions.search", ' "vllm"}'),
            # 3. Final Content
            TokenState("final", None, "."),
# ... 22 lines omitted for brevity ...
        assert header.id == "call_batch_test"
        assert header.index == 0

        assert payload.index == 0
        assert payload.function.arguments == '{"query": "vllm"}'

        assert delta_message.content == "."
        assert tools_streamed is True
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_complex_batch_permutation`. Key inputs are `mock_make_id`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `delta_message is not None` and `delta_message.reasoning == 'Reasoning about query...'`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_complex_batch_permutation`。 关键输入包括 `mock_make_id`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `delta_message is not None` and `delta_message.reasoning == 'Reasoning about query...'`。

### Test method / 测试方法: TestExtractHarmonyStreamingDelta.test_tool_call_index_consistency_with_ongoing_call (L396-L445)
```python
    @patch("vllm.entrypoints.openai.chat_completion.stream_harmony.make_tool_call_id")
    def test_tool_call_index_consistency_with_ongoing_call(self, mock_make_id):
        """
        Test that an ongoing tool call continuation and subsequent new calls
        maintain correct indexing when interleaved with content.
        """
        mock_make_id.side_effect = ["id_b", "id_c"]

        messages = [
            MockMessage(channel="commentary", recipient="functions.previous_tool")
        ]
        parser = MockStreamableParser(messages=messages)

        token_states = [
            TokenState("commentary", "functions.tool_a", '{"key_a": "val_a"}'),
            TokenState("final", None, "Thinking..."),
            TokenState("commentary", "functions.tool_b", '{"key_b": "val_b"}'),
            TokenState("final", None, " Thinking again..."),
# ... 24 lines omitted for brevity ...
        tool_c_start = next(t for t in delta_message.tool_calls if t.id == "id_c")
        assert tool_c_start.index == 3
        tool_c_args = next(
            t for t in delta_message.tool_calls if t.index == 3 and t.id is None
        )
        assert tool_c_args.function.arguments == '{"key_c": "val_c"}'

        assert delta_message.content == "Thinking... Thinking again..."
```
**EN:** This test validates `TestExtractHarmonyStreamingDelta.test_tool_call_index_consistency_with_ongoing_call`. Key inputs are `mock_make_id`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `delta_message is not None` and `len(tool_a_deltas) > 0`.
**CN:** 这个测试验证 `TestExtractHarmonyStreamingDelta.test_tool_call_index_consistency_with_ongoing_call`。 关键输入包括 `mock_make_id`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `delta_message is not None` and `len(tool_a_deltas) > 0`。

### Class / 类: TestToolCallsOnNonStandardChannels (L448-L534)
```python
class TestToolCallsOnNonStandardChannels:
    """Tool calls are detected by recipient, not channel.

    Models sometimes emit tool calls on unexpected channels (e.g. ``comment``
    instead of ``commentary``).  These tests verify that the streaming delta
    extraction is channel-agnostic for tool call detection.
    """

    @patch("vllm.entrypoints.openai.chat_completion.stream_harmony.make_tool_call_id")
    def test_prefixed_tool_call_on_comment_channel(self, mock_make_tool_call_id):
        mock_make_tool_call_id.return_value = "call_comment_chan"
        parser = MockStreamableParser()

        token_states = [
# ... 65 lines omitted for brevity ...
        delta_message, _ = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient="functions.tool2",
            include_reasoning=False,
        )

        assert delta_message.tool_calls[0].index == 1
```
**EN:** This class groups related scenarios in `TestToolCallsOnNonStandardChannels`. It contains 4 test method(s) and 0 supporting method(s). Representative methods include `test_prefixed_tool_call_on_comment_channel`, `test_bare_tool_call_on_comment_channel`, `test_tool_call_arguments_on_comment_channel`.
**CN:** 该类将与 `TestToolCallsOnNonStandardChannels` 相关的场景组织在一起。 它包含 4 个测试方法和 0 个辅助方法。 代表性方法包括 `test_prefixed_tool_call_on_comment_channel`、`test_bare_tool_call_on_comment_channel`、`test_tool_call_arguments_on_comment_channel`。

### Test method / 测试方法: TestToolCallsOnNonStandardChannels.test_prefixed_tool_call_on_comment_channel (L456-L475)
```python
    @patch("vllm.entrypoints.openai.chat_completion.stream_harmony.make_tool_call_id")
    def test_prefixed_tool_call_on_comment_channel(self, mock_make_tool_call_id):
        mock_make_tool_call_id.return_value = "call_comment_chan"
        parser = MockStreamableParser()

        token_states = [
            TokenState(channel="comment", recipient="functions.get_weather", text="")
        ]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient=None,
            include_reasoning=False,
        )

        assert delta_message is not None
        assert len(delta_message.tool_calls) == 1
        assert delta_message.tool_calls[0].function.name == "get_weather"
        assert tools_streamed is True
```
**EN:** This test validates `TestToolCallsOnNonStandardChannels.test_prefixed_tool_call_on_comment_channel`. Key inputs are `mock_make_tool_call_id`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `delta_message is not None` and `len(delta_message.tool_calls) == 1`.
**CN:** 这个测试验证 `TestToolCallsOnNonStandardChannels.test_prefixed_tool_call_on_comment_channel`。 关键输入包括 `mock_make_tool_call_id`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `delta_message is not None` and `len(delta_message.tool_calls) == 1`。

### Test method / 测试方法: TestToolCallsOnNonStandardChannels.test_bare_tool_call_on_comment_channel (L477-L494)
```python
    @patch("vllm.entrypoints.openai.chat_completion.stream_harmony.make_tool_call_id")
    def test_bare_tool_call_on_comment_channel(self, mock_make_tool_call_id):
        mock_make_tool_call_id.return_value = "call_bare_comment"
        parser = MockStreamableParser()

        token_states = [TokenState(channel="comment", recipient="get_weather", text="")]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient=None,
            include_reasoning=False,
        )

        assert delta_message is not None
        assert len(delta_message.tool_calls) == 1
        assert delta_message.tool_calls[0].function.name == "get_weather"
        assert tools_streamed is True
```
**EN:** This test validates `TestToolCallsOnNonStandardChannels.test_bare_tool_call_on_comment_channel`. Key inputs are `mock_make_tool_call_id`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `delta_message is not None` and `len(delta_message.tool_calls) == 1`.
**CN:** 这个测试验证 `TestToolCallsOnNonStandardChannels.test_bare_tool_call_on_comment_channel`。 关键输入包括 `mock_make_tool_call_id`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `delta_message is not None` and `len(delta_message.tool_calls) == 1`。

### Test method / 测试方法: TestToolCallsOnNonStandardChannels.test_tool_call_arguments_on_comment_channel (L496-L515)
```python
    def test_tool_call_arguments_on_comment_channel(self):
        parser = MockStreamableParser()
        args_text = '{"location": "Paris"}'

        token_states = [
            TokenState(
                channel="comment", recipient="functions.get_weather", text=args_text
            )
        ]

        delta_message, tools_streamed = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient="functions.get_weather",
            include_reasoning=False,
        )

        assert delta_message is not None
        assert delta_message.tool_calls[0].function.arguments == args_text
        assert tools_streamed is True
```
**EN:** This test validates `TestToolCallsOnNonStandardChannels.test_tool_call_arguments_on_comment_channel`. The main assertion is `delta_message is not None` and `delta_message.tool_calls[0].function.arguments == args_text`.
**CN:** 这个测试验证 `TestToolCallsOnNonStandardChannels.test_tool_call_arguments_on_comment_channel`。 核心断言是 `delta_message is not None` and `delta_message.tool_calls[0].function.arguments == args_text`。

### Test method / 测试方法: TestToolCallsOnNonStandardChannels.test_base_index_counts_tool_calls_on_comment_channel (L517-L534)
```python
    def test_base_index_counts_tool_calls_on_comment_channel(self):
        messages = [
            MockMessage(channel="comment", recipient="functions.tool1"),
        ]
        parser = MockStreamableParser(messages=messages)

        token_states = [
            TokenState(channel="commentary", recipient="functions.tool2", text="args")
        ]

        delta_message, _ = extract_harmony_streaming_delta(
            harmony_parser=parser,
            token_states=token_states,
            prev_recipient="functions.tool2",
            include_reasoning=False,
        )

        assert delta_message.tool_calls[0].index == 1
```
**EN:** This test validates `TestToolCallsOnNonStandardChannels.test_base_index_counts_tool_calls_on_comment_channel`. The main assertion is `delta_message.tool_calls[0].index == 1`.
**CN:** 这个测试验证 `TestToolCallsOnNonStandardChannels.test_base_index_counts_tool_calls_on_comment_channel`。 核心断言是 `delta_message.tool_calls[0].index == 1`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `dataclasses.dataclass`, `dataclasses.field`, `unittest.mock.patch`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.entrypoints.openai.chat_completion.stream_harmony.TokenState`, `vllm.entrypoints.openai.chat_completion.stream_harmony.extract_harmony_streaming_delta`
