# seg_la.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/linear/seg_la.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the main seg la classes used by the SGLang attention stack, together with helper routines that support runtime execution. / 该模块定义 SGLang 注意力栈使用的主要 seg la 类，并提供支撑运行时执行的辅助例程。
## Line-by-Line Analysis / 逐行分析
### Lines 2-4: docstring
```python
"""
Copyright (c) Ant Financial Service Group and its affiliates.
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 8-13: imports
```python
from dataclasses import dataclass
from typing import Optional

import torch
import triton
import triton.language as tl
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 17-29: class SegLaMeta
```python
@dataclass
class SegLaMeta:
    batch_size: int  # batch size, num of requests
    max_q_length: int  # max(seq_lens)
    q_offsets: torch.Tensor  # [bs+1], query_start_locations,
    s_offsets: torch.Tensor  # [bs], slot_ids
    q_lengths: torch.Tensor  # [bs], query length
    s_scales: torch.Tensor  # [bs], prefill = 0, decode = 1
    s_offsets_stride: int = 0
    q_offsets_stride: int = 0
    s_scales_stride: int = 0
    decay_scales_stride: int = 0
    mask: Optional[torch.Tensor] = None  # Currently not supported
```
**EN:** Dataclass-style container that stores structured runtime state for seg la meta.
**CN:** 该数据类风格的容器用于存储 seg la meta 的结构化运行时状态。

### Lines 33-202: function seg_la_kernel
```python
@triton.jit
def seg_la_kernel(
    Q,
    K,
    V,
    S,
    Out,
    softmax_scale,
    stride_q,
    stride_k,
    stride_v,
    stride_s,
    stride_o,
    s_offsets,
    q_offsets,
    q_lengths,
    s_scales,
    decay_scales,
    HEAD_DIM: tl.constexpr,
    SPLIT_DIM: tl.constexpr,
    BLOCK: tl.constexpr,
    EVEN: tl.constexpr,
    DECOUPLE: tl.constexpr,
):
    bid = tl.program_id(0)
    hid = tl.program_id(1)
    sid = tl.program_id(2)

    # s_scale is 0 (prefill) or 1 (decode)
    s_scale = tl.load(s_scales + bid)
    q_length = tl.load(q_lengths + bid)
    q_offset = tl.load(q_offsets + bid)
# ... omitted 126 lines ...

    else:
        q = tl.trans(tl.load(q_ptrs)).to(tl.float32) * softmax_scale
        k = tl.trans(tl.load(k_ptrs)).to(tl.float32)
        v = tl.load(v_ptrs).to(tl.float32)
        state = state * tl.exp(decay_scale) + k * v

        o = tl.sum(q * state, axis=0, keep_dims=True)

        tl.store(out_ptrs, o.to(Out.dtype.element_ty))

        tl.store(s_ptrs, state.to(S.dtype.element_ty))
```
**EN:** Implements the seg la kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 seg la kernel 例程。

### Lines 206-343: function seg_la_p_kernel
```python
@triton.jit
def seg_la_p_kernel(
    Q,
    K,
    V,
    S,
    Out,
    softmax_scale,
    stride_q,
    stride_k,
    stride_v,
    stride_s,
    stride_o,
    s_offsets,
    q_offsets,
    q_lengths,
    s_scales,
    decay_scales,
    HEAD_DIM: tl.constexpr,
    K_SPLIT_DIM: tl.constexpr,
    V_SPLIT_DIM: tl.constexpr,
    BLOCK: tl.constexpr,
    EVEN: tl.constexpr,
):
    bid = tl.program_id(0)
    hid = tl.program_id(1)
    kvid = tl.program_id(2)
    N = HEAD_DIM // V_SPLIT_DIM
    kid = kvid // N
    vid = kvid % N
    H = tl.num_programs(1)

# ... omitted 94 lines ...
        state = state * block_decay + tl.dot(k, v)

        if EVEN:
            tl.store(out_ptrs + n * H * HEAD_DIM, o.to(Out.dtype.element_ty))
        else:
            tl.store(
                out_ptrs + n * H * HEAD_DIM,
                o.to(Out.dtype.element_ty),
                mask=(n + offs_b)[:, None] < q_length,
            )

    tl.store(s_ptrs, state.to(S.dtype.element_ty))
```
**EN:** Implements the seg la p kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 seg la p kernel 例程。

### Lines 347-478: function seg_la_s_kernel
```python
@triton.jit
def seg_la_s_kernel(
    Q,
    K,
    V,
    S,
    Out,
    Mask,
    softmax_scale,
    stride_q,
    stride_k,
    stride_v,
    stride_s,
    stride_o,
    s_offsets,
    q_offsets,
    q_lengths,
    s_scales,
    decay_scales,
    HEAD_DIM: tl.constexpr,
    K_SPLIT_DIM: tl.constexpr,
    V_SPLIT_DIM: tl.constexpr,
    BLOCK: tl.constexpr,
    EVEN: tl.constexpr,
):
    bid = tl.program_id(0)
    hid = tl.program_id(1)
    kvid = tl.program_id(2)
    N = HEAD_DIM // V_SPLIT_DIM
    kid = kvid // N
    vid = kvid % N
    H = tl.num_programs(1)
# ... omitted 88 lines ...
    k = k * decays[None, :]
    qk = tl.dot(q, k) * softmax_scale
    qk = qk * mask.to(tl.float32)
    o = tl.dot(qk, v)

    block_decay = tl.exp(decay_scale * (max_pos + 1))
    o = tl.dot(q, state) * block_decay * softmax_scale + o

    if EVEN:
        tl.store(out_ptrs, o.to(Out.dtype.element_ty))
    else:
        tl.store(out_ptrs, o.to(Out.dtype.element_ty), mask=offs_b[:, None] < q_length)
```
**EN:** Implements the seg la s kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 seg la s kernel 例程。

### Lines 482-549: function seg_la_d_kernel
```python
@triton.jit
def seg_la_d_kernel(
    Q,
    K,
    V,
    S,
    Out,
    softmax_scale,
    stride_q,
    stride_k,
    stride_v,
    stride_s,
    stride_o,
    s_offsets,
    decay_scales,
    HEAD_DIM: tl.constexpr,
    K_SPLIT_DIM: tl.constexpr,
    V_SPLIT_DIM: tl.constexpr,
):
    bid = tl.program_id(0)
    hid = tl.program_id(1)
    kvid = tl.program_id(2)
    N = HEAD_DIM // V_SPLIT_DIM
    kid = kvid // N
    vid = kvid % N
    H = tl.num_programs(1)

    # s_scale is 0 (first prefill chunk) or 1 (next prefill chunk)
    s_offset = tl.load(s_offsets + bid)
    if s_offset == -1:
        return

# ... omitted 24 lines ...
    )
    state = tl.load(s_ptrs).to(tl.float32)

    k = tl.load(k_ptrs).to(tl.float32)
    v = tl.load(v_ptrs).to(tl.float32)
    q = tl.load(q_ptrs).to(tl.float32) * softmax_scale

    state = state * tl.exp(decay_scale) + k[:, None] * v
    o = tl.sum(q[:, None] * state, axis=0)

    tl.store(out_ptrs, o.to(Out.dtype.element_ty))
    tl.store(s_ptrs, state.to(S.dtype.element_ty))
```
**EN:** Implements the seg la d kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 seg la d kernel 例程。

### Lines 553-641: function seg_la_mtp_kernel
```python
@triton.jit
def seg_la_mtp_kernel(
    Q,
    K,
    V,
    S,
    CACHES,
    Out,
    softmax_scale,
    stride_q,
    stride_k,
    stride_v,
    stride_s,
    stride_c,
    stride_o,
    s_offsets,
    cache_indices,
    decay_scales,
    step,
    HEAD_DIM: tl.constexpr,
    K_SPLIT_DIM: tl.constexpr,
    V_SPLIT_DIM: tl.constexpr,
):
    bid = tl.program_id(0)
    hid = tl.program_id(1)
    kvid = tl.program_id(2)
    N = HEAD_DIM // V_SPLIT_DIM
    kid = kvid // N
    vid = kvid % N
    H = tl.num_programs(1)

    s_offset = tl.load(s_offsets + bid)
# ... omitted 45 lines ...
        v = tl.load(v_ptrs).to(tl.float32)

        state = state * decay_scale + k[:, None] * v
        o = tl.sum(q[:, None] * state, axis=0)

        tl.store(out_ptrs, o.to(Out.dtype.element_ty))
        tl.store(c_ptrs, state.to(CACHES.dtype.element_ty))
        q_ptrs += stride_q
        k_ptrs += stride_k
        v_ptrs += stride_v
        out_ptrs += H * HEAD_DIM
        c_ptrs += H * HEAD_DIM * HEAD_DIM
```
**EN:** Implements the seg la mtp kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 seg la mtp kernel 例程。

### Lines 645-654: function seg_la_sum_kernel
```python
@triton.jit
def seg_la_sum_kernel(T, O, DIM: tl.constexpr, NUM_BLOCK: tl.constexpr):
    pid = tl.program_id(0)
    length = tl.num_programs(0)
    x = tl.zeros((DIM,), dtype=tl.float32)
    for i in range(NUM_BLOCK):
        x += tl.load(T + i * length * DIM + pid * DIM + tl.arange(0, DIM)).to(
            tl.float32
        )
    tl.store(O + pid * DIM + tl.arange(0, DIM), x)
```
**EN:** Implements the seg la sum kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 seg la sum kernel 例程。

### Lines 657-910: function seg_la_fwd
```python
def seg_la_fwd(
    q,
    k,
    v,
    s,
    decay_scales,
    meta,
    caches=None,
    cache_indices=None,
    softmax_scale=None,
    decouple=False,
):
    length, qo_heads, HEAD_DIM = q.shape
    _, kv_heads, _ = k.shape
    bs = meta.batch_size
    if softmax_scale is None:
        softmax_scale = HEAD_DIM ** (-0.5)

    # MAX_LENGTH = meta.max_q_length
    MAX_LENGTH = triton.cdiv(length, bs)

    assert qo_heads == kv_heads, "seg_la does NOT support GQA currently"

    if MAX_LENGTH > 1:
        # prefill with partitioning q/k/v
        # BLOCK should <= 64 with decouple
        K_SPLIT_DIM = 32
        V_SPLIT_DIM = 32 if bs <= 2 else 64

        num_warps = 2  # 2
        num_stages = 3  # 3

# ... omitted 210 lines ...
    #         meta.q_lengths,
    #         meta.s_scales,
    #         decay_scales,
    #         HEAD_DIM=HEAD_DIM,
    #         SPLIT_DIM=SPLIT_DIM,
    #         BLOCK=BLOCK,
    #         EVEN=EVEN,
    #         DECOUPLE=decouple,
    #         num_warps=num_warps,
    #         num_stages=num_stages
    #     )
    return o
```
**EN:** Implements the seg la fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 seg la fwd 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Speculative decoding support / **CN:** 投机解码支持
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `dataclasses.dataclass`
- `typing.Optional`
- `torch`
- `triton`
- `triton.language`
