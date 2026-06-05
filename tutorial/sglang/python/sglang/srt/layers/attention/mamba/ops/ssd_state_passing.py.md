# ssd_state_passing.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/mamba/ops/ssd_state_passing.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the ssd state passing structures that carry execution metadata through the attention pipeline. / 该模块定义 ssd state passing 结构，用于在注意力流水线中传递执行元数据。
## Line-by-Line Analysis / 逐行分析
### Lines 11-13: imports
```python
import torch
import triton
import triton.language as tl
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 16-168: function _state_passing_fwd_kernel
```python
@triton.jit
def _state_passing_fwd_kernel(
    # Pointers to matrices
    states_ptr,
    out_ptr,
    final_states_ptr,
    dA_cs_ptr,
    initstates_ptr,
    seq_idx_ptr,
    chunk_offsets_ptr,
    chunk_meta_num,
    # Matrix dimensions
    dim,
    nchunks,
    seqlen,
    chunk_size,
    # Strides
    stride_states_batch,
    stride_states_chunk,
    stride_states_head,
    stride_states_dim,
    stride_out_batch,
    stride_out_chunk,
    stride_out_head,
    stride_out_dim,
    stride_final_states_batch,
    stride_final_states_head,
    stride_final_states_dim,
    stride_dA_cs_batch,
    stride_dA_cs_chunk,
    stride_dA_cs_head,
    stride_dA_cs_csize,
# ... omitted 109 lines ...
                scale_mask = seq_idx_chunk_end == prev_seq_idx_chunk_end
            prev_seq_idx_chunk_end = seq_idx_chunk_end

        scale = tl.where(scale_mask, tl.exp(dA_cs), 0.0)
        states = scale * states + new_states
        if c < nchunks - 1:
            tl.store(out_ptrs, states, mask=offs_m < dim)
        else:
            tl.store(final_states_ptrs, states, mask=offs_m < dim)
        states_ptrs += stride_states_chunk
        dA_cs_ptr += stride_dA_cs_chunk
        out_ptrs += stride_out_chunk
```
**EN:** Implements the state passing fwd kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 state passing fwd kernel 例程。

### Lines 171-264: function _state_passing_fwd
```python
def _state_passing_fwd(
    states,
    dA_cumsum,
    initial_states=None,
    seq_idx=None,
    chunk_size=None,
    out_dtype=None,
    is_cont_batched=False,
    chunk_offsets=None,
):
    batch, nchunks, nheads, dim = states.shape
    if chunk_size is None:
        chunk_size = dA_cumsum.shape[-1]
    else:
        assert chunk_size == dA_cumsum.shape[-1]
    assert dA_cumsum.shape == (batch, nheads, nchunks, chunk_size)
    if initial_states is not None:
        if is_cont_batched:
            # - if cu_seqlens is provided, then the initial states
            #   are used for continuous batching. In which case we
            #   require seq_idx to be provided
            assert (
                seq_idx is not None
            ), "seq_idx must be provided for continuous batching"
            # - we also need chunk_offsets to be provided, to account
            #   for computation of dA_cumsum from the start of the
            #   sequence
            assert (
                chunk_offsets is not None
            ), "chunk_offsets must be provided for continuous batching"
        else:
            # - this is the regular batching case, where initial
# ... omitted 50 lines ...
                else (0, 0, 0)
            ),
            *(
                (seq_idx.stride(0), seq_idx.stride(1))
                if seq_idx is not None
                else (0, 0)
            ),
            HAS_INITSTATES=initial_states is not None,
            HAS_SEQ_IDX=seq_idx is not None,
            IS_CONT_BATCHED=is_cont_batched,
        )
    return out, final_states
```
**EN:** Implements the state passing fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 state passing fwd 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language`
