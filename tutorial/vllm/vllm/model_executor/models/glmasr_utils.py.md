# glmasr_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/glmasr_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Utility module that supports the glmasr_utils integration inside vLLM. / 工具模块，为 glmasr_utils 在 vLLM 中的集成提供辅助能力。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-8)
```python
from collections.abc import Sequence
from typing import cast

import torch
import torch.nn as nn
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 10-10)
```python
DEFAULT_MAX_AUDIO_LEN_S = 655
```
**EN:** This block defines DEFAULT_MAX_AUDIO_LEN_S, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 DEFAULT_MAX_AUDIO_LEN_S，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 11-11)
```python
DEFAULT_MERGE_FACTOR = 4
```
**EN:** This block defines DEFAULT_MERGE_FACTOR, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 DEFAULT_MERGE_FACTOR，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 14-14)
```python
DEFAULT_CONV_PARAMS = [(1, 3, 1), (1, 3, 2)]
```
**EN:** This block defines DEFAULT_CONV_PARAMS, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 DEFAULT_CONV_PARAMS，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Function `_calculate_conv_output_length` (lines 17-22)
```python
def _calculate_conv_output_length(
    input_length: torch.Tensor, padding: int, kernel_size: int, stride: int
) -> torch.Tensor:
    """Calculate Conv1d output length using standard formula."""
    # in sync with `hf_processor._get_audio_token_length`
    return (input_length + 2 * padding - (kernel_size - 1) - 1) // stride + 1
```
**EN:** Function `_calculate_conv_output_length` encapsulates a focused piece of reusable logic inside this module. The docstring says: Calculate Conv1d output length using standard formula.
**CN:** Function `_calculate_conv_output_length` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Calculate Conv1d output length using standard formula。

### Function `_as_list_chunk_counts` (lines 25-33)
```python
def _as_list_chunk_counts(
    chunk_counts: torch.Tensor | list[int] | list[torch.Tensor],
) -> list[int]:
    if isinstance(chunk_counts, torch.Tensor):
        return chunk_counts.tolist()
    if chunk_counts and isinstance(chunk_counts[0], torch.Tensor):
        tensor_counts = cast(list[torch.Tensor], chunk_counts)
        return [int(c.item()) for c in tensor_counts]
    return [int(c) for c in chunk_counts]
```
**EN:** Function `_as_list_chunk_counts` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_as_list_chunk_counts` 封装了该模块中的一段可复用核心逻辑。

### Function `_normalize_chunk_counts` (lines 36-42)
```python
def _normalize_chunk_counts(
    chunk_counts: torch.Tensor | list[int] | list[torch.Tensor] | None,
    num_chunks: int,
) -> list[int]:
    if chunk_counts is None:
        return [1] * num_chunks
    return _as_list_chunk_counts(chunk_counts)
```
**EN:** Function `_normalize_chunk_counts` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_normalize_chunk_counts` 封装了该模块中的一段可复用核心逻辑。

### Function `_get_audio_output_lengths_from_lengths` (lines 45-54)
```python
def _get_audio_output_lengths_from_lengths(
    audio_lengths: torch.Tensor,
    merge_factor: int,
    conv_params: list[tuple[int, int, int]],
) -> torch.Tensor:
    for padding, kernel_size, stride in conv_params:
        audio_lengths = _calculate_conv_output_length(
            audio_lengths, padding, kernel_size, stride
        )
    return (audio_lengths - merge_factor) // merge_factor + 1
```
**EN:** Function `_get_audio_output_lengths_from_lengths` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_get_audio_output_lengths_from_lengths` 封装了该模块中的一段可复用核心逻辑。

### Function `_get_audio_output_lengths_from_mask` (lines 57-65)
```python
def _get_audio_output_lengths_from_mask(
    mask: torch.Tensor,
    merge_factor: int,
    conv_params: list[tuple[int, int, int]],
) -> torch.Tensor:
    audio_lengths = mask.sum(-1)
    return _get_audio_output_lengths_from_lengths(
        audio_lengths, merge_factor, conv_params
    )
```
**EN:** Function `_get_audio_output_lengths_from_mask` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_get_audio_output_lengths_from_mask` 封装了该模块中的一段可复用核心逻辑。

### Function `_get_audio_output_lengths_for_tower` (lines 68-104)
```python
def _get_audio_output_lengths_for_tower(
    audio_tower: nn.Module,
    audio_lengths: torch.Tensor,
    merge_factor: int,
    conv_params: list[tuple[int, int, int]],
) -> torch.Tensor:
    """
    Calculate the output lengths after audio processing.

    The output length accounts for:
    1. Convolution layers (downsampling)
    2. Merge factor (further downsampling during projection)

    Args:
        audio_tower: The audio encoder module
        audio_lengths: Input feature lengths [batch_size]
        merge_factor: Factor for merging adjacent features
        conv_params: List of (padding, kernel_size, stride) for each conv layer

    Returns:
        Output lengths after all processing [batch_size]
    """
    # First, calculate the output length after convolutions
    if hasattr(audio_tower, "_get_feat_extract_output_lengths"):
        _, conv_output_lengths = audio_tower._get_feat_extract_output_lengths(
            audio_lengths
        )
    else:
        conv_output_lengths = audio_lengths
        for padding, kernel_size, stride in conv_params:
            conv_output_lengths = _calculate_conv_output_length(
                conv_output_lengths, padding, kernel_size, stride
            )

    # Then, apply merge_factor to get final output length
    # Formula: (conv_output_lengths - merge_factor) // merge_factor + 1
    return (conv_output_lengths - merge_factor) // merge_factor + 1
```
**EN:** Function `_get_audio_output_lengths_for_tower` encapsulates a focused piece of reusable logic inside this module. The docstring says: Calculate the output lengths after audio processing.
**CN:** Function `_get_audio_output_lengths_for_tower` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Calculate the output lengths after audio processing。

### Function `_flatten_audio_features_by_length` (lines 107-119)
```python
def _flatten_audio_features_by_length(
    audio_features: torch.Tensor,
    audio_output_lengths: torch.Tensor,
) -> torch.Tensor:
    num_chunks, max_audio_tokens, embed_dim = audio_features.shape
    audio_output_lengths = audio_output_lengths.unsqueeze(1)
    audio_features_mask = (
        torch.arange(max_audio_tokens)
        .expand(num_chunks, max_audio_tokens)
        .to(audio_output_lengths.device)
        < audio_output_lengths
    )
    return audio_features[audio_features_mask].view(-1, embed_dim)
```
**EN:** Function `_flatten_audio_features_by_length` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_flatten_audio_features_by_length` 封装了该模块中的一段可复用核心逻辑。

### Function `_group_audio_embeddings` (lines 122-132)
```python
def _group_audio_embeddings(
    chunk_embeddings: Sequence[torch.Tensor],
    chunk_counts: Sequence[int],
) -> tuple[torch.Tensor, ...]:
    grouped_embeddings = []
    current_idx = 0
    for count in chunk_counts:
        audio_chunks = chunk_embeddings[current_idx : current_idx + count]
        grouped_embeddings.append(torch.cat(audio_chunks, dim=0))
        current_idx += count
    return tuple(grouped_embeddings)
```
**EN:** Function `_group_audio_embeddings` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Function `_group_audio_embeddings` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Sequence`, `from typing import cast`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`
