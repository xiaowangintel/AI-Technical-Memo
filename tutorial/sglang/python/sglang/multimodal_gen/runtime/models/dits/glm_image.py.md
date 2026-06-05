# glm_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/glm_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for glm image within the multimodal runtime. Key symbols include `GlmImageLayerKVCache`, `GlmImageKVCache`, `GlmImageTimestepEmbedding`. / 该模块实现多模态运行时中与 glm image 相关的模型构件。 关键符号包括 `GlmImageLayerKVCache`, `GlmImageKVCache`, `GlmImageTimestepEmbedding`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-52: Imports and module setup / 导入与模块初始化
```python
# Copyright 2025 The CogView team, Tsinghua University & ZhipuAI and The HuggingFace Team. All rights reserved.
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

logger = init_logger(__name__)

_is_cuda = current_platform.is_cuda()
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 55-77: Class `GlmImageLayerKVCache` / 类 `GlmImageLayerKVCache`
```python
class GlmImageLayerKVCache:
    """KV cache for GlmImage model."""

    def __init__(self):
        self.k_cache = None
        self.v_cache = None
        self.mode: Optional[str] = None  # "write", "read", "skip"

    def store(self, k: torch.Tensor, v: torch.Tensor):
        if self.k_cache is None:
            self.k_cache = k
            self.v_cache = v
        else:
            self.k_cache = torch.cat([self.k_cache, k], dim=2)
# ...
    def clear(self):
        self.k_cache = None
        self.v_cache = None
        self.mode = None
```
**EN:** This class models `GlmImageLayerKVCache`. KV cache for GlmImage model. Important methods include `__init__`, `store`, `get`, `clear`.
**CN:** 该类实现 `GlmImageLayerKVCache`。 文档字符串指出：KV cache for GlmImage model. 其中较重要的方法包括 `__init__`, `store`, `get`, `clear`。

### Lines 80-100: Class `GlmImageKVCache` / 类 `GlmImageKVCache`
```python
class GlmImageKVCache:
    """Container for all layers' KV caches."""

    def __init__(self, num_layers: int):
        self.num_layers = num_layers
        self.caches = [GlmImageLayerKVCache() for _ in range(num_layers)]

    def __getitem__(self, layer_idx: int) -> GlmImageLayerKVCache:
        return self.caches[layer_idx]

    def set_mode(self, mode: Optional[str]):
        if mode is not None and mode not in ["write", "read", "skip"]:
            raise ValueError(
                f"Invalid mode: {mode}, must be one of 'write', 'read', 'skip'"
# ...

    def clear(self):
        for cache in self.caches:
            cache.clear()
```
**EN:** This class models `GlmImageKVCache`. Container for all layers' KV caches. Important methods include `__init__`, `__getitem__`, `set_mode`, `clear`.
**CN:** 该类实现 `GlmImageKVCache`。 文档字符串指出：Container for all layers' KV caches. 其中较重要的方法包括 `__init__`, `__getitem__`, `set_mode`, `clear`。

### Lines 103-132: Class `GlmImageTimestepEmbedding` / 类 `GlmImageTimestepEmbedding`
```python
class GlmImageTimestepEmbedding(nn.Module):
    """
    Replacement for diffusers TimestepEmbedding using ReplicatedLinear.
    Structure: linear_1 -> act(silu) -> linear_2
    """

    def __init__(
        self,
        in_channels: int,
        time_embed_dim: int,
        act_fn: str = "silu",
        out_dim: int = None,
    ):
        super().__init__()
# ...
        sample, _ = self.linear_1(sample)
        sample = self.act(sample)
        sample, _ = self.linear_2(sample)
        return sample
```
**EN:** This class models `GlmImageTimestepEmbedding` as a specialization of `nn.Module`. Replacement for diffusers TimestepEmbedding using ReplicatedLinear. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GlmImageTimestepEmbedding`，并继承/扩展 `nn.Module`。 文档字符串指出：Replacement for diffusers TimestepEmbedding using ReplicatedLinear. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 135-164: Class `GlmImageTextProjection` / 类 `GlmImageTextProjection`
```python
class GlmImageTextProjection(nn.Module):
    """
    Replacement for diffusers PixArtAlphaTextProjection using ReplicatedLinear.
    Structure: linear_1 -> act_1 -> linear_2
    """

    def __init__(
        self,
        in_features: int,
        hidden_size: int,
        out_features: int = None,
        act_fn: str = "silu",
    ):
        super().__init__()
# ...
        hidden_states, _ = self.linear_1(caption)
        hidden_states = self.act_1(hidden_states)
        hidden_states, _ = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** This class models `GlmImageTextProjection` as a specialization of `nn.Module`. Replacement for diffusers PixArtAlphaTextProjection using ReplicatedLinear. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GlmImageTextProjection`，并继承/扩展 `nn.Module`。 文档字符串指出：Replacement for diffusers PixArtAlphaTextProjection using ReplicatedLinear. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 167-217: Class `GlmImageCombinedTimestepSizeEmbeddings` / 类 `GlmImageCombinedTimestepSizeEmbeddings`
```python
class GlmImageCombinedTimestepSizeEmbeddings(nn.Module):
    def __init__(
        self,
        embedding_dim: int,
        condition_dim: int,
        pooled_projection_dim: int,
        timesteps_dim: int = 256,
    ):
        super().__init__()

        self.time_proj = Timesteps(
            num_channels=timesteps_dim, flip_sin_to_cos=True, downscale_freq_shift=0
        )
        self.condition_proj = Timesteps(
# ...
        )  # (B, embedding_dim)

        conditioning = timesteps_emb + condition_emb
        return conditioning
```
**EN:** This class models `GlmImageCombinedTimestepSizeEmbeddings` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GlmImageCombinedTimestepSizeEmbeddings`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 220-250: Class `GlmImageImageProjector` / 类 `GlmImageImageProjector`
```python
class GlmImageImageProjector(nn.Module):
    def __init__(
        self,
        in_channels: int = 16,
        hidden_size: int = 2560,
        patch_size: int = 2,
    ):
        super().__init__()
        self.patch_size = patch_size

        self.proj = nn.Linear(in_channels * patch_size**2, hidden_size)

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        batch_size, channel, height, width = hidden_states.shape
# ...
        )
        hidden_states = self.proj(hidden_states)

        return hidden_states
```
**EN:** This class models `GlmImageImageProjector` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GlmImageImageProjector`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 253-307: Class `GlmImageAdaLayerNormZero` / 类 `GlmImageAdaLayerNormZero`
```python
class GlmImageAdaLayerNormZero(nn.Module):
    def __init__(self, embedding_dim: int, dim: int) -> None:
        super().__init__()

        self.norm = nn.LayerNorm(dim, elementwise_affine=False, eps=1e-5)
        self.norm_context = nn.LayerNorm(dim, elementwise_affine=False, eps=1e-5)
        self.linear = ReplicatedLinear(embedding_dim, 12 * dim, bias=True)

    def forward(
        self,
        hidden_states: torch.Tensor,
        encoder_hidden_states: torch.Tensor,
        temb: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
# ...
            c_shift_mlp,
            c_scale_mlp,
            c_gate_mlp,
        )
```
**EN:** This class models `GlmImageAdaLayerNormZero` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GlmImageAdaLayerNormZero`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 310-467: Class `GlmImageAttention` / 类 `GlmImageAttention`
```python
class GlmImageAttention(torch.nn.Module):
    def __init__(
        self,
        query_dim,
        heads,
        dim_head,
        out_dim,
        bias,
        qk_norm,
        elementwise_affine,
        eps,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        prefix: str = "",
        quant_config: QuantizationConfig | None = None,
# ...
        encoder_hidden_states, hidden_states = hidden_states.split(
            [text_seq_length, hidden_states.size(1) - text_seq_length], dim=1
        )
        return hidden_states, encoder_hidden_states
```
**EN:** This class models `GlmImageAttention` as a specialization of `torch.nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GlmImageAttention`，并继承/扩展 `torch.nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 470-574: Class `GlmImageTransformerBlock` / 类 `GlmImageTransformerBlock`
```python
class GlmImageTransformerBlock(nn.Module):
    def __init__(
        self,
        dim: int = 2560,
        num_attention_heads: int = 64,
        attention_head_dim: int = 40,
        time_embed_dim: int = 512,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        prefix: str = "",
        quant_config: QuantizationConfig | None = None,
    ) -> None:
        super().__init__()

        # 1. Attention
# ...
            encoder_hidden_states + ff_output_context * c_gate_mlp.unsqueeze(1)
        )

        return hidden_states, encoder_hidden_states
```
**EN:** This class models `GlmImageTransformerBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GlmImageTransformerBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 577-625: Class `GlmImageRotaryPosEmbed` / 类 `GlmImageRotaryPosEmbed`
```python
class GlmImageRotaryPosEmbed(nn.Module):
    def __init__(self, dim: int, patch_size: int, theta: float = 10000.0) -> None:
        super().__init__()

        self.dim = dim
        self.patch_size = patch_size
        self.theta = theta

    def forward(self, hidden_states: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
        batch_size, num_channels, height, width = hidden_states.shape
        height, width = height // self.patch_size, width // self.patch_size
        device = hidden_states.device

        dim_h, dim_w = self.dim // 2, self.dim // 2
# ...
        # Concatenate along last dimension to get [height, width, dim//2]
        freqs = torch.cat([freqs_h, freqs_w], dim=-1)
        freqs = freqs.reshape(height * width, -1)  # [height * width, dim//2]
        return (freqs.cos(), freqs.sin())
```
**EN:** This class models `GlmImageRotaryPosEmbed` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GlmImageRotaryPosEmbed`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 628-663: Class `GlmImageAdaLayerNormContinuous` / 类 `GlmImageAdaLayerNormContinuous`
```python
class GlmImageAdaLayerNormContinuous(nn.Module):
    """
    GlmImage-only final AdaLN: LN(x) -> Linear(cond) -> chunk -> affine. Matches Megatron: **no activation** before the
    Linear on conditioning embedding.
    """

    def __init__(
        self,
        embedding_dim: int,
        conditioning_embedding_dim: int,
        elementwise_affine: bool = True,
        eps: float = 1e-5,
        bias: bool = True,
        norm_type: str = "layer_norm",
# ...
        emb = self.linear(conditioning_embedding.to(x.dtype))
        scale, shift = torch.chunk(emb, 2, dim=1)
        x = self.norm(x) * (1 + scale)[:, None, :] + shift[:, None, :]
        return x
```
**EN:** This class models `GlmImageAdaLayerNormContinuous` as a specialization of `nn.Module`. GlmImage-only final AdaLN: LN(x) -> Linear(cond) -> chunk -> affine. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GlmImageAdaLayerNormContinuous`，并继承/扩展 `nn.Module`。 文档字符串指出：GlmImage-only final AdaLN: LN(x) -> Linear(cond) -> chunk -> affine. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 666-868: Class `GlmImageTransformer2DModel` / 类 `GlmImageTransformer2DModel`
```python
class GlmImageTransformer2DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    r"""
    Args:
        patch_size (`int`, defaults to `2`):
            The size of the patches to use in the patch embedding layer.
        in_channels (`int`, defaults to `16`):
            The number of channels in the input.
        num_layers (`int`, defaults to `30`):
            The number of layers of Transformer blocks to use.
        attention_head_dim (`int`, defaults to `40`):
            The number of channels in each head.
        num_attention_heads (`int`, defaults to `64`):
            The number of heads to use for multi-head attention.
        out_channels (`int`, defaults to `16`):
# ...
        )
        output = hidden_states.permute(0, 3, 1, 4, 2, 5).flatten(4, 5).flatten(2, 3)

        return output.float()
```
**EN:** This class models `GlmImageTransformer2DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. Args: Important methods include `__init__`, `forward`.
**CN:** 该类实现 `GlmImageTransformer2DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：Args: 其中较重要的方法包括 `__init__`, `forward`。

### Lines 869-873: Top-level configuration / 顶层配置
```python
        # float()
        # reference: https://github.com/zRzRzRzRzRzRzR/diffusers/blob/6cfc83b4abc5b083fef56a18ec4700f48ba3aaba/src/diffusers/pipelines/glm_image/pipeline_glm_image.py#L737


EntryClass = GlmImageTransformer2DModel
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
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.glmimage`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.mlp`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.layers.rotary_embedding`
- **External / 外部**: `torch`, `torch.nn`, `torch.nn.functional`
- **Stdlib / 标准库**: `typing`
