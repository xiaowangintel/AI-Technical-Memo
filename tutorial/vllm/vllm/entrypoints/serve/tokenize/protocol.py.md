# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/serve/tokenize/protocol.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Declares request/response schemas, protocol objects, and validation helpers. Scope: serving tokenization. / 声明请求/响应模式、协议对象与校验辅助逻辑。 作用域：服务扩展 / 分词。

## Line-by-Line Analysis / 逐行分析
### Lines 5-21 — Imports and shared dependencies
```python
from typing import Annotated, Any, TypeAlias

from pydantic import ConfigDict, Field, model_validator

from vllm.config import ModelConfig
from vllm.entrypoints.chat_utils import (
    ChatCompletionMessageParam,
    ChatTemplateContentFormatOption,
)
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionToolsParam,
)
from vllm.entrypoints.openai.engine.protocol import (
    OpenAIBaseModel,
)
from vllm.exceptions import VLLMValidationError
from vllm.renderers import ChatParams, TokenizeParams, merge_kwargs
```
**EN:** This import block pulls in standard-library modules such as `typing`, uses third-party packages like `pydantic`, depends on internal helpers such as `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.renderers`.
**CN:** 该导入块引入 `typing` 等标准库模块，使用 `pydantic` 等第三方库，依赖 `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.renderers` 等 vLLM 内部模块。

### Lines 24-47 — Class `TokenizeCompletionRequest`
```python
class TokenizeCompletionRequest(OpenAIBaseModel):
    model: str | None = None
    prompt: str

    add_special_tokens: bool = Field(
        default=True,
        description=(
            "If true (the default), special tokens (e.g. BOS) will be added to "
            "the prompt."
        ),
    )
    return_token_strs: bool | None = Field(
        default=False,
        description=(
            "If true, also return the token strings corresponding to the token ids."
        ),
    )

    def build_tok_params(self, model_config: ModelConfig) -> TokenizeParams:
        return TokenizeParams(
            max_total_tokens=None,
            max_output_tokens=0,
            add_special_tokens=self.add_special_tokens,
        )
```
**EN:** Class `TokenizeCompletionRequest` acts as a protocol/data model with about 4 field declarations and helper methods such as `build_tok_params`.
**CN:** 类 `TokenizeCompletionRequest` 充当协议/数据模型，包含约 4 个字段声明，并提供 `build_tok_params` 等辅助方法。

### Lines 50-153 — Class `TokenizeChatRequest`
```python
class TokenizeChatRequest(OpenAIBaseModel):
    model: str | None = None
    messages: list[ChatCompletionMessageParam]

    add_generation_prompt: bool = Field(
        default=True,
        description=(
            "If true, the generation prompt will be added to the chat template. "
            "This is a parameter used by chat template in tokenizer config of the "
            "model."
        ),
    )
    return_token_strs: bool | None = Field(
        default=False,
        description=(
            "If true, also return the token strings corresponding to the token ids."
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
...
        )

    def build_tok_params(self, model_config: ModelConfig) -> TokenizeParams:
        return TokenizeParams(
            max_total_tokens=None,
            max_output_tokens=0,
            add_special_tokens=self.add_special_tokens,
        )
```
**EN:** Class `TokenizeChatRequest` acts as a protocol/data model with about 11 field declarations and helper methods such as `check_generation_prompt`, `build_chat_params`, `build_tok_params`.
**CN:** 类 `TokenizeChatRequest` 充当协议/数据模型，包含约 11 个字段声明，并提供 `check_generation_prompt`、`build_chat_params`、`build_tok_params` 等辅助方法。

### Lines 156-156 — Module constants and state
```python
TokenizeRequest: TypeAlias = TokenizeCompletionRequest | TokenizeChatRequest
```
**EN:** This block initializes `TokenizeRequest`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `TokenizeRequest`，为后续逻辑准备模块级常量或共享状态。

### Lines 159-163 — Class `TokenizeResponse`
```python
class TokenizeResponse(OpenAIBaseModel):
    count: int
    max_model_len: int
    tokens: list[int]
    token_strs: list[str] | None = None
```
**EN:** Class `TokenizeResponse` acts as a protocol/data model with about 4 field declarations and helper methods such as its inherited interface.
**CN:** 类 `TokenizeResponse` 充当协议/数据模型，包含约 4 个字段声明，并提供 继承接口 等辅助方法。

### Lines 166-178 — Class `DetokenizeRequest`
```python
class DetokenizeRequest(OpenAIBaseModel):
    model: str | None = None
    # TODO: Factor `torch.iinfo` out. `torch.iinfo` pulls torch into a
    # Pydantic protocol file that currently has no torch dependency.
    # See: https://github.com/vllm-project/vllm/pull/34468#discussion_r2801173630
    tokens: list[Annotated[int, Field(ge=0, le=2**63 - 1)]]

    def build_tok_params(self, model_config: ModelConfig) -> TokenizeParams:
        return TokenizeParams(
            max_total_tokens=None,
            max_output_tokens=0,
            needs_detokenization=True,
        )
```
**EN:** Class `DetokenizeRequest` acts as a protocol/data model with about 2 field declarations and helper methods such as `build_tok_params`.
**CN:** 类 `DetokenizeRequest` 充当协议/数据模型，包含约 2 个字段声明，并提供 `build_tok_params` 等辅助方法。

### Lines 181-182 — Class `DetokenizeResponse`
```python
class DetokenizeResponse(OpenAIBaseModel):
    prompt: str
```
**EN:** Class `DetokenizeResponse` acts as a protocol/data model with about 1 field declarations and helper methods such as its inherited interface.
**CN:** 类 `DetokenizeResponse` 充当协议/数据模型，包含约 1 个字段声明，并提供 继承接口 等辅助方法。

### Lines 185-192 — Class `TokenizerInfoResponse`
```python
class TokenizerInfoResponse(OpenAIBaseModel):
    """
    Response containing tokenizer configuration
    equivalent to tokenizer_config.json
    """

    model_config = ConfigDict(extra="allow")
    tokenizer_class: str
```
**EN:** Class `TokenizerInfoResponse` is introduced here. Its docstring describes the intent as: Response containing tokenizer configuration equivalent to tokenizer_config.json
**CN:** 这里定义类 `TokenizerInfoResponse`。其文档字符串说明了该类的职责与使用方式。

## Key Concepts / 关键概念
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Structured request/response models / 结构化请求/响应模型
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Auxiliary serving endpoints / 辅助服务端点

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing`
- **Third-party / 第三方**: `pydantic`
- **vLLM Internal / vLLM 内部**: `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.renderers`
