# io_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/scoring/io_processor.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Transforms external inputs/outputs into the internal shapes expected by vLLM. Scope: pooling scoring. / 将外部输入/输出转换为 vLLM 内部期望的数据形态。 作用域：池化 / 评分。

## Line-by-Line Analysis / 逐行分析
### Lines 3-33 — Imports and shared dependencies
```python
import time
from collections.abc import Sequence
from typing import Any, TypeAlias

import torch.nn.functional as F

from vllm import PoolingParams, PoolingRequestOutput, TokensPrompt
from vllm.inputs import EngineInput
from vllm.renderers import TokenizeParams
from vllm.renderers.hf import safe_apply_chat_template
from vllm.tasks import PoolingTask
from vllm.utils.mistral import is_mistral_tokenizer

from ...chat_utils import ChatTemplateResolutionError
from ..base.io_processor import PoolingIOProcessor
from ..typing import (
    OfflineInputsContext,
    OfflineOutputsContext,
    PoolingServeContext,
)
from .protocol import RerankRequest, ScoreRequest, ScoringRequest
from .typing import ScoreData, ScoreInput, ScoringData
from .utils import (
    compress_token_type_ids,
    compute_maxsim_score,
    get_num_special_tokens_for_pair,
    parse_score_data,
    score_data_to_prompts,
    truncate_text_to_tokens,
    validate_score_input,
)
```
**EN:** This import block pulls in standard-library modules such as `time`, `collections`, `typing`, uses third-party packages like `torch`, depends on internal helpers such as `vllm`, `vllm.inputs`, `vllm.renderers`, `vllm.renderers.hf`, `vllm.tasks`, `vllm.utils.mistral`.
**CN:** 该导入块引入 `time`, `collections`, `typing` 等标准库模块，使用 `torch` 等第三方库，依赖 `vllm`, `vllm.inputs`, `vllm.renderers`, `vllm.renderers.hf`, `vllm.tasks`, `vllm.utils.mistral` 等 vLLM 内部模块。

### Lines 35-35 — Module constants and state
```python
ScoringServeContext: TypeAlias = PoolingServeContext[ScoringRequest]
```
**EN:** This block initializes `ScoringServeContext`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `ScoringServeContext`，为后续逻辑准备模块级常量或共享状态。

### Lines 38-122 — Class `ScoringIOProcessor`
```python
class ScoringIOProcessor(PoolingIOProcessor):
    name: str
    pooling_task: PoolingTask

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self.tokenizer = self.renderer.get_tokenizer()
        self.architecture = self.model_config.architecture
        self.is_multimodal_model = self.model_config.is_multimodal_model
        self.pad_token_id = self.tokenizer.pad_token_id

    def create_pooling_params(self, request):
        return request.to_pooling_params(self.pooling_task)

    def _validate_token_limit(self, value: int, name: str) -> None:
        if value < 0:
            raise ValueError(f"{name} must be a non-negative integer")
        if value >= self.model_config.max_model_len:
            raise ValueError(
                f"{name} ({value}) must be less "
                f"than max_model_len ({self.model_config.max_model_len})."
            )

    def _get_token_limits(
        self,
        request: ScoringRequest | None = None,
        pooling_params: PoolingParams | None = None,
...
    ) -> ScoringData:
        scoring_data = validate_score_input(
            data_1,
            data_2,
            is_multimodal_model=self.is_multimodal_model,
            architecture=self.architecture,
        )
        return scoring_data
```
**EN:** Class `ScoringIOProcessor` is defined here, extending `PoolingIOProcessor`, and groups behavior through methods like `__init__`, `create_pooling_params`, `_validate_token_limit`, `_get_token_limits`.
**CN:** 这里定义类 `ScoringIOProcessor`，其职责是封装相关状态与行为，并通过 `__init__`、`create_pooling_params`、`_validate_token_limit`、`_get_token_limits` 等方法组织逻辑。

### Lines 125-255 — Class `BiEncoderIOProcessor`
```python
class BiEncoderIOProcessor(ScoringIOProcessor):
    name = "bi-encoder"
    pooling_task: PoolingTask = "embed"

    #######################################
    # online APIs

    def pre_process_online(self, ctx: ScoringServeContext):
        request = ctx.request

        if isinstance(request, ScoreRequest):
            data_1 = request.data_1
            data_2 = request.data_2
        elif isinstance(request, RerankRequest):
            data_1 = request.query
            data_2 = request.documents
        else:
            raise ValueError(f"Invalid {self.name} request type")

        scoring_data = self.valid_inputs(data_1, data_2)

        max_tokens_per_query, max_tokens_per_doc = self._get_token_limits(
            request=request
        )
        if max_tokens_per_query > 0 or max_tokens_per_doc > 0:
            scoring_data = self._truncate_scoring_data(
                scoring_data, max_tokens_per_query, max_tokens_per_doc
            )
...
                    request_id=f"{emb_1.request_id}_{emb_2.request_id}",
                    outputs=pair_score,
                    prompt_token_ids=tokens,
                    num_cached_tokens=emb_1.num_cached_tokens + emb_2.num_cached_tokens,
                    finished=True,
                )
            )
        return final_res_batch
```
**EN:** Class `BiEncoderIOProcessor` is defined here, extending `ScoringIOProcessor`, and groups behavior through methods like `pre_process_online`, `post_process_online`, `pre_process_offline`, `post_process_offline`.
**CN:** 这里定义类 `BiEncoderIOProcessor`，其职责是封装相关状态与行为，并通过 `pre_process_online`、`post_process_online`、`pre_process_offline`、`post_process_offline` 等方法组织逻辑。

### Lines 258-296 — Class `LateInteractionIOProcessor`
```python
class LateInteractionIOProcessor(BiEncoderIOProcessor):
    name = "late-interaction"
    pooling_task: PoolingTask = "token_embed"

    def _post_process(self, outputs: list[PoolingRequestOutput], n_queries: int):
        # Split into query and document embeddings
        emb_data_1 = outputs[:n_queries]
        emb_data_2 = outputs[n_queries:]

        # Expand queries if 1:N scoring
        if len(emb_data_1) == 1:
            emb_data_1 = emb_data_1 * len(emb_data_2)

        final_res_batch: list[PoolingRequestOutput] = []
        padding: list[int] = []
        if (pad_token_id := self.pad_token_id) is not None:
            padding = [pad_token_id]

        # Compute MaxSim scores
        for emb_1, emb_2 in zip(emb_data_1, emb_data_2):
            # emb_1.outputs.data: [query_len, dim]
            # emb_2.outputs.data: [doc_len, dim]
            q_emb = emb_1.outputs.data
            d_emb = emb_2.outputs.data

            maxsim_score = compute_maxsim_score(q_emb, d_emb)

            tokens = emb_1.prompt_token_ids + padding + emb_2.prompt_token_ids

            final_res_batch.append(
                PoolingRequestOutput(
                    request_id=f"{emb_1.request_id}_{emb_2.request_id}",
                    outputs=maxsim_score,
                    prompt_token_ids=tokens,
                    num_cached_tokens=emb_1.num_cached_tokens + emb_2.num_cached_tokens,
                    finished=True,
                )
            )
        return final_res_batch
```
**EN:** Class `LateInteractionIOProcessor` is defined here, extending `BiEncoderIOProcessor`, and groups behavior through methods like `_post_process`.
**CN:** 这里定义类 `LateInteractionIOProcessor`，其职责是封装相关状态与行为，并通过 `_post_process` 等方法组织逻辑。

### Lines 299-333 — Class `FlashLateInteractionIOProcessor`
```python
class FlashLateInteractionIOProcessor(LateInteractionIOProcessor):
    name = "flash-late-interaction"

    def post_process_online(
        self,
        ctx: ScoringServeContext,
    ):
        assert ctx.query_final_res_batch is not None
        assert ctx.final_res_batch is not None
        assert isinstance(ctx.n_queries, int)

        # Expand queries if 1:N scoring
        if len(ctx.query_final_res_batch) == 1:
            ctx.query_final_res_batch = ctx.query_final_res_batch * len(
                ctx.final_res_batch
            )

        final_res_batch: list[PoolingRequestOutput] = []
        for d1, d2 in zip(ctx.query_final_res_batch, ctx.final_res_batch):
            padding: list[int] = []
            if (pad_token_id := self.pad_token_id) is not None:
                padding = [pad_token_id]

            tokens = d1.prompt_token_ids + padding + d2.prompt_token_ids

            final_res_batch.append(
                PoolingRequestOutput(
                    request_id=f"{d1.request_id}_{d2.request_id}",
                    outputs=d2.outputs,
                    prompt_token_ids=tokens,
                    num_cached_tokens=d1.num_cached_tokens + d2.num_cached_tokens,
                    finished=True,
                )
            )
        ctx.final_res_batch = final_res_batch
```
**EN:** Class `FlashLateInteractionIOProcessor` is defined here, extending `LateInteractionIOProcessor`, and groups behavior through methods like `post_process_online`.
**CN:** 这里定义类 `FlashLateInteractionIOProcessor`，其职责是封装相关状态与行为，并通过 `post_process_online` 等方法组织逻辑。

### Lines 336-602 — Class `CrossEncoderIOProcessor`
```python
class CrossEncoderIOProcessor(ScoringIOProcessor):
    name = "cross-encoder"
    pooling_task: PoolingTask = "classify"

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        if is_mistral_tokenizer(self.tokenizer):
            raise ValueError("MistralTokenizer not supported for cross-encoding")

        from vllm.model_executor.model_loader import get_model_cls
        from vllm.model_executor.models.interfaces import supports_score_template

        model = get_model_cls(self.model_config)
        self.supports_score_template = supports_score_template(model)
        self.model = model if self.supports_score_template else None
        self.use_sep_token = self.model_config.use_sep_token

    #######################################
    # online APIs

    def pre_process_online(self, ctx: ScoringServeContext):
        request = ctx.request

        if isinstance(request, ScoreRequest):
            data_1 = request.data_1
            data_2 = request.data_2
        elif isinstance(request, RerankRequest):
...
            self.model.post_process_tokens(engine_prompt)

        if mm_data is not None:
            engine_prompt["multi_modal_data"] = mm_data
        if mm_uuids is not None:
            engine_prompt["multi_modal_uuids"] = mm_uuids

        return full_prompt, engine_prompt
```
**EN:** Class `CrossEncoderIOProcessor` is defined here, extending `ScoringIOProcessor`, and groups behavior through methods like `__init__`, `pre_process_online`, `pre_process_offline`, `_pre_process`.
**CN:** 这里定义类 `CrossEncoderIOProcessor`，其职责是封装相关状态与行为，并通过 `__init__`、`pre_process_online`、`pre_process_offline`、`_pre_process` 等方法组织逻辑。

### Lines 605-676 — Class `JinaRankingIOProcessorMixin`
```python
class JinaRankingIOProcessorMixin:
    @staticmethod
    def sanitize_input(text: str, special_tokens: dict[str, str]) -> str:
        for token in special_tokens.values():
            text = text.replace(token, "")
        return text

    @staticmethod
    def format_docs_prompts_func(
        query: str,
        docs: list[str],
        special_tokens: dict[str, str] | None = None,
        instruction: str | None = None,
        no_thinking: bool = True,
    ) -> str:
        # TODO: Try converting the code below into a chat template.

        default_special_tokens = {
            "query_embed_token": "<|rerank_token|>",
            "doc_embed_token": "<|embed_token|>",
        }
        if special_tokens is None:
            special_tokens = default_special_tokens

        query = JinaRankingIOProcessorMixin.sanitize_input(query, special_tokens)
        docs = [
            JinaRankingIOProcessorMixin.sanitize_input(doc, special_tokens)
            for doc in docs
...
        text: list[str] = []
        for prompt in data:
            if not isinstance(prompt, str):
                raise ValueError(
                    "The JinaForRanking model only supports text as input."
                )
            text.append(prompt)
        return text
```
**EN:** Class `JinaRankingIOProcessorMixin` is defined here, as a standalone type, and groups behavior through methods like `sanitize_input`, `format_docs_prompts_func`, `ensure_str`.
**CN:** 这里定义类 `JinaRankingIOProcessorMixin`，其职责是封装相关状态与行为，并通过 `sanitize_input`、`format_docs_prompts_func`、`ensure_str` 等方法组织逻辑。

### Lines 679-727 — Class `JinaRankingIOProcessor`
```python
class JinaRankingIOProcessor(LateInteractionIOProcessor, JinaRankingIOProcessorMixin):
    name = "jina-reranking-scoring"
    pooling_task: PoolingTask = "token_embed"

    def _pre_process(
        self,
        scoring_data: ScoringData,
        tok_params: TokenizeParams,
        prompt_extras: dict[str, Any] | None = None,
    ) -> Sequence[EngineInput]:
        queries = self.ensure_str(scoring_data.data_1)
        docs = self.ensure_str(scoring_data.data_2)

        if len(queries) == 1:
            prompts = [self.format_docs_prompts_func(query=queries[0], docs=docs)]
        else:
            prompts = [
                self.format_docs_prompts_func(query=q, docs=[d])
                for q, d in zip(queries, docs)
            ]

        return self._preprocess_cmpl_offline(
            prompts=prompts, tok_params=tok_params, prompt_extras=prompt_extras
        )

    def _post_process(self, outputs: list[PoolingRequestOutput], n_queries: int):
        final_res_batch: list[PoolingRequestOutput] = []

...
                        request_id=outputs[i].request_id,
                        outputs=score,
                        prompt_token_ids=outputs[i].prompt_token_ids,
                        num_cached_tokens=outputs[i].num_cached_tokens,
                        finished=True,
                    )
                )
        return final_res_batch
```
**EN:** Class `JinaRankingIOProcessor` is defined here, extending `LateInteractionIOProcessor`, `JinaRankingIOProcessorMixin`, and groups behavior through methods like `_pre_process`, `_post_process`.
**CN:** 这里定义类 `JinaRankingIOProcessor`，其职责是封装相关状态与行为，并通过 `_pre_process`、`_post_process` 等方法组织逻辑。

### Lines 730-739 — Module constants and state
```python
ScoringIOProcessors: dict[str, type[ScoringIOProcessor]] = {
    p.name: p
    for p in [
        BiEncoderIOProcessor,
        LateInteractionIOProcessor,
        JinaRankingIOProcessor,
        FlashLateInteractionIOProcessor,
        CrossEncoderIOProcessor,
    ]
}
```
**EN:** This block initializes `ScoringIOProcessors`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `ScoringIOProcessors`，为后续逻辑准备模块级常量或共享状态。

## Key Concepts / 关键概念
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Batch workflow handling / 批处理工作流
- Pooling task support / 池化任务支持
- Embedding generation / 嵌入生成
- Scoring/ranking logic / 评分/排序逻辑
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `time`, `collections`, `typing`
- **Third-party / 第三方**: `torch`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.inputs`, `vllm.renderers`, `vllm.renderers.hf`, `vllm.tasks`, `vllm.utils.mistral`, `...chat_utils`, `..base.io_processor`, `..typing`, `.protocol`, `.typing`, `.utils`
