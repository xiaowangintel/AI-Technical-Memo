# moe_align_block_size.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/triton_utils/moe_align_block_size.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `moe_align_block_size` and connects them to backend-specific paths such as `CUDA`, `Triton`, and `XPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `moe_align_block_size` 等符号，并把这些符号连接到 `CUDA`、`Triton` 以及 `XPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

from typing import Tuple

import torch
import triton

from sglang.srt.utils import is_cuda, is_hip, is_musa, is_xpu

_is_cuda = is_cuda()
_is_hip = is_hip()
_is_xpu = is_xpu()
_is_musa = is_musa()

if _is_cuda or _is_hip or _is_xpu or _is_musa:
    from sgl_kernel import moe_align_block_size as sgl_moe_align_block_size
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `typing.Tuple`, `torch`, `triton`, `sglang.srt.utils.is_cuda`, and `sglang.srt.utils.is_hip`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_cuda`, `_is_hip`, `_is_xpu`, and `_is_musa` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`typing.Tuple`、`torch`、`triton`、`sglang.srt.utils.is_cuda` 以及 `sglang.srt.utils.is_hip`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_cuda`、`_is_hip`、`_is_xpu` 以及 `_is_musa` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 19-87: Function `moe_align_block_size` and its core logic
```python
def moe_align_block_size(
    topk_ids: torch.Tensor, block_size: int, num_experts: int
) -> Tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Aligns the token distribution across experts to be compatible with block
    size for matrix multiplication.

    Parameters:
    - topk_ids: A tensor of shape [total_tokens, top_k] representing the
        top-k expert indices for each token.
    - block_size: The block size used in block matrix multiplication.
    - num_experts: The total number of experts.

    Returns:
    - sorted_token_ids: A tensor containing the sorted token indices according
        to their allocated expert.
    - expert_ids: A tensor indicating the assigned expert index for each block.
    - num_tokens_post_padded: The total number of tokens after padding,
        ensuring divisibility by block_size.

    This function pads the number of tokens that each expert needs to process
    so that it is divisible by block_size.
    Padding ensures that during block matrix multiplication, the dimensions
    align correctly.

    Example:
    Given topk_ids = [[2, 3, 4], [1, 2, 4], [1, 3, 4], [1, 2, 3]],
    block_size = 4, and num_experts = 4:
    - We initially have 12 tokens (after repeating 'top_k' times) and 4 experts,
        with each expert needing to process 3 tokens.
    - As block_size is 4, we pad 1 token for each expert.
    - First, flatten topk_ids to [2, 3, 4, 1, 2, 4, 1, 3, 4, 1, 2, 3].
    - Then append padding tokens [12, 12, 12, 12] for each block.
    - After sorting by expert index, we obtain token_ids
        [3, 6, 9, 12, 0, 4, 10, 12, 1, 7, 11, 12, 2, 5, 8, 12].
        Tokens 12 are non-existent (padding) and are ignored in
        the subsequent matrix multiplication.
    - The padding ensures that the total number of tokens is now divisible
        by block_size for proper block matrix operations.
    """
    if topk_ids.numel() < num_experts + 1:
        max_num_tokens_padded = topk_ids.numel() * block_size
    else:
        max_num_tokens_padded = topk_ids.numel() + (num_experts + 1) * (block_size - 1)
    sorted_ids = torch.empty(
        (max_num_tokens_padded,), dtype=torch.int32, device=topk_ids.device
    )
    max_num_m_blocks = triton.cdiv(max_num_tokens_padded, block_size)
    expert_ids = torch.empty(
        (max_num_m_blocks,), dtype=torch.int32, device=topk_ids.device
    )
    num_tokens_post_pad = torch.empty((1), dtype=torch.int32, device=topk_ids.device)

    # In EP, expert_ids for filtered experts are -1. We have num_experts + 1 ids in total.
    cumsum_buffer = torch.empty(
        (num_experts + 2,), dtype=torch.int32, device=topk_ids.device
    )

    sgl_moe_align_block_size(
        topk_ids,
        num_experts + 1,
        block_size,
        sorted_ids,
        expert_ids,
        num_tokens_post_pad,
        cumsum_buffer,
        True,
    )
    return sorted_ids, expert_ids, num_tokens_post_pad
```
**EN:** This block defines `moe_align_block_size` and contains the main logic for this step. It mainly invokes `torch.empty`, `triton.cdiv`, `sgl_moe_align_block_size`, and `topk_ids.numel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `sorted_ids`, `max_num_m_blocks`, `expert_ids`, `num_tokens_post_pad`, and `cumsum_buffer` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `moe_align_block_size`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`triton.cdiv`、`sgl_moe_align_block_size` 以及 `topk_ids.numel`，说明该流程会编排底层辅助函数或计算内核。 像 `sorted_ids`、`max_num_m_blocks`、`expert_ids`、`num_tokens_post_pad` 以及 `cumsum_buffer` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `moe_align_block_size`. / **主要符号**：核心入口包括 `moe_align_block_size`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations` and `typing.Tuple` / **标准库**：`__future__.annotations` 和 `typing.Tuple`
- **Third-party**: `torch`, `triton`, and `sgl_kernel.moe_align_block_size` / **第三方依赖**：`torch`、`triton` 以及 `sgl_kernel.moe_align_block_size`
- **Internal SGLang modules**: `sglang.srt.utils.is_cuda`, `sglang.srt.utils.is_hip`, `sglang.srt.utils.is_musa`, and `sglang.srt.utils.is_xpu` / **SGLang 内部模块**：`sglang.srt.utils.is_cuda`、`sglang.srt.utils.is_hip`、`sglang.srt.utils.is_musa` 以及 `sglang.srt.utils.is_xpu`
