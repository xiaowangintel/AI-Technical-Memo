# mova_video_dit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/mova_video_dit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for mova video dit within the multimodal runtime. Key symbols include `modulate`, `sinusoidal_embedding_1d`, `precompute_freqs_cis_3d`. / 该模块实现多模态运行时中与 mova video dit 相关的模型构件。 关键符号包括 `modulate`, `sinusoidal_embedding_1d`, `precompute_freqs_cis_3d`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-43: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: mossVG/mova/diffusion/models/wan_video_dit.py
# SPDX-License-Identifier: Apache-2.0
#
# NOTE: This module shares common functions (sinusoidal_embedding_1d, precompute_freqs_cis, etc.)
# with wanvideo.py. These functions are kept here for MOVA-specific model architecture,
# but could be refactored to a common module in the future.

import math
from typing import Any, Tuple

import torch
import torch.nn as nn
from einops import rearrange
from torch.distributed.tensor import DTensor
# ...
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 47-48: Function `modulate` / 函数 `modulate`
```python
def modulate(x: torch.Tensor, shift: torch.Tensor, scale: torch.Tensor):
    return x * (1 + scale) + shift
```
**EN:** This function drives `modulate` with inputs such as `x`, `shift`, `scale`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `modulate`，主要处理 `x`, `shift`, `scale` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 51-62: Function `sinusoidal_embedding_1d` / 函数 `sinusoidal_embedding_1d`
```python
def sinusoidal_embedding_1d(dim, position):
    sinusoid = torch.outer(
        position.type(torch.float64),
        torch.pow(
            10000,
            -torch.arange(dim // 2, dtype=torch.float64, device=position.device).div(
                dim // 2
            ),
        ),
    )
    x = torch.cat([torch.cos(sinusoid), torch.sin(sinusoid)], dim=1)
    return x.to(position.dtype)
```
**EN:** This function drives `sinusoidal_embedding_1d` with inputs such as `dim`, `position`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `sinusoidal_embedding_1d`，主要处理 `dim`, `position` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 65-70: Function `precompute_freqs_cis_3d` / 函数 `precompute_freqs_cis_3d`
```python
def precompute_freqs_cis_3d(dim: int, end: int = 1024, theta: float = 10000.0):
    # 3d rope precompute
    f_freqs_cis = precompute_freqs_cis(dim - 2 * (dim // 3), end, theta)
    h_freqs_cis = precompute_freqs_cis(dim // 3, end, theta)
    w_freqs_cis = precompute_freqs_cis(dim // 3, end, theta)
    return f_freqs_cis, h_freqs_cis, w_freqs_cis
```
**EN:** This function drives `precompute_freqs_cis_3d` with inputs such as `dim`, `end`, `theta`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `precompute_freqs_cis_3d`，主要处理 `dim`, `end`, `theta` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 73-82: Function `precompute_freqs_cis` / 函数 `precompute_freqs_cis`
```python
def precompute_freqs_cis(
    dim: int, end: int = 1024, theta: float = 10000.0, s: float = 1.0
):
    # 1d rope precompute
    # Note: s parameter is used for audio-specific scaling (e.g., tps adjustment)
    freqs = 1.0 / (theta ** (torch.arange(0, dim, 2)[: (dim // 2)].double() / dim))
    pos = torch.arange(end, dtype=torch.float64, device=freqs.device) * s
    freqs = torch.outer(pos, freqs)
    freqs_cis = torch.polar(torch.ones_like(freqs), freqs)  # complex64
    return freqs_cis
```
**EN:** This function drives `precompute_freqs_cis` with inputs such as `dim`, `end`, `theta`, `s`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `precompute_freqs_cis`，主要处理 `dim`, `end`, `theta`, `s` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 85-91: Function `rope_apply` / 函数 `rope_apply`
```python
def rope_apply(x, freqs, num_heads):
    x = rearrange(x, "b s (n d) -> b s n d", n=num_heads)
    x_out = torch.view_as_complex(
        x.to(torch.float64).reshape(x.shape[0], x.shape[1], x.shape[2], -1, 2)
    )
    x_out = torch.view_as_real(x_out * freqs).flatten(2)
    return x_out.to(x.dtype)
```
**EN:** This function drives `rope_apply` with inputs such as `x`, `freqs`, `num_heads`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `rope_apply`，主要处理 `x`, `freqs`, `num_heads` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 94-101: Function `rope_apply_head_dim` / 函数 `rope_apply_head_dim`
```python
def rope_apply_head_dim(x, freqs, head_dim):
    x = rearrange(x, "b s (n d) -> b s n d", d=head_dim)
    x_out = torch.view_as_complex(
        x.to(torch.float64).reshape(x.shape[0], x.shape[1], x.shape[2], -1, 2)
    )
    # print(f"{x_out.shape = }, {freqs.shape = }")
    x_out = torch.view_as_real(x_out * freqs).flatten(2)
    return x_out.to(x.dtype)
```
**EN:** This function drives `rope_apply_head_dim` with inputs such as `x`, `freqs`, `head_dim`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `rope_apply_head_dim`，主要处理 `x`, `freqs`, `head_dim` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 104-197: Class `SelfAttention` / 类 `SelfAttention`
```python
class SelfAttention(nn.Module):
    """
    Self-Attention module for MOVA DiT with Sequence Parallelism support.

    SP is handled at the pipeline level (latents are pre-sharded before DiT forward).
    USPAttention internally handles the all-to-all communication for distributed attention.
    Input x should already be the local shard [B, S_local, D] when SP is enabled.
    """

    def __init__(
        self,
        dim: int,
        num_heads: int,
        eps: float = 1e-6,
# ...
        out = rearrange(out, "b s n d -> b s (n d)")

        out, _ = self.o(out)
        return out
```
**EN:** This class models `SelfAttention` as a specialization of `nn.Module`. Self-Attention module for MOVA DiT with Sequence Parallelism support. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SelfAttention`，并继承/扩展 `nn.Module`。 文档字符串指出：Self-Attention module for MOVA DiT with Sequence Parallelism support. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 200-283: Class `CrossAttention` / 类 `CrossAttention`
```python
class CrossAttention(nn.Module):
    """
    Cross-Attention module for MOVA DiT.

    Cross-attention does NOT require SP communication because:
    - Query comes from the main sequence (already sharded by SP)
    - Key/Value come from context (text embeddings, which are replicated across all ranks)

    Uses LocalAttention instead of USPAttention for efficiency.
    """

    def __init__(
        self,
        dim: int,
# ...
        x = self.attn(q, k, v)
        x = rearrange(x, "b s n d -> b s (n d)")
        x, _ = self.o(x)
        return x
```
**EN:** This class models `CrossAttention` as a specialization of `nn.Module`. Cross-Attention module for MOVA DiT. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `CrossAttention`，并继承/扩展 `nn.Module`。 文档字符串指出：Cross-Attention module for MOVA DiT. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 286-291: Class `MulAdd` / 类 `MulAdd`
```python
class MulAdd(nn.Module):
    def __init__(self):
        super().__init__()

    def forward(self, x, gate, residual):
        return residual + gate * x
```
**EN:** This class models `MulAdd` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `MulAdd`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 294-362: Class `DiTBlock` / 类 `DiTBlock`
```python
class DiTBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        num_heads: int,
        ffn_dim: int,
        eps: float = 1e-6,
        quant_config: QuantizationConfig | None = None,
    ):
        super().__init__()
        self.dim = dim
        self.num_heads = num_heads
        self.ffn_dim = ffn_dim

# ...
        # 4. Feed-forward
        x = self.mlp_residual(self.ffn(input_x), gate_mlp, x)
        x = x.to(orig_dtype)
        return x
```
**EN:** This class models `DiTBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `DiTBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 365-391: Class `Head` / 类 `Head`
```python
class Head(nn.Module):
    def __init__(
        self, dim: int, out_dim: int, patch_size: Tuple[int, int, int], eps: float
    ):
        super().__init__()
        self.dim = dim
        self.patch_size = patch_size
        self.norm = LayerNormScaleShift(
            dim, eps=eps, elementwise_affine=False, dtype=torch.float32
        )
        # Output dim is small for MOVA; replicate to avoid TP shape coupling.
        self.head = ReplicatedLinear(dim, out_dim * math.prod(patch_size))
        self.modulation = nn.Parameter(torch.randn(1, 2, dim) / dim**0.5)

# ...
                self.modulation.to(dtype=t_mod.dtype, device=t_mod.device) + t_mod
            ).chunk(2, dim=1)
            x, _ = self.head(self.norm(x, shift, scale))
        return x
```
**EN:** This class models `Head` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Head`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 394-421: Class `Conv3dLocalIsland` / 类 `Conv3dLocalIsland`
```python
class Conv3dLocalIsland(nn.Conv3d):
    """
    Inherits from Conv3d and overrides the forward method.

    Key behaviors:
    - Parameters are kept as DTensor to maintain optimizer consistency.
    - The forward pass aggregates input, weight, and bias into a Replicate state,
      then performs the convolution locally using to_local().
    - The output is then redistributed as a DTensor (defaults to Replicate,
      but placements can be customized).
    """

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
# ...

            return self._conv_forward(x_local, w_local, b_local)
        else:
            return super().forward(input)
```
**EN:** This class models `Conv3dLocalIsland` as a specialization of `nn.Conv3d`. Inherits from Conv3d and overrides the forward method. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Conv3dLocalIsland`，并继承/扩展 `nn.Conv3d`。 文档字符串指出：Inherits from Conv3d and overrides the forward method. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 424-588: Class `WanModel` / 类 `WanModel`
```python
class WanModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    _fsdp_shard_conditions = MOVAVideoConfig()._fsdp_shard_conditions
    _compile_conditions = MOVAVideoConfig()._compile_conditions
    _supported_attention_backends = MOVAVideoConfig()._supported_attention_backends
    param_names_mapping = MOVAVideoConfig().param_names_mapping
    reverse_param_names_mapping = MOVAVideoConfig().reverse_param_names_mapping
    lora_param_names_mapping = MOVAVideoConfig().lora_param_names_mapping

    def __init__(
        self,
        config: MOVAVideoConfig,
        hf_config: dict[str, Any],
        quant_config: QuantizationConfig | None = None,
    ) -> None:
# ...

        x = self.head(x, t)
        x = self.unpatchify(x, (f, h, w))
        return x
```
**EN:** This class models `WanModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `_init_freqs`, `patchify`, `unpatchify`.
**CN:** 该类实现 `WanModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `_init_freqs`, `patchify`, `unpatchify`。

### Lines 589-591: Top-level configuration / 顶层配置
```python


EntryClass = WanModel
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Pipeline orchestration / 流水线编排
- Caching strategy / 缓存策略

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.mova_video`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.mlp`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`
- **External / 外部**: `torch`, `torch.nn`, `einops`, `torch.distributed.tensor`
- **Stdlib / 标准库**: `math`, `typing`
