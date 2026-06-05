# chunk_o.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/chunk_o.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects chunk o helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 chunk o 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 5-13: imports
```python
from typing import Optional

import torch
import triton
import triton.language as tl

from sglang.srt.layers.attention.fla.index import prepare_chunk_indices
from sglang.srt.layers.attention.fla.op import exp, safe_exp
from sglang.srt.layers.attention.fla.utils import check_shared_mem, is_nvidia_hopper
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 15-16: module constants
```python
BKV_LIST = [64, 128] if check_shared_mem() else [32, 64]
NUM_WARPS = [2, 4] if is_nvidia_hopper else [2, 4, 8]
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 29-123: function chunk_fwd_kernel_o
```python
@triton.jit(do_not_specialize=["T"])
def chunk_fwd_kernel_o(
    q,
    k,
    v,
    h,
    g,
    o,
    cu_seqlens,
    chunk_indices,
    scale,
    T,
    H: tl.constexpr,
    Hg: tl.constexpr,
    K: tl.constexpr,
    V: tl.constexpr,
    BT: tl.constexpr,
    BK: tl.constexpr,
    BV: tl.constexpr,
    USE_G: tl.constexpr,
    IS_VARLEN: tl.constexpr,
):
    i_v, i_t, i_bh = tl.program_id(0), tl.program_id(1), tl.program_id(2)
    i_b, i_h = i_bh // H, i_bh % H

    if IS_VARLEN:
        i_tg = i_t
        i_n, i_t = tl.load(chunk_indices + i_t * 2).to(tl.int32), tl.load(
            chunk_indices + i_t * 2 + 1
        ).to(tl.int32)
        bos, eos = tl.load(cu_seqlens + i_n).to(tl.int32), tl.load(
            cu_seqlens + i_n + 1
# ... omitted 51 lines ...
    p_v = tl.make_block_ptr(
        v, (T, V), (H * V, 1), (i_t * BT, i_v * BV), (BT, BV), (1, 0)
    )
    p_o = tl.make_block_ptr(
        o, (T, V), (H * V, 1), (i_t * BT, i_v * BV), (BT, BV), (1, 0)
    )
    b_v = tl.load(p_v, boundary_check=(0, 1))

    # to fix mma -> mma layout conversion
    # already solved by triton v3.2 or higher
    b_o = b_o * scale + tl.dot(b_A.to(b_v.dtype), b_v) * scale
    tl.store(p_o, b_o.to(p_o.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** Implements the chunk fwd kernel o routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk fwd kernel o 例程。

### Lines 126-174: function chunk_fwd_o
```python
def chunk_fwd_o(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    h: torch.Tensor,
    g: Optional[torch.Tensor] = None,  # cumsum of log decay
    scale: Optional[float] = None,
    cu_seqlens: Optional[torch.LongTensor] = None,
    chunk_size: int = 64,
) -> torch.Tensor:
    B, T, Hg, K, V = *q.shape, v.shape[-1]
    H = v.shape[-2]
    BT = min(chunk_size, max(16, triton.next_power_of_2(T)))
    chunk_indices = (
        prepare_chunk_indices(cu_seqlens, BT) if cu_seqlens is not None else None
    )
    NT = triton.cdiv(T, BT) if cu_seqlens is None else len(chunk_indices)
    if scale is None:
        scale = k.shape[-1] ** -0.5

    o = torch.zeros_like(v)

    def grid(meta):
        return (triton.cdiv(V, meta["BV"]), NT, B * H)

    chunk_fwd_kernel_o[grid](
        q,
        k,
        v,
        h,
        g,
        o,
# ... omitted 5 lines ...
        Hg=Hg,
        K=K,
        V=V,
        BT=BT,
        BK=128,
        BV=64,
        USE_G=g is not None,
        IS_VARLEN=cu_seqlens is not None,
        num_warps=4,
        num_stages=2,
    )
    return o
```
**EN:** Implements the chunk fwd o routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk fwd o 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `typing.Optional`
- `torch`
- `triton`
- `triton.language`
- `sglang.srt.layers.attention.fla.index.prepare_chunk_indices`
- `sglang.srt.layers.attention.fla.op.exp`
- `sglang.srt.layers.attention.fla.op.safe_exp`
- `sglang.srt.layers.attention.fla.utils.check_shared_mem`
- `sglang.srt.layers.attention.fla.utils.is_nvidia_hopper`
