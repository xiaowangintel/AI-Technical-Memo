# test_generate_stream.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/serve/disagg/test_generate_stream.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers serve subsystem behavior. The file defines 9 test(s), 0 fixture(s), and 20 helper/class block(s) to validate this area. / [CN] 该文件覆盖服务子系统行为。它定义了 9 个测试、0 个 fixture，以及 20 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L25)
```python
import json
from dataclasses import dataclass, field
from typing import Any
from unittest.mock import AsyncMock, MagicMock

import pytest

from vllm.config.multimodal import MultiModalConfig
from vllm.entrypoints.openai.engine.protocol import StreamOptions
from vllm.entrypoints.openai.models.protocol import BaseModelPath
from vllm.entrypoints.openai.models.serving import OpenAIServingModels
from vllm.entrypoints.serve.disagg.protocol import (
    GenerateRequest,
    GenerateResponse,
)
from vllm.entrypoints.serve.disagg.serving import ServingTokens
from vllm.entrypoints.serve.render.serving import OpenAIServingRender
from vllm.logprobs import Logprob
from vllm.outputs import CompletionOutput, RequestOutput
from vllm.renderers import renderer_from_config
from vllm.sampling_params import SamplingParams
from vllm.v1.engine.async_llm import AsyncLLM
```
**EN:** Imports standard-library modules such as `dataclasses.dataclass`, `dataclasses.field`, `json`, third-party packages like `pytest`, project helpers such as `vllm.config.multimodal.MultiModalConfig`, `vllm.entrypoints.openai.engine.protocol.StreamOptions`, `vllm.entrypoints.openai.models.protocol.BaseModelPath`.
**CN:** 导入标准库模块（如 `dataclasses.dataclass`、`dataclasses.field`、`json`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.config.multimodal.MultiModalConfig`、`vllm.entrypoints.openai.engine.protocol.StreamOptions`、`vllm.entrypoints.openai.models.protocol.BaseModelPath`）。

### Module setup / 模块级配置: MODEL_NAME, BASE_MODEL_PATHS (L27-L30)
```python
MODEL_NAME = "openai-community/gpt2"
BASE_MODEL_PATHS = [
    BaseModelPath(name=MODEL_NAME, model_path=MODEL_NAME),
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `BASE_MODEL_PATHS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`BASE_MODEL_PATHS`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: MockHFConfig (L33-L35)
```python
@dataclass
class MockHFConfig:
    model_type: str = "any"
```
**EN:** This class groups related scenarios in `MockHFConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockHFConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: MockModelConfig (L38-L64)
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
# ... 5 lines omitted for brevity ...
    media_io_kwargs: dict[str, dict[str, Any]] = field(default_factory=dict)
    skip_tokenizer_init = False
    is_encoder_decoder: bool = False
    is_multimodal_model: bool = False
    renderer_num_workers: int = 1

    def get_diff_sampling_param(self):
        return self.diff_sampling_param or {}
```
**EN:** This class groups related scenarios in `MockModelConfig`. Decorators such as `@dataclass` make it a compact metadata container. It contains 0 test method(s) and 1 supporting method(s). Representative methods include `get_diff_sampling_param`.
**CN:** 该类将与 `MockModelConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 它包含 0 个测试方法和 1 个辅助方法。 代表性方法包括 `get_diff_sampling_param`。

### Helper method / 辅助方法: MockModelConfig.get_diff_sampling_param (L63-L64)
```python
    def get_diff_sampling_param(self):
        return self.diff_sampling_param or {}
```
**EN:** This helper encapsulates reusable logic in `MockModelConfig.get_diff_sampling_param`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `MockModelConfig.get_diff_sampling_param` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: MockParallelConfig (L67-L69)
```python
@dataclass
class MockParallelConfig:
    _api_process_rank: int = 0
```
**EN:** This class groups related scenarios in `MockParallelConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockParallelConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: MockVllmConfig (L72-L75)
```python
@dataclass
class MockVllmConfig:
    model_config: MockModelConfig
    parallel_config: MockParallelConfig
```
**EN:** This class groups related scenarios in `MockVllmConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockVllmConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Helper / 辅助函数: _build_renderer (L78-L81)
```python
def _build_renderer(model_config: MockModelConfig):
    return renderer_from_config(
        MockVllmConfig(model_config, parallel_config=MockParallelConfig()),
    )
```
**EN:** This helper encapsulates reusable logic in `_build_renderer`. Key inputs are `model_config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_renderer` 中。 关键输入包括 `model_config`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _build_serving_tokens (L84-L111)
```python
def _build_serving_tokens(engine: AsyncLLM, **kwargs) -> ServingTokens:
    models = OpenAIServingModels(
        engine_client=engine,
        base_model_paths=BASE_MODEL_PATHS,
    )
    serving_render = OpenAIServingRender(
        model_config=engine.model_config,
        renderer=engine.renderer,
        model_registry=models.registry,
        request_logger=None,
        chat_template=None,
        chat_template_content_format="auto",
    )
    serving = ServingTokens(
        engine,
        models,
        openai_serving_render=serving_render,
        request_logger=None,
        **kwargs,
    )

    async def _fake_preprocess(*args, **kwargs):
        return [{"prompt_token_ids": [1, 2, 3]}]

    serving.openai_serving_render.preprocess_completion = AsyncMock(
        side_effect=_fake_preprocess
    )
    return serving
```
**EN:** This helper encapsulates reusable logic in `_build_serving_tokens`. Key inputs are `engine`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_serving_tokens` 中。 关键输入包括 `engine`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _make_request_output (L114-L145)
```python
def _make_request_output(
    request_id: str,
    token_ids: list[int],
    finish_reason: str | None = None,
    finished: bool = False,
    prompt_token_ids: list[int] | None = None,
    logprobs: list[dict[int, Any] | None] | None = None,
    num_cached_tokens: int | None = None,
    index: int = 0,
) -> RequestOutput:
    return RequestOutput(
        request_id=request_id,
        prompt=None,
        prompt_token_ids=prompt_token_ids or [1, 2, 3],
        prompt_logprobs=None,
        outputs=[
            CompletionOutput(
                index=index,
# ... 6 lines omitted for brevity ...
        ],
        finished=finished,
        metrics=None,
        lora_request=None,
        encoder_prompt=None,
        encoder_prompt_token_ids=None,
        num_cached_tokens=num_cached_tokens,
    )
```
**EN:** This helper encapsulates reusable logic in `_make_request_output`. Key inputs are `request_id`, `token_ids`, `finish_reason`, `finished`, `prompt_token_ids`, `logprobs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_request_output` 中。 关键输入包括 `request_id`、`token_ids`、`finish_reason`、`finished`、`prompt_token_ids`、`logprobs`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _mock_engine (L148-L154)
```python
def _mock_engine() -> MagicMock:
    engine = MagicMock(spec=AsyncLLM)
    engine.errored = False
    engine.model_config = MockModelConfig()
    engine.input_processor = MagicMock()
    engine.renderer = _build_renderer(engine.model_config)
    return engine
```
**EN:** This helper encapsulates reusable logic in `_mock_engine`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_mock_engine` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _parse_sse_chunks (L157-L167)
```python
def _parse_sse_chunks(chunks: list[str]) -> list[Any]:
    """Parse SSE chunks into dicts (JSON) or raw strings ([DONE])."""
    parsed: list[Any] = []
    for chunk in chunks:
        assert chunk.startswith("data: ") and chunk.endswith("\n\n")
        payload = chunk[len("data: ") : -len("\n\n")]
        if payload == "[DONE]":
            parsed.append("[DONE]")
        else:
            parsed.append(json.loads(payload))
    return parsed
```
**EN:** This helper encapsulates reusable logic in `_parse_sse_chunks`. Key inputs are `chunks`. It returns computed state or helper objects back to the caller. The main assertion is `chunk.startswith('data: ') and chunk.endswith('\n\n')`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_parse_sse_chunks` 中。 关键输入包括 `chunks`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `chunk.startswith('data: ') and chunk.endswith('\n\n')`。

### Test / 测试: test_serve_tokens_skips_mm_cache_for_remote_engine_execution (L170-L197)
```python
@pytest.mark.asyncio
async def test_serve_tokens_skips_mm_cache_for_remote_engine_execution():
    engine = _mock_engine()

    async def mock_generate(*args, **kwargs):
        yield _make_request_output(
            "req-1", token_ids=[10], finish_reason="stop", finished=True
        )

    engine.generate = MagicMock(side_effect=mock_generate)
    serving = _build_serving_tokens(engine)

    request = GenerateRequest(
        token_ids=[1, 2, 3],
        sampling_params=SamplingParams(max_tokens=1),
        model=MODEL_NAME,
        stream=False,
    )

    response = await serving.serve_tokens(request)

    assert isinstance(response, GenerateResponse)
    assert (
        serving.openai_serving_render.preprocess_completion.call_args.kwargs[
            "skip_mm_cache"
        ]
        is True
    )
```
**EN:** This async test validates `test_serve_tokens_skips_mm_cache_for_remote_engine_execution`. Relevant pytest markers include `asyncio`. The main assertion is `isinstance(response, GenerateResponse)` and `serving.openai_serving_render.preprocess_completion.call_args.kwargs['skip_mm_cache'] is True`.
**CN:** 这个异步测试验证 `test_serve_tokens_skips_mm_cache_for_remote_engine_execution`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `isinstance(response, GenerateResponse)` and `serving.openai_serving_render.preprocess_completion.call_args.kwargs['skip_mm_cache'] is True`。

### Test / 测试: test_stream_basic (L200-L237)
```python
@pytest.mark.asyncio
async def test_stream_basic():
    """Streaming returns SSE chunks with correct token_ids and ends with [DONE]."""
    engine = _mock_engine()

    async def mock_generate(*args, **kwargs):
        yield _make_request_output("req-1", token_ids=[10])
        yield _make_request_output("req-1", token_ids=[20, 30])
        yield _make_request_output(
            "req-1", token_ids=[40], finish_reason="stop", finished=True
        )

    engine.generate = MagicMock(side_effect=mock_generate)
    serving = _build_serving_tokens(engine)

    request = GenerateRequest(
        token_ids=[1, 2, 3],
        sampling_params=SamplingParams(max_tokens=10),
# ... 12 lines omitted for brevity ...
    assert parsed[-1] == "[DONE]"
    data_chunks = [c for c in parsed if c != "[DONE]"]
    assert len(data_chunks) == 3

    assert data_chunks[0]["choices"][0]["token_ids"] == [10]
    assert data_chunks[1]["choices"][0]["token_ids"] == [20, 30]
    assert data_chunks[2]["choices"][0]["token_ids"] == [40]
    assert data_chunks[2]["choices"][0]["finish_reason"] == "stop"
```
**EN:** This async test validates `test_stream_basic`. Relevant pytest markers include `asyncio`. The main assertion is `parsed[-1] == '[DONE]'` and `len(data_chunks) == 3`.
**CN:** 这个异步测试验证 `test_stream_basic`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `parsed[-1] == '[DONE]'` and `len(data_chunks) == 3`。

### Test / 测试: test_stream_error_mid_generation (L240-L270)
```python
@pytest.mark.asyncio
async def test_stream_error_mid_generation():
    """finish_reason='error' mid-stream yields error chunk then [DONE]."""
    engine = _mock_engine()

    async def mock_generate(*args, **kwargs):
        yield _make_request_output("req-1", token_ids=[10])
        yield _make_request_output(
            "req-1", token_ids=[20], finish_reason="error", finished=True
        )

    engine.generate = MagicMock(side_effect=mock_generate)
    serving = _build_serving_tokens(engine)

    request = GenerateRequest(
        token_ids=[1, 2, 3],
        sampling_params=SamplingParams(max_tokens=10),
        model=MODEL_NAME,
# ... 5 lines omitted for brevity ...
    async for chunk in response:
        chunks.append(chunk)

    assert len(chunks) >= 2
    assert any("Internal server error" in chunk for chunk in chunks), (
        f"Expected error message in chunks: {chunks}"
    )
    assert chunks[-1] == "data: [DONE]\n\n"
```
**EN:** This async test validates `test_stream_error_mid_generation`. Relevant pytest markers include `asyncio`. The main assertion is `len(chunks) >= 2` and `any(('Internal server error' in chunk for chunk in chunks))`.
**CN:** 这个异步测试验证 `test_stream_error_mid_generation`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `len(chunks) >= 2` and `any(('Internal server error' in chunk for chunk in chunks))`。

### Test / 测试: test_stream_error_with_empty_delta (L273-L302)
```python
@pytest.mark.asyncio
async def test_stream_error_with_empty_delta():
    """finish_reason='error' with empty delta_token_ids still raises."""
    engine = _mock_engine()

    async def mock_generate(*args, **kwargs):
        yield _make_request_output("req-1", token_ids=[10])
        yield _make_request_output(
            "req-1", token_ids=[], finish_reason="error", finished=True
        )

    engine.generate = MagicMock(side_effect=mock_generate)
    serving = _build_serving_tokens(engine)

    request = GenerateRequest(
        token_ids=[1, 2, 3],
        sampling_params=SamplingParams(max_tokens=10),
        model=MODEL_NAME,
        stream=True,
    )

    response = await serving.serve_tokens(request)
    chunks = []
    async for chunk in response:
        chunks.append(chunk)

    assert any("Internal server error" in chunk for chunk in chunks), (
        f"Expected error message in chunks: {chunks}"
    )
    assert chunks[-1] == "data: [DONE]\n\n"
```
**EN:** This async test validates `test_stream_error_with_empty_delta`. Relevant pytest markers include `asyncio`. The main assertion is `any(('Internal server error' in chunk for chunk in chunks))` and `chunks[-1] == 'data: [DONE]\n\n'`.
**CN:** 这个异步测试验证 `test_stream_error_with_empty_delta`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `any(('Internal server error' in chunk for chunk in chunks))` and `chunks[-1] == 'data: [DONE]\n\n'`。

### Test / 测试: test_stream_skips_empty_token_output (L305-L339)
```python
@pytest.mark.asyncio
async def test_stream_skips_empty_token_output():
    """Outputs with empty token_ids are skipped (no chunk emitted)."""
    engine = _mock_engine()

    async def mock_generate(*args, **kwargs):
        yield _make_request_output("req-1", token_ids=[10])
        yield _make_request_output("req-1", token_ids=[])
        yield _make_request_output(
            "req-1", token_ids=[20], finish_reason="stop", finished=True
        )

    engine.generate = MagicMock(side_effect=mock_generate)
    serving = _build_serving_tokens(engine)

    request = GenerateRequest(
        token_ids=[1, 2, 3],
        sampling_params=SamplingParams(max_tokens=10),
# ... 9 lines omitted for brevity ...
    parsed = _parse_sse_chunks(chunks)
    assert parsed[-1] == "[DONE]"
    data_chunks = [c for c in parsed if c != "[DONE]"]

    # Only 2 data chunks — the empty one is skipped
    assert len(data_chunks) == 2
    assert data_chunks[0]["choices"][0]["token_ids"] == [10]
    assert data_chunks[1]["choices"][0]["token_ids"] == [20]
```
**EN:** This async test validates `test_stream_skips_empty_token_output`. Relevant pytest markers include `asyncio`. The main assertion is `parsed[-1] == '[DONE]'` and `len(data_chunks) == 2`.
**CN:** 这个异步测试验证 `test_stream_skips_empty_token_output`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `parsed[-1] == '[DONE]'` and `len(data_chunks) == 2`。

### Test / 测试: test_stream_include_usage (L342-L377)
```python
@pytest.mark.asyncio
async def test_stream_include_usage():
    """stream_options.include_usage emits a final usage-only chunk."""
    engine = _mock_engine()

    async def mock_generate(*args, **kwargs):
        yield _make_request_output("req-1", token_ids=[10])
        yield _make_request_output(
            "req-1", token_ids=[20], finish_reason="stop", finished=True
        )

    engine.generate = MagicMock(side_effect=mock_generate)
    serving = _build_serving_tokens(engine)

    request = GenerateRequest(
        token_ids=[1, 2, 3],
        sampling_params=SamplingParams(max_tokens=10),
        model=MODEL_NAME,
# ... 10 lines omitted for brevity ...
    assert parsed[-1] == "[DONE]"

    # The chunk before [DONE] should be the usage-only chunk
    usage_chunk = parsed[-2]
    assert usage_chunk["choices"] == []
    assert usage_chunk["usage"]["prompt_tokens"] == 3
    assert usage_chunk["usage"]["completion_tokens"] == 2
    assert usage_chunk["usage"]["total_tokens"] == 5
```
**EN:** This async test validates `test_stream_include_usage`. Relevant pytest markers include `asyncio`. The main assertion is `parsed[-1] == '[DONE]'` and `usage_chunk['choices'] == []`.
**CN:** 这个异步测试验证 `test_stream_include_usage`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `parsed[-1] == '[DONE]'` and `usage_chunk['choices'] == []`。

### Test / 测试: test_stream_continuous_usage (L380-L424)
```python
@pytest.mark.asyncio
async def test_stream_continuous_usage():
    """continuous_usage_stats adds usage to every data chunk."""
    engine = _mock_engine()

    async def mock_generate(*args, **kwargs):
        yield _make_request_output("req-1", token_ids=[10])
        yield _make_request_output(
            "req-1", token_ids=[20], finish_reason="stop", finished=True
        )

    engine.generate = MagicMock(side_effect=mock_generate)
    serving = _build_serving_tokens(engine)

    request = GenerateRequest(
        token_ids=[1, 2, 3],
        sampling_params=SamplingParams(max_tokens=10),
        model=MODEL_NAME,
# ... 19 lines omitted for brevity ...

    # First chunk: 1 completion token
    assert data_chunks[0]["usage"]["completion_tokens"] == 1
    assert data_chunks[0]["usage"]["total_tokens"] == 4

    # Second chunk: 2 completion tokens (cumulative)
    assert data_chunks[1]["usage"]["completion_tokens"] == 2
    assert data_chunks[1]["usage"]["total_tokens"] == 5
```
**EN:** This async test validates `test_stream_continuous_usage`. Relevant pytest markers include `asyncio`. The main assertion is `data_chunks[0]['usage']['completion_tokens'] == 1` and `data_chunks[0]['usage']['total_tokens'] == 4`.
**CN:** 这个异步测试验证 `test_stream_continuous_usage`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `data_chunks[0]['usage']['completion_tokens'] == 1` and `data_chunks[0]['usage']['total_tokens'] == 4`。

### Test / 测试: test_stream_with_logprobs (L427-L468)
```python
@pytest.mark.asyncio
async def test_stream_with_logprobs():
    """Streaming with logprobs includes logprob data in each chunk."""
    engine = _mock_engine()

    async def mock_generate(*args, **kwargs):
        yield _make_request_output(
            "req-1",
            token_ids=[10],
            logprobs=[{10: Logprob(logprob=-0.5)}],
        )
        yield _make_request_output(
            "req-1",
            token_ids=[20],
            logprobs=[{20: Logprob(logprob=-1.0)}],
            finish_reason="stop",
            finished=True,
        )
# ... 16 lines omitted for brevity ...
    parsed = _parse_sse_chunks(chunks)
    data_chunks = [c for c in parsed if isinstance(c, dict) and c.get("choices")]

    for dc in data_chunks:
        lp = dc["choices"][0]["logprobs"]
        assert lp is not None
        assert len(lp["content"]) == 1
        assert lp["content"][0]["token"].startswith("token_id:")
```
**EN:** This async test validates `test_stream_with_logprobs`. Relevant pytest markers include `asyncio`. The main assertion is `lp is not None` and `len(lp['content']) == 1`.
**CN:** 这个异步测试验证 `test_stream_with_logprobs`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `lp is not None` and `len(lp['content']) == 1`。

### Test / 测试: test_stream_prompt_tokens_details (L471-L505)
```python
@pytest.mark.asyncio
async def test_stream_prompt_tokens_details():
    """enable_prompt_tokens_details includes cached_tokens in final usage."""
    engine = _mock_engine()

    async def mock_generate(*args, **kwargs):
        yield _make_request_output(
            "req-1",
            token_ids=[10],
            finish_reason="stop",
            finished=True,
            num_cached_tokens=2,
        )

    engine.generate = MagicMock(side_effect=mock_generate)
    serving = _build_serving_tokens(engine, enable_prompt_tokens_details=True)

    request = GenerateRequest(
# ... 9 lines omitted for brevity ...
    async for chunk in response:
        chunks.append(chunk)

    parsed = _parse_sse_chunks(chunks)
    # Usage-only chunk (before [DONE])
    usage_chunk = parsed[-2]
    assert usage_chunk["choices"] == []
    assert usage_chunk["usage"]["prompt_tokens_details"]["cached_tokens"] == 2
```
**EN:** This async test validates `test_stream_prompt_tokens_details`. Relevant pytest markers include `asyncio`. The main assertion is `usage_chunk['choices'] == []` and `usage_chunk['usage']['prompt_tokens_details']['cached_tokens'] == 2`.
**CN:** 这个异步测试验证 `test_stream_prompt_tokens_details`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `usage_chunk['choices'] == []` and `usage_chunk['usage']['prompt_tokens_details']['cached_tokens'] == 2`。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `dataclasses.dataclass`, `dataclasses.field`, `json`, `typing.Any`, `unittest.mock.AsyncMock`, `unittest.mock.MagicMock`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.config.multimodal.MultiModalConfig`, `vllm.entrypoints.openai.engine.protocol.StreamOptions`, `vllm.entrypoints.openai.models.protocol.BaseModelPath`, `vllm.entrypoints.openai.models.serving.OpenAIServingModels`, `vllm.entrypoints.serve.disagg.protocol.GenerateRequest`, `vllm.entrypoints.serve.disagg.protocol.GenerateResponse`, `vllm.entrypoints.serve.disagg.serving.ServingTokens`, `vllm.entrypoints.serve.render.serving.OpenAIServingRender`, `vllm.logprobs.Logprob`, `vllm.outputs.CompletionOutput`, `vllm.outputs.RequestOutput`, `vllm.renderers.renderer_from_config`, `vllm.sampling_params.SamplingParams`, `vllm.v1.engine.async_llm.AsyncLLM`
