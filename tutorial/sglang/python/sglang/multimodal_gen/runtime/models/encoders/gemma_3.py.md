# gemma_3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/encoders/gemma_3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for gemma 3 within the multimodal runtime. Key symbols include `get_attention_sliding_window_size`, `Gemma3RMSNorm`, `Gemma3MLP`. / 该模块实现多模态运行时中与 gemma 3 相关的模型构件。 关键符号包括 `get_attention_sliding_window_size`, `Gemma3RMSNorm`, `Gemma3MLP`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Adapted from sglang: python/sglang/srt/models/gemma3_causal.py

import logging
from functools import partial
from typing import Any, Iterable, Optional, Set, Tuple

import torch
from torch import nn

from sglang.multimodal_gen.configs.models.encoders.base import BaseEncoderOutput
from sglang.multimodal_gen.configs.models.encoders.gemma_3 import Gemma3Config
# ...
)
from sglang.multimodal_gen.runtime.utils.common import add_prefix

logger = logging.getLogger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 35-36: Function `get_attention_sliding_window_size` / 函数 `get_attention_sliding_window_size`
```python
def get_attention_sliding_window_size(config):
    return config.sliding_window - 1
```
**EN:** This function drives `get_attention_sliding_window_size` with inputs such as `config`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_attention_sliding_window_size`，主要处理 `config` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 39-54: Class `Gemma3RMSNorm` / 类 `Gemma3RMSNorm`
```python
class Gemma3RMSNorm(nn.Module):
    def __init__(self, dim: int, eps: float = 1e-6):
        super().__init__()
        self.eps = eps
        self.weight = nn.Parameter(torch.zeros(dim))

    def _norm(self, x):
        return x * torch.rsqrt(x.pow(2).mean(-1, keepdim=True) + self.eps)

    def forward(self, x):
        output = self._norm(x.float())
        output = output * (1.0 + self.weight.float())
        return output.type_as(x)

    def extra_repr(self):
        return f"{tuple(self.weight.shape)}, eps={self.eps}"
```
**EN:** This class models `Gemma3RMSNorm` as a specialization of `nn.Module`. Important methods include `__init__`, `_norm`, `forward`, `extra_repr`.
**CN:** 该类实现 `Gemma3RMSNorm`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `_norm`, `forward`, `extra_repr`。

### Lines 57-93: Class `Gemma3MLP` / 类 `Gemma3MLP`
```python
class Gemma3MLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=hidden_size,
            output_sizes=[intermediate_size] * 2,
            bias=False,
# ...
        x, _ = self.gate_up_proj(x)
        x = self.act_fn(x)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This class models `Gemma3MLP` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Gemma3MLP`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 96-98: Function `_rotate_half` / 函数 `_rotate_half`
```python
def _rotate_half(x: torch.Tensor) -> torch.Tensor:
    x1, x2 = x[..., : x.shape[-1] // 2], x[..., x.shape[-1] // 2 :]
    return torch.cat((-x2, x1), dim=-1)
```
**EN:** This function drives `_rotate_half` with inputs such as `x`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_rotate_half`，主要处理 `x` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 101-344: Class `Gemma3Attention` / 类 `Gemma3Attention`
```python
class Gemma3Attention(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: Gemma3Config,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.hidden_size = hidden_size
# ...
        )

        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This class models `Gemma3Attention` as a specialization of `nn.Module`. Important methods include `__init__`, `_apply_rotary_pos_emb`, `forward`.
**CN:** 该类实现 `Gemma3Attention`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `_apply_rotary_pos_emb`, `forward`。

### Lines 347-425: Class `Gemma3DecoderLayer` / 类 `Gemma3DecoderLayer`
```python
class Gemma3DecoderLayer(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: Gemma3Config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.text_config.hidden_size
        self.self_attn = Gemma3Attention(
            layer_id=layer_id,
            config=config,
            hidden_size=self.hidden_size,
# ...
        hidden_states = self.post_feedforward_layernorm(hidden_states)
        hidden_states = residual_mlp + hidden_states

        return hidden_states, None
```
**EN:** This class models `Gemma3DecoderLayer` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Gemma3DecoderLayer`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 428-440: Class `Gemma3TextScaledWordEmbedding` / 类 `Gemma3TextScaledWordEmbedding`
```python
class Gemma3TextScaledWordEmbedding(nn.Embedding):
    def __init__(
        self,
        num_embeddings: int,
        embedding_dim: int,
        padding_idx: int,
        embed_scale: Optional[float] = 1.0,
    ):
        super().__init__(num_embeddings, embedding_dim, padding_idx)
        self.embed_scale = embed_scale

    def forward(self, input_ids: torch.Tensor):
        return super().forward(input_ids) * self.embed_scale
```
**EN:** This class models `Gemma3TextScaledWordEmbedding` as a specialization of `nn.Embedding`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Gemma3TextScaledWordEmbedding`，并继承/扩展 `nn.Embedding`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 446-448: Class `QuickGELU` / 类 `QuickGELU`
```python
class QuickGELU(nn.Module):
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return x * torch.sigmoid(1.702 * x)
```
**EN:** This class models `QuickGELU` as a specialization of `nn.Module`. Important methods include `forward`.
**CN:** 该类实现 `QuickGELU`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `forward`。

### Lines 451-485: Class `SiglipVisionEmbeddings` / 类 `SiglipVisionEmbeddings`
```python
class SiglipVisionEmbeddings(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = config.image_size
        self.patch_size = config.patch_size

        self.patch_embedding = nn.Conv2d(
            in_channels=config.num_channels,
            out_channels=self.embed_dim,
            kernel_size=self.patch_size,
            stride=self.patch_size,
            padding="valid",
# ...
        embeddings = patch_embeds.flatten(2).transpose(1, 2)
        embeddings = embeddings + self.position_embedding(self.position_ids)

        return embeddings
```
**EN:** This class models `SiglipVisionEmbeddings` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SiglipVisionEmbeddings`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 488-515: Class `SiglipMLP` / 类 `SiglipMLP`
```python
class SiglipMLP(nn.Module):
    def __init__(
        self,
        config,
        act_layer: type[nn.Module] = QuickGELU,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.fc1 = ColumnParallelLinear(
            config.hidden_size,
            config.intermediate_size,
            quant_config=quant_config,
            prefix=add_prefix("fc1", prefix),
# ...
        x_parallel, _ = self.fc1(x)
        x_parallel = self.act(x_parallel)
        x, _ = self.fc2(x_parallel)
        return x
```
**EN:** This class models `SiglipMLP` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SiglipMLP`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 518-576: Class `SiglipAttention` / 类 `SiglipAttention`
```python
class SiglipAttention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = hidden_size
        self.num_heads = num_heads
        tp_size = get_tp_world_size()
        self.head_dim = hidden_size // num_heads
        self.num_heads_per_partition = num_heads // tp_size
# ...
        )

        output, _ = self.out_proj(attn_output)
        return output
```
**EN:** This class models `SiglipAttention` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SiglipAttention`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 579-619: Class `SiglipEncoderLayer` / 类 `SiglipEncoderLayer`
```python
class SiglipEncoderLayer(nn.Module):
    def __init__(
        self,
        config,
        act_layer: type[nn.Module] = QuickGELU,
        norm_layer: type[nn.Module] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=config.layer_norm_eps)
        self.layer_norm1 = norm_layer(config.hidden_size)
        self.layer_norm2 = norm_layer(config.hidden_size)
# ...
        hidden_states = self.layer_norm2(hidden_states)
        hidden_states = self.mlp(hidden_states)
        hidden_states = residual + hidden_states
        return hidden_states
```
**EN:** This class models `SiglipEncoderLayer` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SiglipEncoderLayer`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 622-652: Class `SiglipEncoder` / 类 `SiglipEncoder`
```python
class SiglipEncoder(nn.Module):
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        num_hidden_layers = config.num_hidden_layers
        norm_layer = partial(nn.LayerNorm, eps=config.layer_norm_eps)
        self.layers = nn.ModuleList(
            [
                SiglipEncoderLayer(
# ...
        hidden_states = inputs_embeds
        for encoder_layer in self.layers:
            hidden_states = encoder_layer(hidden_states)
        return hidden_states
```
**EN:** This class models `SiglipEncoder` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SiglipEncoder`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 655-681: Class `SiglipVisionTransformer` / 类 `SiglipVisionTransformer`
```python
class SiglipVisionTransformer(nn.Module):
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        embed_dim = config.hidden_size
        self.embeddings = SiglipVisionEmbeddings(config)
        self.encoder = SiglipEncoder(
            config=config,
            quant_config=quant_config,
# ...
        hidden_states = self.embeddings(pixel_values.to(self.device))
        last_hidden_state = self.encoder(inputs_embeds=hidden_states)
        last_hidden_state = self.post_layernorm(last_hidden_state)
        return last_hidden_state
```
**EN:** This class models `SiglipVisionTransformer` as a specialization of `nn.Module`. Important methods include `__init__`, `device`, `forward`.
**CN:** 该类实现 `SiglipVisionTransformer`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `device`, `forward`。

### Lines 684-701: Class `SiglipVisionModel` / 类 `SiglipVisionModel`
```python
class SiglipVisionModel(nn.Module):
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.vision_model = SiglipVisionTransformer(
            config, quant_config, prefix=add_prefix("vision_model", prefix)
        )

    @property
    def device(self) -> torch.device:
        return self.vision_model.device

    def forward(self, pixel_values: torch.Tensor):
        return self.vision_model(pixel_values)
```
**EN:** This class models `SiglipVisionModel` as a specialization of `nn.Module`. Important methods include `__init__`, `device`, `forward`.
**CN:** 该类实现 `SiglipVisionModel`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `device`, `forward`。

### Lines 704-752: Class `Gemma3MultiModalProjector` / 类 `Gemma3MultiModalProjector`
```python
class Gemma3MultiModalProjector(nn.Module):
    """Projector for Gemma3 multimodal."""

    def __init__(self, config: Gemma3Config):
        super().__init__()

        self.mm_input_projection_weight = nn.Parameter(
            torch.zeros(
                config.vision_config.hidden_size, config.text_config.hidden_size
            )
        )

        self.mm_soft_emb_norm = Gemma3RMSNorm(
            config.vision_config.hidden_size, eps=config.vision_config.layer_norm_eps
# ...
            normed_vision_outputs, self.mm_input_projection_weight
        )

        return projected_vision_outputs.type_as(vision_outputs)
```
**EN:** This class models `Gemma3MultiModalProjector` as a specialization of `nn.Module`. Projector for Gemma3 multimodal. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Gemma3MultiModalProjector`，并继承/扩展 `nn.Module`。 文档字符串指出：Projector for Gemma3 multimodal. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 755-937: Class `Gemma3TextModel` / 类 `Gemma3TextModel`
```python
class Gemma3TextModel(nn.Module):
    def __init__(self, config: Gemma3Config):
        super().__init__()
        self.config = config
        # TODO(yinfan.1024) support text encoding model quant later
        self.quant_config = None

        # Use VocabParallelEmbedding
        from sglang.multimodal_gen.runtime.layers.vocab_parallel_embedding import (
            VocabParallelEmbedding,
        )

        self.vocab_size = config.text_config.vocab_size
        self.embed_tokens = VocabParallelEmbedding(
# ...
                weight_loader(param, loaded_weight)

            loaded_params.add(name)
        return loaded_params
```
**EN:** This class models `Gemma3TextModel` as a specialization of `nn.Module`. Important methods include `__init__`, `get_input_embeddings`, `forward`, `load_weights`.
**CN:** 该类实现 `Gemma3TextModel`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `get_input_embeddings`, `forward`, `load_weights`。

### Lines 940-1244: Class `Gemma3ForConditionalGeneration` / 类 `Gemma3ForConditionalGeneration`
```python
class Gemma3ForConditionalGeneration(nn.Module, LayerwiseOffloadableModuleMixin):
    # transformers 5.6.0 flattened SiglipVisionModel, dropping the
    # `vision_model` intermediate wrapper. Our reimpl keeps it, so remap
    # HF source keys back into our nested namespace when transferring weights.
    layerwise_offload_dit_group_enabled = False
    layer_names = ["language_model.layers"]

    param_names_mapping = {
        r"^(vision_tower\.)(embeddings|encoder|post_layernorm|head)\.": r"\1vision_model.\2.",
    }
    reverse_param_names_mapping = {
        r"^(vision_tower\.)vision_model\.(embeddings|encoder|post_layernorm|head)\.": r"\1\2.",
    }

# ...
            sliding_window = getattr(self.config, "sliding_window", None)
        if sliding_window is None:
            return None
        return int(sliding_window) - 1
```
**EN:** This class models `Gemma3ForConditionalGeneration` as a specialization of `nn.Module`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `get_placeholder_mask`, `forward`, `load_weights`.
**CN:** 该类实现 `Gemma3ForConditionalGeneration`，并继承/扩展 `nn.Module`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `get_placeholder_mask`, `forward`, `load_weights`。

### Lines 1245-1247: Top-level configuration / 顶层配置
```python


EntryClass = Gemma3ForConditionalGeneration
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
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.encoders.gemma_3`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.activation`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization`, `sglang.multimodal_gen.runtime.layers.rotary_embedding`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `logging`, `functools`, `typing`
