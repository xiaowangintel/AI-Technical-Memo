# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/scoring/utils.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Provides reusable helper utilities shared across related entrypoints. Scope: pooling scoring. / 提供在相关入口之间复用的辅助工具函数。 作用域：池化 / 评分。

## Line-by-Line Analysis / 逐行分析
### Lines 3-25 — Imports and shared dependencies
```python
from collections.abc import Iterable
from typing import cast

import torch

from vllm import PromptType, TextPrompt
from vllm.config import ModelConfig
from vllm.entrypoints.chat_utils import (
    BaseMultiModalItemTracker,
    ChatCompletionContentPartParam,
    ChatCompletionContentPartTextParam,
    ConversationMessage,
    MultiModalItemTracker,
    _parse_chat_message_content_parts,
)
from vllm.inputs import MultiModalDataDict, MultiModalUUIDDict

from .typing import (
    ScoreContentPartParam,
    ScoreData,
    ScoreInput,
    ScoringData,
)
```
**EN:** This import block pulls in standard-library modules such as `collections`, `typing`, uses third-party packages like `torch`, depends on internal helpers such as `vllm`, `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.inputs`, `.typing`.
**CN:** 该导入块引入 `collections`, `typing` 等标准库模块，使用 `torch` 等第三方库，依赖 `vllm`, `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.inputs`, `.typing` 等 vLLM 内部模块。

### Lines 28-38 — Function `get_num_special_tokens_for_pair`
```python
def get_num_special_tokens_for_pair(tokenizer) -> int:
    """Get number of special tokens added for a text pair encoding."""
    method = getattr(tokenizer, "num_special_tokens_to_add", None)
    if method is not None:
        try:
            return method(pair=True)
        except TypeError:
            pass
    # Fallback: compute by tokenizing empty strings
    empty_encoding = tokenizer("", text_pair="", add_special_tokens=True)
    return len(empty_encoding["input_ids"])
```
**EN:** This function `get_num_special_tokens_for_pair` is documented as: Get number of special tokens added for a text pair encoding.
**CN:** 这里定义函数 `get_num_special_tokens_for_pair`，其文档字符串说明了主要职责与调用约定。

### Lines 41-56 — Function `truncate_text_to_tokens`
```python
def truncate_text_to_tokens(
    text: str,
    tokenizer,
    max_tokens: int,
) -> str:
    """Truncate text to a maximum number of content tokens.

    Uses offset_mapping to slice the original text at the exact character
    boundary, avoiding lossy encode→decode round-trips that can shift
    the token count by 1-3 tokens due to BPE merge boundary changes.
    """
    encoding = tokenizer(text, add_special_tokens=False, return_offsets_mapping=True)
    if len(encoding["input_ids"]) <= max_tokens:
        return text
    char_end = encoding["offset_mapping"][max_tokens - 1][1]
    return text[:char_end]
```
**EN:** This function `truncate_text_to_tokens` is documented as: Truncate text to a maximum number of content tokens.
**CN:** 这里定义函数 `truncate_text_to_tokens`，其文档字符串说明了主要职责与调用约定。

### Lines 59-74 — Function `compute_maxsim_score`
```python
def compute_maxsim_score(q_emb: torch.Tensor, d_emb: torch.Tensor) -> torch.Tensor:
    """
    Compute ColBERT MaxSim score.

    Args:
        q_emb: Query token embeddings [query_len, dim]
        d_emb: Document token embeddings [doc_len, dim]

    Returns:
        MaxSim score (sum over query tokens of max similarity to any doc token)
    """
    # compute in float32 for numerical stability
    # [query_len, doc_len]
    token_scores = torch.matmul(q_emb.float(), d_emb.float().T)
    # Max over document tokens, sum over query tokens
    return token_scores.amax(dim=-1).sum()
```
**EN:** This function `compute_maxsim_score` is documented as: Compute ColBERT MaxSim score.
**CN:** 这里定义函数 `compute_maxsim_score`，其文档字符串说明了主要职责与调用约定。

### Lines 77-91 — Function `_validate_mm_score_input`
```python
def _validate_mm_score_input(
    data: list[ScoreInput],
    is_multimodal_model: bool,
    architecture: str,
) -> list[ScoreData]:
    out: list[ScoreData] = []
    for d in data:
        if isinstance(d, str):
            out.append(d)
        else:
            if not is_multimodal_model:
                raise ValueError(f"MultiModalParam is not supported for {architecture}")
            content = cast(list[ScoreContentPartParam], d.get("content", []))
            out.append(content)
    return out
```
**EN:** This function `_validate_mm_score_input` checks or normalizes inputs.
**CN:** 该函数 `_validate_mm_score_input` 检查或规范化输入。

### Lines 94-106 — Function `_validate_score_input_lens`
```python
def _validate_score_input_lens(
    data_1: list[ScoreData],
    data_2: list[ScoreData],
):
    len_1 = len(data_1)
    len_2 = len(data_2)

    if len_1 > 1 and len_1 != len_2:
        raise ValueError("Input lengths must be either 1:1, 1:N or N:N")
    if len_1 == 0:
        raise ValueError("At least one text element must be given")
    if len_2 == 0:
        raise ValueError("At least one text_pair element must be given")
```
**EN:** This function `_validate_score_input_lens` checks or normalizes inputs.
**CN:** 该函数 `_validate_score_input_lens` 检查或规范化输入。

### Lines 109-124 — Function `validate_score_input`
```python
def validate_score_input(
    data_1: ScoreInput | list[ScoreInput],
    data_2: ScoreInput | list[ScoreInput],
    is_multimodal_model: bool,
    architecture: str,
) -> ScoringData:
    if not isinstance(data_1, list):
        data_1 = [data_1]

    if not isinstance(data_2, list):
        data_2 = [data_2]

    score_input_1 = _validate_mm_score_input(data_1, is_multimodal_model, architecture)
    score_input_2 = _validate_mm_score_input(data_2, is_multimodal_model, architecture)
    _validate_score_input_lens(score_input_1, score_input_2)
    return ScoringData(data_1=score_input_1, data_2=score_input_2)
```
**EN:** This function `validate_score_input` checks or normalizes inputs.
**CN:** 该函数 `validate_score_input` 检查或规范化输入。

### Lines 127-154 — Function `score_data_to_prompts`
```python
def score_data_to_prompts(
    data_list: list[ScoreData],
    role: str,
    model_config: ModelConfig,
) -> list[PromptType]:
    """Convert a list of ScoreData into PromptType objects.

    For plain text inputs, returns the string directly.
    For multimodal inputs (list of content parts), parses them into
    a :class:`TextPrompt` with attached ``multi_modal_data`` /
    ``multi_modal_uuids``.

    This is used by late-interaction scoring where each query/document
    is encoded independently.
    """
    prompts: list[PromptType] = []
    for data in data_list:
        if isinstance(data, str):
            prompts.append(data)
        else:
            text, mm_data, mm_uuids = parse_score_data_single(data, role, model_config)
            prompt: TextPrompt = TextPrompt(prompt=text)
            if mm_data is not None:
                prompt["multi_modal_data"] = mm_data
            if mm_uuids is not None:
                prompt["multi_modal_uuids"] = mm_uuids
            prompts.append(prompt)
    return prompts
```
**EN:** This function `score_data_to_prompts` is documented as: Convert a list of ScoreData into PromptType objects.
**CN:** 这里定义函数 `score_data_to_prompts`，其文档字符串说明了主要职责与调用约定。

### Lines 157-163 — Function `_ensure_str`
```python
def _ensure_str(content: list[ConversationMessage]) -> str:
    """Extract a single string prompt from parsed conversation content."""
    assert len(content) == 1
    prompt = content[0]["content"]
    if prompt is not None and isinstance(prompt, str):
        return cast(str, prompt)
    raise ValueError(f"Only string content is supported, but got {content}.")
```
**EN:** This function `_ensure_str` is documented as: Extract a single string prompt from parsed conversation content.
**CN:** 这里定义函数 `_ensure_str`，其文档字符串说明了主要职责与调用约定。

### Lines 166-199 — Function `_parse_score_content`
```python
def _parse_score_content(
    role: str,
    data: ScoreData,
    mm_tracker: BaseMultiModalItemTracker,
) -> list[ConversationMessage]:
    parts: Iterable[ChatCompletionContentPartParam]
    if isinstance(data, str):
        parts = [ChatCompletionContentPartTextParam(type="text", text=data)]
    else:
        parts = cast(Iterable[ChatCompletionContentPartParam], data)

    mm_parser = mm_tracker.create_parser()

    parse_res = _parse_chat_message_content_parts(
        role=role,
        parts=parts,
        mm_tracker=mm_tracker,
        wrap_dicts=False,
        interleave_strings=False,
        multimodal_content_part_separator="",
    )

    if parse_res:
        return parse_res

    mm_placeholder_storage = mm_parser.mm_placeholder_storage()

    if (
        len(mm_placeholder_storage) != 1
        or len(next(iter(mm_placeholder_storage.values()))) != 1
    ):
        raise ValueError("Only one multi-modal item is supported")

    return next(iter(mm_placeholder_storage.values()))[0]
```
**EN:** This function `_parse_score_content` checks or normalizes inputs.
**CN:** 该函数 `_parse_score_content` 检查或规范化输入。

### Lines 202-220 — Function `parse_score_data_single`
```python
def parse_score_data_single(
    data: ScoreData,
    role: str,
    model_config: ModelConfig,
) -> tuple[str, MultiModalDataDict | None, MultiModalUUIDDict | None]:
    """Parse **one** ScoreData into a text prompt and its own multi-modal
    data.

    Unlike :func:`parse_score_data`, each call creates an **independent**
    :class:`MultiModalItemTracker` so multi-modal items are kept separate.
    This is the correct behaviour for late-interaction scoring, where
    query and document are encoded independently.
    """
    mm_tracker = MultiModalItemTracker(model_config)
    content = _parse_score_content(role, data, mm_tracker)

    prompt = _ensure_str(content)
    mm_items, mm_uuids = mm_tracker.resolve_items()
    return prompt, mm_items, mm_uuids
```
**EN:** This function `parse_score_data_single` is documented as: Parse **one** ScoreData into a text prompt and its own multi-modal data.
**CN:** 这里定义函数 `parse_score_data_single`，其文档字符串说明了主要职责与调用约定。

### Lines 223-245 — Function `parse_score_data`
```python
def parse_score_data(
    data_1: ScoreData,
    data_2: ScoreData,
    model_config: ModelConfig,
) -> tuple[str, str, MultiModalDataDict | None, MultiModalUUIDDict | None]:
    """Parse a query-document pair into text prompts and shared multi-modal
    data.

    Uses a **single** :class:`MultiModalItemTracker` so that multi-modal
    items from both inputs are merged into one ``mm_data`` dict.  This is
    the correct behaviour for cross-encoder scoring, where query and
    document are concatenated into a single model prompt.
    """
    mm_tracker = MultiModalItemTracker(model_config)

    content_1 = _parse_score_content("query", data_1, mm_tracker)
    content_2 = _parse_score_content("document", data_2, mm_tracker)

    prompt_1 = _ensure_str(content_1)
    prompt_2 = _ensure_str(content_2)
    mm_items, mm_uuids = mm_tracker.resolve_items()

    return prompt_1, prompt_2, mm_items, mm_uuids
```
**EN:** This function `parse_score_data` is documented as: Parse a query-document pair into text prompts and shared multi-modal data.
**CN:** 这里定义函数 `parse_score_data`，其文档字符串说明了主要职责与调用约定。

### Lines 248-266 — Function `compress_token_type_ids`
```python
def compress_token_type_ids(token_type_ids: list[int]) -> int:
    """
    Return position of the first 1 or the length of the list
    if not found.
    """
    first_one = len(token_type_ids)
    err_msg = (
        "Token type ids are expected to be a sequence"
        " of zeros followed by a sequence of ones"
    )
    for i, type_id in enumerate(token_type_ids):
        if type_id == 0 and first_one < i:
            raise ValueError(err_msg)
        elif type_id == 1 and first_one > i:
            first_one = i
        elif type_id > 1:
            raise ValueError(err_msg)

    return first_one
```
**EN:** This function `compress_token_type_ids` is documented as: Return position of the first 1 or the length of the list if not found.
**CN:** 这里定义函数 `compress_token_type_ids`，其文档字符串说明了主要职责与调用约定。

## Key Concepts / 关键概念
- Tokenization or token-level processing / 分词或 token 级处理
- Embedding generation / 嵌入生成
- Scoring/ranking logic / 评分/排序逻辑
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections`, `typing`
- **Third-party / 第三方**: `torch`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.inputs`, `.typing`
