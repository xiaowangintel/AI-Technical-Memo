# test_context.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/test_context.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers `test_context` scenarios. The file defines 22 test(s), 1 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖`test_context` 场景。它定义了 22 个测试、1 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L15)
```python
from unittest.mock import MagicMock, patch

import pytest
from openai_harmony import Author, Message, Role, StreamState, TextContent

from vllm.entrypoints.openai.responses.context import (
    HarmonyContext,
    SimpleContext,
    StreamingHarmonyContext,
    TurnMetrics,
)
from vllm.outputs import CompletionOutput, RequestOutput
```
**EN:** Imports standard-library modules such as `unittest.mock.MagicMock`, `unittest.mock.patch`, third-party packages like `openai_harmony.Author`, `openai_harmony.Message`, `openai_harmony.Role`, project helpers such as `vllm.entrypoints.openai.responses.context.HarmonyContext`, `vllm.entrypoints.openai.responses.context.SimpleContext`, `vllm.entrypoints.openai.responses.context.StreamingHarmonyContext`.
**CN:** 导入标准库模块（如 `unittest.mock.MagicMock`、`unittest.mock.patch`）、第三方包（如 `openai_harmony.Author`、`openai_harmony.Message`、`openai_harmony.Role`）、项目内辅助模块（如 `vllm.entrypoints.openai.responses.context.HarmonyContext`、`vllm.entrypoints.openai.responses.context.SimpleContext`、`vllm.entrypoints.openai.responses.context.StreamingHarmonyContext`）。

### Helper / 辅助函数: create_mock_request_output (L18-L47)
```python
def create_mock_request_output(
    prompt_token_ids=None,
    output_token_ids=None,
    num_cached_tokens=0,
    finished=True,
):
    """Helper function to create a mock RequestOutput object for testing."""
    outputs = []
    token_ids = output_token_ids if output_token_ids is not None else []
    outputs = [
        CompletionOutput(
            index=0,
            text="Test output",
            token_ids=token_ids,
            cumulative_logprob=0.0,
            logprobs=None,
            finish_reason=None,
            stop_reason=None,
        )
    ]

    return RequestOutput(
        request_id="test-id",
        prompt="Test prompt",
        prompt_token_ids=prompt_token_ids,
        prompt_logprobs=None,
        outputs=outputs,
        finished=finished,
        num_cached_tokens=num_cached_tokens,
    )
```
**EN:** This helper encapsulates reusable logic in `create_mock_request_output`. Key inputs are `prompt_token_ids`, `output_token_ids`, `num_cached_tokens`, `finished`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_mock_request_output` 中。 关键输入包括 `prompt_token_ids`、`output_token_ids`、`num_cached_tokens`、`finished`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: generate_mock_outputs (L50-L68)
```python
async def generate_mock_outputs(
    num_turns, prompt_token_counts, output_token_counts, cached_token_counts=None
):
    """Generate a sequence of mock RequestOutput objects to simulate multiple
    turns."""
    if cached_token_counts is None:
        cached_token_counts = [0] * num_turns

    for i in range(num_turns):
        # Create mock prompt token IDs and output token IDs
        prompt_token_ids = list(range(1, prompt_token_counts[i] + 1))
        output_token_ids = list(range(1, output_token_counts[i] + 1))

        # Create and yield the RequestOutput
        yield create_mock_request_output(
            prompt_token_ids=prompt_token_ids,
            output_token_ids=output_token_ids,
            num_cached_tokens=cached_token_counts[i],
        )
```
**EN:** This async helper encapsulates reusable logic in `generate_mock_outputs`. Key inputs are `num_turns`, `prompt_token_counts`, `output_token_counts`, `cached_token_counts`.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `generate_mock_outputs` 中。 关键输入包括 `num_turns`、`prompt_token_counts`、`output_token_counts`、`cached_token_counts`。

### Fixture / 夹具: mock_parser (L71-L83)
```python
@pytest.fixture
def mock_parser():
    """Set up a mock parser for tests."""
    with patch(
        "vllm.entrypoints.openai.responses.context.get_streamable_parser_for_assistant"
    ) as mock_parser_factory:
        # Create a mock parser object
        parser = MagicMock()
        parser.messages = []
        parser.current_channel = None
        parser.state = StreamState.EXPECT_START
        mock_parser_factory.return_value = parser
        yield parser
```
**EN:** This fixture prepares `mock_parser` for dependent tests. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `mock_parser`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_single_turn_token_counting (L86-L114)
```python
def test_single_turn_token_counting():
    """Test token counting behavior for a single turn."""
    # Create a context
    context = HarmonyContext(messages=[], available_tools=[])

    # Create a mock RequestOutput with specific token counts
    mock_output = create_mock_request_output(
        prompt_token_ids=[1, 2, 3, 4, 5],  # 5 prompt tokens
        output_token_ids=[6, 7, 8],  # 3 output tokens
        num_cached_tokens=2,  # 2 cached tokens
    )

    # Append the output to the context
    context.append_output(mock_output)

    # Verify the token counts
    assert context.num_prompt_tokens == 5
    assert context.num_output_tokens == 3
    assert context.num_cached_tokens == 2
    assert context.num_tool_output_tokens == 0  # No tool tokens in first turn

    # Verify internal state tracking
    assert not context.is_first_turn
    assert len(context.all_turn_metrics) == 1
    previous_turn = context.all_turn_metrics[0]
    assert previous_turn.input_tokens == 5
    assert previous_turn.output_tokens == 3
    assert previous_turn.cached_input_tokens == 2
    assert previous_turn.tool_output_tokens == 0
```
**EN:** This test validates `test_single_turn_token_counting`. The main assertion is `context.num_prompt_tokens == 5` and `context.num_output_tokens == 3`.
**CN:** 这个测试验证 `test_single_turn_token_counting`。 核心断言是 `context.num_prompt_tokens == 5` and `context.num_output_tokens == 3`。

### Test / 测试: test_multi_turn_token_counting (L117-L175)
```python
@pytest.mark.asyncio
async def test_multi_turn_token_counting():
    """Test token counting behavior across multiple turns with tool output."""
    # Create a context
    context = HarmonyContext(messages=[], available_tools=["browser"])

    # Simulate a conversation with 3 turns
    # Turn 1: prefill 5, decode 3, tool 7
    # Turn 2: prefill 15, cached 5, decode 4, tool 1
    # Turn 3: prefill 20, cached 15, decode 5
    prompt_token_counts = [5, 15, 20]
    output_token_counts = [3, 4, 5]
    cached_token_counts = [0, 5, 15]
    mock_generator = generate_mock_outputs(
        3, prompt_token_counts, output_token_counts, cached_token_counts
    )

    # First turn - initial prompt and response
# ... 33 lines omitted for brevity ...
    # Validate all turn metrics
    assert len(context.all_turn_metrics) == 3
    for i, turn in enumerate(context.all_turn_metrics):
        assert turn.input_tokens == prompt_token_counts[i]
        assert turn.output_tokens == output_token_counts[i]
        assert turn.cached_input_tokens == cached_token_counts[i]
    assert context.all_turn_metrics[1].tool_output_tokens == 7
    assert context.all_turn_metrics[2].tool_output_tokens == 1
```
**EN:** This async test validates `test_multi_turn_token_counting`. Relevant pytest markers include `asyncio`. The main assertion is `context.num_prompt_tokens == 5` and `context.num_output_tokens == 3`.
**CN:** 这个异步测试验证 `test_multi_turn_token_counting`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `context.num_prompt_tokens == 5` and `context.num_output_tokens == 3`。

### Test / 测试: test_empty_output_tokens (L178-L195)
```python
def test_empty_output_tokens():
    """Test behavior when RequestOutput has empty output tokens."""
    context = HarmonyContext(messages=[], available_tools=[])

    # Create a RequestOutput with empty output tokens
    mock_output = create_mock_request_output(
        prompt_token_ids=[1, 2, 3],  # 3 prompt tokens
        output_token_ids=[],  # Empty output tokens list
        num_cached_tokens=1,
    )

    context.append_output(mock_output)

    # Should handle empty outputs gracefully
    assert context.num_prompt_tokens == 3
    assert context.num_output_tokens == 0  # No output tokens
    assert context.num_cached_tokens == 1
    assert context.num_tool_output_tokens == 0
```
**EN:** This test validates `test_empty_output_tokens`. The main assertion is `context.num_prompt_tokens == 3` and `context.num_output_tokens == 0`.
**CN:** 这个测试验证 `test_empty_output_tokens`。 核心断言是 `context.num_prompt_tokens == 3` and `context.num_output_tokens == 0`。

### Test / 测试: test_missing_prompt_token_ids (L198-L216)
```python
def test_missing_prompt_token_ids():
    """Test behavior when RequestOutput has None prompt_token_ids."""
    context = HarmonyContext(messages=[], available_tools=[])

    mock_output = create_mock_request_output(
        prompt_token_ids=None,  # No prompt token IDs
        output_token_ids=[1, 2],  # 2 output tokens
        num_cached_tokens=0,
    )

    # Logger.error will be called, but we don't need to check for warnings
    # here Just ensure it doesn't raise an exception
    context.append_output(mock_output)

    # Should handle missing prompt tokens gracefully
    assert context.num_prompt_tokens == 0
    assert context.num_output_tokens == 2
    assert context.num_cached_tokens == 0
    assert context.num_tool_output_tokens == 0
```
**EN:** This test validates `test_missing_prompt_token_ids`. The main assertion is `context.num_prompt_tokens == 0` and `context.num_output_tokens == 2`.
**CN:** 这个测试验证 `test_missing_prompt_token_ids`。 核心断言是 `context.num_prompt_tokens == 0` and `context.num_output_tokens == 2`。

### Test / 测试: test_reasoning_tokens_counting (L219-L236)
```python
def test_reasoning_tokens_counting(mock_parser):
    """Test that reasoning tokens are counted correctly."""
    context = HarmonyContext(messages=[], available_tools=[])

    # Mock parser to simulate reasoning channel
    mock_parser.current_channel = "analysis"  # Reasoning channel

    mock_output = create_mock_request_output(
        prompt_token_ids=[1, 2, 3],
        output_token_ids=[4, 5, 6, 7],  # 4 tokens, all in reasoning
        num_cached_tokens=0,
    )

    context.append_output(mock_output)

    # All output tokens should be counted as reasoning
    assert context.num_reasoning_tokens == 4
    assert context.num_output_tokens == 4
```
**EN:** This test validates `test_reasoning_tokens_counting`. Key inputs are `mock_parser`. The main assertion is `context.num_reasoning_tokens == 4` and `context.num_output_tokens == 4`.
**CN:** 这个测试验证 `test_reasoning_tokens_counting`。 关键输入包括 `mock_parser`。 核心断言是 `context.num_reasoning_tokens == 4` and `context.num_output_tokens == 4`。

### Test / 测试: test_preamble_tokens_not_counted_as_reasoning (L239-L255)
```python
def test_preamble_tokens_not_counted_as_reasoning(mock_parser):
    """Preambles (commentary with no recipient) are visible user text,
    not hidden reasoning. They must NOT inflate num_reasoning_tokens."""
    context = HarmonyContext(messages=[], available_tools=[])

    mock_parser.current_channel = "commentary"
    mock_parser.current_recipient = None  # preamble

    mock_output = create_mock_request_output(
        prompt_token_ids=[1, 2, 3],
        output_token_ids=[4, 5, 6],
        num_cached_tokens=0,
    )
    context.append_output(mock_output)

    assert context.num_reasoning_tokens == 0
    assert context.num_output_tokens == 3
```
**EN:** This test validates `test_preamble_tokens_not_counted_as_reasoning`. Key inputs are `mock_parser`. The main assertion is `context.num_reasoning_tokens == 0` and `context.num_output_tokens == 3`.
**CN:** 这个测试验证 `test_preamble_tokens_not_counted_as_reasoning`。 关键输入包括 `mock_parser`。 核心断言是 `context.num_reasoning_tokens == 0` and `context.num_output_tokens == 3`。

### Test / 测试: test_commentary_with_recipient_counted_as_reasoning (L258-L274)
```python
def test_commentary_with_recipient_counted_as_reasoning(mock_parser):
    """Commentary directed at a tool (recipient != None) is hidden from
    the user, so it should still count as reasoning tokens."""
    context = HarmonyContext(messages=[], available_tools=[])

    mock_parser.current_channel = "commentary"
    mock_parser.current_recipient = "python"

    mock_output = create_mock_request_output(
        prompt_token_ids=[1, 2, 3],
        output_token_ids=[4, 5, 6],
        num_cached_tokens=0,
    )
    context.append_output(mock_output)

    assert context.num_reasoning_tokens == 3
    assert context.num_output_tokens == 3
```
**EN:** This test validates `test_commentary_with_recipient_counted_as_reasoning`. Key inputs are `mock_parser`. The main assertion is `context.num_reasoning_tokens == 3` and `context.num_output_tokens == 3`.
**CN:** 这个测试验证 `test_commentary_with_recipient_counted_as_reasoning`。 关键输入包括 `mock_parser`。 核心断言是 `context.num_reasoning_tokens == 3` and `context.num_output_tokens == 3`。

### Test / 测试: test_zero_tokens_edge_case (L277-L296)
```python
def test_zero_tokens_edge_case():
    """Test behavior with all zero token counts."""
    context = HarmonyContext(messages=[], available_tools=[])

    # Create a request with empty lists (not None) for both prompt and
    # output tokens
    mock_output = create_mock_request_output(
        prompt_token_ids=[],  # Empty prompt tokens
        output_token_ids=[],  # Empty output tokens
        num_cached_tokens=0,
    )

    context.append_output(mock_output)

    # All counts should be zero
    assert context.num_prompt_tokens == 0
    assert context.num_output_tokens == 0
    assert context.num_cached_tokens == 0
    assert context.num_tool_output_tokens == 0
    assert context.num_reasoning_tokens == 0
```
**EN:** This test validates `test_zero_tokens_edge_case`. The main assertion is `context.num_prompt_tokens == 0` and `context.num_output_tokens == 0`.
**CN:** 这个测试验证 `test_zero_tokens_edge_case`。 核心断言是 `context.num_prompt_tokens == 0` and `context.num_output_tokens == 0`。

### Test / 测试: test_single_turn_no_tool_output (L299-L318)
```python
@pytest.mark.asyncio
async def test_single_turn_no_tool_output():
    """Test that first turn never generates tool output tokens."""
    context = HarmonyContext(
        messages=[],
        available_tools=["browser"],  # Tools available
    )

    # Even with large prompt in first turn, no tool tokens should be counted
    mock_output = create_mock_request_output(
        prompt_token_ids=list(range(100)),  # 100 tokens
        output_token_ids=[1, 2, 3],
        num_cached_tokens=0,
    )

    context.append_output(mock_output)

    # First turn should never have tool output tokens
    assert context.num_tool_output_tokens == 0
    assert context.is_first_turn is False  # Should be updated after first turn
```
**EN:** This async test validates `test_single_turn_no_tool_output`. Relevant pytest markers include `asyncio`. The main assertion is `context.num_tool_output_tokens == 0` and `context.is_first_turn is False`.
**CN:** 这个异步测试验证 `test_single_turn_no_tool_output`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `context.num_tool_output_tokens == 0` and `context.is_first_turn is False`。

### Test / 测试: test_negative_tool_tokens_edge_case (L321-L359)
```python
@pytest.mark.asyncio
async def test_negative_tool_tokens_edge_case():
    """Test edge case where calculation could result in negative tool
    tokens. We should log an error and clamp the value to 0."""
    # Use patch to check if logger.error was called
    with patch("vllm.entrypoints.openai.responses.context.logger.error") as mock_log:
        context = HarmonyContext(messages=[], available_tools=["browser"])

        # First turn
        mock_output1 = create_mock_request_output(
            prompt_token_ids=list(range(10)),  # 10 tokens
            output_token_ids=[1, 2, 3, 4, 5],  # 5 tokens
        )
        context.append_output(mock_output1)

        # Second turn with fewer new tokens than previous output
        # This could happen in edge cases with aggressive caching
        mock_output2 = create_mock_request_output(
# ... 13 lines omitted for brevity ...

        # Extract the actual log message and arguments from the call
        args, _ = mock_log.call_args
        log_message = args[0]

        # Check for key parts of the message
        assert "Negative tool output tokens calculated" in log_message
        assert "-3" in str(args)  # Check that -3 is in the arguments
```
**EN:** This async test validates `test_negative_tool_tokens_edge_case`. Relevant pytest markers include `asyncio`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `context.num_tool_output_tokens == 0` and `context.num_prompt_tokens == 10 + 12`.
**CN:** 这个异步测试验证 `test_negative_tool_tokens_edge_case`。 相关的 pytest 标记包括 `asyncio`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `context.num_tool_output_tokens == 0` and `context.num_prompt_tokens == 10 + 12`。

### Test / 测试: test_streaming_multi_turn_token_counting (L362-L519)
```python
@pytest.mark.asyncio
async def test_streaming_multi_turn_token_counting(mock_parser):
    """Test token counting for streaming multi-turn conversations.

    This test focuses on how StreamingHarmonyContext counts tokens in a
    multi-turn conversation with streaming (token-by-token) outputs and
    message boundaries.
    """
    # Create a streaming context
    context = StreamingHarmonyContext(messages=[], available_tools=["browser"])

    num_prompt_tokens = [3, 8, 13]
    num_output_tokens = [3, 3, 2]
    num_cached_tokens = [0, 3, 8]

    # Simulate three turns of conversation:
    # Turn 1: stream tokens one by one, then finish the message
    # Turn 2: new prompt, stream more tokens with a reasoning segment
# ... 132 lines omitted for brevity ...
    # Validate all turn metrics
    assert len(context.all_turn_metrics) == 3
    for i, turn in enumerate(context.all_turn_metrics):
        assert turn.input_tokens == num_prompt_tokens[i]
        assert turn.output_tokens == num_output_tokens[i]
        assert turn.cached_input_tokens == num_cached_tokens[i]
    assert context.all_turn_metrics[1].tool_output_tokens == 2
    assert context.all_turn_metrics[2].tool_output_tokens == 2
```
**EN:** This async test validates `test_streaming_multi_turn_token_counting`. Relevant pytest markers include `asyncio`. Key inputs are `mock_parser`. The main assertion is `context.num_prompt_tokens == 3` and `context.num_output_tokens == 3`.
**CN:** 这个异步测试验证 `test_streaming_multi_turn_token_counting`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mock_parser`。 核心断言是 `context.num_prompt_tokens == 3` and `context.num_output_tokens == 3`。

### Test / 测试: test_streaming_message_synchronization (L522-L595)
```python
@pytest.mark.asyncio
async def test_streaming_message_synchronization(mock_parser):
    """Test message synchronization logic from lines 413-417 in context.py.

    This test verifies that when parser.messages contains more messages than
    the context's _messages (minus initial messages), the context properly
    extends its message list with the new parser messages.
    """

    # Create a streaming context with some initial messages
    initial_messages = [
        Message(
            author=Author(role=Role.USER, name="user"),
            content=[TextContent(text="Hello")],
            recipient=Role.ASSISTANT,
        )
    ]
    context = StreamingHarmonyContext(messages=initial_messages, available_tools=[])
# ... 48 lines omitted for brevity ...
    )

    context.append_output(mock_output2)

    # Verify the fourth message was added, num_init_messages is still 1
    assert len(context._messages) == 3
    assert context.num_init_messages == 1
    assert context._messages[2].content[0].text == "Response 4"
```
**EN:** This async test validates `test_streaming_message_synchronization`. Relevant pytest markers include `asyncio`. Key inputs are `mock_parser`. The main assertion is `len(context._messages) == 1` and `context.num_init_messages == 1`.
**CN:** 这个异步测试验证 `test_streaming_message_synchronization`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mock_parser`。 核心断言是 `len(context._messages) == 1` and `context.num_init_messages == 1`。

### Test / 测试: test_turn_metrics_copy_and_reset (L598-L638)
```python
def test_turn_metrics_copy_and_reset():
    """Test TurnMetrics copy and reset methods work correctly."""
    # Create a TurnMetrics with specific values
    original_metrics = TurnMetrics(
        input_tokens=10,
        output_tokens=20,
        cached_input_tokens=5,
        tool_output_tokens=3,
    )

    # Test copy functionality
    copied_metrics = original_metrics.copy()

    # Verify copy has same values
    assert copied_metrics.input_tokens == 10
    assert copied_metrics.output_tokens == 20
    assert copied_metrics.cached_input_tokens == 5
    assert copied_metrics.tool_output_tokens == 3
# ... 15 lines omitted for brevity ...
    assert original_metrics.cached_input_tokens == 0
    assert original_metrics.tool_output_tokens == 0

    # Verify copied metrics are unaffected by reset
    assert copied_metrics.input_tokens == 999
    assert copied_metrics.output_tokens == 20
    assert copied_metrics.cached_input_tokens == 5
    assert copied_metrics.tool_output_tokens == 3
```
**EN:** This test validates `test_turn_metrics_copy_and_reset`. The main assertion is `copied_metrics.input_tokens == 10` and `copied_metrics.output_tokens == 20`.
**CN:** 这个测试验证 `test_turn_metrics_copy_and_reset`。 核心断言是 `copied_metrics.input_tokens == 10` and `copied_metrics.output_tokens == 20`。

### Helper / 辅助函数: create_simple_context_output (L644-L675)
```python
def create_simple_context_output(
    text="",
    token_ids=None,
    prompt="Test prompt",
    prompt_token_ids=None,
    num_cached_tokens=0,
    logprobs=None,
    finished=True,
):
    """Helper to create a RequestOutput with customizable text for
    SimpleContext tests."""
    if token_ids is None:
        token_ids = []
    return RequestOutput(
        request_id="test-id",
        prompt=prompt,
        prompt_token_ids=prompt_token_ids,
        prompt_logprobs=None,
# ... 6 lines omitted for brevity ...
                logprobs=logprobs,
                finish_reason=None,
                stop_reason=None,
            )
        ],
        finished=finished,
        num_cached_tokens=num_cached_tokens,
    )
```
**EN:** This helper encapsulates reusable logic in `create_simple_context_output`. Key inputs are `text`, `token_ids`, `prompt`, `prompt_token_ids`, `num_cached_tokens`, `logprobs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_simple_context_output` 中。 关键输入包括 `text`、`token_ids`、`prompt`、`prompt_token_ids`、`num_cached_tokens`、`logprobs`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_simple_context_output_messages_empty (L678-L681)
```python
def test_simple_context_output_messages_empty():
    """output_messages should be empty before any output is appended."""
    context = SimpleContext()
    assert context.output_messages == []
```
**EN:** This test validates `test_simple_context_output_messages_empty`. The main assertion is `context.output_messages == []`.
**CN:** 这个测试验证 `test_simple_context_output_messages_empty`。 核心断言是 `context.output_messages == []`。

### Test / 测试: test_simple_context_output_messages_single_call (L684-L698)
```python
def test_simple_context_output_messages_single_call():
    """Non-streaming: single append_output produces a single output message."""
    context = SimpleContext()
    output = create_simple_context_output(
        text="Hello world",
        token_ids=[10, 20, 30],
        prompt_token_ids=[1, 2, 3],
    )
    context.append_output(output)

    messages = context.output_messages
    assert len(messages) == 1
    assert messages[0].message == "Hello world"
    assert messages[0].tokens == [10, 20, 30]
    assert messages[0].type == "raw_message_tokens"
```
**EN:** This test validates `test_simple_context_output_messages_single_call`. The main assertion is `len(messages) == 1` and `messages[0].message == 'Hello world'`.
**CN:** 这个测试验证 `test_simple_context_output_messages_single_call`。 核心断言是 `len(messages) == 1` and `messages[0].message == 'Hello world'`。

### Test / 测试: test_simple_context_output_messages_streaming_consolidation (L701-L731)
```python
def test_simple_context_output_messages_streaming_consolidation():
    """Streaming: multiple append_output calls consolidate into one message."""
    context = SimpleContext()

    # Simulate 3 streaming deltas
    context.append_output(
        create_simple_context_output(
            text="Hello",
            token_ids=[10],
            prompt_token_ids=[1, 2, 3],
        )
    )
    context.append_output(
        create_simple_context_output(
            text=" world",
            token_ids=[20],
            prompt_token_ids=[1, 2, 3],
        )
# ... 5 lines omitted for brevity ...
            prompt_token_ids=[1, 2, 3],
        )
    )

    messages = context.output_messages
    assert len(messages) == 1
    assert messages[0].message == "Hello world!"
    assert messages[0].tokens == [10, 20, 30]
```
**EN:** This test validates `test_simple_context_output_messages_streaming_consolidation`. The main assertion is `len(messages) == 1` and `messages[0].message == 'Hello world!'`.
**CN:** 这个测试验证 `test_simple_context_output_messages_streaming_consolidation`。 核心断言是 `len(messages) == 1` and `messages[0].message == 'Hello world!'`。

### Test / 测试: test_simple_context_output_messages_many_deltas (L734-L751)
```python
def test_simple_context_output_messages_many_deltas():
    """Streaming with many small deltas still produces a single message."""
    context = SimpleContext()

    words = ["The", " quick", " brown", " fox", " jumps"]
    for i, word in enumerate(words):
        context.append_output(
            create_simple_context_output(
                text=word,
                token_ids=[100 + i],
                prompt_token_ids=[1, 2],
            )
        )

    messages = context.output_messages
    assert len(messages) == 1
    assert messages[0].message == "The quick brown fox jumps"
    assert messages[0].tokens == [100, 101, 102, 103, 104]
```
**EN:** This test validates `test_simple_context_output_messages_many_deltas`. The main assertion is `len(messages) == 1` and `messages[0].message == 'The quick brown fox jumps'`.
**CN:** 这个测试验证 `test_simple_context_output_messages_many_deltas`。 核心断言是 `len(messages) == 1` and `messages[0].message == 'The quick brown fox jumps'`。

### Test / 测试: test_simple_context_input_messages (L754-L782)
```python
def test_simple_context_input_messages():
    """input_messages is populated on the first append_output call."""
    context = SimpleContext()
    assert context.input_messages == []

    context.append_output(
        create_simple_context_output(
            text="Hi",
            token_ids=[10],
            prompt="My prompt text",
            prompt_token_ids=[1, 2, 3],
        )
    )

    assert len(context.input_messages) == 1
    assert context.input_messages[0].message == "My prompt text"
    assert context.input_messages[0].tokens == [1, 2, 3]

    # Second call should not add another input message
    context.append_output(
        create_simple_context_output(
            text=" there",
            token_ids=[20],
            prompt="My prompt text",
            prompt_token_ids=[1, 2, 3],
        )
    )

    assert len(context.input_messages) == 1
```
**EN:** This test validates `test_simple_context_input_messages`. The main assertion is `context.input_messages == []` and `len(context.input_messages) == 1`.
**CN:** 这个测试验证 `test_simple_context_input_messages`。 核心断言是 `context.input_messages == []` and `len(context.input_messages) == 1`。

### Test / 测试: test_simple_context_token_counting (L785-L808)
```python
def test_simple_context_token_counting():
    """Token counting accumulates across streaming deltas."""
    context = SimpleContext()

    context.append_output(
        create_simple_context_output(
            text="a",
            token_ids=[10, 11],
            prompt_token_ids=[1, 2, 3, 4, 5],
            num_cached_tokens=2,
        )
    )
    context.append_output(
        create_simple_context_output(
            text="b",
            token_ids=[12],
            prompt_token_ids=[1, 2, 3, 4, 5],
            num_cached_tokens=2,
        )
    )

    assert context.num_prompt_tokens == 5
    assert context.num_output_tokens == 3  # 2 + 1
    assert context.num_cached_tokens == 2
```
**EN:** This test validates `test_simple_context_token_counting`. The main assertion is `context.num_prompt_tokens == 5` and `context.num_output_tokens == 3`.
**CN:** 这个测试验证 `test_simple_context_token_counting`。 核心断言是 `context.num_prompt_tokens == 5` and `context.num_output_tokens == 3`。

### Test / 测试: test_simple_context_final_output (L811-L833)
```python
def test_simple_context_final_output():
    """final_output reconstructs accumulated text and token_ids."""
    context = SimpleContext()

    context.append_output(
        create_simple_context_output(
            text="foo",
            token_ids=[1, 2],
            prompt_token_ids=[10],
        )
    )
    context.append_output(
        create_simple_context_output(
            text="bar",
            token_ids=[3],
            prompt_token_ids=[10],
        )
    )

    final = context.final_output
    assert final is not None
    assert final.outputs[0].text == "foobar"
    assert final.outputs[0].token_ids == (1, 2, 3)
```
**EN:** This test validates `test_simple_context_final_output`. The main assertion is `final is not None` and `final.outputs[0].text == 'foobar'`.
**CN:** 这个测试验证 `test_simple_context_final_output`。 核心断言是 `final is not None` and `final.outputs[0].text == 'foobar'`。

### Test / 测试: test_simple_context_output_messages_empty_text_with_tokens (L836-L851)
```python
def test_simple_context_output_messages_empty_text_with_tokens():
    """output_messages should be returned when tokens exist even if text is
    empty (e.g. special tokens)."""
    context = SimpleContext()
    context.append_output(
        create_simple_context_output(
            text="",
            token_ids=[99],
            prompt_token_ids=[1],
        )
    )

    messages = context.output_messages
    assert len(messages) == 1
    assert messages[0].message == ""
    assert messages[0].tokens == [99]
```
**EN:** This test validates `test_simple_context_output_messages_empty_text_with_tokens`. The main assertion is `len(messages) == 1` and `messages[0].message == ''`.
**CN:** 这个测试验证 `test_simple_context_output_messages_empty_text_with_tokens`。 核心断言是 `len(messages) == 1` and `messages[0].message == ''`。

### Test / 测试: test_simple_context_output_messages_no_mutation (L854-L883)
```python
def test_simple_context_output_messages_no_mutation():
    """Each call to output_messages returns a fresh list; callers can't
    corrupt internal state."""
    context = SimpleContext()
    context.append_output(
        create_simple_context_output(
            text="hello",
            token_ids=[1],
            prompt_token_ids=[10],
        )
    )

    msgs1 = context.output_messages
    msgs2 = context.output_messages
    assert msgs1 is not msgs2
    assert msgs1[0].message == msgs2[0].message

    # Appending more output updates the property
    context.append_output(
        create_simple_context_output(
            text=" world",
            token_ids=[2],
            prompt_token_ids=[10],
        )
    )

    msgs3 = context.output_messages
    assert len(msgs3) == 1
    assert msgs3[0].message == "hello world"
    assert msgs3[0].tokens == [1, 2]
```
**EN:** This test validates `test_simple_context_output_messages_no_mutation`. The main assertion is `msgs1 is not msgs2` and `msgs1[0].message == msgs2[0].message`.
**CN:** 这个测试验证 `test_simple_context_output_messages_no_mutation`。 核心断言是 `msgs1 is not msgs2` and `msgs1[0].message == msgs2[0].message`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `unittest.mock.MagicMock`, `unittest.mock.patch`
- **Third-party / 第三方**: `openai_harmony.Author`, `openai_harmony.Message`, `openai_harmony.Role`, `openai_harmony.StreamState`, `openai_harmony.TextContent`, `pytest`
- **Project / 项目内**: `vllm.entrypoints.openai.responses.context.HarmonyContext`, `vllm.entrypoints.openai.responses.context.SimpleContext`, `vllm.entrypoints.openai.responses.context.StreamingHarmonyContext`, `vllm.entrypoints.openai.responses.context.TurnMetrics`, `vllm.outputs.CompletionOutput`, `vllm.outputs.RequestOutput`
