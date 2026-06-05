# io_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/embed/io_processor.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Transforms external inputs/outputs into the internal shapes expected by vLLM. Scope: pooling embedding. / 将外部输入/输出转换为 vLLM 内部期望的数据形态。 作用域：池化 / 嵌入。

## Line-by-Line Analysis / 逐行分析
### Lines 3-41 — Imports and shared dependencies
```python
from collections.abc import Sequence
from typing import Any, Literal, cast

import torch
from openai.types.chat import (
    ChatCompletionContentPartImageParam,
    ChatCompletionContentPartTextParam,
)
from openai.types.chat.chat_completion_content_part_image_param import ImageURL

from vllm import PoolingParams
from vllm.entrypoints.chat_utils import (
    ChatCompletionContentPartParam,
    ChatCompletionMessageParam,
    CustomChatCompletionMessageParam,
)
from vllm.inputs import EngineInput, tokens_input
from vllm.logger import init_logger
from vllm.outputs import PoolingOutput, PoolingRequestOutput
from vllm.renderers import merge_kwargs
from vllm.renderers.hf import resolve_chat_template
from vllm.utils.collection_utils import chunk_list
from vllm.utils.mistral import is_mistral_tokenizer

from ..base.io_processor import PoolingIOProcessor
from ..scoring.io_processor import JinaRankingIOProcessorMixin
...
)
from .protocol import (
    CohereEmbedContent,
    CohereEmbedInput,
    CohereEmbedRequest,
    EmbeddingChatRequest,
    EmbeddingCompletionRequest,
)
```
**EN:** This import block pulls in standard-library modules such as `collections`, `typing`, uses third-party packages like `torch`, `openai`, depends on internal helpers such as `vllm`, `vllm.entrypoints.chat_utils`, `vllm.inputs`, `vllm.logger`, `vllm.outputs`, `vllm.renderers`.
**CN:** 该导入块引入 `collections`, `typing` 等标准库模块，使用 `torch`, `openai` 等第三方库，依赖 `vllm`, `vllm.entrypoints.chat_utils`, `vllm.inputs`, `vllm.logger`, `vllm.outputs`, `vllm.renderers` 等 vLLM 内部模块。

### Lines 43-43 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 46-558 — Class `EmbedIOProcessor`
```python
class EmbedIOProcessor(PoolingIOProcessor):
    name = "embed"

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        assert self.model_config.pooler_config is not None

        self.pooler_config = self.model_config.pooler_config
        self.enable_chunked_processing = self.pooler_config.enable_chunked_processing

        # Load task instructions from HF config or sentence-transformers config
        self.task_instructions: dict[str, str] | None = self._load_task_instructions(
            self.model_config.hf_config
        ) or self._load_st_prompts(self.model_config.model, self.model_config.revision)
        if self.task_instructions:
            logger.info(
                "Loaded prompt prefixes for input_type: %s",
                list(self.task_instructions.keys()),
            )

    def pre_process_online(self, ctx: PoolingServeContext):
        if isinstance(ctx.request, CohereEmbedRequest):
            self._pre_process_cohere_online(ctx)
        else:
            super().pre_process_online(ctx)

        if self.enable_chunked_processing:
            self._pre_process_chunked(ctx)
...
            return None
        return self.task_instructions.get(input_type) or None

    def _enforce_cohere_max_tokens(self, ctx: PoolingServeContext) -> None:
        if isinstance(ctx.request, CohereEmbedRequest):
            request = ctx.request
            if request.truncate == "NONE" and request.max_tokens is not None:
                self._check_cohere_max_tokens(ctx.final_res_batch, request.max_tokens)
```
**EN:** Class `EmbedIOProcessor` is defined here, extending `PoolingIOProcessor`, and groups behavior through methods like `__init__`, `pre_process_online`, `post_process_online`, `_pre_process_chunked`.
**CN:** 这里定义类 `EmbedIOProcessor`，其职责是封装相关状态与行为，并通过 `__init__`、`pre_process_online`、`post_process_online`、`_pre_process_chunked` 等方法组织逻辑。

### Lines 561-562 — Class `TokenEmbedIOProcessor`
```python
class TokenEmbedIOProcessor(PoolingIOProcessor):
    name = "token_embed"
```
**EN:** Class `TokenEmbedIOProcessor` is defined here, extending `PoolingIOProcessor`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `TokenEmbedIOProcessor`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 565-607 — Class `JinaRankingTokenEmbedIOProcessor`
```python
class JinaRankingTokenEmbedIOProcessor(
    TokenEmbedIOProcessor, JinaRankingIOProcessorMixin
):
    def pre_process_online(self, ctx: PoolingServeContext):
        request = ctx.request
        if isinstance(request, PoolingCompletionLikeRequest):
            prompts = request.input
            if not isinstance(prompts, Sequence) or len(prompts) < 2:
                raise ValueError("The JinaForRanking model requires at least 2 inputs.")

            text_prompts = self.ensure_str(prompts)

            # The JinaForRanking model concatenates docs first, then query.
            # Let's stay consistent with this novel design.
            prompt_input = self.format_docs_prompts_func(
                query=text_prompts[-1], docs=text_prompts[:-1]
            )

            engine_inputs = self._preprocess_cmpl_online(
                request,
                prompt_input=prompt_input,
                prompt_embeds=None,
            )
        elif isinstance(request, PoolingChatLikeRequest):
            raise ValueError("The JinaForRanking does not support chat Request.")
        else:
            raise ValueError(f"Invalid {self.name} request type")

...

        # The JinaForRanking model concatenates docs first, then query.
        # Let's stay consistent with this novel design.
        ctx.prompts = self.format_docs_prompts_func(
            query=text_prompts[-1], docs=text_prompts[:-1]
        )

        return super().pre_process_offline(ctx)
```
**EN:** Class `JinaRankingTokenEmbedIOProcessor` is defined here, extending `TokenEmbedIOProcessor`, `JinaRankingIOProcessorMixin`, and groups behavior through methods like `pre_process_online`, `pre_process_offline`.
**CN:** 这里定义类 `JinaRankingTokenEmbedIOProcessor`，其职责是封装相关状态与行为，并通过 `pre_process_online`、`pre_process_offline` 等方法组织逻辑。

## Key Concepts / 关键概念
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Batch workflow handling / 批处理工作流
- Pooling task support / 池化任务支持
- Embedding generation / 嵌入生成
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections`, `typing`
- **Third-party / 第三方**: `torch`, `openai`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.entrypoints.chat_utils`, `vllm.inputs`, `vllm.logger`, `vllm.outputs`, `vllm.renderers`, `vllm.renderers.hf`, `vllm.utils.collection_utils`, `vllm.utils.mistral`, `..base.io_processor`, `..scoring.io_processor`, `..typing`
