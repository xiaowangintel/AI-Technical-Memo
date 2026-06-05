# test_mcp_tools.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_mcp_tools.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 5 test(s), 2 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 5 个测试、2 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L22)
```python
from __future__ import annotations

import pytest
import pytest_asyncio
from openai import OpenAI
from openai_harmony import Message, ToolDescription, ToolNamespaceConfig

from tests.utils import RemoteOpenAIServer
from vllm.entrypoints.mcp.tool_server import MCPToolServer

from .conftest import (
    BASE_TEST_ENV,
    events_contain_type,
    log_response_diagnostics,
    retry_for_tool_call,
    retry_streaming_for,
    validate_streaming_event_stack,
)
```
**EN:** Imports standard-library modules such as `__future__.annotations`, third-party packages like `openai.OpenAI`, `openai_harmony.Message`, `openai_harmony.ToolDescription`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.mcp.tool_server.MCPToolServer`, `vllm.entrypoints.openai.parser.harmony_utils.BUILTIN_TOOL_TO_MCP_SERVER_LABEL`.
**CN:** 导入标准库模块（如 `__future__.annotations`）、第三方包（如 `openai.OpenAI`、`openai_harmony.Message`、`openai_harmony.ToolDescription`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.entrypoints.mcp.tool_server.MCPToolServer`、`vllm.entrypoints.openai.parser.harmony_utils.BUILTIN_TOOL_TO_MCP_SERVER_LABEL`）。

### Module setup / 模块级配置: MODEL_NAME, _BASE_SERVER_ARGS, _PYTHON_TOOL_INSTRUCTION (L24-L36)
```python
MODEL_NAME = "openai/gpt-oss-20b"

_BASE_SERVER_ARGS = [
    "--enforce-eager",
    "--tool-server",
    "demo",
    "--max_model_len",
    "5000",
]

_PYTHON_TOOL_INSTRUCTION = (
    "You must use the Python tool to execute code. Never simulate execution."
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `_BASE_SERVER_ARGS`, `_PYTHON_TOOL_INSTRUCTION`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`_BASE_SERVER_ARGS`、`_PYTHON_TOOL_INSTRUCTION`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: TestMCPToolServerUnit (L39-L110)
```python
class TestMCPToolServerUnit:
    """Test MCPToolServer.get_tool_description filtering logic.

    Note: The wildcard "*" is normalized to None by
    _extract_allowed_tools_from_mcp_requests before reaching this layer,
    so we only test None and specific tool filtering here.
    See responses/test_serving_responses.py for "*" normalization tests.
    """

    def test_get_tool_description(self):
        pytest.importorskip("mcp")

        server = MCPToolServer()
        tool1 = ToolDescription.new(
# ... 50 lines omitted for brevity ...
            MCP_BUILTIN_TOOLS,
        )

        assert set(BUILTIN_TOOL_TO_MCP_SERVER_LABEL.values()) == MCP_BUILTIN_TOOLS, (
            f"MCP_BUILTIN_TOOLS {MCP_BUILTIN_TOOLS} does not match "
            f"BUILTIN_TOOL_TO_MCP_SERVER_LABEL values "
            f"{set(BUILTIN_TOOL_TO_MCP_SERVER_LABEL.values())}"
        )
```
**EN:** This class groups related scenarios in `TestMCPToolServerUnit`. It contains 2 test method(s) and 0 supporting method(s). Representative methods include `test_get_tool_description`, `test_builtin_tools_consistency`.
**CN:** 该类将与 `TestMCPToolServerUnit` 相关的场景组织在一起。 它包含 2 个测试方法和 0 个辅助方法。 代表性方法包括 `test_get_tool_description`、`test_builtin_tools_consistency`。

### Test method / 测试方法: TestMCPToolServerUnit.test_get_tool_description (L48-L97)
```python
    def test_get_tool_description(self):
        pytest.importorskip("mcp")

        server = MCPToolServer()
        tool1 = ToolDescription.new(
            name="tool1", description="First", parameters={"type": "object"}
        )
        tool2 = ToolDescription.new(
            name="tool2", description="Second", parameters={"type": "object"}
        )
        tool3 = ToolDescription.new(
            name="tool3", description="Third", parameters={"type": "object"}
        )

        server.harmony_tool_descriptions = {
            "test_server": ToolNamespaceConfig(
                name="test_server",
                description="test",
# ... 24 lines omitted for brevity ...
        # No matching tools - returns None
        result = server.get_tool_description(
            "test_server", allowed_tools=["nonexistent"]
        )
        assert result is None

        # Empty list - returns None
        assert server.get_tool_description("test_server", allowed_tools=[]) is None
```
**EN:** This test validates `TestMCPToolServerUnit.test_get_tool_description`. The main assertion is `server.get_tool_description('nonexistent') is None` and `len(result.tools) == 3`.
**CN:** 这个测试验证 `TestMCPToolServerUnit.test_get_tool_description`。 核心断言是 `server.get_tool_description('nonexistent') is None` and `len(result.tools) == 3`。

### Test method / 测试方法: TestMCPToolServerUnit.test_builtin_tools_consistency (L99-L110)
```python
    def test_builtin_tools_consistency(self):
        """MCP_BUILTIN_TOOLS must match BUILTIN_TOOL_TO_MCP_SERVER_LABEL values."""
        from vllm.entrypoints.openai.parser.harmony_utils import (
            BUILTIN_TOOL_TO_MCP_SERVER_LABEL,
            MCP_BUILTIN_TOOLS,
        )

        assert set(BUILTIN_TOOL_TO_MCP_SERVER_LABEL.values()) == MCP_BUILTIN_TOOLS, (
            f"MCP_BUILTIN_TOOLS {MCP_BUILTIN_TOOLS} does not match "
            f"BUILTIN_TOOL_TO_MCP_SERVER_LABEL values "
            f"{set(BUILTIN_TOOL_TO_MCP_SERVER_LABEL.values())}"
        )
```
**EN:** This test validates `TestMCPToolServerUnit.test_builtin_tools_consistency`. The main assertion is `set(BUILTIN_TOOL_TO_MCP_SERVER_LABEL.values()) == MCP_BUILTIN_TOOLS`.
**CN:** 这个测试验证 `TestMCPToolServerUnit.test_builtin_tools_consistency`。 核心断言是 `set(BUILTIN_TOOL_TO_MCP_SERVER_LABEL.values()) == MCP_BUILTIN_TOOLS`。

### Class / 类: TestMCPEnabled (L113-L243)
```python
class TestMCPEnabled:
    """Tests that require MCP tools to be enabled via environment variable."""

    @pytest.fixture(scope="class")
    def mcp_enabled_server(self):
        env_dict = {
            **BASE_TEST_ENV,
            "VLLM_ENABLE_RESPONSES_API_STORE": "1",
            "PYTHON_EXECUTION_BACKEND": "dangerously_use_uv",
            "VLLM_GPT_OSS_SYSTEM_TOOL_MCP_LABELS": ("code_interpreter,container"),
            "VLLM_GPT_OSS_HARMONY_SYSTEM_INSTRUCTIONS": "1",
        }
        with RemoteOpenAIServer(
            MODEL_NAME, list(_BASE_SERVER_ARGS), env_dict=env_dict
# ... 109 lines omitted for brevity ...
            validate_events=_has_mcp_events,
            input=("What is 123 * 456? Use Python to calculate the result."),
            tools=[{"type": "mcp", "server_label": "code_interpreter"}],
            instructions=_PYTHON_TOOL_INSTRUCTION,
            temperature=0.0,
        )

        validate_streaming_event_stack(events, pairs_of_event_types)
```
**EN:** This class groups related scenarios in `TestMCPEnabled`. It contains 3 test method(s) and 4 supporting method(s). Representative methods include `test_mcp_tool_env_flag_enabled`, `test_mcp_tool_with_allowed_tools_star`, `test_mcp_tool_calling_streaming_types`, `mcp_enabled_server`, `client`.
**CN:** 该类将与 `TestMCPEnabled` 相关的场景组织在一起。 它包含 3 个测试方法和 4 个辅助方法。 代表性方法包括 `test_mcp_tool_env_flag_enabled`、`test_mcp_tool_with_allowed_tools_star`、`test_mcp_tool_calling_streaming_types`、`mcp_enabled_server`、`client`。

### Fixture method / 夹具方法: TestMCPEnabled.mcp_enabled_server (L116-L128)
```python
    @pytest.fixture(scope="class")
    def mcp_enabled_server(self):
        env_dict = {
            **BASE_TEST_ENV,
            "VLLM_ENABLE_RESPONSES_API_STORE": "1",
            "PYTHON_EXECUTION_BACKEND": "dangerously_use_uv",
            "VLLM_GPT_OSS_SYSTEM_TOOL_MCP_LABELS": ("code_interpreter,container"),
            "VLLM_GPT_OSS_HARMONY_SYSTEM_INSTRUCTIONS": "1",
        }
        with RemoteOpenAIServer(
            MODEL_NAME, list(_BASE_SERVER_ARGS), env_dict=env_dict
        ) as remote_server:
            yield remote_server
```
**EN:** This fixture prepares `TestMCPEnabled.mcp_enabled_server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `TestMCPEnabled.mcp_enabled_server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture method / 夹具方法: TestMCPEnabled.client (L130-L133)
```python
    @pytest_asyncio.fixture
    async def client(self, mcp_enabled_server):
        async with mcp_enabled_server.get_async_client() as async_client:
            yield async_client
```
**EN:** This async fixture prepares `TestMCPEnabled.client` for dependent tests. Key inputs are `mcp_enabled_server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `TestMCPEnabled.client`。 关键输入包括 `mcp_enabled_server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Helper method / 辅助方法: TestMCPEnabled._mcp_tools_payload (L135-L144)
```python
    @staticmethod
    def _mcp_tools_payload(*, allowed_tools: list[str] | None = None) -> list[dict]:
        tool: dict = {
            "type": "mcp",
            "server_label": "code_interpreter",
            "server_url": "http://localhost:8888",
        }
        if allowed_tools is not None:
            tool["allowed_tools"] = allowed_tools
        return [tool]
```
**EN:** This helper encapsulates reusable logic in `TestMCPEnabled._mcp_tools_payload`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestMCPEnabled._mcp_tools_payload` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: TestMCPEnabled._python_exec_input (L146-L150)
```python
    @staticmethod
    def _python_exec_input(code: str = "") -> str:
        if not code:
            code = "import random; print(random.randint(1, 1000000))"
        return f"Execute the following code: {code}"
```
**EN:** This helper encapsulates reusable logic in `TestMCPEnabled._python_exec_input`. Key inputs are `code`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestMCPEnabled._python_exec_input` 中。 关键输入包括 `code`。 它把计算得到的状态或辅助对象返回给调用方。

### Test method / 测试方法: TestMCPEnabled.test_mcp_tool_env_flag_enabled (L152-L191)
```python
    @pytest.mark.asyncio
    @pytest.mark.parametrize("model_name", [MODEL_NAME])
    async def test_mcp_tool_env_flag_enabled(self, client: OpenAI, model_name: str):
        response = await retry_for_tool_call(
            client,
            model=model_name,
            expected_tool_type="mcp_call",
            input=self._python_exec_input(),
            instructions=_PYTHON_TOOL_INSTRUCTION,
            tools=self._mcp_tools_payload(),
            temperature=0.0,
            extra_body={"enable_response_messages": True},
        )

        assert response.status == "completed"
        log_response_diagnostics(response, label="MCP Enabled")

        tool_call_found = False
# ... 14 lines omitted for brevity ...
            f"No Python tool call found. "
            f"Output types: "
            f"{[getattr(o, 'type', None) for o in response.output]}"
        )
        assert tool_response_found, "No Python tool response found"

        for message in response.input_messages:
            assert Message.from_dict(message).author.role != "developer"
```
**EN:** This async test validates `TestMCPEnabled.test_mcp_tool_env_flag_enabled`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. The main assertion is `response.status == 'completed'` and `tool_call_found`.
**CN:** 这个异步测试验证 `TestMCPEnabled.test_mcp_tool_env_flag_enabled`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 核心断言是 `response.status == 'completed'` and `tool_call_found`。

### Test method / 测试方法: TestMCPEnabled.test_mcp_tool_with_allowed_tools_star (L193-L220)
```python
    @pytest.mark.asyncio
    @pytest.mark.parametrize("model_name", [MODEL_NAME])
    async def test_mcp_tool_with_allowed_tools_star(
        self, client: OpenAI, model_name: str
    ):
        response = await retry_for_tool_call(
            client,
            model=model_name,
            expected_tool_type="mcp_call",
            input=self._python_exec_input(),
            instructions=_PYTHON_TOOL_INSTRUCTION,
            tools=self._mcp_tools_payload(allowed_tools=["*"]),
            temperature=0.0,
            extra_body={"enable_response_messages": True},
        )

        assert response.status == "completed"
        log_response_diagnostics(response, label="MCP Allowed Tools *")

        tool_call_found = any(
            (msg.get("recipient") or "").startswith("python")
            for msg in response.output_messages
        )
        assert tool_call_found, (
            f"No Python tool call with '*'. "
            f"Output types: "
            f"{[getattr(o, 'type', None) for o in response.output]}"
        )
```
**EN:** This async test validates `TestMCPEnabled.test_mcp_tool_with_allowed_tools_star`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. The main assertion is `response.status == 'completed'` and `tool_call_found`.
**CN:** 这个异步测试验证 `TestMCPEnabled.test_mcp_tool_with_allowed_tools_star`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 核心断言是 `response.status == 'completed'` and `tool_call_found`。

### Test method / 测试方法: TestMCPEnabled.test_mcp_tool_calling_streaming_types (L222-L243)
```python
    @pytest.mark.asyncio
    @pytest.mark.parametrize("model_name", [MODEL_NAME])
    async def test_mcp_tool_calling_streaming_types(
        self,
        pairs_of_event_types: dict[str, str],
        client: OpenAI,
        model_name: str,
    ):
        def _has_mcp_events(events: list) -> bool:
            return events_contain_type(events, "mcp_call")

        events = await retry_streaming_for(
            client,
            model=model_name,
            validate_events=_has_mcp_events,
            input=("What is 123 * 456? Use Python to calculate the result."),
            tools=[{"type": "mcp", "server_label": "code_interpreter"}],
            instructions=_PYTHON_TOOL_INSTRUCTION,
            temperature=0.0,
        )

        validate_streaming_event_stack(events, pairs_of_event_types)
```
**EN:** This async test validates `TestMCPEnabled.test_mcp_tool_calling_streaming_types`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `pairs_of_event_types`, `client`, `model_name`.
**CN:** 这个异步测试验证 `TestMCPEnabled.test_mcp_tool_calling_streaming_types`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `pairs_of_event_types`、`client`、`model_name`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `__future__.annotations`
- **Third-party / 第三方**: `openai.OpenAI`, `openai_harmony.Message`, `openai_harmony.ToolDescription`, `openai_harmony.ToolNamespaceConfig`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.mcp.tool_server.MCPToolServer`, `vllm.entrypoints.openai.parser.harmony_utils.BUILTIN_TOOL_TO_MCP_SERVER_LABEL`, `vllm.entrypoints.openai.parser.harmony_utils.MCP_BUILTIN_TOOLS`
- **Local relative imports / 本地相对导入**: `.conftest.BASE_TEST_ENV`, `.conftest.events_contain_type`, `.conftest.log_response_diagnostics`, `.conftest.retry_for_tool_call`, `.conftest.retry_streaming_for`, `.conftest.validate_streaming_event_stack`
