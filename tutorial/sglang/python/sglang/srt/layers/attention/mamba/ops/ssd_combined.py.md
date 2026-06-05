# ssd_combined.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/mamba/ops/ssd_combined.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects ssd combined helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 ssd combined 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 11-19: imports
```python
import torch
import triton
from einops import rearrange
from packaging import version

from .ssd_bmm import _bmm_chunk_fwd
from .ssd_chunk_scan import _chunk_scan_fwd
from .ssd_chunk_state import _chunk_cumsum_fwd, _chunk_state_fwd, chunk_state_varlen
from .ssd_state_passing import _state_passing_fwd
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 21-21: module constants
```python
TRITON_22 = version.parse(triton.__version__) >= version.parse("2.2.0")
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 24-25: function is_int_pow_2
```python
def is_int_pow_2(n):
    return isinstance(n, int) and n > 0 and (n & (n - 1)) == 0
```
**EN:** Checks whether is int pow 2 is valid for the current configuration or execution state.
**CN:** 检查在当前配置或执行状态下 is int pow 2 是否成立。

### Lines 28-178: function _mamba_chunk_scan_combined_fwd
```python
def _mamba_chunk_scan_combined_fwd(
    x,
    dt,
    A,
    B,
    C,
    chunk_size,
    D=None,
    z=None,
    dt_bias=None,
    initial_states=None,
    seq_idx=None,
    chunk_indices=None,
    chunk_offsets=None,
    cu_seqlens=None,
    dt_softplus=False,
    dt_limit=(0.0, float("inf")),
    state_dtype=None,
    out=None,
):
    assert is_int_pow_2(chunk_size), "chunk_size must be integer power of 2"
    batch, seqlen, nheads, headdim = x.shape
    _, _, ngroups, dstate = B.shape
    assert nheads % ngroups == 0
    assert B.shape == (batch, seqlen, ngroups, dstate)
    assert dt.shape == (batch, seqlen, nheads)
    assert A.shape == (nheads,)
    assert C.shape == B.shape
    if z is not None:
        assert z.shape == x.shape
    if D is not None:
        assert D.shape == (nheads, headdim) or D.shape == (nheads,)
# ... omitted 107 lines ...
            batch == 1
        ), "passing cu_seqlens to get the varlen states is only supported if batch dimension is 1"
        varlen_states = chunk_state_varlen(
            B.squeeze(0),
            x.squeeze(0),
            dt.squeeze(0),
            dA_cumsum.squeeze(0),
            cu_seqlens,
            states.squeeze(0),
            initial_states=initial_states,
        )
        return out_x, dt, dA_cumsum, states, final_states, varlen_states
```
**EN:** Implements the mamba chunk scan combined fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 mamba chunk scan combined fwd 例程。

### Lines 181-275: function mamba_chunk_scan_combined
```python
def mamba_chunk_scan_combined(
    x,
    dt,
    A,
    B,
    C,
    chunk_size,
    D=None,
    z=None,
    dt_bias=None,
    initial_states=None,
    seq_idx=None,
    chunk_indices=None,
    chunk_offsets=None,
    cu_seqlens=None,
    dt_softplus=False,
    dt_limit=(0.0, float("inf")),
    out=None,
    return_final_states=False,
    return_varlen_states=False,
    return_intermediate_states=False,
    state_dtype=None,
):
    """
    Argument:
        x: (batch, seqlen, nheads, headdim)
        dt: (batch, seqlen, nheads)
        A: (nheads)
        B: (batch, seqlen, ngroups, dstate)
        C: (batch, seqlen, ngroups, dstate)
        chunk_size: int
        D: (nheads, headdim) or (nheads,)
# ... omitted 51 lines ...
    if not return_varlen_states:
        if not return_final_states:
            return
        else:
            return final_states
    else:
        varlen_states = rest[0]
        return (
            (varlen_states)
            if not return_final_states
            else (final_states, varlen_states)
        )
```
**EN:** Implements the mamba chunk scan combined routine used by this attention module.
**CN:** 实现该注意力模块使用的 mamba chunk scan combined 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** Speculative decoding support / **CN:** 投机解码支持
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `torch`
- `triton`
- `einops.rearrange`
- `packaging.version`
- `.ssd_bmm._bmm_chunk_fwd`
- `.ssd_chunk_scan._chunk_scan_fwd`
- `.ssd_chunk_state._chunk_cumsum_fwd`
- `.ssd_chunk_state._chunk_state_fwd`
- `.ssd_chunk_state.chunk_state_varlen`
- `.ssd_state_passing._state_passing_fwd`
