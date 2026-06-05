# graph_lora_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/torch_ops/graph_lora_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides PyTorch-side helper operators for LoRA execution, typically as graph-friendly glue code or non-Triton fallbacks. It helps integrate LoRA math with the broader runtime. / 该文件提供了 LoRA 执行所需的 PyTorch 侧辅助算子，通常作为适配 CUDA Graph 的胶水代码或非 Triton 回退实现。它帮助将 LoRA 计算集成到更大的运行时中。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6: Module header, imports, and shared constants
```python
from typing import Optional

import torch
import torch.nn.functional as F
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 7-39: sgemm lora a embedding graph fwd function
```python
def sgemm_lora_a_embedding_graph_fwd(
    inputs: torch.Tensor,
    weights: torch.Tensor,
    weight_indices: torch.Tensor,
    seg_len_tensor: torch.Tensor,
    scaling_tensor: torch.Tensor,
    vocab_size: int,
) -> torch.Tensor:
    total_seq_len = inputs.shape[0]
    if weights.numel() == 0:
        return torch.zeros(total_seq_len, 0, dtype=weights.dtype, device=weights.device)

    num_loras, max_rank, _ = weights.shape

    output = torch.zeros(
        total_seq_len, max_rank, dtype=weights.dtype, device=weights.device
    )

    for lora_idx in range(num_loras):

        batch_token_mask = weight_indices[:total_seq_len] == lora_idx

        x_seq = torch.where(batch_token_mask, inputs, 0)
        w_seq = weights[lora_idx]

        output.add_(
            scaling_tensor[lora_idx]
            * torch.where(
                batch_token_mask.unsqueeze(1), F.embedding(x_seq, w_seq.t()), 0
            )
        )

    return output
```
**EN:** This block uses `sgemm_lora_a_embedding_graph_fwd` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `sgemm_lora_a_embedding_graph_fwd` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 42-70: sgemm lora a graph fwd function
```python
def sgemm_lora_a_graph_fwd(
    inputs: torch.Tensor,
    weights: torch.Tensor,
    weight_indices: torch.Tensor,
    seg_len_tensor: torch.Tensor,
    scaling_tensor: torch.Tensor,
    num_slices: int = 1,
) -> torch.Tensor:
    total_seq_len, input_dim = inputs.shape
    if weights.numel() == 0:
        return torch.zeros(total_seq_len, 0, dtype=inputs.dtype, device=inputs.device)

    num_loras, weight_out_dim, _ = weights.shape
    max_rank = weight_out_dim // num_slices

    output = torch.zeros(
        total_seq_len, num_slices * max_rank, dtype=inputs.dtype, device=inputs.device
    )

    for lora_idx in range(num_loras):

        batch_token_mask = (weight_indices[:total_seq_len] == lora_idx).unsqueeze(1)

        x_seq = torch.where(batch_token_mask, inputs, 0)
        w_seq = weights[lora_idx]

        output.add_(scaling_tensor[lora_idx] * torch.mm(x_seq, w_seq.t()))

    return output
```
**EN:** This block uses `sgemm_lora_a_graph_fwd` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `sgemm_lora_a_graph_fwd` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 73-120: sgemm lora b graph fwd function
```python
def sgemm_lora_b_graph_fwd(
    inputs: torch.Tensor,
    weights: torch.Tensor,
    weight_indices: torch.Tensor,
    seg_len_tensor: torch.Tensor,
    slice_offsets: torch.Tensor,
    base_output: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    total_seq_len, input_dim = inputs.shape
    num_loras, weight_out_dim, _ = weights.shape
    total_output_dim = slice_offsets[-1].item() if len(slice_offsets) > 0 else 0

    if weights.numel() == 0:
        return torch.zeros(
            total_seq_len, total_output_dim, dtype=inputs.dtype, device=inputs.device
        )

    num_slices = len(slice_offsets) - 1
    max_rank = input_dim // num_slices

    if base_output is not None:
        output = base_output
    else:
        output = torch.zeros(
            total_seq_len, total_output_dim, dtype=inputs.dtype, device=inputs.device
        )

    for lora_idx in range(num_loras):

        batch_token_mask = (weight_indices[:total_seq_len] == lora_idx).unsqueeze(1)
        inputs_masked = torch.where(batch_token_mask, inputs, 0)

        for slice_idx in range(num_slices):
            slice_start_input = slice_idx * max_rank
            slice_end_input = (slice_idx + 1) * max_rank

            slice_start_output = slice_offsets[slice_idx]
            slice_end_output = slice_offsets[slice_idx + 1]

            x_slice = inputs_masked[..., slice_start_input:slice_end_input]
            w_slice = weights[
                lora_idx, slice_start_output:slice_end_output
            ]  # (slice_dim, max_rank)
            output[..., slice_start_output:slice_end_output].add_(
                torch.mm(x_slice, w_slice.t())
            )

    return output
```
**EN:** This block uses `sgemm_lora_b_graph_fwd` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `sgemm_lora_b_graph_fwd` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- PyTorch operator glue / PyTorch 算子胶水层
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- None / 无
### External / 外部
- `torch`
- `typing` (stdlib)
