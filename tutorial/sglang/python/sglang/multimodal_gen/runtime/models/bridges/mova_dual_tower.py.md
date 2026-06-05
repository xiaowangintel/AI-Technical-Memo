# mova_dual_tower.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/bridges/mova_dual_tower.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for mova dual tower within the multimodal runtime. Key symbols include `compute_rope_cos_sin`, `PerFrameAttentionPooling`, `CrossModalInteractionController`. / 该模块实现多模态运行时中与 mova dual tower 相关的模型构件。 关键符号包括 `compute_rope_cos_sin`, `PerFrameAttentionPooling`, `CrossModalInteractionController`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
# Copied and adapted from: mossVG/mova/diffusion/models/interactionv2.py


from typing import Any, Dict, List, Optional, Tuple

import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange

from sglang.multimodal_gen.configs.models.bridges.mova_dual_tower import (
    MOVADualTowerConfig,
)
# ...
from sglang.multimodal_gen.runtime.models.dits.base import CachableDiT
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 38-84: Function `compute_rope_cos_sin` / 函数 `compute_rope_cos_sin`
```python
@torch.no_grad()
def compute_rope_cos_sin(
    position_ids: torch.Tensor,
    head_dim: int,
    base: float = 10000.0,
    device: Optional[torch.device] = None,
    dtype: Optional[torch.dtype] = None,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """
    Compute RoPE cos/sin embeddings for given position IDs.

    This is a functional implementation that doesn't require storing buffers,
    making it compatible with FSDP meta device initialization.

# ...
    cos = emb.cos().to(dtype=dtype)
    sin = emb.sin().to(dtype=dtype)

    return cos, sin
```
**EN:** This function drives `compute_rope_cos_sin` with inputs such as `position_ids`, `head_dim`, `base`, `device`. Compute RoPE cos/sin embeddings for given position IDs.
**CN:** 这个函数负责 `compute_rope_cos_sin`，主要处理 `position_ids`, `head_dim`, `base`, `device` 等输入。 文档字符串说明：Compute RoPE cos/sin embeddings for given position IDs.

### Lines 87-135: Class `PerFrameAttentionPooling` / 类 `PerFrameAttentionPooling`
```python
class PerFrameAttentionPooling(nn.Module):
    """Per-frame multi-head attention pooling.

    Flattens the input sequence [B, L, D] and grid size (T, H, W).
    Performs single-query attention pooling on the H*W tokens for each time frame.
    Output shape: [B, T, D].
    """

    def __init__(self, dim: int, num_heads: int, eps: float = 1e-6):
        super().__init__()
        assert dim % num_heads == 0, "dim must be divisible by num_heads"
        self.dim = dim
        self.num_heads = num_heads

# ...

        pooled = pooled_bt_d.view(B, T, D)
        pooled = self.layernorm(pooled)
        return pooled
```
**EN:** This class models `PerFrameAttentionPooling` as a specialization of `nn.Module`. Per-frame multi-head attention pooling. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `PerFrameAttentionPooling`，并继承/扩展 `nn.Module`。 文档字符串指出：Per-frame multi-head attention pooling. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 138-187: Class `CrossModalInteractionController` / 类 `CrossModalInteractionController`
```python
class CrossModalInteractionController:
    """Strategy class to control dual-tower interaction.

    Manages the interaction mapping between Visual DiT (e.g., 30 layers)
    and Audio DiT (e.g., 30 layers).
    """

    def __init__(self, visual_layers: int = 30, audio_layers: int = 30):
        self.visual_layers = visual_layers
        self.audio_layers = audio_layers
        self.min_layers = min(visual_layers, audio_layers)

    def get_interaction_layers(
        self, strategy: str = "shallow_focus"
# ...
        """Determines if the specified layer needs to interact."""
        if direction not in interaction_mapping:
            return False
        return any(src == layer_idx for src, _ in interaction_mapping[direction])
```
**EN:** This class models `CrossModalInteractionController`. Strategy class to control dual-tower interaction. Important methods include `__init__`, `get_interaction_layers`, `should_interact`.
**CN:** 该类实现 `CrossModalInteractionController`。 文档字符串指出：Strategy class to control dual-tower interaction. 其中较重要的方法包括 `__init__`, `get_interaction_layers`, `should_interact`。

### Lines 190-299: Class `ConditionalCrossAttention` / 类 `ConditionalCrossAttention`
```python
class ConditionalCrossAttention(nn.Module):
    """
    Cross-modal attention for dual-tower bridge with Tensor Parallel support.

    This module handles attention between video and audio hidden states,
    which have different sequence lengths.
    """

    def __init__(self, dim: int, kv_dim: int, num_heads: int, eps: float = 1e-6):
        super().__init__()
        self.q_dim = dim
        self.kv_dim = kv_dim
        self.num_heads = num_heads
        self.head_dim = self.q_dim // num_heads
# ...
        x = self.attn(q, k, v)
        x = rearrange(x, "b l h d -> b l (h d)")
        x, _ = self.o(x)
        return x
```
**EN:** This class models `ConditionalCrossAttention` as a specialization of `nn.Module`. Cross-modal attention for dual-tower bridge with Tensor Parallel support. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ConditionalCrossAttention`，并继承/扩展 `nn.Module`。 文档字符串指出：Cross-modal attention for dual-tower bridge with Tensor Parallel support. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 302-351: Class `AdaLayerNorm` / 类 `AdaLayerNorm`
```python
class AdaLayerNorm(nn.Module):
    """
    Norm layer modified to incorporate timestep embeddings.
    """

    def __init__(
        self,
        embedding_dim: int,
        num_embeddings: Optional[int] = None,
        output_dim: Optional[int] = None,
        norm_elementwise_affine: bool = False,
        norm_eps: float = 1e-5,
        chunk_dim: int = 0,
    ):
# ...
            scale, shift = temb.chunk(2, dim=0)

        x = self.norm(x) * (1 + scale) + shift
        return x
```
**EN:** This class models `AdaLayerNorm` as a specialization of `nn.Module`. Norm layer modified to incorporate timestep embeddings. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `AdaLayerNorm`，并继承/扩展 `nn.Module`。 文档字符串指出：Norm layer modified to incorporate timestep embeddings. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 354-397: Class `ConditionalCrossAttentionBlock` / 类 `ConditionalCrossAttentionBlock`
```python
class ConditionalCrossAttentionBlock(nn.Module):
    """A wrapper block for ConditionalCrossAttention that applies LayerNorm to the condition input y."""

    def __init__(
        self,
        dim: int,
        kv_dim: int,
        num_heads: int,
        eps: float = 1e-6,
        pooled_adaln: bool = False,
    ):
        super().__init__()
        self.y_norm = nn.LayerNorm(kv_dim, eps=eps)
        self.inner = ConditionalCrossAttention(
# ...
                ).permute(0, 2, 1)
            x = self.adaln(x, temb=pooled_y)
        y = self.y_norm(y)
        return self.inner(x=x, y=y, x_freqs=x_freqs, y_freqs=y_freqs)
```
**EN:** This class models `ConditionalCrossAttentionBlock` as a specialization of `nn.Module`. A wrapper block for ConditionalCrossAttention that applies LayerNorm to the condition input y. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ConditionalCrossAttentionBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A wrapper block for ConditionalCrossAttention that applies LayerNorm to the condition input y. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 400-673: Class `DualTowerConditionalBridge` / 类 `DualTowerConditionalBridge`
```python
class DualTowerConditionalBridge(
    CachableDiT,
    LayerwiseOffloadableModuleMixin,
):
    """Dual-tower conditional bridge module v2 (SGLang optimized version).

    Implements the correct architecture:
    1. Audio latents -> Audio DiT -> Audio hidden states [B, L, 1536].
    2. Visual latents -> Visual DiT -> Visual hidden states [B, L, 5120].
    3. Cross-attention interaction between the hidden states of the two DiTs.
    """

    layerwise_offload_dit_group_enabled = False

# ...
            video_grid_size=video_grid_size,
        )

        return visual_conditioned, audio_conditioned
```
**EN:** This class models `DualTowerConditionalBridge` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. Dual-tower conditional bridge module v2 (SGLang optimized version). Important methods include `__init__`, `build_aligned_freqs`, `should_interact`, `apply_conditional_control`.
**CN:** 该类实现 `DualTowerConditionalBridge`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：Dual-tower conditional bridge module v2 (SGLang optimized version). 其中较重要的方法包括 `__init__`, `build_aligned_freqs`, `should_interact`, `apply_conditional_control`。

### Lines 674-676: Top-level configuration / 顶层配置
```python


EntryClass = DualTowerConditionalBridge
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.bridges.mova_dual_tower`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.rotary_embedding`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`, `sglang.multimodal_gen.runtime.models.dits.base`
- **External / 外部**: `torch`, `torch.nn`, `torch.nn.functional`, `einops`
- **Stdlib / 标准库**: `typing`
