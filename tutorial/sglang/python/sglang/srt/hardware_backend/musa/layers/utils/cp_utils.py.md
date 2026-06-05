# cp_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/musa/layers/utils/cp_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MUSA backend support for shared runtime utilities inside the SGLang runtime. / 为 SGLang 运行时提供面向 MUSA 后端的共享运行时工具支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Module setup and shared state / 模块设置与共享状态
```python
from typing import TYPE_CHECKING, Callable

import torch

if TYPE_CHECKING:
    from sglang.srt.hardware_backend.musa.attention.flashattention_backend import (
        MusaFlashAttentionBackend,
    )
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `torch`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `torch`。

### Lines 12-57: Function `musa_cp_attn_forward_extend` / 函数 `musa_cp_attn_forward_extend`
```python
def musa_cp_attn_forward_extend(
    musa_fa_backend: "MusaFlashAttentionBackend",
    forward_batch: "ForwardBatch",
    q: torch.Tensor,
    device: torch.device,
    attn_fn: Callable[[torch.Tensor, torch.Tensor, torch.Tensor, int], torch.Tensor],
) -> torch.Tensor:
    """
    Split q into prev/next zigzag halves based on CP metadata, call the
    backend-specific attention function twice with appropriate per-half
    metadata, and concatenate the results.

    attn_fn signature:
        attn_fn(q, cu_seqlens_q, cache_seqlens, max_seqlen_q) -> result
    where only these four CP-varying parameters differ between halves.
    All other backend-specific args should be captured in the closure.
    """
    cp_meta = forward_batch.attn_cp_metadata

    q_prev, q_next = torch.chunk(q, 2, dim=0)

    cu_seqlens_q_prev = torch.tensor(
        [0, cp_meta.actual_seq_q_prev], device=device, dtype=torch.int32
    )
    if hasattr(musa_fa_backend, "_current_prefix"):
        musa_fa_backend._current_prefix = "forward_extend_cp_prev"
    result_prev = attn_fn(
        q_prev,
# ... omitted for brevity ...
        cu_seqlens_q_next,
        cp_meta.kv_len_next_tensor,
        cp_meta.actual_seq_q_next,
    )

    return torch.concat([result_prev, result_next], dim=0)
```
**EN:** This function implements `musa_cp_attn_forward_extend`. It primarily calls `torch.chunk`, `torch.tensor`, `hasattr`, `attn_fn`, `torch.concat` to complete its work. State updates are written into `cp_meta`, `cu_seqlens_q_prev`, `result_prev`, `cu_seqlens_q_next`, `result_next`, `musa_fa_backend._current_prefix`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `musa_cp_attn_forward_extend`。 它主要通过调用 `torch.chunk`, `torch.tensor`, `hasattr`, `attn_fn`, `torch.concat` 来完成任务。 状态更新主要写入 `cp_meta`, `cu_seqlens_q_prev`, `result_prev`, `cu_seqlens_q_next`, `result_next`, `musa_fa_backend._current_prefix`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Functions / 函数**: `musa_cp_attn_forward_extend`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.musa.attention.flashattention_backend`, `sglang.srt.model_executor.forward_batch_info`
- **External / 外部依赖**: `torch`
- **Standard library / 标准库**: `typing`
