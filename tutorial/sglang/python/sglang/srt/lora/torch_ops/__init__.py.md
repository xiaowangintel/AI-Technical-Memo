# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/torch_ops/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides PyTorch-side helper operators for LoRA execution, typically as graph-friendly glue code or non-Triton fallbacks. It helps integrate LoRA math with the broader runtime. / 该文件提供了 LoRA 执行所需的 PyTorch 侧辅助算子，通常作为适配 CUDA Graph 的胶水代码或非 Triton 回退实现。它帮助将 LoRA 计算集成到更大的运行时中。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16: Module header, imports, and shared constants
```python
from typing import Optional

import torch

from sglang.srt.lora.utils import LoRABatchInfo

from .graph_lora_ops import (
    sgemm_lora_a_embedding_graph_fwd,
    sgemm_lora_a_graph_fwd,
    sgemm_lora_b_graph_fwd,
)
from .lora_ops import sgemm_lora_a_embedding_fwd as sgemm_lora_a_embedding_control_fwd
from .lora_ops import sgemm_lora_a_fwd as sgemm_lora_a_control_fwd
from .lora_ops import sgemm_lora_b_fwd as sgemm_lora_b_control_fwd
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 17-43: sgemm lora a embedding fwd function
```python
def sgemm_lora_a_embedding_fwd(
    inputs: torch.Tensor,
    weights: torch.Tensor,
    batch_info: LoRABatchInfo,
    vocab_size: int,
) -> torch.Tensor:
    output: torch.Tensor
    if batch_info.use_cuda_graph:
        output = sgemm_lora_a_embedding_graph_fwd(
            inputs,
            weights,
            batch_info.weight_indices,
            batch_info.seg_lens,
            batch_info.scalings,
            vocab_size,
        )
    else:
        output = sgemm_lora_a_embedding_control_fwd(
            inputs,
            weights,
            batch_info.weight_indices_cpu,
            batch_info.seg_lens_cpu,
            batch_info.lora_ranks_cpu,
            batch_info.scalings_cpu,
            vocab_size,
        )
    return output
```
**EN:** This block uses `sgemm_lora_a_embedding_fwd` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `sgemm_lora_a_embedding_fwd` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 46-72: sgemm lora a fwd function
```python
def sgemm_lora_a_fwd(
    inputs: torch.Tensor,
    weights: torch.Tensor,
    batch_info: LoRABatchInfo,
    num_slices: int = 1,
) -> torch.Tensor:
    output: torch.Tensor
    if batch_info.use_cuda_graph:
        output = sgemm_lora_a_graph_fwd(
            inputs,
            weights,
            batch_info.weight_indices,
            batch_info.seg_lens,
            batch_info.scalings,
            num_slices,
        )
    else:
        output = sgemm_lora_a_control_fwd(
            inputs,
            weights,
            batch_info.weight_indices_cpu,
            batch_info.seg_lens_cpu,
            batch_info.lora_ranks_cpu,
            batch_info.scalings_cpu,
            num_slices,
        )
    return output
```
**EN:** This block uses `sgemm_lora_a_fwd` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `sgemm_lora_a_fwd` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 75-102: sgemm lora b fwd function
```python
def sgemm_lora_b_fwd(
    inputs: torch.Tensor,
    weights: torch.Tensor,
    batch_info: LoRABatchInfo,
    slice_offsets: torch.Tensor,
    base_output: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    output: torch.Tensor
    if batch_info.use_cuda_graph:
        output = sgemm_lora_b_graph_fwd(
            inputs,
            weights,
            batch_info.weight_indices,
            batch_info.seg_lens,
            slice_offsets,
            base_output,
        )
    else:
        output = sgemm_lora_b_control_fwd(
            inputs,
            weights,
            batch_info.weight_indices_cpu,
            batch_info.seg_lens_cpu,
            batch_info.lora_ranks_cpu,
            slice_offsets,
            base_output,
        )
    return output
```
**EN:** This block uses `sgemm_lora_b_fwd` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `sgemm_lora_b_fwd` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 103-109: Module-level supporting statements
```python


__all__ = [
    "sgemm_lora_a_embedding_fwd",
    "sgemm_lora_a_fwd",
    "sgemm_lora_b_fwd",
]
```
**EN:** This block contains supporting statements for the module, such as constants, attributes, registrations, or small helper expressions that shape the surrounding control flow.
**CN:** 该代码块包含模块的配套语句，例如常量、属性、注册逻辑或小型辅助表达式，它们共同塑造了周边控制流程。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- PyTorch operator glue / PyTorch 算子胶水层
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.lora.utils`
### External / 外部
- `graph_lora_ops`
- `lora_ops`
- `torch`
- `typing` (stdlib)
