# hunyuanvideo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/hunyuanvideo.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for hunyuanvideo within the multimodal runtime. Key symbols include `MMDoubleStreamBlock`, `MMSingleStreamBlock`, `HunyuanVideoTransformer3DModel`. / 该模块实现多模态运行时中与 hunyuanvideo 相关的模型构件。 关键符号包括 `MMDoubleStreamBlock`, `MMSingleStreamBlock`, `HunyuanVideoTransformer3DModel`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-48: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0

from typing import Any

import numpy as np
import torch
import torch.nn as nn

from sglang.multimodal_gen.configs.models.dits import HunyuanVideoConfig
from sglang.multimodal_gen.configs.sample.teacache import TeaCacheParams
from sglang.multimodal_gen.runtime.distributed.parallel_state import get_sp_world_size
from sglang.multimodal_gen.runtime.layers.attention import (
# ...
from sglang.multimodal_gen.runtime.platforms import (
    AttentionBackendEnum,
    current_platform,
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 51-279: Class `MMDoubleStreamBlock` / 类 `MMDoubleStreamBlock`
```python
class MMDoubleStreamBlock(nn.Module):
    """
    A multimodal DiT block with separate modulation for text and image/video,
    using distributed attention and linear layers.
    """

    def __init__(
        self,
        hidden_size: int,
        num_attention_heads: int,
        mlp_ratio: float,
        dtype: torch.dtype | None = None,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        prefix: str = "",
# ...
        txt_mlp_out = self.txt_mlp(txt_mlp_input)
        txt = self.txt_mlp_residual(txt_mlp_out, txt_mlp_gate, txt_residual)

        return img, txt
```
**EN:** This class models `MMDoubleStreamBlock` as a specialization of `nn.Module`. A multimodal DiT block with separate modulation for text and image/video, Important methods include `__init__`, `forward`.
**CN:** 该类实现 `MMDoubleStreamBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A multimodal DiT block with separate modulation for text and image/video, 其中较重要的方法包括 `__init__`, `forward`。

### Lines 282-420: Class `MMSingleStreamBlock` / 类 `MMSingleStreamBlock`
```python
class MMSingleStreamBlock(nn.Module):
    """
    A DiT block with parallel linear layers using distributed attention
    and tensor parallelism.
    """

    def __init__(
        self,
        hidden_size: int,
        num_attention_heads: int,
        mlp_ratio: float = 4.0,
        dtype: torch.dtype | None = None,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        prefix: str = "",
# ...
        output, _ = self.linear2(combined)

        # Apply residual connection with gating using fused operation
        return self.output_residual(output, mod_gate, x)
```
**EN:** This class models `MMSingleStreamBlock` as a specialization of `nn.Module`. A DiT block with parallel linear layers using distributed attention Important methods include `__init__`, `forward`.
**CN:** 该类实现 `MMSingleStreamBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A DiT block with parallel linear layers using distributed attention 其中较重要的方法包括 `__init__`, `forward`。

### Lines 423-793: Class `HunyuanVideoTransformer3DModel` / 类 `HunyuanVideoTransformer3DModel`
```python
class HunyuanVideoTransformer3DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    """
    HunyuanVideo Transformer backbone adapted for distributed training.

    This implementation uses distributed attention and linear layers for efficient
    parallel processing across multiple GPUs.

    Based on the architecture from:
    - Flux.1: https://github.com/black-forest-labs/flux
    - MMDiT: http://arxiv.org/abs/2403.03206
    """

    # PY: we make the input args the same as HF config

# ...
        return not should_calc

    def retrieve_cached_states(self, hidden_states: torch.Tensor) -> torch.Tensor:
        return hidden_states + self.previous_residual
```
**EN:** This class models `HunyuanVideoTransformer3DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. HunyuanVideo Transformer backbone adapted for distributed training. Important methods include `__init__`, `forward`, `maybe_cache_states`, `should_skip_forward_for_cached_states`.
**CN:** 该类实现 `HunyuanVideoTransformer3DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：HunyuanVideo Transformer backbone adapted for distributed training. 其中较重要的方法包括 `__init__`, `forward`, `maybe_cache_states`, `should_skip_forward_for_cached_states`。

### Lines 796-867: Class `SingleTokenRefiner` / 类 `SingleTokenRefiner`
```python
class SingleTokenRefiner(nn.Module):
    """
    A token refiner that processes text embeddings with attention to improve
    their representation for cross-attention with image features.
    """

    def __init__(
        self,
        in_channels,
        hidden_size,
        num_attention_heads,
        depth=2,
        qkv_bias=True,
        dtype=None,
# ...
        # Process through refiner blocks
        for block in self.refiner_blocks:
            x = block(x, c)
        return x
```
**EN:** This class models `SingleTokenRefiner` as a specialization of `nn.Module`. A token refiner that processes text embeddings with attention to improve Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SingleTokenRefiner`，并继承/扩展 `nn.Module`。 文档字符串指出：A token refiner that processes text embeddings with attention to improve 其中较重要的方法包括 `__init__`, `forward`。

### Lines 870-966: Class `IndividualTokenRefinerBlock` / 类 `IndividualTokenRefinerBlock`
```python
class IndividualTokenRefinerBlock(nn.Module):
    """
    A transformer block for refining individual tokens with self-attention.
    """

    def __init__(
        self,
        hidden_size,
        num_attention_heads,
        mlp_ratio=4.0,
        qkv_bias=True,
        dtype=None,
        prefix: str = "",
    ) -> None:
# ...
        mlp_out = self.mlp(self.norm2(x))
        x = x + mlp_out * gate_mlp.unsqueeze(1)

        return x
```
**EN:** This class models `IndividualTokenRefinerBlock` as a specialization of `nn.Module`. A transformer block for refining individual tokens with self-attention. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `IndividualTokenRefinerBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A transformer block for refining individual tokens with self-attention. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 969-1008: Class `FinalLayer` / 类 `FinalLayer`
```python
class FinalLayer(nn.Module):
    """
    The final layer of DiT that projects features to pixel space.
    """

    def __init__(
        self, hidden_size, patch_size, out_channels, dtype=None, prefix: str = ""
    ) -> None:
        super().__init__()

        # Normalization
        self.norm_final = nn.LayerNorm(
            hidden_size, eps=1e-6, elementwise_affine=False, dtype=dtype
        )
# ...
        scale, shift = self.adaLN_modulation(c).chunk(2, dim=-1)
        x = self.norm_final(x) * (1.0 + scale.unsqueeze(1)) + shift.unsqueeze(1)
        x, _ = self.linear(x)
        return x
```
**EN:** This class models `FinalLayer` as a specialization of `nn.Module`. The final layer of DiT that projects features to pixel space. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `FinalLayer`，并继承/扩展 `nn.Module`。 文档字符串指出：The final layer of DiT that projects features to pixel space. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 1009-1011: Top-level configuration / 顶层配置
```python


EntryClass = HunyuanVideoTransformer3DModel
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
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits`, `sglang.multimodal_gen.configs.sample.teacache`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.elementwise`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.mlp`
- **External / 外部**: `numpy`, `torch`, `torch.nn`
- **Stdlib / 标准库**: `typing`
