# test_parallel_tool_calls.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_use/test_parallel_tool_calls.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Parallel Tool Calls behavior in the Tool Use test area through focused pytest scenarios. It focuses on scenarios such as Parallel Tool Calls, Parallel Tool Calls With Results, Parallel Tool Calls False. / 该文件在 Tool Use 测试域中，通过有针对性的 pytest 场景验证 Parallel Tool Calls 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-16)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json

import openai
import pytest

from .utils import (
    MESSAGES_ASKING_FOR_PARALLEL_TOOLS,
    MESSAGES_WITH_PARALLEL_TOOL_RESPONSE,
    SEARCH_TOOL,
    SEED,
    WEATHER_TOOL,
    ServerConfig,
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `openai`, `pytest`, `.utils`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_parallel_tool_calls (lines 23-147)
```python
@pytest.mark.asyncio
async def test_parallel_tool_calls(
    client: openai.AsyncOpenAI, server_config: ServerConfig
):
    if not server_config.get("supports_parallel", True):
        pytest.skip(
            "The {} model doesn't support parallel tool calls".format(
                server_config["model"]
            )
        )

    models = await client.models.list()
    model_name: str = models.data[0].id
    chat_completion = await client.chat.completions.create(
        messages=MESSAGES_ASKING_FOR_PARALLEL_TOOLS,
        temperature=0,
        max_completion_tokens=200,
        model=model_name,
        tools=[WEATHER_TOOL, SEARCH_TOOL],
# ... omitted for brevity ...
                    assert isinstance(tool_call.function.arguments, str)

                    tool_call_args[tool_call.index] += tool_call.function.arguments

    assert finish_reason_count == 1
    assert role_name == "assistant"

    assert len(non_streamed_tool_calls) == len(tool_call_names) == len(tool_call_args)

    for i in range(2):
        assert non_streamed_tool_calls[i].function.name == tool_call_names[i]
        streamed_args = json.loads(tool_call_args[i])
        non_streamed_args = json.loads(non_streamed_tool_calls[i].function.arguments)
        assert streamed_args == non_streamed_args
```
**EN:** Async Checks Parallel Tool Calls under a focused test scenario. The body exercises logic via `range`, `server_config.get`, `pytest.skip` before asserting the expected outcome.
**CN:** 该测试用例验证 Parallel Tool Calls 在特定场景下的行为。 函数体会先通过 `range`, `server_config.get`, `pytest.skip` 驱动目标逻辑，再断言预期结果。

### Test: test_parallel_tool_calls_with_results (lines 152-219)
```python
@pytest.mark.asyncio
async def test_parallel_tool_calls_with_results(
    client: openai.AsyncOpenAI, server_config: ServerConfig
):
    if not server_config.get("supports_parallel", True):
        pytest.skip(
            "The {} model doesn't support parallel tool calls".format(
                server_config["model"]
            )
        )

    models = await client.models.list()
    model_name: str = models.data[0].id
    chat_completion = await client.chat.completions.create(
        messages=MESSAGES_WITH_PARALLEL_TOOL_RESPONSE,
        temperature=0,
        max_completion_tokens=200,
        model=model_name,
        tools=[WEATHER_TOOL, SEARCH_TOOL],
# ... omitted for brevity ...

        if delta.content:
            chunks.append(delta.content)

        if chunk.choices[0].finish_reason is not None:
            finish_reason_count += 1
            assert chunk.choices[0].finish_reason == choice.finish_reason

        assert not delta.tool_calls or len(delta.tool_calls) == 0

    assert role_sent
    assert finish_reason_count == 1
    assert len(chunks)
    assert "".join(chunks) == choice.message.content
```
**EN:** Async Checks Parallel Tool Calls With Results under a focused test scenario. The body exercises logic via `len`, `server_config.get`, `pytest.skip` before asserting the expected outcome.
**CN:** 该测试用例验证 Parallel Tool Calls With Results 在特定场景下的行为。 函数体会先通过 `len`, `server_config.get`, `pytest.skip` 驱动目标逻辑，再断言预期结果。

### Test: test_parallel_tool_calls_false (lines 222-278)
```python
@pytest.mark.asyncio
async def test_parallel_tool_calls_false(client: openai.AsyncOpenAI):
    """
    Ensure only one tool call is returned when parallel_tool_calls is False.
    """

    models = await client.models.list()
    model_name: str = models.data[0].id
    chat_completion = await client.chat.completions.create(
        messages=MESSAGES_ASKING_FOR_PARALLEL_TOOLS,
        temperature=0,
        max_completion_tokens=200,
        model=model_name,
        tools=[WEATHER_TOOL, SEARCH_TOOL],
        logprobs=False,
        seed=SEED,
        parallel_tool_calls=False,
    )

# ... omitted for brevity ...
        # if there's a finish reason make sure it's tools
        if chunk.choices[0].finish_reason:
            finish_reason_count += 1
            assert chunk.choices[0].finish_reason == "tool_calls"

        streamed_tool_calls = chunk.choices[0].delta.tool_calls
        if streamed_tool_calls and len(streamed_tool_calls) > 0:
            tool_call = streamed_tool_calls[0]
            if tool_call.id:
                tool_call_id_count += 1

    # make sure only 1 streaming tool call is present
    assert tool_call_id_count == 1
    assert finish_reason_count == 1
```
**EN:** Ensure only one tool call is returned when parallel_tool_calls is False. The body exercises logic via `client.models.list`, `client.chat.completions.create`, `len` before asserting the expected outcome.
**CN:** 该测试用例验证 Parallel Tool Calls False 在特定场景下的行为。 函数体会先通过 `client.models.list`, `client.chat.completions.create`, `len` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Tool-call parsing / 工具调用解析**
  - **EN:** The scenarios verify extraction of structured tool calls from model outputs and streaming deltas.
  - **CN:** 这些场景验证如何从模型输出和流式增量中提取结构化工具调用。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。
- **Sampling behavior / 采样行为**
  - **EN:** The assertions target decoding controls and the shape of generated outputs.
  - **CN:** 这些断言关注解码控制参数以及生成结果的形态。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Third-party / 第三方依赖**: `openai`, `pytest`
- **Local test utilities / 本地测试辅助**: `.utils`
