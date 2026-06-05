# test_serving_chat.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_serving_chat.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 27 test(s), 13 fixture(s), and 21 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 27 个测试、13 个 fixture，以及 21 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L47)
```python
import asyncio
import json
from contextlib import suppress
from dataclasses import dataclass, field
from typing import Any
from unittest.mock import AsyncMock, MagicMock

import pytest
import pytest_asyncio
from openai import OpenAI

from tests.entrypoints.openai.utils import (
    accumulate_streaming_response,
    verify_chat_response,
    verify_harmony_messages,
)
from tests.utils import RemoteOpenAIServer
from vllm._aiter_ops import is_aiter_found_and_supported
# ... 23 lines omitted for brevity ...
from vllm.tokenizers.mistral import MistralTokenizer
from vllm.tokenizers.registry import cached_tokenizer_from_config
from vllm.tool_parsers import ToolParserManager
from vllm.v1.engine.async_llm import AsyncLLM
```
**EN:** Imports standard-library modules such as `asyncio`, `contextlib.suppress`, `dataclasses.dataclass`, third-party packages like `_pytest.monkeypatch.MonkeyPatch`, `openai.OpenAI`, `pytest`, project helpers such as `tests.entrypoints.openai.utils.accumulate_streaming_response`, `tests.entrypoints.openai.utils.verify_chat_response`, `tests.entrypoints.openai.utils.verify_harmony_messages`.
**CN:** 导入标准库模块（如 `asyncio`、`contextlib.suppress`、`dataclasses.dataclass`）、第三方包（如 `_pytest.monkeypatch.MonkeyPatch`、`openai.OpenAI`、`pytest`）、项目内辅助模块（如 `tests.entrypoints.openai.utils.accumulate_streaming_response`、`tests.entrypoints.openai.utils.verify_chat_response`、`tests.entrypoints.openai.utils.verify_harmony_messages`）。

### Module setup / 模块级配置: GPT_OSS_MODEL_NAME, GPT_OSS_SPECULATOR_NAME (L49-L50)
```python
GPT_OSS_MODEL_NAME = "openai/gpt-oss-20b"
GPT_OSS_SPECULATOR_NAME = "RedHatAI/gpt-oss-20b-speculator.eagle3"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `GPT_OSS_MODEL_NAME`, `GPT_OSS_SPECULATOR_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `GPT_OSS_MODEL_NAME`、`GPT_OSS_SPECULATOR_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: monkeypatch_module (L53-L59)
```python
@pytest.fixture(scope="module")
def monkeypatch_module():
    from _pytest.monkeypatch import MonkeyPatch

    mpatch = MonkeyPatch()
    yield mpatch
    mpatch.undo()
```
**EN:** This fixture prepares `monkeypatch_module` for dependent tests. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `monkeypatch_module`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: with_tool_parser (L62-L68)
```python
@pytest.fixture(
    scope="module",
    params=[True, False],
    ids=["with_tool_parser", "without_tool_parser"],
)
def with_tool_parser(request) -> bool:
    return request.param
```
**EN:** This fixture prepares `with_tool_parser` for dependent tests. Key inputs are `request`.
**CN:** 这个 fixture 为依赖它的测试准备 `with_tool_parser`。 关键输入包括 `request`。

### Fixture / 夹具: exclude_tools_when_tool_choice_none (L71-L77)
```python
@pytest.fixture(
    scope="module",
    params=[True],
    ids=["exclude_tools_when_tool_choice_none"],
)
def exclude_tools_when_tool_choice_none(request) -> bool:
    return request.param
```
**EN:** This fixture prepares `exclude_tools_when_tool_choice_none` for dependent tests. Key inputs are `request`.
**CN:** 这个 fixture 为依赖它的测试准备 `exclude_tools_when_tool_choice_none`。 关键输入包括 `request`。

### Fixture / 夹具: default_server_args (L80-L105)
```python
@pytest.fixture(scope="module")
def default_server_args(
    with_tool_parser: bool,
    exclude_tools_when_tool_choice_none: bool,
):
    args = [
        # use half precision for speed and memory savings in CI environment
        "--enforce-eager",
        "--max-model-len",
        "4096",
        "--reasoning-parser",
        "openai_gptoss",
        "--gpu-memory-utilization",
        "0.85",
    ]
    if with_tool_parser:
        args.extend(
            [
                "--tool-call-parser",
                "openai",
                "--enable-auto-tool-choice",
            ]
        )
    if exclude_tools_when_tool_choice_none:
        args.append("--exclude-tools-when-tool-choice-none")
    return args
```
**EN:** This fixture prepares `default_server_args` for dependent tests. Key inputs are `with_tool_parser`, `exclude_tools_when_tool_choice_none`.
**CN:** 这个 fixture 为依赖它的测试准备 `default_server_args`。 关键输入包括 `with_tool_parser`、`exclude_tools_when_tool_choice_none`。

### Fixture / 夹具: gptoss_server (L108-L112)
```python
@pytest.fixture(scope="class")
def gptoss_server(default_server_args: list[str]):
    server_args = default_server_args + ["--attention-backend=TRITON_ATTN"]
    with RemoteOpenAIServer(GPT_OSS_MODEL_NAME, server_args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `gptoss_server` for dependent tests. Key inputs are `default_server_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `gptoss_server`。 关键输入包括 `default_server_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: gptoss_speculative_server (L115-L137)
```python
@pytest.fixture(scope="class")
def gptoss_speculative_server(default_server_args: list[str]):
    attention_backend = (
        "TRITON_ATTN"
        if not is_aiter_found_and_supported()
        else "ROCM_AITER_UNIFIED_ATTN"
    )
    server_args = default_server_args + [
        "--speculative-config",
        f'{{"model": "{GPT_OSS_SPECULATOR_NAME}", '
        f'"method": "eagle3", "num_speculative_tokens": 3}}',
        f"--attention-backend={attention_backend}",
    ]
    # gpt-oss requires AITER unified attention on ROCm
    # TODO: Remove after fixing TRITON_ATTN issue on ROCm
    # https://github.com/vllm-project/vllm/issues/32434
    env_dict = None
    if is_aiter_found_and_supported():
        env_dict = {"VLLM_ROCM_USE_AITER": "1"}
    with RemoteOpenAIServer(
        GPT_OSS_MODEL_NAME, server_args, env_dict=env_dict, max_wait_seconds=480
    ) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `gptoss_speculative_server` for dependent tests. Key inputs are `default_server_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `gptoss_speculative_server`。 关键输入包括 `default_server_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: gptoss_client (L140-L143)
```python
@pytest_asyncio.fixture
async def gptoss_client(gptoss_server):
    async with gptoss_server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `gptoss_client` for dependent tests. Key inputs are `gptoss_server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `gptoss_client`。 关键输入包括 `gptoss_server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: gptoss_speculative_client (L146-L149)
```python
@pytest_asyncio.fixture
async def gptoss_speculative_client(gptoss_speculative_server):
    async with gptoss_speculative_server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `gptoss_speculative_client` for dependent tests. Key inputs are `gptoss_speculative_server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `gptoss_speculative_client`。 关键输入包括 `gptoss_speculative_server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Class / 类: TestGPTOSSChat (L152-L463)
```python
class TestGPTOSSChat:
    @pytest.mark.asyncio
    async def test_gpt_oss_chat_tool_call_streaming(
        self, gptoss_client: OpenAI, with_tool_parser: bool
    ):
        tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_current_weather",
                    "description": "Get the current weather in a given location",
                    "parameters": {
                        "type": "object",
                        "properties": {
# ... 290 lines omitted for brevity ...
            messages=messages,
            tools=tools,
            tool_choice="none",
            temperature=0.0,
        )

        msg = tool_choice_none.choices[0].message
        assert len(msg.tool_calls) == 0
```
**EN:** This class groups related scenarios in `TestGPTOSSChat`. It contains 4 test method(s) and 0 supporting method(s). Representative methods include `test_gpt_oss_chat_tool_call_streaming`, `test_gpt_oss_multi_turn_chat`, `test_gpt_oss_tool_message_array_content`.
**CN:** 该类将与 `TestGPTOSSChat` 相关的场景组织在一起。 它包含 4 个测试方法和 0 个辅助方法。 代表性方法包括 `test_gpt_oss_chat_tool_call_streaming`、`test_gpt_oss_multi_turn_chat`、`test_gpt_oss_tool_message_array_content`。

### Test method / 测试方法: TestGPTOSSChat.test_gpt_oss_chat_tool_call_streaming (L153-L209)
```python
    @pytest.mark.asyncio
    async def test_gpt_oss_chat_tool_call_streaming(
        self, gptoss_client: OpenAI, with_tool_parser: bool
    ):
        tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_current_weather",
                    "description": "Get the current weather in a given location",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "city": {"type": "string"},
                            "state": {"type": "string"},
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
# ... 31 lines omitted for brevity ...
                content_buf += delta.content
        if with_tool_parser:
            assert name is not None
            assert len(args_buf) > 0
        else:
            assert name is None
            assert len(args_buf) == 0
            assert len(content_buf) > 0
```
**EN:** This async test validates `TestGPTOSSChat.test_gpt_oss_chat_tool_call_streaming`. Relevant pytest markers include `asyncio`. Key inputs are `gptoss_client`, `with_tool_parser`. It drives client-facing request creation through the API surface under test. The main assertion is `name is not None` and `len(args_buf) > 0`.
**CN:** 这个异步测试验证 `TestGPTOSSChat.test_gpt_oss_chat_tool_call_streaming`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `gptoss_client`、`with_tool_parser`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `name is not None` and `len(args_buf) > 0`。

### Test method / 测试方法: TestGPTOSSChat.test_gpt_oss_multi_turn_chat (L211-L275)
```python
    @pytest.mark.asyncio
    async def test_gpt_oss_multi_turn_chat(
        self, gptoss_client: OpenAI, with_tool_parser: bool
    ):
        if not with_tool_parser:
            pytest.skip("skip non-tool for multi-turn tests")
        tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_current_weather",
                    "description": "Get the current weather in a given location",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "city": {"type": "string"},
                            "state": {"type": "string"},
                            "unit": {
# ... 39 lines omitted for brevity ...
            messages=messages,
            tools=tools,
            temperature=0.0,
        )
        second_msg = second.choices[0].message
        assert (second_msg.content is not None and len(second_msg.content) > 0) or (
            second_msg.tool_calls is not None and len(second_msg.tool_calls) > 0
        )
```
**EN:** This async test validates `TestGPTOSSChat.test_gpt_oss_multi_turn_chat`. Relevant pytest markers include `asyncio`. Key inputs are `gptoss_client`, `with_tool_parser`. It drives client-facing request creation through the API surface under test. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `first_msg.tool_calls is not None and len(first_msg.tool_calls) > 0` and `tc.function is not None and tc.function.name == 'get_current_weather'`.
**CN:** 这个异步测试验证 `TestGPTOSSChat.test_gpt_oss_multi_turn_chat`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `gptoss_client`、`with_tool_parser`。 它通过被测 API 表面触发面向客户端的请求创建流程。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `first_msg.tool_calls is not None and len(first_msg.tool_calls) > 0` and `tc.function is not None and tc.function.name == 'get_current_weather'`。

### Test method / 测试方法: TestGPTOSSChat.test_gpt_oss_tool_message_array_content (L277-L400)
```python
    @pytest.mark.asyncio
    async def test_gpt_oss_tool_message_array_content(
        self, gptoss_client: OpenAI, with_tool_parser: bool
    ):
        """Test that tool messages support both string and array content formats."""
        if not with_tool_parser:
            pytest.skip("skip non-tool for array content tests")

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_weather",
                    "description": "Get the current weather in a given location",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "city": {"type": "string"},
# ... 98 lines omitted for brevity ...
            model=GPT_OSS_MODEL_NAME,
            messages=messages_multi_array,
            tools=tools,
            temperature=0.0,
        )

        assert response_multi_array is not None
        assert response_multi_array.choices[0].message is not None
```
**EN:** This async test validates `TestGPTOSSChat.test_gpt_oss_tool_message_array_content`. Relevant pytest markers include `asyncio`. Key inputs are `gptoss_client`, `with_tool_parser`. It drives client-facing request creation through the API surface under test. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `response_string is not None` and `response_string.choices[0].message is not None`.
**CN:** 这个异步测试验证 `TestGPTOSSChat.test_gpt_oss_tool_message_array_content`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `gptoss_client`、`with_tool_parser`。 它通过被测 API 表面触发面向客户端的请求创建流程。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `response_string is not None` and `response_string.choices[0].message is not None`。

### Test method / 测试方法: TestGPTOSSChat.test_gpt_oss_tool_choice_none (L402-L463)
```python
    @pytest.mark.asyncio
    async def test_gpt_oss_tool_choice_none(
        self,
        gptoss_client: OpenAI,
        with_tool_parser: bool,
        exclude_tools_when_tool_choice_none: bool,
    ):
        if not (with_tool_parser and exclude_tools_when_tool_choice_none):
            pytest.skip(
                "skip tool_choice tests when non-tool or "
                "--exclude-tools-when-tool-choice-none not set"
            )

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_current_weather",
# ... 36 lines omitted for brevity ...
            messages=messages,
            tools=tools,
            tool_choice="none",
            temperature=0.0,
        )

        msg = tool_choice_none.choices[0].message
        assert len(msg.tool_calls) == 0
```
**EN:** This async test validates `TestGPTOSSChat.test_gpt_oss_tool_choice_none`. Relevant pytest markers include `asyncio`. Key inputs are `gptoss_client`, `with_tool_parser`, `exclude_tools_when_tool_choice_none`. It drives client-facing request creation through the API surface under test. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `len(msg.tool_calls) == 1` and `len(msg.tool_calls) == 0`.
**CN:** 这个异步测试验证 `TestGPTOSSChat.test_gpt_oss_tool_choice_none`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `gptoss_client`、`with_tool_parser`、`exclude_tools_when_tool_choice_none`。 它通过被测 API 表面触发面向客户端的请求创建流程。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `len(msg.tool_calls) == 1` and `len(msg.tool_calls) == 0`。

### Class / 类: TestGPTOSSSpeculativeChat (L466-L499)
```python
class TestGPTOSSSpeculativeChat:
    @pytest.mark.asyncio
    async def test_gpt_oss_speculative_reasoning_leakage(
        self,
        gptoss_speculative_client: OpenAI,
        with_tool_parser: bool,
    ):
        if not with_tool_parser:
            pytest.skip("skip non-tool for array content tests")

        messages = [
            {"role": "user", "content": "Calculate 2+2. Return the answer 4 only."},
        ]

# ... 12 lines omitted for brevity ...
                content += delta.content

            chunk_reasoning = getattr(delta, "reasoning", None)
            if chunk_reasoning:
                reasoning += delta.reasoning

        assert len(reasoning) > 0, "No reasoning was generated."
        assert content.strip() == "4"
```
**EN:** This class groups related scenarios in `TestGPTOSSSpeculativeChat`. It contains 1 test method(s) and 0 supporting method(s). Representative methods include `test_gpt_oss_speculative_reasoning_leakage`.
**CN:** 该类将与 `TestGPTOSSSpeculativeChat` 相关的场景组织在一起。 它包含 1 个测试方法和 0 个辅助方法。 代表性方法包括 `test_gpt_oss_speculative_reasoning_leakage`。

### Test method / 测试方法: TestGPTOSSSpeculativeChat.test_gpt_oss_speculative_reasoning_leakage (L467-L499)
```python
    @pytest.mark.asyncio
    async def test_gpt_oss_speculative_reasoning_leakage(
        self,
        gptoss_speculative_client: OpenAI,
        with_tool_parser: bool,
    ):
        if not with_tool_parser:
            pytest.skip("skip non-tool for array content tests")

        messages = [
            {"role": "user", "content": "Calculate 2+2. Return the answer 4 only."},
        ]

        stream = await gptoss_speculative_client.chat.completions.create(
            model=GPT_OSS_MODEL_NAME,
            messages=messages,
            stream=True,
            temperature=0.0,
# ... 7 lines omitted for brevity ...
                content += delta.content

            chunk_reasoning = getattr(delta, "reasoning", None)
            if chunk_reasoning:
                reasoning += delta.reasoning

        assert len(reasoning) > 0, "No reasoning was generated."
        assert content.strip() == "4"
```
**EN:** This async test validates `TestGPTOSSSpeculativeChat.test_gpt_oss_speculative_reasoning_leakage`. Relevant pytest markers include `asyncio`. Key inputs are `gptoss_speculative_client`, `with_tool_parser`. It drives client-facing request creation through the API surface under test. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `len(reasoning) > 0` and `content.strip() == '4'`.
**CN:** 这个异步测试验证 `TestGPTOSSSpeculativeChat.test_gpt_oss_speculative_reasoning_leakage`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `gptoss_speculative_client`、`with_tool_parser`。 它通过被测 API 表面触发面向客户端的请求创建流程。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `len(reasoning) > 0` and `content.strip() == '4'`。

### Module setup / 模块级配置: MODEL_NAME, MODEL_NAME_SHORT, CHAT_TEMPLATE (L502-L508)
```python
MODEL_NAME = "openai-community/gpt2"
MODEL_NAME_SHORT = "gpt2"
CHAT_TEMPLATE = "Dummy chat template for testing {}"
BASE_MODEL_PATHS = [
    BaseModelPath(name=MODEL_NAME, model_path=MODEL_NAME),
    BaseModelPath(name=MODEL_NAME_SHORT, model_path=MODEL_NAME_SHORT),
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MODEL_NAME_SHORT`, `CHAT_TEMPLATE`, `BASE_MODEL_PATHS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MODEL_NAME_SHORT`、`CHAT_TEMPLATE`、`BASE_MODEL_PATHS`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: MockHFConfig (L511-L513)
```python
@dataclass
class MockHFConfig:
    model_type: str = "any"
```
**EN:** This class groups related scenarios in `MockHFConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockHFConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: MockModelConfig (L516-L544)
```python
@dataclass
class MockModelConfig:
    task = "generate"
    runner_type = "generate"
    model = MODEL_NAME
    tokenizer = MODEL_NAME
    trust_remote_code = False
    tokenizer_mode = "auto"
    max_model_len = 100
    tokenizer_revision = None
    multimodal_config = MultiModalConfig()
    hf_config = MockHFConfig()
    hf_text_config = MockHFConfig()
    logits_processors: list[str] | None = None
# ... 7 lines omitted for brevity ...
    skip_tokenizer_init: bool = False
    is_encoder_decoder: bool = False
    is_multimodal_model: bool = False
    renderer_num_workers: int = 1
    enable_prompt_embeds: bool = False

    def get_diff_sampling_param(self):
        return self.diff_sampling_param or {}
```
**EN:** This class groups related scenarios in `MockModelConfig`. Decorators such as `@dataclass` make it a compact metadata container. It contains 0 test method(s) and 1 supporting method(s). Representative methods include `get_diff_sampling_param`.
**CN:** 该类将与 `MockModelConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 它包含 0 个测试方法和 1 个辅助方法。 代表性方法包括 `get_diff_sampling_param`。

### Helper method / 辅助方法: MockModelConfig.get_diff_sampling_param (L543-L544)
```python
    def get_diff_sampling_param(self):
        return self.diff_sampling_param or {}
```
**EN:** This helper encapsulates reusable logic in `MockModelConfig.get_diff_sampling_param`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `MockModelConfig.get_diff_sampling_param` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: MockParallelConfig (L547-L549)
```python
@dataclass
class MockParallelConfig:
    _api_process_rank: int = 0
```
**EN:** This class groups related scenarios in `MockParallelConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockParallelConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: MockVllmConfig (L552-L555)
```python
@dataclass
class MockVllmConfig:
    model_config: MockModelConfig
    parallel_config: MockParallelConfig
```
**EN:** This class groups related scenarios in `MockVllmConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockVllmConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Helper / 辅助函数: _build_renderer (L558-L562)
```python
def _build_renderer(model_config: MockModelConfig):
    return HfRenderer(
        MockVllmConfig(model_config, parallel_config=MockParallelConfig()),
        cached_tokenizer_from_config(model_config),
    )
```
**EN:** This helper encapsulates reusable logic in `_build_renderer`. Key inputs are `model_config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_renderer` 中。 关键输入包括 `model_config`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _build_serving_render (L565-L575)
```python
def _build_serving_render(
    engine, model_registry: OpenAIModelRegistry
) -> OpenAIServingRender:
    return OpenAIServingRender(
        model_config=engine.model_config,
        renderer=engine.renderer,
        model_registry=model_registry,
        request_logger=None,
        chat_template=CHAT_TEMPLATE,
        chat_template_content_format="auto",
    )
```
**EN:** This helper encapsulates reusable logic in `_build_serving_render`. Key inputs are `engine`, `model_registry`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_serving_render` 中。 关键输入包括 `engine`、`model_registry`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _build_serving_chat (L578-L595)
```python
def _build_serving_chat(engine: AsyncLLM) -> OpenAIServingChat:
    models = OpenAIServingModels(
        engine_client=engine,
        base_model_paths=BASE_MODEL_PATHS,
    )
    openai_serving_render = _build_serving_render(engine, models.registry)

    serving_chat = OpenAIServingChat(
        engine,
        models,
        response_role="assistant",
        openai_serving_render=openai_serving_render,
        chat_template=CHAT_TEMPLATE,
        chat_template_content_format="auto",
        request_logger=None,
    )

    return serving_chat
```
**EN:** This helper encapsulates reusable logic in `_build_serving_chat`. Key inputs are `engine`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_serving_chat` 中。 关键输入包括 `engine`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: MockEngine (L598-L602)
```python
@dataclass
class MockEngine:
    model_config: MockModelConfig = field(default_factory=MockModelConfig)
    input_processor: MagicMock = field(default_factory=MagicMock)
    renderer: MagicMock = field(default_factory=MagicMock)
```
**EN:** This class groups related scenarios in `MockEngine`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockEngine` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Helper / 辅助函数: _async_serving_chat_init (L605-L620)
```python
async def _async_serving_chat_init():
    engine = MockEngine()

    models = OpenAIServingModels(engine, BASE_MODEL_PATHS)
    openai_serving_render = _build_serving_render(engine, models.registry)

    serving_completion = OpenAIServingChat(
        engine,
        models,
        response_role="assistant",
        openai_serving_render=openai_serving_render,
        chat_template=CHAT_TEMPLATE,
        chat_template_content_format="auto",
        request_logger=None,
    )
    return serving_completion
```
**EN:** This async helper encapsulates reusable logic in `_async_serving_chat_init`. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `_async_serving_chat_init` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_async_serving_chat_init (L623-L625)
```python
def test_async_serving_chat_init():
    serving_completion = asyncio.run(_async_serving_chat_init())
    assert serving_completion.chat_template == CHAT_TEMPLATE
```
**EN:** This test validates `test_async_serving_chat_init`. The main assertion is `serving_completion.chat_template == CHAT_TEMPLATE`.
**CN:** 这个测试验证 `test_async_serving_chat_init`。 核心断言是 `serving_completion.chat_template == CHAT_TEMPLATE`。

### Test / 测试: test_serving_chat_returns_correct_model_name (L628-L654)
```python
@pytest.mark.asyncio
async def test_serving_chat_returns_correct_model_name():
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_chat = _build_serving_chat(mock_engine)
    messages = [{"role": "user", "content": "what is 1+1?"}]

    async def return_model_name(*args):
        return args[3]

    serving_chat.chat_completion_full_generator = return_model_name

    # Test that full name is returned when short name is requested
    req = ChatCompletionRequest(model=MODEL_NAME_SHORT, messages=messages)
    assert await serving_chat.create_chat_completion(req) == MODEL_NAME

    # Test that full name is returned when empty string is specified
    req = ChatCompletionRequest(model="", messages=messages)
    assert await serving_chat.create_chat_completion(req) == MODEL_NAME

    # Test that full name is returned when no model is specified
    req = ChatCompletionRequest(messages=messages)
    assert await serving_chat.create_chat_completion(req) == MODEL_NAME
```
**EN:** This async test validates `test_serving_chat_returns_correct_model_name`. Relevant pytest markers include `asyncio`. It drives client-facing request creation through the API surface under test. The main assertion is `await serving_chat.create_chat_completion(req) == MODEL_NAME` and `await serving_chat.create_chat_completion(req) == MODEL_NAME`.
**CN:** 这个异步测试验证 `test_serving_chat_returns_correct_model_name`。 相关的 pytest 标记包括 `asyncio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `await serving_chat.create_chat_completion(req) == MODEL_NAME` and `await serving_chat.create_chat_completion(req) == MODEL_NAME`。

### Test / 测试: test_serving_chat_should_set_correct_max_tokens (L657-L807)
```python
@pytest.mark.asyncio
async def test_serving_chat_should_set_correct_max_tokens():
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_chat = _build_serving_chat(mock_engine)

    req = ChatCompletionRequest(
        model=MODEL_NAME,
        messages=[{"role": "user", "content": "what is 1+1?"}],
    )

    with suppress(Exception):
        await serving_chat.create_chat_completion(req)

# ... 125 lines omitted for brevity ...

    # Test Case 3: Request's max_tokens set lower than server accepts
    req.max_tokens = 5

    with suppress(Exception):
        await serving_chat.create_chat_completion(req)

    assert mock_engine.generate.call_args.args[1].max_tokens == 5
```
**EN:** This async test validates `test_serving_chat_should_set_correct_max_tokens`. Relevant pytest markers include `asyncio`. It drives client-facing request creation through the API surface under test. The main assertion is `mock_engine.generate.call_args.args[1].max_tokens == 93` and `mock_engine.generate.call_args.args[1].max_tokens == 10`.
**CN:** 这个异步测试验证 `test_serving_chat_should_set_correct_max_tokens`。 相关的 pytest 标记包括 `asyncio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `mock_engine.generate.call_args.args[1].max_tokens == 93` and `mock_engine.generate.call_args.args[1].max_tokens == 10`。

### Test / 测试: test_serving_chat_truncate_prompt_tokens_max_token_accounting (L810-L858)
```python
@pytest.mark.asyncio
async def test_serving_chat_truncate_prompt_tokens_max_token_accounting():
    """When truncate_prompt_tokens is set, max_tokens must be calculated using
    the truncated prompt length, not the original prompt length.

    Regression: without the fix, get_max_tokens received the untruncated prompt
    length, causing the output budget to be underestimated.
    """
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_chat = _build_serving_chat(mock_engine)

    # "what is 1+1?" tokenizes to 7 tokens with the test chat template
    # (max_model_len=100 -> max_tokens = 93 without truncation, confirmed by
# ... 23 lines omitted for brevity ...
    req = ChatCompletionRequest(
        model=MODEL_NAME,
        messages=messages,
        truncate_prompt_tokens=-1,
    )
    with suppress(Exception):
        await serving_chat.create_chat_completion(req)
    assert mock_engine.generate.call_args.args[1].max_tokens == 93
```
**EN:** This async test validates `test_serving_chat_truncate_prompt_tokens_max_token_accounting`. Relevant pytest markers include `asyncio`. It drives client-facing request creation through the API surface under test. The main assertion is `mock_engine.generate.call_args.args[1].max_tokens == 93` and `mock_engine.generate.call_args.args[1].max_tokens == 95`.
**CN:** 这个异步测试验证 `test_serving_chat_truncate_prompt_tokens_max_token_accounting`。 相关的 pytest 标记包括 `asyncio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `mock_engine.generate.call_args.args[1].max_tokens == 93` and `mock_engine.generate.call_args.args[1].max_tokens == 95`。

### Test / 测试: test_serving_chat_mistral_token_ids_prompt_is_validated (L861-L897)
```python
@pytest.mark.asyncio
async def test_serving_chat_mistral_token_ids_prompt_is_validated():
    """Regression test: when the Mistral tokenizer path returns token IDs
    directly, we must still apply input length + max_tokens validation.
    """

    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig(skip_tokenizer_init=True)
    mock_engine.input_processor = MagicMock()

    mock_tokenizer = MagicMock(spec=MistralTokenizer)
    mock_renderer = MistralRenderer(
        MockVllmConfig(mock_engine.model_config, parallel_config=MockParallelConfig()),
        tokenizer=mock_tokenizer,
    )
    # Force the Mistral chat template renderer to return token IDs.
    # Choose a prompt length that is < max_model_len, but large enough that
# ... 11 lines omitted for brevity ...
    req = ChatCompletionRequest(
        model=MODEL_NAME,
        messages=[{"role": "user", "content": "what is 1+1?"}],
        max_tokens=10,
    )

    with pytest.raises(VLLMValidationError):
        await serving_chat.create_chat_completion(req)
```
**EN:** This async test validates `test_serving_chat_mistral_token_ids_prompt_is_validated`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_serving_chat_mistral_token_ids_prompt_is_validated`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_serving_chat_mistral_token_ids_prompt_too_long_is_rejected (L900-L937)
```python
@pytest.mark.asyncio
async def test_serving_chat_mistral_token_ids_prompt_too_long_is_rejected():
    """Regression test: MistralTokenizer token-id prompts must still enforce
    the max context length for the input itself (token_num >= max_model_len).
    """

    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig(skip_tokenizer_init=True)
    mock_engine.input_processor = MagicMock()

    mock_tokenizer = MagicMock(spec=MistralTokenizer)
    mock_renderer = MistralRenderer(
        MockVllmConfig(mock_engine.model_config, parallel_config=MockParallelConfig()),
        tokenizer=mock_tokenizer,
    )
    # prompt_token_ids length == max_model_len should be rejected for
    # completion-like requests (ChatCompletionRequest).
# ... 12 lines omitted for brevity ...
    req = ChatCompletionRequest(
        model=MODEL_NAME,
        messages=[{"role": "user", "content": "what is 1+1?"}],
        max_tokens=1,
    )

    with pytest.raises(VLLMValidationError):
        await serving_chat.create_chat_completion(req)
```
**EN:** This async test validates `test_serving_chat_mistral_token_ids_prompt_too_long_is_rejected`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_serving_chat_mistral_token_ids_prompt_too_long_is_rejected`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_serving_chat_could_load_correct_generation_config (L940-L984)
```python
@pytest.mark.asyncio
async def test_serving_chat_could_load_correct_generation_config():
    mock_model_config = MockModelConfig()
    mock_model_config.diff_sampling_param = {
        "temperature": 0.5,
        "repetition_penalty": 1.05,
    }

    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = mock_model_config
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    # Initialize the serving chat
    serving_chat = _build_serving_chat(mock_engine)

    req = ChatCompletionRequest(
# ... 19 lines omitted for brevity ...
    # Test When temperature==0.0
    req.temperature = 0.0

    with suppress(Exception):
        await serving_chat.create_chat_completion(req)

    assert mock_engine.generate.call_args.args[1].temperature == 0.0
    assert mock_engine.generate.call_args.args[1].repetition_penalty == 1.05
```
**EN:** This async test validates `test_serving_chat_could_load_correct_generation_config`. Relevant pytest markers include `asyncio`. It drives client-facing request creation through the API surface under test. The main assertion is `mock_engine.generate.call_args.args[1].temperature == 0.5` and `mock_engine.generate.call_args.args[1].repetition_penalty == 1.05`.
**CN:** 这个异步测试验证 `test_serving_chat_could_load_correct_generation_config`。 相关的 pytest 标记包括 `asyncio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `mock_engine.generate.call_args.args[1].temperature == 0.5` and `mock_engine.generate.call_args.args[1].repetition_penalty == 1.05`。

### Test / 测试: test_serving_chat_did_set_correct_cache_salt (L987-L1036)
```python
@pytest.mark.parametrize("model_type", ["gpt_oss", "any"])
@pytest.mark.asyncio
async def test_serving_chat_did_set_correct_cache_salt(model_type):
    mock_model_config = MockModelConfig()
    mock_model_config.hf_config.model_type = model_type

    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = mock_model_config
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_chat = _build_serving_chat(mock_engine)

    orig_render_chat_request = serving_chat.render_chat_request
    captured_inputs = []

    async def render_chat_request(request):
# ... 24 lines omitted for brevity ...

    # Test with certain cache_salt
    req.cache_salt = "test_salt"
    with suppress(Exception):
        await serving_chat.create_chat_completion(req)

    assert len(captured_inputs) == 1
    assert captured_inputs[0]["cache_salt"] == "test_salt"
```
**EN:** This async test validates `test_serving_chat_did_set_correct_cache_salt`. It uses parameterization over `model_type`. Relevant pytest markers include `asyncio`. Key inputs are `model_type`. It drives client-facing request creation through the API surface under test. The main assertion is `len(captured_inputs) == 1` and `'cache_salt' not in captured_inputs[0]`.
**CN:** 这个异步测试验证 `test_serving_chat_did_set_correct_cache_salt`。 它通过参数化组合 `model_type`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `model_type`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(captured_inputs) == 1` and `'cache_salt' not in captured_inputs[0]`。

### Test / 测试: test_serving_chat_data_parallel_rank_extraction (L1039-L1111)
```python
@pytest.mark.asyncio
async def test_serving_chat_data_parallel_rank_extraction():
    """Test that data_parallel_rank is properly extracted from header and
    passed to engine."""
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    # Mock the generate method to return an async generator
    async def mock_generate(*args, **kwargs):
        # Yield a fake RequestOutput
        from vllm.outputs import CompletionOutput, RequestOutput

        yield RequestOutput(
            request_id="test-request",
            prompt="test prompt",
# ... 47 lines omitted for brevity ...
    mock_raw_request_no_dp.state = MagicMock()

    with suppress(Exception):
        await serving_chat.create_chat_completion(req_no_dp, mock_raw_request_no_dp)

    # Verify that data_parallel_rank defaults to None
    assert "data_parallel_rank" in mock_engine.generate.call_args.kwargs
    assert mock_engine.generate.call_args.kwargs["data_parallel_rank"] is None
```
**EN:** This async test validates `test_serving_chat_data_parallel_rank_extraction`. Relevant pytest markers include `asyncio`. It drives client-facing request creation through the API surface under test. The main assertion is `'data_parallel_rank' in mock_engine.generate.call_args.kwargs` and `mock_engine.generate.call_args.kwargs['data_parallel_rank'] == 2`.
**CN:** 这个异步测试验证 `test_serving_chat_data_parallel_rank_extraction`。 相关的 pytest 标记包括 `asyncio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'data_parallel_rank' in mock_engine.generate.call_args.kwargs` and `mock_engine.generate.call_args.kwargs['data_parallel_rank'] == 2`。

### Class / 类: TestServingChatWithHarmony (L1114-L1763)
```python
class TestServingChatWithHarmony:
    """
    These tests ensure Chat Completion requests are being properly converted into
    Harmony messages and Harmony response messages back into Chat Completion responses.
    These tests are not exhaustive, but each one was created to cover a specific case
    that we got wrong but is now fixed.

    Any changes to the tests and their expectations may result in changes to the
    accuracy of model prompting and responses generated. It is suggested to run
    an evaluation or benchmarking suite (such as bfcl multi_turn) to understand
    any impact of changes in how we prompt Harmony models.
    """

    @pytest.fixture(params=[False, True], ids=["non_streaming", "streaming"])
# ... 628 lines omitted for brevity ...
                {"role": "user", "content": messages[0]["content"]},
                {
                    "role": "assistant",
                    "channel": "analysis",
                    "content": messages[1]["reasoning"],
                },
            ],
        )
```
**EN:** This class groups related scenarios in `TestServingChatWithHarmony`. It contains 7 test method(s) and 7 supporting method(s). Representative methods include `test_simple_chat`, `test_tool_call_response_with_content`, `test_tools_and_reasoning`, `stream`, `mock_engine`.
**CN:** 该类将与 `TestServingChatWithHarmony` 相关的场景组织在一起。 它包含 7 个测试方法和 7 个辅助方法。 代表性方法包括 `test_simple_chat`、`test_tool_call_response_with_content`、`test_tools_and_reasoning`、`stream`、`mock_engine`。

### Fixture method / 夹具方法: TestServingChatWithHarmony.stream (L1127-L1130)
```python
    @pytest.fixture(params=[False, True], ids=["non_streaming", "streaming"])
    def stream(self, request) -> bool:
        """Parameterize tests to run in both non-streaming and streaming modes."""
        return request.param
```
**EN:** This fixture prepares `TestServingChatWithHarmony.stream` for dependent tests. Key inputs are `request`.
**CN:** 这个 fixture 为依赖它的测试准备 `TestServingChatWithHarmony.stream`。 关键输入包括 `request`。

### Fixture method / 夹具方法: TestServingChatWithHarmony.mock_engine (L1132-L1139)
```python
    @pytest.fixture()
    def mock_engine(self) -> AsyncLLM:
        mock_engine = MagicMock(spec=AsyncLLM)
        mock_engine.errored = False
        mock_engine.model_config = MockModelConfig()
        mock_engine.input_processor = MagicMock()
        mock_engine.renderer = _build_renderer(mock_engine.model_config)
        return mock_engine
```
**EN:** This fixture prepares `TestServingChatWithHarmony.mock_engine` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `TestServingChatWithHarmony.mock_engine`。

### Fixture method / 夹具方法: TestServingChatWithHarmony.serving_chat (L1141-L1146)
```python
    @pytest.fixture()
    def serving_chat(self, mock_engine) -> OpenAIServingChat:
        chat = _build_serving_chat(mock_engine)
        chat.use_harmony = True
        chat.tool_parser = ToolParserManager.get_tool_parser("openai")
        return chat
```
**EN:** This fixture prepares `TestServingChatWithHarmony.serving_chat` for dependent tests. Key inputs are `mock_engine`.
**CN:** 这个 fixture 为依赖它的测试准备 `TestServingChatWithHarmony.serving_chat`。 关键输入包括 `mock_engine`。

### Helper method / 辅助方法: TestServingChatWithHarmony.mock_request_output_from_req_and_token_ids (L1148-L1166)
```python
    def mock_request_output_from_req_and_token_ids(
        self, req: ChatCompletionRequest, token_ids: list[int], finished: bool = False
    ) -> RequestOutput:
        # Our tests don't use most fields, so just get the token ids correct
        completion_output = CompletionOutput(
            index=0,
            text="",
            token_ids=token_ids,
            cumulative_logprob=0.0,
            logprobs=None,
        )
        return RequestOutput(
            request_id=req.request_id,
            prompt=[],
            prompt_token_ids=[],
            prompt_logprobs=None,
            outputs=[completion_output],
            finished=finished,
        )
```
**EN:** This helper encapsulates reusable logic in `TestServingChatWithHarmony.mock_request_output_from_req_and_token_ids`. Key inputs are `req`, `token_ids`, `finished`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestServingChatWithHarmony.mock_request_output_from_req_and_token_ids` 中。 关键输入包括 `req`、`token_ids`、`finished`。 它把计算得到的状态或辅助对象返回给调用方。

### Fixture method / 夹具方法: TestServingChatWithHarmony.weather_tools (L1168-L1185)
```python
    @pytest.fixture
    def weather_tools(self) -> list[dict[str, Any]]:
        return [
            {
                "type": "function",
                "function": {
                    "name": "get_weather",
                    "description": "Get the weather in a given location",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "location": {"type": "string"},
                        },
                        "required": ["location"],
                    },
                },
            },
        ]
```
**EN:** This fixture prepares `TestServingChatWithHarmony.weather_tools` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `TestServingChatWithHarmony.weather_tools`。

### Fixture method / 夹具方法: TestServingChatWithHarmony.weather_messages_start (L1187-L1194)
```python
    @pytest.fixture
    def weather_messages_start(self) -> list[dict[str, Any]]:
        return [
            {
                "role": "user",
                "content": "What's the weather like in Paris today?",
            },
        ]
```
**EN:** This fixture prepares `TestServingChatWithHarmony.weather_messages_start` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `TestServingChatWithHarmony.weather_messages_start`。

### Helper method / 辅助方法: TestServingChatWithHarmony.generate_response_from_harmony_str (L1196-L1240)
```python
    async def generate_response_from_harmony_str(
        self,
        serving_chat: OpenAIServingChat,
        req: ChatCompletionRequest,
        harmony_str: str,
        stream: bool = False,
    ) -> ChatCompletionResponse:
        harmony_token_ids = get_encoding().encode(harmony_str, allowed_special="all")

        async def result_generator():
            if stream:
                for token_id in harmony_token_ids:
                    yield self.mock_request_output_from_req_and_token_ids(
                        req, [token_id]
                    )
                yield self.mock_request_output_from_req_and_token_ids(
                    req, [], finished=True
                )
# ... 19 lines omitted for brevity ...
                request_id=req.request_id,
                model_name=req.model,
            ),
        )

        if stream:
            return await accumulate_streaming_response(result)
        return await result
```
**EN:** This async helper encapsulates reusable logic in `TestServingChatWithHarmony.generate_response_from_harmony_str`. Key inputs are `serving_chat`, `req`, `harmony_str`, `stream`. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `TestServingChatWithHarmony.generate_response_from_harmony_str` 中。 关键输入包括 `serving_chat`、`req`、`harmony_str`、`stream`。 它把计算得到的状态或辅助对象返回给调用方。

### Test method / 测试方法: TestServingChatWithHarmony.test_simple_chat (L1242-L1289)
```python
    @pytest.mark.asyncio
    async def test_simple_chat(self, serving_chat, stream):
        messages = [{"role": "user", "content": "what is 1+1?"}]

        # Test the Harmony messages for the first turn's input
        req = ChatCompletionRequest(model=MODEL_NAME, messages=messages)
        input_messages, _ = (
            serving_chat.openai_serving_render._make_request_with_harmony(req)
        )
        verify_harmony_messages(
            input_messages,
            [
                {"role": "system"},
                {"role": "user", "content": messages[0]["content"]},
            ],
        )

        # Test the Chat Completion response for the first turn's output
# ... 22 lines omitted for brevity ...
            [
                {"role": "system"},
                {"role": "user"},
                # The analysis message should be dropped on subsequent inputs because
                # of the subsequent assistant message to the final channel.
                {"role": "assistant", "channel": "final", "content": final_str},
            ],
        )
```
**EN:** This async test validates `TestServingChatWithHarmony.test_simple_chat`. Relevant pytest markers include `asyncio`. Key inputs are `serving_chat`, `stream`.
**CN:** 这个异步测试验证 `TestServingChatWithHarmony.test_simple_chat`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `serving_chat`、`stream`。

### Test method / 测试方法: TestServingChatWithHarmony.test_tool_call_response_with_content (L1291-L1374)
```python
    @pytest.mark.asyncio
    async def test_tool_call_response_with_content(
        self, serving_chat, stream, weather_tools, weather_messages_start
    ):
        tools = weather_tools
        messages = list(weather_messages_start)

        # Test the Harmony messages for the first turn's input
        req = ChatCompletionRequest(model=MODEL_NAME, messages=messages, tools=tools)
        input_messages, _ = (
            serving_chat.openai_serving_render._make_request_with_harmony(req)
        )
        verify_harmony_messages(
            input_messages,
            [
                {"role": "system"},
                {"role": "developer", "tool_definitions": ["get_weather"]},
                {"role": "user", "content": messages[0]["content"]},
# ... 58 lines omitted for brevity ...
                    "role": "tool",
                    "author_name": "functions.get_weather",
                    "channel": "commentary",
                    "recipient": "assistant",
                    "content": "20 degrees Celsius",
                },
            ],
        )
```
**EN:** This async test validates `TestServingChatWithHarmony.test_tool_call_response_with_content`. Relevant pytest markers include `asyncio`. Key inputs are `serving_chat`, `stream`, `weather_tools`, `weather_messages_start`.
**CN:** 这个异步测试验证 `TestServingChatWithHarmony.test_tool_call_response_with_content`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `serving_chat`、`stream`、`weather_tools`、`weather_messages_start`。

### Test method / 测试方法: TestServingChatWithHarmony.test_tools_and_reasoning (L1376-L1459)
```python
    @pytest.mark.asyncio
    async def test_tools_and_reasoning(
        self, serving_chat, stream, weather_tools, weather_messages_start
    ):
        tools = weather_tools
        messages = list(weather_messages_start)

        # Test the Harmony messages for the first turn's input
        req = ChatCompletionRequest(model=MODEL_NAME, messages=messages, tools=tools)
        input_messages, _ = (
            serving_chat.openai_serving_render._make_request_with_harmony(req)
        )
        verify_harmony_messages(
            input_messages,
            [
                {"role": "system"},
                {"role": "developer", "tool_definitions": ["get_weather"]},
                {"role": "user", "content": messages[0]["content"]},
# ... 58 lines omitted for brevity ...
                    "role": "tool",
                    "author_name": "functions.get_weather",
                    "channel": "commentary",
                    "recipient": "assistant",
                    "content": "20 degrees Celsius",
                },
            ],
        )
```
**EN:** This async test validates `TestServingChatWithHarmony.test_tools_and_reasoning`. Relevant pytest markers include `asyncio`. Key inputs are `serving_chat`, `stream`, `weather_tools`, `weather_messages_start`.
**CN:** 这个异步测试验证 `TestServingChatWithHarmony.test_tools_and_reasoning`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `serving_chat`、`stream`、`weather_tools`、`weather_messages_start`。

### Test method / 测试方法: TestServingChatWithHarmony.test_multi_turn_tools_and_reasoning (L1461-L1668)
```python
    @pytest.mark.asyncio
    async def test_multi_turn_tools_and_reasoning(
        self, serving_chat, stream, weather_tools, weather_messages_start
    ):
        tools = weather_tools
        messages = list(weather_messages_start)

        # Test the Harmony messages for the first turn's input
        req = ChatCompletionRequest(model=MODEL_NAME, messages=messages, tools=tools)
        input_messages, _ = (
            serving_chat.openai_serving_render._make_request_with_harmony(req)
        )
        verify_harmony_messages(
            input_messages,
            [
                {"role": "system"},
                {"role": "developer", "tool_definitions": ["get_weather"]},
                {"role": "user", "content": messages[0]["content"]},
# ... 182 lines omitted for brevity ...
                    "role": "tool",
                    "author_name": "functions.get_weather",
                    "channel": "commentary",
                    "recipient": "assistant",
                    "content": "10 degrees Celsius",
                },
            ],
        )
```
**EN:** This async test validates `TestServingChatWithHarmony.test_multi_turn_tools_and_reasoning`. Relevant pytest markers include `asyncio`. Key inputs are `serving_chat`, `stream`, `weather_tools`, `weather_messages_start`.
**CN:** 这个异步测试验证 `TestServingChatWithHarmony.test_multi_turn_tools_and_reasoning`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `serving_chat`、`stream`、`weather_tools`、`weather_messages_start`。

### Test method / 测试方法: TestServingChatWithHarmony.test_non_tool_reasoning (L1670-L1701)
```python
    @pytest.mark.asyncio
    async def test_non_tool_reasoning(self, serving_chat):
        messages: list[dict[str, Any]] = [
            {
                "role": "user",
                "content": "What's 2+2?",
            },
            {
                "role": "assistant",
                "reasoning": "Adding 2 and 2 is easy. The result is 4.",
                "content": "4",
            },
        ]
        req = ChatCompletionRequest(model=MODEL_NAME, messages=messages)
        input_messages, _ = (
            serving_chat.openai_serving_render._make_request_with_harmony(req)
        )

# ... 6 lines omitted for brevity ...
                # dropped because of a later assistant message to the final channel.
                {
                    "role": "assistant",
                    "channel": "final",
                    "content": messages[1]["content"],
                },
            ],
        )
```
**EN:** This async test validates `TestServingChatWithHarmony.test_non_tool_reasoning`. Relevant pytest markers include `asyncio`. Key inputs are `serving_chat`.
**CN:** 这个异步测试验证 `TestServingChatWithHarmony.test_non_tool_reasoning`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `serving_chat`。

### Test method / 测试方法: TestServingChatWithHarmony.test_non_tool_reasoning_empty_content (L1703-L1732)
```python
    @pytest.mark.asyncio
    async def test_non_tool_reasoning_empty_content(self, serving_chat):
        messages: list[dict[str, Any]] = [
            {
                "role": "user",
                "content": "What's 2+2?",
            },
            {
                "role": "assistant",
                "reasoning": "Adding 2 and 2 is easy. The result is 4.",
                "content": "",
            },
        ]
        req = ChatCompletionRequest(model=MODEL_NAME, messages=messages)
        input_messages, _ = (
            serving_chat.openai_serving_render._make_request_with_harmony(req)
        )

        verify_harmony_messages(
            input_messages,
            [
                {"role": "system"},
                {"role": "user", "content": messages[0]["content"]},
                {
                    "role": "assistant",
                    "channel": "analysis",
                    "content": messages[1]["reasoning"],
                },
            ],
        )
```
**EN:** This async test validates `TestServingChatWithHarmony.test_non_tool_reasoning_empty_content`. Relevant pytest markers include `asyncio`. Key inputs are `serving_chat`.
**CN:** 这个异步测试验证 `TestServingChatWithHarmony.test_non_tool_reasoning_empty_content`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `serving_chat`。

### Test method / 测试方法: TestServingChatWithHarmony.test_non_tool_reasoning_empty_content_list (L1734-L1763)
```python
    @pytest.mark.asyncio
    async def test_non_tool_reasoning_empty_content_list(self, serving_chat):
        messages: list[dict[str, Any]] = [
            {
                "role": "user",
                "content": "What's 2+2?",
            },
            {
                "role": "assistant",
                "reasoning": "Adding 2 and 2 is easy. The result is 4.",
                "content": [],
            },
        ]
        req = ChatCompletionRequest(model=MODEL_NAME, messages=messages)
        input_messages, _ = (
            serving_chat.openai_serving_render._make_request_with_harmony(req)
        )

        verify_harmony_messages(
            input_messages,
            [
                {"role": "system"},
                {"role": "user", "content": messages[0]["content"]},
                {
                    "role": "assistant",
                    "channel": "analysis",
                    "content": messages[1]["reasoning"],
                },
            ],
        )
```
**EN:** This async test validates `TestServingChatWithHarmony.test_non_tool_reasoning_empty_content_list`. Relevant pytest markers include `asyncio`. Key inputs are `serving_chat`.
**CN:** 这个异步测试验证 `TestServingChatWithHarmony.test_non_tool_reasoning_empty_content_list`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `serving_chat`。

### Test / 测试: test_tool_choice_validation_without_parser (L1766-L1831)
```python
@pytest.mark.asyncio
async def test_tool_choice_validation_without_parser():
    """Test that tool_choice='required' or named tool without tool_parser
    returns an appropriate error message."""
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    models = OpenAIServingModels(
        engine_client=mock_engine,
        base_model_paths=BASE_MODEL_PATHS,
    )
    openai_serving_render = _build_serving_render(mock_engine, models.registry)

    # Create serving_chat without tool_parser (enable_auto_tools=False)
    serving_chat = OpenAIServingChat(
# ... 40 lines omitted for brevity ...
        messages=[{"role": "user", "content": "What's the weather?"}],
        tools=tools,
        tool_choice={"type": "function", "function": {"name": "get_weather"}},
    )
    response_named = await serving_chat.create_chat_completion(req_named)
    assert isinstance(response_named, ErrorResponse)
    assert "tool_choice" in response_named.error.message
    assert "--tool-call-parser" in response_named.error.message
```
**EN:** This async test validates `test_tool_choice_validation_without_parser`. Relevant pytest markers include `asyncio`. It drives client-facing request creation through the API surface under test. The main assertion is `isinstance(response_required, ErrorResponse)` and `'tool_choice' in response_required.error.message`.
**CN:** 这个异步测试验证 `test_tool_choice_validation_without_parser`。 相关的 pytest 标记包括 `asyncio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `isinstance(response_required, ErrorResponse)` and `'tool_choice' in response_required.error.message`。

### Test / 测试: test_streaming_n_gt1_independent_tool_parsers (L1834-L1994)
```python
@pytest.mark.asyncio
async def test_streaming_n_gt1_independent_tool_parsers():
    """n>1 streaming must use independent parser instances
    and token-id histories per choice.
    """
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    models = OpenAIServingModels(
        engine_client=mock_engine,
        base_model_paths=BASE_MODEL_PATHS,
    )
    openai_serving_render = _build_serving_render(mock_engine, models.registry)

    serving_chat = OpenAIServingChat(
# ... 135 lines omitted for brevity ...

        assert name == "get_weather", (
            f"Choice {choice_idx}: expected 'get_weather', got {name!r}"
        )
        parsed_args = json.loads(args_buf)
        assert parsed_args == {"city": "Tokyo"}, (
            f"Choice {choice_idx}: expected {{'city': 'Tokyo'}}, got {parsed_args}"
        )
```
**EN:** This async test validates `test_streaming_n_gt1_independent_tool_parsers`. Relevant pytest markers include `asyncio`. The main assertion is `len(deltas) > 0` and `name == 'get_weather'`.
**CN:** 这个异步测试验证 `test_streaming_n_gt1_independent_tool_parsers`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `len(deltas) > 0` and `name == 'get_weather'`。

### Class / 类: TestCreateRemainingArgsDelta (L1997-L2133)
```python
class TestCreateRemainingArgsDelta:
    """Tests for _create_remaining_args_delta helper function.

    This helper is used when streaming tool calls to preserve id/type/name
    fields in the finish chunk, which would otherwise be lost.
    """

    def test_preserves_id_type_name(self):
        """Test that id, type, and name are preserved from original delta."""
        from vllm.entrypoints.openai.chat_completion.serving import OpenAIServingChat
        from vllm.entrypoints.openai.engine.protocol import (
            DeltaFunctionCall,
            DeltaMessage,
            DeltaToolCall,
# ... 115 lines omitted for brevity ...

        assert len(result.tool_calls) == 1
        tc = result.tool_calls[0]
        assert tc.index == 0
        assert tc.id == "call_nofunc"
        assert tc.type == "function"
        assert tc.function.name is None
        assert tc.function.arguments == '{"data": "value"}'
```
**EN:** This class groups related scenarios in `TestCreateRemainingArgsDelta`. It contains 4 test method(s) and 0 supporting method(s). Representative methods include `test_preserves_id_type_name`, `test_matches_by_index`, `test_no_matching_tool_call`.
**CN:** 该类将与 `TestCreateRemainingArgsDelta` 相关的场景组织在一起。 它包含 4 个测试方法和 0 个辅助方法。 代表性方法包括 `test_preserves_id_type_name`、`test_matches_by_index`、`test_no_matching_tool_call`。

### Test method / 测试方法: TestCreateRemainingArgsDelta.test_preserves_id_type_name (L2004-L2037)
```python
    def test_preserves_id_type_name(self):
        """Test that id, type, and name are preserved from original delta."""
        from vllm.entrypoints.openai.chat_completion.serving import OpenAIServingChat
        from vllm.entrypoints.openai.engine.protocol import (
            DeltaFunctionCall,
            DeltaMessage,
            DeltaToolCall,
        )

        original_delta = DeltaMessage(
            tool_calls=[
                DeltaToolCall(
                    index=0,
                    id="call_abc123",
                    type="function",
                    function=DeltaFunctionCall(
                        name="get_weather",
                        arguments='{"location": "Paris"}',
# ... 8 lines omitted for brevity ...

        assert len(result.tool_calls) == 1
        tc = result.tool_calls[0]
        assert tc.index == 0
        assert tc.id == "call_abc123"
        assert tc.type == "function"
        assert tc.function.name == "get_weather"
        assert tc.function.arguments == '", "unit": "celsius"}'
```
**EN:** This test validates `TestCreateRemainingArgsDelta.test_preserves_id_type_name`. The main assertion is `len(result.tool_calls) == 1` and `tc.index == 0`.
**CN:** 这个测试验证 `TestCreateRemainingArgsDelta.test_preserves_id_type_name`。 核心断言是 `len(result.tool_calls) == 1` and `tc.index == 0`。

### Test method / 测试方法: TestCreateRemainingArgsDelta.test_matches_by_index (L2039-L2073)
```python
    def test_matches_by_index(self):
        """Test that the correct tool call is matched by index."""
        from vllm.entrypoints.openai.chat_completion.serving import OpenAIServingChat
        from vllm.entrypoints.openai.engine.protocol import (
            DeltaFunctionCall,
            DeltaMessage,
            DeltaToolCall,
        )

        original_delta = DeltaMessage(
            tool_calls=[
                DeltaToolCall(
                    index=0,
                    id="call_first",
                    type="function",
                    function=DeltaFunctionCall(name="func_a", arguments="{}"),
                ),
                DeltaToolCall(
# ... 9 lines omitted for brevity ...
            original_delta, '{"extra": true}', 1
        )

        assert len(result.tool_calls) == 1
        tc = result.tool_calls[0]
        assert tc.index == 1
        assert tc.id == "call_second"
        assert tc.function.name == "func_b"
```
**EN:** This test validates `TestCreateRemainingArgsDelta.test_matches_by_index`. The main assertion is `len(result.tool_calls) == 1` and `tc.index == 1`.
**CN:** 这个测试验证 `TestCreateRemainingArgsDelta.test_matches_by_index`。 核心断言是 `len(result.tool_calls) == 1` and `tc.index == 1`。

### Test method / 测试方法: TestCreateRemainingArgsDelta.test_no_matching_tool_call (L2075-L2105)
```python
    def test_no_matching_tool_call(self):
        """Test graceful handling when no matching tool call is found."""
        from vllm.entrypoints.openai.chat_completion.serving import OpenAIServingChat
        from vllm.entrypoints.openai.engine.protocol import (
            DeltaFunctionCall,
            DeltaMessage,
            DeltaToolCall,
        )

        original_delta = DeltaMessage(
            tool_calls=[
                DeltaToolCall(
                    index=0,
                    id="call_zero",
                    type="function",
                    function=DeltaFunctionCall(name="func", arguments="{}"),
                )
            ]
# ... 5 lines omitted for brevity ...

        assert len(result.tool_calls) == 1
        tc = result.tool_calls[0]
        assert tc.index == 5
        assert tc.id is None
        assert tc.type is None
        assert tc.function.name is None
        assert tc.function.arguments == '{"arg": 1}'
```
**EN:** This test validates `TestCreateRemainingArgsDelta.test_no_matching_tool_call`. The main assertion is `len(result.tool_calls) == 1` and `tc.index == 5`.
**CN:** 这个测试验证 `TestCreateRemainingArgsDelta.test_no_matching_tool_call`。 核心断言是 `len(result.tool_calls) == 1` and `tc.index == 5`。

### Test method / 测试方法: TestCreateRemainingArgsDelta.test_function_is_none (L2107-L2133)
```python
    def test_function_is_none(self):
        """Test handling when original tool call has no function."""
        from vllm.entrypoints.openai.chat_completion.serving import OpenAIServingChat
        from vllm.entrypoints.openai.engine.protocol import DeltaMessage, DeltaToolCall

        original_delta = DeltaMessage(
            tool_calls=[
                DeltaToolCall(
                    index=0,
                    id="call_nofunc",
                    type="function",
                    function=None,
                )
            ]
        )

        result = OpenAIServingChat._create_remaining_args_delta(
            original_delta, '{"data": "value"}', 0
        )

        assert len(result.tool_calls) == 1
        tc = result.tool_calls[0]
        assert tc.index == 0
        assert tc.id == "call_nofunc"
        assert tc.type == "function"
        assert tc.function.name is None
        assert tc.function.arguments == '{"data": "value"}'
```
**EN:** This test validates `TestCreateRemainingArgsDelta.test_function_is_none`. The main assertion is `len(result.tool_calls) == 1` and `tc.index == 0`.
**CN:** 这个测试验证 `TestCreateRemainingArgsDelta.test_function_is_none`。 核心断言是 `len(result.tool_calls) == 1` and `tc.index == 0`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `contextlib.suppress`, `dataclasses.dataclass`, `dataclasses.field`, `json`, `typing.Any`, `unittest.mock.AsyncMock`, `unittest.mock.MagicMock`
- **Third-party / 第三方**: `_pytest.monkeypatch.MonkeyPatch`, `openai.OpenAI`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.entrypoints.openai.utils.accumulate_streaming_response`, `tests.entrypoints.openai.utils.verify_chat_response`, `tests.entrypoints.openai.utils.verify_harmony_messages`, `tests.utils.RemoteOpenAIServer`, `vllm._aiter_ops.is_aiter_found_and_supported`, `vllm.config.MultiModalConfig`, `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionRequest`, `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionResponse`, `vllm.entrypoints.openai.chat_completion.serving.OpenAIServingChat`, `vllm.entrypoints.openai.engine.protocol.DeltaFunctionCall`, `vllm.entrypoints.openai.engine.protocol.DeltaMessage`, `vllm.entrypoints.openai.engine.protocol.DeltaToolCall`, `vllm.entrypoints.openai.engine.protocol.ErrorResponse`, `vllm.entrypoints.openai.engine.protocol.RequestResponseMetadata`, `vllm.entrypoints.openai.models.serving.BaseModelPath`, `vllm.entrypoints.openai.models.serving.OpenAIModelRegistry`, `vllm.entrypoints.openai.models.serving.OpenAIServingModels`, `vllm.entrypoints.openai.parser.harmony_utils.get_encoding`, `vllm.entrypoints.serve.render.serving.OpenAIServingRender`, `vllm.exceptions.VLLMValidationError`, `vllm.inputs.TokensPrompt`, `vllm.outputs.CompletionOutput`, `vllm.outputs.RequestOutput`, `vllm.renderers.hf.HfRenderer`, `vllm.renderers.mistral.MistralRenderer`, `vllm.tokenizers.get_tokenizer`, `vllm.tokenizers.mistral.MistralTokenizer`, `vllm.tokenizers.registry.cached_tokenizer_from_config`, `vllm.tool_parsers.ToolParserManager`, `vllm.v1.engine.async_llm.AsyncLLM`
