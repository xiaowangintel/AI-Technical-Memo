# tool_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/mcp/tool_server.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Starts the MCP tool-serving process and request loop. Scope: MCP. / 启动 MCP 工具服务进程与请求循环。 作用域：MCP。

## Line-by-Line Analysis / 逐行分析
### Lines 3-10 — Imports and shared dependencies
```python
from abc import ABC, abstractmethod
from contextlib import AbstractAsyncContextManager, asynccontextmanager
from typing import TYPE_CHECKING, Any

from openai_harmony import ToolDescription, ToolNamespaceConfig

from vllm.entrypoints.mcp.tool import HarmonyBrowserTool, HarmonyPythonTool, Tool
from vllm.logger import init_logger
```
**EN:** This import block pulls in standard-library modules such as `abc`, `contextlib`, `typing`, uses third-party packages like `openai_harmony`, depends on internal helpers such as `vllm.entrypoints.mcp.tool`, `vllm.logger`.
**CN:** 该导入块引入 `abc`, `contextlib`, `typing` 等标准库模块，使用 `openai_harmony` 等第三方库，依赖 `vllm.entrypoints.mcp.tool`, `vllm.logger` 等 vLLM 内部模块。

### Lines 12-12 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 14-15 — Conditional top-level flow
```python
if TYPE_CHECKING:
    from mcp.types import ListToolsResult
```
**EN:** This top-level conditional toggles behavior based on runtime configuration or platform state.
**CN:** 该顶层条件分支会根据运行时配置或平台状态切换行为。

### Lines 18-28 — Function `list_server_and_tools`
```python
async def list_server_and_tools(server_url: str):
    from mcp import ClientSession
    from mcp.client.sse import sse_client

    async with (
        sse_client(url=server_url) as streams,
        ClientSession(*streams) as session,
    ):
        initialize_response = await session.initialize()
        list_tools_response = await session.list_tools()
        return initialize_response, list_tools_response
```
**EN:** This async function `list_server_and_tools` starts or coordinates serving.
**CN:** 该异步函数 `list_server_and_tools` 启动或协调服务流程。

### Lines 31-53 — Function `trim_schema`
```python
def trim_schema(schema: dict) -> dict:
    # Turn JSON Schema from MCP generated into Harmony's variant.
    if "title" in schema:
        del schema["title"]
    if "default" in schema and schema["default"] is None:
        del schema["default"]
    if "anyOf" in schema:
        # Turn "anyOf": [{"type": "type-1"}, {"type": "type-2"}]
        # into "type": ["type-1", "type-2"]
        # if there's more than 1 types, also remove "null" type as Harmony will
        # just ignore it
        types = [
            type_dict["type"]
            for type_dict in schema["anyOf"]
            if type_dict["type"] != "null"
        ]
        schema["type"] = types
        del schema["anyOf"]
    if "properties" in schema:
        schema["properties"] = {
            k: trim_schema(v) for k, v in schema["properties"].items()
        }
    return schema
```
**EN:** This function `trim_schema` implements the `trim schema` step within the module flow.
**CN:** 该函数 `trim_schema` 实现了模块流程中的“trimschema”步骤。

### Lines 56-71 — Function `post_process_tools_description`
```python
def post_process_tools_description(
    list_tools_result: "ListToolsResult",
) -> "ListToolsResult":
    # Adapt the MCP tool result for Harmony
    for tool in list_tools_result.tools:
        tool.inputSchema = trim_schema(tool.inputSchema)

    # Some tools schema don't need to be part of the prompt (e.g. simple text
    # in text out for Python)
    list_tools_result.tools = [
        tool
        for tool in list_tools_result.tools
        if getattr(tool.annotations, "include_in_prompt", True)
    ]

    return list_tools_result
```
**EN:** This function `post_process_tools_description` implements the `post processes tools description` step within the module flow.
**CN:** 该函数 `post_process_tools_description` 实现了模块流程中的“后处理toolsdescription”步骤。

### Lines 74-99 — Class `ToolServer`
```python
class ToolServer(ABC):
    @abstractmethod
    def has_tool(self, tool_name: str) -> bool:
        """
        Return True if the tool is supported, False otherwise.
        """
        pass

    @abstractmethod
    def get_tool_description(
        self, tool_name: str, allowed_tools: list[str] | None = None
    ) -> ToolNamespaceConfig | None:
        """
        Return the tool description for the given tool name.
        If the tool is not supported, return None.
        """
        pass

    @abstractmethod
    def new_session(
        self, tool_name: str, session_id: str, headers: dict[str, str] | None = None
    ) -> AbstractAsyncContextManager[Any]:
        """
        Create a session for the tool.
        """
        ...
```
**EN:** Class `ToolServer` is defined here, extending `ABC`, and groups behavior through methods like `has_tool`, `get_tool_description`, `new_session`.
**CN:** 这里定义类 `ToolServer`，其职责是封装相关状态与行为，并通过 `has_tool`、`get_tool_description`、`new_session` 等方法组织逻辑。

### Lines 102-194 — Class `MCPToolServer`
```python
class MCPToolServer(ToolServer):
    def __init__(self):
        try:
            import mcp  # noqa: F401
        except ImportError:
            raise ImportError(
                "mcp is not installed. Please run `pip install mcp` to use "
                "MCPToolServer."
            ) from None
        self.harmony_tool_descriptions = {}

    async def add_tool_server(self, server_url: str):
        tool_urls = server_url.split(",")
        self.harmony_tool_descriptions = {}
        self.urls: dict[str, str] = {}
        for url in tool_urls:
            url = f"http://{url}/sse"
            initialize_response, list_tools_response = await list_server_and_tools(url)

            list_tools_response = post_process_tools_description(list_tools_response)

            tool_from_mcp = ToolNamespaceConfig(
                name=initialize_response.serverInfo.name,
                description=initialize_response.instructions,
                tools=[
                    ToolDescription.new(
                        name=tool.name,
                        description=tool.description,
...
        if not url:
            raise KeyError(f"Tool '{tool_name}' is not supported")
        async with (
            sse_client(url=url, headers=request_headers) as streams,
            ClientSession(*streams) as session,
        ):
            await session.initialize()
            yield session
```
**EN:** Class `MCPToolServer` is defined here, extending `ToolServer`, and groups behavior through methods like `__init__`, `add_tool_server`, `has_tool`, `get_tool_description`.
**CN:** 这里定义类 `MCPToolServer`，其职责是封装相关状态与行为，并通过 `__init__`、`add_tool_server`、`has_tool`、`get_tool_description` 等方法组织逻辑。

### Lines 197-234 — Class `DemoToolServer`
```python
class DemoToolServer(ToolServer):
    def __init__(self):
        self.tools: dict[str, Tool] = {}

    async def init_and_validate(self):
        browser_tool = HarmonyBrowserTool()
        python_tool = HarmonyPythonTool()
        await python_tool.validate()
        if browser_tool.enabled:
            self.tools["browser"] = browser_tool
        if python_tool.enabled:
            self.tools["python"] = python_tool
        logger.info(
            "DemoToolServer initialized with tools: %s", list(self.tools.keys())
        )

    def has_tool(self, tool_name: str) -> bool:
        return tool_name in self.tools

    def get_tool_description(
        self, tool_name: str, allowed_tools: list[str] | None = None
    ) -> ToolNamespaceConfig | None:
        if tool_name not in self.tools:
            return None
        if tool_name == "browser":
            return ToolNamespaceConfig.browser()
        elif tool_name == "python":
            return ToolNamespaceConfig.python()
        else:
            raise ValueError(f"Unknown tool {tool_name}")

    @asynccontextmanager
    async def new_session(
        self, tool_name: str, session_id: str, headers: dict[str, str] | None = None
    ):
        if tool_name not in self.tools:
            raise KeyError(f"Tool '{tool_name}' is not supported")
        yield self.tools[tool_name]
```
**EN:** Class `DemoToolServer` is defined here, extending `ToolServer`, and groups behavior through methods like `__init__`, `init_and_validate`, `has_tool`, `get_tool_description`.
**CN:** 这里定义类 `DemoToolServer`，其职责是封装相关状态与行为，并通过 `__init__`、`init_and_validate`、`has_tool`、`get_tool_description` 等方法组织逻辑。

## Key Concepts / 关键概念
- Async request handling / 异步请求处理
- Incremental result emission / 增量结果输出
- MCP tool integration / MCP 工具集成
- MCP tool exposure / MCP 工具暴露

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `abc`, `contextlib`, `typing`
- **Third-party / 第三方**: `openai_harmony`, `mcp`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.mcp.tool`, `vllm.logger`
