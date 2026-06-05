# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/base/protocol.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Declares request/response schemas, protocol objects, and validation helpers. Scope: pooling base. / 声明请求/响应模式、协议对象与校验辅助逻辑。 作用域：池化 / 基础层。

## Line-by-Line Analysis / 逐行分析
### Lines 5-18 — Imports and shared dependencies
```python
from typing import Annotated, Any, Literal

from pydantic import Field, model_validator

from vllm.config import ModelConfig
from vllm.entrypoints.chat_utils import (
    ChatCompletionMessageParam,
    ChatTemplateContentFormatOption,
)
from vllm.entrypoints.openai.engine.protocol import OpenAIBaseModel
from vllm.exceptions import VLLMValidationError
from vllm.renderers import ChatParams, TokenizeParams, merge_kwargs
from vllm.utils import random_uuid
from vllm.utils.serial_utils import EmbedDType, EncodingFormat, Endianness
```
**EN:** This import block pulls in standard-library modules such as `typing`, uses third-party packages like `pydantic`, depends on internal helpers such as `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.renderers`, `vllm.utils`.
**CN:** 该导入块引入 `typing` 等标准库模块，使用 `pydantic` 等第三方库，依赖 `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.renderers`, `vllm.utils` 等 vLLM 内部模块。

### Lines 21-103 — Class `PoolingBasicRequestMixin`
```python
class PoolingBasicRequestMixin(OpenAIBaseModel):
    # --8<-- [start:pooling-common-params]
    model: str | None = None
    user: str | None = None
    # --8<-- [end:pooling-common-params]

    # --8<-- [start:pooling-common-extra-params]
    truncate_prompt_tokens: Annotated[int, Field(ge=-1)] | None = None
    truncation_side: Literal["left", "right"] | None = Field(
        default=None,
        description=(
            "Which side to truncate from when truncate_prompt_tokens is active. "
            "'right' keeps the first N tokens. "
            "'left' keeps the last N tokens."
        ),
    )
    request_id: str = Field(
        default_factory=random_uuid,
        description=(
            "The request_id related to this request. If the caller does "
            "not set it, a random_uuid will be generated. This id is used "
            "through out the inference process and return in response."
        ),
    )
    priority: int = Field(
        default=0,
        ge=-(2**63),
        le=2**63 - 1,
...
            max_output_tokens=max_output_tokens,
            truncate_prompt_tokens=self.truncate_prompt_tokens,
            truncation_side=self.truncation_side,
            do_lower_case=encoder_config.get("do_lower_case", False),
            add_special_tokens=add_special_tokens,
            max_total_tokens_param=max_total_tokens_param,
            max_output_tokens_param=max_output_tokens_param,
        )
```
**EN:** Class `PoolingBasicRequestMixin` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like `_build_pooling_tok_params`.
**CN:** 这里定义类 `PoolingBasicRequestMixin`，其职责是封装相关状态与行为，并通过 `_build_pooling_tok_params` 等方法组织逻辑。

### Lines 106-119 — Class `PoolingTokenizeParamsMixin`
```python
class PoolingTokenizeParamsMixin:
    add_special_tokens: bool

    def _build_pooling_tok_params(
        self,
        model_config: ModelConfig,
        *,
        add_special_tokens: bool,
        max_total_tokens: int | None,
        max_output_tokens: int,
        max_total_tokens_param: str = "max_model_len",
        max_output_tokens_param: str | None = None,
    ) -> TokenizeParams:
        raise NotImplementedError
```
**EN:** Class `PoolingTokenizeParamsMixin` is defined here, as a standalone type, and groups behavior through methods like `_build_pooling_tok_params`.
**CN:** 这里定义类 `PoolingTokenizeParamsMixin`，其职责是封装相关状态与行为，并通过 `_build_pooling_tok_params` 等方法组织逻辑。

### Lines 122-129 — Class `FixedMaxLenTokenizeParamsMixin`
```python
class FixedMaxLenTokenizeParamsMixin(PoolingTokenizeParamsMixin):
    def build_tok_params(self, model_config: ModelConfig) -> TokenizeParams:
        return self._build_pooling_tok_params(
            model_config,
            add_special_tokens=self.add_special_tokens,
            max_total_tokens=model_config.max_model_len,
            max_output_tokens=0,
        )
```
**EN:** Class `FixedMaxLenTokenizeParamsMixin` is defined here, extending `PoolingTokenizeParamsMixin`, and groups behavior through methods like `build_tok_params`.
**CN:** 这里定义类 `FixedMaxLenTokenizeParamsMixin`，其职责是封装相关状态与行为，并通过 `build_tok_params` 等方法组织逻辑。

### Lines 132-152 — Class `EmbeddingTokenizeParamsMixin`
```python
class EmbeddingTokenizeParamsMixin(PoolingTokenizeParamsMixin):
    def build_tok_params(self, model_config: ModelConfig) -> TokenizeParams:
        default_max_total_tokens = model_config.max_model_len
        max_total_tokens: int | None = default_max_total_tokens
        max_output_tokens = 0

        pooler_config = model_config.pooler_config
        if pooler_config is not None:
            if pooler_config.enable_chunked_processing:
                max_total_tokens = None
            else:
                max_embed_len = pooler_config.max_embed_len or default_max_total_tokens
                max_output_tokens = default_max_total_tokens - max_embed_len

        return self._build_pooling_tok_params(
            model_config,
            add_special_tokens=self.add_special_tokens,
            max_total_tokens=max_total_tokens,
            max_output_tokens=max_output_tokens,
            max_output_tokens_param="max_model_len - max_embed_len",
        )
```
**EN:** Class `EmbeddingTokenizeParamsMixin` is defined here, extending `PoolingTokenizeParamsMixin`, and groups behavior through methods like `build_tok_params`.
**CN:** 这里定义类 `EmbeddingTokenizeParamsMixin`，其职责是封装相关状态与行为，并通过 `build_tok_params` 等方法组织逻辑。

### Lines 155-167 — Class `CompletionRequestMixin`
```python
class CompletionRequestMixin(OpenAIBaseModel):
    # --8<-- [start:completion-params]
    input: list[int] | list[list[int]] | str | list[str]
    # --8<-- [end:completion-params]

    # --8<-- [start:completion-extra-params]
    add_special_tokens: bool = Field(
        default=True,
        description=(
            "If true (the default), special tokens (e.g. BOS) will be added to "
            "the prompt."
        ),
    )
```
**EN:** Class `CompletionRequestMixin` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `CompletionRequestMixin`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 171-256 — Class `ChatRequestMixin`
```python
class ChatRequestMixin(OpenAIBaseModel):
    # --8<-- [start:chat-params]
    messages: list[ChatCompletionMessageParam]
    # --8<-- [end:chat-params]

    # --8<-- [start:chat-extra-params]
    add_generation_prompt: bool = Field(
        default=False,
        description=(
            "If true, the generation prompt will be added to the chat template. "
            "This is a parameter used by chat template in tokenizer config of the "
            "model."
        ),
    )
    continue_final_message: bool = Field(
        default=False,
        description=(
            "If this is set, the chat will be formatted so that the final "
            "message in the chat is open-ended, without any EOS tokens. The "
            "model will continue this message rather than starting a new one. "
            'This allows you to "prefill" part of the model\'s response for it. '
            "Cannot be used at the same time as `add_generation_prompt`."
        ),
    )
    add_special_tokens: bool = Field(
        default=False,
        description=(
            "If true, special tokens (e.g. BOS) will be added to the prompt "
...
                self.chat_template_kwargs,
                dict(
                    add_generation_prompt=self.add_generation_prompt,
                    continue_final_message=self.continue_final_message,
                ),
            ),
            media_io_kwargs=self.media_io_kwargs,
        )
```
**EN:** Class `ChatRequestMixin` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like `check_generation_prompt`, `build_chat_params`.
**CN:** 这里定义类 `ChatRequestMixin`，其职责是封装相关状态与行为，并通过 `check_generation_prompt`、`build_chat_params` 等方法组织逻辑。

### Lines 259-280 — Class `EncodingRequestMixin`
```python
class EncodingRequestMixin(OpenAIBaseModel):
    # --8<-- [start:encoding-params]
    encoding_format: EncodingFormat = "float"
    # --8<-- [end:encoding-params]

    # --8<-- [start:encoding-extra-params]
    embed_dtype: EmbedDType = Field(
        default="float32",
        description=(
            "What dtype to use for encoding. Default to using float32 for base64 "
            "encoding to match the OpenAI python client behavior. "
            "This parameter will affect base64 and binary_response."
        ),
    )
    endianness: Endianness = Field(
        default="native",
        description=(
            "What endianness to use for encoding. Default to using native for "
            "base64 encoding to match the OpenAI python client behavior."
            "This parameter will affect base64 and binary_response."
        ),
    )
```
**EN:** Class `EncodingRequestMixin` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `EncodingRequestMixin`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 284-294 — Class `EmbedRequestMixin`
```python
class EmbedRequestMixin(EncodingRequestMixin):
    # --8<-- [start:embed-params]
    dimensions: int | None = None
    # --8<-- [end:embed-params]

    # --8<-- [start:embed-extra-params]
    use_activation: bool | None = Field(
        default=None,
        description="Whether to use activation for the pooler outputs. "
        "`None` uses the pooler's default, which is `True` in most cases.",
    )
```
**EN:** Class `EmbedRequestMixin` is defined here, extending `EncodingRequestMixin`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `EmbedRequestMixin`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 298-304 — Class `ClassifyRequestMixin`
```python
class ClassifyRequestMixin(OpenAIBaseModel):
    # --8<-- [start:classify-extra-params]
    use_activation: bool | None = Field(
        default=None,
        description="Whether to use activation for the pooler outputs. "
        "`None` uses the pooler's default, which is `True` in most cases.",
    )
```
**EN:** Class `ClassifyRequestMixin` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ClassifyRequestMixin`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

## Key Concepts / 关键概念
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Structured request/response models / 结构化请求/响应模型
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Pooling task support / 池化任务支持
- Embedding generation / 嵌入生成
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing`
- **Third-party / 第三方**: `pydantic`
- **vLLM Internal / vLLM 内部**: `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.renderers`, `vllm.utils`, `vllm.utils.serial_utils`
