# test_completion_error.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/completion/test_completion_error.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers text completion behavior and OpenAI-compatible serving. The file defines 7 test(s), 0 fixture(s), and 9 helper/class block(s) to validate this area. / [CN] 该文件覆盖文本补全行为与OpenAI 兼容服务。它定义了 7 个测试、0 个 fixture，以及 9 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L20)
```python
from dataclasses import dataclass, field
from typing import Any
from unittest.mock import AsyncMock, MagicMock

import pytest

from vllm.config.multimodal import MultiModalConfig
from vllm.entrypoints.openai.completion.protocol import CompletionRequest
from vllm.entrypoints.openai.completion.serving import OpenAIServingCompletion
from vllm.entrypoints.openai.engine.protocol import GenerationError
from vllm.entrypoints.openai.models.protocol import BaseModelPath
from vllm.entrypoints.openai.models.serving import OpenAIServingModels
from vllm.entrypoints.serve.render.serving import OpenAIServingRender
from vllm.outputs import CompletionOutput, RequestOutput
from vllm.renderers.hf import HfRenderer
from vllm.tokenizers.registry import cached_tokenizer_from_config
from vllm.v1.engine.async_llm import AsyncLLM
```
**EN:** Imports standard-library modules such as `dataclasses.dataclass`, `dataclasses.field`, `typing.Any`, third-party packages like `pytest`, project helpers such as `vllm.config.multimodal.MultiModalConfig`, `vllm.entrypoints.openai.completion.protocol.CompletionRequest`, `vllm.entrypoints.openai.completion.serving.OpenAIServingCompletion`.
**CN:** 导入标准库模块（如 `dataclasses.dataclass`、`dataclasses.field`、`typing.Any`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.config.multimodal.MultiModalConfig`、`vllm.entrypoints.openai.completion.protocol.CompletionRequest`、`vllm.entrypoints.openai.completion.serving.OpenAIServingCompletion`）。

### Module setup / 模块级配置: MODEL_NAME, MODEL_NAME_SHORT, BASE_MODEL_PATHS (L22-L27)
```python
MODEL_NAME = "openai-community/gpt2"
MODEL_NAME_SHORT = "gpt2"
BASE_MODEL_PATHS = [
    BaseModelPath(name=MODEL_NAME, model_path=MODEL_NAME),
    BaseModelPath(name=MODEL_NAME_SHORT, model_path=MODEL_NAME_SHORT),
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MODEL_NAME_SHORT`, `BASE_MODEL_PATHS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MODEL_NAME_SHORT`、`BASE_MODEL_PATHS`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: MockHFConfig (L30-L32)
```python
@dataclass
class MockHFConfig:
    model_type: str = "any"
```
**EN:** This class groups related scenarios in `MockHFConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockHFConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: MockModelConfig (L35-L60)
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
    logits_processors: list[str] | None = None
    diff_sampling_param: dict | None = None
    allowed_local_media_path: str = ""
    allowed_media_domains: list[str] | None = None
    encoder_config = None
    generation_config: str = "auto"
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

### Helper method / 辅助方法: MockModelConfig.get_diff_sampling_param (L59-L60)
```python
    def get_diff_sampling_param(self):
        return self.diff_sampling_param or {}
```
**EN:** This helper encapsulates reusable logic in `MockModelConfig.get_diff_sampling_param`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `MockModelConfig.get_diff_sampling_param` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: MockParallelConfig (L63-L65)
```python
@dataclass
class MockParallelConfig:
    _api_process_rank: int = 0
```
**EN:** This class groups related scenarios in `MockParallelConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockParallelConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: MockVllmConfig (L68-L71)
```python
@dataclass
class MockVllmConfig:
    model_config: MockModelConfig
    parallel_config: MockParallelConfig
```
**EN:** This class groups related scenarios in `MockVllmConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockVllmConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Helper / 辅助函数: _build_serving_completion (L74-L92)
```python
def _build_serving_completion(engine: AsyncLLM) -> OpenAIServingCompletion:
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
    return OpenAIServingCompletion(
        engine,
        models,
        openai_serving_render=serving_render,
        request_logger=None,
    )
```
**EN:** This helper encapsulates reusable logic in `_build_serving_completion`. Key inputs are `engine`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_serving_completion` 中。 关键输入包括 `engine`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _build_renderer (L95-L99)
```python
def _build_renderer(model_config: MockModelConfig):
    return HfRenderer(
        MockVllmConfig(model_config, parallel_config=MockParallelConfig()),
        cached_tokenizer_from_config(model_config),
    )
```
**EN:** This helper encapsulates reusable logic in `_build_renderer`. Key inputs are `model_config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_renderer` 中。 关键输入包括 `model_config`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_completion_error_non_stream (L102-L148)
```python
@pytest.mark.asyncio
async def test_completion_error_non_stream():
    """test finish_reason='error' returns 500 InternalServerError (non-streaming)"""
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_completion = _build_serving_completion(mock_engine)

    completion_output = CompletionOutput(
        index=0,
        text="",
        token_ids=[],
        cumulative_logprob=None,
        logprobs=None,
        finish_reason="error",
# ... 21 lines omitted for brevity ...
        model=MODEL_NAME,
        prompt="Test prompt",
        max_tokens=10,
        stream=False,
    )

    with pytest.raises(GenerationError):
        await serving_completion.create_completion(request)
```
**EN:** This async test validates `test_completion_error_non_stream`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_completion_error_non_stream`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_openai_completion_keeps_mm_cache_for_engine_execution (L151-L177)
```python
@pytest.mark.asyncio
async def test_openai_completion_keeps_mm_cache_for_engine_execution():
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_completion = _build_serving_completion(mock_engine)
    serving_completion.openai_serving_render.preprocess_completion = AsyncMock(
        return_value=[{"prompt_token_ids": [1, 2, 3]}]
    )

    request = CompletionRequest(
        model=MODEL_NAME,
        prompt="Test prompt",
    )

    result = await serving_completion.render_completion_request(request)

    assert isinstance(result, list)
    assert (
        serving_completion.openai_serving_render.preprocess_completion.call_args.kwargs[
            "skip_mm_cache"
        ]
        is False
    )
```
**EN:** This async test validates `test_openai_completion_keeps_mm_cache_for_engine_execution`. Relevant pytest markers include `asyncio`. The main assertion is `isinstance(result, list)` and `serving_completion.openai_serving_render.preprocess_completion.call_args.kwargs['skip_mm_cache'] is False`.
**CN:** 这个异步测试验证 `test_openai_completion_keeps_mm_cache_for_engine_execution`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `isinstance(result, list)` and `serving_completion.openai_serving_render.preprocess_completion.call_args.kwargs['skip_mm_cache'] is False`。

### Test / 测试: test_renderer_only_completion_request_skips_mm_cache (L180-L208)
```python
@pytest.mark.asyncio
async def test_renderer_only_completion_request_skips_mm_cache():
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_completion = _build_serving_completion(mock_engine)
    serving_completion.openai_serving_render.preprocess_completion = AsyncMock(
        return_value=[{"prompt_token_ids": [1, 2, 3]}]
    )

    request = CompletionRequest(
        model=MODEL_NAME,
        prompt="Test prompt",
    )

    result = await serving_completion.openai_serving_render.render_completion_request(
        request
    )

    assert isinstance(result, list)
    assert (
        serving_completion.openai_serving_render.preprocess_completion.call_args.kwargs[
            "skip_mm_cache"
        ]
        is True
    )
```
**EN:** This async test validates `test_renderer_only_completion_request_skips_mm_cache`. Relevant pytest markers include `asyncio`. The main assertion is `isinstance(result, list)` and `serving_completion.openai_serving_render.preprocess_completion.call_args.kwargs['skip_mm_cache'] is True`.
**CN:** 这个异步测试验证 `test_renderer_only_completion_request_skips_mm_cache`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `isinstance(result, list)` and `serving_completion.openai_serving_render.preprocess_completion.call_args.kwargs['skip_mm_cache'] is True`。

### Test / 测试: test_completion_error_stream (L211-L289)
```python
@pytest.mark.asyncio
async def test_completion_error_stream():
    """test finish_reason='error' returns 500 InternalServerError (streaming)"""
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_completion = _build_serving_completion(mock_engine)

    completion_output_1 = CompletionOutput(
        index=0,
        text="Hello",
        token_ids=[100],
        cumulative_logprob=None,
        logprobs=None,
        finish_reason=None,
# ... 53 lines omitted for brevity ...
    async for chunk in response:
        chunks.append(chunk)

    assert len(chunks) >= 2
    assert any("Internal server error" in chunk for chunk in chunks), (
        f"Expected error message in chunks: {chunks}"
    )
    assert chunks[-1] == "data: [DONE]\n\n"
```
**EN:** This async test validates `test_completion_error_stream`. Relevant pytest markers include `asyncio`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chunks) >= 2` and `any(('Internal server error' in chunk for chunk in chunks))`.
**CN:** 这个异步测试验证 `test_completion_error_stream`。 相关的 pytest 标记包括 `asyncio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chunks) >= 2` and `any(('Internal server error' in chunk for chunk in chunks))`。

### Test / 测试: test_json_schema_response_format_missing_schema (L292-L302)
```python
def test_json_schema_response_format_missing_schema():
    """When response_format type is 'json_schema' but the json_schema field
    is not provided, request construction should raise a validation error
    so the API returns 400 instead of 500."""
    with pytest.raises(Exception, match="json_schema.*must be provided"):
        CompletionRequest(
            model=MODEL_NAME,
            prompt="Test prompt",
            max_tokens=10,
            response_format={"type": "json_schema"},
        )
```
**EN:** This test validates `test_json_schema_response_format_missing_schema`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_json_schema_response_format_missing_schema`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_negative_prompt_token_ids_nested (L305-L312)
```python
def test_negative_prompt_token_ids_nested():
    """Negative token IDs in prompt (nested list) should raise validation error."""
    with pytest.raises(Exception, match="greater than or equal to 0"):
        CompletionRequest(
            model=MODEL_NAME,
            prompt=[[-1]],
            max_tokens=10,
        )
```
**EN:** This test validates `test_negative_prompt_token_ids_nested`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_negative_prompt_token_ids_nested`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_negative_prompt_token_ids_flat (L315-L322)
```python
def test_negative_prompt_token_ids_flat():
    """Negative token IDs in prompt (flat list) should raise validation error."""
    with pytest.raises(Exception, match="greater than or equal to 0"):
        CompletionRequest(
            model=MODEL_NAME,
            prompt=[-1],
            max_tokens=10,
        )
```
**EN:** This test validates `test_negative_prompt_token_ids_flat`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_negative_prompt_token_ids_flat`。 它使用 `pytest.raises` 检查预期失败路径。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `dataclasses.dataclass`, `dataclasses.field`, `typing.Any`, `unittest.mock.AsyncMock`, `unittest.mock.MagicMock`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.config.multimodal.MultiModalConfig`, `vllm.entrypoints.openai.completion.protocol.CompletionRequest`, `vllm.entrypoints.openai.completion.serving.OpenAIServingCompletion`, `vllm.entrypoints.openai.engine.protocol.GenerationError`, `vllm.entrypoints.openai.models.protocol.BaseModelPath`, `vllm.entrypoints.openai.models.serving.OpenAIServingModels`, `vllm.entrypoints.serve.render.serving.OpenAIServingRender`, `vllm.outputs.CompletionOutput`, `vllm.outputs.RequestOutput`, `vllm.renderers.hf.HfRenderer`, `vllm.tokenizers.registry.cached_tokenizer_from_config`, `vllm.v1.engine.async_llm.AsyncLLM`
