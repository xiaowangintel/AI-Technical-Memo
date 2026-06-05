# deepseek_v2_attention_mla_npu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/modules/deepseek_v2_attention_mla_npu.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for attention execution and masking inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的注意力执行与掩码处理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Module setup and shared state / 模块设置与共享状态
```python
import re
from typing import TYPE_CHECKING

import torch
import torch_npu
from sgl_kernel_npu.norm.fused_split_qk_norm import fused_split_qk_norm

from sglang.srt.environ import envs
from sglang.srt.hardware_backend.npu.attention.mla_preprocess import (
    NPUFusedMLAPreprocess,
    is_fia_nz,
    is_mla_preprocess_enabled,
)
from sglang.srt.layers.attention.nsa.nsa_indexer import scattered_to_tp_attn_full
from sglang.srt.layers.attention.nsa.utils import (
    nsa_use_prefill_cp,
)
from sglang.srt.layers.communicator import ScatterMode, get_attn_tp_context

if TYPE_CHECKING:
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
    from sglang.srt.models.deepseek_v2 import DeepseekV2AttentionMLA
    from sglang.srt.utils import BumpAllocator
_use_ag_after_qlora = envs.SGLANG_USE_AG_AFTER_QLORA.get()
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `re`, `typing`, `torch`, `torch_npu`, `sgl_kernel_npu.norm.fused_split_qk_norm`, `sglang.srt.environ`. It also defines symbols such as `_use_ag_after_qlora` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `re`, `typing`, `torch`, `torch_npu`, `sgl_kernel_npu.norm.fused_split_qk_norm`, `sglang.srt.environ`。 同时定义了 `_use_ag_after_qlora` 等符号，供后续逻辑使用。

### Lines 28-130: Function `forward_mha_prepare_npu` / 函数 `forward_mha_prepare_npu`
```python
def forward_mha_prepare_npu(
    m: "DeepseekV2AttentionMLA",
    positions: torch.Tensor,
    hidden_states: torch.Tensor,
    forward_batch: "ForwardBatch",
    zero_allocator: "BumpAllocator",
    layer_scatter_modes,
):
    if m.q_lora_rank is not None:
        q, latent_cache = (
            get_attn_tp_context()
            .fetch_qkv_latent()
            .split(
                [m.q_lora_rank, m.kv_lora_rank + m.qk_rope_head_dim],
                dim=-1,
            )
        )

        # NSA Indexer: cache quantized keys, auto-skip topk for sequences <= nsa_index_topk

        if m.use_nsa:
            q_lora = m.q_a_layernorm(q)
            q = m.q_b_proj(q_lora)[0].view(-1, m.num_local_heads, m.qk_head_dim)
            _ = m.indexer(
                x=hidden_states,
                q_lora=q_lora,
                positions=positions,
                forward_batch=forward_batch,
# ... omitted for brevity ...
    kv = kv.view(-1, m.num_local_heads, m.qk_nope_head_dim + m.v_head_dim)
    k_nope = kv[..., : m.qk_nope_head_dim]
    v = kv[..., m.qk_nope_head_dim :]

    k = m._concat_and_cast_mha_k(k_nope, k_pe, forward_batch)
    return q, k, v, forward_batch
```
**EN:** This function implements `forward_mha_prepare_npu`. It primarily calls `q.split`, `latent_cache.split`, `latent_cache.unsqueeze`, `kv.view`, `m._concat_and_cast_mha_k`, `get_attn_tp_context.fetch_qkv_latent.split` to complete its work. State updates are written into `latent_cache`, `q`, `kv`, `k_nope`, `v`, `k`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `forward_mha_prepare_npu`。 它主要通过调用 `q.split`, `latent_cache.split`, `latent_cache.unsqueeze`, `kv.view`, `m._concat_and_cast_mha_k`, `get_attn_tp_context.fetch_qkv_latent.split` 来完成任务。 状态更新主要写入 `latent_cache`, `q`, `kv`, `k_nope`, `v`, `k`。 实现中使用了条件分支。

### Lines 133-143: Function `forward_mha_core_npu` / 函数 `forward_mha_core_npu`
```python
def forward_mha_core_npu(
    m: "DeepseekV2AttentionMLA",
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    forward_batch: "ForwardBatch",
) -> torch.Tensor:
    attn_output = m.attn_mha(q, k, v, forward_batch, save_kv_cache=False)
    attn_output = attn_output.reshape(-1, m.num_local_heads * m.v_head_dim)
    output, _ = m.o_proj(attn_output)
    return output
```
**EN:** This function implements `forward_mha_core_npu`. It primarily calls `m.attn_mha`, `attn_output.reshape`, `m.o_proj` to complete its work. State updates are written into `attn_output`.
**CN:** 该函数实现了 `forward_mha_core_npu`。 它主要通过调用 `m.attn_mha`, `attn_output.reshape`, `m.o_proj` 来完成任务。 状态更新主要写入 `attn_output`。

### Lines 150-253: Function `forward_mla_prepare_npu` / 函数 `forward_mla_prepare_npu`
```python
def forward_mla_prepare_npu(
    m: "DeepseekV2AttentionMLA",
    positions: torch.Tensor,
    hidden_states: torch.Tensor,
    forward_batch: "ForwardBatch",
    zero_allocator: "BumpAllocator",
    layer_scatter_modes,
):
    if is_mla_preprocess_enabled():
        if not hasattr(m, "mla_preprocess"):
            m.mla_preprocess = NPUFusedMLAPreprocess(
                m.fused_qkv_a_proj_with_mqa,
                m.q_a_layernorm,
                m.kv_a_layernorm,
                m.q_b_proj,
                m.w_kc,
                m.rotary_emb,
                m.layer_id,
                m.num_local_heads,
                m.qk_nope_head_dim,
                m.qk_rope_head_dim,
                m.quant_config,
            )
        (
            q_pe,
            k_pe,
            q_nope_out,
            k_nope,
# ... omitted for brevity ...
        k_nope,
        forward_batch,
        zero_allocator,
        positions,
        topk_indices,
    )
```
**EN:** This function implements `forward_mla_prepare_npu`. It primarily calls `is_mla_preprocess_enabled`, `m.mla_preprocess.forward`, `q.split`, `latent_cache.unsqueeze`, `torch.bmm`, `q_nope_out.transpose` to complete its work. State updates are written into `topk_indices`, `q_lora`, `k_pe`, `q_nope_out`, `m.mla_preprocess`, `k_nope`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `forward_mla_prepare_npu`。 它主要通过调用 `is_mla_preprocess_enabled`, `m.mla_preprocess.forward`, `q.split`, `latent_cache.unsqueeze`, `torch.bmm`, `q_nope_out.transpose` 来完成任务。 状态更新主要写入 `topk_indices`, `q_lora`, `k_pe`, `q_nope_out`, `m.mla_preprocess`, `k_nope`。 实现中使用了条件分支。

### Lines 256-291: Function `forward_mla_core_npu` / 函数 `forward_mla_core_npu`
```python
def forward_mla_core_npu(
    m: "DeepseekV2AttentionMLA",
    q_pe: torch.Tensor,
    k_pe: torch.Tensor,
    q_nope_out: torch.Tensor,
    k_nope: torch.Tensor,
    forward_batch: "ForwardBatch",
    zero_allocator: "BumpAllocator",
    positions: torch.Tensor,
    topk_indices: torch.Tensor,
) -> torch.Tensor:
    attn_output = m.attn_mqa(
        q_nope_out,
        k_nope,
        k_nope,
        forward_batch,
        q_rope=q_pe,
        k_rope=k_pe,
        **(dict(topk_indices=topk_indices) if topk_indices is not None else {}),
    )

    attn_output = attn_output.view(-1, m.num_local_heads, m.kv_lora_rank)

    attn_bmm_output = torch.empty(
        (attn_output.shape[0], m.num_local_heads, m.v_head_dim),
        dtype=attn_output.dtype,
        device=attn_output.device,
    )

    attn_output = attn_output.contiguous()
    torch.ops.npu.batch_matmul_transpose(attn_output, m.w_vc, attn_bmm_output)

    attn_bmm_output = attn_bmm_output.reshape(-1, m.num_local_heads * m.v_head_dim)
    output, _ = m.o_proj(attn_bmm_output)

    return output
```
**EN:** This function implements `forward_mla_core_npu`. It primarily calls `m.attn_mqa`, `attn_output.view`, `torch.empty`, `attn_output.contiguous`, `torch.ops.npu.batch_matmul_transpose`, `attn_bmm_output.reshape` to complete its work. State updates are written into `attn_output`, `attn_bmm_output`.
**CN:** 该函数实现了 `forward_mla_core_npu`。 它主要通过调用 `m.attn_mqa`, `attn_output.view`, `torch.empty`, `attn_output.contiguous`, `torch.ops.npu.batch_matmul_transpose`, `attn_bmm_output.reshape` 来完成任务。 状态更新主要写入 `attn_output`, `attn_bmm_output`。

### Lines 298-429: Function `forward_dsa_prepare_npu` / 函数 `forward_dsa_prepare_npu`
```python
def forward_dsa_prepare_npu(
    m: "DeepseekV2AttentionMLA",
    positions: torch.Tensor,
    hidden_states: torch.Tensor,
    forward_batch: "ForwardBatch",
    zero_allocator: "BumpAllocator",
    layer_scatter_modes,
    prev_topk_indices: torch.Tensor = None,
):
    dynamic_scale = None
    if is_mla_preprocess_enabled() and forward_batch.forward_mode.is_decode():
        (
            q_pe,
            k_pe,
            q_nope_out,
            k_nope,
            q_lora,
            forward_batch,
            zero_allocator,
            positions,
            dynamic_scale,
        ) = npu_mla_preprocess(
            m,
            hidden_states,
            positions,
            forward_batch,
            zero_allocator,
        )
# ... omitted for brevity ...
        k_nope,
        topk_indices,
        forward_batch,
        zero_allocator,
        positions,
    )
```
**EN:** This function implements `forward_dsa_prepare_npu`. It primarily calls `is_mla_preprocess_enabled`, `forward_batch.forward_mode.is_decode`, `npu_mla_preprocess`, `q.split`, `torch.bmm`, `q_nope_out.transpose` to complete its work. State updates are written into `dynamic_scale`, `fused_qkv_a_proj_out`, `q_nope_out`, `topk_indices`, `q`, `q_lora`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `forward_dsa_prepare_npu`。 它主要通过调用 `is_mla_preprocess_enabled`, `forward_batch.forward_mode.is_decode`, `npu_mla_preprocess`, `q.split`, `torch.bmm`, `q_nope_out.transpose` 来完成任务。 状态更新主要写入 `dynamic_scale`, `fused_qkv_a_proj_out`, `q_nope_out`, `topk_indices`, `q`, `q_lora`。 实现中使用了条件分支、上下文管理资源。

### Lines 432-484: Function `forward_dsa_core_npu` / 函数 `forward_dsa_core_npu`
```python
def forward_dsa_core_npu(
    m: "DeepseekV2AttentionMLA",
    q_pe: torch.Tensor,
    k_pe: torch.Tensor,
    q_nope_out: torch.Tensor,
    k_nope: torch.Tensor,
    topk_indices: torch.Tensor,
    forward_batch: "ForwardBatch",
    zero_allocator: "BumpAllocator",
    positions: torch.Tensor,
) -> torch.Tensor:
    attn_output = m.attn_mqa(
        q_nope_out.contiguous(),
        k_nope.contiguous(),
        k_nope.contiguous(),
        forward_batch,
        save_kv_cache=True,  # False if forward_batch.forward_mode.is_extend() else True,
        q_rope=q_pe.contiguous(),
        k_rope=k_pe.contiguous(),
        topk_indices=topk_indices,
    )
    attn_output = attn_output.view(-1, m.num_local_heads, m.kv_lora_rank)

    attn_bmm_output = torch.empty(
        (attn_output.shape[0], m.num_local_heads, m.v_head_dim),
        dtype=attn_output.dtype,
        device=attn_output.device,
    )
# ... omitted for brevity ...

    output, _ = m.o_proj(attn_bmm_output)
    if not m.next_skip_topk:
        return output, None
    else:
        return output, topk_indices
```
**EN:** This function implements `forward_dsa_core_npu`. It primarily calls `m.attn_mqa`, `attn_output.view`, `torch.empty`, `attn_bmm_output.reshape`, `m.o_proj`, `q_nope_out.contiguous` to complete its work. State updates are written into `attn_output`, `attn_bmm_output`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `forward_dsa_core_npu`。 它主要通过调用 `m.attn_mqa`, `attn_output.view`, `torch.empty`, `attn_bmm_output.reshape`, `m.o_proj`, `q_nope_out.contiguous` 来完成任务。 状态更新主要写入 `attn_output`, `attn_bmm_output`。 实现中使用了条件分支。

### Lines 487-580: Function `npu_mla_preprocess` / 函数 `npu_mla_preprocess`
```python
def npu_mla_preprocess(
    m: "DeepseekV2AttentionMLA",
    hidden_states: torch.Tensor,
    positions: torch.Tensor,
    forward_batch: "ForwardBatch",
    zero_allocator: "BumpAllocator",
):
    dynamic_scale = None
    if not hasattr(m, "mla_preprocess"):
        m.mla_preprocess = NPUFusedMLAPreprocess(
            m.fused_qkv_a_proj_with_mqa,
            m.q_a_layernorm,
            m.kv_a_layernorm,
            m.q_b_proj,
            m.w_kc,
            m.rotary_emb,
            m.layer_id,
            m.num_local_heads,
            m.qk_nope_head_dim,
            m.qk_rope_head_dim,
            m.v_head_dim,
            m.quant_config,
        )
    # mlaprolog does not require additional calculation of q_lora
    _is_mlaprolog = hasattr(m.quant_config, "ignore") and any(
        re.fullmatch(r".*kv_b_proj", l) for l in m.quant_config.ignore
    )
    if _is_mlaprolog:
# ... omitted for brevity ...
        q_lora,
        forward_batch,
        zero_allocator,
        positions,
        dynamic_scale,
    )
```
**EN:** This function implements `npu_mla_preprocess`. It primarily calls `hasattr`, `NPUFusedMLAPreprocess`, `any`, `m.mla_preprocess.forward`, `torch.npu.Event`, `mla_event.record` to complete its work. State updates are written into `dynamic_scale`, `_is_mlaprolog`, `m.mla_preprocess`, `mla_event`, `fused_qkv_a_proj_out`, `q_lora`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `npu_mla_preprocess`。 它主要通过调用 `hasattr`, `NPUFusedMLAPreprocess`, `any`, `m.mla_preprocess.forward`, `torch.npu.Event`, `mla_event.record` 来完成任务。 状态更新主要写入 `dynamic_scale`, `_is_mlaprolog`, `m.mla_preprocess`, `mla_event`, `fused_qkv_a_proj_out`, `q_lora`。 实现中使用了条件分支、上下文管理资源。

## Key Concepts / 关键概念
- **Functions / 函数**: `forward_mha_prepare_npu`, `forward_mha_core_npu`, `forward_mla_prepare_npu`, `forward_mla_core_npu`, `forward_dsa_prepare_npu`, `forward_dsa_core_npu`, `npu_mla_preprocess`
- **Themes / 主题**: `attention`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`, `sglang.srt.hardware_backend.npu.attention.mla_preprocess`, `sglang.srt.layers.attention.nsa.nsa_indexer`, `sglang.srt.layers.attention.nsa.utils`, `sglang.srt.layers.communicator`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.models.deepseek_v2`, `sglang.srt.utils`
- **External / 外部依赖**: `torch`, `torch_npu`, `sgl_kernel_npu.norm.fused_split_qk_norm`
- **Standard library / 标准库**: `re`, `typing`
