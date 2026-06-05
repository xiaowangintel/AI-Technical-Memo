# whisper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/whisper.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the whisper model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 whisper 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19: Module imports
```python
from typing import Any, Iterable, List, Optional, Tuple

import torch
from transformers import WhisperConfig

from sglang.srt.distributed import get_tensor_model_parallel_world_size
from sglang.srt.layers.activation import get_act_fn
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor, LogitsProcessorOutput
from sglang.srt.layers.quantization import QuantizationConfig
from sglang.srt.layers.radix_attention import AttentionType, RadixAttention
from sglang.srt.layers.vocab_parallel_embedding import ParallelLMHead
from sglang.srt.managers.schedule_batch import MultimodalInputs
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 22-24: Class `WhisperAttention` overview
```python
class WhisperAttention(torch.nn.Module):
    """Multi-headed attention from 'Attention Is All You Need' paper"""
```
**EN:** Defines `WhisperAttention` as a reusable runtime type derived from torch.nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `WhisperAttention`，其继承关系为 torch.nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 25-86: Method `WhisperAttention.__init__`
```python
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        bias: bool = True,
        layer_id: Optional[int] = None,
        quant_config: Optional[QuantizationConfig] = None,
        is_cross_attention: bool = False,
        is_encoder=False,
    ):
        super().__init__()
        self.total_num_heads = num_heads
        head_dim = embed_dim // num_heads
        self.is_cross_attention = is_cross_attention
        self.is_encoder = is_encoder

        tp_size = get_tensor_model_parallel_world_size()
        assert (
            num_heads % tp_size == 0
        ), f"num_heads ({num_heads}) must be divisible by tp_size ({tp_size})"
        self.num_heads = num_heads // tp_size

        if (head_dim * num_heads) != embed_dim:
            raise ValueError(
                f"embed_dim must be divisible by num_heads (got `embed_dim`: {embed_dim}"
                f" and `num_heads`: {num_heads})."
            )
        self.scaling = head_dim**-0.5
        self.head_dim = head_dim
        self.kv_size = self.num_heads * head_dim

        if is_cross_attention:
            self.q_proj = ColumnParallelLinear(
                embed_dim, embed_dim, quant_config=quant_config
            )
            self.kv_proj = QKVParallelLinear(
                hidden_size=embed_dim,
                head_size=head_dim,
                total_num_heads=0,
                total_num_kv_heads=num_heads,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(embed_dim: ..., num_heads: ..., bias: ...=..., layer_id: ...=..., quant_config: ...=..., is_cross_attention: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(embed_dim: ..., num_heads: ..., bias: ...=..., layer_id: ...=..., quant_config: ...=..., is_cross_attention: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 88-132: Method `WhisperAttention.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        cross_hidden_states: Optional[torch.Tensor] = None,
    ) -> tuple[torch.Tensor, Optional[torch.Tensor]]:
        """Input shape: Batch x Time x Channel"""

        if self.is_cross_attention:
            # Cross-attention: KV cached during prefill, read from pool during decode.
            q, _ = self.q_proj(hidden_states)
            q = q * self.scaling
            if cross_hidden_states is not None:
                kv, _ = self.kv_proj(cross_hidden_states)
                k, v = kv.split([self.kv_size, self.kv_size], dim=-1)
            else:
                k = None
                v = None
            attn_output = self.attn(q, k, v, forward_batch)
        else:
            qkv, _ = self.qkv_proj(hidden_states)
            q, k, v = qkv.chunk(chunks=3, dim=-1)
            q = q * self.scaling

            if self.is_encoder:
                num_heads = self.attn.tp_q_head_num
                head_dim = self.attn.head_dim
                batch_size, seq_len, _ = hidden_states.shape

                q = q.view(batch_size, seq_len, num_heads, head_dim).permute(0, 2, 1, 3)
                k = k.view(batch_size, seq_len, num_heads, head_dim).permute(0, 2, 1, 3)
                v = v.view(batch_size, seq_len, num_heads, head_dim).permute(0, 2, 1, 3)

                attn_output = torch.nn.functional.scaled_dot_product_attention(
                    q, k, v, scale=1.0
                )
                attn_output = attn_output.permute(0, 2, 1, 3).reshape(
                    batch_size, seq_len, num_heads * head_dim
                )
            else:
# ... truncated for brevity ...
```
**EN:** This method implements `forward(hidden_states: ..., forward_batch: ..., cross_hidden_states: ...=...)` and Input shape: Batch x Time x Channel.
**CN:** 这个方法实现了 `forward(hidden_states: ..., forward_batch: ..., cross_hidden_states: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 135-135: Class `WhisperEncoderLayer` overview
```python
class WhisperEncoderLayer(torch.nn.Module):
```
**EN:** Defines `WhisperEncoderLayer` as a reusable runtime type derived from torch.nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `WhisperEncoderLayer`，其继承关系为 torch.nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 136-160: Method `WhisperEncoderLayer.__init__`
```python
    def __init__(
        self,
        config: WhisperConfig,
        layer_id: Optional[int] = None,
        quant_config: Optional[QuantizationConfig] = None,
    ):
        super().__init__()
        self.embed_dim = config.d_model

        self.self_attn = WhisperAttention(
            embed_dim=self.embed_dim,
            num_heads=config.encoder_attention_heads,
            layer_id=layer_id,
            quant_config=quant_config,
            is_encoder=True,
        )
        self.self_attn_layer_norm = torch.nn.LayerNorm(self.embed_dim)

        self.activation_fn = get_act_fn(
            config.activation_function, quant_config=quant_config
        )

        self.fc1 = ColumnParallelLinear(self.embed_dim, config.encoder_ffn_dim)
        self.fc2 = RowParallelLinear(config.encoder_ffn_dim, self.embed_dim)
        self.final_layer_norm = torch.nn.LayerNorm(self.embed_dim)
```
**EN:** This method implements `__init__(config: ..., layer_id: ...=..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ...=..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 162-188: Method `WhisperEncoderLayer.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:

        residual = hidden_states
        hidden_states = self.self_attn_layer_norm(hidden_states)
        hidden_states = self.self_attn(hidden_states, forward_batch)

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
        return hidden_states
```
**EN:** This method implements `forward(hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 191-191: Class `WhisperDecoderLayer` overview
```python
class WhisperDecoderLayer(torch.nn.Module):
```
**EN:** Defines `WhisperDecoderLayer` as a reusable runtime type derived from torch.nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `WhisperDecoderLayer`，其继承关系为 torch.nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 192-229: Method `WhisperDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: WhisperConfig,
        layer_id: Optional[int] = None,
        quant_config: Optional[QuantizationConfig] = None,
    ):
        super().__init__()
        self.embed_dim = config.d_model

        # Offset decoder layer IDs to avoid overlap with encoder layers
        decoder_self_attn_layer_id = config.encoder_layers + layer_id
        decoder_cross_attn_layer_id = (
            config.encoder_layers + config.decoder_layers + layer_id
        )

        self.self_attn = WhisperAttention(
            embed_dim=self.embed_dim,
            num_heads=config.decoder_attention_heads,
            layer_id=decoder_self_attn_layer_id,
            quant_config=quant_config,
        )

        self.activation_fn = get_act_fn(
            config.activation_function, quant_config=quant_config
        )

        self.self_attn_layer_norm = torch.nn.LayerNorm(self.embed_dim)
        self.encoder_attn = WhisperAttention(
            embed_dim=self.embed_dim,
            num_heads=config.decoder_attention_heads,
            layer_id=decoder_cross_attn_layer_id,
            quant_config=quant_config,
            is_cross_attention=True,
        )
        self.encoder_attn_layer_norm = torch.nn.LayerNorm(self.embed_dim)
        self.fc1 = ColumnParallelLinear(self.embed_dim, config.decoder_ffn_dim)
        self.fc2 = RowParallelLinear(config.decoder_ffn_dim, self.embed_dim)
        self.final_layer_norm = torch.nn.LayerNorm(self.embed_dim)
```
**EN:** This method implements `__init__(config: ..., layer_id: ...=..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ...=..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 231-258: Method `WhisperDecoderLayer.forward`
```python
    def forward(
        self,
        decoder_hidden_states: torch.Tensor,
        encoder_hidden_states: Optional[torch.Tensor],
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:

        residual = decoder_hidden_states
        decoder_hidden_states = self.self_attn_layer_norm(decoder_hidden_states)
        decoder_hidden_states = self.self_attn(decoder_hidden_states, forward_batch)
        decoder_hidden_states = residual + decoder_hidden_states

        residual = decoder_hidden_states
        decoder_hidden_states = self.encoder_attn_layer_norm(decoder_hidden_states)
        decoder_hidden_states = self.encoder_attn(
            decoder_hidden_states, forward_batch, encoder_hidden_states
        )
        decoder_hidden_states = residual + decoder_hidden_states

        residual = decoder_hidden_states
        decoder_hidden_states = self.final_layer_norm(decoder_hidden_states)
        decoder_hidden_states, _ = self.fc1(decoder_hidden_states)
        decoder_hidden_states = self.activation_fn(decoder_hidden_states)
        decoder_hidden_states, _ = self.fc2(decoder_hidden_states)

        decoder_hidden_states = residual + decoder_hidden_states

        return decoder_hidden_states
```
**EN:** This method implements `forward(decoder_hidden_states: ..., encoder_hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(decoder_hidden_states: ..., encoder_hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 261-262: Class `WhisperEncoder` overview
```python
class WhisperEncoder(torch.nn.Module):
```
**EN:** Defines `WhisperEncoder` as a reusable runtime type derived from torch.nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `WhisperEncoder`，其继承关系为 torch.nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 263-287: Method `WhisperEncoder.__init__`
```python
    def __init__(
        self, config: WhisperConfig, quant_config: Optional[QuantizationConfig] = None
    ):
        super().__init__()

        embed_dim = config.d_model
        self.embed_scale = embed_dim**-0.5 if config.scale_embedding else 1.0

        self.conv1 = torch.nn.Conv1d(
            config.num_mel_bins, embed_dim, kernel_size=3, padding=1
        )
        self.conv2 = torch.nn.Conv1d(
            embed_dim, embed_dim, kernel_size=3, stride=2, padding=1
        )
        self.embed_positions = torch.nn.Embedding(
            config.max_source_positions, embed_dim
        )

        self.layers = torch.nn.ModuleList(
            [
                WhisperEncoderLayer(config, id, quant_config)
                for id in range(config.encoder_layers)
            ]
        )
        self.layer_norm = torch.nn.LayerNorm(config.d_model)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 289-310: Method `WhisperEncoder.forward`
```python
    def forward(
        self,
        input_features: torch.Tensor,
        position_ids: torch.Tensor,
        forward_batch: ForwardBatch,
    ):
        device = self.conv1.weight.device
        input_features = input_features.to(device=device)
        position_ids = position_ids.to(device=device)

        inputs_embeds = torch.nn.functional.gelu(self.conv1(input_features))
        inputs_embeds = torch.nn.functional.gelu(self.conv2(inputs_embeds))

        inputs_embeds = inputs_embeds.mT

        hidden_states = inputs_embeds + self.embed_positions(position_ids)

        for encoder_layer in self.layers:
            hidden_states = encoder_layer(hidden_states, forward_batch)

        hidden_states = self.layer_norm(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(input_features: ..., position_ids: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_features: ..., position_ids: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 313-314: Class `WhisperDecoder` overview
```python
class WhisperDecoder(torch.nn.Module):
```
**EN:** Defines `WhisperDecoder` as a reusable runtime type derived from torch.nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `WhisperDecoder`，其继承关系为 torch.nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 315-337: Method `WhisperDecoder.__init__`
```python
    def __init__(
        self, config: WhisperConfig, quant_config: Optional[QuantizationConfig] = None
    ):
        super().__init__()
        self.max_target_positions = config.max_target_positions
        self.max_source_positions = config.max_source_positions
        self.embed_scale = config.d_model**-0.5 if config.scale_embedding else 1.0

        self.embed_tokens = torch.nn.Embedding(
            config.vocab_size, config.d_model, padding_idx=config.pad_token_id
        )
        self.embed_positions = torch.nn.Embedding(
            self.max_target_positions, config.d_model
        )

        self.layers = torch.nn.ModuleList(
            [
                WhisperDecoderLayer(config, layer_idx, quant_config)
                for layer_idx in range(config.decoder_layers)
            ]
        )

        self.layer_norm = torch.nn.LayerNorm(config.d_model)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 339-358: Method `WhisperDecoder.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        encoder_hidden_states: Optional[torch.Tensor],
        forward_batch: ForwardBatch,
        position_ids=None,
    ):
        inputs_embeds = self.embed_tokens(input_ids)
        position_ids = position_ids.clamp(max=self.max_target_positions - 1)
        positions = self.embed_positions(position_ids)
        hidden_states = inputs_embeds + positions.to(inputs_embeds.device)

        for decoder_layer in self.layers:
            hidden_states = decoder_layer(
                hidden_states, encoder_hidden_states, forward_batch
            )

        hidden_states = self.layer_norm(hidden_states)

        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., encoder_hidden_states: ..., forward_batch: ..., position_ids=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., encoder_hidden_states: ..., forward_batch: ..., position_ids=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 361-362: Class `WhisperForConditionalGeneration` overview
```python
class WhisperForConditionalGeneration(torch.nn.Module):
```
**EN:** Defines `WhisperForConditionalGeneration` as a reusable runtime type derived from torch.nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `WhisperForConditionalGeneration`，其继承关系为 torch.nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 363-373: Method `WhisperForConditionalGeneration.__init__`
```python
    def __init__(
        self, config: WhisperConfig, quant_config: Optional[QuantizationConfig] = None
    ):
        super().__init__()
        self.encoder = WhisperEncoder(config, quant_config)
        self.decoder = WhisperDecoder(config, quant_config)
        self.proj_out = ParallelLMHead(
            config.vocab_size, config.d_model, quant_config=quant_config
        )
        self.logits_processor = LogitsProcessor(config)
        self.config = config
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 375-419: Method `WhisperForConditionalGeneration.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            (".self_attn.qkv_proj", ".self_attn.q_proj", "q"),
            (".self_attn.qkv_proj", ".self_attn.k_proj", "k"),
            (".self_attn.qkv_proj", ".self_attn.v_proj", "v"),
            (".encoder_attn.kv_proj", ".encoder_attn.k_proj", "k"),
            (".encoder_attn.kv_proj", ".encoder_attn.v_proj", "v"),
        ]

        params_dict = dict(self.named_parameters())
        weights_dict = dict(weights)

        # Whisper has no k_proj bias, create zeros
        for layer_idx in range(self.config.decoder_layers):
            layer_prefix = f"model.decoder.layers.{layer_idx}.encoder_attn."
            k_proj_key = layer_prefix + "k_proj.weight"
            if k_proj_key in weights_dict:
                k_proj_weight = weights_dict[k_proj_key]
                bias_key = layer_prefix + "k_proj.bias"
                if bias_key not in weights_dict:
                    weights_dict[bias_key] = torch.zeros(k_proj_weight.size(0))

        weights_dict["proj_out.weight"] = weights_dict[
            "model.decoder.embed_tokens.weight"
        ]

        for name, loaded_weight in weights_dict.items():
            name = name.replace("model.", "")

            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)
                if name not in params_dict:
                    break
                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 421-428: Method `WhisperForConditionalGeneration.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], mm_inputs: MultimodalInputs):
        # Prepend dummy encoder tokens so that prepare_encoder_info_extend
        # correctly allocates encoder KV cache locations in the KV pool.
        # These dummy tokens are stripped before the model forward receives input_ids.
        encoder_len = self.config.max_source_positions
        mm_inputs.num_image_tokens = encoder_len
        pad_ids = [0] * encoder_len
        return pad_ids + input_ids
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `WhisperForConditionalGeneration`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `WhisperForConditionalGeneration` 内部调用。

### Lines 430-486: Method `WhisperForConditionalGeneration.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        **kwargs: Any,
    ) -> LogitsProcessorOutput:
        dtype = self.encoder.conv1.weight.dtype

        # Run encoder for requests that haven't cached encoder output yet.
        # During decode or when encoder is already cached, encoder_hidden_states
        # is None and cross-attention reads KV from the pool via RadixAttention.
        encoder_hidden_states = None
        if not forward_batch.forward_mode.is_decode():
            mm_inputs_list = forward_batch.mm_inputs if forward_batch.mm_inputs else []
            encoder_cached_list = (
                forward_batch.encoder_cached if forward_batch.encoder_cached else []
            )

            # Collect features from all uncached requests for batched encoding
            features_to_encode = []
            for mm_input, cached in zip(mm_inputs_list, encoder_cached_list):
                if cached or mm_input is None or not mm_input.mm_items:
                    continue
                features = mm_input.mm_items[0].feature
                if features.ndim == 2:
                    features = features.unsqueeze(0)
                features_to_encode.append(features.to(dtype))

            if features_to_encode:
                # Batch all features and run encoder once instead of sequentially
                features_batch = torch.cat(features_to_encode, dim=0)
                encoder_len = features_batch.shape[-1] // 2
                encoder_position_ids = torch.arange(
                    encoder_len, device=features_batch.device
                )

                batched_output = self.encoder(
                    features_batch, encoder_position_ids, forward_batch
                )
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 489-489: Top-level assign
```python
EntryClass = [WhisperForConditionalGeneration]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影
- **EN:** Multimodal feature processing / **CN:** 多模态特征处理

## Dependencies / 依赖关系
- `typing.Any`
- `typing.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `transformers.WhisperConfig`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation.get_act_fn`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.logits_processor.LogitsProcessorOutput`
- `sglang.srt.layers.quantization.QuantizationConfig`
- `sglang.srt.layers.radix_attention.AttentionType`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
