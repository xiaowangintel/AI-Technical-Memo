# ltx_2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/ltx_2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for ltx 2 within the multimodal runtime. Key symbols include `adaln_embedding_coefficient`, `_ltx2_is_perturbed`, `_ltx2_build_batched_perturbation_states`. / 该模块实现多模态运行时中与 ltx 2 相关的模型构件。 关键符号包括 `adaln_embedding_coefficient`, `_ltx2_is_perturbed`, `_ltx2_build_batched_perturbation_states`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from LTX-2 and WanVideo implementations.
#
# SPDX-License-Identifier: Apache-2.0

from __future__ import annotations

from typing import Any, Optional, Tuple, Union

import torch
import torch.nn as nn
import torch.nn.functional as F

from sglang.multimodal_gen.configs.models.dits.ltx_2 import LTX2ArchConfig, LTX2Config
from sglang.multimodal_gen.runtime.distributed import (
# ...
logger = init_logger(__name__)

ADALN_NUM_BASE_PARAMS = 6
ADALN_NUM_CROSS_ATTN_PARAMS = 3
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 47-50: Function `adaln_embedding_coefficient` / 函数 `adaln_embedding_coefficient`
```python
def adaln_embedding_coefficient(cross_attention_adaln: bool) -> int:
    return ADALN_NUM_BASE_PARAMS + (
        ADALN_NUM_CROSS_ATTN_PARAMS if cross_attention_adaln else 0
    )
```
**EN:** This function drives `adaln_embedding_coefficient` with inputs such as `cross_attention_adaln`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `adaln_embedding_coefficient`，主要处理 `cross_attention_adaln` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 53-63: Function `_ltx2_is_perturbed` / 函数 `_ltx2_is_perturbed`
```python
def _ltx2_is_perturbed(
    perturbation_config: dict[str, object],
    key: str,
    block_idx: int,
) -> bool:
    value = perturbation_config.get(key)
    if value is None:
        return False
    if key.endswith("_blocks"):
        return block_idx in value
    return bool(value)
```
**EN:** This function drives `_ltx2_is_perturbed` with inputs such as `perturbation_config`, `key`, `block_idx`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_ltx2_is_perturbed`，主要处理 `perturbation_config`, `key`, `block_idx` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 66-97: Function `_ltx2_build_batched_perturbation_states` / 函数 `_ltx2_build_batched_perturbation_states`
```python
def _ltx2_build_batched_perturbation_states(
    perturbation_configs: tuple[dict[str, object], ...],
    key: str,
    block_indices: tuple[int, ...],
    values: torch.Tensor,
) -> dict[int, tuple[torch.Tensor | None, bool]]:
    mask_cache: dict[tuple[int, ...], torch.Tensor] = {}
    states: dict[int, tuple[torch.Tensor | None, bool]] = {}
    for block_idx in block_indices:
        keep_values = []
        any_perturbed = False
        all_perturbed = True
        for config in perturbation_configs:
            perturbed = _ltx2_is_perturbed(config, key, block_idx)
# ...
                ).view(len(keep_values), *([1] * (values.ndim - 1)))
                mask_cache[cache_key] = mask
            states[block_idx] = (mask, False)
    return states
```
**EN:** This function drives `_ltx2_build_batched_perturbation_states` with inputs such as `perturbation_configs`, `key`, `block_indices`, `values`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_ltx2_build_batched_perturbation_states`，主要处理 `perturbation_configs`, `key`, `block_indices`, `values` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 100-106: Function `apply_interleaved_rotary_emb` / 函数 `apply_interleaved_rotary_emb`
```python
def apply_interleaved_rotary_emb(
    x: torch.Tensor, freqs: Tuple[torch.Tensor, torch.Tensor]
) -> torch.Tensor:
    cos, sin = freqs
    x_real, x_imag = x.unflatten(2, (-1, 2)).unbind(-1)
    x_rotated = torch.stack([-x_imag, x_real], dim=-1).flatten(2)
    return x * cos + x_rotated * sin
```
**EN:** This function drives `apply_interleaved_rotary_emb` with inputs such as `x`, `freqs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `apply_interleaved_rotary_emb`，主要处理 `x`, `freqs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 109-162: Function `apply_split_rotary_emb` / 函数 `apply_split_rotary_emb`
```python
def apply_split_rotary_emb(
    x: torch.Tensor, freqs: Tuple[torch.Tensor, torch.Tensor]
) -> torch.Tensor:
    cos, sin = freqs
    if (
        x.ndim == 3
        and cos.ndim == 4
        and sin.ndim == 4
        and x.dtype == torch.bfloat16
        and cos.dtype == torch.bfloat16
        and sin.dtype == torch.bfloat16
        and x.is_cuda
        and x.is_contiguous()
        and cos.is_cuda
# ...
    out = out.reshape(*out.shape[:-2], last)
    if needs_reshape:
        out = out.swapaxes(1, 2).reshape(b, t, -1)
    return out.to(dtype=x_dtype)
```
**EN:** This function drives `apply_split_rotary_emb` with inputs such as `x`, `freqs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `apply_split_rotary_emb`，主要处理 `x`, `freqs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 170-393: Class `LTX2AudioVideoRotaryPosEmbed` / 类 `LTX2AudioVideoRotaryPosEmbed`
```python
class LTX2AudioVideoRotaryPosEmbed(nn.Module):
    def __init__(
        self,
        dim: int,
        patch_size: int = 1,
        patch_size_t: int = 1,
        base_num_frames: int = 20,
        base_height: int = 2048,
        base_width: int = 2048,
        sampling_rate: int = 16000,
        hop_length: int = 160,
        scale_factors: Tuple[int, ...] = (8, 32, 32),
        theta: float = 10000.0,
        causal_offset: int = 1,
# ...
            cos_freqs = torch.swapaxes(cos_freq, 1, 2)
            sin_freqs = torch.swapaxes(sin_freq, 1, 2)

        return cos_freqs.to(dtype=out_dtype), sin_freqs.to(dtype=out_dtype)
```
**EN:** This class models `LTX2AudioVideoRotaryPosEmbed` as a specialization of `nn.Module`. Important methods include `__init__`, `prepare_video_coords`, `prepare_audio_coords`, `prepare_coords`.
**CN:** 该类实现 `LTX2AudioVideoRotaryPosEmbed`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `prepare_video_coords`, `prepare_audio_coords`, `prepare_coords`。

### Lines 396-397: Function `rms_norm` / 函数 `rms_norm`
```python
def rms_norm(x: torch.Tensor, eps: float) -> torch.Tensor:
    return F.rms_norm(x, normalized_shape=(x.shape[-1],), eps=eps)
```
**EN:** This function drives `rms_norm` with inputs such as `x`, `eps`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `rms_norm`，主要处理 `x`, `eps` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 400-430: Class `LTX2TextProjection` / 类 `LTX2TextProjection`
```python
class LTX2TextProjection(nn.Module):
    def __init__(
        self,
        in_features: int,
        hidden_size: int,
        out_features: int | None = None,
        act_fn: str = "gelu_tanh",
    ) -> None:
        super().__init__()
        if out_features is None:
            out_features = hidden_size

        self.linear_1 = ColumnParallelLinear(
            in_features, hidden_size, bias=True, gather_output=True
# ...
        hidden_states, _ = self.linear_1(caption)
        hidden_states = self.act_1(hidden_states)
        hidden_states, _ = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** This class models `LTX2TextProjection` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2TextProjection`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 433-447: Class `LTX2TimestepEmbedder` / 类 `LTX2TimestepEmbedder`
```python
class LTX2TimestepEmbedder(nn.Module):
    def __init__(self, embedding_dim: int, in_channels: int = 256) -> None:
        super().__init__()
        self.linear_1 = ColumnParallelLinear(
            in_channels, embedding_dim, bias=True, gather_output=True
        )
        self.linear_2 = ColumnParallelLinear(
            embedding_dim, embedding_dim, bias=True, gather_output=True
        )

    def forward(self, t_emb: torch.Tensor) -> torch.Tensor:
        x, _ = self.linear_1(t_emb)
        x = F.silu(x)
        x, _ = self.linear_2(x)
        return x
```
**EN:** This class models `LTX2TimestepEmbedder` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2TimestepEmbedder`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 450-462: Class `LTX2PixArtAlphaCombinedTimestepSizeEmbeddings` / 类 `LTX2PixArtAlphaCombinedTimestepSizeEmbeddings`
```python
class LTX2PixArtAlphaCombinedTimestepSizeEmbeddings(nn.Module):
    def __init__(self, embedding_dim: int) -> None:
        super().__init__()
        self.timestep_embedder = LTX2TimestepEmbedder(embedding_dim, in_channels=256)

    def forward(
        self, timestep: torch.Tensor, hidden_dtype: torch.dtype | None = None
    ) -> torch.Tensor:
        t = timestep.reshape(-1).to(dtype=torch.float32)
        t_emb = timestep_embedding(t, dim=256, max_period=10000, dtype=torch.float32)
        if hidden_dtype is not None:
            t_emb = t_emb.to(dtype=hidden_dtype)
        return self.timestep_embedder(t_emb)
```
**EN:** This class models `LTX2PixArtAlphaCombinedTimestepSizeEmbeddings` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2PixArtAlphaCombinedTimestepSizeEmbeddings`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 465-484: Class `LTX2AdaLayerNormSingle` / 类 `LTX2AdaLayerNormSingle`
```python
class LTX2AdaLayerNormSingle(nn.Module):
    def __init__(self, embedding_dim: int, embedding_coefficient: int = 6) -> None:
        super().__init__()
        self.emb = LTX2PixArtAlphaCombinedTimestepSizeEmbeddings(embedding_dim)
        self.silu = nn.SiLU()
        self.linear = ColumnParallelLinear(
            embedding_dim,
            embedding_coefficient * embedding_dim,
            bias=True,
            gather_output=True,
        )

    def forward(
        self, timestep: torch.Tensor, hidden_dtype: torch.dtype | None = None
# ...
            dtype=self.linear.weight.dtype
        )
        out, _ = self.linear(self.silu(embedded_timestep))
        return out, embedded_timestep
```
**EN:** This class models `LTX2AdaLayerNormSingle` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2AdaLayerNormSingle`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 487-520: Class `LTX2TPRMSNormAcrossHeads` / 类 `LTX2TPRMSNormAcrossHeads`
```python
class LTX2TPRMSNormAcrossHeads(nn.Module):
    def __init__(
        self, full_hidden_size: int, local_hidden_size: int, eps: float
    ) -> None:
        super().__init__()
        self.full_hidden_size = full_hidden_size
        self.local_hidden_size = local_hidden_size
        self.eps = eps
        self.weight = nn.Parameter(torch.ones(local_hidden_size))

        tp_rank = get_tp_rank()

        def _weight_loader(param: torch.Tensor, loaded_weight: torch.Tensor) -> None:
            shard = loaded_weight.narrow(
# ...

        inv_rms_fp32 = torch.rsqrt(var + self.eps)
        y = (x.float() * inv_rms_fp32).to(dtype=orig_dtype)
        return y * self.weight.to(dtype=orig_dtype)
```
**EN:** This class models `LTX2TPRMSNormAcrossHeads` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2TPRMSNormAcrossHeads`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 523-782: Class `LTX2Attention` / 类 `LTX2Attention`
```python
class LTX2Attention(nn.Module):
    def __init__(
        self,
        query_dim: int,
        context_dim: int | None = None,
        heads: int = 8,
        dim_head: int = 64,
        norm_eps: float = 1e-6,
        qk_norm: bool = True,
        use_local_attention: bool = False,
        apply_gated_attention: bool = False,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        prefix: str = "",
        quant_config: QuantizationConfig | None = None,
# ...
            start = tp_rank * local_d
            end = start + local_d
            return cos[:, :, start:end], sin[:, :, start:end]
        raise ValueError(f"Unexpected RoPE tensor rank: {cos.ndim}. Expected 3 or 4.")
```
**EN:** This class models `LTX2Attention` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`, `_slice_rope_for_tp`.
**CN:** 该类实现 `LTX2Attention`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`, `_slice_rope_for_tp`。

### Lines 785-814: Class `LTX2FeedForward` / 类 `LTX2FeedForward`
```python
class LTX2FeedForward(nn.Module):
    def __init__(
        self,
        dim: int,
        dim_out: int | None = None,
        mult: int = 4,
        quant_config: QuantizationConfig | None = None,
    ) -> None:
        super().__init__()
        if dim_out is None:
            dim_out = dim
        inner_dim = int(dim * mult)

        self.proj_in = ColumnParallelLinear(
# ...
        x, _ = self.proj_in(x)
        x = self.act(x)
        x, _ = self.proj_out(x)
        return x
```
**EN:** This class models `LTX2FeedForward` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2FeedForward`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 817-1237: Class `LTX2TransformerBlock` / 类 `LTX2TransformerBlock`
```python
class LTX2TransformerBlock(nn.Module):
    def __init__(
        self,
        idx: int,
        dim: int,
        num_attention_heads: int,
        attention_head_dim: int,
        cross_attention_dim: int,
        audio_dim: int,
        audio_num_attention_heads: int,
        audio_attention_head_dim: int,
        audio_cross_attention_dim: int,
        qk_norm: bool = True,
        norm_eps: float = 1e-6,
# ...
        )
        audio_ff_output = self.audio_ff(norm_audio_hidden_states)
        audio_hidden_states = audio_hidden_states + audio_ff_output * agate_mlp
        return hidden_states, audio_hidden_states
```
**EN:** This class models `LTX2TransformerBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `get_ada_values`, `forward`.
**CN:** 该类实现 `LTX2TransformerBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `get_ada_values`, `forward`。

### Lines 1240-1971: Class `LTX2VideoTransformer3DModel` / 类 `LTX2VideoTransformer3DModel`
```python
class LTX2VideoTransformer3DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    _fsdp_shard_conditions = LTX2ArchConfig()._fsdp_shard_conditions
    _compile_conditions = LTX2ArchConfig()._compile_conditions
    _supported_attention_backends = LTX2ArchConfig()._supported_attention_backends
    param_names_mapping = LTX2ArchConfig().param_names_mapping
    reverse_param_names_mapping = LTX2ArchConfig().reverse_param_names_mapping
    lora_param_names_mapping = LTX2ArchConfig().lora_param_names_mapping

    @staticmethod
    def _collapse_prompt_timestep(timestep: torch.Tensor) -> torch.Tensor:
        if timestep.ndim <= 1:
            return timestep
        return timestep.amax(dim=tuple(range(1, timestep.ndim)))

# ...
                # simple reshape for 1D patch
                audio_hidden_states = audio_hidden_states.permute(0, 2, 1)  # [B, C, T]

        return hidden_states, audio_hidden_states
```
**EN:** This class models `LTX2VideoTransformer3DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. Important methods include `_collapse_prompt_timestep`, `_scale_timestep_for_adaln`, `_validate_tp_config`, `__init__`.
**CN:** 该类实现 `LTX2VideoTransformer3DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `_collapse_prompt_timestep`, `_scale_timestep_for_adaln`, `_validate_tp_config`, `__init__`。

### Lines 1972-1976: Top-level configuration / 顶层配置
```python


# Backward-compatible alias (older internal name).
LTXModel = LTX2VideoTransformer3DModel
EntryClass = LTX2VideoTransformer3DModel
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.ltx_2`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.communication_op`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.layers.visual_embedding`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`
- **External / 外部**: `__future__`, `torch`, `torch.nn`, `torch.nn.functional`
- **Stdlib / 标准库**: `typing`
