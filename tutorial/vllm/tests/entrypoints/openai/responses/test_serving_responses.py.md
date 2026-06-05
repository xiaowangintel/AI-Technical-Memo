# test_serving_responses.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_serving_responses.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 20 test(s), 5 fixture(s), and 31 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 20 个测试、5 个 fixture，以及 31 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L54)
```python
from contextlib import AsyncExitStack
from unittest.mock import MagicMock

import pytest
import pytest_asyncio
from openai.types.responses import (
    ResponseOutputItemDoneEvent,
    ResponseReasoningItem,
    ResponseReasoningTextDeltaEvent,
    ResponseReasoningTextDoneEvent,
    ResponseTextConfig,
    ResponseTextDeltaEvent,
)
from openai.types.responses.response_format_text_json_schema_config import (
    ResponseFormatTextJSONSchemaConfig,
)
from openai.types.responses.tool import (
    CodeInterpreterContainerCodeInterpreterToolAuto,
# ... 29 lines omitted for brevity ...
)
from vllm.inputs import tokens_input
from vllm.outputs import CompletionOutput, RequestOutput
from vllm.sampling_params import SamplingParams
```
**EN:** Imports standard-library modules such as `contextlib.AsyncExitStack`, `unittest.mock.MagicMock`, third-party packages like `openai.types.responses.ResponseOutputItemDoneEvent`, `openai.types.responses.ResponseReasoningItem`, `openai.types.responses.ResponseReasoningTextDeltaEvent`, project helpers such as `vllm.entrypoints.mcp.tool_server.ToolServer`, `vllm.entrypoints.openai.engine.protocol.DeltaFunctionCall`, `vllm.entrypoints.openai.engine.protocol.DeltaMessage`.
**CN:** 导入标准库模块（如 `contextlib.AsyncExitStack`、`unittest.mock.MagicMock`）、第三方包（如 `openai.types.responses.ResponseOutputItemDoneEvent`、`openai.types.responses.ResponseReasoningItem`、`openai.types.responses.ResponseReasoningTextDeltaEvent`）、项目内辅助模块（如 `vllm.entrypoints.mcp.tool_server.ToolServer`、`vllm.entrypoints.openai.engine.protocol.DeltaFunctionCall`、`vllm.entrypoints.openai.engine.protocol.DeltaMessage`）。

### Class / 类: MockConversationContext (L57-L85)
```python
class MockConversationContext(ConversationContext):
    """Mock conversation context for testing"""

    def __init__(self):
        self.init_tool_sessions_called = False
        self.init_tool_sessions_args = None
        self.init_tool_sessions_kwargs = None

    def append_output(self, output) -> None:
        pass

    def append_tool_output(self, output) -> None:
        pass

# ... 7 lines omitted for brevity ...
        return []

    async def init_tool_sessions(self, tool_server, exit_stack, request_id, mcp_tools):
        self.init_tool_sessions_called = True
        self.init_tool_sessions_args = (tool_server, exit_stack, request_id, mcp_tools)

    async def cleanup_session(self) -> None:
        pass
```
**EN:** This class groups related scenarios in `MockConversationContext`. It contains 0 test method(s) and 8 supporting method(s). Representative methods include `__init__`, `append_output`.
**CN:** 该类将与 `MockConversationContext` 相关的场景组织在一起。 它包含 0 个测试方法和 8 个辅助方法。 代表性方法包括 `__init__`、`append_output`。

### Helper method / 辅助方法: MockConversationContext.__init__ (L60-L63)
```python
    def __init__(self):
        self.init_tool_sessions_called = False
        self.init_tool_sessions_args = None
        self.init_tool_sessions_kwargs = None
```
**EN:** This helper encapsulates reusable logic in `MockConversationContext.__init__`.
**CN:** 这个辅助函数将可复用逻辑封装在 `MockConversationContext.__init__` 中。

### Helper method / 辅助方法: MockConversationContext.append_output (L65-L66)
```python
    def append_output(self, output) -> None:
        pass
```
**EN:** This helper encapsulates reusable logic in `MockConversationContext.append_output`. Key inputs are `output`.
**CN:** 这个辅助函数将可复用逻辑封装在 `MockConversationContext.append_output` 中。 关键输入包括 `output`。

### Helper method / 辅助方法: MockConversationContext.append_tool_output (L68-L69)
```python
    def append_tool_output(self, output) -> None:
        pass
```
**EN:** This helper encapsulates reusable logic in `MockConversationContext.append_tool_output`. Key inputs are `output`.
**CN:** 这个辅助函数将可复用逻辑封装在 `MockConversationContext.append_tool_output` 中。 关键输入包括 `output`。

### Helper method / 辅助方法: MockConversationContext.call_tool (L71-L72)
```python
    async def call_tool(self):
        return []
```
**EN:** This async helper encapsulates reusable logic in `MockConversationContext.call_tool`. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `MockConversationContext.call_tool` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: MockConversationContext.need_builtin_tool_call (L74-L75)
```python
    def need_builtin_tool_call(self) -> bool:
        return False
```
**EN:** This helper encapsulates reusable logic in `MockConversationContext.need_builtin_tool_call`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `MockConversationContext.need_builtin_tool_call` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: MockConversationContext.render_for_completion (L77-L78)
```python
    def render_for_completion(self):
        return []
```
**EN:** This helper encapsulates reusable logic in `MockConversationContext.render_for_completion`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `MockConversationContext.render_for_completion` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: MockConversationContext.init_tool_sessions (L80-L82)
```python
    async def init_tool_sessions(self, tool_server, exit_stack, request_id, mcp_tools):
        self.init_tool_sessions_called = True
        self.init_tool_sessions_args = (tool_server, exit_stack, request_id, mcp_tools)
```
**EN:** This async helper encapsulates reusable logic in `MockConversationContext.init_tool_sessions`. Key inputs are `tool_server`, `exit_stack`, `request_id`, `mcp_tools`.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `MockConversationContext.init_tool_sessions` 中。 关键输入包括 `tool_server`、`exit_stack`、`request_id`、`mcp_tools`。

### Helper method / 辅助方法: MockConversationContext.cleanup_session (L84-L85)
```python
    async def cleanup_session(self) -> None:
        pass
```
**EN:** This async helper encapsulates reusable logic in `MockConversationContext.cleanup_session`.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `MockConversationContext.cleanup_session` 中。

### Test / 测试: test_serialize_message_pydantic_model_returns_dict (L88-L95)
```python
def test_serialize_message_pydantic_model_returns_dict() -> None:
    msg = ResponseRawMessageAndToken(message="hello", tokens=[1, 2, 3])

    serialized = serialize_message(msg)

    assert isinstance(serialized, dict)
    assert serialized["type"] == "raw_message_tokens"
    assert serialized["message"] == "hello"
```
**EN:** This test validates `test_serialize_message_pydantic_model_returns_dict`. The main assertion is `isinstance(serialized, dict)` and `serialized['type'] == 'raw_message_tokens'`.
**CN:** 这个测试验证 `test_serialize_message_pydantic_model_returns_dict`。 核心断言是 `isinstance(serialized, dict)` and `serialized['type'] == 'raw_message_tokens'`。

### Fixture / 夹具: mock_serving_responses (L98-L103)
```python
@pytest.fixture
def mock_serving_responses():
    """Create a mock OpenAIServingResponses instance"""
    serving_responses = MagicMock(spec=OpenAIServingResponses)
    serving_responses.tool_server = MagicMock(spec=ToolServer)
    return serving_responses
```
**EN:** This fixture prepares `mock_serving_responses` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `mock_serving_responses`。

### Fixture / 夹具: mock_context (L106-L109)
```python
@pytest.fixture
def mock_context():
    """Create a mock conversation context"""
    return MockConversationContext()
```
**EN:** This fixture prepares `mock_context` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `mock_context`。

### Fixture / 夹具: mock_exit_stack (L112-L115)
```python
@pytest.fixture
def mock_exit_stack():
    """Create a mock async exit stack"""
    return MagicMock(spec=AsyncExitStack)
```
**EN:** This fixture prepares `mock_exit_stack` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `mock_exit_stack`。

### Test / 测试: test_extract_tool_types (L118-L154)
```python
def test_extract_tool_types(monkeypatch: pytest.MonkeyPatch) -> None:
    tools: list[Tool] = []
    assert extract_tool_types(tools) == set()

    tools.append(LocalShell(type="local_shell"))
    assert extract_tool_types(tools) == {"local_shell"}

    tools.append(CodeInterpreterContainerCodeInterpreterToolAuto(type="auto"))
    assert extract_tool_types(tools) == {"local_shell", "auto"}

    tools.extend(
        [
            Mcp(type="mcp", server_label="random", server_url=""),
            Mcp(type="mcp", server_label="container", server_url=""),
            Mcp(type="mcp", server_label="code_interpreter", server_url=""),
            Mcp(type="mcp", server_label="web_search_preview", server_url=""),
        ]
    )
# ... 11 lines omitted for brevity ...
        "VLLM_GPT_OSS_SYSTEM_TOOL_MCP_LABELS", "code_interpreter,web_search_preview"
    )
    assert extract_tool_types(tools) == {
        "local_shell",
        "auto",
        "code_interpreter",
        "web_search_preview",
    }
```
**EN:** This test validates `test_extract_tool_types`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `extract_tool_types(tools) == set()` and `extract_tool_types(tools) == {'local_shell'}`.
**CN:** 这个测试验证 `test_extract_tool_types`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `extract_tool_types(tools) == set()` and `extract_tool_types(tools) == {'local_shell'}`。

### Test / 测试: test_response_created_event_uses_public_json_schema_alias (L157-L204)
```python
@pytest.mark.skip_global_cleanup
def test_response_created_event_uses_public_json_schema_alias() -> None:
    schema = {
        "type": "object",
        "properties": {
            "event_name": {"type": "string"},
            "date": {"type": "string"},
            "participants": {"type": "array", "items": {"type": "string"}},
        },
        "required": ["event_name", "date", "participants"],
        "additionalProperties": False,
    }
    text = ResponseTextConfig()
    text.format = ResponseFormatTextJSONSchemaConfig(
        type="json_schema",
        name="calendar_event",
        schema=schema,
        description="A calendar event.",
# ... 22 lines omitted for brevity ...
    event = ResponseCreatedEvent(
        type="response.created",
        sequence_number=0,
        response=initial_response,
    )
    assert event.response.text is not None
    assert event.response.text.format is not None
    assert event.response.text.format.model_dump(by_alias=True)["schema"] == schema
```
**EN:** This test validates `test_response_created_event_uses_public_json_schema_alias`. Relevant pytest markers include `skip_global_cleanup`. The main assertion is `fmt['schema'] == schema` and `'schema_' not in fmt`.
**CN:** 这个测试验证 `test_response_created_event_uses_public_json_schema_alias`。 相关的 pytest 标记包括 `skip_global_cleanup`。 核心断言是 `fmt['schema'] == schema` and `'schema_' not in fmt`。

### Class / 类: TestInitializeToolSessions (L207-L292)
```python
class TestInitializeToolSessions:
    """Test class for _initialize_tool_sessions method"""

    @pytest_asyncio.fixture
    async def serving_responses_instance(self):
        """Create a real OpenAIServingResponses instance for testing"""
        # Create minimal mocks for required dependencies
        engine_client = MagicMock()

        model_config = MagicMock()
        model_config.max_model_len = 100
        model_config.hf_config.model_type = "test"
        model_config.get_diff_sampling_param.return_value = {}
        engine_client.model_config = model_config
# ... 64 lines omitted for brevity ...
                    ],
                }
            ],
            previous_response_id="lol",
        )
        error = serving_responses_instance._validate_create_responses_input(request)
        assert error is not None
        assert error.error.type == "invalid_request_error"
```
**EN:** This class groups related scenarios in `TestInitializeToolSessions`. It contains 2 test method(s) and 1 supporting method(s). Representative methods include `test_initialize_tool_sessions`, `test_validate_create_responses_input`, `serving_responses_instance`.
**CN:** 该类将与 `TestInitializeToolSessions` 相关的场景组织在一起。 它包含 2 个测试方法和 1 个辅助方法。 代表性方法包括 `test_initialize_tool_sessions`、`test_validate_create_responses_input`、`serving_responses_instance`。

### Fixture method / 夹具方法: TestInitializeToolSessions.serving_responses_instance (L210-L240)
```python
    @pytest_asyncio.fixture
    async def serving_responses_instance(self):
        """Create a real OpenAIServingResponses instance for testing"""
        # Create minimal mocks for required dependencies
        engine_client = MagicMock()

        model_config = MagicMock()
        model_config.max_model_len = 100
        model_config.hf_config.model_type = "test"
        model_config.get_diff_sampling_param.return_value = {}
        engine_client.model_config = model_config

        engine_client.input_processor = MagicMock()
        engine_client.renderer = MagicMock()

        models = MagicMock()

        tool_server = MagicMock(spec=ToolServer)
# ... 5 lines omitted for brevity ...
            openai_serving_render=MagicMock(),
            request_logger=None,
            chat_template=None,
            chat_template_content_format="auto",
            tool_server=tool_server,
        )

        return instance
```
**EN:** This async fixture prepares `TestInitializeToolSessions.serving_responses_instance` for dependent tests.
**CN:** 这个异步 fixture 为依赖它的测试准备 `TestInitializeToolSessions.serving_responses_instance`。

### Test method / 测试方法: TestInitializeToolSessions.test_initialize_tool_sessions (L242-L270)
```python
    @pytest.mark.asyncio
    async def test_initialize_tool_sessions(
        self, serving_responses_instance, mock_context, mock_exit_stack
    ):
        """Test that method works correctly with only MCP tools"""

        request = ResponsesRequest(input="test input", tools=[])

        # Call the method
        await serving_responses_instance._initialize_tool_sessions(
            request, mock_context, mock_exit_stack
        )
        assert mock_context.init_tool_sessions_called is False

        # Create only MCP tools
        tools = [
            {"type": "web_search_preview"},
            {"type": "code_interpreter", "container": {"type": "auto"}},
        ]

        request = ResponsesRequest(input="test input", tools=tools)

        # Call the method
        await serving_responses_instance._initialize_tool_sessions(
            request, mock_context, mock_exit_stack
        )

        # Verify that init_tool_sessions was called
        assert mock_context.init_tool_sessions_called
```
**EN:** This async test validates `TestInitializeToolSessions.test_initialize_tool_sessions`. Relevant pytest markers include `asyncio`. Key inputs are `serving_responses_instance`, `mock_context`, `mock_exit_stack`. The main assertion is `mock_context.init_tool_sessions_called is False` and `mock_context.init_tool_sessions_called`.
**CN:** 这个异步测试验证 `TestInitializeToolSessions.test_initialize_tool_sessions`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `serving_responses_instance`、`mock_context`、`mock_exit_stack`。 核心断言是 `mock_context.init_tool_sessions_called is False` and `mock_context.init_tool_sessions_called`。

### Test method / 测试方法: TestInitializeToolSessions.test_validate_create_responses_input (L272-L292)
```python
    def test_validate_create_responses_input(
        self, serving_responses_instance, mock_context, mock_exit_stack
    ):
        request = ResponsesRequest(
            input="test input",
            previous_input_messages=[
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "text",
                            "text": "What is my horoscope? I am an Aquarius.",
                        }
                    ],
                }
            ],
            previous_response_id="lol",
        )
        error = serving_responses_instance._validate_create_responses_input(request)
        assert error is not None
        assert error.error.type == "invalid_request_error"
```
**EN:** This test validates `TestInitializeToolSessions.test_validate_create_responses_input`. Key inputs are `serving_responses_instance`, `mock_context`, `mock_exit_stack`. The main assertion is `error is not None` and `error.error.type == 'invalid_request_error'`.
**CN:** 这个测试验证 `TestInitializeToolSessions.test_validate_create_responses_input`。 关键输入包括 `serving_responses_instance`、`mock_context`、`mock_exit_stack`。 核心断言是 `error is not None` and `error.error.type == 'invalid_request_error'`。

### Class / 类: TestValidateGeneratorInput (L295-L348)
```python
class TestValidateGeneratorInput:
    """Test class for _validate_generator_input method"""

    @pytest_asyncio.fixture
    async def serving_responses_instance(self):
        """Create a real OpenAIServingResponses instance for testing"""
        # Create minimal mocks for required dependencies
        engine_client = MagicMock()

        model_config = MagicMock()
        model_config.max_model_len = 100
        model_config.hf_config.model_type = "test"
        model_config.get_diff_sampling_param.return_value = {}
        engine_client.model_config = model_config
# ... 32 lines omitted for brevity ...
        engine_input = tokens_input(invalid_prompt_token_ids)

        # Call the method
        result = serving_responses_instance._validate_generator_input(engine_input)

        # Should return an ErrorResponse
        assert result is not None
        assert isinstance(result, ErrorResponse)
```
**EN:** This class groups related scenarios in `TestValidateGeneratorInput`. It contains 1 test method(s) and 1 supporting method(s). Representative methods include `test_validate_generator_input`, `serving_responses_instance`.
**CN:** 该类将与 `TestValidateGeneratorInput` 相关的场景组织在一起。 它包含 1 个测试方法和 1 个辅助方法。 代表性方法包括 `test_validate_generator_input`、`serving_responses_instance`。

### Fixture method / 夹具方法: TestValidateGeneratorInput.serving_responses_instance (L298-L325)
```python
    @pytest_asyncio.fixture
    async def serving_responses_instance(self):
        """Create a real OpenAIServingResponses instance for testing"""
        # Create minimal mocks for required dependencies
        engine_client = MagicMock()

        model_config = MagicMock()
        model_config.max_model_len = 100
        model_config.hf_config.model_type = "test"
        model_config.get_diff_sampling_param.return_value = {}
        engine_client.model_config = model_config

        engine_client.input_processor = MagicMock()
        engine_client.renderer = MagicMock()

        models = MagicMock()

        # Create the actual instance
        instance = OpenAIServingResponses(
            engine_client=engine_client,
            models=models,
            openai_serving_render=MagicMock(),
            request_logger=None,
            chat_template=None,
            chat_template_content_format="auto",
        )

        return instance
```
**EN:** This async fixture prepares `TestValidateGeneratorInput.serving_responses_instance` for dependent tests.
**CN:** 这个异步 fixture 为依赖它的测试准备 `TestValidateGeneratorInput.serving_responses_instance`。

### Test method / 测试方法: TestValidateGeneratorInput.test_validate_generator_input (L327-L348)
```python
    def test_validate_generator_input(self, serving_responses_instance):
        """Test _validate_generator_input with valid prompt length"""
        # Create an engine prompt with valid length (less than max_model_len)
        valid_prompt_token_ids = list(range(5))  # 5 tokens < 100 max_model_len
        engine_input = tokens_input(valid_prompt_token_ids)

        # Call the method
        result = serving_responses_instance._validate_generator_input(engine_input)

        # Should return None for valid input
        assert result is None

        # create an invalid engine prompt
        invalid_prompt_token_ids = list(range(200))  # 100 tokens >= 100 max_model_len
        engine_input = tokens_input(invalid_prompt_token_ids)

        # Call the method
        result = serving_responses_instance._validate_generator_input(engine_input)

        # Should return an ErrorResponse
        assert result is not None
        assert isinstance(result, ErrorResponse)
```
**EN:** This test validates `TestValidateGeneratorInput.test_validate_generator_input`. Key inputs are `serving_responses_instance`. The main assertion is `result is None` and `result is not None`.
**CN:** 这个测试验证 `TestValidateGeneratorInput.test_validate_generator_input`。 关键输入包括 `serving_responses_instance`。 核心断言是 `result is None` and `result is not None`。

### Test / 测试: test_reasoning_tokens_counted_for_text_reasoning_model (L351-L429)
```python
@pytest.mark.asyncio
async def test_reasoning_tokens_counted_for_text_reasoning_model(monkeypatch):
    """Ensure reasoning_tokens usage is derived from thinking token spans."""

    class FakeTokenizer:
        def __init__(self):
            self._vocab = {"<think>": 1, "</think>": 2, "reason": 3, "final": 4}

        def get_vocab(self):
            return self._vocab

    # Force non-harmony, SimpleContext path
    monkeypatch.setattr(envs, "VLLM_USE_EXPERIMENTAL_PARSER_CONTEXT", False)

    engine_client = MagicMock()
    model_config = MagicMock()
    model_config.hf_config.model_type = "test"
    model_config.hf_text_config = MagicMock()
# ... 53 lines omitted for brevity ...
        result_generator=dummy_result_generator(),
        context=context,
        model_name="test-model",
        tokenizer=tokenizer,
        request_metadata=metadata,
    )

    assert response.usage.output_tokens_details.reasoning_tokens == 1
```
**EN:** This async test validates `test_reasoning_tokens_counted_for_text_reasoning_model`. Relevant pytest markers include `asyncio`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `response.usage.output_tokens_details.reasoning_tokens == 1`.
**CN:** 这个异步测试验证 `test_reasoning_tokens_counted_for_text_reasoning_model`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `response.usage.output_tokens_details.reasoning_tokens == 1`。

### Class / 类: TestExtractAllowedToolsFromMcpRequests (L432-L524)
```python
class TestExtractAllowedToolsFromMcpRequests:
    """Test class for _extract_allowed_tools_from_mcp_requests function"""

    def test_extract_allowed_tools_basic_formats(self):
        """Test extraction with list format, object format, and None."""
        from openai.types.responses.tool import McpAllowedToolsMcpToolFilter

        tools = [
            # List format
            Mcp(
                type="mcp",
                server_label="server1",
                allowed_tools=["tool1", "tool2"],
            ),
# ... 71 lines omitted for brevity ...
            ),
        ]
        result = _extract_allowed_tools_from_mcp_requests(tools)
        # Non-MCP tools should be ignored
        assert result == {
            "server1": ["tool1"],
            "server2": ["tool2"],
        }
```
**EN:** This class groups related scenarios in `TestExtractAllowedToolsFromMcpRequests`. It contains 3 test method(s) and 0 supporting method(s). Representative methods include `test_extract_allowed_tools_basic_formats`, `test_extract_allowed_tools_star_normalization`, `test_extract_allowed_tools_filters_non_mcp`.
**CN:** 该类将与 `TestExtractAllowedToolsFromMcpRequests` 相关的场景组织在一起。 它包含 3 个测试方法和 0 个辅助方法。 代表性方法包括 `test_extract_allowed_tools_basic_formats`、`test_extract_allowed_tools_star_normalization`、`test_extract_allowed_tools_filters_non_mcp`。

### Test method / 测试方法: TestExtractAllowedToolsFromMcpRequests.test_extract_allowed_tools_basic_formats (L435-L466)
```python
    def test_extract_allowed_tools_basic_formats(self):
        """Test extraction with list format, object format, and None."""
        from openai.types.responses.tool import McpAllowedToolsMcpToolFilter

        tools = [
            # List format
            Mcp(
                type="mcp",
                server_label="server1",
                allowed_tools=["tool1", "tool2"],
            ),
            # Object format
            Mcp(
                type="mcp",
                server_label="server2",
                allowed_tools=McpAllowedToolsMcpToolFilter(
                    tool_names=["tool3", "tool4"]
                ),
# ... 6 lines omitted for brevity ...
            ),
        ]
        result = _extract_allowed_tools_from_mcp_requests(tools)
        assert result == {
            "server1": ["tool1", "tool2"],
            "server2": ["tool3", "tool4"],
            "server3": None,
        }
```
**EN:** This test validates `TestExtractAllowedToolsFromMcpRequests.test_extract_allowed_tools_basic_formats`. The main assertion is `result == {'server1': ['tool1', 'tool2'], 'server2': ['tool3', 'tool4'], 'server3': None}`.
**CN:** 这个测试验证 `TestExtractAllowedToolsFromMcpRequests.test_extract_allowed_tools_basic_formats`。 核心断言是 `result == {'server1': ['tool1', 'tool2'], 'server2': ['tool3', 'tool4'], 'server3': None}`。

### Test method / 测试方法: TestExtractAllowedToolsFromMcpRequests.test_extract_allowed_tools_star_normalization (L468-L502)
```python
    def test_extract_allowed_tools_star_normalization(self):
        """Test that '*' wildcard is normalized to None (select all tools).

        This is the key test requested by reviewers to explicitly demonstrate
        that the "*" select-all scenario is handled correctly.
        """
        from openai.types.responses.tool import McpAllowedToolsMcpToolFilter

        tools = [
            # Star in list format
            Mcp(
                type="mcp",
                server_label="server1",
                allowed_tools=["*"],
            ),
            # Star mixed with other tools in list
            Mcp(
                type="mcp",
# ... 9 lines omitted for brevity ...
        ]
        result = _extract_allowed_tools_from_mcp_requests(tools)
        # All should be normalized to None (allows all tools)
        assert result == {
            "server1": None,
            "server2": None,
            "server3": None,
        }
```
**EN:** This test validates `TestExtractAllowedToolsFromMcpRequests.test_extract_allowed_tools_star_normalization`. The main assertion is `result == {'server1': None, 'server2': None, 'server3': None}`.
**CN:** 这个测试验证 `TestExtractAllowedToolsFromMcpRequests.test_extract_allowed_tools_star_normalization`。 核心断言是 `result == {'server1': None, 'server2': None, 'server3': None}`。

### Test method / 测试方法: TestExtractAllowedToolsFromMcpRequests.test_extract_allowed_tools_filters_non_mcp (L504-L524)
```python
    def test_extract_allowed_tools_filters_non_mcp(self):
        """Test that non-MCP tools are ignored during extraction."""
        tools = [
            Mcp(
                type="mcp",
                server_label="server1",
                allowed_tools=["tool1"],
            ),
            LocalShell(type="local_shell"),  # Non-MCP tool should be ignored
            Mcp(
                type="mcp",
                server_label="server2",
                allowed_tools=["tool2"],
            ),
        ]
        result = _extract_allowed_tools_from_mcp_requests(tools)
        # Non-MCP tools should be ignored
        assert result == {
            "server1": ["tool1"],
            "server2": ["tool2"],
        }
```
**EN:** This test validates `TestExtractAllowedToolsFromMcpRequests.test_extract_allowed_tools_filters_non_mcp`. The main assertion is `result == {'server1': ['tool1'], 'server2': ['tool2']}`.
**CN:** 这个测试验证 `TestExtractAllowedToolsFromMcpRequests.test_extract_allowed_tools_filters_non_mcp`。 核心断言是 `result == {'server1': ['tool1'], 'server2': ['tool2']}`。

### Class / 类: TestHarmonyPreambleStreaming (L527-L636)
```python
class TestHarmonyPreambleStreaming:
    """Tests for preamble (commentary with no recipient) streaming events."""

    @staticmethod
    def _make_ctx(*, channel, recipient, delta="hello"):
        """Build a lightweight mock StreamingHarmonyContext."""
        ctx = MagicMock()
        ctx.last_content_delta = delta
        ctx.parser.current_channel = channel
        ctx.parser.current_recipient = recipient
        return ctx

    @staticmethod
    def _make_previous_item(*, channel, recipient, text="preamble text"):
# ... 88 lines omitted for brevity ...
        )
        state = StreamingState()
        state.current_item_id = "fc_test"

        events = emit_previous_item_done_events(previous, state)

        type_names = [e.type for e in events]
        assert "response.output_text.done" not in type_names
```
**EN:** This class groups related scenarios in `TestHarmonyPreambleStreaming`. It contains 5 test method(s) and 2 supporting method(s). Representative methods include `test_preamble_delta_emits_text_events`, `test_preamble_delta_second_token_no_added`, `test_commentary_with_function_recipient_not_preamble`, `_make_ctx`, `_make_previous_item`.
**CN:** 该类将与 `TestHarmonyPreambleStreaming` 相关的场景组织在一起。 它包含 5 个测试方法和 2 个辅助方法。 代表性方法包括 `test_preamble_delta_emits_text_events`、`test_preamble_delta_second_token_no_added`、`test_commentary_with_function_recipient_not_preamble`、`_make_ctx`、`_make_previous_item`。

### Helper method / 辅助方法: TestHarmonyPreambleStreaming._make_ctx (L530-L537)
```python
    @staticmethod
    def _make_ctx(*, channel, recipient, delta="hello"):
        """Build a lightweight mock StreamingHarmonyContext."""
        ctx = MagicMock()
        ctx.last_content_delta = delta
        ctx.parser.current_channel = channel
        ctx.parser.current_recipient = recipient
        return ctx
```
**EN:** This helper encapsulates reusable logic in `TestHarmonyPreambleStreaming._make_ctx`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestHarmonyPreambleStreaming._make_ctx` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: TestHarmonyPreambleStreaming._make_previous_item (L539-L548)
```python
    @staticmethod
    def _make_previous_item(*, channel, recipient, text="preamble text"):
        """Build a lightweight mock previous_item (openai_harmony Message)."""
        content_part = MagicMock()
        content_part.text = text
        item = MagicMock()
        item.channel = channel
        item.recipient = recipient
        item.content = [content_part]
        return item
```
**EN:** This helper encapsulates reusable logic in `TestHarmonyPreambleStreaming._make_previous_item`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestHarmonyPreambleStreaming._make_previous_item` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Test method / 测试方法: TestHarmonyPreambleStreaming.test_preamble_delta_emits_text_events (L550-L563)
```python
    def test_preamble_delta_emits_text_events(self) -> None:
        """commentary + recipient=None should emit output_text.delta events."""
        from vllm.entrypoints.openai.responses.streaming_events import (
            emit_content_delta_events,
        )

        ctx = self._make_ctx(channel="commentary", recipient=None)
        state = StreamingState()

        events = emit_content_delta_events(ctx, state)

        type_names = [e.type for e in events]
        assert "response.output_text.delta" in type_names
        assert "response.output_item.added" in type_names
```
**EN:** This test validates `TestHarmonyPreambleStreaming.test_preamble_delta_emits_text_events`. The main assertion is `'response.output_text.delta' in type_names` and `'response.output_item.added' in type_names`.
**CN:** 这个测试验证 `TestHarmonyPreambleStreaming.test_preamble_delta_emits_text_events`。 核心断言是 `'response.output_text.delta' in type_names` and `'response.output_item.added' in type_names`。

### Test method / 测试方法: TestHarmonyPreambleStreaming.test_preamble_delta_second_token_no_added (L565-L581)
```python
    def test_preamble_delta_second_token_no_added(self) -> None:
        """Second preamble token should emit delta only, not added again."""
        from vllm.entrypoints.openai.responses.streaming_events import (
            emit_content_delta_events,
        )

        ctx = self._make_ctx(channel="commentary", recipient=None, delta="w")
        state = StreamingState()
        state.sent_output_item_added = True
        state.current_item_id = "msg_test"
        state.current_content_index = 0

        events = emit_content_delta_events(ctx, state)

        type_names = [e.type for e in events]
        assert "response.output_text.delta" in type_names
        assert "response.output_item.added" not in type_names
```
**EN:** This test validates `TestHarmonyPreambleStreaming.test_preamble_delta_second_token_no_added`. The main assertion is `'response.output_text.delta' in type_names` and `'response.output_item.added' not in type_names`.
**CN:** 这个测试验证 `TestHarmonyPreambleStreaming.test_preamble_delta_second_token_no_added`。 核心断言是 `'response.output_text.delta' in type_names` and `'response.output_item.added' not in type_names`。

### Test method / 测试方法: TestHarmonyPreambleStreaming.test_commentary_with_function_recipient_not_preamble (L583-L598)
```python
    def test_commentary_with_function_recipient_not_preamble(self) -> None:
        """commentary + recipient='functions.X' must NOT use preamble path."""
        from vllm.entrypoints.openai.responses.streaming_events import (
            emit_content_delta_events,
        )

        ctx = self._make_ctx(
            channel="commentary",
            recipient="functions.get_weather",
        )
        state = StreamingState()

        events = emit_content_delta_events(ctx, state)

        type_names = [e.type for e in events]
        assert "response.output_text.delta" not in type_names
```
**EN:** This test validates `TestHarmonyPreambleStreaming.test_commentary_with_function_recipient_not_preamble`. The main assertion is `'response.output_text.delta' not in type_names`.
**CN:** 这个测试验证 `TestHarmonyPreambleStreaming.test_commentary_with_function_recipient_not_preamble`。 核心断言是 `'response.output_text.delta' not in type_names`。

### Test method / 测试方法: TestHarmonyPreambleStreaming.test_preamble_done_emits_text_done_events (L600-L618)
```python
    def test_preamble_done_emits_text_done_events(self) -> None:
        """Completed preamble should emit text done + content_part done +
        output_item done, same shape as final channel."""
        from vllm.entrypoints.openai.responses.streaming_events import (
            emit_previous_item_done_events,
        )

        previous = self._make_previous_item(channel="commentary", recipient=None)
        state = StreamingState()
        state.current_item_id = "msg_test"
        state.current_output_index = 0
        state.current_content_index = 0

        events = emit_previous_item_done_events(previous, state)

        type_names = [e.type for e in events]
        assert "response.output_text.done" in type_names
        assert "response.content_part.done" in type_names
        assert "response.output_item.done" in type_names
```
**EN:** This test validates `TestHarmonyPreambleStreaming.test_preamble_done_emits_text_done_events`. The main assertion is `'response.output_text.done' in type_names` and `'response.content_part.done' in type_names`.
**CN:** 这个测试验证 `TestHarmonyPreambleStreaming.test_preamble_done_emits_text_done_events`。 核心断言是 `'response.output_text.done' in type_names` and `'response.content_part.done' in type_names`。

### Test method / 测试方法: TestHarmonyPreambleStreaming.test_commentary_with_recipient_no_preamble_done (L620-L636)
```python
    def test_commentary_with_recipient_no_preamble_done(self) -> None:
        """commentary + recipient='functions.X' should route to function call
        done, not preamble done."""
        from vllm.entrypoints.openai.responses.streaming_events import (
            emit_previous_item_done_events,
        )

        previous = self._make_previous_item(
            channel="commentary", recipient="functions.get_weather"
        )
        state = StreamingState()
        state.current_item_id = "fc_test"

        events = emit_previous_item_done_events(previous, state)

        type_names = [e.type for e in events]
        assert "response.output_text.done" not in type_names
```
**EN:** This test validates `TestHarmonyPreambleStreaming.test_commentary_with_recipient_no_preamble_done`. The main assertion is `'response.output_text.done' not in type_names`.
**CN:** 这个测试验证 `TestHarmonyPreambleStreaming.test_commentary_with_recipient_no_preamble_done`。 核心断言是 `'response.output_text.done' not in type_names`。

### Helper / 辅助函数: _make_simple_context_with_output (L639-L661)
```python
def _make_simple_context_with_output(text, token_ids):
    """Create a SimpleContext with a RequestOutput containing the given text."""
    ctx = SimpleContext()
    completion = CompletionOutput(
        index=0,
        text=text,
        token_ids=token_ids,
        cumulative_logprob=0.0,
        logprobs=None,
        finish_reason=None,
        stop_reason=None,
    )
    req_output = RequestOutput(
        request_id="req",
        prompt="hi",
        prompt_token_ids=[7, 8],
        prompt_logprobs=None,
        outputs=[completion],
        finished=False,
        num_cached_tokens=0,
    )
    ctx.append_output(req_output)
    return ctx
```
**EN:** This helper encapsulates reusable logic in `_make_simple_context_with_output`. Key inputs are `text`, `token_ids`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_simple_context_with_output` 中。 关键输入包括 `text`、`token_ids`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _make_serving_instance_with_reasoning (L664-L687)
```python
def _make_serving_instance_with_reasoning():
    """Create an OpenAIServingResponses with a mocked reasoning parser."""
    engine_client = MagicMock()
    model_config = MagicMock()
    model_config.max_model_len = 100
    model_config.hf_config.model_type = "test"
    model_config.hf_text_config = MagicMock()
    model_config.get_diff_sampling_param.return_value = {}
    engine_client.model_config = model_config
    engine_client.input_processor = MagicMock()
    engine_client.renderer = MagicMock()

    models = MagicMock()

    serving = OpenAIServingResponses(
        engine_client=engine_client,
        models=models,
        openai_serving_render=MagicMock(),
        request_logger=None,
        chat_template=None,
        chat_template_content_format="auto",
        reasoning_parser="qwen3",
    )
    return serving
```
**EN:** This helper encapsulates reusable logic in `_make_serving_instance_with_reasoning`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_serving_instance_with_reasoning` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _identity_increment (L690-L696)
```python
def _identity_increment(event):
    """Simple identity callable for _increment_sequence_number_and_return."""
    seq = getattr(_identity_increment, "_counter", 0)
    if hasattr(event, "sequence_number"):
        event.sequence_number = seq
    _identity_increment._counter = seq + 1  # type: ignore
    return event
```
**EN:** This helper encapsulates reusable logic in `_identity_increment`. Key inputs are `event`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_identity_increment` 中。 关键输入包括 `event`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _mock_parser_with_reasoning (L699-L721)
```python
def _mock_parser_with_reasoning(serving, delta_sequence: list[DeltaMessage]):
    """Set up serving.parser so that it returns a mock parser instance
    with a reasoning parser that returns the given delta_sequence.

    The mock has reasoning_parser set (truthy) but tool_parser as None,
    so the parser's parse_delta enters the reasoning-only branch.
    """
    call_count = 0

    def mock_parse_delta(**kwargs):
        nonlocal call_count
        if call_count >= len(delta_sequence):
            return None
        result = delta_sequence[call_count]
        call_count += 1
        return result

    mock_parser_instance = MagicMock()
    mock_parser_instance.reasoning_parser = MagicMock()  # truthy
    mock_parser_instance.tool_parser = None
    mock_parser_instance.parse_delta = mock_parse_delta
    mock_parser_instance.is_reasoning_end = MagicMock(return_value=False)
    serving.parser = MagicMock(return_value=mock_parser_instance)
```
**EN:** This helper encapsulates reusable logic in `_mock_parser_with_reasoning`. Key inputs are `serving`, `delta_sequence`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_mock_parser_with_reasoning` 中。 关键输入包括 `serving`、`delta_sequence`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: TestStreamingReasoningToContentTransition (L724-L932)
```python
class TestStreamingReasoningToContentTransition:
    """Tests for _process_simple_streaming_events reasoning-to-content
    transition, specifically the fix for mixed deltas that carry both
    reasoning and content simultaneously."""

    @pytest.mark.asyncio
    async def test_mixed_delta_reasoning_and_content_emits_reasoning_delta(
        self, monkeypatch
    ):
        """When the reasoning parser produces a delta with both reasoning
        and content set (e.g. reasoning end and content start in the same
        chunk), the trailing reasoning text must be emitted as a
        ResponseReasoningTextDeltaEvent and included in the
        ResponseReasoningTextDoneEvent text."""
# ... 187 lines omitted for brevity ...
        assert len(text_deltas) == 0

        # Final item should be a reasoning item
        item_done_events = [
            e for e in events if isinstance(e, ResponseOutputItemDoneEvent)
        ]
        assert len(item_done_events) == 1
        assert isinstance(item_done_events[0].item, ResponseReasoningItem)
```
**EN:** This class groups related scenarios in `TestStreamingReasoningToContentTransition`. It contains 3 test method(s) and 0 supporting method(s). Representative methods include `test_mixed_delta_reasoning_and_content_emits_reasoning_delta`, `test_transition_without_mixed_delta_no_extra_reasoning_event`, `test_reasoning_only_stream_no_content`.
**CN:** 该类将与 `TestStreamingReasoningToContentTransition` 相关的场景组织在一起。 它包含 3 个测试方法和 0 个辅助方法。 代表性方法包括 `test_mixed_delta_reasoning_and_content_emits_reasoning_delta`、`test_transition_without_mixed_delta_no_extra_reasoning_event`、`test_reasoning_only_stream_no_content`。

### Test method / 测试方法: TestStreamingReasoningToContentTransition.test_mixed_delta_reasoning_and_content_emits_reasoning_delta (L729-L800)
```python
    @pytest.mark.asyncio
    async def test_mixed_delta_reasoning_and_content_emits_reasoning_delta(
        self, monkeypatch
    ):
        """When the reasoning parser produces a delta with both reasoning
        and content set (e.g. reasoning end and content start in the same
        chunk), the trailing reasoning text must be emitted as a
        ResponseReasoningTextDeltaEvent and included in the
        ResponseReasoningTextDoneEvent text."""

        monkeypatch.setattr(envs, "VLLM_USE_EXPERIMENTAL_PARSER_CONTEXT", False)
        serving = _make_serving_instance_with_reasoning()

        # Sequence of DeltaMessages the mock orchestrator will return
        delta_sequence = [
            DeltaMessage(reasoning="thinking..."),
            DeltaMessage(reasoning=" end", content="hello"),  # mixed delta
            DeltaMessage(content=" world"),
# ... 46 lines omitted for brevity ...
        assert reasoning_done[0].text == "thinking... end"

        # Content deltas should be emitted for both the mixed delta's
        # content and the pure content delta
        text_deltas = [e for e in events if isinstance(e, ResponseTextDeltaEvent)]
        assert len(text_deltas) == 2
        assert text_deltas[0].delta == "hello"
        assert text_deltas[1].delta == " world"
```
**EN:** This async test validates `TestStreamingReasoningToContentTransition.test_mixed_delta_reasoning_and_content_emits_reasoning_delta`. Relevant pytest markers include `asyncio`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `len(reasoning_deltas) == 2` and `reasoning_deltas[0].delta == 'thinking...'`.
**CN:** 这个异步测试验证 `TestStreamingReasoningToContentTransition.test_mixed_delta_reasoning_and_content_emits_reasoning_delta`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `len(reasoning_deltas) == 2` and `reasoning_deltas[0].delta == 'thinking...'`。

### Test method / 测试方法: TestStreamingReasoningToContentTransition.test_transition_without_mixed_delta_no_extra_reasoning_event (L802-L863)
```python
    @pytest.mark.asyncio
    async def test_transition_without_mixed_delta_no_extra_reasoning_event(
        self, monkeypatch
    ):
        """When the transition from reasoning to content is clean (no mixed
        delta), no extra reasoning delta event should be emitted."""

        monkeypatch.setattr(envs, "VLLM_USE_EXPERIMENTAL_PARSER_CONTEXT", False)
        serving = _make_serving_instance_with_reasoning()

        delta_sequence = [
            DeltaMessage(reasoning="thinking"),
            DeltaMessage(content="answer"),
        ]
        _mock_parser_with_reasoning(serving, delta_sequence)

        contexts = [
            _make_simple_context_with_output("chunk1", [10]),
# ... 36 lines omitted for brevity ...
        ]
        assert len(reasoning_done) == 1
        assert reasoning_done[0].text == "thinking"

        # One content delta
        text_deltas = [e for e in events if isinstance(e, ResponseTextDeltaEvent)]
        assert len(text_deltas) == 1
        assert text_deltas[0].delta == "answer"
```
**EN:** This async test validates `TestStreamingReasoningToContentTransition.test_transition_without_mixed_delta_no_extra_reasoning_event`. Relevant pytest markers include `asyncio`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `len(reasoning_deltas) == 1` and `reasoning_deltas[0].delta == 'thinking'`.
**CN:** 这个异步测试验证 `TestStreamingReasoningToContentTransition.test_transition_without_mixed_delta_no_extra_reasoning_event`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `len(reasoning_deltas) == 1` and `reasoning_deltas[0].delta == 'thinking'`。

### Test method / 测试方法: TestStreamingReasoningToContentTransition.test_reasoning_only_stream_no_content (L865-L932)
```python
    @pytest.mark.asyncio
    async def test_reasoning_only_stream_no_content(self, monkeypatch):
        """When the stream has only reasoning deltas and no content, the
        reasoning done event should be emitted at finalization with the
        full accumulated text, and no text delta events should appear."""

        monkeypatch.setattr(envs, "VLLM_USE_EXPERIMENTAL_PARSER_CONTEXT", False)
        serving = _make_serving_instance_with_reasoning()

        delta_sequence = [
            DeltaMessage(reasoning="step 1"),
            DeltaMessage(reasoning=" step 2"),
        ]
        _mock_parser_with_reasoning(serving, delta_sequence)

        contexts = [
            _make_simple_context_with_output("chunk1", [10]),
            _make_simple_context_with_output("chunk2", [20]),
# ... 42 lines omitted for brevity ...
        assert len(text_deltas) == 0

        # Final item should be a reasoning item
        item_done_events = [
            e for e in events if isinstance(e, ResponseOutputItemDoneEvent)
        ]
        assert len(item_done_events) == 1
        assert isinstance(item_done_events[0].item, ResponseReasoningItem)
```
**EN:** This async test validates `TestStreamingReasoningToContentTransition.test_reasoning_only_stream_no_content`. Relevant pytest markers include `asyncio`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `len(reasoning_deltas) == 2` and `reasoning_deltas[0].delta == 'step 1'`.
**CN:** 这个异步测试验证 `TestStreamingReasoningToContentTransition.test_reasoning_only_stream_no_content`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `len(reasoning_deltas) == 2` and `reasoning_deltas[0].delta == 'step 1'`。

### Class / 类: TestAutoToolStreaming (L935-L1126)
```python
class TestAutoToolStreaming:
    @staticmethod
    async def _collect_events(delta_sequence: list[DeltaMessage]):
        serving = _make_serving_instance_with_reasoning()
        _mock_parser_with_reasoning(serving, delta_sequence)

        contexts = [
            _make_simple_context_with_output("chunk", [i])
            for i in range(len(delta_sequence))
        ]

        async def result_generator():
            for ctx in contexts:
                yield ctx
# ... 170 lines omitted for brevity ...
        assert function_items[0].item.name == "get_weather"

        argument_deltas = [
            event.delta
            for event in events
            if event.type == "response.function_call_arguments.delta"
        ]
        assert "".join(argument_deltas) == '{"location":"Berlin"}'
```
**EN:** This class groups related scenarios in `TestAutoToolStreaming`. It contains 2 test method(s) and 1 supporting method(s). Representative methods include `test_auto_multi_tool_streaming_opens_one_item_per_tool`, `test_auto_tool_choice_first_delta_tool_call_does_not_duplicate_item`, `_collect_events`.
**CN:** 该类将与 `TestAutoToolStreaming` 相关的场景组织在一起。 它包含 2 个测试方法和 1 个辅助方法。 代表性方法包括 `test_auto_multi_tool_streaming_opens_one_item_per_tool`、`test_auto_tool_choice_first_delta_tool_call_does_not_duplicate_item`、`_collect_events`。

### Helper method / 辅助方法: TestAutoToolStreaming._collect_events (L936-L985)
```python
    @staticmethod
    async def _collect_events(delta_sequence: list[DeltaMessage]):
        serving = _make_serving_instance_with_reasoning()
        _mock_parser_with_reasoning(serving, delta_sequence)

        contexts = [
            _make_simple_context_with_output("chunk", [i])
            for i in range(len(delta_sequence))
        ]

        async def result_generator():
            for ctx in contexts:
                yield ctx

        request = ResponsesRequest(
            input="hi",
            tools=[
                {
# ... 24 lines omitted for brevity ...
            model_name="test-model",
            tokenizer=MagicMock(),
            request_metadata=metadata,
            created_time=0,
            _increment_sequence_number_and_return=_identity_increment,
        ):
            events.append(event)
        return events
```
**EN:** This async helper encapsulates reusable logic in `TestAutoToolStreaming._collect_events`. Key inputs are `delta_sequence`. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `TestAutoToolStreaming._collect_events` 中。 关键输入包括 `delta_sequence`。 它把计算得到的状态或辅助对象返回给调用方。

### Test method / 测试方法: TestAutoToolStreaming.test_auto_multi_tool_streaming_opens_one_item_per_tool (L987-L1076)
```python
    @pytest.mark.skip_global_cleanup
    @pytest.mark.asyncio
    async def test_auto_multi_tool_streaming_opens_one_item_per_tool(self, monkeypatch):
        monkeypatch.setattr(envs, "VLLM_USE_EXPERIMENTAL_PARSER_CONTEXT", False)

        delta_sequence = [
            DeltaMessage(
                tool_calls=[
                    DeltaToolCall(
                        id="call_vienna",
                        type="function",
                        index=0,
                        function=DeltaFunctionCall(
                            name="get_weather",
                            arguments="",
                        ),
                    )
                ]
# ... 64 lines omitted for brevity ...
            if event.type == "response.output_item.done"
            and getattr(event.item, "type", None) == "function_call"
        ]
        assert [event.item.arguments for event in function_done] == [
            '{"location":"Vienna"}',
            '{"location":"Berlin"}',
        ]
        assert [event.output_index for event in function_done] == [0, 1]
```
**EN:** This async test validates `TestAutoToolStreaming.test_auto_multi_tool_streaming_opens_one_item_per_tool`. Relevant pytest markers include `skip_global_cleanup`, `asyncio`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `len(function_items) == 2` and `[event.item.name for event in function_items] == ['get_weather', 'get_weather']`.
**CN:** 这个异步测试验证 `TestAutoToolStreaming.test_auto_multi_tool_streaming_opens_one_item_per_tool`。 相关的 pytest 标记包括 `skip_global_cleanup`、`asyncio`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `len(function_items) == 2` and `[event.item.name for event in function_items] == ['get_weather', 'get_weather']`。

### Test method / 测试方法: TestAutoToolStreaming.test_auto_tool_choice_first_delta_tool_call_does_not_duplicate_item (L1078-L1126)
```python
    @pytest.mark.skip_global_cleanup
    @pytest.mark.asyncio
    async def test_auto_tool_choice_first_delta_tool_call_does_not_duplicate_item(
        self, monkeypatch
    ):
        monkeypatch.setattr(envs, "VLLM_USE_EXPERIMENTAL_PARSER_CONTEXT", False)

        delta_sequence = [
            DeltaMessage(
                tool_calls=[
                    DeltaToolCall(
                        id="call_test",
                        type="function",
                        index=0,
                        function=DeltaFunctionCall(
                            name="get_weather",
                            arguments="",
                        ),
# ... 23 lines omitted for brevity ...
        assert function_items[0].item.name == "get_weather"

        argument_deltas = [
            event.delta
            for event in events
            if event.type == "response.function_call_arguments.delta"
        ]
        assert "".join(argument_deltas) == '{"location":"Berlin"}'
```
**EN:** This async test validates `TestAutoToolStreaming.test_auto_tool_choice_first_delta_tool_call_does_not_duplicate_item`. Relevant pytest markers include `skip_global_cleanup`, `asyncio`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `len(function_items) == 1` and `function_items[0].item.name == 'get_weather'`.
**CN:** 这个异步测试验证 `TestAutoToolStreaming.test_auto_tool_choice_first_delta_tool_call_does_not_duplicate_item`。 相关的 pytest 标记包括 `skip_global_cleanup`、`asyncio`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `len(function_items) == 1` and `function_items[0].item.name == 'get_weather'`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `contextlib.AsyncExitStack`, `unittest.mock.MagicMock`
- **Third-party / 第三方**: `openai.types.responses.ResponseOutputItemDoneEvent`, `openai.types.responses.ResponseReasoningItem`, `openai.types.responses.ResponseReasoningTextDeltaEvent`, `openai.types.responses.ResponseReasoningTextDoneEvent`, `openai.types.responses.ResponseTextConfig`, `openai.types.responses.ResponseTextDeltaEvent`, `openai.types.responses.response_format_text_json_schema_config.ResponseFormatTextJSONSchemaConfig`, `openai.types.responses.tool.CodeInterpreterContainerCodeInterpreterToolAuto`, `openai.types.responses.tool.LocalShell`, `openai.types.responses.tool.Mcp`, `openai.types.responses.tool.McpAllowedToolsMcpToolFilter`, `openai.types.responses.tool.Tool`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `vllm.entrypoints.mcp.tool_server.ToolServer`, `vllm.entrypoints.openai.engine.protocol.DeltaFunctionCall`, `vllm.entrypoints.openai.engine.protocol.DeltaMessage`, `vllm.entrypoints.openai.engine.protocol.DeltaToolCall`, `vllm.entrypoints.openai.engine.protocol.ErrorResponse`, `vllm.entrypoints.openai.engine.protocol.RequestResponseMetadata`, `vllm.entrypoints.openai.responses.context.ConversationContext`, `vllm.entrypoints.openai.responses.context.SimpleContext`, `vllm.entrypoints.openai.responses.protocol.ResponseCreatedEvent`, `vllm.entrypoints.openai.responses.protocol.ResponseRawMessageAndToken`, `vllm.entrypoints.openai.responses.protocol.ResponsesRequest`, `vllm.entrypoints.openai.responses.protocol.ResponsesResponse`, `vllm.entrypoints.openai.responses.protocol.serialize_message`, `vllm.entrypoints.openai.responses.serving.OpenAIServingResponses`, `vllm.entrypoints.openai.responses.serving._extract_allowed_tools_from_mcp_requests`, `vllm.entrypoints.openai.responses.serving.extract_tool_types`, `vllm.entrypoints.openai.responses.streaming_events.StreamingState`, `vllm.entrypoints.openai.responses.streaming_events.emit_content_delta_events`, `vllm.entrypoints.openai.responses.streaming_events.emit_previous_item_done_events`, `vllm.envs`, `vllm.inputs.tokens_input`, `vllm.outputs.CompletionOutput`, `vllm.outputs.RequestOutput`, `vllm.sampling_params.SamplingParams`
