# flux.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/flux.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for flux within the multimodal runtime. Key symbols include `_fused_gelu_mlp`, `_get_qkv_projections`, `FluxAttention`. / 该模块实现多模态运行时中与 flux 相关的模型构件。 关键符号包括 `_fused_gelu_mlp`, `_get_qkv_projections`, `FluxAttention`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-84: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

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
# ...
    NunchakuAdaLayerNormZeroSingle = None
    _svdq_gemm_w4a4 = None
    _svdq_quantize_w4a4 = None
    _nunchaku_fused_ops_available = False
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 87-183: Function `_fused_gelu_mlp` / 函数 `_fused_gelu_mlp`
```python
def _fused_gelu_mlp(
    x: torch.Tensor,
    fc1,
    fc2,
    pad_size: int = 256,
) -> torch.Tensor:
    """
    Fused GELU MLP matching nunchaku's fused_gelu_mlp kernel path.

    nunchaku's single-block MLP checkpoint is calibrated for the fused path where:
      1. fc1 GEMM + GELU + 0.171875 shift + unsigned re-quantization + fc2.lora_down
         are all done in a single fused kernel call
      2. fc2 GEMM then receives unsigned INT4 activations (act_unsigned=True)

# ...
        act_unsigned=True,
    )

    return output.view(batch_size, seq_len, -1)
```
**EN:** This function drives `_fused_gelu_mlp` with inputs such as `x`, `fc1`, `fc2`, `pad_size`. Fused GELU MLP matching nunchaku's fused_gelu_mlp kernel path.
**CN:** 这个函数负责 `_fused_gelu_mlp`，主要处理 `x`, `fc1`, `fc2`, `pad_size` 等输入。 文档字符串说明：Fused GELU MLP matching nunchaku's fused_gelu_mlp kernel path.

### Lines 186-209: Function `_get_qkv_projections` / 函数 `_get_qkv_projections`
```python
def _get_qkv_projections(
    attn: "FluxAttention", hidden_states, encoder_hidden_states=None
):
    if getattr(attn, "use_fused_qkv", False):
        qkv, _ = attn.to_qkv(hidden_states)
        query, key, value = [x.contiguous() for x in qkv.chunk(3, dim=-1)]
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

### Lines 212-453: Class `FluxAttention` / 类 `FluxAttention`
```python
class FluxAttention(torch.nn.Module, AttentionModuleMixin):
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
        context_pre_only: Optional[bool] = None,
# ...
                x, _ = self.to_out[0](x)
                if len(self.to_out) == 2:
                    x = self.to_out[1](x)
            return x
```
**EN:** This class models `FluxAttention` as a specialization of `torch.nn.Module`, `AttentionModuleMixin`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `FluxAttention`，并继承/扩展 `torch.nn.Module`, `AttentionModuleMixin`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 456-595: Class `FluxSingleTransformerBlock` / 类 `FluxSingleTransformerBlock`
```python
class FluxSingleTransformerBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        num_attention_heads: int,
        attention_head_dim: int,
        mlp_ratio: float = 4.0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.mlp_hidden_dim = int(dim * mlp_ratio)
        self.use_nunchaku_structure = isinstance(quant_config, NunchakuConfig)

# ...
            hidden_states[:, :text_seq_len],
            hidden_states[:, text_seq_len:],
        )
        return encoder_hidden_states, hidden_states
```
**EN:** This class models `FluxSingleTransformerBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `FluxSingleTransformerBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 598-732: Class `FluxTransformerBlock` / 类 `FluxTransformerBlock`
```python
class FluxTransformerBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        num_attention_heads: int,
        attention_head_dim: int,
        qk_norm: str = "rms_norm",
        eps: float = 1e-6,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.norm1 = AdaLayerNormZero(dim)
# ...
        if encoder_hidden_states.dtype == torch.float16:
            encoder_hidden_states = encoder_hidden_states.clip(-65504, 65504)

        return encoder_hidden_states, hidden_states
```
**EN:** This class models `FluxTransformerBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `FluxTransformerBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 735-756: Class `FluxPosEmbed` / 类 `FluxPosEmbed`
```python
class FluxPosEmbed(nn.Module):
    # modified from https://github.com/black-forest-labs/flux/blob/c00d7c60b085fce8058b9df845e036090873f2ce/src/flux/modules/layers.py#L11
    def __init__(self, theta: int, axes_dim: List[int]):
        super().__init__()
        self.rope = NDRotaryEmbedding(
            rope_dim_list=axes_dim,
            rope_theta=theta,
            use_real=False,
            repeat_interleave_real=False,
            dtype=(
                torch.float64
                if current_platform.is_float64_supported()
                else torch.float32
            ),
# ...
        # TODO: potential error: flux use n_axes = ids.shape[-1]
        # see: https://github.com/huggingface/diffusers/blob/17c0e79dbdf53fb6705e9c09cc1a854b84c39249/src/diffusers/models/transformers/transformer_flux.py#L509
        freqs_cos, freqs_sin = self.rope.forward_uncached(pos=pos)
        return freqs_cos.contiguous().float(), freqs_sin.contiguous().float()
```
**EN:** This class models `FluxPosEmbed` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `FluxPosEmbed`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 759-956: Class `FluxTransformer2DModel` / 类 `FluxTransformer2DModel`
```python
class FluxTransformer2DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    """
    The Transformer model introduced in Flux.

    Reference: https://blackforestlabs.ai/announcing-black-forest-labs/
    """

    param_names_mapping = FluxConfig().arch_config.param_names_mapping

    @classmethod
    def get_nunchaku_quant_rules(cls) -> dict[str, list[str]]:
        return {
            "skip": [
                "norm",
# ...

        output, _ = self.proj_out(hidden_states)

        return output
```
**EN:** This class models `FluxTransformer2DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. The Transformer model introduced in Flux. Important methods include `get_nunchaku_quant_rules`, `__init__`, `forward`.
**CN:** 该类实现 `FluxTransformer2DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：The Transformer model introduced in Flux. 其中较重要的方法包括 `get_nunchaku_quant_rules`, `__init__`, `forward`。

### Lines 957-959: Top-level configuration / 顶层配置
```python


EntryClass = FluxTransformer2DModel
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
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.flux`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.mlp`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.layers.quantization.configs.nunchaku_config`, `sglang.multimodal_gen.runtime.layers.rotary_embedding`
- **External / 外部**: `torch`, `torch.nn`, `diffusers.models.attention`, `diffusers.models.modeling_outputs`, `diffusers.models.normalization`, `nunchaku.models.attention`, `nunchaku.models.normalization`, `nunchaku.ops.gemm`
- **Stdlib / 标准库**: `typing`
