# whisper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/whisper.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Whisper multimodal model adapter used for inference in vLLM. / 实现 Whisper 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-86)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import enum
import math
from collections.abc import Iterable, Mapping, Sequence
from contextlib import nullcontext
from typing import Annotated

import numpy as np
import torch
from torch import nn
from transformers import (
    BatchFeature,
    WhisperConfig,
    WhisperFeatureExtractor,
)
from transformers.models.whisper.modeling_whisper import sinusoids
# ... omitted for brevity ...
)

from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsTranscription,
)
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    cast_overflow_tensors,
    make_layers,
    maybe_prefix,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as numpy, torch, transformers, transformers.models.whisper.modeling_whisper supply framework primitives, while internal modules like vllm.compilation.decorators, vllm.config, vllm.config.multimodal, vllm.config.speech_to_text connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 numpy, torch, transformers, transformers.models.whisper.modeling_whisper 这样的外部依赖提供基础框架能力，而 vllm.compilation.decorators, vllm.config, vllm.config.multimodal, vllm.config.speech_to_text 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger` (lines 87-87)
```python
logger = init_logger(__name__)
```
**EN:** This assignment initializes the module-level logger used for diagnostics, backend selection messages, and compatibility warnings.
**CN:** 这个赋值语句初始化了模块级日志记录器，用于输出诊断信息、后端选择消息和兼容性警告。

### Class `WhisperAudioInputs` (lines 96-107)
```python
class WhisperAudioInputs(TensorSchema):
    """
    Dimensions:
        - b: Batch size
        - nmb: Number of mel bins
        - t: Time frames (M)
    """

    input_features: Annotated[
        list[torch.Tensor] | None,
        TensorShape("b", "nmb", "t"),
    ]
```
**EN:** Defines `WhisperAudioInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - b: Batch size - nmb: Number of mel bins - t: Time frames (M)."
**CN:** 定义 `WhisperAudioInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - b: Batch size - nmb: Number of mel bins - t: Time frames (M)。”

### Class `WhisperEncoderAttention` (lines 110-135)
```python
class WhisperEncoderAttention(MMEncoderAttention):
    """Multi-headed attention for Whisper encoder with 2D tensor support."""

    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
    ) -> torch.Tensor:
        """
        Input shape: batch_size x seq_len x hidden_size
                     or seq_len x hidden_size
        """
        is_2d = query.dim() == 2
        if is_2d:
            query = query.unsqueeze(0)
            key = key.unsqueeze(0)
            value = value.unsqueeze(0)

        # Call the parent forward method
        out = super().forward(query, key, value)

        if is_2d:
            out = out.squeeze(0)

        return out
```
**EN:** Defines `WhisperEncoderAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from MMEncoderAttention. Key methods such as `forward` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Multi-headed attention for Whisper encoder with 2D tensor support."
**CN:** 定义 `WhisperEncoderAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 MMEncoderAttention。 `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Multi-headed attention for Whisper encoder with 2D tensor support。”

### Class `WhisperAttention` (lines 146-252)
```python
class WhisperAttention(nn.Module):
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        bias: bool = True,
        attn_type: AttentionType = AttentionType.DECODER,
        per_layer_sliding_window: int | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.embed_dim = embed_dim
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        if self.total_num_heads >= tp_size:
            # Number of heads is greater than TP size, so we partition
# ... omitted for brevity ...
    def _init_qkv(
        self,
        embed_dim: int,
        bias: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        self.qkv_proj = QKVParallelLinear(
            hidden_size=embed_dim,
            head_size=self.head_dim,
            total_num_heads=self.total_num_heads,
            total_num_kv_heads=self.total_num_heads,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )
# ... omitted for brevity ...
    def forward(
        self,
        hidden_states: torch.Tensor,
    ):
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)

        attn_output = self.attn(q, k, v)

        output, _ = self.out_proj(attn_output)

        return output
```
**EN:** Defines `WhisperAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `_init_qkv`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `WhisperAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `_init_qkv`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `WhisperCrossAttention` (lines 255-319)
```python
class WhisperCrossAttention(WhisperAttention):
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        bias: bool = True,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__(
            embed_dim=embed_dim,
            num_heads=num_heads,
            bias=bias,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=prefix,
            attn_type=AttentionType.ENCODER_DECODER,
        )

    def _init_qkv(
        self,
        embed_dim: int,
        bias: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        self.q_proj = ColumnParallelLinear(
            input_size=embed_dim,
            output_size=embed_dim,
# ... omitted for brevity ...
        # which handles 2-slice configurations.
        self.kv_proj = MergedColumnParallelLinear(
            input_size=embed_dim,
            output_sizes=[embed_dim, embed_dim],
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.kv_proj",
        )

    def forward(
        self,
        hidden_states: torch.Tensor,
        encoder_hidden_states: torch.Tensor | None,
    ):
        q, _ = self.q_proj(hidden_states)

        # Encoder hidden states are only computed once during prefill phase.
        # Afterwards, the keys and values should be available in the kv-cache.
        if encoder_hidden_states is not None:
            kv, _ = self.kv_proj(encoder_hidden_states)
            k, v = kv.split([self.kv_size, self.kv_size], dim=-1)
        else:
            k = v = None

        attn_output = self.attn(q, k, v)

        output, _ = self.out_proj(attn_output)

        return output
```
**EN:** Defines `WhisperCrossAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from WhisperAttention. Key methods such as `__init__`, `_init_qkv`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `WhisperCrossAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 WhisperAttention。 `__init__`, `_init_qkv`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `WhisperEncoderLayer` (lines 354-397)
```python
class WhisperEncoderLayer(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        sliding_window = getattr(config, "sliding_window", None)
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.embed_dim = config.d_model
        self.self_attn = WhisperAttention(
            embed_dim=self.embed_dim,
            num_heads=config.encoder_attention_heads,
            attn_type=AttentionType.ENCODER,
            per_layer_sliding_window=sliding_window,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.self_attn_layer_norm = nn.LayerNorm(self.embed_dim)
        self.mlp = WhisperMLP(
            embed_dim=config.d_model,
            ffn_dim=config.encoder_ffn_dim,
            act_fn=config.activation_function,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )
        self.final_layer_norm = nn.LayerNorm(self.embed_dim)

    def forward(
        self,
        hidden_states: torch.Tensor,
    ):
        residual = hidden_states
        hidden_states = self.self_attn_layer_norm(hidden_states)
        hidden_states = self.self_attn(hidden_states=hidden_states)
        hidden_states = residual + hidden_states
        residual = hidden_states
        hidden_states = self.final_layer_norm(hidden_states)
        hidden_states = self.mlp(hidden_states)
        hidden_states = residual + hidden_states

        hidden_states = cast_overflow_tensors(hidden_states)

        return hidden_states
```
**EN:** Defines `WhisperEncoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `WhisperEncoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `WhisperDecoderLayer` (lines 400-456)
```python
class WhisperDecoderLayer(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.self_attn = WhisperAttention(
            embed_dim=config.d_model,
            num_heads=config.decoder_attention_heads,
            attn_type=AttentionType.DECODER,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.self_attn_layer_norm = nn.LayerNorm(config.d_model)
        self.encoder_attn = WhisperCrossAttention(
            embed_dim=config.d_model,
            num_heads=config.decoder_attention_heads,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.encoder_attn",
        )
        self.encoder_attn_layer_norm = nn.LayerNorm(config.d_model)
        self.mlp = WhisperMLP(
            embed_dim=config.d_model,
            ffn_dim=config.decoder_ffn_dim,
            act_fn=config.activation_function,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )
        self.final_layer_norm = nn.LayerNorm(config.d_model)

    def forward(
        self,
        hidden_states: torch.Tensor,
        encoder_hidden_states: torch.Tensor | None,
    ):
        residual = hidden_states
        hidden_states = self.self_attn_layer_norm(hidden_states)
        hidden_states = self.self_attn(hidden_states=hidden_states)
        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.encoder_attn_layer_norm(hidden_states)
        hidden_states = self.encoder_attn(
            hidden_states=hidden_states,
            encoder_hidden_states=encoder_hidden_states,
        )
        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.final_layer_norm(hidden_states)
        hidden_states = self.mlp(hidden_states)
        hidden_states = residual + hidden_states

        return hidden_states
```
**EN:** Defines `WhisperDecoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `WhisperDecoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `WhisperEncoder` (lines 459-536)
```python
class WhisperEncoder(nn.Module):
    def __init__(
        self, *, vllm_config: VllmConfig, prefix: str = "", init_in_fp32: bool = False
    ):
        super().__init__()
        config = vllm_config.model_config.hf_config
        embed_dim = config.d_model

        self.pos_embed_type = WhisperPosEmbedType(
            getattr(config, "pos_embed", "sinusoidal")
        )
        self.num_mel_bins = config.num_mel_bins
        self.max_source_positions = config.max_source_positions
        self.embed_scale = math.sqrt(embed_dim) if config.scale_embedding else 1.0

        self.conv1 = nn.Conv1d(self.num_mel_bins, embed_dim, kernel_size=3, padding=1)
        self.conv2 = nn.Conv1d(embed_dim, embed_dim, stride=2, kernel_size=3, padding=1)

        self.total_stride = self.conv1.stride[0] * self.conv2.stride[0]
        self.start_layer, self.end_layer, self.layers = make_layers(
# ... omitted for brevity ...
    def forward(
        self, input_features: torch.Tensor | list[torch.Tensor]
    ) -> torch.Tensor:
        hidden_states = []
        input_is_batched = False
        for features in input_features:
            embeds = nn.functional.gelu(self.conv1(features))
            embeds = nn.functional.gelu(self.conv2(embeds))

            embeds = embeds.transpose(-1, -2)
            embeds = (embeds + self.embed_positions.weight[: embeds.size(-2), :]).to(
                embeds.dtype
            )

            hidden_states.append(embeds)
            input_is_batched = embeds.ndim > 2
        # Input to MHA must be B x T x D
        if input_is_batched:
            # Models using WhisperEncoder may handle batching internally.
```
**EN:** Defines `WhisperEncoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `WhisperEncoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `WhisperDecoder` (lines 539-585)
```python
@support_torch_compile(dynamic_arg_dims={"input_ids": 0, "positions": -1})
class WhisperDecoder(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        self.layerdrop = config.decoder_layerdrop
        self.padding_idx = config.pad_token_id
        self.max_target_positions = config.max_target_positions
        self.max_source_positions = config.max_source_positions
        self.embed_scale = math.sqrt(config.d_model) if config.scale_embedding else 1.0

        self.embed_tokens = nn.Embedding(
            config.vocab_size, config.d_model, self.padding_idx
        )
        self.embed_positions = WhisperPositionalEmbedding(
            self.max_target_positions, config.d_model
        )
        self.start_layer, self.end_layer, self.layers = make_layers(
            config.decoder_layers,
            lambda prefix: WhisperDecoderLayer(
                vllm_config=vllm_config, prefix=f"{prefix}.layers"
            ),
            prefix=f"{prefix}.layers",
        )
        self.layer_norm = nn.LayerNorm(config.d_model)

    def forward(
        self,
        input_ids,
        positions: torch.Tensor,
        encoder_hidden_states: torch.Tensor | None,
    ):
        inputs_embeds = self.embed_input_ids(input_ids)
        positions = self.embed_positions(positions)
        hidden_states = inputs_embeds + positions

        for decoder_layer in self.layers:
            hidden_states = decoder_layer(
                hidden_states,
                encoder_hidden_states=encoder_hidden_states,
            )

        hidden_states = self.layer_norm(hidden_states)
        return hidden_states

    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** Defines `WhisperDecoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward`, `embed_input_ids` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `WhisperDecoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward`, `embed_input_ids` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Function `_create_fake_bias_for_k_proj` (lines 1029-1041)
```python
def _create_fake_bias_for_k_proj(
    weights: Iterable[tuple[str, torch.Tensor]], fake_bias_key_name: str
) -> Iterable[tuple[str, torch.Tensor]]:
    """
    Create full zeros bias for k_proj weight in self-attn and x-attn layers.
    So that the bias for k_proj in qkv_proj can be initialized with zeros.
    """
    for name, weight in weights:
        yield name, weight
        if name.endswith(fake_bias_key_name):
            bias = torch.zeros(weight.size(0))
            bias_name = name.replace("weight", "bias")
            yield bias_name, bias
```
**EN:** The function `_create_fake_bias_for_k_proj` helps provide a reusable helper for the surrounding model code. Its main inputs are `weights`, `fake_bias_key_name`. Docstring hint: "Create full zeros bias for k_proj weight in self-attn and x-attn layers."
**CN:** 函数 `_create_fake_bias_for_k_proj` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `weights`、`fake_bias_key_name`。 文档提示：“Create full zeros bias for k_proj weight in self-attn and x-attn layers。”

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Distributed execution**
  - **EN:** The implementation is aware of tensor parallelism or pipeline parallelism, so layers and weights can be sharded across devices.
  - **CN:** 实现考虑了张量并行或流水线并行，因此层和权重可以在多设备间切分。
- **Quantized weights**
  - **EN:** Quantization hooks allow the model to load compressed checkpoints or specialized linear layers.
  - **CN:** 量化钩子使模型能够加载压缩检查点或专用线性层。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。

## Dependencies / 依赖关系
- **Standard library**: enum, math, collections.abc, contextlib, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: numpy, torch, transformers, transformers.models.whisper.modeling_whisper
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.compilation.decorators, vllm.config, vllm.config.multimodal, vllm.config.speech_to_text, vllm.distributed, vllm.inputs, vllm.logger, vllm.model_executor.layers.activation
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
