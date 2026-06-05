# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/speech_to_text/translation/protocol.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Declares request/response schemas, protocol objects, and validation helpers. Scope: speech-to-text translation. / 声明请求/响应模式、协议对象与校验辅助逻辑。 作用域：语音转文本 / 翻译。

## Line-by-Line Analysis / 逐行分析
### Lines 4-28 — Imports and shared dependencies
```python
import time
from typing import TYPE_CHECKING, Literal, TypeAlias

from fastapi import UploadFile
from pydantic import (
    Field,
    model_validator,
)

from vllm.config.speech_to_text import SpeechToTextParams
from vllm.entrypoints.openai.engine.protocol import (
    DeltaMessage,
    OpenAIBaseModel,
    UsageInfo,
)
from vllm.exceptions import VLLMValidationError
from vllm.logger import init_logger
from vllm.sampling_params import (
    BeamSearchParams,
    RequestOutputKind,
    SamplingParams,
)
from vllm.utils import random_uuid

from ..base.protocol import _LONG_INFO, AudioResponseFormat
```
**EN:** This import block pulls in standard-library modules such as `time`, `typing`, uses third-party packages like `fastapi`, `pydantic`, depends on internal helpers such as `vllm.config.speech_to_text`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`, `vllm.sampling_params`, `vllm.utils`.
**CN:** 该导入块引入 `time`, `typing` 等标准库模块，使用 `fastapi`, `pydantic` 等第三方库，依赖 `vllm.config.speech_to_text`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`, `vllm.sampling_params`, `vllm.utils` 等 vLLM 内部模块。

### Lines 30-33 — Conditional top-level flow
```python
if TYPE_CHECKING:
    import numpy as np

    from vllm.config import ModelConfig, SpeechToTextConfig
```
**EN:** This top-level conditional toggles behavior based on runtime configuration or platform state.
**CN:** 该顶层条件分支会根据运行时配置或平台状态切换行为。

### Lines 35-35 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 38-41 — Class `TranslationResponseStreamChoice`
```python
class TranslationResponseStreamChoice(OpenAIBaseModel):
    delta: DeltaMessage
    finish_reason: str | None = None
    stop_reason: int | str | None = None
```
**EN:** Class `TranslationResponseStreamChoice` acts as a protocol/data model with about 3 field declarations and helper methods such as its inherited interface.
**CN:** 类 `TranslationResponseStreamChoice` 充当协议/数据模型，包含约 3 个字段声明，并提供 继承接口 等辅助方法。

### Lines 44-50 — Class `TranslationStreamResponse`
```python
class TranslationStreamResponse(OpenAIBaseModel):
    id: str = Field(default_factory=lambda: f"trsl-{random_uuid()}")
    object: Literal["translation.chunk"] = "translation.chunk"
    created: int = Field(default_factory=lambda: int(time.time()))
    model: str
    choices: list[TranslationResponseStreamChoice]
    usage: UsageInfo | None = Field(default=None)
```
**EN:** Class `TranslationStreamResponse` acts as a protocol/data model with about 6 field declarations and helper methods such as its inherited interface.
**CN:** 类 `TranslationStreamResponse` 充当协议/数据模型，包含约 6 个字段声明，并提供 继承接口 等辅助方法。

### Lines 53-229 — Class `TranslationRequest`
```python
class TranslationRequest(OpenAIBaseModel):
    # Ordered by official OpenAI API documentation
    # https://platform.openai.com/docs/api-reference/audio/createTranslation

    file: UploadFile
    """
    The audio file object (not file name) to translate, in one of these
    formats: flac, mp3, mp4, mpeg, mpga, m4a, ogg, wav, or webm.
    """

    model: str | None = None
    """ID of the model to use.
    """

    prompt: str = Field(default="")
    """An optional text to guide the model's style or continue a previous audio
    segment.

    The [prompt](https://platform.openai.com/docs/guides/speech-to-text#prompting)
    should match the audio language.
    """

    response_format: AudioResponseFormat = Field(default="json")
    """
    The format of the output, in one of these options: `json`, `text`, `srt`,
    `verbose_json`, or `vtt`.
    """

...
                "stream_include_usage",
            )
            raise VLLMValidationError(
                "Stream options can only be defined when `stream=True`.",
                parameter=invalid_param,
            )

        return data
```
**EN:** Class `TranslationRequest` acts as a protocol/data model with about 18 field declarations and helper methods such as `build_stt_params`, `to_beam_search_params`, `to_sampling_params`, `validate_stream_options`.
**CN:** 类 `TranslationRequest` 充当协议/数据模型，包含约 18 个字段声明，并提供 `build_stt_params`、`to_beam_search_params`、`to_sampling_params`、`validate_stream_options` 等辅助方法。

### Lines 233-235 — Class `TranslationResponse`
```python
class TranslationResponse(OpenAIBaseModel):
    text: str
    """The translated text."""
```
**EN:** Class `TranslationResponse` acts as a protocol/data model with about 1 field declarations and helper methods such as its inherited interface.
**CN:** 类 `TranslationResponse` 充当协议/数据模型，包含约 1 个字段声明，并提供 继承接口 等辅助方法。

### Lines 238-246 — Class `TranslationWord`
```python
class TranslationWord(OpenAIBaseModel):
    end: float
    """End time of the word in seconds."""

    start: float
    """Start time of the word in seconds."""

    word: str
    """The text content of the word."""
```
**EN:** Class `TranslationWord` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `TranslationWord`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 249-288 — Class `TranslationSegment`
```python
class TranslationSegment(OpenAIBaseModel):
    id: int
    """Unique identifier of the segment."""

    avg_logprob: float
    """Average logprob of the segment.

    If the value is lower than -1, consider the logprobs failed.
    """

    compression_ratio: float
    """Compression ratio of the segment.

    If the value is greater than 2.4, consider the compression failed.
    """

    end: float
    """End time of the segment in seconds."""

    no_speech_prob: float | None = None
    """Probability of no speech in the segment.

    If the value is higher than 1.0 and the `avg_logprob` is below -1, consider
    this segment silent.
    """

    seek: int
    """Seek offset of the segment."""

    start: float
    """Start time of the segment in seconds."""

    temperature: float
    """Temperature parameter used for generating the segment."""

    text: str
    """Text content of the segment."""

    tokens: list[int]
    """Array of token IDs for the text content."""
```
**EN:** Class `TranslationSegment` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `TranslationSegment`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 291-305 — Class `TranslationResponseVerbose`
```python
class TranslationResponseVerbose(OpenAIBaseModel):
    duration: str
    """The duration of the input audio."""

    language: str
    """The language of the input audio."""

    text: str
    """The translated text."""

    segments: list[TranslationSegment] | None = None
    """Segments of the translated text and their corresponding details."""

    words: list[TranslationWord] | None = None
    """Extracted words and their corresponding timestamps."""
```
**EN:** Class `TranslationResponseVerbose` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `TranslationResponseVerbose`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 308-308 — Module constants and state
```python
TranslationResponseVariant: TypeAlias = TranslationResponse | TranslationResponseVerbose
```
**EN:** This block initializes `TranslationResponseVariant`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `TranslationResponseVariant`，为后续逻辑准备模块级常量或共享状态。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Structured request/response models / 结构化请求/响应模型
- Tokenization or token-level processing / 分词或 token 级处理
- Speech transcription flow / 语音转写流程
- Speech translation flow / 语音翻译流程
- Speech-to-text serving stack / 语音转文本服务栈

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `time`, `typing`
- **Third-party / 第三方**: `fastapi`, `pydantic`, `numpy`
- **vLLM Internal / vLLM 内部**: `vllm.config.speech_to_text`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`, `vllm.sampling_params`, `vllm.utils`, `..base.protocol`, `vllm.config`
