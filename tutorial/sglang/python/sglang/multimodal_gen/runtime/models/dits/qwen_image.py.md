# qwen_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/qwen_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for qwen image within the multimodal runtime. Key symbols include `_local_seq_len`, `_get_qkv_projections`, `QwenTimestepProjEmbeddings`. / 该模块实现多模态运行时中与 qwen image 相关的模型构件。 关键符号包括 `_local_seq_len`, `_get_qkv_projections`, `QwenTimestepProjEmbeddings`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0

import functools
from typing import Any, Dict, List, Optional, Tuple, Union

import diffusers
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
from diffusers.models.embeddings import TimestepEmbedding, Timesteps
from diffusers.models.modeling_outputs import Transformer2DModelOutput
# ...
try:
    from nunchaku.models.attention import NunchakuFeedForward  # type: ignore[import]
except Exception:
    NunchakuFeedForward = None
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 63-70: Function `_local_seq_len` / 函数 `_local_seq_len`
```python
def _local_seq_len(seq_len: int, sp_world_size: int) -> int:
    """get the local seq len, from seq_len padding to the next multiple of sp_world_size, then shard to local"""
    if sp_world_size <= 1:
        return seq_len
    padded_len = seq_len
    if padded_len % sp_world_size != 0:
        padded_len += sp_world_size - (padded_len % sp_world_size)
    return padded_len // sp_world_size
```
**EN:** This function drives `_local_seq_len` with inputs such as `seq_len`, `sp_world_size`. get the local seq len, from seq_len padding to the next multiple of sp_world_size, then shard to local
**CN:** 这个函数负责 `_local_seq_len`，主要处理 `seq_len`, `sp_world_size` 等输入。 文档字符串说明：get the local seq len, from seq_len padding to the next multiple of sp_world_size, then shard to local

### Lines 73-98: Function `_get_qkv_projections` / 函数 `_get_qkv_projections`
```python
def _get_qkv_projections(
    attn: "QwenImageCrossAttention", hidden_states, encoder_hidden_states=None
):
    if attn.use_fused_qkv:
        img_qkv, _ = attn.to_qkv(hidden_states)
        img_query, img_key, img_value = [
            x.contiguous() for x in img_qkv.chunk(3, dim=-1)
        ]
    else:
        img_query, _ = attn.to_q(hidden_states)
        img_key, _ = attn.to_k(hidden_states)
        img_value, _ = attn.to_v(hidden_states)

    txt_query = txt_key = txt_value = None
# ...
            txt_key, _ = attn.add_k_proj(encoder_hidden_states)
            txt_value, _ = attn.add_v_proj(encoder_hidden_states)

    return img_query, img_key, img_value, txt_query, txt_key, txt_value
```
**EN:** This function drives `_get_qkv_projections` with inputs such as `attn`, `hidden_states`, `encoder_hidden_states`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_get_qkv_projections`，主要处理 `attn`, `hidden_states`, `encoder_hidden_states` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 101-131: Class `QwenTimestepProjEmbeddings` / 类 `QwenTimestepProjEmbeddings`
```python
class QwenTimestepProjEmbeddings(nn.Module):
    def __init__(self, embedding_dim, use_additional_t_cond=False):
        super().__init__()

        self.time_proj = Timesteps(
            num_channels=256, flip_sin_to_cos=True, downscale_freq_shift=0, scale=1000
        )
        self.timestep_embedder = TimestepEmbedding(
            in_channels=256, time_embed_dim=embedding_dim
        )
        self.use_additional_t_cond = use_additional_t_cond
        if use_additional_t_cond:
            self.addition_t_embedding = nn.Embedding(2, embedding_dim)

# ...
            addition_t_emb = addition_t_emb.to(dtype=hidden_states.dtype)
            conditioning = conditioning + addition_t_emb

        return conditioning
```
**EN:** This class models `QwenTimestepProjEmbeddings` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `QwenTimestepProjEmbeddings`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 134-290: Class `QwenEmbedRope` / 类 `QwenEmbedRope`
```python
class QwenEmbedRope(nn.Module):
    def __init__(self, theta: int, axes_dim: List[int], scale_rope=False):
        super().__init__()
        self.theta = theta
        self.axes_dim = axes_dim
        pos_index = torch.arange(4096)
        neg_index = torch.arange(4096).flip(0) * -1 - 1
        self.pos_freqs = torch.cat(
            [
                self.rope_params(pos_index, self.axes_dim[0], self.theta),
                self.rope_params(pos_index, self.axes_dim[1], self.theta),
                self.rope_params(pos_index, self.axes_dim[2], self.theta),
            ],
            dim=1,
# ...
        freqs = torch.cat([freqs_frame, freqs_height, freqs_width], dim=-1).reshape(
            seq_lens, -1
        )
        return freqs.clone().contiguous()
```
**EN:** This class models `QwenEmbedRope` as a specialization of `nn.Module`. Important methods include `__init__`, `rope_params`, `forward`, `_compute_video_freqs`.
**CN:** 该类实现 `QwenEmbedRope`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `rope_params`, `forward`, `_compute_video_freqs`。

### Lines 293-484: Class `QwenEmbedLayer3DRope` / 类 `QwenEmbedLayer3DRope`
```python
class QwenEmbedLayer3DRope(nn.Module):
    def __init__(self, theta: int, axes_dim: List[int], scale_rope=False):
        super().__init__()
        self.theta = theta
        self.axes_dim = axes_dim
        pos_index = torch.arange(4096)
        neg_index = torch.arange(4096).flip(0) * -1 - 1
        self.pos_freqs = torch.cat(
            [
                self.rope_params(pos_index, self.axes_dim[0], self.theta),
                self.rope_params(pos_index, self.axes_dim[1], self.theta),
                self.rope_params(pos_index, self.axes_dim[2], self.theta),
            ],
            dim=1,
# ...
        freqs = torch.cat([freqs_frame, freqs_height, freqs_width], dim=-1).reshape(
            seq_lens, -1
        )
        return freqs.clone().contiguous()
```
**EN:** This class models `QwenEmbedLayer3DRope` as a specialization of `nn.Module`. Important methods include `__init__`, `rope_params`, `forward`, `_compute_video_freqs`.
**CN:** 该类实现 `QwenEmbedLayer3DRope`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `rope_params`, `forward`, `_compute_video_freqs`。

### Lines 487-754: Class `QwenImageCrossAttention` / 类 `QwenImageCrossAttention`
```python
class QwenImageCrossAttention(nn.Module):
    def __init__(
        self,
        dim: int,  # query_dim
        num_heads: int,
        head_dim: int,
        window_size=(-1, -1),
        added_kv_proj_dim: int = None,
        out_bias: bool = True,
        qk_norm=True,  # rmsnorm
        eps=1e-6,
        pre_only=False,
        context_pre_only: bool = False,
        parallel_attention=False,
# ...

        txt_attn_output, _ = self.to_add_out(txt_attn_output)

        return img_attn_output, txt_attn_output
```
**EN:** This class models `QwenImageCrossAttention` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `QwenImageCrossAttention`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 757-776: Class `QwenImageGELU` / 类 `QwenImageGELU`
```python
class QwenImageGELU(nn.Module):
    def __init__(
        self,
        dim: int,
        inner_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.proj = ReplicatedLinear(
            dim,
            inner_dim,
            bias=True,
            quant_config=quant_config,
# ...

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.proj(hidden_states)
        return F.gelu(hidden_states, approximate="tanh")
```
**EN:** This class models `QwenImageGELU` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `QwenImageGELU`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 779-813: Class `QwenImageFeedForward` / 类 `QwenImageFeedForward`
```python
class QwenImageFeedForward(nn.Module):
    def __init__(
        self,
        dim: int,
        dim_out: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        mult: int = 4,
    ) -> None:
        super().__init__()
        inner_dim = dim * mult
        self.net = nn.ModuleList(
            [
                QwenImageGELU(
# ...
        hidden_states = self.net[0](hidden_states)
        hidden_states = self.net[1](hidden_states)
        hidden_states, _ = self.net[2](hidden_states)
        return hidden_states
```
**EN:** This class models `QwenImageFeedForward` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `QwenImageFeedForward`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 816-1119: Class `QwenImageTransformerBlock` / 类 `QwenImageTransformerBlock`
```python
class QwenImageTransformerBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        num_attention_heads: int,
        attention_head_dim: int,
        qk_norm: str = "rms_norm",
        eps: float = 1e-6,
        quant_config: Optional[QuantizationConfig] | NunchakuConfig = None,
        prefix: str = "",
        zero_cond_t: bool = False,
    ):
        super().__init__()
        self.prefix = prefix
# ...
        if hidden_states.dtype == torch.float16:
            hidden_states = hidden_states.clip(-65504, 65504)

        return encoder_hidden_states, hidden_states
```
**EN:** This class models `QwenImageTransformerBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `_modulate`, `forward`.
**CN:** 该类实现 `QwenImageTransformerBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `_modulate`, `forward`。

### Lines 1122-1125: Function `to_hashable` / 函数 `to_hashable`
```python
def to_hashable(obj):
    if isinstance(obj, list):
        return tuple(to_hashable(x) for x in obj)
    return obj
```
**EN:** This function drives `to_hashable` with inputs such as `obj`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `to_hashable`，主要处理 `obj` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1128-1395: Class `QwenImageTransformer2DModel` / 类 `QwenImageTransformer2DModel`
```python
class QwenImageTransformer2DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    """
    The Transformer model introduced in Qwen.

    """

    _supports_gradient_checkpointing = True
    _no_split_modules = ["QwenImageTransformerBlock"]
    _skip_layerwise_casting_patterns = ["pos_embed", "norm"]
    _repeated_blocks = ["QwenImageTransformerBlock"]

    param_names_mapping = QwenImageDitConfig().arch_config.param_names_mapping
    _fsdp_shard_conditions = QwenImageDitConfig().arch_config._fsdp_shard_conditions

# ...
        hidden_states = self.norm_out(hidden_states, temb_txt)

        output, _ = self.proj_out(hidden_states)
        return output
```
**EN:** This class models `QwenImageTransformer2DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. The Transformer model introduced in Qwen. Important methods include `get_nunchaku_quant_rules`, `__init__`, `build_modulate_index`, `forward`.
**CN:** 该类实现 `QwenImageTransformer2DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：The Transformer model introduced in Qwen. 其中较重要的方法包括 `get_nunchaku_quant_rules`, `__init__`, `build_modulate_index`, `forward`。

### Lines 1396-1398: Top-level configuration / 顶层配置
```python


EntryClass = QwenImageTransformer2DModel
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.qwenimage`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.elementwise`, `sglang.multimodal_gen.runtime.layers.fused_scale_shift_gate`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`
- **External / 外部**: `diffusers`, `numpy`, `torch`, `torch.nn`, `torch.nn.functional`, `diffusers.models.embeddings`, `diffusers.models.modeling_outputs`, `diffusers.models.normalization`
- **Stdlib / 标准库**: `functools`, `typing`
