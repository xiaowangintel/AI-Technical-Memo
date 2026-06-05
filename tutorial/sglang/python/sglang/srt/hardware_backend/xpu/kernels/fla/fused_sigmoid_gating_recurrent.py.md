# fused_sigmoid_gating_recurrent.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/xpu/kernels/fla/fused_sigmoid_gating_recurrent.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements XPU backend support for kernel-level operators inside the SGLang runtime. / 为 SGLang 运行时提供面向 XPU 后端的内核级算子支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module setup and shared state / 模块设置与共享状态
```python
from typing import Optional

import torch
import triton

from sglang.srt.layers.attention.fla.fused_sigmoid_gating_recurrent import (
    fused_sigmoid_gating_delta_rule_update_kernel,
)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `torch`, `triton`, `sglang.srt.layers.attention.fla.fused_sigmoid_gating_recurrent`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `torch`, `triton`, `sglang.srt.layers.attention.fla.fused_sigmoid_gating_recurrent`。

### Lines 11-128: Function `fused_sigmoid_gating_delta_rule_update` / 函数 `fused_sigmoid_gating_delta_rule_update`
```python
def fused_sigmoid_gating_delta_rule_update(
    A_log: torch.Tensor,
    a: torch.Tensor,
    dt_bias: torch.Tensor,
    softplus_beta: float,
    softplus_threshold: float,
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    b: torch.Tensor,
    initial_state_source: torch.Tensor,
    initial_state_indices: torch.Tensor,
    scale: Optional[float] = None,
    use_qk_l2norm_in_kernel: bool = False,
    cu_seqlens: Optional[torch.Tensor] = None,
    is_kda: bool = False,
    # Optional parameters for target_verify support
    disable_state_update: bool = False,
    intermediate_states_buffer: Optional[torch.Tensor] = None,
    intermediate_state_indices: Optional[torch.Tensor] = None,
    cache_steps: Optional[int] = None,
    retrieve_parent_token: Optional[torch.Tensor] = None,
):
    """
    Fused triton implementation of sigmoid gating delta rule update.
    This function uses a single fused kernel that combines both sigmoid gating computation
    and the recurrent delta rule update for better performance.

# ... omitted for brevity ...
        HAS_EAGLE_TREE_CUSTOM_ATTN_MASK=retrieve_parent_token is not None,
        num_warps=num_warps,
        num_stages=num_stages,
    )
    o = o.squeeze(0)
    return o
```
**EN:** This function implements `fused_sigmoid_gating_delta_rule_update`. It primarily calls `q.new_empty`, `triton.next_power_of_2`, `fused_sigmoid_gating_delta_rule_update_kernel`, `o.squeeze`, `q.stride`, `k.stride` to complete its work. State updates are written into `stride_q`, `stride_k`, `stride_v`, `stride_b`, `stride_a`, `HV`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `fused_sigmoid_gating_delta_rule_update`。 它主要通过调用 `q.new_empty`, `triton.next_power_of_2`, `fused_sigmoid_gating_delta_rule_update_kernel`, `o.squeeze`, `q.stride`, `k.stride` 来完成任务。 状态更新主要写入 `stride_q`, `stride_k`, `stride_v`, `stride_b`, `stride_a`, `HV`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Functions / 函数**: `fused_sigmoid_gating_delta_rule_update`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.layers.attention.fla.fused_sigmoid_gating_recurrent`
- **External / 外部依赖**: `torch`, `triton`
- **Standard library / 标准库**: `typing`
