# sana.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/sana.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for sana within the multimodal runtime. Key symbols include `SanaCombinedTimestepSizeEmbeddings`, `SanaAdaLayerNormSingle`, `SanaModulatedNorm`. / 该模块实现多模态运行时中与 sana 相关的模型构件。 关键符号包括 `SanaCombinedTimestepSizeEmbeddings`, `SanaAdaLayerNormSingle`, `SanaModulatedNorm`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0

import torch
import torch.nn as nn
import torch.nn.functional as F
from diffusers.models.embeddings import PixArtAlphaTextProjection, TimestepEmbedding

from sglang.multimodal_gen.configs.models.dits.sana import SanaConfig
from sglang.multimodal_gen.runtime.layers.layernorm import RMSNorm
from sglang.multimodal_gen.runtime.layers.visual_embedding import Timesteps
from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload import (
    LayerwiseOffloadableModuleMixin,
)
from sglang.multimodal_gen.runtime.models.dits.base import CachableDiT
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 20-35: Class `SanaCombinedTimestepSizeEmbeddings` / 类 `SanaCombinedTimestepSizeEmbeddings`
```python
class SanaCombinedTimestepSizeEmbeddings(nn.Module):
    def __init__(self, embedding_dim):
        super().__init__()
        self.time_proj = Timesteps(
            num_channels=256, flip_sin_to_cos=True, downscale_freq_shift=0
        )
        self.timestep_embedder = TimestepEmbedding(
            in_channels=256, time_embed_dim=embedding_dim
        )

    def forward(self, timestep, hidden_dtype=None):
        timesteps_proj = self.time_proj(timestep)
        if hidden_dtype is not None:
            timesteps_proj = timesteps_proj.to(dtype=hidden_dtype)
        timesteps_emb = self.timestep_embedder(timesteps_proj)
        return timesteps_emb
```
**EN:** This class models `SanaCombinedTimestepSizeEmbeddings` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SanaCombinedTimestepSizeEmbeddings`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 38-48: Class `SanaAdaLayerNormSingle` / 类 `SanaAdaLayerNormSingle`
```python
class SanaAdaLayerNormSingle(nn.Module):
    def __init__(self, embedding_dim):
        super().__init__()
        self.emb = SanaCombinedTimestepSizeEmbeddings(embedding_dim)
        self.silu = nn.SiLU()
        self.linear = nn.Linear(embedding_dim, 6 * embedding_dim, bias=True)

    def forward(self, timestep, hidden_dtype=None):
        embedded_timestep = self.emb(timestep, hidden_dtype=hidden_dtype)
        out = self.linear(self.silu(embedded_timestep))
        return out, embedded_timestep
```
**EN:** This class models `SanaAdaLayerNormSingle` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SanaAdaLayerNormSingle`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 51-60: Class `SanaModulatedNorm` / 类 `SanaModulatedNorm`
```python
class SanaModulatedNorm(nn.Module):
    def __init__(self, dim, eps=1e-6):
        super().__init__()
        self.norm = nn.LayerNorm(dim, elementwise_affine=False, eps=eps)

    def forward(self, x, temb, scale_shift_table):
        x = self.norm(x)
        shift, scale = (scale_shift_table[None] + temb[:, None]).chunk(2, dim=1)
        x = x * (1 + scale) + shift
        return x
```
**EN:** This class models `SanaModulatedNorm` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SanaModulatedNorm`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 63-88: Class `GLUMBConv` / 类 `GLUMBConv`
```python
class GLUMBConv(nn.Module):
    """Gated Linear Unit with Multi-Branch Convolution."""

    def __init__(self, in_channels, out_channels, expand_ratio=2.5):
        super().__init__()
        hidden_channels = int(expand_ratio * in_channels)
        self.nonlinearity = nn.SiLU()
        self.conv_inverted = nn.Conv2d(in_channels, hidden_channels * 2, 1, 1, 0)
        self.conv_depth = nn.Conv2d(
            hidden_channels * 2,
            hidden_channels * 2,
            3,
            1,
            1,
# ...
        hidden_states, gate = torch.chunk(hidden_states, 2, dim=1)
        hidden_states = hidden_states * self.nonlinearity(gate)
        hidden_states = self.conv_point(hidden_states)
        return hidden_states
```
**EN:** This class models `GLUMBConv` as a specialization of `nn.Module`. Gated Linear Unit with Multi-Branch Convolution. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GLUMBConv`，并继承/扩展 `nn.Module`。 文档字符串指出：Gated Linear Unit with Multi-Branch Convolution. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 91-129: Class `SanaLinearAttention` / 类 `SanaLinearAttention`
```python
class SanaLinearAttention(nn.Module):
    """Linear attention with O(N*D^2) complexity instead of O(N^2*D)."""

    def __init__(self, query_dim, num_heads, head_dim, bias=False):
        super().__init__()
        inner_dim = num_heads * head_dim
        self.num_heads = num_heads
        self.head_dim = head_dim

        self.to_q = nn.Linear(query_dim, inner_dim, bias=bias)
        self.to_k = nn.Linear(query_dim, inner_dim, bias=bias)
        self.to_v = nn.Linear(query_dim, inner_dim, bias=bias)
        self.to_out = nn.ModuleList(
            [nn.Linear(inner_dim, query_dim, bias=True), nn.Identity()]
# ...

        hidden_states = hidden_states.transpose(1, 2).reshape(B, S, -1)
        hidden_states = self.to_out[0](hidden_states)
        return hidden_states
```
**EN:** This class models `SanaLinearAttention` as a specialization of `nn.Module`. Linear attention with O(N*D^2) complexity instead of O(N^2*D). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SanaLinearAttention`，并继承/扩展 `nn.Module`。 文档字符串指出：Linear attention with O(N*D^2) complexity instead of O(N^2*D). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 132-170: Class `SanaCrossAttention` / 类 `SanaCrossAttention`
```python
class SanaCrossAttention(nn.Module):
    def __init__(self, query_dim, cross_attention_dim, num_heads, head_dim, bias=False):
        super().__init__()
        inner_dim = num_heads * head_dim
        self.num_heads = num_heads
        self.head_dim = head_dim

        self.to_q = nn.Linear(query_dim, inner_dim, bias=bias)
        self.to_k = nn.Linear(cross_attention_dim, inner_dim, bias=bias)
        self.to_v = nn.Linear(cross_attention_dim, inner_dim, bias=bias)
        self.to_out = nn.ModuleList(
            [nn.Linear(inner_dim, query_dim, bias=True), nn.Identity()]
        )

# ...
        )
        hidden_states = hidden_states.transpose(1, 2).reshape(B, S, -1)
        hidden_states = self.to_out[0](hidden_states)
        return hidden_states
```
**EN:** This class models `SanaCrossAttention` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SanaCrossAttention`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 173-241: Class `SanaTransformerBlock` / 类 `SanaTransformerBlock`
```python
class SanaTransformerBlock(nn.Module):
    def __init__(
        self,
        dim,
        num_attention_heads,
        attention_head_dim,
        num_cross_attention_heads,
        cross_attention_head_dim,
        cross_attention_dim,
        mlp_ratio,
        norm_eps,
        attention_bias=False,
    ):
        super().__init__()
# ...
        ff_output = ff_output.flatten(2, 3).permute(0, 2, 1)
        hidden_states = hidden_states + gate_mlp * ff_output

        return hidden_states
```
**EN:** This class models `SanaTransformerBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SanaTransformerBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 244-384: Class `SanaTransformer2DModel` / 类 `SanaTransformer2DModel`
```python
class SanaTransformer2DModel(CachableDiT, LayerwiseOffloadableModuleMixin):

    _fsdp_shard_conditions = [
        lambda n, m: isinstance(m, SanaTransformerBlock),
    ]
    _compile_conditions = [
        lambda n, m: isinstance(m, SanaTransformerBlock),
    ]
    param_names_mapping = SanaConfig().arch_config.param_names_mapping
    reverse_param_names_mapping = {}

    def __init__(self, config: SanaConfig, hf_config=None, **kwargs):
        super().__init__(config, hf_config=hf_config or {}, **kwargs)

# ...
            batch_size, self.out_channels, height, width
        )

        return hidden_states
```
**EN:** This class models `SanaTransformer2DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SanaTransformer2DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 385-387: Top-level configuration / 顶层配置
```python


EntryClass = SanaTransformer2DModel
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Symbol `SanaCombinedTimestepSizeEmbeddings` anchors the module API / 符号 `SanaCombinedTimestepSizeEmbeddings` 构成该模块的核心 API
- Symbol `SanaAdaLayerNormSingle` anchors the module API / 符号 `SanaAdaLayerNormSingle` 构成该模块的核心 API
- Symbol `SanaModulatedNorm` anchors the module API / 符号 `SanaModulatedNorm` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.sana`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.visual_embedding`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`, `sglang.multimodal_gen.runtime.models.dits.base`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`, `torch.nn`, `torch.nn.functional`, `diffusers.models.embeddings`
