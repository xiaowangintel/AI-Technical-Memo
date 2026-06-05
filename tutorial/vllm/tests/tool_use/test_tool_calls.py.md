# test_tool_calls.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_use/test_tool_calls.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Tool Calls behavior in the Tool Use test area through focused pytest scenarios. It focuses on scenarios such as Tool Call And Choice, Tool Call With Results. / 该文件在 Tool Use 测试域中，通过有针对性的 pytest 场景验证 Tool Calls 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-15)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json

import openai
import pytest

from .utils import (
    MESSAGES_ASKING_FOR_TOOLS,
    MESSAGES_WITH_TOOL_RESPONSE,
    SEARCH_TOOL,
    SEED,
    WEATHER_TOOL,
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `openai`, `pytest`, `.utils`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_tool_call_and_choice (lines 20-144)
```python
@pytest.mark.asyncio
async def test_tool_call_and_choice(client: openai.AsyncOpenAI):
    models = await client.models.list()
    model_name: str = models.data[0].id
    chat_completion = await client.chat.completions.create(
        messages=MESSAGES_ASKING_FOR_TOOLS,
        temperature=0,
        max_completion_tokens=100,
        model=model_name,
        tools=[WEATHER_TOOL, SEARCH_TOOL],
        logprobs=False,
        seed=SEED,
    )

    choice = chat_completion.choices[0]
    stop_reason = chat_completion.choices[0].finish_reason
    tool_calls = chat_completion.choices[0].message.tool_calls

    # make sure a tool call is present
# ... omitted for brevity ...
    assert isinstance(streamed_args.get("state"), str)
    assert streamed_args.get("city") == "Dallas"
    assert streamed_args.get("state") == "TX"

    # make sure everything matches non-streaming except for ID
    assert function_name == tool_calls[0].function.name
    assert choice.message.role == role_name
    assert choice.message.tool_calls[0].function.name == function_name

    # compare streamed with non-streamed args dict-wise, not string-wise
    # because character-to-character comparison might not work e.g. the tool
    # call parser adding extra spaces or something like that. we care about the
    # dicts matching not byte-wise match
    assert parsed_arguments == streamed_args
```
**EN:** Async Checks Tool Call And Choice under a focused test scenario. The body exercises logic via `isinstance`, `json.loads`, `client.models.list` before asserting the expected outcome.
**CN:** 该测试用例验证 Tool Call And Choice 在特定场景下的行为。 函数体会先通过 `isinstance`, `json.loads`, `client.models.list` 驱动目标逻辑，再断言预期结果。

### Test: test_tool_call_with_results (lines 149-206)
```python
@pytest.mark.asyncio
async def test_tool_call_with_results(client: openai.AsyncOpenAI):
    models = await client.models.list()
    model_name: str = models.data[0].id
    chat_completion = await client.chat.completions.create(
        messages=MESSAGES_WITH_TOOL_RESPONSE,
        temperature=0,
        max_completion_tokens=100,
        model=model_name,
        tools=[WEATHER_TOOL, SEARCH_TOOL],
        logprobs=False,
        seed=SEED,
    )

    choice = chat_completion.choices[0]

    assert choice.finish_reason != "tool_calls"  # "stop" or "length"
    assert choice.message.role == "assistant"
    assert choice.message.tool_calls is None or len(choice.message.tool_calls) == 0
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
**EN:** Async Checks Tool Call With Results under a focused test scenario. The body exercises logic via `len`, `client.models.list`, `client.chat.completions.create` before asserting the expected outcome.
**CN:** 该测试用例验证 Tool Call With Results 在特定场景下的行为。 函数体会先通过 `len`, `client.models.list`, `client.chat.completions.create` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Tool-call parsing / 工具调用解析**
  - **EN:** The scenarios verify extraction of structured tool calls from model outputs and streaming deltas.
  - **CN:** 这些场景验证如何从模型输出和流式增量中提取结构化工具调用。
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
