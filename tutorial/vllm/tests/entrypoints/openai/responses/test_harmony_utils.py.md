# test_harmony_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_harmony_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 32 test(s), 0 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 32 个测试、0 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L17)
```python
from openai.types.responses import (
    ResponseFunctionToolCall,
    ResponseOutputMessage,
    ResponseReasoningItem,
)
from openai.types.responses.response_output_item import McpCall
from openai_harmony import Author, Message, Role, TextContent

from vllm.entrypoints.openai.responses.harmony import (
    harmony_to_response_output,
    parser_state_to_response_output,
    response_previous_input_to_harmony,
)
```
**EN:** Imports standard-library modules such as `unittest.mock.Mock`, third-party packages like `openai.types.responses.ResponseFunctionToolCall`, `openai.types.responses.ResponseOutputMessage`, `openai.types.responses.ResponseReasoningItem`, project helpers such as `vllm.entrypoints.openai.responses.harmony.harmony_to_response_output`, `vllm.entrypoints.openai.responses.harmony.parser_state_to_response_output`, `vllm.entrypoints.openai.responses.harmony.response_previous_input_to_harmony`.
**CN:** 导入标准库模块（如 `unittest.mock.Mock`）、第三方包（如 `openai.types.responses.ResponseFunctionToolCall`、`openai.types.responses.ResponseOutputMessage`、`openai.types.responses.ResponseReasoningItem`）、项目内辅助模块（如 `vllm.entrypoints.openai.responses.harmony.harmony_to_response_output`、`vllm.entrypoints.openai.responses.harmony.parser_state_to_response_output`、`vllm.entrypoints.openai.responses.harmony.response_previous_input_to_harmony`）。

### Class / 类: TestResponsePreviousInputToHarmony (L20-L92)
```python
class TestResponsePreviousInputToHarmony:
    """
    Tests for scenarios that are specific to the Responses API
    response_previous_input_to_harmony function.
    """

    def test_message_with_empty_content(self):
        """Test parsing message with empty string content."""
        chat_msg = {
            "role": "user",
            "content": "",
        }

        messages = response_previous_input_to_harmony(chat_msg)
# ... 51 lines omitted for brevity ...
        }

        messages = response_previous_input_to_harmony(chat_msg)

        assert len(messages) == 1
        assert messages[0].author.role == Role.TOOL
        assert messages[0].author.name == "functions.empty_tool"
        assert messages[0].content[0].text == ""
```
**EN:** This class groups related scenarios in `TestResponsePreviousInputToHarmony`. It contains 4 test method(s) and 0 supporting method(s). Representative methods include `test_message_with_empty_content`, `test_tool_message_with_string_content`, `test_tool_message_with_array_content`.
**CN:** 该类将与 `TestResponsePreviousInputToHarmony` 相关的场景组织在一起。 它包含 4 个测试方法和 0 个辅助方法。 代表性方法包括 `test_message_with_empty_content`、`test_tool_message_with_string_content`、`test_tool_message_with_array_content`。

### Test method / 测试方法: TestResponsePreviousInputToHarmony.test_message_with_empty_content (L26-L36)
```python
    def test_message_with_empty_content(self):
        """Test parsing message with empty string content."""
        chat_msg = {
            "role": "user",
            "content": "",
        }

        messages = response_previous_input_to_harmony(chat_msg)

        assert len(messages) == 1
        assert messages[0].content[0].text == ""
```
**EN:** This test validates `TestResponsePreviousInputToHarmony.test_message_with_empty_content`. The main assertion is `len(messages) == 1` and `messages[0].content[0].text == ''`.
**CN:** 这个测试验证 `TestResponsePreviousInputToHarmony.test_message_with_empty_content`。 核心断言是 `len(messages) == 1` and `messages[0].content[0].text == ''`。

### Test method / 测试方法: TestResponsePreviousInputToHarmony.test_tool_message_with_string_content (L38-L54)
```python
    def test_tool_message_with_string_content(self):
        """Test parsing tool message with string content."""
        chat_msg = {
            "role": "tool",
            "name": "get_weather",
            "content": "The weather in San Francisco is sunny, 72°F",
        }

        messages = response_previous_input_to_harmony(chat_msg)

        assert len(messages) == 1
        assert messages[0].author.role == Role.TOOL
        assert messages[0].author.name == "functions.get_weather"
        assert (
            messages[0].content[0].text == "The weather in San Francisco is sunny, 72°F"
        )
        assert messages[0].channel == "commentary"
```
**EN:** This test validates `TestResponsePreviousInputToHarmony.test_tool_message_with_string_content`. The main assertion is `len(messages) == 1` and `messages[0].author.role == Role.TOOL`.
**CN:** 这个测试验证 `TestResponsePreviousInputToHarmony.test_tool_message_with_string_content`。 核心断言是 `len(messages) == 1` and `messages[0].author.role == Role.TOOL`。

### Test method / 测试方法: TestResponsePreviousInputToHarmony.test_tool_message_with_array_content (L56-L77)
```python
    def test_tool_message_with_array_content(self):
        """Test parsing tool message with array content."""
        chat_msg = {
            "role": "tool",
            "name": "search_results",
            "content": [
                {"type": "text", "text": "Result 1: "},
                {"type": "text", "text": "Result 2: "},
                {
                    "type": "image",
                    "url": "http://example.com/img.png",
                },  # Should be ignored
                {"type": "text", "text": "Result 3"},
            ],
        }

        messages = response_previous_input_to_harmony(chat_msg)

        assert len(messages) == 1
        assert messages[0].author.role == Role.TOOL
        assert messages[0].author.name == "functions.search_results"
        assert messages[0].content[0].text == "Result 1: Result 2: Result 3"
```
**EN:** This test validates `TestResponsePreviousInputToHarmony.test_tool_message_with_array_content`. The main assertion is `len(messages) == 1` and `messages[0].author.role == Role.TOOL`.
**CN:** 这个测试验证 `TestResponsePreviousInputToHarmony.test_tool_message_with_array_content`。 核心断言是 `len(messages) == 1` and `messages[0].author.role == Role.TOOL`。

### Test method / 测试方法: TestResponsePreviousInputToHarmony.test_tool_message_with_empty_content (L79-L92)
```python
    def test_tool_message_with_empty_content(self):
        """Test parsing tool message with None content."""
        chat_msg = {
            "role": "tool",
            "name": "empty_tool",
            "content": None,
        }

        messages = response_previous_input_to_harmony(chat_msg)

        assert len(messages) == 1
        assert messages[0].author.role == Role.TOOL
        assert messages[0].author.name == "functions.empty_tool"
        assert messages[0].content[0].text == ""
```
**EN:** This test validates `TestResponsePreviousInputToHarmony.test_tool_message_with_empty_content`. The main assertion is `len(messages) == 1` and `messages[0].author.role == Role.TOOL`.
**CN:** 这个测试验证 `TestResponsePreviousInputToHarmony.test_tool_message_with_empty_content`。 核心断言是 `len(messages) == 1` and `messages[0].author.role == Role.TOOL`。

### Class / 类: TestHarmonyToResponseOutput (L95-L287)
```python
class TestHarmonyToResponseOutput:
    """Tests for harmony_to_response_output function."""

    def test_commentary_with_no_recipient_creates_message(self):
        """Test that commentary with recipient=None (preambles) creates message items.

        Per Harmony format, preambles are intended to be shown to end-users,
        unlike analysis channel content which is hidden reasoning.
        See: https://cookbook.openai.com/articles/openai-harmony
        """
        message = Message.from_role_and_content(
            Role.ASSISTANT, "I will now search for the weather information."
        )
        message = message.with_channel("commentary")
# ... 171 lines omitted for brevity ...
        message = Message.from_author_and_content(
            Author.new(Role.TOOL, "functions.get_weather"),
            "The weather is sunny, 72°F",
        )

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 0
```
**EN:** This class groups related scenarios in `TestHarmonyToResponseOutput`. It contains 11 test method(s) and 0 supporting method(s). Representative methods include `test_commentary_with_no_recipient_creates_message`, `test_commentary_with_function_recipient_creates_function_call`, `test_commentary_with_python_recipient_creates_reasoning`.
**CN:** 该类将与 `TestHarmonyToResponseOutput` 相关的场景组织在一起。 它包含 11 个测试方法和 0 个辅助方法。 代表性方法包括 `test_commentary_with_no_recipient_creates_message`、`test_commentary_with_function_recipient_creates_function_call`、`test_commentary_with_python_recipient_creates_reasoning`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_commentary_with_no_recipient_creates_message (L98-L123)
```python
    def test_commentary_with_no_recipient_creates_message(self):
        """Test that commentary with recipient=None (preambles) creates message items.

        Per Harmony format, preambles are intended to be shown to end-users,
        unlike analysis channel content which is hidden reasoning.
        See: https://cookbook.openai.com/articles/openai-harmony
        """
        message = Message.from_role_and_content(
            Role.ASSISTANT, "I will now search for the weather information."
        )
        message = message.with_channel("commentary")
        # recipient is None by default, representing a preamble

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseOutputMessage)
        assert output_items[0].type == "message"
        assert output_items[0].role == "assistant"
        assert output_items[0].status == "completed"
        assert len(output_items[0].content) == 1
        assert output_items[0].content[0].type == "output_text"
        assert (
            output_items[0].content[0].text
            == "I will now search for the weather information."
        )
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_commentary_with_no_recipient_creates_message`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseOutputMessage)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_commentary_with_no_recipient_creates_message`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseOutputMessage)`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_commentary_with_function_recipient_creates_function_call (L125-L144)
```python
    def test_commentary_with_function_recipient_creates_function_call(self):
        """Test commentary with recipient='functions.X' creates function calls."""
        message = Message.from_role_and_content(
            Role.ASSISTANT, '{"location": "San Francisco", "units": "celsius"}'
        )
        message = message.with_channel("commentary")
        message = message.with_recipient("functions.get_weather")

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseFunctionToolCall)
        assert output_items[0].type == "function_call"
        assert output_items[0].name == "get_weather"
        assert (
            output_items[0].arguments
            == '{"location": "San Francisco", "units": "celsius"}'
        )
        assert output_items[0].call_id.startswith("call_")
        assert output_items[0].id.startswith("fc_")
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_commentary_with_function_recipient_creates_function_call`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_commentary_with_function_recipient_creates_function_call`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_commentary_with_python_recipient_creates_reasoning (L146-L162)
```python
    def test_commentary_with_python_recipient_creates_reasoning(self):
        """Test that commentary with recipient='python' creates reasoning items."""
        message = Message.from_role_and_content(
            Role.ASSISTANT, "import numpy as np\nprint(np.array([1, 2, 3]))"
        )
        message = message.with_channel("commentary")
        message = message.with_recipient("python")

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseReasoningItem)
        assert output_items[0].type == "reasoning"
        assert (
            output_items[0].content[0].text
            == "import numpy as np\nprint(np.array([1, 2, 3]))"
        )
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_commentary_with_python_recipient_creates_reasoning`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseReasoningItem)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_commentary_with_python_recipient_creates_reasoning`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseReasoningItem)`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_commentary_with_browser_recipient_creates_reasoning (L164-L177)
```python
    def test_commentary_with_browser_recipient_creates_reasoning(self):
        """Test that commentary with recipient='browser' creates reasoning items."""
        message = Message.from_role_and_content(
            Role.ASSISTANT, "Navigating to the specified URL"
        )
        message = message.with_channel("commentary")
        message = message.with_recipient("browser")

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseReasoningItem)
        assert output_items[0].type == "reasoning"
        assert output_items[0].content[0].text == "Navigating to the specified URL"
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_commentary_with_browser_recipient_creates_reasoning`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseReasoningItem)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_commentary_with_browser_recipient_creates_reasoning`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseReasoningItem)`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_commentary_with_container_recipient_creates_reasoning (L179-L192)
```python
    def test_commentary_with_container_recipient_creates_reasoning(self):
        """Test that commentary with recipient='container' creates reasoning items."""
        message = Message.from_role_and_content(
            Role.ASSISTANT, "Running command in container"
        )
        message = message.with_channel("commentary")
        message = message.with_recipient("container")

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseReasoningItem)
        assert output_items[0].type == "reasoning"
        assert output_items[0].content[0].text == "Running command in container"
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_commentary_with_container_recipient_creates_reasoning`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseReasoningItem)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_commentary_with_container_recipient_creates_reasoning`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseReasoningItem)`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_commentary_with_empty_content_and_no_recipient (L194-L203)
```python
    def test_commentary_with_empty_content_and_no_recipient(self):
        """Test edge case: empty commentary with recipient=None."""
        message = Message.from_role_and_content(Role.ASSISTANT, "")
        message = message.with_channel("commentary")

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseOutputMessage)
        assert output_items[0].content[0].text == ""
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_commentary_with_empty_content_and_no_recipient`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseOutputMessage)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_commentary_with_empty_content_and_no_recipient`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseOutputMessage)`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_commentary_with_multiple_contents_and_no_recipient (L205-L222)
```python
    def test_commentary_with_multiple_contents_and_no_recipient(self):
        """Test multiple content items in commentary with no recipient."""
        contents = [
            TextContent(text="Step 1: Analyze the request"),
            TextContent(text="Step 2: Prepare to call functions"),
        ]
        message = Message.from_role_and_contents(Role.ASSISTANT, contents)
        message = message.with_channel("commentary")

        output_items = harmony_to_response_output(message)

        # _parse_final_message returns single ResponseOutputMessage with
        # multiple contents
        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseOutputMessage)
        assert len(output_items[0].content) == 2
        assert output_items[0].content[0].text == "Step 1: Analyze the request"
        assert output_items[0].content[1].text == "Step 2: Prepare to call functions"
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_commentary_with_multiple_contents_and_no_recipient`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseOutputMessage)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_commentary_with_multiple_contents_and_no_recipient`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseOutputMessage)`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_commentary_with_multiple_function_calls (L224-L241)
```python
    def test_commentary_with_multiple_function_calls(self):
        """Test multiple function calls in commentary channel."""
        contents = [
            TextContent(text='{"location": "San Francisco"}'),
            TextContent(text='{"location": "New York"}'),
        ]
        message = Message.from_role_and_contents(Role.ASSISTANT, contents)
        message = message.with_channel("commentary")
        message = message.with_recipient("functions.get_weather")

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 2
        assert all(isinstance(item, ResponseFunctionToolCall) for item in output_items)
        assert output_items[0].name == "get_weather"
        assert output_items[1].name == "get_weather"
        assert output_items[0].arguments == '{"location": "San Francisco"}'
        assert output_items[1].arguments == '{"location": "New York"}'
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_commentary_with_multiple_function_calls`. The main assertion is `len(output_items) == 2` and `all((isinstance(item, ResponseFunctionToolCall) for item in output_items))`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_commentary_with_multiple_function_calls`。 核心断言是 `len(output_items) == 2` and `all((isinstance(item, ResponseFunctionToolCall) for item in output_items))`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_commentary_with_unknown_recipient_creates_mcp_call (L243-L256)
```python
    def test_commentary_with_unknown_recipient_creates_mcp_call(self):
        """Test that commentary with unknown recipient creates MCP call."""
        message = Message.from_role_and_content(Role.ASSISTANT, '{"arg": "value"}')
        message = message.with_channel("commentary")
        message = message.with_recipient("custom_tool")

        fn_names = frozenset({"other_tool"})
        output_items = harmony_to_response_output(message, fn_names)

        assert len(output_items) == 1
        assert isinstance(output_items[0], McpCall)
        assert output_items[0].type == "mcp_call"
        assert output_items[0].name == "custom_tool"
        assert output_items[0].server_label == "custom_tool"
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_commentary_with_unknown_recipient_creates_mcp_call`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], McpCall)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_commentary_with_unknown_recipient_creates_mcp_call`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], McpCall)`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_analysis_channel_creates_reasoning (L258-L272)
```python
    def test_analysis_channel_creates_reasoning(self):
        """Test that analysis channel creates reasoning items."""
        message = Message.from_role_and_content(
            Role.ASSISTANT, "Analyzing the problem step by step..."
        )
        message = message.with_channel("analysis")

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseReasoningItem)
        assert output_items[0].type == "reasoning"
        assert (
            output_items[0].content[0].text == "Analyzing the problem step by step..."
        )
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_analysis_channel_creates_reasoning`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseReasoningItem)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_analysis_channel_creates_reasoning`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseReasoningItem)`。

### Test method / 测试方法: TestHarmonyToResponseOutput.test_non_assistant_message_returns_empty (L274-L287)
```python
    def test_non_assistant_message_returns_empty(self):
        """Test that non-assistant messages return empty list.

        Per the implementation, tool messages to assistant (e.g., search results)
        are not included in final output to align with OpenAI behavior.
        """
        message = Message.from_author_and_content(
            Author.new(Role.TOOL, "functions.get_weather"),
            "The weather is sunny, 72°F",
        )

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 0
```
**EN:** This test validates `TestHarmonyToResponseOutput.test_non_assistant_message_returns_empty`. The main assertion is `len(output_items) == 0`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutput.test_non_assistant_message_returns_empty`。 核心断言是 `len(output_items) == 0`。

### Class / 类: TestHarmonyToResponseOutputWithFunctionToolNames (L290-L357)
```python
class TestHarmonyToResponseOutputWithFunctionToolNames:
    """Tests for bare function name handling with function_tool_names."""

    def test_bare_name_creates_function_call_when_in_tool_names(self):
        """Bare function name matching a known tool creates function call."""
        message = Message.from_role_and_content(
            Role.ASSISTANT, '{"location": "San Francisco"}'
        )
        message = message.with_channel("commentary")
        message = message.with_recipient("get_weather")

        fn_names = frozenset({"get_weather"})
        output_items = harmony_to_response_output(message, fn_names)

# ... 46 lines omitted for brevity ...
        message = message.with_channel("commentary")
        message = message.with_recipient("functions.get_weather")

        output_items = harmony_to_response_output(message, frozenset())

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseFunctionToolCall)
        assert output_items[0].name == "get_weather"
```
**EN:** This class groups related scenarios in `TestHarmonyToResponseOutputWithFunctionToolNames`. It contains 5 test method(s) and 0 supporting method(s). Representative methods include `test_bare_name_creates_function_call_when_in_tool_names`, `test_bare_name_creates_mcp_call_when_not_in_tool_names`, `test_dotted_function_name_creates_function_call`.
**CN:** 该类将与 `TestHarmonyToResponseOutputWithFunctionToolNames` 相关的场景组织在一起。 它包含 5 个测试方法和 0 个辅助方法。 代表性方法包括 `test_bare_name_creates_function_call_when_in_tool_names`、`test_bare_name_creates_mcp_call_when_not_in_tool_names`、`test_dotted_function_name_creates_function_call`。

### Test method / 测试方法: TestHarmonyToResponseOutputWithFunctionToolNames.test_bare_name_creates_function_call_when_in_tool_names (L293-L308)
```python
    def test_bare_name_creates_function_call_when_in_tool_names(self):
        """Bare function name matching a known tool creates function call."""
        message = Message.from_role_and_content(
            Role.ASSISTANT, '{"location": "San Francisco"}'
        )
        message = message.with_channel("commentary")
        message = message.with_recipient("get_weather")

        fn_names = frozenset({"get_weather"})
        output_items = harmony_to_response_output(message, fn_names)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseFunctionToolCall)
        assert output_items[0].type == "function_call"
        assert output_items[0].name == "get_weather"
        assert output_items[0].arguments == '{"location": "San Francisco"}'
```
**EN:** This test validates `TestHarmonyToResponseOutputWithFunctionToolNames.test_bare_name_creates_function_call_when_in_tool_names`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutputWithFunctionToolNames.test_bare_name_creates_function_call_when_in_tool_names`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`。

### Test method / 测试方法: TestHarmonyToResponseOutputWithFunctionToolNames.test_bare_name_creates_mcp_call_when_not_in_tool_names (L310-L321)
```python
    def test_bare_name_creates_mcp_call_when_not_in_tool_names(self):
        """Bare name not matching any known tool creates MCP call."""
        message = Message.from_role_and_content(Role.ASSISTANT, '{"arg": "value"}')
        message = message.with_channel("commentary")
        message = message.with_recipient("custom_tool")

        fn_names = frozenset({"get_weather"})
        output_items = harmony_to_response_output(message, fn_names)

        assert len(output_items) == 1
        assert isinstance(output_items[0], McpCall)
        assert output_items[0].type == "mcp_call"
```
**EN:** This test validates `TestHarmonyToResponseOutputWithFunctionToolNames.test_bare_name_creates_mcp_call_when_not_in_tool_names`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], McpCall)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutputWithFunctionToolNames.test_bare_name_creates_mcp_call_when_not_in_tool_names`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], McpCall)`。

### Test method / 测试方法: TestHarmonyToResponseOutputWithFunctionToolNames.test_dotted_function_name_creates_function_call (L323-L334)
```python
    def test_dotted_function_name_creates_function_call(self):
        """Dotted function name in tool names creates function call."""
        message = Message.from_role_and_content(Role.ASSISTANT, '{"a": 1, "b": 2}')
        message = message.with_channel("commentary")
        message = message.with_recipient("math.sum")

        fn_names = frozenset({"math.sum"})
        output_items = harmony_to_response_output(message, fn_names)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseFunctionToolCall)
        assert output_items[0].name == "math.sum"
```
**EN:** This test validates `TestHarmonyToResponseOutputWithFunctionToolNames.test_dotted_function_name_creates_function_call`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutputWithFunctionToolNames.test_dotted_function_name_creates_function_call`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`。

### Test method / 测试方法: TestHarmonyToResponseOutputWithFunctionToolNames.test_empty_tool_names_defaults_to_mcp (L336-L345)
```python
    def test_empty_tool_names_defaults_to_mcp(self):
        """With empty function_tool_names, bare names become MCP calls."""
        message = Message.from_role_and_content(Role.ASSISTANT, '{"arg": "value"}')
        message = message.with_channel("commentary")
        message = message.with_recipient("get_weather")

        output_items = harmony_to_response_output(message, frozenset())

        assert len(output_items) == 1
        assert isinstance(output_items[0], McpCall)
```
**EN:** This test validates `TestHarmonyToResponseOutputWithFunctionToolNames.test_empty_tool_names_defaults_to_mcp`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], McpCall)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutputWithFunctionToolNames.test_empty_tool_names_defaults_to_mcp`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], McpCall)`。

### Test method / 测试方法: TestHarmonyToResponseOutputWithFunctionToolNames.test_prefixed_name_always_function_call (L347-L357)
```python
    def test_prefixed_name_always_function_call(self):
        """functions. prefix always creates function call even with empty tool names."""
        message = Message.from_role_and_content(Role.ASSISTANT, '{"arg": "value"}')
        message = message.with_channel("commentary")
        message = message.with_recipient("functions.get_weather")

        output_items = harmony_to_response_output(message, frozenset())

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseFunctionToolCall)
        assert output_items[0].name == "get_weather"
```
**EN:** This test validates `TestHarmonyToResponseOutputWithFunctionToolNames.test_prefixed_name_always_function_call`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`.
**CN:** 这个测试验证 `TestHarmonyToResponseOutputWithFunctionToolNames.test_prefixed_name_always_function_call`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`。

### Class / 类: TestParserStateWithFunctionToolNames (L360-L394)
```python
class TestParserStateWithFunctionToolNames:
    """Tests for parser_state_to_response_output with function_tool_names."""

    def test_bare_name_creates_function_call(self):
        from unittest.mock import Mock

        parser = Mock()
        parser.current_content = '{"arg": "value"}'
        parser.current_role = Role.ASSISTANT
        parser.current_channel = "commentary"
        parser.current_recipient = "get_weather"

        fn_names = frozenset({"get_weather"})
        items = parser_state_to_response_output(parser, fn_names)
# ... 13 lines omitted for brevity ...
        parser.current_recipient = "unknown_tool"

        fn_names = frozenset({"get_weather"})
        items = parser_state_to_response_output(parser, fn_names)

        assert len(items) == 1
        assert isinstance(items[0], McpCall)
        assert items[0].name == "unknown_tool"
```
**EN:** This class groups related scenarios in `TestParserStateWithFunctionToolNames`. It contains 2 test method(s) and 0 supporting method(s). Representative methods include `test_bare_name_creates_function_call`, `test_bare_name_creates_mcp_when_not_in_tool_names`.
**CN:** 该类将与 `TestParserStateWithFunctionToolNames` 相关的场景组织在一起。 它包含 2 个测试方法和 0 个辅助方法。 代表性方法包括 `test_bare_name_creates_function_call`、`test_bare_name_creates_mcp_when_not_in_tool_names`。

### Test method / 测试方法: TestParserStateWithFunctionToolNames.test_bare_name_creates_function_call (L363-L378)
```python
    def test_bare_name_creates_function_call(self):
        from unittest.mock import Mock

        parser = Mock()
        parser.current_content = '{"arg": "value"}'
        parser.current_role = Role.ASSISTANT
        parser.current_channel = "commentary"
        parser.current_recipient = "get_weather"

        fn_names = frozenset({"get_weather"})
        items = parser_state_to_response_output(parser, fn_names)

        assert len(items) == 1
        assert isinstance(items[0], ResponseFunctionToolCall)
        assert items[0].name == "get_weather"
        assert items[0].status == "in_progress"
```
**EN:** This test validates `TestParserStateWithFunctionToolNames.test_bare_name_creates_function_call`. The main assertion is `len(items) == 1` and `isinstance(items[0], ResponseFunctionToolCall)`.
**CN:** 这个测试验证 `TestParserStateWithFunctionToolNames.test_bare_name_creates_function_call`。 核心断言是 `len(items) == 1` and `isinstance(items[0], ResponseFunctionToolCall)`。

### Test method / 测试方法: TestParserStateWithFunctionToolNames.test_bare_name_creates_mcp_when_not_in_tool_names (L380-L394)
```python
    def test_bare_name_creates_mcp_when_not_in_tool_names(self):
        from unittest.mock import Mock

        parser = Mock()
        parser.current_content = '{"arg": "value"}'
        parser.current_role = Role.ASSISTANT
        parser.current_channel = "commentary"
        parser.current_recipient = "unknown_tool"

        fn_names = frozenset({"get_weather"})
        items = parser_state_to_response_output(parser, fn_names)

        assert len(items) == 1
        assert isinstance(items[0], McpCall)
        assert items[0].name == "unknown_tool"
```
**EN:** This test validates `TestParserStateWithFunctionToolNames.test_bare_name_creates_mcp_when_not_in_tool_names`. The main assertion is `len(items) == 1` and `isinstance(items[0], McpCall)`.
**CN:** 这个测试验证 `TestParserStateWithFunctionToolNames.test_bare_name_creates_mcp_when_not_in_tool_names`。 核心断言是 `len(items) == 1` and `isinstance(items[0], McpCall)`。

### Class / 类: TestToolCallsOnNonStandardChannels (L397-L452)
```python
class TestToolCallsOnNonStandardChannels:
    """Tests verifying tool calls are detected regardless of channel."""

    def test_function_call_on_comment_channel(self):
        message = Message.from_role_and_content(Role.ASSISTANT, '{"query": "weather"}')
        message = message.with_channel("comment")
        message = message.with_recipient("functions.get_weather")

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseFunctionToolCall)
        assert output_items[0].type == "function_call"
        assert output_items[0].name == "get_weather"
# ... 34 lines omitted for brevity ...
        parser.current_channel = "comment"
        parser.current_recipient = "mcp.server.tool"

        fn_names: frozenset[str] = frozenset()
        items = parser_state_to_response_output(parser, fn_names)

        assert len(items) == 1
        assert isinstance(items[0], McpCall)
```
**EN:** This class groups related scenarios in `TestToolCallsOnNonStandardChannels`. It contains 4 test method(s) and 0 supporting method(s). Representative methods include `test_function_call_on_comment_channel`, `test_bare_function_on_comment_channel`, `test_parser_state_comment_channel_function`.
**CN:** 该类将与 `TestToolCallsOnNonStandardChannels` 相关的场景组织在一起。 它包含 4 个测试方法和 0 个辅助方法。 代表性方法包括 `test_function_call_on_comment_channel`、`test_bare_function_on_comment_channel`、`test_parser_state_comment_channel_function`。

### Test method / 测试方法: TestToolCallsOnNonStandardChannels.test_function_call_on_comment_channel (L400-L410)
```python
    def test_function_call_on_comment_channel(self):
        message = Message.from_role_and_content(Role.ASSISTANT, '{"query": "weather"}')
        message = message.with_channel("comment")
        message = message.with_recipient("functions.get_weather")

        output_items = harmony_to_response_output(message)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseFunctionToolCall)
        assert output_items[0].type == "function_call"
        assert output_items[0].name == "get_weather"
```
**EN:** This test validates `TestToolCallsOnNonStandardChannels.test_function_call_on_comment_channel`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`.
**CN:** 这个测试验证 `TestToolCallsOnNonStandardChannels.test_function_call_on_comment_channel`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`。

### Test method / 测试方法: TestToolCallsOnNonStandardChannels.test_bare_function_on_comment_channel (L412-L422)
```python
    def test_bare_function_on_comment_channel(self):
        message = Message.from_role_and_content(Role.ASSISTANT, '{"query": "weather"}')
        message = message.with_channel("comment")
        message = message.with_recipient("get_weather")

        fn_names = frozenset({"get_weather"})
        output_items = harmony_to_response_output(message, fn_names)

        assert len(output_items) == 1
        assert isinstance(output_items[0], ResponseFunctionToolCall)
        assert output_items[0].name == "get_weather"
```
**EN:** This test validates `TestToolCallsOnNonStandardChannels.test_bare_function_on_comment_channel`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`.
**CN:** 这个测试验证 `TestToolCallsOnNonStandardChannels.test_bare_function_on_comment_channel`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], ResponseFunctionToolCall)`。

### Test method / 测试方法: TestToolCallsOnNonStandardChannels.test_parser_state_comment_channel_function (L424-L437)
```python
    def test_parser_state_comment_channel_function(self):
        from unittest.mock import Mock

        parser = Mock()
        parser.current_content = '{"arg": "value"}'
        parser.current_role = Role.ASSISTANT
        parser.current_channel = "comment"
        parser.current_recipient = "functions.get_weather"

        items = parser_state_to_response_output(parser)

        assert len(items) == 1
        assert isinstance(items[0], ResponseFunctionToolCall)
        assert items[0].name == "get_weather"
```
**EN:** This test validates `TestToolCallsOnNonStandardChannels.test_parser_state_comment_channel_function`. The main assertion is `len(items) == 1` and `isinstance(items[0], ResponseFunctionToolCall)`.
**CN:** 这个测试验证 `TestToolCallsOnNonStandardChannels.test_parser_state_comment_channel_function`。 核心断言是 `len(items) == 1` and `isinstance(items[0], ResponseFunctionToolCall)`。

### Test method / 测试方法: TestToolCallsOnNonStandardChannels.test_parser_state_comment_channel_mcp (L439-L452)
```python
    def test_parser_state_comment_channel_mcp(self):
        from unittest.mock import Mock

        parser = Mock()
        parser.current_content = '{"arg": "value"}'
        parser.current_role = Role.ASSISTANT
        parser.current_channel = "comment"
        parser.current_recipient = "mcp.server.tool"

        fn_names: frozenset[str] = frozenset()
        items = parser_state_to_response_output(parser, fn_names)

        assert len(items) == 1
        assert isinstance(items[0], McpCall)
```
**EN:** This test validates `TestToolCallsOnNonStandardChannels.test_parser_state_comment_channel_mcp`. The main assertion is `len(items) == 1` and `isinstance(items[0], McpCall)`.
**CN:** 这个测试验证 `TestToolCallsOnNonStandardChannels.test_parser_state_comment_channel_mcp`。 核心断言是 `len(items) == 1` and `isinstance(items[0], McpCall)`。

### Test / 测试: test_parse_mcp_call_basic (L455-L470)
```python
def test_parse_mcp_call_basic() -> None:
    """Test that MCP calls are parsed with correct type and server_label."""
    message = Message.from_role_and_content(Role.ASSISTANT, '{"path": "/tmp"}')
    message = message.with_recipient("filesystem")
    message = message.with_channel("commentary")

    fn_names: frozenset[str] = frozenset()
    output_items = harmony_to_response_output(message, fn_names)

    assert len(output_items) == 1
    assert isinstance(output_items[0], McpCall)
    assert output_items[0].type == "mcp_call"
    assert output_items[0].name == "filesystem"
    assert output_items[0].server_label == "filesystem"
    assert output_items[0].arguments == '{"path": "/tmp"}'
    assert output_items[0].status == "completed"
```
**EN:** This test validates `test_parse_mcp_call_basic`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], McpCall)`.
**CN:** 这个测试验证 `test_parse_mcp_call_basic`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], McpCall)`。

### Test / 测试: test_parse_mcp_call_dotted_recipient (L473-L485)
```python
def test_parse_mcp_call_dotted_recipient() -> None:
    """Test that dotted recipients extract the tool name correctly."""
    message = Message.from_role_and_content(Role.ASSISTANT, '{"cmd": "ls"}')
    message = message.with_recipient("repo_browser.list")
    message = message.with_channel("commentary")

    fn_names: frozenset[str] = frozenset()
    output_items = harmony_to_response_output(message, fn_names)

    assert len(output_items) == 1
    assert isinstance(output_items[0], McpCall)
    assert output_items[0].name == "list"
    assert output_items[0].server_label == "repo_browser"
```
**EN:** This test validates `test_parse_mcp_call_dotted_recipient`. The main assertion is `len(output_items) == 1` and `isinstance(output_items[0], McpCall)`.
**CN:** 这个测试验证 `test_parse_mcp_call_dotted_recipient`。 核心断言是 `len(output_items) == 1` and `isinstance(output_items[0], McpCall)`。

### Test / 测试: test_mcp_vs_function_call (L488-L498)
```python
def test_mcp_vs_function_call() -> None:
    """Test that function calls are not parsed as MCP calls."""
    func_message = Message.from_role_and_content(Role.ASSISTANT, '{"arg": "value"}')
    func_message = func_message.with_recipient("functions.my_tool")
    func_message = func_message.with_channel("commentary")

    func_items = harmony_to_response_output(func_message)

    assert len(func_items) == 1
    assert not isinstance(func_items[0], McpCall)
    assert func_items[0].type == "function_call"
```
**EN:** This test validates `test_mcp_vs_function_call`. The main assertion is `len(func_items) == 1` and `not isinstance(func_items[0], McpCall)`.
**CN:** 这个测试验证 `test_mcp_vs_function_call`。 核心断言是 `len(func_items) == 1` and `not isinstance(func_items[0], McpCall)`。

### Test / 测试: test_mcp_vs_builtin_tools (L501-L512)
```python
def test_mcp_vs_builtin_tools() -> None:
    """Test that built-in tools (python, container) are not parsed as MCP calls."""
    # Test python (built-in tool) - should be reasoning, not MCP
    python_message = Message.from_role_and_content(Role.ASSISTANT, "print('hello')")
    python_message = python_message.with_recipient("python")
    python_message = python_message.with_channel("commentary")

    python_items = harmony_to_response_output(python_message)

    assert len(python_items) == 1
    assert not isinstance(python_items[0], McpCall)
    assert python_items[0].type == "reasoning"
```
**EN:** This test validates `test_mcp_vs_builtin_tools`. The main assertion is `len(python_items) == 1` and `not isinstance(python_items[0], McpCall)`.
**CN:** 这个测试验证 `test_mcp_vs_builtin_tools`。 核心断言是 `len(python_items) == 1` and `not isinstance(python_items[0], McpCall)`。

### Test / 测试: test_parser_state_to_response_output_commentary_channel (L515-L580)
```python
def test_parser_state_to_response_output_commentary_channel() -> None:
    """Test parser_state_to_response_output with commentary
    channel and various recipients."""
    from unittest.mock import Mock

    # Test 1: functions.* recipient -> should return function tool call
    parser_func = Mock()
    parser_func.current_content = '{"arg": "value"}'
    parser_func.current_role = Role.ASSISTANT
    parser_func.current_channel = "commentary"
    parser_func.current_recipient = "functions.my_tool"

    func_items = parser_state_to_response_output(parser_func)

    assert len(func_items) == 1
    assert not isinstance(func_items[0], McpCall)
    assert func_items[0].type == "function_call"
    assert func_items[0].name == "my_tool"
# ... 40 lines omitted for brevity ...

    preamble_items = parser_state_to_response_output(parser_preamble)

    assert len(preamble_items) == 1
    assert isinstance(preamble_items[0], ResponseOutputMessage)
    assert preamble_items[0].type == "message"
    assert preamble_items[0].content[0].text == "I'll search for that information now."
    assert preamble_items[0].status == "incomplete"  # streaming
```
**EN:** This test validates `test_parser_state_to_response_output_commentary_channel`. The main assertion is `len(func_items) == 1` and `not isinstance(func_items[0], McpCall)`.
**CN:** 这个测试验证 `test_parser_state_to_response_output_commentary_channel`。 核心断言是 `len(func_items) == 1` and `not isinstance(func_items[0], McpCall)`。

### Test / 测试: test_parser_state_to_response_output_analysis_channel (L583-L633)
```python
def test_parser_state_to_response_output_analysis_channel() -> None:
    """Test parser_state_to_response_output with analysis
    channel and various recipients."""
    from unittest.mock import Mock

    # Test 1: functions.* recipient -> should return function tool call
    parser_func = Mock()
    parser_func.current_content = '{"arg": "value"}'
    parser_func.current_role = Role.ASSISTANT
    parser_func.current_channel = "analysis"
    parser_func.current_recipient = "functions.my_tool"

    func_items = parser_state_to_response_output(parser_func)

    assert len(func_items) == 1
    assert not isinstance(func_items[0], McpCall)
    assert func_items[0].type == "function_call"
    assert func_items[0].name == "my_tool"
# ... 25 lines omitted for brevity ...
    parser_builtin.current_recipient = "container"

    builtin_items = parser_state_to_response_output(parser_builtin)

    # Should fall through to reasoning logic
    assert len(builtin_items) == 1
    assert not isinstance(builtin_items[0], McpCall)
    assert builtin_items[0].type == "reasoning"
```
**EN:** This test validates `test_parser_state_to_response_output_analysis_channel`. The main assertion is `len(func_items) == 1` and `not isinstance(func_items[0], McpCall)`.
**CN:** 这个测试验证 `test_parser_state_to_response_output_analysis_channel`。 核心断言是 `len(func_items) == 1` and `not isinstance(func_items[0], McpCall)`。

## Key Concepts / 关键概念
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `unittest.mock.Mock`
- **Third-party / 第三方**: `openai.types.responses.ResponseFunctionToolCall`, `openai.types.responses.ResponseOutputMessage`, `openai.types.responses.ResponseReasoningItem`, `openai.types.responses.response_output_item.McpCall`, `openai_harmony.Author`, `openai_harmony.Message`, `openai_harmony.Role`, `openai_harmony.TextContent`
- **Project / 项目内**: `vllm.entrypoints.openai.responses.harmony.harmony_to_response_output`, `vllm.entrypoints.openai.responses.harmony.parser_state_to_response_output`, `vllm.entrypoints.openai.responses.harmony.response_previous_input_to_harmony`
