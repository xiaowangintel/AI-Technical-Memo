# helios.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/helios.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for helios within the multimodal runtime. Key symbols include `pad_for_3d_conv`, `center_down_sample_3d`, `apply_rotary_emb_transposed`. / 该模块实现多模态运行时中与 helios 相关的模型构件。 关键符号包括 `pad_for_3d_conv`, `center_down_sample_3d`, `apply_rotary_emb_transposed`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-56: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
# Adapted from Helios diffusers transformer:
# https://github.com/BestWishYsh/Helios
"""
Helios Transformer 3D model for video generation.

Implements the HeliosTransformer3DModel with multi-term memory patches,
3D rotary position embeddings, and per-block scale-shift modulation.
"""

import math
from functools import lru_cache
from typing import Any

# ...
from sglang.multimodal_gen.runtime.models.dits.base import CachableDiT
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 64-71: Function `pad_for_3d_conv` / 函数 `pad_for_3d_conv`
```python
def pad_for_3d_conv(x, kernel_size):
    """Pad input to make it divisible by kernel_size using replicate mode."""
    b, c, t, h, w = x.shape
    pt, ph, pw = kernel_size
    pad_t = (pt - (t % pt)) % pt
    pad_h = (ph - (h % ph)) % ph
    pad_w = (pw - (w % pw)) % pw
    return F.pad(x, (0, pad_w, 0, pad_h, 0, pad_t), mode="replicate")
```
**EN:** This function drives `pad_for_3d_conv` with inputs such as `x`, `kernel_size`. Pad input to make it divisible by kernel_size using replicate mode.
**CN:** 这个函数负责 `pad_for_3d_conv`，主要处理 `x`, `kernel_size` 等输入。 文档字符串说明：Pad input to make it divisible by kernel_size using replicate mode.

### Lines 74-76: Function `center_down_sample_3d` / 函数 `center_down_sample_3d`
```python
def center_down_sample_3d(x, kernel_size):
    """Average pooling for 3D downsampling."""
    return F.avg_pool3d(x, kernel_size, stride=kernel_size)
```
**EN:** This function drives `center_down_sample_3d` with inputs such as `x`, `kernel_size`. Average pooling for 3D downsampling.
**CN:** 这个函数负责 `center_down_sample_3d`，主要处理 `x`, `kernel_size` 等输入。 文档字符串说明：Average pooling for 3D downsampling.

### Lines 79-86: Function `apply_rotary_emb_transposed` / 函数 `apply_rotary_emb_transposed`
```python
def apply_rotary_emb_transposed(hidden_states, freqs_cis):
    """Apply rotary positional embeddings with transposed cos/sin format."""
    x_1, x_2 = hidden_states.unflatten(-1, (-1, 2)).unbind(-1)
    cos, sin = freqs_cis.unsqueeze(-2).chunk(2, dim=-1)
    out = torch.empty_like(hidden_states)
    out[..., 0::2] = x_1 * cos[..., 0::2] - x_2 * sin[..., 1::2]
    out[..., 1::2] = x_1 * sin[..., 1::2] + x_2 * cos[..., 0::2]
    return out.type_as(hidden_states)
```
**EN:** This function drives `apply_rotary_emb_transposed` with inputs such as `hidden_states`, `freqs_cis`. Apply rotary positional embeddings with transposed cos/sin format.
**CN:** 这个函数负责 `apply_rotary_emb_transposed`，主要处理 `hidden_states`, `freqs_cis` 等输入。 文档字符串说明：Apply rotary positional embeddings with transposed cos/sin format.

### Lines 94-111: Class `HeliosOutputNorm` / 类 `HeliosOutputNorm`
```python
class HeliosOutputNorm(nn.Module):
    def __init__(self, dim: int, eps: float = 1e-6):
        super().__init__()
        self.scale_shift_table = nn.Parameter(torch.randn(1, 2, dim) / dim**0.5)
        self.norm = LayerNormScaleShift(
            dim, eps=eps, elementwise_affine=False, dtype=torch.float32
        )

    def forward(self, hidden_states, temb, original_context_length):
        temb = temb[:, -original_context_length:, :]
        shift, scale = (
            self.scale_shift_table.unsqueeze(0).to(temb.device) + temb.unsqueeze(2)
        ).chunk(2, dim=2)
        shift = shift.squeeze(2).to(hidden_states.device)
        scale = scale.squeeze(2).to(hidden_states.device)
        hidden_states = hidden_states[:, -original_context_length:, :]
        hidden_states = self.norm(hidden_states, shift, scale)
        return hidden_states
```
**EN:** This class models `HeliosOutputNorm` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HeliosOutputNorm`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 119-196: Class `HeliosRotaryPosEmbed` / 类 `HeliosRotaryPosEmbed`
```python
class HeliosRotaryPosEmbed(nn.Module):
    """3D rotary position embeddings for (time, height, width)."""

    def __init__(self, rope_dim, theta):
        super().__init__()
        self.DT, self.DY, self.DX = rope_dim
        self.theta = theta
        # Store as plain attributes (not buffers) to avoid meta-device issues
        # during FSDP loading. They'll be re-created on the correct device in forward.
        self._freqs_base_t = None
        self._freqs_base_y = None
        self._freqs_base_x = None

    def _get_freqs_base(self, dim):
# ...
            ],
            dim=0,
        )
        return result.permute(1, 0, 2, 3, 4)
```
**EN:** This class models `HeliosRotaryPosEmbed` as a specialization of `nn.Module`. 3D rotary position embeddings for (time, height, width). Important methods include `__init__`, `_get_freqs_base`, `_ensure_freqs_base`, `get_frequency_batched`.
**CN:** 该类实现 `HeliosRotaryPosEmbed`，并继承/扩展 `nn.Module`。 文档字符串指出：3D rotary position embeddings for (time, height, width). 其中较重要的方法包括 `__init__`, `_get_freqs_base`, `_ensure_freqs_base`, `get_frequency_batched`。

### Lines 204-226: Class `HeliosTimeTextEmbedding` / 类 `HeliosTimeTextEmbedding`
```python
class HeliosTimeTextEmbedding(nn.Module):
    """Condition embedder combining timestep and text embeddings."""

    def __init__(self, dim, time_freq_dim, time_proj_dim, text_embed_dim):
        super().__init__()
        self.time_embedder = TimestepEmbedder(
            dim, frequency_embedding_size=time_freq_dim, act_layer="silu"
        )
        self.time_modulation = ModulateProjection(dim, factor=6, act_layer="silu")
        self.text_embedder = MLP(
            text_embed_dim, dim, dim, bias=True, act_type="gelu_pytorch_tanh"
        )

    def forward(
# ...
        if encoder_hidden_states is not None and is_return_encoder_hidden_states:
            encoder_hidden_states = self.text_embedder(encoder_hidden_states)

        return temb, timestep_proj, encoder_hidden_states
```
**EN:** This class models `HeliosTimeTextEmbedding` as a specialization of `nn.Module`. Condition embedder combining timestep and text embeddings. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HeliosTimeTextEmbedding`，并继承/扩展 `nn.Module`。 文档字符串指出：Condition embedder combining timestep and text embeddings. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 234-328: Class `HeliosSelfAttention` / 类 `HeliosSelfAttention`
```python
class HeliosSelfAttention(nn.Module):
    """Self-attention with RMSNorm Q/K, optional history key amplification."""

    def __init__(
        self,
        dim: int,
        num_heads: int,
        eps: float = 1e-6,
        is_amplify_history: bool = False,
        history_scale_mode: str = "per_head",
        quant_config: QuantizationConfig | None = None,
    ):
        super().__init__()
        self.dim = dim
# ...
        x = self.attn(q, k, v, num_replicated_prefix=history_seq_len)
        x = x.flatten(2)
        x, _ = self.to_out(x)
        return x
```
**EN:** This class models `HeliosSelfAttention` as a specialization of `nn.Module`. Self-attention with RMSNorm Q/K, optional history key amplification. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HeliosSelfAttention`，并继承/扩展 `nn.Module`。 文档字符串指出：Self-attention with RMSNorm Q/K, optional history key amplification. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 336-395: Class `HeliosCrossAttention` / 类 `HeliosCrossAttention`
```python
class HeliosCrossAttention(nn.Module):
    """Cross-attention with RMSNorm Q/K normalization."""

    def __init__(
        self,
        dim: int,
        num_heads: int,
        eps: float = 1e-6,
        quant_config: QuantizationConfig | None = None,
    ):
        super().__init__()
        self.dim = dim
        self.num_heads = num_heads
        self.head_dim = dim // num_heads
# ...
        x = self.attn(q, k, v)
        x = x.flatten(2)
        x, _ = self.to_out(x)
        return x
```
**EN:** This class models `HeliosCrossAttention` as a specialization of `nn.Module`. Cross-attention with RMSNorm Q/K normalization. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HeliosCrossAttention`，并继承/扩展 `nn.Module`。 文档字符串指出：Cross-attention with RMSNorm Q/K normalization. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 403-522: Class `HeliosTransformerBlock` / 类 `HeliosTransformerBlock`
```python
class HeliosTransformerBlock(nn.Module):
    """
    Single transformer block with self-attention, cross-attention, FFN,
    and scale-shift modulation from timestep embeddings.
    """

    def __init__(
        self,
        dim: int,
        ffn_dim: int,
        num_heads: int,
        cross_attn_norm: bool = True,
        eps: float = 1e-6,
        guidance_cross_attn: bool = True,
# ...
            hidden_states.float() + ff_output.float() * c_gate_msa
        ).type_as(hidden_states)

        return hidden_states
```
**EN:** This class models `HeliosTransformerBlock` as a specialization of `nn.Module`. Single transformer block with self-attention, cross-attention, FFN, Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HeliosTransformerBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：Single transformer block with self-attention, cross-attention, FFN, 其中较重要的方法包括 `__init__`, `forward`。

### Lines 530-894: Class `HeliosTransformer3DModel` / 类 `HeliosTransformer3DModel`
```python
class HeliosTransformer3DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    """
    Helios Transformer 3D model for video generation.

    Implements multi-scale history patches, 3D RoPE, and chunked denoising
    with zero_history_timestep and guidance_cross_attn.
    """

    _fsdp_shard_conditions = HeliosConfig()._fsdp_shard_conditions
    _compile_conditions = HeliosConfig()._compile_conditions
    _supported_attention_backends = HeliosConfig()._supported_attention_backends
    param_names_mapping = HeliosConfig().param_names_mapping
    reverse_param_names_mapping = HeliosConfig().reverse_param_names_mapping
    lora_param_names_mapping = HeliosConfig().lora_param_names_mapping
# ...
        hidden_states = hidden_states.permute(0, 7, 1, 4, 2, 5, 3, 6)
        output = hidden_states.flatten(6, 7).flatten(4, 5).flatten(2, 3)

        return output
```
**EN:** This class models `HeliosTransformer3DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. Helios Transformer 3D model for video generation. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HeliosTransformer3DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：Helios Transformer 3D model for video generation. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 895-897: Top-level configuration / 顶层配置
```python


EntryClass = HeliosTransformer3DModel
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
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.helios`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.communication_op`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.mlp`
- **External / 外部**: `torch`, `torch.nn`, `torch.nn.functional`
- **Stdlib / 标准库**: `math`, `functools`, `typing`
