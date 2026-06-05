# gemma4_audio.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/gemma4_audio.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the gemma4 Audio architecture into SGLang's serving runtime. SGLang-native TP-sharded audio encoder for Gemma 4. / 该模块将 gemma4 Audio 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：SGLang-native TP-sharded audio encoder for Gemma 4。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13: Module header and imports / 模块头与导入
```python
# Copyright 2025 SGLang Team
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
# ==============================================================================
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 53-53: Assignment: _SSCP_INPUT_FEAT_SIZE / 赋值：_SSCP_INPUT_FEAT_SIZE
```python
_SSCP_INPUT_FEAT_SIZE = 128
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 54-54: Assignment: _SSCP_CONV_KERNEL_SIZES / 赋值：_SSCP_CONV_KERNEL_SIZES
```python
_SSCP_CONV_KERNEL_SIZES = ((3, 3), (3, 3))
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 55-55: Assignment: _SSCP_CONV_STRIDE_SIZES / 赋值：_SSCP_CONV_STRIDE_SIZES
```python
_SSCP_CONV_STRIDE_SIZES = ((2, 2), (2, 2))
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 62-204: Class: Gemma4AudioRelativePositionEmbedding / 类：Gemma4AudioRelativePositionEmbedding
```python
class Gemma4AudioRelativePositionEmbedding(nn.Module):
    def __init__(
        self,
        config: Gemma4AudioConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        tp_size = get_attention_tp_size()
        total_num_heads = config.num_attention_heads
        self.channels = config.hidden_size
        self.head_dim = self.channels // total_num_heads
        self.num_heads = total_num_heads // tp_size
        self.max_backward = max(0, config.attention_context_left - 1)
        self.max_forward = config.attention_context_right

        self.pos_proj = ColumnParallelLinear(
            self.channels,
            config.hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("pos_proj", prefix),
        )

        min_timescale = 1.0
        max_timescale = 1.0e4
        num_timescales = self.channels // 2
        log_timescale_increment = math.log(
            float(max_timescale) / float(min_timescale)
        ) / max(num_timescales - 1, 1)
        inv_timescales = min_timescale * torch.exp(
            torch.arange(num_timescales) * -log_timescale_increment
        )
        self.register_buffer(
            "inv_timescales",
            inv_timescales.float().unsqueeze(0).unsqueeze(0),
            persistent=False,
        )
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Audio Relative Position Embedding inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Audio Relative Position Embedding，用于封装该模型组件的状态与方法。

### Lines 212-382: Class: Gemma4AudioAttention / 类：Gemma4AudioAttention
```python
class Gemma4AudioAttention(nn.Module):
    def __init__(
        self,
        config: Gemma4AudioConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        tp_size = get_attention_tp_size()
        total_num_heads = config.num_attention_heads
        self.hidden_size = config.hidden_size
        self.head_dim = self.hidden_size // total_num_heads
        self.num_heads = total_num_heads // tp_size

        self.chunk_size = config.attention_chunk_size
        self.max_future_horizon = config.attention_context_right
        self.max_past_horizon = max(0, config.attention_context_left - 1)
        self.attention_logits_soft_cap = config.attention_logit_cap
        self.context_size = (
            self.chunk_size + self.max_past_horizon + self.max_future_horizon
        )

        self.relative_position_embedding = Gemma4AudioRelativePositionEmbedding(
            config,
            quant_config,
            prefix=add_prefix("relative_position_embedding", prefix),
        )
        self.per_dim_scale = nn.Parameter(torch.zeros((self.head_dim,)))

        self.qkv = ClippableQKVParallelLinear(
            hidden_size=self.hidden_size,
            head_size=self.head_dim,
            total_num_heads=total_num_heads,
            total_num_kv_heads=total_num_heads,
            bias=False,
            quant_config=quant_config,
            prefix=prefix,
        )
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Audio Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Audio Attention，用于封装该模型组件的状态与方法。

### Lines 390-456: Class: Gemma4AudioSSCPConvBlock / 类：Gemma4AudioSSCPConvBlock
```python
class Gemma4AudioSSCPConvBlock(nn.Module):
    """Single 2D conv block with LayerNorm and semicausal padding."""

    def __init__(
        self,
        config: Gemma4AudioConfig,
        idx: int,
        input_freq_dim: int,
    ):
        super().__init__()
        self.config = config

        conv_channels = config.subsampling_conv_channels
        in_channels = 1 if idx == 0 else conv_channels[idx - 1]
        out_channels = conv_channels[idx]
        kernel_t, kernel_f = _SSCP_CONV_KERNEL_SIZES[idx]
        stride_t, stride_f = _SSCP_CONV_STRIDE_SIZES[idx]
        self.time_stride = stride_t

        # Semicausal padding (hardcoded — streaming is not supported)
        pad_t_top = kernel_t // 2
        pad_t_bottom = kernel_t // 2

        pad_f_left = 1
        pad_f_right = 1

        self.manual_padding = (pad_f_left, pad_f_right, pad_t_top, pad_t_bottom)

        self.conv = nn.Conv2d(
            in_channels=in_channels,
            out_channels=out_channels,
            kernel_size=(kernel_t, kernel_f),
            stride=(stride_t, stride_f),
            padding=(0, 0),
            bias=False,
        )

        f_in_padded = input_freq_dim + pad_f_left + pad_f_right
        self.f_out_conv = (f_in_padded - kernel_f) // stride_f + 1

# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Audio S S C P Conv Block inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Single 2D conv block with LayerNorm and semicausal padding..
**CN:** 该类定义了 Gemma4 Audio S S C P Conv Block，用于封装该模型组件的状态与方法。 文档字符串摘要：Single 2D conv block with LayerNorm and semicausal padding.。

### Lines 459-519: Class: Gemma4AudioSubSampleConvProjection / 类：Gemma4AudioSubSampleConvProjection
```python
class Gemma4AudioSubSampleConvProjection(nn.Module):
    def __init__(
        self,
        config: Gemma4AudioConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        conv_channels = config.subsampling_conv_channels

        current_f = _SSCP_INPUT_FEAT_SIZE
        calculated_f_out_dims = []

        for i in range(2):
            kernel_h, kernel_w = _SSCP_CONV_KERNEL_SIZES[i]
            stride_h, stride_w = _SSCP_CONV_STRIDE_SIZES[i]

            pad_f_left = 1
            pad_f_right = 1
            f_in_padded = current_f + pad_f_left + pad_f_right
            f_out = (f_in_padded - kernel_w) // stride_w + 1
            calculated_f_out_dims.append(f_out)
            current_f = f_out

        self.conv_0 = Gemma4AudioSSCPConvBlock(
            idx=0,
            input_freq_dim=_SSCP_INPUT_FEAT_SIZE,
            config=config,
        )
        self.conv_1 = Gemma4AudioSSCPConvBlock(
            idx=1,
            input_freq_dim=calculated_f_out_dims[0],
            config=config,
        )

        final_c_out = conv_channels[-1]
        final_f_out = calculated_f_out_dims[-1]
        self.input_proj_in_features = final_c_out * final_f_out
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Audio Sub Sample Conv Projection inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Audio Sub Sample Conv Projection，用于封装该模型组件的状态与方法。

### Lines 527-581: Class: Gemma4AudioConformerAttention / 类：Gemma4AudioConformerAttention
```python
class Gemma4AudioConformerAttention(nn.Module):
    def __init__(
        self,
        config: Gemma4AudioConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.post_in_features = config.hidden_size

        self.register_buffer(
            "gradient_clipping",
            torch.tensor(config.gradient_clipping),
            persistent=False,
        )

        self.pre_attn_norm = Gemma4RMSNorm(config.hidden_size, scale_shift=0.0)
        self.attn = Gemma4AudioAttention(
            config, quant_config, prefix=add_prefix("attn", prefix)
        )
        self.post = ClippableRowParallelLinear(
            self.post_in_features,
            config.hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("post", prefix),
        )
        self.post_norm = Gemma4RMSNorm(config.hidden_size, scale_shift=0.0)

    def forward(
        self,
        audio_encodings: torch.Tensor,
        audio_mel_mask: torch.BoolTensor,
        causal_valid_mask: torch.BoolTensor,
    ) -> torch.Tensor:
        audio_encodings_input_to_attn = audio_encodings
        audio_encodings = torch.clamp(
            audio_encodings, -self.gradient_clipping, self.gradient_clipping
        )
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Audio Conformer Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Audio Conformer Attention，用于封装该模型组件的状态与方法。

### Lines 584-631: Class: Gemma4AudioConformerFeedForward / 类：Gemma4AudioConformerFeedForward
```python
class Gemma4AudioConformerFeedForward(nn.Module):
    def __init__(
        self,
        config: Gemma4AudioConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        self.register_buffer(
            "gradient_clipping",
            torch.tensor(config.gradient_clipping),
            persistent=False,
        )

        self.pre_layer_norm = Gemma4RMSNorm(config.hidden_size, scale_shift=0.0)
        self.ffw_layer_1 = ClippableColumnParallelLinear(
            config.hidden_size,
            config.hidden_size * 4,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("ffw_layer_1", prefix),
        )
        self.ffw_layer_2 = ClippableRowParallelLinear(
            config.hidden_size * 4,
            config.hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("ffw_layer_2", prefix),
        )
        self.post_layer_norm = Gemma4RMSNorm(config.hidden_size, scale_shift=0.0)
        self.post_layer_scale = config.residual_weight

    def forward(self, audio_encodings: torch.Tensor) -> torch.Tensor:
        residual = audio_encodings
        audio_encodings = torch.clamp(
            audio_encodings, -self.gradient_clipping, self.gradient_clipping
        )
        audio_encodings = self.pre_layer_norm(audio_encodings)
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Audio Conformer Feed Forward inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Audio Conformer Feed Forward，用于封装该模型组件的状态与方法。

### Lines 634-712: Class: Gemma4AudioConformerLightConv1d / 类：Gemma4AudioConformerLightConv1d
```python
class Gemma4AudioConformerLightConv1d(nn.Module):
    def __init__(
        self,
        config: Gemma4AudioConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.causal_padding = config.conv_kernel_size - 1
        tp_size = get_attention_tp_size()
        hidden_per_tp = config.hidden_size // tp_size

        self.register_buffer(
            "gradient_clipping",
            torch.tensor(config.gradient_clipping),
            persistent=False,
        )

        self.pre_layer_norm = Gemma4RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps, scale_shift=0.0
        )
        self.linear_start = ClippableGLUParallelLinear(
            config.hidden_size,
            config.hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("linear_start", prefix),
        )
        self.depthwise_conv1d = nn.Conv1d(
            in_channels=hidden_per_tp,
            out_channels=hidden_per_tp,
            kernel_size=config.conv_kernel_size,
            stride=1,
            padding=0,
            groups=hidden_per_tp,
            bias=False,
        )
        self.conv_norm = Gemma4RMSNorm(
            hidden_per_tp, eps=config.rms_norm_eps, scale_shift=0.0
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Audio Conformer Light Conv1d inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Audio Conformer Light Conv1d，用于封装该模型组件的状态与方法。

### Lines 715-765: Class: Gemma4AudioConformerBlock / 类：Gemma4AudioConformerBlock
```python
class Gemma4AudioConformerBlock(nn.Module):
    def __init__(
        self,
        config: Gemma4AudioConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        self.ffw_layer_start = Gemma4AudioConformerFeedForward(
            config, quant_config, prefix=add_prefix("ffw_layer_start", prefix)
        )
        self.attention = Gemma4AudioConformerAttention(
            config, quant_config, prefix=add_prefix("attention", prefix)
        )
        self.lconv1d = Gemma4AudioConformerLightConv1d(
            config, quant_config, prefix=add_prefix("lconv1d", prefix)
        )
        self.ffw_layer_end = Gemma4AudioConformerFeedForward(
            config, quant_config, prefix=add_prefix("ffw_layer_end", prefix)
        )
        self.register_buffer(
            "gradient_clipping",
            torch.tensor(config.gradient_clipping),
            persistent=False,
        )
        self.norm = Gemma4RMSNorm(config.hidden_size, scale_shift=0.0)

    def forward(
        self,
        audio_encodings: torch.Tensor,
        audio_mel_mask: torch.BoolTensor,
        causal_valid_mask: torch.BoolTensor,
    ) -> torch.Tensor:
        audio_encodings = self.ffw_layer_start(audio_encodings)
        audio_encodings = self.attention(
            audio_encodings, audio_mel_mask, causal_valid_mask
        )
        validity_mask_for_lconv = ~audio_mel_mask
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Audio Conformer Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Audio Conformer Block，用于封装该模型组件的状态与方法。

### Lines 773-873: Class: Gemma4AudioEncoder / 类：Gemma4AudioEncoder
```python
class Gemma4AudioEncoder(nn.Module):
    """SGLang-native TP-sharded Gemma 4 audio encoder (USM Conformer + SSCP)."""

    def __init__(
        self,
        config: Gemma4AudioConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        self.subsample_conv_projection = Gemma4AudioSubSampleConvProjection(
            config, quant_config, prefix=add_prefix("subsample_conv_projection", prefix)
        )
        self.conformer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: Gemma4AudioConformerBlock(
                config=config,
                quant_config=quant_config,
                prefix=prefix,
            ),
            prefix=add_prefix("conformer", prefix),
        )

        if config.output_proj_dims is not None:
            self.output_proj = RowParallelLinear(
                config.hidden_size,
                config.output_proj_dims,
                bias=True,
                input_is_parallel=False,
                quant_config=quant_config,
                prefix=add_prefix("output_proj", prefix),
            )
        else:
            self.output_proj = None

        # Precompute causal_valid_mask — depends only on static config values.
        chunk_size = config.attention_chunk_size
        max_future_horizon = config.attention_context_right
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Audio Encoder inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: SGLang-native TP-sharded Gemma 4 audio encoder (USM Conformer + SSCP)..
**CN:** 该类定义了 Gemma4 Audio Encoder，用于封装该模型组件的状态与方法。 文档字符串摘要：SGLang-native TP-sharded Gemma 4 audio encoder (USM Conformer + SSCP).。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `math`
- `typing: Optional, Tuple`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `transformers: Gemma4AudioConfig`
- `sglang.srt.layers.clippable_linear: ClippableColumnParallelLinear, ClippableGLUParallelLinear, ClippableQKVParallelLinear, ClippableRowParallelLinear`
- `sglang.srt.layers.dp_attention: get_attention_tp_rank, get_attention_tp_size`
- `sglang.srt.layers.layernorm: Gemma4RMSNorm`
- `sglang.srt.layers.linear: ColumnParallelLinear, RowParallelLinear`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.utils: add_prefix, make_layers, set_weight_attrs`
