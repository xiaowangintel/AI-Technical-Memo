# funaudiochat.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/funaudiochat.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for funaudiochat, including encoder/decoder glue and vLLM runtime adaptation. / 面向 funaudiochat 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 14-56)
```python
from __future__ import annotations

from collections.abc import Iterable, Mapping, Sequence
from functools import cached_property
from typing import Any

import numpy as np
import torch
import torch.nn as nn
from transformers import PreTrainedTokenizerFast, WhisperFeatureExtractor
from transformers.activations import get_activation
from transformers.feature_extraction_utils import BatchFeature
from transformers.modeling_outputs import BaseModelOutput

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.attention.mm_encoder_attention import MMEncoderAttention
from vllm.model_executor.layers.linear import QKVParallelLinear, RowParallelLinear
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import (
    AudioProcessorItems,
    MultiModalDataItems,
    MultiModalDataParser,
)
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
    PromptUpdateDetails,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.import_utils import _has_module

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .utils import AutoWeightsLoader, init_vllm_registered_model, maybe_prefix
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `_SinusoidsPositionEmbedding` (lines 59-76)
```python
class _SinusoidsPositionEmbedding(nn.Module):
    def __init__(self, length: int, channels: int, max_timescale: float = 10000.0):
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
**EN:** Class `_SinusoidsPositionEmbedding` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__.
**CN:** 类 `_SinusoidsPositionEmbedding` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__。

### Method `_SinusoidsPositionEmbedding.__init__` (lines 60-76)
```python
    def __init__(self, length: int, channels: int, max_timescale: float = 10000.0):
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
**EN:** Method `_SinusoidsPositionEmbedding.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `_SinusoidsPositionEmbedding.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `FunAudioChatAudioAttention` (lines 79-192)
```python
class FunAudioChatAudioAttention(nn.Module):
    """Multi-headed attention used inside the continuous audio tower."""

    def __init__(self, config: Any):
        super().__init__()
        self.embed_dim = int(config.d_model)
        self.total_num_heads = int(config.encoder_attention_heads)
        self.dropout = float(getattr(config, "attention_dropout", 0.0))
        self.head_dim = self.embed_dim // self.total_num_heads
        self.num_key_value_groups = 1  # needed for eager attention
        self.config = config

        if self.head_dim * self.total_num_heads != self.embed_dim:
            raise ValueError(
                "embed_dim must be divisible by num_heads "
                f"(got embed_dim={self.embed_dim}, "
                f"num_heads={self.total_num_heads})."
            )
        self.scaling = self.head_dim**-0.5
        self.attention_dropout = 0.0
        self.is_decoder = False
        self.is_causal = False

        self.qkv_proj = QKVParallelLinear(
            self.embed_dim,
```
**EN:** Class `FunAudioChatAudioAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, load_weights, forward.
**CN:** 类 `FunAudioChatAudioAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, load_weights, forward。

### Method `FunAudioChatAudioAttention.__init__` (lines 82-124)
```python
    def __init__(self, config: Any):
        super().__init__()
        self.embed_dim = int(config.d_model)
        self.total_num_heads = int(config.encoder_attention_heads)
        self.dropout = float(getattr(config, "attention_dropout", 0.0))
        self.head_dim = self.embed_dim // self.total_num_heads
        self.num_key_value_groups = 1  # needed for eager attention
        self.config = config

        if self.head_dim * self.total_num_heads != self.embed_dim:
            raise ValueError(
                "embed_dim must be divisible by num_heads "
                f"(got embed_dim={self.embed_dim}, "
                f"num_heads={self.total_num_heads})."
            )
        self.scaling = self.head_dim**-0.5
        self.attention_dropout = 0.0
        self.is_decoder = False
        self.is_causal = False

        self.qkv_proj = QKVParallelLinear(
            self.embed_dim,
            self.head_dim,
            self.total_num_heads,
            bias=True,
        )
        self.num_heads = self.qkv_proj.num_heads
        self.num_kv_heads = self.qkv_proj.num_kv_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim

        self.attn = MMEncoderAttention(
            num_heads=self.num_heads,
            head_size=self.head_dim,
            scale=self.scaling,
            num_kv_heads=self.num_kv_heads,
            prefix="funaudiochat_audio_tower.attn",
        )
        self.out_proj = RowParallelLinear(
            self.embed_dim,
            self.embed_dim,
            bias=True,
        )
```
**EN:** Method `FunAudioChatAudioAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FunAudioChatAudioAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FunAudioChatAudioAttention.load_weights` (lines 126-161)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]

        params_dict = dict(self.named_parameters())
        with torch.no_grad():
            if self.qkv_proj.bias is not None:
                # HF FunAudioChat uses bias=False for k_proj. Ensure the missing
                # shard starts as zeros, while allowing q/v shards to load.
                self.qkv_proj.bias.zero_()

        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            for param_name, shard_name, shard_id in stacked_params_mapping:
                if shard_name not in name:
                    continue
                name = name.replace(shard_name, param_name)
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue

                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)

            loaded_params.add(name)

        return loaded_params
```
**EN:** Method `FunAudioChatAudioAttention.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `FunAudioChatAudioAttention.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `FunAudioChatAudioEncoderLayer` (lines 195-237)
```python
class FunAudioChatAudioEncoderLayer(nn.Module):
    def __init__(self, config: Any):
        super().__init__()
        self.embed_dim = int(config.d_model)
        self.self_attn = FunAudioChatAudioAttention(config)
        self.self_attn_layer_norm = nn.LayerNorm(self.embed_dim)
        self.dropout = float(config.dropout)
        self.activation_fn = get_activation(str(config.activation_function))
        self.activation_dropout = float(config.activation_dropout)
        self.fc1 = nn.Linear(self.embed_dim, int(config.encoder_ffn_dim))
        self.fc2 = nn.Linear(int(config.encoder_ffn_dim), self.embed_dim)
        self.final_layer_norm = nn.LayerNorm(self.embed_dim)

    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        attention_mask: torch.Tensor | None = None,
        **kwargs: object,
    ) -> tuple[torch.Tensor]:
        residual = hidden_states
        hidden_states = self.self_attn_layer_norm(hidden_states)
        hidden_states = self.self_attn(
            hidden_states=hidden_states,
            cu_seqlens=cu_seqlens,
```
**EN:** Class `FunAudioChatAudioEncoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `FunAudioChatAudioEncoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `FunAudioChatAudioEncoderLayer.__init__` (lines 196-206)
```python
    def __init__(self, config: Any):
        super().__init__()
        self.embed_dim = int(config.d_model)
        self.self_attn = FunAudioChatAudioAttention(config)
        self.self_attn_layer_norm = nn.LayerNorm(self.embed_dim)
        self.dropout = float(config.dropout)
        self.activation_fn = get_activation(str(config.activation_function))
        self.activation_dropout = float(config.activation_dropout)
        self.fc1 = nn.Linear(self.embed_dim, int(config.encoder_ffn_dim))
        self.fc2 = nn.Linear(int(config.encoder_ffn_dim), self.embed_dim)
        self.final_layer_norm = nn.LayerNorm(self.embed_dim)
```
**EN:** Method `FunAudioChatAudioEncoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FunAudioChatAudioEncoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FunAudioChatAudioEncoderLayer.forward` (lines 208-237)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        attention_mask: torch.Tensor | None = None,
        **kwargs: object,
    ) -> tuple[torch.Tensor]:
        residual = hidden_states
        hidden_states = self.self_attn_layer_norm(hidden_states)
        hidden_states = self.self_attn(
            hidden_states=hidden_states,
            cu_seqlens=cu_seqlens,
            attention_mask=attention_mask,
            **kwargs,
        )
        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.final_layer_norm(hidden_states)
        hidden_states = self.activation_fn(self.fc1(hidden_states))
        hidden_states = nn.functional.dropout(
            hidden_states, p=self.activation_dropout, training=self.training
        )
        hidden_states = self.fc2(hidden_states)
        hidden_states = nn.functional.dropout(
            hidden_states, p=self.dropout, training=self.training
        )
        hidden_states = residual + hidden_states

        return (hidden_states,)
```
**EN:** Method `FunAudioChatAudioEncoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FunAudioChatAudioEncoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `FunAudioChatAudioEncoder` (lines 240-480)
```python
class FunAudioChatAudioEncoder(nn.Module):
    """Continuous audio tower."""

    def __init__(self, config: Any):
        super().__init__()
        self.config = config

        embed_dim = int(config.d_model)
        self.num_mel_bins = int(config.num_mel_bins)
        self.max_source_positions = int(config.max_source_positions)
        self.embed_scale = (embed_dim**0.5) if bool(config.scale_embedding) else 1.0
        self.n_window = int(config.n_window)

        self.conv1 = nn.Conv1d(self.num_mel_bins, embed_dim, kernel_size=3, padding=1)
        self.conv2 = nn.Conv1d(embed_dim, embed_dim, kernel_size=3, stride=2, padding=1)
        self.layers = nn.ModuleList(
            [
                FunAudioChatAudioEncoderLayer(config)
                for _ in range(int(config.encoder_layers))
            ]
        )
        self.ln_post = nn.LayerNorm(embed_dim)
        self.avg_pooler = nn.AvgPool1d(2, stride=2)
        self.proj = nn.Linear(embed_dim, int(config.output_dim))
        self.positional_embedding = _SinusoidsPositionEmbedding(
```
**EN:** Class `FunAudioChatAudioEncoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, dtype, _prepare_attention_mask, forward, padded_and_mask_function, _get_feat_extract_output_lengths.
**CN:** 类 `FunAudioChatAudioEncoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, dtype, _prepare_attention_mask, forward, padded_and_mask_function, _get_feat_extract_output_lengths。

### Method `FunAudioChatAudioEncoder.__init__` (lines 243-269)
```python
    def __init__(self, config: Any):
        super().__init__()
        self.config = config

        embed_dim = int(config.d_model)
        self.num_mel_bins = int(config.num_mel_bins)
        self.max_source_positions = int(config.max_source_positions)
        self.embed_scale = (embed_dim**0.5) if bool(config.scale_embedding) else 1.0
        self.n_window = int(config.n_window)

        self.conv1 = nn.Conv1d(self.num_mel_bins, embed_dim, kernel_size=3, padding=1)
        self.conv2 = nn.Conv1d(embed_dim, embed_dim, kernel_size=3, stride=2, padding=1)
        self.layers = nn.ModuleList(
            [
                FunAudioChatAudioEncoderLayer(config)
                for _ in range(int(config.encoder_layers))
            ]
        )
        self.ln_post = nn.LayerNorm(embed_dim)
        self.avg_pooler = nn.AvgPool1d(2, stride=2)
        self.proj = nn.Linear(embed_dim, int(config.output_dim))
        self.positional_embedding = _SinusoidsPositionEmbedding(
            self.max_source_positions, embed_dim
        )

        # Present in HF weights even if unused during S2T.
        self.audio_bos_eos_token = nn.Embedding(2, int(config.output_dim))
```
**EN:** Method `FunAudioChatAudioEncoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FunAudioChatAudioEncoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FunAudioChatAudioEncoder.dtype` (lines 272-273)
```python
    @property
    def dtype(self) -> torch.dtype:
        return self.conv1.weight.dtype
```
**EN:** Method `FunAudioChatAudioEncoder.dtype` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FunAudioChatAudioEncoder.dtype` 封装了该模块中的一段可复用核心逻辑。

### Class `FunAudioChatDiscreteEncoder` (lines 483-548)
```python
class FunAudioChatDiscreteEncoder(nn.Module):
    """Discrete audio encoder (speech tokenizer -> grouped embeddings)."""

    def __init__(self, config: Any):
        super().__init__()
        self.padding_idx = int(config.pad_token_id)
        self.group_size = int(config.group_size)
        self.hidden_size = int(config.output_dim)
        self.continuous_features_mode = getattr(
            config, "continuous_features_mode", "add"
        )
        self.embed_tokens = nn.Embedding(
            int(config.codebook_size), self.hidden_size, self.padding_idx
        )
        self.output_matching = nn.Linear(self.hidden_size, self.hidden_size, bias=False)
        self.continual_output_matching = nn.Linear(
            self.hidden_size, self.hidden_size, bias=False
        )

    def forward(
        self,
        audio_ids: torch.Tensor,
        continuous_audio_features: torch.Tensor | None = None,
        continuous_audio_output_lengths: torch.Tensor | None = None,
        feature_exist_mask: torch.Tensor | None = None,
```
**EN:** Class `FunAudioChatDiscreteEncoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward, _get_feat_extract_output_lengths.
**CN:** 类 `FunAudioChatDiscreteEncoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward, _get_feat_extract_output_lengths。

### Method `FunAudioChatDiscreteEncoder.__init__` (lines 486-500)
```python
    def __init__(self, config: Any):
        super().__init__()
        self.padding_idx = int(config.pad_token_id)
        self.group_size = int(config.group_size)
        self.hidden_size = int(config.output_dim)
        self.continuous_features_mode = getattr(
            config, "continuous_features_mode", "add"
        )
        self.embed_tokens = nn.Embedding(
            int(config.codebook_size), self.hidden_size, self.padding_idx
        )
        self.output_matching = nn.Linear(self.hidden_size, self.hidden_size, bias=False)
        self.continual_output_matching = nn.Linear(
            self.hidden_size, self.hidden_size, bias=False
        )
```
**EN:** Method `FunAudioChatDiscreteEncoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FunAudioChatDiscreteEncoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FunAudioChatDiscreteEncoder.forward` (lines 502-542)
```python
    def forward(
        self,
        audio_ids: torch.Tensor,
        continuous_audio_features: torch.Tensor | None = None,
        continuous_audio_output_lengths: torch.Tensor | None = None,
        feature_exist_mask: torch.Tensor | None = None,
    ) -> torch.Tensor:
        del continuous_audio_output_lengths

        inputs_embeds = self.embed_tokens(audio_ids)
        hidden_states = inputs_embeds.reshape(
            inputs_embeds.shape[0], -1, self.group_size * self.hidden_size
        )
        hidden_states = hidden_states.reshape(
            hidden_states.shape[0], -1, self.group_size, self.hidden_size
        ).mean(dim=2)
        hidden_states = self.output_matching(hidden_states)

        if continuous_audio_features is not None:
            continuous_audio_features = continuous_audio_features.reshape(
                continuous_audio_features.shape[0],
                -1,
                self.group_size,
                self.hidden_size,
            ).mean(dim=2)
            continuous_audio_hidden_states = self.continual_output_matching(
                continuous_audio_features
            )

            if feature_exist_mask is None:
                feature_exist_mask = torch.ones(
                    (hidden_states.shape[0],),
                    dtype=torch.bool,
                    device=hidden_states.device,
                )
            if self.continuous_features_mode == "add":
                hidden_states[feature_exist_mask] += continuous_audio_hidden_states
            else:
                hidden_states[feature_exist_mask] = continuous_audio_hidden_states

        return hidden_states
```
**EN:** Method `FunAudioChatDiscreteEncoder.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FunAudioChatDiscreteEncoder.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `FunAudioChatProcessingInfo` (lines 551-598)
```python
class FunAudioChatProcessingInfo(BaseProcessingInfo):
    token_fps: int = 25

    @cached_property
    def feature_extractor(self) -> WhisperFeatureExtractor:
        return WhisperFeatureExtractor.from_pretrained(self.model_id)

    @cached_property
    def speech_tokenizer(self) -> PreTrainedTokenizerFast:
        return PreTrainedTokenizerFast.from_pretrained(
            self.model_id, subfolder="speech_tokenizer"
        )

    def get_feature_extractor(self) -> WhisperFeatureExtractor:
        return self.feature_extractor

    def get_speech_tokenizer(self) -> PreTrainedTokenizerFast:
        return self.speech_tokenizer

    def get_data_parser(self):
        return MultiModalDataParser(
            target_sr=int(self.feature_extractor.sampling_rate),
            target_channels=self.get_target_channels(),
            expected_hidden_size=self._get_expected_hidden_size(),
        )
```
**EN:** Class `FunAudioChatProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include feature_extractor, speech_tokenizer, get_feature_extractor, get_speech_tokenizer, get_data_parser, get_supported_mm_limits.
**CN:** 类 `FunAudioChatProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 feature_extractor, speech_tokenizer, get_feature_extractor, get_speech_tokenizer, get_data_parser, get_supported_mm_limits。

### Method `FunAudioChatProcessingInfo.feature_extractor` (lines 555-556)
```python
    @cached_property
    def feature_extractor(self) -> WhisperFeatureExtractor:
        return WhisperFeatureExtractor.from_pretrained(self.model_id)
```
**EN:** Method `FunAudioChatProcessingInfo.feature_extractor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FunAudioChatProcessingInfo.feature_extractor` 封装了该模块中的一段可复用核心逻辑。

### Method `FunAudioChatProcessingInfo.speech_tokenizer` (lines 559-562)
```python
    @cached_property
    def speech_tokenizer(self) -> PreTrainedTokenizerFast:
        return PreTrainedTokenizerFast.from_pretrained(
            self.model_id, subfolder="speech_tokenizer"
        )
```
**EN:** Method `FunAudioChatProcessingInfo.speech_tokenizer` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FunAudioChatProcessingInfo.speech_tokenizer` 封装了该模块中的一段可复用核心逻辑。

### Class `FunAudioChatDummyInputsBuilder` (lines 601-638)
```python
class FunAudioChatDummyInputsBuilder(
    BaseDummyInputsBuilder[FunAudioChatProcessingInfo]
):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_audios = mm_counts.get("audio", 0)
        return "<|audio_bos|><|AUDIO|><|audio_eos|>" * int(num_audios)

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        feature_extractor = self.info.get_feature_extractor()
        sampling_rate = int(feature_extractor.sampling_rate)

        # Dummy inputs are used for profiling; construct the worst-case audio
        # length that maximizes the number of encoder tokens.
        cfg = self.info.get_hf_config()
        audio_cfg = getattr(cfg, "audio_config", None)
        max_audio_tokens = int(getattr(audio_cfg, "max_source_positions", 1500))
        group_size = self.info.get_audio_group_size()
        token_fps = int(getattr(self.info, "token_fps", 25))
        target_num_frames = max(1, max_audio_tokens) * max(1, group_size)
        audio_len = max(
```
**EN:** Class `FunAudioChatDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[FunAudioChatProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `FunAudioChatDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[FunAudioChatProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `FunAudioChatDummyInputsBuilder.get_dummy_text` (lines 604-606)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_audios = mm_counts.get("audio", 0)
        return "<|audio_bos|><|AUDIO|><|audio_eos|>" * int(num_audios)
```
**EN:** Method `FunAudioChatDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FunAudioChatDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `FunAudioChatDummyInputsBuilder.get_dummy_mm_data` (lines 608-638)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        feature_extractor = self.info.get_feature_extractor()
        sampling_rate = int(feature_extractor.sampling_rate)

        # Dummy inputs are used for profiling; construct the worst-case audio
        # length that maximizes the number of encoder tokens.
        cfg = self.info.get_hf_config()
        audio_cfg = getattr(cfg, "audio_config", None)
        max_audio_tokens = int(getattr(audio_cfg, "max_source_positions", 1500))
        group_size = self.info.get_audio_group_size()
        token_fps = int(getattr(self.info, "token_fps", 25))
        target_num_frames = max(1, max_audio_tokens) * max(1, group_size)
        audio_len = max(
            1,
            (target_num_frames * sampling_rate + token_fps - 1) // token_fps,
        )
        num_audios = int(mm_counts.get("audio", 0))

        audio_overrides = mm_options.get("audio")
        return {
            "audio": self._get_dummy_audios(
                length=audio_len,
                num_audios=num_audios,
                overrides=audio_overrides,
            )
        }
```
**EN:** Method `FunAudioChatDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FunAudioChatDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `FunAudioChatMultiModalProcessor` (lines 641-781)
```python
class FunAudioChatMultiModalProcessor(
    BaseMultiModalProcessor[FunAudioChatProcessingInfo]
):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        tokenizer = self.info.get_tokenizer()
        input_ids = torch.tensor([tokenizer.encode(prompt, **tok_kwargs)])

        audios = mm_data.get("audios", [])
        if not audios:
            return BatchFeature({"input_ids": input_ids})

        feature_extractor = self.info.get_feature_extractor(**mm_kwargs)
        sr = int(feature_extractor.sampling_rate)
        min_samples = int(getattr(feature_extractor, "n_fft", 400) or 400)

        wavs: list[np.ndarray] = []
        speech_strs: list[str] = []

        speech_tokenizer = self.info.get_speech_tokenizer()
```
**EN:** Class `FunAudioChatMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[FunAudioChatProcessingInfo]. Key methods include _call_hf_processor, _hf_processor_applies_updates, _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `FunAudioChatMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[FunAudioChatProcessingInfo]。 关键方法包括 _call_hf_processor, _hf_processor_applies_updates, _get_mm_fields_config, _get_prompt_updates。

### Class `FunAudioChatForConditionalGeneration` (lines 789-1000)
```python
@MULTIMODAL_REGISTRY.register_processor(
    FunAudioChatMultiModalProcessor,
    info=FunAudioChatProcessingInfo,
    dummy_inputs=FunAudioChatDummyInputsBuilder,
)
class FunAudioChatForConditionalGeneration(nn.Module, SupportsMultiModal, SupportsPP):
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("audio"):
            return "<|audio_bos|><|AUDIO|><|audio_eos|>"

        raise ValueError("Only audio modality is supported")

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = config
        self.multimodal_config = multimodal_config
        self.quant_config = quant_config

        with self._mark_tower_model(vllm_config, "audio"):
            self.continuous_audio_tower = FunAudioChatAudioEncoder(config.audio_config)
            self.audio_tower = FunAudioChatDiscreteEncoder(config.audio_config)
```
**EN:** Class `FunAudioChatForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods include get_placeholder_str, __init__, _get_continuous_audio_features, embed_multimodal, forward, compute_logits.
**CN:** 类 `FunAudioChatForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 关键方法包括 get_placeholder_str, __init__, _get_continuous_audio_features, embed_multimodal, forward, compute_logits。

### Method `FunAudioChatForConditionalGeneration.get_placeholder_str` (lines 791-795)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("audio"):
            return "<|audio_bos|><|AUDIO|><|audio_eos|>"

        raise ValueError("Only audio modality is supported")
```
**EN:** Method `FunAudioChatForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FunAudioChatForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `FunAudioChatForConditionalGeneration.__init__` (lines 797-820)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = config
        self.multimodal_config = multimodal_config
        self.quant_config = quant_config

        with self._mark_tower_model(vllm_config, "audio"):
            self.continuous_audio_tower = FunAudioChatAudioEncoder(config.audio_config)
            self.audio_tower = FunAudioChatDiscreteEncoder(config.audio_config)

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=config.text_config,
                prefix=maybe_prefix(prefix, "language_model"),
                architectures=["Qwen3ForCausalLM"],
            )

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )
```
**EN:** Method `FunAudioChatForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FunAudioChatForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。
- **KV cache / KV 缓存**
  - **EN:** The runtime stores key/value state so decoding can reuse prior computation efficiently.
  - **CN:** 运行时会保存 key/value 状态，以便解码阶段高效复用历史计算。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from __future__ import annotations`, `from collections.abc import Iterable, Mapping, Sequence`, `from functools import cached_property`, `from typing import Any`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `import torch.nn as nn`, `from transformers import PreTrainedTokenizerFast, WhisperFeatureExtractor`, `from transformers.activations import get_activation`, `from transformers.feature_extraction_utils import BatchFeature`, `from transformers.modeling_outputs import BaseModelOutput`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.attention.mm_encoder_attention import MMEncoderAttention`, `from vllm.model_executor.layers.linear import QKVParallelLinear, RowParallelLinear`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import (`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`, `from vllm.utils.import_utils import _has_module`
- **Module note / 模块说明**: **EN:** Inference-only FunAudioChat model compatible with HuggingFace weights.  FunAudioChat is a Qwen3 text model augmented with: - a continuous audio encoder (Whisper-mel frontend + transformer) - a discrete audio encoder (speech tokenizer + projector)  In the HF implementation, audio features are scattered into `<|AUDIO|>` token positions via `inputs_embeds`, while `position_ids` (RoPE) remains standard 1D. **CN:** 模块文档字符串给出的原始说明是：Inference-only FunAudioChat model compatible with HuggingFace weights.  FunAudioChat is a Qwen3 text model augmented with: - a continuous audio encoder (Whisper-mel frontend + transformer) - a discrete audio encoder (speech tokenizer + projector)  In the HF implementation, audio features are scattered into `<|AUDIO|>` token positions via `inputs_embeds`, while `position_ids` (RoPE) remains standard 1D.。
