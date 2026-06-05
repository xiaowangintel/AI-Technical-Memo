# ernie_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/ernie_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for ernie image within the multimodal runtime. Key symbols include `_rope`, `EmbedND3`, `ErnieImageSelfAttention`. / 该模块实现多模态运行时中与 ernie image 相关的模型构件。 关键符号包括 `_rope`, `EmbedND3`, `ErnieImageSelfAttention`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-39: Imports and module setup / 导入与模块初始化
```python
# Copyright 2026 Baidu ERNIE-Image Team and The HuggingFace Team. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# ...
from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload import (
    LayerwiseOffloadableModuleMixin,
)
from sglang.multimodal_gen.runtime.models.dits.base import CachableDiT
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 42-47: Function `_rope` / 函数 `_rope`
```python
def _rope(pos: torch.Tensor, dim: int, theta: int) -> torch.Tensor:
    assert dim % 2 == 0
    scale = torch.arange(0, dim, 2, dtype=torch.float64, device=pos.device) / dim
    omega = 1.0 / (theta**scale)
    out = torch.einsum("...n,d->...nd", pos, omega)  # codespell:ignore nd
    return out.float()
```
**EN:** This function drives `_rope` with inputs such as `pos`, `dim`, `theta`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_rope`，主要处理 `pos`, `dim`, `theta` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 50-65: Class `EmbedND3` / 类 `EmbedND3`
```python
class EmbedND3(nn.Module):
    """3D rotary positional embedding for (temporal/batch_idx, height, width)."""

    def __init__(self, dim: int, theta: int, axes_dim: Tuple[int, int, int]):
        super().__init__()
        self.dim = dim
        self.theta = theta
        self.axes_dim = list(axes_dim)

    def forward(self, ids: torch.Tensor) -> torch.Tensor:
        emb = torch.cat(
            [_rope(ids[..., i], self.axes_dim[i], self.theta) for i in range(3)],
            dim=-1,
        )
        emb = emb.unsqueeze(1).permute(2, 0, 1, 3)
        return torch.stack([emb, emb], dim=-1).reshape(*emb.shape[:-1], -1)
```
**EN:** This class models `EmbedND3` as a specialization of `nn.Module`. 3D rotary positional embedding for (temporal/batch_idx, height, width). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `EmbedND3`，并继承/扩展 `nn.Module`。 文档字符串指出：3D rotary positional embedding for (temporal/batch_idx, height, width). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 68-173: Class `ErnieImageSelfAttention` / 类 `ErnieImageSelfAttention`
```python
class ErnieImageSelfAttention(nn.Module):
    """Self-attention with separate Q/K/V projections and QK LayerNorm.

    Module name hierarchy matches diffusers Attention naming convention:
      self_attention.to_q, self_attention.to_k, self_attention.to_v,
      self_attention.to_out.0, self_attention.norm_q, self_attention.norm_k.

    Supports tensor parallelism: Q/K/V projections use ColumnParallelLinear
    (output dim sharded by heads), output projection uses RowParallelLinear
    (input dim sharded, all-reduce after matmul).
    """

    def __init__(
        self,
# ...
        attn_out = self.attn(q, k, v)
        attn_out = attn_out.reshape(B, S, self.num_local_heads * self.head_dim)
        out, _ = self.to_out[0](attn_out)
        return out
```
**EN:** This class models `ErnieImageSelfAttention` as a specialization of `nn.Module`. Self-attention with separate Q/K/V projections and QK LayerNorm. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ErnieImageSelfAttention`，并继承/扩展 `nn.Module`。 文档字符串指出：Self-attention with separate Q/K/V projections and QK LayerNorm. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 176-204: Class `ErnieImageMLP` / 类 `ErnieImageMLP`
```python
class ErnieImageMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        ffn_hidden_size: int,
        prefix: str = "",
    ):
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [ffn_hidden_size, ffn_hidden_size],
            bias=False,
            gather_output=False,
            prefix=f"{prefix}.gate_up_proj",
# ...
        gate, up = gate_up.chunk(2, dim=-1)
        x = up * F.gelu(gate)
        x, _ = self.linear_fc2(x)
        return x
```
**EN:** This class models `ErnieImageMLP` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ErnieImageMLP`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 207-252: Class `ErnieImageSharedAdaLNBlock` / 类 `ErnieImageSharedAdaLNBlock`
```python
class ErnieImageSharedAdaLNBlock(nn.Module):
    """Single-stream transformer block with externally-computed Shared AdaLN."""

    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        head_dim: int,
        ffn_hidden_size: int,
        eps: float = 1e-6,
        qk_layernorm: bool = True,
        prefix: str = "",
    ):
        super().__init__()
# ...
        x = self.adaLN_mlp_ln(x) * (1 + scale_mlp) + shift_mlp
        x = residual + gate_mlp * self.mlp(x)

        return x
```
**EN:** This class models `ErnieImageSharedAdaLNBlock` as a specialization of `nn.Module`. Single-stream transformer block with externally-computed Shared AdaLN. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ErnieImageSharedAdaLNBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：Single-stream transformer block with externally-computed Shared AdaLN. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 255-267: Function `_apply_rotary_bshd` / 函数 `_apply_rotary_bshd`
```python
def _apply_rotary_bshd(x: torch.Tensor, freqs: torch.Tensor) -> torch.Tensor:
    freqs = freqs.permute(1, 0, 2, 3)
    rot_dim = freqs.shape[-1]
    x_rot, x_pass = x[..., :rot_dim], x[..., rot_dim:]

    cos_ = torch.cos(freqs).to(x.dtype)
    sin_ = torch.sin(freqs).to(x.dtype)

    x1, x2 = x_rot.chunk(2, dim=-1)
    x_rotated = torch.cat((-x2, x1), dim=-1)

    x_rot = x_rot * cos_ + x_rotated * sin_
    return torch.cat((x_rot, x_pass), dim=-1)
```
**EN:** This function drives `_apply_rotary_bshd` with inputs such as `x`, `freqs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_apply_rotary_bshd`，主要处理 `x`, `freqs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 270-475: Class `ErnieImageTransformer2DModel` / 类 `ErnieImageTransformer2DModel`
```python
class ErnieImageTransformer2DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    """ErnieImage DiT: Single-stream transformer with Shared AdaLN."""

    _supports_gradient_checkpointing = True
    _no_split_modules = ["ErnieImageSharedAdaLNBlock"]
    _skip_layerwise_casting_patterns = ["pos_embed", "norm"]

    _fsdp_shard_conditions = ErnieImageDitConfig().arch_config._fsdp_shard_conditions
    _compile_conditions = []
    param_names_mapping = ErnieImageDitConfig().arch_config.param_names_mapping
    reverse_param_names_mapping = {}

    def __init__(
        self,
# ...
        output = output.permute(0, 5, 1, 3, 2, 4).contiguous()
        output = output.view(B, self.out_channels, H, W)

        return output
```
**EN:** This class models `ErnieImageTransformer2DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. ErnieImage DiT: Single-stream transformer with Shared AdaLN. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ErnieImageTransformer2DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：ErnieImage DiT: Single-stream transformer with Shared AdaLN. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 476-478: Top-level configuration / 顶层配置
```python


EntryClass = ErnieImageTransformer2DModel
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Distributed execution / 分布式执行
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.ernie_image`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.attention.layer`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`, `sglang.multimodal_gen.runtime.models.dits.base`
- **External / 外部**: `torch`, `torch.nn`, `torch.nn.functional`, `diffusers.models.embeddings`
- **Stdlib / 标准库**: `typing`
