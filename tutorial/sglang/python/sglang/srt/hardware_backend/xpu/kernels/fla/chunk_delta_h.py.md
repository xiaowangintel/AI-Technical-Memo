# chunk_delta_h.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/xpu/kernels/fla/chunk_delta_h.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements XPU backend support for kernel-level operators inside the SGLang runtime. / 为 SGLang 运行时提供面向 XPU 后端的内核级算子支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Module setup and shared state / 模块设置与共享状态
```python
from typing import Optional, Tuple

import torch
import triton
import triton.language as tl

from sglang.srt.layers.attention.fla.index import (
    prepare_chunk_indices,
    prepare_chunk_offsets,
)
from sglang.srt.layers.attention.fla.op import exp, make_tensor_descriptor, safe_exp
from sglang.srt.layers.attention.fla.utils import (
    autotune_cache_kwargs,
)

CHUNK_SIZE = 64
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `torch`, `triton`, `triton.language`, `sglang.srt.layers.attention.fla.index`, `sglang.srt.layers.attention.fla.op`. It also defines symbols such as `CHUNK_SIZE` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `torch`, `triton`, `triton.language`, `sglang.srt.layers.attention.fla.index`, `sglang.srt.layers.attention.fla.op`。 同时定义了 `CHUNK_SIZE` 等符号，供后续逻辑使用。

### Lines 20-173: Function `chunk_gated_delta_rule_fwd_kernel_h_blockdim64_k_loop` / 函数 `chunk_gated_delta_rule_fwd_kernel_h_blockdim64_k_loop`
```python
@triton.autotune(
    configs=[triton.Config({"BV": 64}, num_warps=8, num_stages=2)],
    key=["H", "K", "V", "BT", "USE_GK", "USE_INITIAL_STATE", "NT_BUCKET"],
    **autotune_cache_kwargs,
)
@triton.jit(do_not_specialize=["T"])
def chunk_gated_delta_rule_fwd_kernel_h_blockdim64_k_loop(
    k,
    v,
    w,
    v_new,
    g,
    gk,
    h,
    initial_state,
    initial_state_indices,
    cu_seqlens,
    chunk_offsets,
    T,
    H: tl.constexpr,
    Hg: tl.constexpr,
    K: tl.constexpr,
    V: tl.constexpr,
    BT: tl.constexpr,
    BV: tl.constexpr,
    USE_G: tl.constexpr,
    USE_GK: tl.constexpr,
    USE_INITIAL_STATE: tl.constexpr,
# ... omitted for brevity ...
        # epilogue
        if INPLACE_UPDATE:
            p_ht = tl.make_block_ptr(
                ht, (V, K), (K, 1), (i_v * BV, k_start), (BV, 64), (1, 0)
            )
            tl.store(p_ht, b_h1.to(p_ht.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** This function implements `chunk_gated_delta_rule_fwd_kernel_h_blockdim64_k_loop`. It primarily calls `triton.autotune`, `triton.jit`, `to`, `make_tensor_descriptor`, `tl.load.to`, `range` to complete its work. State updates are written into `h`, `v`, `k`, `w`, `stride_v`, `stride_h`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `chunk_gated_delta_rule_fwd_kernel_h_blockdim64_k_loop`。 它主要通过调用 `triton.autotune`, `triton.jit`, `to`, `make_tensor_descriptor`, `tl.load.to`, `range` 来完成任务。 状态更新主要写入 `h`, `v`, `k`, `w`, `stride_v`, `stride_h`。 实现中使用了条件分支、迭代逻辑。

### Lines 176-240: Function `chunk_gated_delta_rule_fwd_h` / 函数 `chunk_gated_delta_rule_fwd_h`
```python
def chunk_gated_delta_rule_fwd_h(
    k: torch.Tensor,
    w: torch.Tensor,
    u: torch.Tensor,
    g: Optional[torch.Tensor] = None,
    gk: Optional[torch.Tensor] = None,
    initial_state: Optional[torch.Tensor] = None,
    initial_state_indices: Optional[torch.Tensor] = None,
    save_new_value: bool = True,
    cu_seqlens: Optional[torch.LongTensor] = None,
    chunk_indices: Optional[torch.LongTensor] = None,
) -> Tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    B, T, Hg, K, V = *k.shape, u.shape[-1]
    H = u.shape[-2]
    BT = CHUNK_SIZE

    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, CHUNK_SIZE)
    # N: the actual number of sequences in the batch with either equal or variable lengths
    if cu_seqlens is None:
        N, NT, chunk_offsets = B, triton.cdiv(T, BT), None
    else:
        N, NT, chunk_offsets = (
            len(cu_seqlens) - 1,
            len(chunk_indices),
            prepare_chunk_offsets(cu_seqlens, BT),
        )
    assert K <= 256, "current kernel does not support head dimension larger than 256."
# ... omitted for brevity ...
        INPLACE_UPDATE=True,
        SAVE_NEW_VALUE=v_new is not None,
        IS_VARLEN=cu_seqlens is not None,
        NT_BUCKET=(0 if NT <= 32 else (1 if NT <= 128 else 2)),
    )
    return h, v_new
```
**EN:** This function implements `chunk_gated_delta_rule_fwd_h`. It primarily calls `k.new_empty`, `kernel`, `prepare_chunk_indices`, `torch.empty_like`, `triton.cdiv`, `len` to complete its work. State updates are written into `H`, `BT`, `h`, `v_new`, `kernel`, `chunk_indices`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `chunk_gated_delta_rule_fwd_h`。 它主要通过调用 `k.new_empty`, `kernel`, `prepare_chunk_indices`, `torch.empty_like`, `triton.cdiv`, `len` 来完成任务。 状态更新主要写入 `H`, `BT`, `h`, `v_new`, `kernel`, `chunk_indices`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Functions / 函数**: `chunk_gated_delta_rule_fwd_kernel_h_blockdim64_k_loop`, `chunk_gated_delta_rule_fwd_h`, `grid`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.layers.attention.fla.index`, `sglang.srt.layers.attention.fla.op`, `sglang.srt.layers.attention.fla.utils`
- **External / 外部依赖**: `torch`, `triton`, `triton.language`
- **Standard library / 标准库**: `typing`
