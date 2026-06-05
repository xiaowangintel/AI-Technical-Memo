# rocm_mla_decode_rope.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/triton_ops/rocm_mla_decode_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects rocm mla decode rope helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 rocm mla decode rope 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 14-17: docstring
```python
"""
Memory-efficient attention for decoding.
It supports page size = 1.
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 23-28: imports
```python
import triton
import triton.language as tl

from sglang.srt.layers.attention.triton_ops.decode_attention import (
    _decode_softmax_reducev_fwd,
)
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 31-32: function is_hip
```python
def is_hip():
    return triton.runtime.driver.active.get_current_target().backend == "hip"
```
**EN:** Checks whether is hip is valid for the current configuration or execution state.
**CN:** 检查在当前配置或执行状态下 is hip 是否成立。

### Lines 35-35: module constants
```python
_is_hip = is_hip()
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 38-41: function tanh
```python
@triton.jit
def tanh(x):
    # Tanh is just a scaled sigmoid
    return 2 * tl.sigmoid(2 * x) - 1
```
**EN:** Implements the tanh routine used by this attention module.
**CN:** 实现该注意力模块使用的 tanh 例程。

### Lines 44-306: function _fwd_grouped_kernel_stage1_rope
```python
@triton.jit
def _fwd_grouped_kernel_stage1_rope(
    Q,  # Holds [Q_NOPE; Q_PE], b x h x (d+r)
    K_Buffer,  # Holds [KV; K_PE], b*s x (c+r)
    V_buffer,  # Holds [KV], b*s x (c)
    cos_sin_cache,  # max_seq_len x (rotary_dim * 2)
    positions,  # sequence positions
    sm_scale,
    kv_indptr,
    kv_indices,
    Att_Out,  # b x h x NUM_KV_SPLITS x (kv_lora_rank + 1)
    k_pe_t_out,
    stride_qb,
    stride_qh,
    stride_buf_kbs,
    stride_buf_vbs,
    stride_mid_ob,
    stride_mid_oh,
    stride_mid_os,
    stride_kpe_tokens_out_b,
    stride_cos_sin_cache_s,
    stride_positions_b,
    rotary_dim: tl.constexpr,
    kv_lora_rank: tl.constexpr,
    qk_rope_head_dim: tl.constexpr,
    kv_group_num: tl.constexpr,
    q_head_num: tl.constexpr,
    BLOCK_C: tl.constexpr,
    BLOCK_R: tl.constexpr,
    BLOCK_N: tl.constexpr,
    BLOCK_H: tl.constexpr,
    NUM_KV_SPLITS: tl.constexpr,
# ... omitted 219 lines ...
        offs_mid_o_1 = (
            cur_batch * stride_mid_ob
            + cur_head * stride_mid_oh
            + split_kv_id * stride_mid_os
            + kv_lora_rank
        )

        tl.store(
            Att_Out + offs_mid_o_1,
            e_max + tl.log(e_sum),
            mask=mask_h,
        )
```
**EN:** Implements the fwd grouped kernel stage1 rope routine used by this attention module.
**CN:** 实现该注意力模块使用的 fwd grouped kernel stage1 rope 例程。

### Lines 310-399: function _decode_grouped_att_m_fwd_rope
```python
def _decode_grouped_att_m_fwd_rope(
    q,
    k_buffer,
    v_buffer,
    att_out,
    k_pe_tokens_out,
    kv_lora_rank,  # c
    cos_sin_cache,
    positions,
    rotary_dim,
    kv_indptr,
    kv_indices,
    num_kv_splits,
    sm_scale,
    logit_cap,
    use_rope,
    is_neox_style=True,
):
    if use_rope:
        assert (
            k_pe_tokens_out is not None
        ), "We must output the k_pe tokens with rope applied if rope fusion enabled."

    BLOCK = 32

    # # [TODO] work around shmem limit on MI3xx
    # if _is_hip and kv_lora_rank >= 576:
    #     BLOCK = 16

    qk_rope_head_dim = k_buffer.shape[-1] - kv_lora_rank
    batch, head_num = kv_indptr.shape[0] - 1, q.shape[1]
    kv_group_num = q.shape[1] // k_buffer.shape[1]
# ... omitted 46 lines ...
        BLOCK_C=BLOCK_C,
        BLOCK_R=BLOCK_R,
        BLOCK_N=BLOCK,
        BLOCK_H=BLOCK_H,
        NUM_KV_SPLITS=NUM_KV_SPLITS,
        logit_cap=logit_cap,
        USE_ROPE=use_rope,
        IS_NEOX_STYLE=is_neox_style,
        num_warps=4,
        num_stages=num_stages,
        **extra_kargs,
    )
```
**EN:** Implements the decode grouped att m fwd rope routine used by this attention module.
**CN:** 实现该注意力模块使用的 decode grouped att m fwd rope 例程。

### Lines 402-439: function decode_attention_fwd_grouped_rope
```python
def decode_attention_fwd_grouped_rope(
    q,
    k_buffer,
    v_buffer,
    o,
    kv_indptr,
    kv_indices,
    k_pe_tokens,
    kv_lora_rank,
    rotary_dim,
    cos_sin_cache,
    positions,
    attn_logits,
    num_kv_splits,
    sm_scale,
    logit_cap=0.0,
    use_rope=False,
    is_neox_style=False,
):
    _decode_grouped_att_m_fwd_rope(
        q,
        k_buffer,
        v_buffer,
        attn_logits,
        k_pe_tokens,
        kv_lora_rank,
        cos_sin_cache,
        positions,
        rotary_dim,
        kv_indptr,
        kv_indices,
        num_kv_splits,
        sm_scale,
        logit_cap,
        use_rope,
        is_neox_style,
    )
    _decode_softmax_reducev_fwd(attn_logits, q, o, v_buffer, kv_indptr, num_kv_splits)
```
**EN:** Implements the decode attention fwd grouped rope routine used by this attention module.
**CN:** 实现该注意力模块使用的 decode attention fwd grouped rope 例程。

## Key Concepts / 关键概念
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Speculative decoding support / **CN:** 投机解码支持
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `triton`
- `triton.language`
- `sglang.srt.layers.attention.triton_ops.decode_attention._decode_softmax_reducev_fwd`
