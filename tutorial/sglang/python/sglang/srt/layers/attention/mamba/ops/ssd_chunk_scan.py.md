# ssd_chunk_scan.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/mamba/ops/ssd_chunk_scan.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects ssd chunk scan helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 ssd chunk scan 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 11-14: imports
```python
import torch
import triton
import triton.language as tl
from packaging import version
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 16-16: module constants
```python
TRITON_22 = version.parse(triton.__version__) >= version.parse("2.2.0")
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 19-419: function _chunk_scan_fwd_kernel
```python
@triton.jit
def _chunk_scan_fwd_kernel(
    # Pointers to matrices
    cb_ptr,
    x_ptr,
    z_ptr,
    out_ptr,
    out_x_ptr,
    dt_ptr,
    dA_cumsum_ptr,
    seq_idx_ptr,
    C_ptr,
    states_ptr,
    D_ptr,
    initstates_ptr,
    chunk_indices_ptr,
    chunk_offsets_ptr,
    chunk_meta_num,
    # Matrix dimensions
    chunk_size,
    hdim,
    dstate,
    batch,
    seqlen,
    nheads_ngroups_ratio,
    # Strides
    stride_cb_batch,
    stride_cb_chunk,
    stride_cb_head,
    stride_cb_csize_m,
    stride_cb_csize_k,
    stride_x_batch,
# ... omitted 357 lines ...
        pid_b * stride_out_batch
        + c_idx * chunk_size * stride_out_seqlen
        + pid_h * stride_out_head
    )
    out_ptrs = out_ptr + (
        stride_out_seqlen * offs_out_m[:, None] + offs_out_n[None, :] * stride_out_hdim
    )
    tl.store(
        out_ptrs,
        acc,
        mask=(offs_out_m[:, None] < chunk_size_limit) & (offs_out_n[None, :] < hdim),
    )
```
**EN:** Implements the chunk scan fwd kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk scan fwd kernel 例程。

### Lines 422-562: function _chunk_scan_fwd
```python
def _chunk_scan_fwd(
    cb,
    x,
    dt,
    dA_cumsum,
    C,
    states,
    D=None,
    z=None,
    seq_idx=None,
    chunk_indices=None,
    chunk_offsets=None,
    initial_states=None,
    out=None,
):
    batch, seqlen, nheads, headdim = x.shape
    _, _, nchunks, chunk_size = dt.shape
    _, _, ngroups, dstate = C.shape
    assert nheads % ngroups == 0
    assert C.shape == (batch, seqlen, ngroups, dstate)
    assert cb.shape == (batch, nchunks, ngroups, chunk_size, chunk_size)
    if z is not None:
        assert z.shape == x.shape
    if D is not None:
        assert D.shape == (nheads, headdim) or D.shape == (nheads,)
    assert dt.shape == (batch, nheads, nchunks, chunk_size)
    assert dA_cumsum.shape == (batch, nheads, nchunks, chunk_size)
    assert states.shape == (batch, nchunks, nheads, headdim, dstate)

    if seq_idx is not None:
        assert seq_idx.shape == (batch, seqlen)

# ... omitted 97 lines ...
        ),
        D.stride(0) if D is not None else 0,
        True,
        D is not None,
        D.dim() == 2 if D is not None else True,
        BLOCK_SIZE_DSTATE=max(triton.next_power_of_2(dstate), 16),
        HAS_Z=z is not None,
        HAS_SEQ_IDX=seq_idx is not None,
        IS_TRITON_22=TRITON_22,
        HAS_INITSTATES=initial_states is not None,
    )
    return out_x
```
**EN:** Implements the chunk scan fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk scan fwd 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language`
- `packaging.version`
