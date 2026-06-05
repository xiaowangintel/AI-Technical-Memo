# wanvideo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/wanvideo.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for wanvideo within the multimodal runtime. Key symbols include `WanImageEmbedding`, `WanTimeTextImageEmbedding`, `WanSelfAttention`. / 该模块实现多模态运行时中与 wanvideo 相关的模型构件。 关键符号包括 `WanImageEmbedding`, `WanTimeTextImageEmbedding`, `WanSelfAttention`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-71: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0

import math
from functools import lru_cache
from typing import Any

import torch
import torch.nn as nn

from sglang.multimodal_gen.configs.models.dits import WanVideoConfig
from sglang.multimodal_gen.runtime.distributed import (
    divide,
# ...
_is_cuda = current_platform.is_cuda()

if _use_aiter:
    from aiter.ops.rope import rope_cached_2c_fwd_inplace
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 74-87: Class `WanImageEmbedding` / 类 `WanImageEmbedding`
```python
class WanImageEmbedding(torch.nn.Module):
    def __init__(self, in_features: int, out_features: int):
        super().__init__()

        self.norm1 = FP32LayerNorm(in_features)
        self.ff = MLP(in_features, in_features, out_features, act_type="gelu")
        self.norm2 = FP32LayerNorm(out_features)

    def forward(self, encoder_hidden_states_image: torch.Tensor) -> torch.Tensor:
        dtype = encoder_hidden_states_image.dtype
        hidden_states = self.norm1(encoder_hidden_states_image)
        hidden_states = self.ff(hidden_states)
        hidden_states = self.norm2(hidden_states).to(dtype)
        return hidden_states
```
**EN:** This class models `WanImageEmbedding` as a specialization of `torch.nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanImageEmbedding`，并继承/扩展 `torch.nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 90-129: Class `WanTimeTextImageEmbedding` / 类 `WanTimeTextImageEmbedding`
```python
class WanTimeTextImageEmbedding(nn.Module):
    def __init__(
        self,
        dim: int,
        time_freq_dim: int,
        text_embed_dim: int,
        image_embed_dim: int | None = None,
    ):
        super().__init__()

        self.time_embedder = TimestepEmbedder(
            dim, frequency_embedding_size=time_freq_dim, act_layer="silu"
        )
        self.time_modulation = ModulateProjection(dim, factor=6, act_layer="silu")
# ...
                encoder_hidden_states_image
            )

        return temb, timestep_proj, encoder_hidden_states, encoder_hidden_states_image
```
**EN:** This class models `WanTimeTextImageEmbedding` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanTimeTextImageEmbedding`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 132-208: Class `WanSelfAttention` / 类 `WanSelfAttention`
```python
class WanSelfAttention(nn.Module):
    def __init__(
        self,
        dim: int,
        num_heads: int,
        window_size=(-1, -1),
        qk_norm=True,
        eps=1e-6,
        parallel_attention=False,
        prefix: str = "",
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        is_cross_attention: bool = False,
        quant_config: QuantizationConfig | None = None,
    ) -> None:
# ...
        Args:
            x(Tensor): Shape [B, L, num_heads, C / num_heads]
        """
        pass
```
**EN:** This class models `WanSelfAttention` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanSelfAttention`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 211-245: Class `WanT2VCrossAttention` / 类 `WanT2VCrossAttention`
```python
class WanT2VCrossAttention(WanSelfAttention):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs, is_cross_attention=True)

    def forward(self, x, context, context_lens):
        r"""
        Args:
            x(Tensor): Shape [B, L1, C]
            context(Tensor): Shape [B, L2, C]
            context_lens(Tensor): Shape [B]
        """
        q, _ = self.to_q(x)
        if self.tp_rmsnorm:
            q = tensor_parallel_rms_norm(q, self.norm_q)
# ...
        # output
        x = x.flatten(2)
        x, _ = self.to_out(x)
        return x
```
**EN:** This class models `WanT2VCrossAttention` as a specialization of `WanSelfAttention`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanT2VCrossAttention`，并继承/扩展 `WanSelfAttention`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 248-332: Class `WanI2VCrossAttention` / 类 `WanI2VCrossAttention`
```python
class WanI2VCrossAttention(WanSelfAttention):
    def __init__(
        self,
        dim: int,
        num_heads: int,
        window_size=(-1, -1),
        qk_norm=True,
        eps=1e-6,
        prefix: str = "",
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        quant_config: QuantizationConfig | None = None,
    ) -> None:
        super().__init__(
            dim,
# ...
        img_x = img_x.flatten(2)
        x = x + img_x
        x, _ = self.to_out(x)
        return x
```
**EN:** This class models `WanI2VCrossAttention` as a specialization of `WanSelfAttention`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanI2VCrossAttention`，并继承/扩展 `WanSelfAttention`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 335-612: Class `WanTransformerBlock` / 类 `WanTransformerBlock`
```python
class WanTransformerBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        ffn_dim: int,
        num_heads: int,
        qk_norm: str = "rms_norm_across_heads",
        cross_attn_norm: bool = False,
        eps: float = 1e-6,
        added_kv_proj_dim: int | None = None,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        prefix: str = "",
        attention_type: str = "original",
        sla_topk: float = 0.1,
# ...
        hidden_states = self.mlp_residual(ff_output, c_gate_msa, hidden_states)
        hidden_states = hidden_states.to(orig_dtype)

        return hidden_states
```
**EN:** This class models `WanTransformerBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanTransformerBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 615-856: Class `WanTransformerBlock_VSA` / 类 `WanTransformerBlock_VSA`
```python
class WanTransformerBlock_VSA(nn.Module):
    def __init__(
        self,
        dim: int,
        ffn_dim: int,
        num_heads: int,
        qk_norm: str = "rms_norm_across_heads",
        cross_attn_norm: bool = False,
        eps: float = 1e-6,
        added_kv_proj_dim: int | None = None,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        prefix: str = "",
        quant_config: QuantizationConfig | None = None,
    ):
# ...
        hidden_states = self.mlp_residual(ff_output, c_gate_msa, hidden_states)
        hidden_states = hidden_states.to(orig_dtype)

        return hidden_states
```
**EN:** This class models `WanTransformerBlock_VSA` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanTransformerBlock_VSA`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 859-1251: Class `WanTransformer3DModel` / 类 `WanTransformer3DModel`
```python
class WanTransformer3DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    _fsdp_shard_conditions = WanVideoConfig()._fsdp_shard_conditions
    _compile_conditions = WanVideoConfig()._compile_conditions
    _supported_attention_backends = WanVideoConfig()._supported_attention_backends
    param_names_mapping = WanVideoConfig().param_names_mapping
    reverse_param_names_mapping = WanVideoConfig().reverse_param_names_mapping
    lora_param_names_mapping = WanVideoConfig().lora_param_names_mapping

    def __init__(
        self,
        config: WanVideoConfig,
        hf_config: dict[str, Any],
        quant_config: QuantizationConfig | None = None,
    ) -> None:
# ...
        if not self.is_cfg_negative:
            return hidden_states + self.previous_residual
        else:
            return hidden_states + self.previous_residual_negative
```
**EN:** This class models `WanTransformer3DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `_compute_rope_for_sequence_shard`, `forward`, `maybe_cache_states`.
**CN:** 该类实现 `WanTransformer3DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `_compute_rope_for_sequence_shard`, `forward`, `maybe_cache_states`。

### Lines 1252-1254: Top-level configuration / 顶层配置
```python


EntryClass = WanTransformer3DModel
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
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.elementwise`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.mlp`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`
- **External / 外部**: `torch`, `torch.nn`, `aiter.ops.rope`
- **Stdlib / 标准库**: `math`, `functools`, `typing`
