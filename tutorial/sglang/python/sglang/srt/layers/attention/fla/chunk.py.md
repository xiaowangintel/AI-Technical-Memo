# chunk.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/chunk.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the main chunk classes used by the SGLang attention stack, together with helper routines that support runtime execution. / 该模块定义 SGLang 注意力栈使用的主要 chunk 类，并提供支撑运行时执行的辅助例程。
## Line-by-Line Analysis / 逐行分析
### Lines 5-23: imports
```python
from typing import Optional

import torch
from einops import rearrange

from sglang.srt.layers.attention.fla.chunk_delta_h import chunk_gated_delta_rule_fwd_h
from sglang.srt.layers.attention.fla.chunk_fwd import chunk_gated_delta_rule_fwd_intra
from sglang.srt.layers.attention.fla.chunk_o import chunk_fwd_o
from sglang.srt.layers.attention.fla.cumsum import chunk_local_cumsum
from sglang.srt.layers.attention.fla.index import (
    prepare_chunk_indices,
)
from sglang.srt.layers.attention.fla.l2norm import l2norm_fwd
from sglang.srt.layers.attention.fla.utils import (
    SUPPRESS_LEVEL,
    autocast_custom_fwd,
    input_guard,
    is_intel,
)
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 25-31: conditional branch
```python
if is_intel:
    from sglang.srt.hardware_backend.xpu.kernels.fla.chunk_delta_h import (
        chunk_gated_delta_rule_fwd_h,
    )
    from sglang.srt.hardware_backend.xpu.kernels.fla.chunk_fwd import (
        chunk_gated_delta_rule_fwd_intra,
    )
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 33-33: module constants
```python
CHUNK_SIZE = 64
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 36-84: function chunk_gated_delta_rule_fwd
```python
def chunk_gated_delta_rule_fwd(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    scale: float,
    initial_state: torch.Tensor,
    initial_state_indices: torch.Tensor,
    cu_seqlens: Optional[torch.LongTensor] = None,
    chunk_indices: torch.LongTensor | None = None,
):
    g = chunk_local_cumsum(
        g, chunk_size=CHUNK_SIZE, cu_seqlens=cu_seqlens, chunk_indices=chunk_indices
    )

    # fused kkt + solve_tril + recompute_w_u
    w, u, A = chunk_gated_delta_rule_fwd_intra(
        k=k,
        v=v,
        g=g,
        beta=beta,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
    )

    h, v_new = chunk_gated_delta_rule_fwd_h(
        k=k,
        w=w,
        u=u,
        g=g,
        initial_state=initial_state,
# ... omitted 5 lines ...
        q=q,
        k=k,
        v=v_new,
        h=h,
        g=g,
        scale=scale,
        cu_seqlens=cu_seqlens,
    )
    if SUPPRESS_LEVEL < 3:
        return g, o, A, None, h, None
    elif SUPPRESS_LEVEL >= 3:
        return g, o, A, w, h, v_new
```
**EN:** Implements the chunk gated delta rule fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk gated delta rule fwd 例程。

### Lines 87-88: class ChunkGatedDeltaRuleFunction
```python
class ChunkGatedDeltaRuleFunction(torch.autograd.Function):
```
**EN:** Defines the chunk gated delta rule function type and the state it exposes to the rest of the attention stack.
**CN:** 定义 chunk gated delta rule function 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 89-129: method ChunkGatedDeltaRuleFunction.forward
```python
    @staticmethod
    @input_guard
    @autocast_custom_fwd
    def forward(
        ctx,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        g: torch.Tensor,
        beta: torch.Tensor,
        scale: float,
        initial_state: torch.Tensor,
        initial_state_indices: torch.Tensor,
        cu_seqlens: Optional[torch.LongTensor] = None,
        use_qk_l2norm_in_kernel: bool = False,
    ):
        q_orig = q
        k_orig = k

        if use_qk_l2norm_in_kernel:
            q = l2norm_fwd(q)
            k = l2norm_fwd(k)

        chunk_indices = (
            prepare_chunk_indices(cu_seqlens, CHUNK_SIZE)
            if cu_seqlens is not None
            else None
        )
        g, o, A, w, h, v_new = chunk_gated_delta_rule_fwd(
            q=q,
            k=k,
            v=v,
            g=g,
            beta=beta,
            scale=scale,
            initial_state=initial_state,
            initial_state_indices=initial_state_indices,
            cu_seqlens=cu_seqlens,
            chunk_indices=chunk_indices,
        )
        return o.to(q.dtype), h
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 132-262: function chunk_gated_delta_rule
```python
@torch.compiler.disable
def chunk_gated_delta_rule(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    scale: float = None,
    initial_state: torch.Tensor = None,
    initial_state_indices: torch.Tensor = None,
    cu_seqlens: Optional[torch.LongTensor] = None,
    head_first: bool = False,
    use_qk_l2norm_in_kernel: bool = False,
):
    r"""
    Args:
        q (torch.Tensor):
            queries of shape `[B, T, H, K]` if `head_first=False` else `[B, H, T, K]`.
        k (torch.Tensor):
            keys of shape `[B, T, H, K]` if `head_first=False` else `[B, H, T, K]`.
        v (torch.Tensor):
            values of shape `[B, T, H, V]` if `head_first=False` else `[B, H, T, V]`.
        g (torch.Tensor):
            (forget) gating tensor (in log space!) of shape `[B, T, H]` if `head_first=False` else `[B, H, T]`.
        beta (torch.Tensor):
            betas of shape `[B, T, H]` if `head_first=False` else `[B, H, T]`.
        scale (Optional[int]):
            Scale factor for the RetNet attention scores.
            If not provided, it will default to `1 / sqrt(K)`. Default: `None`.
        initial_state (Optional[torch.Tensor]):
            Initial state of shape `[N, H, V, K]` for `N` input sequences.
            For equal-length input sequences, `N` equals the batch size `B`.
# ... omitted 87 lines ...
        v,
        g,
        beta,
        scale,
        initial_state,
        initial_state_indices,
        cu_seqlens,
        use_qk_l2norm_in_kernel,
    )
    if head_first:
        o = rearrange(o, "b t h ... -> b h t ...")
    return o, None, h
```
**EN:** Implements the chunk gated delta rule routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk gated delta rule 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Speculative decoding support / **CN:** 投机解码支持
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `typing.Optional`
- `torch`
- `einops.rearrange`
- `sglang.srt.layers.attention.fla.chunk_delta_h.chunk_gated_delta_rule_fwd_h`
- `sglang.srt.layers.attention.fla.chunk_fwd.chunk_gated_delta_rule_fwd_intra`
- `sglang.srt.layers.attention.fla.chunk_o.chunk_fwd_o`
- `sglang.srt.layers.attention.fla.cumsum.chunk_local_cumsum`
- `sglang.srt.layers.attention.fla.index.prepare_chunk_indices`
- `sglang.srt.layers.attention.fla.l2norm.l2norm_fwd`
- `sglang.srt.layers.attention.fla.utils.SUPPRESS_LEVEL`
- `sglang.srt.layers.attention.fla.utils.autocast_custom_fwd`
- `sglang.srt.layers.attention.fla.utils.input_guard`
- `sglang.srt.layers.attention.fla.utils.is_intel`
- `sglang.srt.hardware_backend.xpu.kernels.fla.chunk_delta_h.chunk_gated_delta_rule_fwd_h`
- `sglang.srt.hardware_backend.xpu.kernels.fla.chunk_fwd.chunk_gated_delta_rule_fwd_intra`
