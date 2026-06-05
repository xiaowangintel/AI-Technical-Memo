# qwen3_omni_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen3_omni_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only Qwen3-VL model compatible with HuggingFace weights. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 15-15: Module docstring
```python
"""Inference-only Qwen3-VL model compatible with HuggingFace weights."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 17-46: Module imports
```python
import math
from typing import Iterable, List, Optional, Tuple

import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
from transformers import PreTrainedModel
from transformers.activations import ACT2FN
from transformers.modeling_outputs import BaseModelOutput

from sglang.srt.configs.qwen3_omni import (
    Qwen3OmniMoeAudioEncoderConfig,
    Qwen3OmniMoeThinkerConfig,
    Qwen3OmniMoeVisionEncoderConfig,
)
from sglang.srt.configs.qwen3_vl import Qwen3VLMoeConfig
from sglang.srt.layers.attention.vision import VisionAttention
from sglang.srt.layers.linear import ColumnParallelLinear, RowParallelLinear
from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.managers.schedule_batch import MultimodalDataItem
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.qwen3_vl import Qwen3VLMoeVisionModel
from sglang.srt.models.qwen3_vl_moe import (
    Qwen3MoeLLMModel,
    Qwen3VLMoeForConditionalGeneration,
    load_fused_expert_weights,
)
from sglang.srt.utils import add_prefix, is_npu, logger
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 49-49: Class `Qwen3OmniMoeAudioEncoderLayer` overview
```python
class Qwen3OmniMoeAudioEncoderLayer(nn.Module):
```
**EN:** Defines `Qwen3OmniMoeAudioEncoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3OmniMoeAudioEncoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 50-85: Method `Qwen3OmniMoeAudioEncoderLayer.__init__`
```python
    def __init__(
        self,
        config: Qwen3OmniMoeAudioEncoderConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        embed_dim = config.d_model
        self.embed_dim = config.d_model
        self.self_attn = VisionAttention(
            embed_dim=embed_dim,
            num_heads=config.encoder_attention_heads,
            projection_size=embed_dim,
            use_qkv_parallel=True,
            proj_bias=True,
            flatten_batch=True,
            quant_config=quant_config,
            prefix=add_prefix("attn", prefix),
        )
        self.self_attn_layer_norm = nn.LayerNorm(self.embed_dim)
        self.dropout = config.dropout
        self.activation_fn = ACT2FN[config.activation_function]
        self.activation_dropout = config.activation_dropout
        self.fc1 = ColumnParallelLinear(
            self.embed_dim,
            config.encoder_ffn_dim,
            bias=True,
            prefix=f"{prefix}.fc1",
        )
        self.fc2 = RowParallelLinear(
            config.encoder_ffn_dim,
            self.embed_dim,
            bias=True,
            prefix=f"{prefix}.fc2",
        )
        self.final_layer_norm = nn.LayerNorm(self.embed_dim)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 87-124: Method `Qwen3OmniMoeAudioEncoderLayer.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        **kwargs,
    ) -> torch.Tensor:
        """
        Args:
            hidden_states (`torch.FloatTensor`): input to the layer of shape `(batch, seq_len, embed_dim)`
            layer_head_mask (`torch.FloatTensor`): mask for attention heads in a given layer of size
                `(encoder_attention_heads,)`.
            output_attentions (`bool`, *optional*):
                Whether or not to return the attentions tensors of all attention layers. See `attentions` under
                returned tensors for more detail.
        """
        residual = hidden_states
        hidden_states = self.self_attn_layer_norm(hidden_states)
        hidden_states = self.self_attn(
            x=hidden_states,
            cu_seqlens=cu_seqlens,
        )
        hidden_states = residual + hidden_states
        residual = hidden_states
        hidden_states = self.final_layer_norm(hidden_states)
        hidden_states, _ = self.fc1(hidden_states)
        hidden_states = self.activation_fn(hidden_states)
        hidden_states, _ = self.fc2(hidden_states)
        hidden_states = residual + hidden_states

        if hidden_states.dtype == torch.float16:
            clamp_value = torch.finfo(hidden_states.dtype).max - 1000
            hidden_states = torch.clamp(
                hidden_states, min=-clamp_value, max=clamp_value
            )

        outputs = (hidden_states,)

        return outputs
```
**EN:** This method implements `forward(hidden_states: ..., cu_seqlens: ..., **kwargs)` and Args:.
**CN:** 这个方法实现了 `forward(hidden_states: ..., cu_seqlens: ..., **kwargs)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 127-127: Class `SinusoidsPositionEmbedding` overview
```python
class SinusoidsPositionEmbedding(nn.Module):
```
**EN:** Defines `SinusoidsPositionEmbedding` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SinusoidsPositionEmbedding`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 128-143: Method `SinusoidsPositionEmbedding.__init__`
```python
    def __init__(self, length, channels, max_timescale=10000):
        super().__init__()
        if channels % 2 != 0:
            raise ValueError("SinusoidsPositionEmbedding needs even channels input")
        log_timescale_increment = np.log(max_timescale) / (channels // 2 - 1)
        inv_timescales = torch.exp(
            -log_timescale_increment * torch.arange(channels // 2).float()
        )
        scaled_time = (
            torch.arange(length)[:, np.newaxis] * inv_timescales[np.newaxis, :]
        )
        self.register_buffer(
            "positional_embedding",
            torch.cat([torch.sin(scaled_time), torch.cos(scaled_time)], dim=1),
            persistent=False,
        )
```
**EN:** This method implements `__init__(length, channels, max_timescale=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(length, channels, max_timescale=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 145-146: Method `SinusoidsPositionEmbedding.forward`
```python
    def forward(self, seqlen: int):
        return self.positional_embedding[:seqlen, :]
```
**EN:** This method implements `forward(seqlen: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(seqlen: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 149-159: Function `_get_feat_extract_output_lengths`
```python
def _get_feat_extract_output_lengths(input_lengths):
    """
    Computes the output length of the convolutional layers and the output length of the audio encoder
    """

    input_lengths_leave = input_lengths % 100
    feat_lengths = (input_lengths_leave - 1) // 2 + 1
    output_lengths = (
        ((feat_lengths - 1) // 2 + 1 - 1) // 2 + 1 + (input_lengths // 100) * 13
    )
    return output_lengths
```
**EN:** This function implements `_get_feat_extract_output_lengths(input_lengths)` and Computes the output length of the convolutional layers and the output length of the audio encoder.
**CN:** 这个函数实现了 `_get_feat_extract_output_lengths(input_lengths)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 162-162: Class `Qwen3OmniMoeAudioEncoder` overview
```python
class Qwen3OmniMoeAudioEncoder(PreTrainedModel):
```
**EN:** Defines `Qwen3OmniMoeAudioEncoder` as a reusable runtime type derived from PreTrainedModel. The class groups 6 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3OmniMoeAudioEncoder`，其继承关系为 PreTrainedModel。这个类组织了 6 个方法，用于实现模型相关行为。

### Lines 163-163: Class `Qwen3OmniMoeAudioEncoder` attributes
```python
    config: Qwen3OmniMoeAudioEncoderConfig
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3OmniMoeAudioEncoder` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3OmniMoeAudioEncoder` 在运行时的行为。

### Lines 165-208: Method `Qwen3OmniMoeAudioEncoder.__init__`
```python
    def __init__(self, config: Qwen3OmniMoeAudioEncoderConfig):
        super().__init__(config)
        self.dropout = config.dropout

        embed_dim = config.d_model
        self.num_mel_bins = config.num_mel_bins
        self.max_source_positions = config.max_source_positions
        self.embed_scale = math.sqrt(embed_dim) if config.scale_embedding else 1.0
        self.n_window = config.n_window
        self.positional_embedding = SinusoidsPositionEmbedding(
            self.max_source_positions, embed_dim
        )
        self.layers = nn.ModuleList(
            [
                Qwen3OmniMoeAudioEncoderLayer(config)
                for _ in range(config.encoder_layers)
            ]
        )
        self.ln_post = nn.LayerNorm(config.d_model)
        self.gradient_checkpointing = False
        self.conv2d1 = nn.Conv2d(1, config.downsample_hidden_size, 3, 2, padding=1)
        self.conv2d2 = nn.Conv2d(
            config.downsample_hidden_size,
            config.downsample_hidden_size,
            3,
            2,
            padding=1,
        )
        self.conv2d3 = nn.Conv2d(
            config.downsample_hidden_size,
            config.downsample_hidden_size,
            3,
            2,
            padding=1,
        )
        conv_out_dim = config.downsample_hidden_size * (
            (((config.num_mel_bins + 1) // 2 + 1) // 2 + 1) // 2
        )
        self.conv_out = nn.Linear(conv_out_dim, config.d_model, bias=False)
        self.proj1 = nn.Linear(config.d_model, config.d_model)
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 210-213: Method `Qwen3OmniMoeAudioEncoder._freeze_parameters`
```python
    def _freeze_parameters(self):
        for param in self.parameters():
            param.requires_grad = False
        self._requires_grad = False
```
**EN:** This method implements `_freeze_parameters()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3OmniMoeAudioEncoder`.
**CN:** 这个方法实现了 `_freeze_parameters()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3OmniMoeAudioEncoder` 内部调用。

### Lines 215-216: Method `Qwen3OmniMoeAudioEncoder.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Module:
        return self.conv1
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 218-219: Method `Qwen3OmniMoeAudioEncoder.set_input_embeddings`
```python
    def set_input_embeddings(self, value: nn.Module):
        self.conv1 = value
```
**EN:** This method implements `set_input_embeddings(value: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_input_embeddings(value: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 221-319: Method `Qwen3OmniMoeAudioEncoder.forward`
```python
    def forward(
        self,
        input_features,
        feature_lens=None,
        aftercnn_lens=None,
    ):
        r"""
        feature_lens (`torch.LongTensor` of shape `(batch_size,)`):
            mel length
        aftercnn_lens (`torch.LongTensor` of shape `(batch_size,)`):
            mel length after cnn
        """
        aftercnn_lens = _get_feat_extract_output_lengths(feature_lens)
        chunk_num = torch.ceil(feature_lens / (self.n_window * 2)).long()

        chunk_lengths = torch.tensor(
            [self.n_window * 2] * chunk_num.sum(),
            dtype=torch.long,
            device=feature_lens.device,
        )
        tail_chunk_index = F.pad(chunk_num, (1, 0), value=-1).cumsum(0)[1:]
        chunk_lengths[tail_chunk_index] = feature_lens % (self.n_window * 2)
        chunk_lengths[chunk_lengths == 0] = self.n_window * 2

        chunk_list = input_features.T.split(chunk_lengths.tolist(), dim=0)
        padded_feature = nn.utils.rnn.pad_sequence(
            chunk_list, batch_first=True
        ).transpose(1, 2)

        # Introduce vectorized mask to avoid many small tensors
        feature_lens_after_cnn = _get_feat_extract_output_lengths(chunk_lengths)
        max_len_after_cnn = (
            int(feature_lens_after_cnn.max().item())
            if feature_lens_after_cnn.numel()
            else 0
        )

        idx = torch.arange(max_len_after_cnn, device=padded_feature.device)
        padded_mask_after_cnn = idx.unsqueeze(0) < feature_lens_after_cnn.unsqueeze(1)

# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_features, feature_lens=..., aftercnn_lens=...)` and feature_lens (`torch.LongTensor` of shape `(batch_size,)`):.
**CN:** 这个方法实现了 `forward(input_features, feature_lens=..., aftercnn_lens=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 322-328: Method `Qwen3OmniMoeAudioEncoder._get_feat_extract_output_lengths`
```python
    def _get_feat_extract_output_lengths(self, input_lengths: torch.LongTensor):
        """
        Computes the output length of the convolutional layers and the output length of the audio encoder
        """
        input_lengths = (input_lengths - 1) // 2 + 1
        output_lengths = (input_lengths - 2) // 2 + 1
        return input_lengths, output_lengths
```
**EN:** This method implements `_get_feat_extract_output_lengths(input_lengths: ...)` and Computes the output length of the convolutional layers and the output length of the audio encoder.
**CN:** 这个方法实现了 `_get_feat_extract_output_lengths(input_lengths: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 331-332: Class `Qwen3OmniMoeVisionPatchMerger` overview
```python
class Qwen3OmniMoeVisionPatchMerger(nn.Module):
```
**EN:** Defines `Qwen3OmniMoeVisionPatchMerger` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3OmniMoeVisionPatchMerger`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 333-366: Method `Qwen3OmniMoeVisionPatchMerger.__init__`
```python
    def __init__(
        self,
        dim: int,
        context_dim: int,
        spatial_merge_size: int = 2,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        use_postshuffle_norm=False,
    ) -> None:
        super().__init__()
        self.hidden_size = context_dim * (spatial_merge_size**2)
        self.use_postshuffle_norm = use_postshuffle_norm
        self.ln_q = nn.LayerNorm(
            self.hidden_size if use_postshuffle_norm else context_dim, eps=1e-6
        )
        self.mlp = nn.ModuleList(
            [
                ColumnParallelLinear(
                    self.hidden_size,
                    self.hidden_size,
                    bias=True,
                    quant_config=quant_config,
                    prefix=add_prefix("mlp.0", prefix),
                ),
                nn.GELU(),
                RowParallelLinear(
                    self.hidden_size,
                    dim,
                    bias=True,
                    quant_config=quant_config,
                    prefix=add_prefix("mlp.2", prefix),
                ),
            ]
        )
```
**EN:** This method implements `__init__(dim: ..., context_dim: ..., spatial_merge_size: ...=..., quant_config: ...=..., prefix: ...=..., use_postshuffle_norm=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., context_dim: ..., spatial_merge_size: ...=..., quant_config: ...=..., prefix: ...=..., use_postshuffle_norm=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 368-383: Method `Qwen3OmniMoeVisionPatchMerger.forward`
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = (
            x.view(-1, self.hidden_size)
            if self.use_postshuffle_norm
            else x.view(-1, x.shape[-1])
        )
        hidden = self.ln_q(x).view(-1, self.hidden_size)
        for layer in self.mlp:
            if isinstance(hidden, tuple):
                hidden = hidden[0]
            hidden = layer(hidden)

        if isinstance(hidden, tuple):
            hidden = hidden[0]

        return hidden
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 386-386: Class `Qwen3OmniMoeVisionEncoder` overview
```python
class Qwen3OmniMoeVisionEncoder(Qwen3VLMoeVisionModel):
```
**EN:** Defines `Qwen3OmniMoeVisionEncoder` as a reusable runtime type derived from Qwen3VLMoeVisionModel. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3OmniMoeVisionEncoder`，其继承关系为 Qwen3VLMoeVisionModel。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 387-387: Class `Qwen3OmniMoeVisionEncoder` attributes
```python
    config: Qwen3OmniMoeVisionEncoderConfig
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3OmniMoeVisionEncoder` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3OmniMoeVisionEncoder` 在运行时的行为。

### Lines 389-423: Method `Qwen3OmniMoeVisionEncoder.__init__`
```python
    def __init__(
        self,
        config: Qwen3OmniMoeVisionEncoderConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = None,
        **kwargs,
    ):
        super().__init__(
            vision_config=config,
            quant_config=quant_config,
            norm_eps=getattr(config, "rms_norm_eps", 1e-6),
        )

        self.merger = Qwen3OmniMoeVisionPatchMerger(
            dim=config.out_hidden_size,
            context_dim=config.hidden_size,
            spatial_merge_size=config.spatial_merge_size,
            quant_config=quant_config,
            use_postshuffle_norm=False,
            prefix=add_prefix("merger", prefix),
        )
        self.merger_list = nn.ModuleList(
            [
                Qwen3OmniMoeVisionPatchMerger(
                    dim=config.out_hidden_size,
                    context_dim=config.hidden_size,
                    spatial_merge_size=config.spatial_merge_size,
                    use_postshuffle_norm=True,
                    quant_config=quant_config,
                    prefix=add_prefix("merger_list", prefix),
                )
                for _ in range(len(config.deepstack_visual_indexes))
            ]
        )
        del self.deepstack_merger_list
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., **kwargs)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., **kwargs)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 425-427: Method `Qwen3OmniMoeVisionEncoder.deepstack_merger_list`
```python
    @property
    def deepstack_merger_list(self):
        return self.merger_list
```
**EN:** This method implements `deepstack_merger_list()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3OmniMoeVisionEncoder` Decorators: property.
**CN:** 这个方法实现了 `deepstack_merger_list()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3OmniMoeVisionEncoder` 内部调用 装饰器：property。

### Lines 429-431: Method `Qwen3OmniMoeVisionEncoder.dtype`
```python
    @property
    def dtype(self) -> torch.dtype:
        return self.patch_embed.proj.weight.dtype
```
**EN:** This method implements `dtype()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3OmniMoeVisionEncoder` Decorators: property.
**CN:** 这个方法实现了 `dtype()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3OmniMoeVisionEncoder` 内部调用 装饰器：property。

### Lines 433-435: Method `Qwen3OmniMoeVisionEncoder.device`
```python
    @property
    def device(self) -> torch.device:
        return self.patch_embed.proj.weight.device
```
**EN:** This method implements `device()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3OmniMoeVisionEncoder` Decorators: property.
**CN:** 这个方法实现了 `device()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3OmniMoeVisionEncoder` 内部调用 装饰器：property。

### Lines 438-438: Class `Qwen3OmniMoeThinkerForConditionalGeneration` overview
```python
class Qwen3OmniMoeThinkerForConditionalGeneration(Qwen3VLMoeForConditionalGeneration):
```
**EN:** Defines `Qwen3OmniMoeThinkerForConditionalGeneration` as a reusable runtime type derived from Qwen3VLMoeForConditionalGeneration. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3OmniMoeThinkerForConditionalGeneration`，其继承关系为 Qwen3VLMoeForConditionalGeneration。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 439-439: Class `Qwen3OmniMoeThinkerForConditionalGeneration` attributes
```python
    config: Qwen3OmniMoeThinkerConfig
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3OmniMoeThinkerForConditionalGeneration` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3OmniMoeThinkerForConditionalGeneration` 在运行时的行为。

### Lines 441-459: Method `Qwen3OmniMoeThinkerForConditionalGeneration.__init__`
```python
    def __init__(
        self,
        config: Qwen3OmniMoeThinkerConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__(
            config, quant_config, prefix, language_model_cls=Qwen3MoeLLMModel
        )
        self.audio_tower = Qwen3OmniMoeAudioEncoder(config.audio_config)
        self.visual = Qwen3OmniMoeVisionEncoder(
            config.vision_config,
            quant_config=quant_config,
            norm_eps=getattr(config, "rms_norm_eps", 1e-6),
            prefix=add_prefix("visual", prefix),
        )
        self.pad_token_id = (
            self.config.pad_token_id if self.config.pad_token_id is not None else -1
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 461-492: Method `Qwen3OmniMoeThinkerForConditionalGeneration.get_audio_feature`
```python
    def get_audio_feature(self, items: List[MultimodalDataItem]):
        device = next(self.audio_tower.parameters()).device
        feature_attention_mask = (
            torch.cat([item.feature_attention_mask for item in items], dim=0)
            .type(torch.long)
            .to(device)
        )
        input_features = (
            torch.cat([item.feature for item in items])
            .type(self.audio_tower.dtype)
            .to(next(self.audio_tower.parameters()).device)
        )
        if feature_attention_mask is not None:
            audio_feature_lengths = torch.sum(feature_attention_mask, dim=1)
            input_features = input_features.permute(0, 2, 1)[
                feature_attention_mask.bool()
            ].permute(1, 0)
        else:
            audio_feature_lengths = None

        feature_lens = (
            audio_feature_lengths
            if audio_feature_lengths is not None
            else feature_attention_mask.sum(-1)
        )
        audio_outputs = self.audio_tower(
            input_features,
            feature_lens=feature_lens,
        )
        audio_features = audio_outputs.last_hidden_state

        return audio_features
```
**EN:** This method implements `get_audio_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_audio_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 495-495: Class `Qwen3OmniMoeForConditionalGeneration` overview
```python
class Qwen3OmniMoeForConditionalGeneration(PreTrainedModel):
```
**EN:** Defines `Qwen3OmniMoeForConditionalGeneration` as a reusable runtime type derived from PreTrainedModel. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3OmniMoeForConditionalGeneration`，其继承关系为 PreTrainedModel。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 496-510: Method `Qwen3OmniMoeForConditionalGeneration.__init__`
```python
    def __init__(
        self,
        config: Qwen3VLMoeConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__(config)
        self.config = config

        self.thinker = Qwen3OmniMoeThinkerForConditionalGeneration(
            config.thinker_config, quant_config=quant_config, prefix=prefix
        )
        self.enable_talker = False
        self.pad_input_ids = self.thinker.pad_input_ids
        self.forward = self.thinker.forward
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 512-683: Method `Qwen3OmniMoeForConditionalGeneration.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            ("gate_up_proj", "up_proj", 1),
            ("gate_up_proj", "gate_proj", 0),
        ]

        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="gate_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="up_proj",
            num_experts=self.config.num_experts,
        )

        # Skip loading extra parameters for GPTQ/modelopt models.
        ignore_suffixes = (
            ".bias",
            "_bias",
            ".k_scale",
            "_k_scale",
            ".v_scale",
            "_v_scale",
            ".weight_scale",
            "_weight_scale",
            ".input_scale",
            "_input_scale",
        )

        is_fused_expert = False
        fused_expert_params_mapping = [
            ("experts.w13_weight", "experts.gate_up_proj", 0, "w1"),
            ("experts.w2_weight", "experts.down_proj", 0, "w2"),
        ]

        num_experts = self.config.num_experts

        # Pre-define `params_dict` to avoid repeated expensive traversal of model parameters.
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 686-686: Top-level assign
```python
EntryClass = Qwen3OmniMoeForConditionalGeneration
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `math`
- `typing.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `numpy`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `transformers.PreTrainedModel`
- `transformers.activations.ACT2FN`
- `transformers.modeling_outputs.BaseModelOutput`
- `sglang.srt.configs.qwen3_omni.Qwen3OmniMoeAudioEncoderConfig`
- `sglang.srt.configs.qwen3_omni.Qwen3OmniMoeThinkerConfig`
- `sglang.srt.configs.qwen3_omni.Qwen3OmniMoeVisionEncoderConfig`
- `sglang.srt.configs.qwen3_vl.Qwen3VLMoeConfig`
- `sglang.srt.layers.attention.vision.VisionAttention`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.qwen3_vl.Qwen3VLMoeVisionModel`
- `sglang.srt.models.qwen3_vl_moe.Qwen3MoeLLMModel`
- `sglang.srt.models.qwen3_vl_moe.Qwen3VLMoeForConditionalGeneration`
- `sglang.srt.models.qwen3_vl_moe.load_fused_expert_weights`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.is_npu`
- `sglang.srt.utils.logger`
