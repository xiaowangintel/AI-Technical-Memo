# tool_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/tool_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements tool server logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 工具 服务器 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6: Import runtime dependencies / 导入运行时依赖
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import logging
from abc import ABC, abstractmethod
from contextlib import AbstractAsyncContextManager, asynccontextmanager
from typing import Any
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-13: Provide supporting module logic / 提供辅助模块逻辑
```python
try:
    from mcp import ClientSession
    from mcp.client.sse import sse_client
    from mcp.types import ListToolsResult
except ImportError as e:
    ClientSession = sse_client = ListToolsResult = e
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 15-15: Import runtime dependencies / 导入运行时依赖
```python
from openai_harmony import ToolDescription, ToolNamespaceConfig
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 17-17: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 20-28: Implement async list server and tools / 实现异步list 服务器 and 工具
```python
async def list_server_and_tools(server_url: str):

    async with (
        sse_client(url=server_url) as streams,
        ClientSession(*streams) as session,
    ):
        initialize_response = await session.initialize()
        list_tools_response = await session.list_tools()
        return initialize_response, list_tools_response
```
**EN:** This block implements the async function `list_server_and_tools(server_url)`. It focuses on handling the tool server responsibilities represented by `list_server_and_tools`, providing reusable behavior for the tool server pipeline.
**CN:** 该代码块实现异步函数 `list_server_and_tools(server_url)`。它围绕 `list_server_and_tools` 所承担的 工具 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 31-53: Implement trim schema / 实现trim schema
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
**EN:** This block implements the function `trim_schema(schema)`. It focuses on handling the tool server responsibilities represented by `trim_schema`, providing reusable behavior for the tool server pipeline.
**CN:** 该代码块实现函数 `trim_schema(schema)`。它围绕 `trim_schema` 所承担的 工具 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 56-71: Implement post process tools description / 实现post process 工具 description
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
**EN:** This block implements the function `post_process_tools_description(list_tools_result)`. It focuses on handling the tool server responsibilities represented by `post_process_tools_description`, providing reusable behavior for the tool server pipeline.
**CN:** 该代码块实现函数 `post_process_tools_description(list_tools_result)`。它围绕 `post_process_tools_description` 所承担的 工具 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 74-85: Define class ToolServer / 定义类 ToolServer
```python
class ToolServer(ABC):

    @abstractmethod
    def has_tool(self, tool_name: str):
        pass

    @abstractmethod
    def get_tool_description(self, tool_name: str):
        pass

    @abstractmethod
    def get_tool_session(self, tool_name: str) -> AbstractAsyncContextManager[Any]: ...
```
**EN:** This block declares the class `ToolServer`. It centers on coordinating tool server behavior, with methods such as has_tool, get_tool_description, get_tool_session.
**CN:** 该代码块声明类 `ToolServer`。它负责承载与 工具 服务器 相关的核心状态与行为，并通过 has_tool, get_tool_description, get_tool_session 等方法组织实现。

### Lines 88-142: Define class MCPToolServer / 定义类 MCPToolServer
```python
class MCPToolServer(ToolServer):

    def __init__(self):
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
                        parameters=tool.inputSchema,
                    )
                    for tool in list_tools_response.tools
                ],
            )
            self.harmony_tool_descriptions[tool_from_mcp.name] = tool_from_mcp
            if tool_from_mcp.name not in self.urls:
                self.urls[tool_from_mcp.name] = url
            else:
                logger.warning(
                    "Tool %s already exists. Ignoring duplicate tool server %s",
                    tool_from_mcp.name,
                    url,
                )

    def has_tool(self, tool_name: str):
        return tool_name in self.harmony_tool_descriptions

    def get_tool_description(self, tool_name: str):
        return self.harmony_tool_descriptions.get(tool_name)

    @asynccontextmanager
    async def get_tool_session(self, tool_name: str):
        url = self.urls.get(tool_name)
        if url:
            async with (
                sse_client(url=url) as streams,
                ClientSession(*streams) as session,
            ):
                await session.initialize()
                yield session
        else:
            logger.warning("Tool %s not found", tool_name)
```
**EN:** This block declares the class `MCPToolServer`. It centers on coordinating tool server behavior, with methods such as __init__, add_tool_server, has_tool, get_tool_description, get_tool_session.
**CN:** 该代码块声明类 `MCPToolServer`。它负责承载与 工具 服务器 相关的核心状态与行为，并通过 __init__, add_tool_server, has_tool, get_tool_description, get_tool_session 等方法组织实现。

### Lines 145-177: Define class DemoToolServer / 定义类 DemoToolServer
```python
class DemoToolServer(ToolServer):

    def __init__(self):
        from sglang.srt.entrypoints.tool import (
            HarmonyBrowserTool,
            HarmonyPythonTool,
            Tool,
        )

        self.tools: dict[str, Tool] = {}
        browser_tool = HarmonyBrowserTool()
        if browser_tool.enabled:
            self.tools["browser"] = browser_tool
        python_tool = HarmonyPythonTool()
        if python_tool.enabled:
            self.tools["python"] = python_tool

    def has_tool(self, tool_name: str):
        return tool_name in self.tools

    def get_tool_description(self, tool_name: str):
        if tool_name not in self.tools:
            return None
        if tool_name == "browser":
            return ToolNamespaceConfig.browser()
        elif tool_name == "python":
            return ToolNamespaceConfig.python()
        else:
            raise ValueError(f"Unknown tool {tool_name}")

    @asynccontextmanager
    async def get_tool_session(self, tool_name: str):
        yield self.tools[tool_name]
```
**EN:** This block declares the class `DemoToolServer`. It centers on coordinating tool server behavior, with methods such as __init__, has_tool, get_tool_description, get_tool_session.
**CN:** 该代码块声明类 `DemoToolServer`。它负责承载与 工具 服务器 相关的核心状态与行为，并通过 __init__, has_tool, get_tool_description, get_tool_session 等方法组织实现。

## Key Concepts / 关键概念
- **Core types / 核心类型**: ToolServer, MCPToolServer, DemoToolServer
- **Main callables / 主要可调用对象**: list_server_and_tools, trim_schema, post_process_tools_description
- **Domain focus / 领域焦点**: tool server / 工具 服务器
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: contextlib, logging, typing
- **Third-party / 第三方库**: abc, mcp, mcp.client.sse, mcp.types, openai_harmony
- **Local Modules / 本地模块**: sglang.srt.entrypoints.tool
