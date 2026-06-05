# metadata.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/sample/metadata.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `SamplingMetadata` for the V1 `sample` subsystem. / 为 V1 的 `sample` 子系统实现 `SamplingMetadata`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from __future__ import annotations

from dataclasses import dataclass

import torch

from vllm.v1.sample.logits_processor import LogitsProcessors
from vllm.v1.sample.thinking_budget_state import ThinkingBudgetStateHolder
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.v1.sample.logits_processor`, `vllm.v1.sample.thinking_budget_state`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.v1.sample.logits_processor`, `vllm.v1.sample.thinking_budget_state` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `SamplingMetadata` class / `SamplingMetadata` 类
```python
@dataclass
class SamplingMetadata:
    temperature: torch.Tensor | None
    all_greedy: bool
    all_random: bool

    top_p: torch.Tensor | None
    top_k: torch.Tensor | None

    generators: dict[int, torch.Generator]

    # None means no logprobs, 0 means sampled token logprobs only
    max_num_logprobs: int | None

    no_penalties: bool
    prompt_token_ids: torch.Tensor | None
    frequency_penalties: torch.Tensor
    presence_penalties: torch.Tensor
    repetition_penalties: torch.Tensor

    output_token_ids: list[list[int]]

    # `allowed_token_ids_mask` is a 2D bool tensor of shape (max batch size,
    # vocab size).
    allowed_token_ids_mask: torch.Tensor | None

    # req_index -> bad_words_token_ids
    bad_words_token_ids: dict[int, list[list[int]]]

    # Loaded logits processors
    logitsprocs: LogitsProcessors

    # Specific token IDs to compute logprobs for (more efficient than full vocab)
    # When set, logprobs are computed only for these token IDs using gather
    # req_index -> list of token IDs to get logprobs for
    logprob_token_ids: dict[int, list[int]] | None = None

    # Speculative token ids
    spec_token_ids: list[list[int]] | None = None
    # When non-None, use ``holder.has_tracked_requests()`` to see if this batch applies
    # thinking-token-budget logits (holder may exist with an empty tracking set).
    thinking_budget_state_holder: ThinkingBudgetStateHolder | None = None
```
**EN:** Uses `@dataclass` to package related state for `SamplingMetadata`. Typical fields include `temperature`, `all_greedy`, `all_random`, `top_p`, `top_k`, `generators`.
**CN:** `SamplingMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `temperature`, `all_greedy`, `all_random`, `top_p`, `top_k`, `generators`。

## Key Concepts / 关键概念
- `SamplingMetadata`: central class or interface in this module. / `SamplingMetadata`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `__future__`, `dataclasses`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.v1.sample.logits_processor`, `vllm.v1.sample.thinking_budget_state`
