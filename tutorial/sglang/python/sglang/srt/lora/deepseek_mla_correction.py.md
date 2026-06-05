# deepseek_mla_correction.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/deepseek_mla_correction.py`
- **Repository**: sgl-project/sglang
- **Purpose**: LoRA correction for absorbed-MLA ``kv_b_proj``. / 该文件实现了核心 LoRA 运行时逻辑，例如适配器元数据、内存管理、层封装或整体编排。它属于 SGLang 支持动态适配器服务能力的基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-33: Module header, imports, and shared constants
```python
"""LoRA correction for absorbed-MLA ``kv_b_proj``.

The absorbed-MLA path in ``DeepseekV2AttentionMLA`` bypasses
``kv_b_proj.forward()`` and folds the K/V contribution into two BMMs against
the pre-computed ``w_kc`` / ``w_vc`` weights, so a standard
``ColumnParallelLinearWithLoRA`` wrapper would never see the activations and
the LoRA delta would silently be dropped. These helpers inject the missing
delta on top of the absorbed intermediates via the SGMM-style Triton kernels
in ``triton_ops/kv_b_lora_absorbed.py``.

Used from ``deepseek_common/attention_forward_methods/forward_mla.py``. Call
sites should gate the call with :func:`is_kv_b_lora_active` so non-LoRA
forwards take a single ``getattr`` and skip the helper entirely.
"""

from __future__ import annotations

from typing import TYPE_CHECKING, Optional, Tuple

import torch

from sglang.srt.lora.triton_ops import (
    step_a_q_fwd,
    step_a_v_fwd,
    step_b_q_fwd,
    step_b_v_fwd,
)

if TYPE_CHECKING:
    from sglang.srt.lora.utils import LoRABatchInfo
    from sglang.srt.models.deepseek_v2 import DeepseekV2AttentionMLA
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 34-38: is kv b lora active function
```python
def is_kv_b_lora_active(attn_module: "DeepseekV2AttentionMLA") -> bool:
    """Cheap precondition check used at call sites in the attention forward
    to skip the entire LoRA-correction path when no ``kv_b_proj`` adapter is
    wrapped on this module (the common case)."""
    return getattr(attn_module.kv_b_proj, "set_lora", False)
```
**EN:** This block uses `is_kv_b_lora_active` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `is_kv_b_lora_active` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 41-60: get state function
```python
def _get_state(
    attn_module: "DeepseekV2AttentionMLA",
) -> Optional[Tuple[torch.Tensor, torch.Tensor, "LoRABatchInfo"]]:
    if not is_kv_b_lora_active(attn_module):
        return None
    if not hasattr(attn_module.kv_b_proj, "A_buffer"):
        return None
    lora_backend = attn_module.kv_b_proj.lora_backend
    if not hasattr(lora_backend, "batch_info"):
        return None
    batch_info = lora_backend.batch_info
    if batch_info is None:
        return None

    # Triton backend exposes _sgemm_info() to group decode-shape repeats of
    # the same adapter; csgmv-style backends just expose batch_info directly.
    sgemm_info = getattr(lora_backend, "_sgemm_info", None)
    if callable(sgemm_info):
        batch_info = sgemm_info()
    return attn_module.kv_b_proj.A_buffer, attn_module.kv_b_proj.B_buffer, batch_info
```
**EN:** This block uses `_get_state` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_get_state` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 63-85: apply q correction function
```python
def apply_q_correction(
    attn_module: "DeepseekV2AttentionMLA",
    q_nope: torch.Tensor,
    q_nope_out: torch.Tensor,
) -> torch.Tensor:
    """LoRA correction for the absorbed ``q_nope @ w_kc`` path.

    Computes ``q_nope_out += q_nope @ B_kc @ A * scaling`` per token, per
    active LoRA slot via two SGMM-style Triton kernels. Factored along the
    LoRA-A/B boundary so we never materialise ``B @ A`` (~268M FMAs per layer
    per slot in the naive implementation)::

      step A_q : ``(S,H,qk_nope) @ B_kc[slot, h] (qk_nope, rank) -> (S,H,rank)``
      step B_q : ``(S,H,rank)    @ A[slot] (rank, kv_lora_rank)  -> += q_nope_out``
    """
    state = _get_state(attn_module)
    if state is None:
        return q_nope_out
    A_buf, B_buf, batch_info = state

    full_K_per_head = attn_module.qk_nope_head_dim + attn_module.v_head_dim
    q_lora_a = step_a_q_fwd(q_nope, B_buf, batch_info, full_K_per_head)
    return step_b_q_fwd(q_lora_a, A_buf, batch_info, q_nope_out)
```
**EN:** This block uses `apply_q_correction` to apply a transformation to tensors or state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `apply_q_correction` 来对张量或状态应用变换。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 88-117: apply v correction function
```python
def apply_v_correction(
    attn_module: "DeepseekV2AttentionMLA",
    attn_output: torch.Tensor,
    attn_bmm_flat: torch.Tensor,
) -> torch.Tensor:
    """LoRA correction for the absorbed ``attn_output @ w_vc`` path.

    Computes ``attn_bmm_flat += attn_output @ A.T @ B_vc.T * scaling`` per
    token, per active LoRA slot. ``attn_bmm_flat`` is the flat
    ``(S, H*v_head_dim)`` view of the absorbed BMM result; we pass strides
    matching the implicit ``(S, H, v_head_dim)`` layout to step B_v.
    """
    state = _get_state(attn_module)
    if state is None:
        return attn_bmm_flat
    A_buf, B_buf, batch_info = state

    attn_lora_a = step_a_v_fwd(attn_output, A_buf, batch_info)
    base_view = attn_bmm_flat.view(
        -1, attn_module.num_local_heads, attn_module.v_head_dim
    )
    step_b_v_fwd(
        attn_lora_a,
        B_buf,
        batch_info,
        base_view,
        attn_module.qk_nope_head_dim,
        attn_module.v_head_dim,
    )
    return attn_bmm_flat
```
**EN:** This block uses `apply_v_correction` to apply a transformation to tensors or state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `apply_v_correction` 来对张量或状态应用变换。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.lora.triton_ops`
- `sglang.srt.lora.utils`
- `sglang.srt.models.deepseek_v2`
### External / 外部
- `__future__`
- `torch`
- `typing` (stdlib)
