# serving.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/speech_to_text/base/serving.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the serving layer that coordinates preprocessing, engine calls, and response shaping. Scope: speech-to-text base. / 实现服务层，负责预处理、引擎调用与响应整形。 作用域：语音转文本 / 基础层。

## Line-by-Line Analysis / 逐行分析
### Lines 3-55 — Imports and shared dependencies
```python
import asyncio
import io
import math
import time
import zlib
from collections.abc import AsyncGenerator, Callable, Set
from functools import cached_property
from typing import Final, Literal, TypeAlias, TypeVar, cast

import numpy as np
from fastapi import Request
from transformers import PreTrainedTokenizerBase

import vllm.envs as envs
from vllm.engine.protocol import EngineClient
from vllm.entrypoints.logger import RequestLogger
from vllm.entrypoints.openai.engine.protocol import (
    DeltaMessage,
    ErrorResponse,
    RequestResponseMetadata,
    UsageInfo,
)
from vllm.entrypoints.openai.engine.serving import OpenAIServing, SpeechToTextRequest
from vllm.entrypoints.openai.models.serving import OpenAIServingModels
from vllm.entrypoints.utils import get_max_tokens
from vllm.exceptions import VLLMValidationError
...
)
from ..translation.protocol import (
    TranslationResponse,
    TranslationResponseStreamChoice,
    TranslationResponseVerbose,
    TranslationSegment,
    TranslationStreamResponse,
)
```
**EN:** This import block pulls in standard-library modules such as `asyncio`, `io`, `math`, `time`, `zlib`, `collections`, uses third-party packages like `numpy`, `fastapi`, `transformers`, depends on internal helpers such as `vllm.envs`, `vllm.engine.protocol`, `vllm.entrypoints.logger`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.engine.serving`, `vllm.entrypoints.openai.models.serving`.
**CN:** 该导入块引入 `asyncio`, `io`, `math`, `time`, `zlib`, `collections` 等标准库模块，使用 `numpy`, `fastapi`, `transformers` 等第三方库，依赖 `vllm.envs`, `vllm.engine.protocol`, `vllm.entrypoints.logger`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.engine.serving`, `vllm.entrypoints.openai.models.serving` 等 vLLM 内部模块。

### Lines 57-73 — Module constants
```python
SpeechToTextResponse: TypeAlias = TranscriptionResponse | TranslationResponse
SpeechToTextResponseVerbose: TypeAlias = (
    TranscriptionResponseVerbose | TranslationResponseVerbose
)
SpeechToTextSegment: TypeAlias = TranscriptionSegment | TranslationSegment
T = TypeVar("T", bound=SpeechToTextResponse)
V = TypeVar("V", bound=SpeechToTextResponseVerbose)
S = TypeVar("S", bound=SpeechToTextSegment)

ResponseType: TypeAlias = (
    TranscriptionResponse
    | TranslationResponse
    | TranscriptionResponseVerbose
    | TranslationResponseVerbose
)

logger = init_logger(__name__)
```
**EN:** This block initializes `SpeechToTextResponse`, `SpeechToTextResponseVerbose`, `SpeechToTextSegment`, `T`, `V`, `S`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `SpeechToTextResponse`, `SpeechToTextResponseVerbose`, `SpeechToTextSegment`, `T`, `V`, `S`，为后续逻辑准备模块级常量或共享状态。

### Lines 76-84 — Function `asr_inter_chunk_separator`
```python
def asr_inter_chunk_separator(
    language: str | None, no_space_languages: Set[str]
) -> str:
    """Space to insert between ASR text chunks for streaming and non-streaming join.

    Languages in ``no_space_languages`` (e.g. Chinese, Japanese) use an empty
    separator; others use a single ASCII space.
    """
    return "" if language and language.lower() in no_space_languages else " "
```
**EN:** This function `asr_inter_chunk_separator` is documented as: Space to insert between ASR text chunks for streaming and non-streaming join.
**CN:** 这里定义函数 `asr_inter_chunk_separator`，其文档字符串说明了主要职责与调用约定。

### Lines 87-756 — Class `OpenAISpeechToText`
```python
class OpenAISpeechToText(OpenAIServing):
    """Base class for speech-to-text operations like transcription and
    translation."""

    def __init__(
        self,
        engine_client: EngineClient,
        models: OpenAIServingModels,
        *,
        request_logger: RequestLogger | None,
        return_tokens_as_token_ids: bool = False,
        task_type: Literal["transcribe", "translate"] = "transcribe",
        enable_force_include_usage: bool = False,
    ):
        super().__init__(
            engine_client=engine_client,
            models=models,
            request_logger=request_logger,
            return_tokens_as_token_ids=return_tokens_as_token_ids,
        )

        self.default_sampling_params = self.model_config.get_diff_sampling_param()
        self.task_type: Final = task_type

        self.asr_config = self.model_cls.get_speech_to_text_config(
            self.model_config, task_type
        )

...
            )

        except Exception as e:
            logger.exception("Error in %s stream generator.", self.task_type)
            data = self.create_streaming_error_response(e)
            yield f"data: {data}\n\n"
        # Send the final done message after all response.n are finished
        yield "data: [DONE]\n\n"
```
**EN:** Class `OpenAISpeechToText` is introduced here. Its docstring describes the intent as: Base class for speech-to-text operations like transcription and translation.
**CN:** 这里定义类 `OpenAISpeechToText`。其文档字符串说明了该类的职责与使用方式。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Async request handling / 异步请求处理
- Incremental result emission / 增量结果输出
- Engine-backed serving orchestration / 基于引擎的服务编排
- Middleware pipeline / 中间件链路
- Tokenization or token-level processing / 分词或 token 级处理
- LoRA adapter support / LoRA 适配器支持
- Prompt or multimodal rendering / 提示词或多模态渲染

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `io`, `math`, `time`, `zlib`, `collections`, `functools`, `typing`
- **Third-party / 第三方**: `numpy`, `fastapi`, `transformers`
- **vLLM Internal / vLLM 内部**: `vllm.envs`, `vllm.engine.protocol`, `vllm.entrypoints.logger`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.engine.serving`, `vllm.entrypoints.openai.models.serving`, `vllm.entrypoints.utils`, `vllm.exceptions`, `vllm.inputs`, `vllm.logger`, `vllm.logprobs`, `vllm.model_executor.models`
