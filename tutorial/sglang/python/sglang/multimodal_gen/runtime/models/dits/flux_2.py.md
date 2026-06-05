# flux_2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/flux_2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for flux 2 within the multimodal runtime. Key symbols include `_get_qkv_projections`, `Flux2SwiGLU`, `Flux2FeedForward`. / 该模块实现多模态运行时中与 flux 2 相关的模型构件。 关键符号包括 `_get_qkv_projections`, `Flux2SwiGLU`, `Flux2FeedForward`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-55: Imports and module setup / 导入与模块初始化
```python
# Copyright 2025 Black Forest Labs, The HuggingFace Team and The InstantX Team. All rights reserved.
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
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)  # pylint: disable=invalid-name
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 58-81: Function `_get_qkv_projections` / 函数 `_get_qkv_projections`
```python
def _get_qkv_projections(
    attn: "Flux2Attention", hidden_states, encoder_hidden_states=None
):
    if attn.use_fused_qkv:
        qkv, _ = attn.to_qkv(hidden_states)
        query, key, value = [t.contiguous() for t in qkv.chunk(3, dim=-1)]
    else:
        query, _ = attn.to_q(hidden_states)
        key, _ = attn.to_k(hidden_states)
        value, _ = attn.to_v(hidden_states)

    encoder_query = encoder_key = encoder_value = None
    if encoder_hidden_states is not None and attn.added_kv_proj_dim is not None:
        if attn.use_fused_added_qkv:
# ...
            encoder_key, _ = attn.add_k_proj(encoder_hidden_states)
            encoder_value, _ = attn.add_v_proj(encoder_hidden_states)

    return query, key, value, encoder_query, encoder_key, encoder_value
```
**EN:** This function drives `_get_qkv_projections` with inputs such as `attn`, `hidden_states`, `encoder_hidden_states`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_get_qkv_projections`，主要处理 `attn`, `hidden_states`, `encoder_hidden_states` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 84-97: Class `Flux2SwiGLU` / 类 `Flux2SwiGLU`
```python
class Flux2SwiGLU(nn.Module):
    """
    Flux 2 uses a SwiGLU-style activation in the transformer feedforward sub-blocks, but with the linear projection
    layer fused into the first linear layer of the FF sub-block. Thus, this module has no trainable parameters.
    """

    def __init__(self):
        super().__init__()
        self.gate_fn = nn.SiLU()

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x1, x2 = x.chunk(2, dim=-1)
        x = self.gate_fn(x1) * x2
        return x
```
**EN:** This class models `Flux2SwiGLU` as a specialization of `nn.Module`. Flux 2 uses a SwiGLU-style activation in the transformer feedforward sub-blocks, but with the linear projection Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Flux2SwiGLU`，并继承/扩展 `nn.Module`。 文档字符串指出：Flux 2 uses a SwiGLU-style activation in the transformer feedforward sub-blocks, but with the linear projection 其中较重要的方法包括 `__init__`, `forward`。

### Lines 100-139: Class `Flux2FeedForward` / 类 `Flux2FeedForward`
```python
class Flux2FeedForward(nn.Module):
    def __init__(
        self,
        dim: int,
        dim_out: Optional[int] = None,
        mult: float = 3.0,
        inner_dim: Optional[int] = None,
        bias: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        if inner_dim is None:
            inner_dim = int(dim * mult)
# ...
        x, _ = self.linear_in(x)
        x = self.act_fn(x)
        x, _ = self.linear_out(x)
        return x
```
**EN:** This class models `Flux2FeedForward` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Flux2FeedForward`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 142-390: Class `Flux2Attention` / 类 `Flux2Attention`
```python
class Flux2Attention(torch.nn.Module, AttentionModuleMixin):
    def __init__(
        self,
        query_dim: int,
        num_heads: int = 8,
        dim_head: int = 64,
        dropout: float = 0.0,
        bias: bool = False,
        added_kv_proj_dim: Optional[int] = None,
        added_proj_bias: Optional[bool] = True,
        out_bias: bool = True,
        eps: float = 1e-5,
        out_dim: int = None,
        elementwise_affine: bool = True,
# ...
        if encoder_hidden_states is not None:
            return hidden_states, encoder_hidden_states
        else:
            return hidden_states
```
**EN:** This class models `Flux2Attention` as a specialization of `torch.nn.Module`, `AttentionModuleMixin`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Flux2Attention`，并继承/扩展 `torch.nn.Module`, `AttentionModuleMixin`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 393-556: Class `Flux2ParallelSelfAttention` / 类 `Flux2ParallelSelfAttention`
```python
class Flux2ParallelSelfAttention(torch.nn.Module, AttentionModuleMixin):
    """
    Flux 2 parallel self-attention for the Flux 2 single-stream transformer blocks.

    This implements a parallel transformer block, where the attention QKV projections are fused to the feedforward (FF)
    input projections, and the attention output projections are fused to the FF output projections. See the [ViT-22B
    paper](https://arxiv.org/abs/2302.05442) for a visual depiction of this type of transformer block.
    """

    # Does not support QKV fusion as the QKV projections are always fused
    _supports_qkv_fusion = False

    def __init__(
        self,
# ...
        hidden_states = torch.cat([hidden_states, mlp_hidden_states], dim=-1)
        hidden_states, _ = self.to_out(hidden_states)

        return hidden_states
```
**EN:** This class models `Flux2ParallelSelfAttention` as a specialization of `torch.nn.Module`, `AttentionModuleMixin`. Flux 2 parallel self-attention for the Flux 2 single-stream transformer blocks. Important methods include `__init__`, `_patch_to_out_weight_loader`, `forward`.
**CN:** 该类实现 `Flux2ParallelSelfAttention`，并继承/扩展 `torch.nn.Module`, `AttentionModuleMixin`。 文档字符串指出：Flux 2 parallel self-attention for the Flux 2 single-stream transformer blocks. 其中较重要的方法包括 `__init__`, `_patch_to_out_weight_loader`, `forward`。

### Lines 559-634: Class `Flux2SingleTransformerBlock` / 类 `Flux2SingleTransformerBlock`
```python
class Flux2SingleTransformerBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        num_attention_heads: int,
        attention_head_dim: int,
        mlp_ratio: float = 3.0,
        eps: float = 1e-6,
        bias: bool = False,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
# ...
            )
            return encoder_hidden_states, hidden_states
        else:
            return hidden_states
```
**EN:** This class models `Flux2SingleTransformerBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Flux2SingleTransformerBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 637-760: Class `Flux2TransformerBlock` / 类 `Flux2TransformerBlock`
```python
class Flux2TransformerBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        num_attention_heads: int,
        attention_head_dim: int,
        mlp_ratio: float = 3.0,
        eps: float = 1e-6,
        bias: bool = False,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
# ...
        if encoder_hidden_states.dtype == torch.float16:
            encoder_hidden_states = encoder_hidden_states.clip(-65504, 65504)

        return encoder_hidden_states, hidden_states
```
**EN:** This class models `Flux2TransformerBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Flux2TransformerBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 763-805: Class `Flux2TimestepGuidanceEmbeddings` / 类 `Flux2TimestepGuidanceEmbeddings`
```python
class Flux2TimestepGuidanceEmbeddings(nn.Module):
    def __init__(
        self,
        in_channels: int = 256,
        embedding_dim: int = 6144,
        bias: bool = False,
        guidance_embeds: bool = True,
    ):
        super().__init__()

        self.time_proj = Timesteps(
            num_channels=in_channels, flip_sin_to_cos=True, downscale_freq_shift=0
        )
        self.timestep_embedder = TimestepEmbedding(
# ...
            time_guidance_emb = timesteps_emb + guidance_emb
            return time_guidance_emb
        else:
            return timesteps_emb
```
**EN:** This class models `Flux2TimestepGuidanceEmbeddings` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Flux2TimestepGuidanceEmbeddings`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 808-830: Class `Flux2Modulation` / 类 `Flux2Modulation`
```python
class Flux2Modulation(nn.Module):
    def __init__(self, dim: int, mod_param_sets: int = 2, bias: bool = False):
        super().__init__()
        self.mod_param_sets = mod_param_sets

        self.linear = ColumnParallelLinear(
            dim, dim * 3 * self.mod_param_sets, bias=bias, gather_output=True
        )
        self.act_fn = nn.SiLU()

    def forward(
        self, temb: torch.Tensor
    ) -> Tuple[Tuple[torch.Tensor, torch.Tensor, torch.Tensor], ...]:
        mod = self.act_fn(temb)
# ...
        # Return tuple of 3-tuples of modulation params shift/scale/gate
        return tuple(
            mod_params[3 * i : 3 * (i + 1)] for i in range(self.mod_param_sets)
        )
```
**EN:** This class models `Flux2Modulation` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Flux2Modulation`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 833-857: Class `Flux2PosEmbed` / 类 `Flux2PosEmbed`
```python
class Flux2PosEmbed(nn.Module):
    def __init__(self, theta: int, axes_dim: List[int]):
        super().__init__()
        self.rope = NDRotaryEmbedding(
            rope_dim_list=axes_dim,
            rope_theta=theta,
            use_real=False,
            repeat_interleave_real=False,
            dtype=(
                torch.float64
                if (
                    current_platform.is_float64_supported()
                    if hasattr(current_platform, "is_float64_supported")
                    else True
# ...
        # TODO: potential error: flux use n_axes = ids.shape[-1]
        # see: https://github.com/huggingface/diffusers/blob/17c0e79dbdf53fb6705e9c09cc1a854b84c39249/src/diffusers/models/transformers/transformer_flux.py#L509
        freqs_cos, freqs_sin = self.rope.forward_uncached(pos=pos)
        return freqs_cos.contiguous().float(), freqs_sin.contiguous().float()
```
**EN:** This class models `Flux2PosEmbed` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Flux2PosEmbed`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 860-1096: Class `Flux2Transformer2DModel` / 类 `Flux2Transformer2DModel`
```python
class Flux2Transformer2DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    """
    The Transformer model introduced in Flux 2.

    Reference: https://blackforestlabs.ai/announcing-black-forest-labs/

    """

    param_names_mapping = FluxConfig().arch_config.param_names_mapping
    scale_shift_swap_params = ("norm_out.linear.weight", "norm_out.linear.bias")
    # FLUX.2 stays closer to the official diffusers output with Torch SDPA.
    # The generic FA path still produces a measurable image-level drift here.
    _supported_attention_backends = {
        AttentionBackendEnum.TORCH_SDPA,
# ...
        hidden_states = self.norm_out(hidden_states, temb)
        output, _ = self.proj_out(hidden_states)

        return output
```
**EN:** This class models `Flux2Transformer2DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. The Transformer model introduced in Flux 2. Important methods include `post_load_weights`, `__init__`, `forward`.
**CN:** 该类实现 `Flux2Transformer2DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：The Transformer model introduced in Flux 2. 其中较重要的方法包括 `post_load_weights`, `__init__`, `forward`。

### Lines 1097-1099: Top-level configuration / 顶层配置
```python


EntryClass = Flux2Transformer2DModel
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Caching strategy / 缓存策略

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.flux`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.layers.quantization.modelopt_quant`, `sglang.multimodal_gen.runtime.layers.rotary_embedding`
- **External / 外部**: `torch`, `torch.nn`, `diffusers.models.attention`, `diffusers.models.embeddings`, `diffusers.models.normalization`
- **Stdlib / 标准库**: `typing`
