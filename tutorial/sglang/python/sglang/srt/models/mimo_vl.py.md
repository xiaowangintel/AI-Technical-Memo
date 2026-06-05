# mimo_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/mimo_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only MiMo vision model: attention + ViT. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module docstring
```python
"""Inference-only MiMo vision model: attention + ViT."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 3-22: Module imports
```python
from __future__ import annotations

from functools import partial
from typing import Optional, Tuple, Type

import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange
from transformers.configuration_utils import PretrainedConfig
from transformers.models.qwen2_5_vl.modeling_qwen2_5_vl import (
    Qwen2_5_VisionRotaryEmbedding,
)

from sglang.srt.layers.attention.vision import VisionAttention
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.quantization import QuantizationConfig
from sglang.srt.models.qwen2_5_vl import Qwen2_5_VisionPatchMerger, Qwen2_5_VLMLP
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import add_prefix
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 25-25: Class `MiMoVLVisionConfig` overview
```python
class MiMoVLVisionConfig(PretrainedConfig):
```
**EN:** Defines `MiMoVLVisionConfig` as a reusable runtime type derived from PretrainedConfig. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoVLVisionConfig`，其继承关系为 PretrainedConfig。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 26-27: Class `MiMoVLVisionConfig` attributes
```python
    model_type = "mimovl"
    base_config_key = "vision_config"
```
**EN:** Defines class-level attributes and metadata that shape how `MiMoVLVisionConfig` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiMoVLVisionConfig` 在运行时的行为。

### Lines 29-76: Method `MiMoVLVisionConfig.__init__`
```python
    def __init__(
        self,
        depth=28,
        hidden_size=1280,
        hidden_act="silu",
        intermediate_size=4608,
        num_heads=32,
        in_channels=3,
        patch_size=16,
        spatial_merge_size=2,
        temporal_patch_size=2,
        tokens_per_second=2,
        window_size=128,
        out_hidden_size=2048,
        fullatt_block_indexes=[7, 15, 23, 31],
        initializer_range=0.02,
        kv_channels=64,
        qk_channels=64,
        num_query_groups=4,
        num_key_value_heads=8,
        vit_window_attn_types=None,
        visual_token_window_size=64,
        **kwargs,
    ):
        super().__init__(**kwargs)

        self.depth = depth
        self.hidden_size = hidden_size
        self.hidden_act = hidden_act
        self.intermediate_size = intermediate_size
        self.num_heads = num_heads
        if num_key_value_heads is None:
            num_key_value_heads = num_heads
        self.num_key_value_heads = num_key_value_heads
        self.in_channels = in_channels
        self.patch_size = patch_size
        self.spatial_merge_size = spatial_merge_size
        self.temporal_patch_size = temporal_patch_size
        self.tokens_per_second = tokens_per_second
        self.window_size = window_size
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(depth=..., hidden_size=..., hidden_act=..., intermediate_size=..., num_heads=..., in_channels=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(depth=..., hidden_size=..., hidden_act=..., intermediate_size=..., num_heads=..., in_channels=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 79-79: Class `MiMoVisionPatchEmbed` overview
```python
class MiMoVisionPatchEmbed(nn.Module):
```
**EN:** Defines `MiMoVisionPatchEmbed` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoVisionPatchEmbed`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 80-101: Method `MiMoVisionPatchEmbed.__init__`
```python
    def __init__(
        self,
        patch_size: int = 16,
        temporal_patch_size: int = 2,
        in_channels: int = 3,
        embed_dim: int = 1536,
    ) -> None:
        super().__init__()
        self.patch_size = patch_size
        self.temporal_patch_size = temporal_patch_size
        self.in_channels = in_channels
        self.embed_dim = embed_dim

        kernel_size = [temporal_patch_size, patch_size, patch_size]
        self.proj = nn.Conv3d(
            in_channels,
            embed_dim,
            kernel_size=kernel_size,
            stride=kernel_size,
            bias=False,
        )
        self.proj_weight_linear_format = None
```
**EN:** This method implements `__init__(patch_size: ...=..., temporal_patch_size: ...=..., in_channels: ...=..., embed_dim: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(patch_size: ...=..., temporal_patch_size: ...=..., in_channels: ...=..., embed_dim: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 103-105: Method `MiMoVisionPatchEmbed.sync_proj_weight_linear_format`
```python
    @torch.no_grad()
    def sync_proj_weight_linear_format(self):
        self.proj_weight_linear_format = self.proj.weight.view(self.embed_dim, -1)
```
**EN:** This method implements `sync_proj_weight_linear_format()` and handles weight mapping, filtering, or loading for this model component Decorators: torch.no_grad().
**CN:** 这个方法实现了 `sync_proj_weight_linear_format()`，其作用是处理该模型组件的权重映射、筛选或加载逻辑 装饰器：torch.no_grad()。

### Lines 107-112: Method `MiMoVisionPatchEmbed.forward`
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        target_dtype = self.proj.weight.dtype
        hidden_states = F.linear(
            hidden_states.to(dtype=target_dtype), self.proj_weight_linear_format
        )
        return hidden_states
```
**EN:** This method implements `forward(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 115-115: Class `MiMoVisionBlock` overview
```python
class MiMoVisionBlock(nn.Module):
```
**EN:** Defines `MiMoVisionBlock` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoVisionBlock`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 116-188: Method `MiMoVisionBlock.__init__`
```python
    def __init__(
        self,
        dim: int,
        intermediate_dim: int,
        num_heads: int,
        hidden_act="silu",
        norm_layer: Type[nn.Module] = None,
        attn_implementation: Optional[str] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        num_dummy_heads: int = 0,
        rms_norm_eps: float = 1e-6,
        use_sink: bool = False,
        window_size: Tuple[int, int] = (-1, -1),
        num_kv_heads: Optional[int] = None,
        head_dim: Optional[int] = None,
        use_data_parallel: bool = False,
    ) -> None:
        super().__init__()
        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm1 = RMSNorm(dim, eps=rms_norm_eps)
        self.norm2 = RMSNorm(dim, eps=rms_norm_eps)
        self.use_data_parallel = use_data_parallel

        if attn_implementation is None:
            softmax_in_single_precision = False
            qkv_backend = None
            flatten_batch = True
        elif attn_implementation == "sdpa":
            softmax_in_single_precision = False
            qkv_backend = "sdpa"
            flatten_batch = True
        elif attn_implementation == "flash_attention_2":
            softmax_in_single_precision = False
            qkv_backend = "triton_attn"
            flatten_batch = True
        elif attn_implementation == "eager":
            softmax_in_single_precision = True
            qkv_backend = "sdpa"
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(dim: ..., intermediate_dim: ..., num_heads: ..., hidden_act=..., norm_layer: ...=..., attn_implementation: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., intermediate_dim: ..., num_heads: ..., hidden_act=..., norm_layer: ...=..., attn_implementation: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 190-223: Method `MiMoVisionBlock.forward`
```python
    def forward(
        self,
        x: torch.Tensor,
        cu_seqlens: torch.Tensor,
        max_seqlen: int,
        position_embeddings: torch.Tensor,
        full_attn: bool = True,
    ) -> torch.Tensor:
        S, B, H = x.shape
        # norm1: flatten to 2D -> [S*B, H], then reshape back
        x2d = x.reshape(-1, H)
        hidden_states = self.norm1(x2d).reshape(S, B, H)

        # Attention expects [B, S, H]
        hidden_states = rearrange(hidden_states, "s b h -> b s h")
        attn = self.attn(
            hidden_states,
            cu_seqlens=cu_seqlens,
            max_seqlen=max_seqlen,
            position_embeddings=position_embeddings,
            full_attn=full_attn,
        )
        attn = rearrange(attn, "b s h -> s b h")

        # norm2 with fused residual-add: also 2D
        attn2d = attn.reshape(-1, H)
        x_norm_2d, x_after_add_2d = self.norm2(x2d, residual=attn2d)
        x_norm = x_norm_2d.reshape(S, B, H)
        x_after_add = x_after_add_2d.reshape(S, B, H)

        # MLP and final residual
        mlp_out = self.mlp(x_norm)
        x = x_after_add + mlp_out
        return x
```
**EN:** This method implements `forward(x: ..., cu_seqlens: ..., max_seqlen: ..., position_embeddings: ..., full_attn: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ..., cu_seqlens: ..., max_seqlen: ..., position_embeddings: ..., full_attn: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 226-226: Class `MiMoVisionTransformer` overview
```python
class MiMoVisionTransformer(nn.Module):
```
**EN:** Defines `MiMoVisionTransformer` as a reusable runtime type derived from nn.Module. The class groups 10 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoVisionTransformer`，其继承关系为 nn.Module。这个类组织了 10 个方法，用于实现模型相关行为。

### Lines 227-309: Method `MiMoVisionTransformer.__init__`
```python
    def __init__(
        self,
        vision_config: MiMoVLVisionConfig,
        norm_eps: float = 1e-6,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.server_args = get_global_server_args()
        self.vit_window_attn_types = vision_config.vit_window_attn_types
        patch_size: int = vision_config.patch_size
        temporal_patch_size: int = vision_config.temporal_patch_size
        spatial_merge_size: int = vision_config.spatial_merge_size
        self.spatial_merge_size = spatial_merge_size
        self.spatial_merge_unit: int = spatial_merge_size * spatial_merge_size
        in_channels: int = vision_config.in_channels
        hidden_size: int = vision_config.hidden_size
        depth: int = vision_config.depth
        num_heads: int = vision_config.num_heads
        num_kv_heads = getattr(vision_config, "num_key_value_heads", None)
        if num_kv_heads is None:
            num_kv_heads = num_heads
        self.num_kv_heads = num_kv_heads
        self.qk_channels = getattr(vision_config, "qk_channels", None)
        self.kv_channels = getattr(vision_config, "kv_channels", None)
        self.fullatt_block_indexes = vision_config.fullatt_block_indexes
        self.window_size = vision_config.window_size
        self.patch_size = vision_config.patch_size
        self.use_data_parallel = self.server_args.mm_enable_dp_encoder
        mlp_hidden_size: int = vision_config.intermediate_size
        self.patch_embed = MiMoVisionPatchEmbed(
            patch_size=patch_size,
            temporal_patch_size=temporal_patch_size,
            in_channels=in_channels,
            embed_dim=hidden_size,
        )
        self.use_sink = getattr(vision_config, "use_sink", False)
        norm_layer = partial(nn.LayerNorm, eps=norm_eps)
        head_dim = (
            self.qk_channels
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(vision_config: ..., norm_eps: ...=..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(vision_config: ..., norm_eps: ...=..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 311-315: Method `MiMoVisionTransformer.apply_index`
```python
    def apply_index(self, tensor: torch.Tensor, index: torch.Tensor):
        tensor = tensor.unflatten(0, (-1, self.spatial_merge_unit))
        tensor = tensor[index]
        tensor = tensor.flatten(0, 1)
        return tensor
```
**EN:** This method implements `apply_index(tensor: ..., index: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoVisionTransformer`.
**CN:** 这个方法实现了 `apply_index(tensor: ..., index: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoVisionTransformer` 内部调用。

### Lines 317-320: Method `MiMoVisionTransformer._post_init`
```python
    def _post_init(self):
        for name, param in self.named_parameters():
            if "bias" in name:
                param.data.zero_()
```
**EN:** This method implements `_post_init()` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `_post_init()`，其作用是执行后续运行时所需的初始化工作。

### Lines 322-343: Method `MiMoVisionTransformer.get_window_index_1d`
```python
    def get_window_index_1d(self, grid_thw, col=True):
        window_index: list = []
        window_index_id = 0
        for grid_t, grid_h, grid_w in grid_thw:
            llm_grid_h, llm_grid_w = (
                grid_h // self.spatial_merge_size,
                grid_w // self.spatial_merge_size,
            )
            index = torch.arange(grid_t * llm_grid_h * llm_grid_w).reshape(
                grid_t, llm_grid_h, llm_grid_w
            )
            if col:
                index_new = index.transpose(1, 2).reshape(-1)
            else:
                index_new = index.reshape(-1)
            window_index.append(index_new + window_index_id)
            window_index_id += (grid_t * llm_grid_h * llm_grid_w).item()
        window_index = torch.cat(
            window_index,
            dim=0,
        )
        return window_index
```
**EN:** This method implements `get_window_index_1d(grid_thw, col=...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_window_index_1d(grid_thw, col=...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 345-347: Method `MiMoVisionTransformer.dtype`
```python
    @property
    def dtype(self) -> torch.dtype:
        return self.patch_embed.proj.weight.dtype
```
**EN:** This method implements `dtype()` and implements a focused helper that supports the surrounding runtime flow inside `MiMoVisionTransformer` Decorators: property.
**CN:** 这个方法实现了 `dtype()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoVisionTransformer` 内部调用 装饰器：property。

### Lines 349-351: Method `MiMoVisionTransformer.device`
```python
    @property
    def device(self) -> torch.device:
        return self.blocks[0].mlp.gate_up_proj.weight.device
```
**EN:** This method implements `device()` and implements a focused helper that supports the surrounding runtime flow inside `MiMoVisionTransformer` Decorators: property.
**CN:** 这个方法实现了 `device()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoVisionTransformer` 内部调用 装饰器：property。

### Lines 353-383: Method `MiMoVisionTransformer.rot_pos_emb`
```python
    def rot_pos_emb(self, grid_thw: torch.Tensor) -> torch.Tensor:
        pos_ids = []
        for i in range(grid_thw.size(0)):
            t, h, w = grid_thw[i].tolist()
            hpos_ids = torch.arange(h).unsqueeze(1).expand(-1, w)

            hpos_ids = hpos_ids.reshape(
                h // self.spatial_merge_size,
                self.spatial_merge_size,
                w // self.spatial_merge_size,
                self.spatial_merge_size,
            )
            hpos_ids = hpos_ids.permute(0, 2, 1, 3)
            hpos_ids = hpos_ids.flatten()

            wpos_ids = torch.arange(w).unsqueeze(0).expand(h, -1)
            wpos_ids = wpos_ids.reshape(
                h // self.spatial_merge_size,
                self.spatial_merge_size,
                w // self.spatial_merge_size,
                self.spatial_merge_size,
            )
            wpos_ids = wpos_ids.permute(0, 2, 1, 3)
            wpos_ids = wpos_ids.flatten()

            pos_ids.append(torch.stack([hpos_ids, wpos_ids], dim=-1).repeat(t, 1))
        pos_ids = torch.cat(pos_ids, dim=0)
        max_grid_size = grid_thw[:, 1:].max()
        rotary_pos_emb_full = self.rotary_pos_emb(max_grid_size)
        rotary_pos_emb = rotary_pos_emb_full[pos_ids].flatten(1)
        return rotary_pos_emb
```
**EN:** This method implements `rot_pos_emb(grid_thw: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoVisionTransformer`.
**CN:** 这个方法实现了 `rot_pos_emb(grid_thw: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoVisionTransformer` 内部调用。

### Lines 385-441: Method `MiMoVisionTransformer._prepare_forward`
```python
    def _prepare_forward(
        self,
        x: torch.Tensor,
        grid_thw: torch.Tensor,
    ):
        # patchify
        x = x.to(device=self.device, dtype=self.dtype)
        x = self.patch_embed(x)
        # compute position embedding
        rotary_pos_emb = self.rot_pos_emb(grid_thw)

        window_index_1d_col = self.get_window_index_1d(grid_thw, col=True).to(
            device=x.device
        )
        reverse_window_index_1d_col = torch.argsort(window_index_1d_col).to(
            device=x.device
        )

        rotary_pos_emb = rotary_pos_emb.to(device=x.device)
        emb = torch.cat((rotary_pos_emb, rotary_pos_emb), dim=-1)

        def get_position_embeddings(emb, x):
            position_embeddings = (emb.cos(), emb.sin())
            position_embeddings = (
                position_embeddings[0].to(x.device),
                position_embeddings[1].to(x.device),
            )
            return position_embeddings

        seqlens = torch.repeat_interleave(
            grid_thw[:, 1] * grid_thw[:, 2], grid_thw[:, 0]
        )
        cu_seqlens = torch.cat(
            [
                torch.tensor([0], device=x.device, dtype=torch.int32),
                seqlens.cumsum(dim=0).to(device=x.device, dtype=torch.int32),
            ]
        )
        max_seqlen = seqlens.max().item()

# ... truncated for brevity ...
```
**EN:** This method implements `_prepare_forward(x: ..., grid_thw: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_prepare_forward(x: ..., grid_thw: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 443-482: Method `MiMoVisionTransformer.run_blocks`
```python
    def run_blocks(
        self,
        x: torch.Tensor,
        row_based_embeddings: Tuple[torch.Tensor, torch.Tensor],
        col_based_embeddings: Tuple[torch.Tensor, torch.Tensor],
        window_index_1d_col: torch.Tensor,
        reverse_window_index_1d_col: torch.Tensor,
        cu_seqlens: torch.Tensor,
        max_seqlen: int,
    ) -> torch.Tensor:
        for layer_num, blk in enumerate(self.blocks):
            window_attn_type = self.vit_window_attn_types[layer_num]

            # window_attn_type = 1: col-based SWA
            if window_attn_type == 1 and (
                layer_num == 0 or self.vit_window_attn_types[layer_num - 1] != 1
            ):
                x = self.apply_index(x, window_index_1d_col)

            if (
                layer_num > 0
                and window_attn_type != 1
                and self.vit_window_attn_types[layer_num - 1] == 1
            ):
                x = self.apply_index(x, reverse_window_index_1d_col)

            position_embeddings = (
                col_based_embeddings if window_attn_type == 1 else row_based_embeddings
            )
            full_attn = layer_num in self.fullatt_block_indexes

            x = blk(
                x,
                cu_seqlens=cu_seqlens,
                max_seqlen=max_seqlen,
                position_embeddings=position_embeddings,
                full_attn=full_attn,
            )
        x = self.merger(x)
        return x
```
**EN:** This method implements `run_blocks(x: ..., row_based_embeddings: ..., col_based_embeddings: ..., window_index_1d_col: ..., reverse_window_index_1d_col: ..., cu_seqlens: ..., ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoVisionTransformer`.
**CN:** 这个方法实现了 `run_blocks(x: ..., row_based_embeddings: ..., col_based_embeddings: ..., window_index_1d_col: ..., reverse_window_index_1d_col: ..., cu_seqlens: ..., ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoVisionTransformer` 内部调用。

### Lines 484-507: Method `MiMoVisionTransformer.forward`
```python
    def forward(
        self,
        x: torch.Tensor,
        grid_thw: torch.Tensor,
    ) -> torch.Tensor:
        (
            x,
            row_based_embeddings,
            col_based_embeddings,
            window_index_1d_col,
            reverse_window_index_1d_col,
            cu_seqlens,
            max_seqlen,
        ) = self._prepare_forward(x, grid_thw)

        return self.run_blocks(
            x,
            row_based_embeddings,
            col_based_embeddings,
            window_index_1d_col,
            reverse_window_index_1d_col,
            cu_seqlens,
            max_seqlen,
        )
```
**EN:** This method implements `forward(x: ..., grid_thw: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ..., grid_thw: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `__future__.annotations`
- `functools.partial`
- `typing.Optional`
- `typing.Tuple`
- `typing.Type`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `einops.rearrange`
- `transformers.configuration_utils.PretrainedConfig`
- `transformers.models.qwen2_5_vl.modeling_qwen2_5_vl.Qwen2_5_VisionRotaryEmbedding`
- `sglang.srt.layers.attention.vision.VisionAttention`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.quantization.QuantizationConfig`
- `sglang.srt.models.qwen2_5_vl.Qwen2_5_VisionPatchMerger`
- `sglang.srt.models.qwen2_5_vl.Qwen2_5_VLMLP`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.add_prefix`
