# hunyuan3d.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/hunyuan3d.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for hunyuan3d within the multimodal runtime. Key symbols include `_fused_add_gate`, `MixedRowParallelLinear`, `_flux_timestep_embedding`. / 该模块实现多模态运行时中与 hunyuan3d 相关的模型构件。 关键符号包括 `_fused_add_gate`, `MixedRowParallelLinear`, `_flux_timestep_embedding`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-37: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/Tencent-Hunyuan/Hunyuan3D-2
from __future__ import annotations

import math
from dataclasses import dataclass
from typing import List, Optional, Tuple

import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange

from sglang.multimodal_gen.configs.models.dits.hunyuan3d import (
    Hunyuan3DDiTArchConfig,
# ...
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 40-43: Function `_fused_add_gate` / 函数 `_fused_add_gate`
```python
def _fused_add_gate(
    residual: torch.Tensor, x: torch.Tensor, gate: torch.Tensor
) -> torch.Tensor:
    return torch.addcmul(residual, x, gate)
```
**EN:** This function drives `_fused_add_gate` with inputs such as `residual`, `x`, `gate`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_fused_add_gate`，主要处理 `residual`, `x`, `gate` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 46-66: Class `MixedRowParallelLinear` / 类 `MixedRowParallelLinear`
```python
class MixedRowParallelLinear(RowParallelLinear):
    """RowParallel for inputs concatenated from multiple separately-sharded sources."""

    def __init__(self, input_sizes: list[int], output_size: int, **kwargs):
        self.input_sizes = input_sizes
        super().__init__(sum(input_sizes), output_size, **kwargs)

    def weight_loader(self, param: nn.Parameter, loaded_weight: torch.Tensor):
        input_dim = getattr(param, "input_dim", None)
        if input_dim is not None:
            shards = []
            offset = 0
            for sz in self.input_sizes:
                part = loaded_weight.narrow(input_dim, offset, sz)
# ...
                offset += sz
            param.data.copy_(torch.cat(shards, dim=input_dim))
        else:
            param.data.copy_(loaded_weight)
```
**EN:** This class models `MixedRowParallelLinear` as a specialization of `RowParallelLinear`. RowParallel for inputs concatenated from multiple separately-sharded sources. Important methods include `__init__`, `weight_loader`.
**CN:** 该类实现 `MixedRowParallelLinear`，并继承/扩展 `RowParallelLinear`。 文档字符串指出：RowParallel for inputs concatenated from multiple separately-sharded sources. 其中较重要的方法包括 `__init__`, `weight_loader`。

### Lines 69-87: Function `_flux_timestep_embedding` / 函数 `_flux_timestep_embedding`
```python
def _flux_timestep_embedding(
    t: torch.Tensor, dim, max_period=10000, time_factor: float = 1000.0
):
    """Create sinusoidal timestep embeddings for Flux-style model."""
    t = time_factor * t
    half = dim // 2
    freqs = torch.exp(
        -math.log(max_period)
        * torch.arange(start=0, end=half, dtype=torch.float32)
        / half
    ).to(t.device)

    args = t[:, None].float() * freqs[None]
    embedding = torch.cat([torch.cos(args), torch.sin(args)], dim=-1)
    if dim % 2:
        embedding = torch.cat([embedding, torch.zeros_like(embedding[:, :1])], dim=-1)
    if torch.is_floating_point(t):
        embedding = embedding.to(t)
    return embedding
```
**EN:** This function drives `_flux_timestep_embedding` with inputs such as `t`, `dim`, `max_period`, `time_factor`. Create sinusoidal timestep embeddings for Flux-style model.
**CN:** 这个函数负责 `_flux_timestep_embedding`，主要处理 `t`, `dim`, `max_period`, `time_factor` 等输入。 文档字符串说明：Create sinusoidal timestep embeddings for Flux-style model.

### Lines 90-96: Class `_FluxGELU` / 类 `_FluxGELU`
```python
class _FluxGELU(nn.Module):
    def __init__(self, approximate="tanh"):
        super().__init__()
        self.approximate = approximate

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return F.gelu(x, approximate=self.approximate)
```
**EN:** This class models `_FluxGELU` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `_FluxGELU`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 99-107: Class `_FluxMLPEmbedder` / 类 `_FluxMLPEmbedder`
```python
class _FluxMLPEmbedder(nn.Module):
    def __init__(self, in_dim: int, hidden_dim: int):
        super().__init__()
        self.in_layer = nn.Linear(in_dim, hidden_dim, bias=True)
        self.silu = nn.SiLU()
        self.out_layer = nn.Linear(hidden_dim, hidden_dim, bias=True)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.out_layer(self.silu(self.in_layer(x)))
```
**EN:** This class models `_FluxMLPEmbedder` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `_FluxMLPEmbedder`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 110-129: Class `_FluxRMSNorm` / 类 `_FluxRMSNorm`
```python
class _FluxRMSNorm(nn.Module):
    def __init__(self, dim: int):
        super().__init__()
        self.scale = nn.Parameter(torch.ones(dim))
        self.variance_epsilon = 1e-6
        self.hidden_size = dim

    @property
    def weight(self) -> nn.Parameter:
        # Keep the original checkpoint key (`scale`) while exposing the
        # interface expected by the fused QK-norm helper.
        return self.scale

    def forward(self, x: torch.Tensor):
# ...
        rrms = torch.rsqrt(
            torch.mean(x**2, dim=-1, keepdim=True) + self.variance_epsilon
        )
        return (x * rrms).to(dtype=x_dtype) * self.scale
```
**EN:** This class models `_FluxRMSNorm` as a specialization of `nn.Module`. Important methods include `__init__`, `weight`, `forward`.
**CN:** 该类实现 `_FluxRMSNorm`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `weight`, `forward`。

### Lines 132-150: Class `_FluxQKNorm` / 类 `_FluxQKNorm`
```python
class _FluxQKNorm(nn.Module):
    def __init__(self, dim: int):
        super().__init__()
        self.dim = dim
        self.query_norm = _FluxRMSNorm(dim)
        self.key_norm = _FluxRMSNorm(dim)

    def forward(
        self, q: torch.Tensor, k: torch.Tensor, v: torch.Tensor
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        q, k = apply_qk_norm(
            q=q.contiguous(),
            k=k.contiguous(),
            q_norm=self.query_norm,
            k_norm=self.key_norm,
            head_dim=self.dim,
            allow_inplace=True,
        )
        return q.to(v), k.to(v)
```
**EN:** This class models `_FluxQKNorm` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `_FluxQKNorm`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 153-199: Class `_FluxSelfAttention` / 类 `_FluxSelfAttention`
```python
class _FluxSelfAttention(nn.Module):
    def __init__(
        self,
        dim: int,
        num_heads: int = 8,
        qkv_bias: bool = False,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
    ):
        super().__init__()
        tp_size = get_tp_world_size()
        self.num_heads = num_heads
        self.local_num_heads = divide(num_heads, tp_size)
        self.head_dim = dim // num_heads

# ...
        x = self.local_attn(q, k, v)
        x = x.flatten(2)
        x, _ = self.proj(x)
        return x
```
**EN:** This class models `_FluxSelfAttention` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `_FluxSelfAttention`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 202-206: Class `_FluxModulationOut` / 类 `_FluxModulationOut`
```python
@dataclass
class _FluxModulationOut:
    shift: torch.Tensor
    scale: torch.Tensor
    gate: torch.Tensor
```
**EN:** This class models `_FluxModulationOut`.
**CN:** 该类实现 `_FluxModulationOut`。

### Lines 209-225: Class `_FluxModulation` / 类 `_FluxModulation`
```python
class _FluxModulation(nn.Module):
    def __init__(self, dim: int, double: bool):
        super().__init__()
        self.is_double = double
        self.multiplier = 6 if double else 3
        self.lin = nn.Linear(dim, self.multiplier * dim, bias=True)

    def forward(
        self, vec: torch.Tensor
    ) -> Tuple[_FluxModulationOut, Optional[_FluxModulationOut]]:
        out = self.lin(F.silu(vec))[:, None, :]
        out = out.chunk(self.multiplier, dim=-1)

        return (
            _FluxModulationOut(*out[:3]),
            _FluxModulationOut(*out[3:]) if self.is_double else None,
        )
```
**EN:** This class models `_FluxModulation` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `_FluxModulation`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 228-348: Class `_FluxDoubleStreamBlock` / 类 `_FluxDoubleStreamBlock`
```python
class _FluxDoubleStreamBlock(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        mlp_ratio: float,
        qkv_bias: bool = False,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
    ):
        super().__init__()
        mlp_hidden_dim = int(hidden_size * mlp_ratio)
        tp_size = get_tp_world_size()
        self.num_heads = num_heads
        self.local_num_heads = divide(num_heads, tp_size)
# ...
            scale=txt_mod2.scale,
        )
        txt = _fused_add_gate(txt, self.txt_mlp(txt_modulated), txt_mod2.gate)
        return img, txt
```
**EN:** This class models `_FluxDoubleStreamBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `_FluxDoubleStreamBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 351-435: Class `_FluxSingleStreamBlock` / 类 `_FluxSingleStreamBlock`
```python
class _FluxSingleStreamBlock(nn.Module):
    """
    A DiT block with parallel linear layers as described in
    https://arxiv.org/abs/2302.05442 and adapted modulation interface.
    """

    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        mlp_ratio: float = 4.0,
        qk_scale: Optional[float] = None,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
    ):
# ...
        attn = attn.flatten(2)

        output, _ = self.linear2(torch.cat((attn, self.mlp_act(mlp)), 2))
        return _fused_add_gate(x, output, mod.gate)
```
**EN:** This class models `_FluxSingleStreamBlock` as a specialization of `nn.Module`. A DiT block with parallel linear layers as described in Important methods include `__init__`, `forward`.
**CN:** 该类实现 `_FluxSingleStreamBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A DiT block with parallel linear layers as described in 其中较重要的方法包括 `__init__`, `forward`。

### Lines 438-455: Class `_FluxLastLayer` / 类 `_FluxLastLayer`
```python
class _FluxLastLayer(nn.Module):
    def __init__(self, hidden_size: int, patch_size: int, out_channels: int):
        super().__init__()
        self.norm_final = LayerNormScaleShift(
            hidden_size, elementwise_affine=False, eps=1e-6
        )
        self.linear = nn.Linear(
            hidden_size, patch_size * patch_size * out_channels, bias=True
        )
        self.adaLN_modulation = nn.Sequential(
            nn.SiLU(), nn.Linear(hidden_size, 2 * hidden_size, bias=True)
        )

    def forward(self, x: torch.Tensor, vec: torch.Tensor) -> torch.Tensor:
        shift, scale = self.adaLN_modulation(vec).chunk(2, dim=1)
        x = self.norm_final(x, shift=shift[:, None, :], scale=scale[:, None, :])
        x = self.linear(x)
        return x
```
**EN:** This class models `_FluxLastLayer` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `_FluxLastLayer`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 458-612: Class `Hunyuan3D2DiT` / 类 `Hunyuan3D2DiT`
```python
class Hunyuan3D2DiT(CachableDiT, LayerwiseOffloadableModuleMixin):
    """Hunyuan3D DiT model (Flux-style architecture for Hunyuan3D-2.0)."""

    _aliases = ["hy3dgen.shapegen.models.Hunyuan3DDiT"]

    param_names_mapping = Hunyuan3DDiTConfig().param_names_mapping

    @classmethod
    def build_config_from_params(cls, params: dict) -> Hunyuan3DDiTConfig:
        """Build a DiTConfig from YAML-style parameter dict."""
        field_mapping = {
            "num_heads": "num_attention_heads",
            "depth": "num_layers",
            "depth_single_blocks": "num_single_layers",
# ...

        latent = latent[:, cond.shape[1] :, ...]
        latent = self.final_layer(latent, vec)
        return latent
```
**EN:** This class models `Hunyuan3D2DiT` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. Hunyuan3D DiT model (Flux-style architecture for Hunyuan3D-2.0). Important methods include `build_config_from_params`, `__init__`, `forward`.
**CN:** 该类实现 `Hunyuan3D2DiT`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：Hunyuan3D DiT model (Flux-style architecture for Hunyuan3D-2.0). 其中较重要的方法包括 `build_config_from_params`, `__init__`, `forward`。

### Lines 613-621: Imports and module setup / 导入与模块初始化
```python


import copy
import json
import os as _os

from diffusers.models import UNet2DConditionModel
from diffusers.models.attention_processor import Attention as DiffusersAttention
from diffusers.models.transformers.transformer_2d import BasicTransformerBlock
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 624-640: Function `_chunked_feed_forward` / 函数 `_chunked_feed_forward`
```python
def _chunked_feed_forward(
    ff: nn.Module, hidden_states: torch.Tensor, chunk_dim: int, chunk_size: int
):
    """Feed forward with chunking to save memory."""
    if hidden_states.shape[chunk_dim] % chunk_size != 0:
        raise ValueError(
            f"`hidden_states` dimension to be chunked: {hidden_states.shape[chunk_dim]}"
            f"has to be divisible by chunk size: {chunk_size}."
            f" Make sure to set an appropriate `chunk_size` when calling `unet.enable_forward_chunking`."
        )

    num_chunks = hidden_states.shape[chunk_dim] // chunk_size
    ff_output = torch.cat(
        [ff(hid_slice) for hid_slice in hidden_states.chunk(num_chunks, dim=chunk_dim)],
        dim=chunk_dim,
    )
    return ff_output
```
**EN:** This function drives `_chunked_feed_forward` with inputs such as `ff`, `hidden_states`, `chunk_dim`, `chunk_size`. Feed forward with chunking to save memory.
**CN:** 这个函数负责 `_chunked_feed_forward`，主要处理 `ff`, `hidden_states`, `chunk_dim`, `chunk_size` 等输入。 文档字符串说明：Feed forward with chunking to save memory.

### Lines 643-720: Class `SGLangAttentionWrapper` / 类 `SGLangAttentionWrapper`
```python
class SGLangAttentionWrapper(torch.nn.Module):
    """Drop-in replacement for DiffusersAttention that uses sglang's attention backend."""

    _SUPPORTED_BACKENDS = {AttentionBackendEnum.FA, AttentionBackendEnum.TORCH_SDPA}

    def __init__(
        self,
        query_dim: int,
        heads: int = 8,
        dim_head: int = 64,
        dropout: float = 0.0,
        bias: bool = False,
        cross_attention_dim: int | None = None,
        out_bias: bool = True,
# ...

        out = self.to_out[0](out)
        out = self.to_out[1](out)
        return out
```
**EN:** This class models `SGLangAttentionWrapper` as a specialization of `torch.nn.Module`. Drop-in replacement for DiffusersAttention that uses sglang's attention backend. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SGLangAttentionWrapper`，并继承/扩展 `torch.nn.Module`。 文档字符串指出：Drop-in replacement for DiffusersAttention that uses sglang's attention backend. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 723-1013: Class `Basic2p5DTransformerBlock` / 类 `Basic2p5DTransformerBlock`
```python
class Basic2p5DTransformerBlock(torch.nn.Module):
    """2.5D Transformer block with Multiview Attention (MVA) and Reference View Attention (RVA)."""

    def __init__(
        self,
        transformer: BasicTransformerBlock,
        layer_name: str,
        use_ma: bool = True,
        use_ra: bool = True,
        is_turbo: bool = False,
        use_sglang_attn: bool = True,
    ) -> None:
        super().__init__()
        self.transformer = transformer
# ...
        if hidden_states.ndim == 4:
            hidden_states = hidden_states.squeeze(1)

        return hidden_states
```
**EN:** This class models `Basic2p5DTransformerBlock` as a specialization of `torch.nn.Module`. 2.5D Transformer block with Multiview Attention (MVA) and Reference View Attention (RVA). Important methods include `__init__`, `_initialize_attn_weights`, `__getattr__`, `forward`.
**CN:** 该类实现 `Basic2p5DTransformerBlock`，并继承/扩展 `torch.nn.Module`。 文档字符串指出：2.5D Transformer block with Multiview Attention (MVA) and Reference View Attention (RVA). 其中较重要的方法包括 `__init__`, `_initialize_attn_weights`, `__getattr__`, `forward`。

### Lines 1016-1059: Function `compute_voxel_grid_mask` / 函数 `compute_voxel_grid_mask`
```python
@torch.no_grad()
def compute_voxel_grid_mask(position: torch.Tensor, grid_resolution: int = 8):
    """Compute voxel grid mask for position-aware attention."""
    position = position.half()
    B, N, _, H, W = position.shape
    assert H % grid_resolution == 0 and W % grid_resolution == 0

    valid_mask = (position != 1).all(dim=2, keepdim=True)
    valid_mask = valid_mask.expand_as(position)
    position[valid_mask == False] = 0

    position = rearrange(
        position,
        "b n c (num_h grid_h) (num_w grid_w) -> b n num_h num_w c grid_h grid_w",
# ...

    weights = weights < grid_distance

    return weights
```
**EN:** This function drives `compute_voxel_grid_mask` with inputs such as `position`, `grid_resolution`. Compute voxel grid mask for position-aware attention.
**CN:** 这个函数负责 `compute_voxel_grid_mask`，主要处理 `position`, `grid_resolution` 等输入。 文档字符串说明：Compute voxel grid mask for position-aware attention.

### Lines 1062-1074: Function `compute_multi_resolution_mask` / 函数 `compute_multi_resolution_mask`
```python
def compute_multi_resolution_mask(
    position_maps: torch.Tensor, grid_resolutions: List[int] = [32, 16, 8]
) -> dict:
    """Compute multi-resolution position attention masks."""
    position_attn_mask = {}
    with torch.no_grad():
        for grid_resolution in grid_resolutions:
            position_mask = compute_voxel_grid_mask(position_maps, grid_resolution)
            position_mask = rearrange(
                position_mask, "b ni nj li lj -> b (ni li) (nj lj)"
            )
            position_attn_mask[position_mask.shape[1]] = position_mask
    return position_attn_mask
```
**EN:** This function drives `compute_multi_resolution_mask` with inputs such as `position_maps`, `grid_resolutions`. Compute multi-resolution position attention masks.
**CN:** 这个函数负责 `compute_multi_resolution_mask`，主要处理 `position_maps`, `grid_resolutions` 等输入。 文档字符串说明：Compute multi-resolution position attention masks.

### Lines 1077-1112: Function `compute_discrete_voxel_indice` / 函数 `compute_discrete_voxel_indice`
```python
@torch.no_grad()
def compute_discrete_voxel_indice(
    position: torch.Tensor, grid_resolution: int = 8, voxel_resolution: int = 128
):
    """Compute discrete voxel indices for position encoding."""
    position = position.half()
    B, N, _, H, W = position.shape
    assert H % grid_resolution == 0 and W % grid_resolution == 0

    valid_mask = (position != 1).all(dim=2, keepdim=True)
    valid_mask = valid_mask.expand_as(position)
    position[valid_mask == False] = 0

    position = rearrange(
# ...
    grid_position = grid_position.permute(0, 1, 4, 2, 3).clamp(0, 1)
    voxel_indices = grid_position * (voxel_resolution - 1)
    voxel_indices = torch.round(voxel_indices).long()
    return voxel_indices
```
**EN:** This function drives `compute_discrete_voxel_indice` with inputs such as `position`, `grid_resolution`, `voxel_resolution`. Compute discrete voxel indices for position encoding.
**CN:** 这个函数负责 `compute_discrete_voxel_indice`，主要处理 `position`, `grid_resolution`, `voxel_resolution` 等输入。 文档字符串说明：Compute discrete voxel indices for position encoding.

### Lines 1115-1134: Function `compute_multi_resolution_discrete_voxel_indice` / 函数 `compute_multi_resolution_discrete_voxel_indice`
```python
def compute_multi_resolution_discrete_voxel_indice(
    position_maps: torch.Tensor,
    grid_resolutions: List[int] = [64, 32, 16, 8],
    voxel_resolutions: List[int] = [512, 256, 128, 64],
) -> dict:
    """Compute multi-resolution discrete voxel indices."""
    voxel_indices = {}
    with torch.no_grad():
        for grid_resolution, voxel_resolution in zip(
            grid_resolutions, voxel_resolutions
        ):
            voxel_indice = compute_discrete_voxel_indice(
                position_maps, grid_resolution, voxel_resolution
            )
# ...
                "voxel_indices": voxel_indice,
                "voxel_resolution": voxel_resolution,
            }
    return voxel_indices
```
**EN:** This function drives `compute_multi_resolution_discrete_voxel_indice` with inputs such as `position_maps`, `grid_resolutions`, `voxel_resolutions`. Compute multi-resolution discrete voxel indices.
**CN:** 这个函数负责 `compute_multi_resolution_discrete_voxel_indice`，主要处理 `position_maps`, `grid_resolutions`, `voxel_resolutions` 等输入。 文档字符串说明：Compute multi-resolution discrete voxel indices.

### Lines 1137-1449: Class `UNet2p5DConditionModel` / 类 `UNet2p5DConditionModel`
```python
class UNet2p5DConditionModel(torch.nn.Module):
    """2.5D UNet for multi-view texture generation."""

    def __init__(self, unet: UNet2DConditionModel) -> None:
        super().__init__()
        self.unet = unet

        self.use_ma = True
        self.use_ra = True
        self.use_camera_embedding = True
        self.use_dual_stream = True
        self.is_turbo = False

        if self.use_dual_stream:
# ...
            ),
            return_dict=False,
            cross_attention_kwargs=cross_attention_kwargs_,
        )
```
**EN:** This class models `UNet2p5DConditionModel` as a specialization of `torch.nn.Module`. 2.5D UNet for multi-view texture generation. Important methods include `__init__`, `from_pretrained`, `init_condition`, `init_camera_embedding`.
**CN:** 该类实现 `UNet2p5DConditionModel`，并继承/扩展 `torch.nn.Module`。 文档字符串指出：2.5D UNet for multi-view texture generation. 其中较重要的方法包括 `__init__`, `from_pretrained`, `init_condition`, `init_camera_embedding`。

### Lines 1450-1453: Top-level configuration / 顶层配置
```python


# Entry class for model registry
EntryClass = [Hunyuan3D2DiT, UNet2p5DConditionModel]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Model/component loading / 模型/组件加载
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.hunyuan3d`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.mlp`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`
- **External / 外部**: `__future__`, `torch`, `torch.nn`, `torch.nn.functional`, `einops`, `diffusers.models`, `diffusers.models.attention_processor`, `diffusers.models.transformers.transformer_2d`
- **Stdlib / 标准库**: `math`, `dataclasses`, `typing`, `copy`, `json`, `os`
