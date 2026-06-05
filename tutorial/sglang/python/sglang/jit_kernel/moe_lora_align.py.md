# moe_lora_align.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/moe_lora_align.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Optional

import torch

from sglang.jit_kernel.utils import cache_once, load_jit, make_cpp_args

if TYPE_CHECKING:
    from tvm_ffi.module import Module


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 14-25: Function `_jit_moe_align_module`
```python
def _jit_moe_align_module(dtype: torch.dtype) -> Module:
    args = make_cpp_args(dtype)
    return load_jit(
        "moe_lora_align_block_size",
        *args,
        cuda_files=["lora/moe_lora_align_kernel.cu"],
        cuda_wrappers=[
            ("moe_lora_align_block_size", f"MoeLoraAlignBlockSizeKernel<{args}>::run"),
        ],
    )
```
**EN:** This block defines `_jit_moe_align_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_moe_align_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 26-74: Function `moe_lora_align_block_size`
```python
def moe_lora_align_block_size(
    topk_ids: torch.Tensor,
    seg_indptr: torch.Tensor,
    req_to_lora: torch.Tensor,
    num_experts: int,
    block_size: int,
    max_loras: int,
    max_num_tokens_padded: int,
    max_num_m_blocks: int,
    sorted_token_ids: torch.Tensor,
    expert_ids: torch.Tensor,
    num_tokens_post_pad: torch.Tensor,
    adapter_enabled: torch.Tensor,
    lora_ids: torch.Tensor,
    maybe_expert_map: Optional[torch.Tensor] = None,
    cumsum_buffer: Optional[torch.Tensor] = None,
    token_mask: Optional[torch.Tensor] = None,
) -> None:
    module = _jit_moe_align_module(topk_ids.dtype)

    if cumsum_buffer is None:
        cumsum_buffer = torch.zeros(
            max_loras * (num_experts + 1), dtype=torch.int32, device=topk_ids.device
        )
    else:
        cumsum_buffer.zero_()
    if token_mask is None:
        token_mask = torch.empty(
            (max_loras * topk_ids.shape[0],), dtype=torch.int32, device=topk_ids.device
        )

    module.moe_lora_align_block_size(
        topk_ids,
        seg_indptr,
        req_to_lora,
        num_experts,
        block_size,
        max_loras,
        max_num_tokens_padded,
        max_num_m_blocks,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_pad,
        adapter_enabled,
        lora_ids,
        maybe_expert_map,
        cumsum_buffer,
        token_mask,
    )
```
**EN:** This block defines `moe_lora_align_block_size`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `moe_lora_align_block_size`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> cache_once`
- `tvm_ffi.module -> Module`
