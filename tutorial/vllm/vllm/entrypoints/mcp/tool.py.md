# tool.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/mcp/tool.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines MCP-facing tool objects or adapters. Scope: MCP. / 定义面向 MCP 的工具对象或适配器。 作用域：MCP。

## Line-by-Line Analysis / 逐行分析
### Lines 3-14 — Imports and shared dependencies
```python
import json
import os
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING, Any

from openai.types.responses.response_function_tool_call_output_item import (
    ResponseFunctionToolCallOutputItem,
)
from openai_harmony import Author, Message, Role, TextContent

from vllm.logger import init_logger
from vllm.utils import random_uuid
```
**EN:** This import block pulls in standard-library modules such as `json`, `os`, `abc`, `typing`, uses third-party packages like `openai`, `openai_harmony`, depends on internal helpers such as `vllm.logger`, `vllm.utils`.
**CN:** 该导入块引入 `json`, `os`, `abc`, `typing` 等标准库模块，使用 `openai`, `openai_harmony` 等第三方库，依赖 `vllm.logger`, `vllm.utils` 等 vLLM 内部模块。

### Lines 16-18 — Conditional top-level flow
```python
if TYPE_CHECKING:
    # Avoid circular import.
    from vllm.entrypoints.openai.responses.context import ConversationContext
```
**EN:** This top-level conditional toggles behavior based on runtime configuration or platform state.
**CN:** 该顶层条件分支会根据运行时配置或平台状态切换行为。

### Lines 20-22 — Module constants
```python
logger = init_logger(__name__)

MIN_GPT_OSS_VERSION = "0.0.7"
```
**EN:** This block initializes `logger`, `MIN_GPT_OSS_VERSION`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `MIN_GPT_OSS_VERSION`，为后续逻辑准备模块级常量或共享状态。

### Lines 25-46 — Function `validate_gpt_oss_install`
```python
def validate_gpt_oss_install():
    """
    Check if the gpt-oss is installed and its version is at least 0.0.7.
    If not, raise an ImportError.
    """
    from importlib.metadata import PackageNotFoundError, version

    from packaging.version import InvalidVersion, Version

    try:
        pkg_version_str = version("gpt_oss")
        pkg_version = Version(pkg_version_str)
    except PackageNotFoundError:
        raise ImportError("Package 'gpt_oss' is not installed.") from None
    except InvalidVersion as e:
        raise ImportError(f"Invalid version string for 'gpt_oss': {e}") from None

    if pkg_version < Version(MIN_GPT_OSS_VERSION):
        raise ImportError(
            f"gpt_oss >= {MIN_GPT_OSS_VERSION} is required, "
            f"but {pkg_version} is installed."
        ) from None
```
**EN:** This function `validate_gpt_oss_install` is documented as: Check if the gpt-oss is installed and its version is at least 0.0.7.
**CN:** 这里定义函数 `validate_gpt_oss_install`，其文档字符串说明了主要职责与调用约定。

### Lines 49-56 — Class `Tool`
```python
class Tool(ABC):
    @abstractmethod
    async def get_result(self, context: "ConversationContext") -> Any:
        pass

    @abstractmethod
    async def get_result_parsable_context(self, context: "ConversationContext") -> Any:
        pass
```
**EN:** Class `Tool` is defined here, extending `ABC`, and groups behavior through methods like `get_result`, `get_result_parsable_context`.
**CN:** 这里定义类 `Tool`，其职责是封装相关状态与行为，并通过 `get_result`、`get_result_parsable_context` 等方法组织逻辑。

### Lines 59-98 — Class `HarmonyBrowserTool`
```python
class HarmonyBrowserTool(Tool):
    def __init__(self):
        self.enabled = True
        exa_api_key = os.getenv("EXA_API_KEY")
        if not exa_api_key:
            self.enabled = False
            logger.warning_once("EXA_API_KEY is not set, browsing is disabled")
            return

        try:
            validate_gpt_oss_install()
            from gpt_oss.tools.simple_browser import SimpleBrowserTool
            from gpt_oss.tools.simple_browser.backend import ExaBackend
        except ImportError as e:
            self.enabled = False
            logger.warning_once(
                "gpt_oss is not installed properly (%s), browsing is disabled", e
            )
            return

        browser_backend = ExaBackend(source="web", api_key=exa_api_key)
        self.browser_tool = SimpleBrowserTool(backend=browser_backend)
        logger.info_once("Browser tool initialized")

    async def get_result(self, context: "ConversationContext") -> Any:
        from vllm.entrypoints.openai.responses.context import HarmonyContext

        assert isinstance(context, HarmonyContext)
        last_msg = context.messages[-1]
        tool_output_msgs = []
        async for msg in self.browser_tool.process(last_msg):
            tool_output_msgs.append(msg)
        return tool_output_msgs

    async def get_result_parsable_context(self, context: "ConversationContext") -> Any:
        raise NotImplementedError("Not implemented yet")

    @property
    def tool_config(self) -> Any:
        return self.browser_tool.tool_config
```
**EN:** Class `HarmonyBrowserTool` is defined here, extending `Tool`, and groups behavior through methods like `__init__`, `get_result`, `get_result_parsable_context`, `tool_config`.
**CN:** 这里定义类 `HarmonyBrowserTool`，其职责是封装相关状态与行为，并通过 `__init__`、`get_result`、`get_result_parsable_context`、`tool_config` 等方法组织逻辑。

### Lines 101-187 — Class `HarmonyPythonTool`
```python
class HarmonyPythonTool(Tool):
    def __init__(self):
        self.enabled = True

        try:
            validate_gpt_oss_install()
            from gpt_oss.tools.python_docker.docker_tool import PythonTool
        except ImportError as e:
            self.enabled = False
            logger.warning_once(
                "gpt_oss is not installed properly (%s), code interpreter is disabled",
                e,
            )
            return

        self.python_tool = PythonTool()

    async def validate(self):
        if not self.enabled:
            return
        try:
            message = Message(
                author=Author(role=Role.ASSISTANT),
                content=[TextContent(text="print('Hello, world!')")],
                channel="analysis",
                recipient="python",
                content_type="code",
            )
...
                status="completed",
            )
            tool_output_msgs.append(processed)
        return tool_output_msgs

    @property
    def tool_config(self) -> Any:
        return self.python_tool.tool_config
```
**EN:** Class `HarmonyPythonTool` is defined here, extending `Tool`, and groups behavior through methods like `__init__`, `validate`, `get_result`, `get_result_parsable_context`.
**CN:** 这里定义类 `HarmonyPythonTool`，其职责是封装相关状态与行为，并通过 `__init__`、`validate`、`get_result`、`get_result_parsable_context` 等方法组织逻辑。

## Key Concepts / 关键概念
- Async request handling / 异步请求处理
- MCP tool exposure / MCP 工具暴露

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `os`, `abc`, `typing`, `importlib`
- **Third-party / 第三方**: `openai`, `openai_harmony`, `packaging`, `gpt_oss`
- **vLLM Internal / vLLM 内部**: `vllm.logger`, `vllm.utils`, `vllm.entrypoints.openai.responses.context`
