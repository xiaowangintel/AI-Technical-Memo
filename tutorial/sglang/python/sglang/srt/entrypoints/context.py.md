# context.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/context.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements context logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 context 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-5: Import runtime dependencies / 导入运行时依赖
```python
# SPDX-License-Identifier: Apache-2.0
# Copied from vLLM
import logging
from abc import ABC, abstractmethod
from typing import Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 7-7: Import runtime dependencies / 导入运行时依赖
```python
import orjson
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 9-9: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 11-14: Provide supporting module logic / 提供辅助模块逻辑
```python
try:
    from mcp import ClientSession
except ImportError as e:
    mcp = e
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 16-16: Import runtime dependencies / 导入运行时依赖
```python
from openai_harmony import Author, Message, Role, StreamState, TextContent
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 18-23: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.harmony_utils import (
    get_encoding,
    get_streamable_parser_for_assistant,
    render_for_completion,
)
from sglang.srt.entrypoints.tool import Tool
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 26-42: Define class ConversationContext / 定义类 ConversationContext
```python
class ConversationContext(ABC):

    @abstractmethod
    def append_output(self, output) -> None:
        pass

    @abstractmethod
    async def call_tool(self) -> list[Message]:
        pass

    @abstractmethod
    def need_builtin_tool_call(self) -> bool:
        pass

    @abstractmethod
    def render_for_completion(self) -> list[int]:
        pass
```
**EN:** This block declares the class `ConversationContext`. It centers on coordinating context behavior, with methods such as append_output, call_tool, need_builtin_tool_call, render_for_completion.
**CN:** 该代码块声明类 `ConversationContext`。它负责承载与 context 相关的核心状态与行为，并通过 append_output, call_tool, need_builtin_tool_call, render_for_completion 等方法组织实现。

### Lines 45-60: Define class SimpleContext / 定义类 SimpleContext
```python
class SimpleContext(ConversationContext):

    def __init__(self):
        self.last_output = None

    def append_output(self, output) -> None:
        self.last_output = output

    def need_builtin_tool_call(self) -> bool:
        return False

    async def call_tool(self) -> list[Message]:
        raise NotImplementedError("Should not be called.")

    def render_for_completion(self) -> list[int]:
        raise NotImplementedError("Should not be called.")
```
**EN:** This block declares the class `SimpleContext`. It centers on coordinating context behavior, with methods such as __init__, append_output, need_builtin_tool_call, call_tool, render_for_completion.
**CN:** 该代码块声明类 `SimpleContext`。它负责承载与 context 相关的核心状态与行为，并通过 __init__, append_output, need_builtin_tool_call, call_tool, render_for_completion 等方法组织实现。

### Lines 63-64: Provide supporting module logic / 提供辅助模块逻辑
```python
class HarmonyContext(ConversationContext):
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 65-81: Initialize HarmonyContext / 初始化 HarmonyContext
```python
    def __init__(
        self,
        messages: list,
        tool_sessions: dict[str, Union["ClientSession", Tool]],
    ):
        # TODO: Remove the hack of Union[ClientSession, Tool] by using MCP
        # when demo.
        self._messages = messages
        self.tool_sessions = tool_sessions

        self.parser = get_streamable_parser_for_assistant()
        self.num_init_messages = len(messages)
        # TODO
        self.num_prompt_tokens = 0
        self.num_cached_tokens = 0
        self.num_output_tokens = 0
        self.num_reasoning_tokens = 0
```
**EN:** This block implements the initializer `__init__(messages, tool_sessions)` for `HarmonyContext`. It prepares the object state and connects the instance to the surrounding context workflow.
**CN:** 该代码块实现 `HarmonyContext` 的初始化方法 `__init__(messages, tool_sessions)`。它负责准备对象状态，并把实例接入 context 相关的运行流程。

### Lines 83-104: Implement append output / 实现append output
```python
    def append_output(self, output) -> None:
        if isinstance(output, dict) and "output_ids" in output:
            output_token_ids = output["output_ids"]

            for token_id in output_token_ids:
                self.parser.process(token_id)
            output_msgs = self.parser.messages

            meta_info = output["meta_info"]

            if isinstance(meta_info, dict):
                if "prompt_token_ids" in meta_info:
                    self.num_prompt_tokens = meta_info["prompt_tokens"]
                if "cached_tokens" in meta_info:
                    self.num_cached_tokens = meta_info["cached_tokens"]
                if "completion_tokens" in meta_info:
                    self.num_output_tokens += meta_info["completion_tokens"]

        else:
            output_msgs = output

        self._messages.extend(output_msgs)
```
**EN:** This block implements the method `append_output(output)` on `HarmonyContext`. It focuses on handling the context responsibilities represented by `append_output`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `HarmonyContext` 上的方法 `append_output(output)`。它围绕 `append_output` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 105-106: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 107-108: Implement messages / 实现messages
```python
    def messages(self) -> list:
        return self._messages
```
**EN:** This block implements the method `messages()` on `HarmonyContext`. It focuses on handling the context responsibilities represented by `messages`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `HarmonyContext` 上的方法 `messages()`。它围绕 `messages` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 110-117: Implement need builtin tool call / 实现need builtin 工具 call
```python
    def need_builtin_tool_call(self) -> bool:
        if not self.messages:
            return False
        last_msg = self.messages[-1]
        recipient = last_msg.recipient
        return recipient is not None and (
            recipient.startswith("browser.") or recipient.startswith("python")
        )
```
**EN:** This block implements the method `need_builtin_tool_call()` on `HarmonyContext`. It focuses on handling the context responsibilities represented by `need_builtin_tool_call`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `HarmonyContext` 上的方法 `need_builtin_tool_call()`。它围绕 `need_builtin_tool_call` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 119-133: Implement async call tool / 实现异步call 工具
```python
    async def call_tool(self) -> list[Message]:
        if not self.messages:
            return []
        last_msg = self.messages[-1]
        recipient = last_msg.recipient
        if recipient is not None:
            if recipient.startswith("browser."):
                return await self.call_search_tool(
                    self.tool_sessions["browser"], last_msg
                )
            elif recipient.startswith("python"):
                return await self.call_python_tool(
                    self.tool_sessions["python"], last_msg
                )
        raise ValueError("No tool call found")
```
**EN:** This block implements the async method `call_tool()` on `HarmonyContext`. It focuses on handling the context responsibilities represented by `call_tool`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `HarmonyContext` 上的异步方法 `call_tool()`。它围绕 `call_tool` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 135-136: Implement render for completion / 实现render for 补全
```python
    def render_for_completion(self) -> list[int]:
        return render_for_completion(self.messages)
```
**EN:** This block implements the method `render_for_completion()` on `HarmonyContext`. It focuses on handling the context responsibilities represented by `render_for_completion`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `HarmonyContext` 上的方法 `render_for_completion()`。它围绕 `render_for_completion` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 138-149: Implement async call search tool / 实现异步call search 工具
```python
    async def call_search_tool(
        self, tool_session: Union["ClientSession", Tool], last_msg: Message
    ) -> list[Message]:
        if isinstance(tool_session, Tool):
            return await tool_session.get_result(self)
        tool_name = last_msg.recipient.split(".")[1]
        args = orjson.loads(last_msg.content[0].text)
        result = await tool_session.call_tool(tool_name, args)
        result_str = result.content[0].text
        content = TextContent(text=result_str)
        author = Author(role=Role.TOOL, name=last_msg.recipient)
        return [Message(author=author, content=[content], recipient=Role.ASSISTANT)]
```
**EN:** This block implements the async method `call_search_tool(tool_session, last_msg)` on `HarmonyContext`. It focuses on handling the context responsibilities represented by `call_search_tool`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `HarmonyContext` 上的异步方法 `call_search_tool(tool_session, last_msg)`。它围绕 `call_search_tool` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 151-172: Implement async call python tool / 实现异步call python 工具
```python
    async def call_python_tool(
        self, tool_session: Union["ClientSession", Tool], last_msg: Message
    ) -> list[Message]:
        if isinstance(tool_session, Tool):
            return await tool_session.get_result(self)
        param = {
            "code": last_msg.content[0].text,
        }
        result = await tool_session.call_tool("python", param)
        result_str = result.content[0].text

        content = TextContent(text=result_str)
        author = Author(role=Role.TOOL, name="python")

        return [
            Message(
                author=author,
                content=[content],
                channel=last_msg.channel,
                recipient=Role.ASSISTANT,
            )
        ]
```
**EN:** This block implements the async method `call_python_tool(tool_session, last_msg)` on `HarmonyContext`. It focuses on handling the context responsibilities represented by `call_python_tool`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `HarmonyContext` 上的异步方法 `call_python_tool(tool_session, last_msg)`。它围绕 `call_python_tool` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 175-176: Provide supporting module logic / 提供辅助模块逻辑
```python
class StreamingHarmonyContext(HarmonyContext):
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 177-184: Initialize StreamingHarmonyContext / 初始化 StreamingHarmonyContext
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.last_output = None

        self.parser = get_streamable_parser_for_assistant()
        self.encoding = get_encoding()
        self.last_tok = None
        self.num_processed_tokens = 0
```
**EN:** This block implements the initializer `__init__(*args, **kwargs)` for `StreamingHarmonyContext`. It prepares the object state and connects the instance to the surrounding context workflow.
**CN:** 该代码块实现 `StreamingHarmonyContext` 的初始化方法 `__init__(*args, **kwargs)`。它负责准备对象状态，并把实例接入 context 相关的运行流程。

### Lines 185-186: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 187-188: Implement messages / 实现messages
```python
    def messages(self) -> list:
        return self.parser.messages
```
**EN:** This block implements the method `messages()` on `StreamingHarmonyContext`. It focuses on handling the context responsibilities represented by `messages`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `StreamingHarmonyContext` 上的方法 `messages()`。它围绕 `messages` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 190-227: Implement append output / 实现append output
```python
    def append_output(self, output) -> None:
        if isinstance(output, dict) and "output_ids" in output:
            # RequestOutput from SGLang with outputs
            output_token_ids = output["output_ids"]

            # Check if we need to handle cumulative tokens
            meta_info = output.get("meta_info", {})
            completion_tokens = meta_info.get("completion_tokens")
            if (
                completion_tokens is not None
                and len(output_token_ids) == completion_tokens
            ):
                # Case 1: When --incremental-streaming-output is not set.
                # The output_ids contains all tokens generated so far.
                # We only need to process the new tokens.
                new_token_ids = output_token_ids[self.num_processed_tokens :]
                self.num_processed_tokens = len(output_token_ids)
            else:
                # Case 2: When --incremental-streaming-output is set.
                # The output_ids contains only the new tokens.
                new_token_ids = output_token_ids
                self.num_processed_tokens += len(output_token_ids)

            for token_id in new_token_ids:
                self.parser.process(token_id)

        else:
            # Handle the case of tool output in direct message format
            assert len(output) == 1, "Tool output should be a single message"
            msg = output[0]
            # Sometimes the recipient is not set for tool messages,
            # so we set it to "assistant"
            if msg.author.role == Role.TOOL and msg.recipient is None:
                msg.recipient = "assistant"
            toks = self.encoding.render(msg)
            for tok in toks:
                self.parser.process(tok)
            self.last_tok = toks[-1]
```
**EN:** This block implements the method `append_output(output)` on `StreamingHarmonyContext`. It focuses on handling the context responsibilities represented by `append_output`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `StreamingHarmonyContext` 上的方法 `append_output(output)`。它围绕 `append_output` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 229-230: Implement is expecting start / 实现is expecting start
```python
    def is_expecting_start(self) -> bool:
        return self.parser.state == StreamState.EXPECT_START
```
**EN:** This block implements the method `is_expecting_start()` on `StreamingHarmonyContext`. It focuses on handling the context responsibilities represented by `is_expecting_start`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `StreamingHarmonyContext` 上的方法 `is_expecting_start()`。它围绕 `is_expecting_start` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 232-233: Implement is assistant action turn / 实现is assistant action turn
```python
    def is_assistant_action_turn(self) -> bool:
        return self.last_tok in self.encoding.stop_tokens_for_assistant_actions()
```
**EN:** This block implements the method `is_assistant_action_turn()` on `StreamingHarmonyContext`. It focuses on handling the context responsibilities represented by `is_assistant_action_turn`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `StreamingHarmonyContext` 上的方法 `is_assistant_action_turn()`。它围绕 `is_assistant_action_turn` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

### Lines 235-249: Implement render for completion / 实现render for 补全
```python
    def render_for_completion(self) -> list[int]:
        # now this list of tokens as next turn's starting tokens
        # `<|start|>assistant``,
        # we need to process them in parser.
        rendered_tokens = super().render_for_completion()

        last_n = -1
        to_process = []
        while rendered_tokens[last_n] != self.last_tok:
            to_process.append(rendered_tokens[last_n])
            last_n -= 1
        for tok in reversed(to_process):
            self.parser.process(tok)

        return rendered_tokens
```
**EN:** This block implements the method `render_for_completion()` on `StreamingHarmonyContext`. It focuses on handling the context responsibilities represented by `render_for_completion`, so the class can advance the context workflow in a self-contained way.
**CN:** 该代码块实现 `StreamingHarmonyContext` 上的方法 `render_for_completion()`。它围绕 `render_for_completion` 所承担的 context 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: ConversationContext, SimpleContext, HarmonyContext, StreamingHarmonyContext
- **Domain focus / 领域焦点**: context / context
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, typing
- **Third-party / 第三方库**: abc, mcp, openai_harmony, orjson
- **Local Modules / 本地模块**: sglang.srt.entrypoints.harmony_utils, sglang.srt.entrypoints.tool
