# test_chat_error.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_chat_error.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 11 test(s), 0 fixture(s), and 10 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 11 个测试、0 个 fixture，以及 10 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L20)
```python
from dataclasses import dataclass, field
from typing import Any
from unittest.mock import AsyncMock, MagicMock, patch

import pytest

from vllm.config.multimodal import MultiModalConfig
from vllm.entrypoints.openai.chat_completion.protocol import ChatCompletionRequest
from vllm.entrypoints.openai.chat_completion.serving import OpenAIServingChat
from vllm.entrypoints.openai.engine.protocol import GenerationError
from vllm.entrypoints.openai.models.protocol import BaseModelPath
from vllm.entrypoints.openai.models.serving import OpenAIServingModels
from vllm.entrypoints.serve.render.serving import OpenAIServingRender
from vllm.outputs import CompletionOutput, RequestOutput
from vllm.renderers.hf import HfRenderer
from vllm.tokenizers.registry import cached_tokenizer_from_config
from vllm.v1.engine.async_llm import AsyncLLM
```
**EN:** Imports standard-library modules such as `dataclasses.dataclass`, `dataclasses.field`, `typing.Any`, third-party packages like `pytest`, project helpers such as `vllm.config.multimodal.MultiModalConfig`, `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionRequest`, `vllm.entrypoints.openai.chat_completion.serving.OpenAIServingChat`.
**CN:** 导入标准库模块（如 `dataclasses.dataclass`、`dataclasses.field`、`typing.Any`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.config.multimodal.MultiModalConfig`、`vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionRequest`、`vllm.entrypoints.openai.chat_completion.serving.OpenAIServingChat`）。

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

### Class / 类: MockModelConfig (L35-L61)
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

### Helper method / 辅助方法: MockModelConfig.get_diff_sampling_param (L60-L61)
```python
    def get_diff_sampling_param(self):
        return self.diff_sampling_param or {}
```
**EN:** This helper encapsulates reusable logic in `MockModelConfig.get_diff_sampling_param`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `MockModelConfig.get_diff_sampling_param` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: MockParallelConfig (L64-L66)
```python
@dataclass
class MockParallelConfig:
    _api_process_rank: int = 0
```
**EN:** This class groups related scenarios in `MockParallelConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockParallelConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: MockVllmConfig (L69-L72)
```python
@dataclass
class MockVllmConfig:
    model_config: MockModelConfig
    parallel_config: MockParallelConfig
```
**EN:** This class groups related scenarios in `MockVllmConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MockVllmConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Helper / 辅助函数: _build_renderer (L75-L79)
```python
def _build_renderer(model_config: MockModelConfig):
    return HfRenderer(
        MockVllmConfig(model_config, parallel_config=MockParallelConfig()),
        cached_tokenizer_from_config(model_config),
    )
```
**EN:** This helper encapsulates reusable logic in `_build_renderer`. Key inputs are `model_config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_renderer` 中。 关键输入包括 `model_config`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _build_serving_chat (L82-L115)
```python
def _build_serving_chat(engine: AsyncLLM) -> OpenAIServingChat:
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
    serving_chat = OpenAIServingChat(
        engine,
        models,
        response_role="assistant",
        openai_serving_render=serving_render,
# ... 8 lines omitted for brevity ...
            [{"role": "user", "content": "Test"}],
            [{"prompt_token_ids": [1, 2, 3]}],
        )

    serving_chat.openai_serving_render.preprocess_chat = AsyncMock(
        side_effect=_fake_preprocess_chat
    )
    return serving_chat
```
**EN:** This helper encapsulates reusable logic in `_build_serving_chat`. Key inputs are `engine`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_serving_chat` 中。 关键输入包括 `engine`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_chat_error_non_stream (L118-L164)
```python
@pytest.mark.asyncio
async def test_chat_error_non_stream():
    """test finish_reason='error' returns 500 InternalServerError (non-streaming)"""
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_chat = _build_serving_chat(mock_engine)

    completion_output = CompletionOutput(
        index=0,
        text="",
        token_ids=[],
        cumulative_logprob=None,
        logprobs=None,
        finish_reason="error",
# ... 21 lines omitted for brevity ...
        model=MODEL_NAME,
        messages=[{"role": "user", "content": "Test prompt"}],
        max_tokens=10,
        stream=False,
    )

    with pytest.raises(GenerationError):
        await serving_chat.create_chat_completion(request)
```
**EN:** This async test validates `test_chat_error_non_stream`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_chat_error_non_stream`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_openai_chat_keeps_mm_cache_for_engine_execution (L167-L190)
```python
@pytest.mark.asyncio
async def test_openai_chat_keeps_mm_cache_for_engine_execution():
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_chat = _build_serving_chat(mock_engine)

    request = ChatCompletionRequest(
        model=MODEL_NAME,
        messages=[{"role": "user", "content": "Test prompt"}],
    )

    result = await serving_chat.render_chat_request(request)

    assert isinstance(result, tuple)
    assert (
        serving_chat.openai_serving_render.preprocess_chat.call_args.kwargs[
            "skip_mm_cache"
        ]
        is False
    )
```
**EN:** This async test validates `test_openai_chat_keeps_mm_cache_for_engine_execution`. Relevant pytest markers include `asyncio`. The main assertion is `isinstance(result, tuple)` and `serving_chat.openai_serving_render.preprocess_chat.call_args.kwargs['skip_mm_cache'] is False`.
**CN:** 这个异步测试验证 `test_openai_chat_keeps_mm_cache_for_engine_execution`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `isinstance(result, tuple)` and `serving_chat.openai_serving_render.preprocess_chat.call_args.kwargs['skip_mm_cache'] is False`。

### Test / 测试: test_renderer_only_chat_request_skips_mm_cache (L193-L216)
```python
@pytest.mark.asyncio
async def test_renderer_only_chat_request_skips_mm_cache():
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_chat = _build_serving_chat(mock_engine)

    request = ChatCompletionRequest(
        model=MODEL_NAME,
        messages=[{"role": "user", "content": "Test prompt"}],
    )

    result = await serving_chat.openai_serving_render.render_chat_request(request)

    assert result.token_ids == [1, 2, 3]
    assert (
        serving_chat.openai_serving_render.preprocess_chat.call_args.kwargs[
            "skip_mm_cache"
        ]
        is True
    )
```
**EN:** This async test validates `test_renderer_only_chat_request_skips_mm_cache`. Relevant pytest markers include `asyncio`. The main assertion is `result.token_ids == [1, 2, 3]` and `serving_chat.openai_serving_render.preprocess_chat.call_args.kwargs['skip_mm_cache'] is True`.
**CN:** 这个异步测试验证 `test_renderer_only_chat_request_skips_mm_cache`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `result.token_ids == [1, 2, 3]` and `serving_chat.openai_serving_render.preprocess_chat.call_args.kwargs['skip_mm_cache'] is True`。

### Test / 测试: test_chat_error_stream (L219-L297)
```python
@pytest.mark.asyncio
async def test_chat_error_stream():
    """test finish_reason='error' returns 500 InternalServerError (streaming)"""
    mock_engine = MagicMock(spec=AsyncLLM)
    mock_engine.errored = False
    mock_engine.model_config = MockModelConfig()
    mock_engine.input_processor = MagicMock()
    mock_engine.renderer = _build_renderer(mock_engine.model_config)

    serving_chat = _build_serving_chat(mock_engine)

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
**EN:** This async test validates `test_chat_error_stream`. Relevant pytest markers include `asyncio`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chunks) >= 2` and `any(('Internal server error' in chunk for chunk in chunks))`.
**CN:** 这个异步测试验证 `test_chat_error_stream`。 相关的 pytest 标记包括 `asyncio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chunks) >= 2` and `any(('Internal server error' in chunk for chunk in chunks))`。

### Test / 测试: test_system_message_warns_on_image (L300-L325)
```python
@pytest.mark.parametrize(
    "image_content",
    [
        [{"type": "image_url", "image_url": {"url": "https://example.com/image.jpg"}}],
        [{"image_url": {"url": "https://example.com/image.jpg"}}],
    ],
)
def test_system_message_warns_on_image(image_content):
    """Test that system messages with image content trigger a warning."""
    with patch(
        "vllm.entrypoints.openai.chat_completion.protocol.logger"
    ) as mock_logger:
        ChatCompletionRequest(
            model=MODEL_NAME,
            messages=[
                {
                    "role": "system",
                    "content": image_content,
                }
            ],
        )

    mock_logger.warning_once.assert_called()
    call_args = str(mock_logger.warning_once.call_args)
    assert "System messages should only contain text" in call_args
    assert "image_url" in call_args
```
**EN:** This test validates `test_system_message_warns_on_image`. It uses parameterization over `image_content`. Key inputs are `image_content`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `'System messages should only contain text' in call_args` and `'image_url' in call_args`.
**CN:** 这个测试验证 `test_system_message_warns_on_image`。 它通过参数化组合 `image_content`。 关键输入包括 `image_content`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `'System messages should only contain text' in call_args` and `'image_url' in call_args`。

### Test / 测试: test_system_message_accepts_text (L328-L337)
```python
def test_system_message_accepts_text():
    """Test that system messages can contain text content."""
    # Should not raise an exception
    request = ChatCompletionRequest(
        model=MODEL_NAME,
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
        ],
    )
    assert request.messages[0]["role"] == "system"
```
**EN:** This test validates `test_system_message_accepts_text`. The main assertion is `request.messages[0]['role'] == 'system'`.
**CN:** 这个测试验证 `test_system_message_accepts_text`。 核心断言是 `request.messages[0]['role'] == 'system'`。

### Test / 测试: test_system_message_accepts_text_array (L340-L352)
```python
def test_system_message_accepts_text_array():
    """Test that system messages can contain an array with text content."""
    # Should not raise an exception
    request = ChatCompletionRequest(
        model=MODEL_NAME,
        messages=[
            {
                "role": "system",
                "content": [{"type": "text", "text": "You are a helpful assistant."}],
            },
        ],
    )
    assert request.messages[0]["role"] == "system"
```
**EN:** This test validates `test_system_message_accepts_text_array`. The main assertion is `request.messages[0]['role'] == 'system'`.
**CN:** 这个测试验证 `test_system_message_accepts_text_array`。 核心断言是 `request.messages[0]['role'] == 'system'`。

### Test / 测试: test_user_message_accepts_image (L355-L373)
```python
def test_user_message_accepts_image():
    """Test that user messages can still contain image content."""
    # Should not raise an exception
    request = ChatCompletionRequest(
        model=MODEL_NAME,
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What's in this image?"},
                    {
                        "type": "image_url",
                        "image_url": {"url": "https://example.com/image.jpg"},
                    },
                ],
            },
        ],
    )
    assert request.messages[0]["role"] == "user"
```
**EN:** This test validates `test_user_message_accepts_image`. The main assertion is `request.messages[0]['role'] == 'user'`.
**CN:** 这个测试验证 `test_user_message_accepts_image`。 核心断言是 `request.messages[0]['role'] == 'user'`。

### Test / 测试: test_system_message_warns_on_audio (L376-L406)
```python
@pytest.mark.parametrize(
    "audio_content",
    [
        [
            {
                "type": "input_audio",
                "input_audio": {"data": "base64data", "format": "wav"},
            }
        ],
        [{"input_audio": {"data": "base64data", "format": "wav"}}],
    ],
)
def test_system_message_warns_on_audio(audio_content):
    """Test that system messages with audio content trigger a warning."""
    with patch(
        "vllm.entrypoints.openai.chat_completion.protocol.logger"
    ) as mock_logger:
        ChatCompletionRequest(
# ... 5 lines omitted for brevity ...
                }
            ],
        )

    mock_logger.warning_once.assert_called()
    call_args = str(mock_logger.warning_once.call_args)
    assert "System messages should only contain text" in call_args
    assert "input_audio" in call_args
```
**EN:** This test validates `test_system_message_warns_on_audio`. It uses parameterization over `audio_content`. Key inputs are `audio_content`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `'System messages should only contain text' in call_args` and `'input_audio' in call_args`.
**CN:** 这个测试验证 `test_system_message_warns_on_audio`。 它通过参数化组合 `audio_content`。 关键输入包括 `audio_content`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `'System messages should only contain text' in call_args` and `'input_audio' in call_args`。

### Test / 测试: test_system_message_warns_on_video (L409-L434)
```python
@pytest.mark.parametrize(
    "video_content",
    [
        [{"type": "video_url", "video_url": {"url": "https://example.com/video.mp4"}}],
        [{"video_url": {"url": "https://example.com/video.mp4"}}],
    ],
)
def test_system_message_warns_on_video(video_content):
    """Test that system messages with video content trigger a warning."""
    with patch(
        "vllm.entrypoints.openai.chat_completion.protocol.logger"
    ) as mock_logger:
        ChatCompletionRequest(
            model=MODEL_NAME,
            messages=[
                {
                    "role": "system",
                    "content": video_content,
                }
            ],
        )

    mock_logger.warning_once.assert_called()
    call_args = str(mock_logger.warning_once.call_args)
    assert "System messages should only contain text" in call_args
    assert "video_url" in call_args
```
**EN:** This test validates `test_system_message_warns_on_video`. It uses parameterization over `video_content`. Key inputs are `video_content`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `'System messages should only contain text' in call_args` and `'video_url' in call_args`.
**CN:** 这个测试验证 `test_system_message_warns_on_video`。 它通过参数化组合 `video_content`。 关键输入包括 `video_content`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `'System messages should only contain text' in call_args` and `'video_url' in call_args`。

### Test / 测试: test_json_schema_response_format_missing_schema (L437-L446)
```python
def test_json_schema_response_format_missing_schema():
    """When response_format type is 'json_schema' but the json_schema field
    is not provided, request construction should raise a validation error
    so the API returns 400 instead of 500."""
    with pytest.raises(Exception, match="json_schema.*must be provided"):
        ChatCompletionRequest(
            model=MODEL_NAME,
            messages=[{"role": "user", "content": "hello"}],
            response_format={"type": "json_schema"},
        )
```
**EN:** This test validates `test_json_schema_response_format_missing_schema`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_json_schema_response_format_missing_schema`。 它使用 `pytest.raises` 检查预期失败路径。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `dataclasses.dataclass`, `dataclasses.field`, `typing.Any`, `unittest.mock.AsyncMock`, `unittest.mock.MagicMock`, `unittest.mock.patch`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.config.multimodal.MultiModalConfig`, `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionRequest`, `vllm.entrypoints.openai.chat_completion.serving.OpenAIServingChat`, `vllm.entrypoints.openai.engine.protocol.GenerationError`, `vllm.entrypoints.openai.models.protocol.BaseModelPath`, `vllm.entrypoints.openai.models.serving.OpenAIServingModels`, `vllm.entrypoints.serve.render.serving.OpenAIServingRender`, `vllm.outputs.CompletionOutput`, `vllm.outputs.RequestOutput`, `vllm.renderers.hf.HfRenderer`, `vllm.tokenizers.registry.cached_tokenizer_from_config`, `vllm.v1.engine.async_llm.AsyncLLM`
