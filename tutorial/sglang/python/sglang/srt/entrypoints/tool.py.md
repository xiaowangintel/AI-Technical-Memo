# tool.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/tool.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements tool logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 工具 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-5: Import runtime dependencies / 导入运行时依赖
```python
# SPDX-License-Identifier: Apache-2.0
import logging
import os
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING, Any
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 7-7: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.utils import print_info_once, print_warning_once
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 9-11: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    # Avoid circular import.
    from sglang.srt.entrypoints.context import ConversationContext
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 13-13: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 16-20: Define class Tool / 定义类 Tool
```python
class Tool(ABC):

    @abstractmethod
    async def get_result(self, context: "ConversationContext") -> Any:
        pass
```
**EN:** This block declares the class `Tool`. It centers on coordinating tool behavior, with methods such as get_result.
**CN:** 该代码块声明类 `Tool`。它负责承载与 工具 相关的核心状态与行为，并通过 get_result 等方法组织实现。

### Lines 23-57: Define class HarmonyBrowserTool / 定义类 HarmonyBrowserTool
```python
class HarmonyBrowserTool(Tool):

    def __init__(self):
        self.enabled = True
        exa_api_key = os.getenv("EXA_API_KEY")
        if not exa_api_key:
            self.enabled = False
            print_warning_once("EXA_API_KEY is not set, browsing is disabled")
            return

        try:
            from gpt_oss.tools.simple_browser import SimpleBrowserTool
            from gpt_oss.tools.simple_browser.backend import ExaBackend
        except ImportError:
            self.enabled = False
            print_warning_once("gpt_oss is not installed, browsing is disabled")
            return

        browser_backend = ExaBackend(source="web", api_key=exa_api_key)
        self.browser_tool = SimpleBrowserTool(backend=browser_backend)
        print_info_once("Browser tool initialized")

    async def get_result(self, context: "ConversationContext") -> Any:
        from sglang.srt.entrypoints.context import HarmonyContext

        assert isinstance(context, HarmonyContext)
        last_msg = context.messages[-1]
        tool_output_msgs = []
        async for msg in self.browser_tool.process(last_msg):
            tool_output_msgs.append(msg)
        return tool_output_msgs

    @property
    def tool_config(self) -> Any:
        return self.browser_tool.tool_config
```
**EN:** This block declares the class `HarmonyBrowserTool`. It centers on coordinating tool behavior, with methods such as __init__, get_result, tool_config.
**CN:** 该代码块声明类 `HarmonyBrowserTool`。它负责承载与 工具 相关的核心状态与行为，并通过 __init__, get_result, tool_config 等方法组织实现。

### Lines 60-87: Define class HarmonyPythonTool / 定义类 HarmonyPythonTool
```python
class HarmonyPythonTool(Tool):

    def __init__(self):
        self.enabled = True

        try:
            from gpt_oss.tools.python_docker.docker_tool import PythonTool
        except ImportError:
            self.enabled = False
            print_warning_once("gpt_oss is not installed, code interpreter is disabled")
            return

        self.python_tool = PythonTool()
        print_info_once("Code interpreter tool initialized")

    async def get_result(self, context: "ConversationContext") -> Any:
        from sglang.srt.entrypoints.context import HarmonyContext

        assert isinstance(context, HarmonyContext)
        last_msg = context.messages[-1]
        tool_output_msgs = []
        async for msg in self.python_tool.process(last_msg):
            tool_output_msgs.append(msg)
        return tool_output_msgs

    @property
    def tool_config(self) -> Any:
        return self.python_tool.tool_config
```
**EN:** This block declares the class `HarmonyPythonTool`. It centers on coordinating tool behavior, with methods such as __init__, get_result, tool_config.
**CN:** 该代码块声明类 `HarmonyPythonTool`。它负责承载与 工具 相关的核心状态与行为，并通过 __init__, get_result, tool_config 等方法组织实现。

## Key Concepts / 关键概念
- **Core types / 核心类型**: Tool, HarmonyBrowserTool, HarmonyPythonTool
- **Domain focus / 领域焦点**: tool / 工具
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, os, typing
- **Third-party / 第三方库**: abc, gpt_oss.tools.python_docker.docker_tool, gpt_oss.tools.simple_browser, gpt_oss.tools.simple_browser.backend
- **Local Modules / 本地模块**: sglang.srt.entrypoints.context, sglang.srt.utils
