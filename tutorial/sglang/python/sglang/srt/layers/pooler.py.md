# pooler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/pooler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements embedding lookup and preprocessing with pooling helpers for the SGLang SRT runtime. It exposes symbols such as `PoolingType`, `EmbeddingPoolerOutput`, `pool_hidden_states`, and `pool_at_delimiter_positions` and connects them to backend-specific paths such as `CUDA`, `NPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了嵌入查找与预处理，并结合池化辅助逻辑。它提供了 `PoolingType`、`EmbeddingPoolerOutput`、`pool_hidden_states` 以及 `pool_at_delimiter_positions` 等符号，并把这些符号连接到 `CUDA`、`NPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: File header and module overview
```python
# adapted from
# https://github.com/vllm-project/vllm/blob/82a1b1a82b1fbb454c82a9ef95730b929c9b270c/vllm/model_executor/layers/pooler.py
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 4-19: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

from dataclasses import dataclass
from enum import IntEnum
from typing import TYPE_CHECKING, List, Optional

import torch
import torch.nn as nn
from transformers import PretrainedConfig

from sglang.srt.layers.activation import get_cross_encoder_activation_function

if TYPE_CHECKING:
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `dataclasses.dataclass`, `enum.IntEnum`, `typing.TYPE_CHECKING`, `typing.List`, and `typing.Optional`, so later blocks can reuse runtime, tensor, or backend helpers. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`dataclasses.dataclass`、`enum.IntEnum`、`typing.TYPE_CHECKING`、`typing.List` 以及 `typing.Optional`，让后续代码可以复用运行时、张量或后端辅助逻辑。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 20-24: Class `PoolingType` declaration and shared state
```python
class PoolingType(IntEnum):
    LAST = 0
    CLS = 1
```
**EN:** This block introduces class `PoolingType` and the state shared by its methods. It inherits from `IntEnum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `PoolingType`，并定义其方法共享的状态。 它继承自 `IntEnum`，说明了它在 SRT 层栈中的接入方式。

### Lines 25-45: Class `EmbeddingPoolerOutput` declaration and shared state
```python
@dataclass
class EmbeddingPoolerOutput:
    """Output of pooler or score_and_pool.

    Attributes:
        embeddings: Pooled embeddings or classification logits.  May be a list
            of tensors when per-request matryoshka dim truncation produces
            different shapes, or when MIS yields a variable number of scores
            per request.
        pooled_hidden_states: Raw transformer hidden states *before* the
            task-specific head, present only when
            ``forward_batch.return_pooled_hidden_states`` is True.  Tensor
            (standard path) or list of tensors (MIS path, one per delimiter).
    """

    # Pooler can return list[tensor] instead of tensor if the dimension of each tensor in the batch is different
    # due to different per-request matryoshka dim truncation
    embeddings: torch.Tensor | list[torch.Tensor]
    pooled_hidden_states: Optional[torch.Tensor | list[torch.Tensor]] = None
```
**EN:** This block introduces class `EmbeddingPoolerOutput` and the state shared by its methods. The class docstring summarizes its role: Output of pooler or score_and_pool.
**CN:** 该代码块引入类 `EmbeddingPoolerOutput`，并定义其方法共享的状态。 类级文档进一步概括了它的职责。

### Lines 46-67: Function `pool_hidden_states` and its core logic
```python
def pool_hidden_states(
    pooling_type: PoolingType,
    hidden_states: torch.Tensor,
    forward_batch: ForwardBatch,
) -> torch.Tensor:
    """Pool hidden_states by PoolingType (LAST/CLS).

    Raw pooling only — no normalize, no dim truncation.
    Returns shape (batch_size, hidden_size).
    """
    if pooling_type == PoolingType.LAST:
        last_token_indices = torch.cumsum(forward_batch.extend_seq_lens, dim=0) - 1
        return hidden_states[last_token_indices]
    elif pooling_type == PoolingType.CLS:
        prompt_lens = forward_batch.extend_seq_lens
        first_token_flat_indices = torch.zeros_like(prompt_lens)
        first_token_flat_indices[1:] += torch.cumsum(prompt_lens, dim=0)[:-1]
        return hidden_states[first_token_flat_indices]
    else:
        raise ValueError(f"Unsupported pooling type: {pooling_type}")
```
**EN:** This block defines `pool_hidden_states` and contains the main logic for this step. It mainly invokes `torch.cumsum`, `torch.zeros_like`, and `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `last_token_indices`, `prompt_lens`, and `first_token_flat_indices` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pool_hidden_states`，并承载这一阶段的核心逻辑。 它主要调用 `torch.cumsum`、`torch.zeros_like` 以及 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `last_token_indices`、`prompt_lens` 以及 `first_token_flat_indices` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 68-107: Function `pool_at_delimiter_positions` and its core logic
```python
def pool_at_delimiter_positions(
    data: torch.Tensor,
    forward_batch: ForwardBatch,
    device: torch.device,
) -> List[torch.Tensor]:
    """Pool a tensor at the position before each MIS delimiter for every request.

    Uses pre-computed delimiter indices from ForwardBatch (CPU tensors),
    moves to GPU with non_blocking=True to avoid CUDA syncs.

    Args:
        data: 2-D tensor [total_tokens, dim] — hidden states or logits.
        forward_batch: Forward batch with extend_seq_lens_cpu and
                       multi_item_delimiter_indices populated.
        device: Device for the index tensor.

    Returns:
        One tensor per request, shaped [num_delimiters, dim].
    """
    all_index_tensors: List[torch.Tensor] = []
    delim_counts: List[int] = []
    offset = 0
    for req_idx, req_seq_len in enumerate(forward_batch.extend_seq_lens_cpu):
        indices_tensor = forward_batch.multi_item_delimiter_indices[req_idx]
        n = len(indices_tensor)
        if n > 0:
            # Note: if the first delimiter is at position 0 (empty query),
            # indices - 1 wraps to -1. This is harmless — the first delimiter
            # entry is always discarded by _process_multi_item_scoring_results.
            all_index_tensors.append(indices_tensor + (offset - 1))
        delim_counts.append(n)
        offset += req_seq_len

    if all_index_tensors:
        index_tensor = torch.cat(all_index_tensors).to(device, non_blocking=True)
    else:
        index_tensor = torch.tensor([], dtype=torch.long, device=device)
    return list(data[index_tensor].split(delim_counts))
```
**EN:** This block defines `pool_at_delimiter_positions` and contains the main logic for this step. It mainly invokes `enumerate`, `list`, `len`, `delim_counts.append`, and `torch.cat.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `all_index_tensors`, `delim_counts`, `offset`, `indices_tensor`, and `n` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pool_at_delimiter_positions`，并承载这一阶段的核心逻辑。 它主要调用 `enumerate`、`list`、`len`、`delim_counts.append` 以及 `torch.cat.to`，说明该流程会编排底层辅助函数或计算内核。 像 `all_index_tensors`、`delim_counts`、`offset`、`indices_tensor` 以及 `n` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 108-158: Function `score_and_pool` and its core logic
```python
def score_and_pool(
    score_head: nn.Module,
    pooler: "Pooler",
    hidden_states: torch.Tensor,
    forward_batch: ForwardBatch,
    input_ids: torch.Tensor,
) -> EmbeddingPoolerOutput:
    """Apply a classification/score head with MIS and pooled-hidden-states support.

    MIS path (pre-computed delimiter indices on forward_batch): extract hidden
    states at positions just before each delimiter, apply the score head, then
    split per-request.

    Standard path: pool hidden states, then apply the score head.

    When ``forward_batch.return_pooled_hidden_states`` is True, the raw pooled
    hidden states (before the score head) are included in the output.
    """
    if (
        forward_batch.multi_item_delimiter_indices is not None
        and forward_batch.is_prefill_only
    ):
        # Pool hidden states at pre-delimiter positions, score only those —
        # avoids wasting compute on tokens that never contribute to the output.
        # pool_at_delimiter_positions returns one tensor per request; we concat
        # to call score_head once, then split back per request.
        per_request_phs = pool_at_delimiter_positions(
            hidden_states, forward_batch, input_ids.device
        )
        phs_flat = torch.cat(per_request_phs, dim=0)
        scores_flat = score_head(phs_flat)
        delim_counts = [t.shape[0] for t in per_request_phs]
        per_request_scores = list(scores_flat.split(delim_counts))
        return EmbeddingPoolerOutput(
            embeddings=per_request_scores,
            pooled_hidden_states=(
                per_request_phs if forward_batch.return_pooled_hidden_states else None
            ),
        )

    # Standard classification path: pool hidden states, then score.
    pooled_hs = pool_hidden_states(pooler.pooling_type, hidden_states, forward_batch)
    scores = score_head(pooled_hs)
    return EmbeddingPoolerOutput(
        embeddings=scores,
        pooled_hidden_states=(
            pooled_hs if forward_batch.return_pooled_hidden_states else None
        ),
    )
```
**EN:** This block defines `score_and_pool` and contains the main logic for this step. It mainly invokes `pool_hidden_states`, `score_head`, `EmbeddingPoolerOutput`, `pool_at_delimiter_positions`, and `torch.cat`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pooled_hs`, `scores`, `per_request_phs`, `phs_flat`, and `scores_flat` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `score_and_pool`，并承载这一阶段的核心逻辑。 它主要调用 `pool_hidden_states`、`score_head`、`EmbeddingPoolerOutput`、`pool_at_delimiter_positions` 以及 `torch.cat`，说明该流程会编排底层辅助函数或计算内核。 像 `pooled_hs`、`scores`、`per_request_phs`、`phs_flat` 以及 `scores_flat` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 159-169: Class `Pooler` declaration and shared state
```python
class Pooler(nn.Module):
    """A layer that pools specific information from hidden states.
    This layer does the following:
    1. Extracts specific tokens or aggregates data based on pooling method.
    2. Normalizes output if specified.
    3. Returns structured results as `PoolerOutput`.
    Attributes:
        pooling_type: The type of pooling to use (LAST, AVERAGE, MAX).
        normalize: Whether to normalize the pooled data.
    """
```
**EN:** This block introduces class `Pooler` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: A layer that pools specific information from hidden states.
**CN:** 该代码块引入类 `Pooler`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 170-174: `Pooler` initialization and state setup
```python
    def __init__(self, pooling_type: PoolingType, normalize: bool):
        super().__init__()
        self.pooling_type = pooling_type
        self.normalize = normalize
```
**EN:** This block defines `Pooler.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.pooling_type` and `self.normalize` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Pooler.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.pooling_type` 和 `self.normalize` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 175-203: `Pooler.forward` main forward path
```python
    def forward(
        self, hidden_states: torch.Tensor, forward_batch: ForwardBatch
    ) -> EmbeddingPoolerOutput:
        pooled_data = pool_hidden_states(
            self.pooling_type, hidden_states, forward_batch
        )

        if forward_batch.dimensions is not None:
            all_same_dimensions = len(set(forward_batch.dimensions)) == 1
            if all_same_dimensions:
                pooled_data = pooled_data[..., : forward_batch.dimensions[0]]
            else:
                pooled_data = [
                    tensor[..., :dim]
                    for tensor, dim in zip(pooled_data, forward_batch.dimensions)
                ]

        if self.normalize:
            if isinstance(pooled_data, list):
                pooled_data = [
                    nn.functional.normalize(tensor, p=2, dim=-1)
                    for tensor in pooled_data
                ]
            else:
                pooled_data = nn.functional.normalize(pooled_data, p=2, dim=-1)

        return EmbeddingPoolerOutput(embeddings=pooled_data)
```
**EN:** This block defines `Pooler.forward` and contains the main logic for this step. It mainly invokes `pool_hidden_states`, `EmbeddingPoolerOutput`, `isinstance`, `len`, and `nn.functional.normalize`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pooled_data` and `all_same_dimensions` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Pooler.forward`，并承载这一阶段的核心逻辑。 它主要调用 `pool_hidden_states`、`EmbeddingPoolerOutput`、`isinstance`、`len` 以及 `nn.functional.normalize`，说明该流程会编排底层辅助函数或计算内核。 像 `pooled_data` 和 `all_same_dimensions` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 204-212: Class `CrossEncodingPooler` declaration and shared state
```python
class CrossEncodingPooler(nn.Module):
    """A layer that pools specific information from hidden states.

    This layer does the following:
    1. Extracts specific tokens or aggregates data based on pooling method.
    2. Normalizes output if specified.
    3. Returns structured results as `EmbeddingPoolerOutput`.
    """
```
**EN:** This block introduces class `CrossEncodingPooler` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: A layer that pools specific information from hidden states.
**CN:** 该代码块引入类 `CrossEncodingPooler`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 213-223: `CrossEncodingPooler` initialization and state setup
```python
    def __init__(
        self,
        config: PretrainedConfig,
        classifier: nn.Module,
        pooler: Optional[nn.Module] = None,
    ):
        super().__init__()
        self.classifier = classifier
        self.pooler = pooler
        self.default_activation_function = get_cross_encoder_activation_function(config)
```
**EN:** This block defines `CrossEncodingPooler.__init__` and contains the main logic for this step. It mainly invokes `super.__init__` and `get_cross_encoder_activation_function`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.classifier`, `self.pooler`, and `self.default_activation_function` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `CrossEncodingPooler.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__` 和 `get_cross_encoder_activation_function`，说明该流程会编排底层辅助函数或计算内核。 像 `self.classifier`、`self.pooler` 以及 `self.default_activation_function` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 224-254: `CrossEncodingPooler.forward` main forward path
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> EmbeddingPoolerOutput:
        """Pools sentence pair scores from the hidden_states."""

        prompt_lens = forward_batch.extend_seq_lens

        offset = 0
        pooled_data_lst = []
        for prompt_len in prompt_lens:
            pooled_data_i = hidden_states[offset : offset + prompt_len]

            if self.pooler is not None:
                final_shape_tensor = self.pooler(pooled_data_i, forward_batch)
            else:
                final_shape_tensor = self.classifier(pooled_data_i)

            pooled_data_lst.append(final_shape_tensor)
            offset += prompt_len

        pooled_output = torch.stack(pooled_data_lst)

        if self.pooler is not None:
            # apply classifier once on the full batch if possible
            pooled_output = self.classifier(pooled_output)

        scores = self.default_activation_function(pooled_output).squeeze(-1)

        return EmbeddingPoolerOutput(embeddings=scores)
```
**EN:** This block defines `CrossEncodingPooler.forward` and contains the main logic for this step. It mainly invokes `torch.stack`, `self.default_activation_function.squeeze`, `EmbeddingPoolerOutput`, `pooled_data_lst.append`, and `self.classifier`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `prompt_lens`, `offset`, `pooled_data_lst`, `pooled_output`, and `scores` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CrossEncodingPooler.forward`，并承载这一阶段的核心逻辑。 它主要调用 `torch.stack`、`self.default_activation_function.squeeze`、`EmbeddingPoolerOutput`、`pooled_data_lst.append` 以及 `self.classifier`，说明该流程会编排底层辅助函数或计算内核。 像 `prompt_lens`、`offset`、`pooled_data_lst`、`pooled_output` 以及 `scores` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `PoolingType`, `EmbeddingPoolerOutput`, `pool_hidden_states`, `pool_at_delimiter_positions`, and `score_and_pool`. / **主要符号**：核心入口包括 `PoolingType`、`EmbeddingPoolerOutput`、`pool_hidden_states`、`pool_at_delimiter_positions` 以及 `score_and_pool`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Activation math**: Captures fused nonlinear transforms and their device-specific fast paths. / **激活数学**：描述融合非线性变换及其设备专用快速路径。
- **Sampling policy**: Covers post-processing of logits and token selection strategies. / **采样策略**：涵盖 logits 后处理与 token 选择策略。
- **Embedding pipeline**: Describes how IDs or features are mapped into model-space tensors. / **嵌入流水线**：说明 ID 或特征如何映射到模型空间张量。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `dataclasses.dataclass`, `enum.IntEnum`, `typing.TYPE_CHECKING`, `typing.List`, and `typing.Optional` / **标准库**：`__future__.annotations`、`dataclasses.dataclass`、`enum.IntEnum`、`typing.TYPE_CHECKING`、`typing.List` 以及 `typing.Optional`
- **Third-party**: `torch`, `torch.nn`, and `transformers.PretrainedConfig` / **第三方依赖**：`torch`、`torch.nn` 以及 `transformers.PretrainedConfig`
- **Internal SGLang modules**: `sglang.srt.layers.activation.get_cross_encoder_activation_function` and `sglang.srt.model_executor.forward_batch_info.ForwardBatch` / **SGLang 内部模块**：`sglang.srt.layers.activation.get_cross_encoder_activation_function` 和 `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
