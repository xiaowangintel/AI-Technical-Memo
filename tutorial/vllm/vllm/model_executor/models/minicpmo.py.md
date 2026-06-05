# minicpmo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/minicpmo.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Minicpmo multimodal model adapter used for inference in vLLM. The module docstring summarizes it as: "Inference-only MiniCPM-O model compatible with HuggingFace weights." / 实现 Minicpmo 在 vLLM 中用于推理的多模态模型适配器。 模块文档字符串还将其概括为：“Inference-only MiniCPM-O model compatible with HuggingFace weights。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-75)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# Adapted from
# https://github.com/huggingface/transformers/blob/v4.28.0/src/transformers/models/llama/modeling_llama.py
# Copyright 2023 The vLLM team.
# Copyright 2022 EleutherAI and the HuggingFace Inc. team. All rights reserved.
#
# This code is based on EleutherAI's GPT-NeoX library and the GPT-NeoX
# and OPT implementations in this library. It has been modified from its
# original forms to accommodate minor architectural differences compared
# to GPT-NeoX and OPT used by the Meta AI team that trained the model.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
# ... omitted for brevity ...
    PromptUpdateDetails,
)
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .minicpmv import (
    _MAX_FRAMES_PER_VIDEO,
    MiniCPMV2_6,
    MiniCPMV4_5,
    MiniCPMVDummyInputsBuilder,
    MiniCPMVMultiModalDataParser,
    MiniCPMVMultiModalProcessor,
    MiniCPMVProcessingInfo,
    _minicpmv_field_config,
)
from .utils import AutoWeightsLoader, cast_overflow_tensors, maybe_prefix
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, transformers, transformers.modeling_outputs, transformers.models.whisper.modeling_whisper supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.multimodal connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, transformers, transformers.modeling_outputs, transformers.models.whisper.modeling_whisper 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.multimodal 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `CPU_DEVICE` (lines 76-76)
```python
CPU_DEVICE = torch.device("cpu")
```
**EN:** This assignment block centers on `CPU_DEVICE` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `CPU_DEVICE` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Class `MiniCPMOAudioFeatureInputs` (lines 120-149)
```python
class MiniCPMOAudioFeatureInputs(TensorSchema):
    """
    Dimensions:
        - bns: Batch size * number of audios * number of slices
        - bn: Batch size * number of audios
        - c: Number of channels
        - l: Length
        - s: Number of slices
    """

    type: Literal["audio_features"] = "audio_features"

    audio_features: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("bns", "c", "l", dynamic_dims={"l"}),
    ]
    """
    Slice here means chunk. Audio that is too long will be split into slices,
    which is the same as image. Padding is used therefore `audio_features` is 
    `torch.Tensor`.
    """

    audio_feature_lens: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("bn", "s"),
    ]
    """
    This should be feature length of each audio slice, 
    which equals to `audio_features.shape[-1]`
    """
```
**EN:** Defines `MiniCPMOAudioFeatureInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - bns: Batch size * number of audios * number of slices - bn: Batch size * number of audios - c: Number of channels - l: Length - s: Number of slices."
**CN:** 定义 `MiniCPMOAudioFeatureInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - bns: Batch size * number of audios * number of slices - bn: Batch size * number of audios - c: Number of channels - l: Length - s: Number of slices。”

### Class `MiniCPMOAudioEmbeddingInputs` (lines 152-167)
```python
class MiniCPMOAudioEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of audios
        - s: Number of slices
        - h: Hidden size (must match language model backbone)

    Length of each slice may vary, so pass it as a list.
    """

    type: Literal["audio_embeds"] = "audio_embeds"

    audio_embeds: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("bn", "s", "h", dynamic_dims={"s"}),
    ]
```
**EN:** Defines `MiniCPMOAudioEmbeddingInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of audios - s: Number of slices - h: Hidden size (must match language model backbone) Length of each slice may vary, so pass it as a list."
**CN:** 定义 `MiniCPMOAudioEmbeddingInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of audios - s: Number of slices - h: Hidden size (must match language model backbone) Length of each slice may vary, so pass it as a list。”

### Class `MiniCPMOAudioEmbeddingItems` (lines 184-198)
```python
class MiniCPMOAudioEmbeddingItems(DictEmbeddingItems):
    def __init__(
        self,
        data: Mapping[str, torch.Tensor],
        fields_factory: Callable[
            [Mapping[str, torch.Tensor]],
            Mapping[str, MultiModalFieldConfig],
        ],
    ) -> None:
        super().__init__(
            data,
            modality="image",
            required_fields={"audio_embeds"},
            fields_factory=fields_factory,
        )
```
**EN:** Defines `MiniCPMOAudioEmbeddingItems`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from DictEmbeddingItems. Key methods such as `__init__` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MiniCPMOAudioEmbeddingItems`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 DictEmbeddingItems。 `__init__` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MiniCPMOMultiModalDataParser` (lines 201-212)
```python
class MiniCPMOMultiModalDataParser(MiniCPMVMultiModalDataParser):
    def _parse_audio_data(
        self,
        data: dict[str, torch.Tensor] | ModalityData[AudioItem],
    ) -> ModalityDataItems[Any, Any] | None:
        if isinstance(data, dict):
            return MiniCPMOAudioEmbeddingItems(
                data,
                fields_factory=_minicpmo_field_config,
            )

        return super()._parse_audio_data(data)
```
**EN:** Defines `MiniCPMOMultiModalDataParser`, a supporting module used by the surrounding model implementation. It inherits from MiniCPMVMultiModalDataParser. Key methods such as `_parse_audio_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MiniCPMOMultiModalDataParser`，它是一个被周边模型实现复用的支撑模块。 它继承自 MiniCPMVMultiModalDataParser。 `_parse_audio_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MultiModalProjector` (lines 438-448)
```python
class MultiModalProjector(nn.Module):
    def __init__(self, in_dim: int, out_dim: int):
        super().__init__()
        self.linear1 = nn.Linear(in_features=in_dim, out_features=out_dim, bias=True)
        self.relu = nn.ReLU()
        self.linear2 = nn.Linear(in_features=out_dim, out_features=out_dim, bias=True)

    def forward(self, audio_features: torch.Tensor) -> torch.Tensor:
        hidden_states = self.relu(self.linear1(audio_features))
        hidden_states = self.linear2(hidden_states)
        return hidden_states
```
**EN:** Defines `MultiModalProjector`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MultiModalProjector`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MiniCPMWhisperEncoderLayer` (lines 451-503)
```python
class MiniCPMWhisperEncoderLayer(nn.Module):
    def __init__(self, config: WhisperConfig, layer_idx: int):
        super().__init__()
        self.embed_dim = config.d_model
        self.self_attn = WhisperAttention(
            embed_dim=self.embed_dim,
            num_heads=config.encoder_attention_heads,
            dropout=config.attention_dropout,
            config=config,
            layer_idx=layer_idx,
        )
        self.self_attn_layer_norm = nn.LayerNorm(self.embed_dim)
        self.dropout = config.dropout
        self.activation_fn = ACT2FN[config.activation_function]
        self.activation_dropout = config.activation_dropout
        self.fc1 = nn.Linear(self.embed_dim, config.encoder_ffn_dim)
        self.fc2 = nn.Linear(config.encoder_ffn_dim, self.embed_dim)
        self.final_layer_norm = nn.LayerNorm(self.embed_dim)

    def forward(
        self,
        hidden_states: torch.Tensor,
        attention_mask: torch.Tensor,
    ) -> torch.Tensor:
        residual = hidden_states
        hidden_states = self.self_attn_layer_norm(hidden_states)
        hidden_states, _ = self.self_attn(
            hidden_states=hidden_states,
            attention_mask=attention_mask,
        )
        hidden_states = nn.functional.dropout(
            hidden_states, p=self.dropout, training=self.training
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

        if hidden_states.dtype == torch.float16:
            hidden_states = cast_overflow_tensors(hidden_states)

        outputs = (hidden_states,)

        return outputs
```
**EN:** Defines `MiniCPMWhisperEncoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MiniCPMWhisperEncoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MiniCPMWhisperEncoder` (lines 506-567)
```python
class MiniCPMWhisperEncoder(WhisperEncoder):
    def __init__(self, config: WhisperConfig):
        super().__init__(config)
        self.layers = nn.ModuleList(
            [
                MiniCPMWhisperEncoderLayer(config, layer_idx=i)
                for i in range(config.encoder_layers)
            ]
        )

    def forward(
        self,
        input_features: torch.Tensor,
        attention_mask: torch.Tensor | None = None,
    ) -> BaseModelOutputWithPast:
        # Ignore copy
        input_features = input_features.to(
            dtype=self.conv1.weight.dtype, device=self.conv1.weight.device
        )

        inputs_embeds = nn.functional.gelu(self.conv1(input_features))
        inputs_embeds = nn.functional.gelu(self.conv2(inputs_embeds))

        inputs_embeds = inputs_embeds.permute(0, 2, 1)

        embed_pos = self.embed_positions.weight

        embed_pos = embed_pos[: inputs_embeds.shape[1], :]

        hidden_states = inputs_embeds + embed_pos
# ... omitted for brevity ...

        encoder_states = ()

        for idx, encoder_layer in enumerate(self.layers):
            encoder_states = encoder_states + (hidden_states,)
            to_drop = False
            if self.training:
                dropout_probability = torch.rand([])
                if dropout_probability < self.layerdrop:  # skip the layer
                    to_drop = True

            # Ignore copy
            if to_drop:
                layer_outputs = (None, None)
            else:
                layer_outputs = encoder_layer(
                    hidden_states,
                    attention_mask,
                )

                hidden_states = layer_outputs[0]

        hidden_states = self.layer_norm(hidden_states)
        encoder_states = encoder_states + (hidden_states,)

        return BaseModelOutputWithPast(
            last_hidden_state=hidden_states,
            hidden_states=encoder_states,
        )
```
**EN:** Defines `MiniCPMWhisperEncoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from WhisperEncoder. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MiniCPMWhisperEncoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 WhisperEncoder。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MiniCPMOBaseModel` (lines 570-812)
```python
class MiniCPMOBaseModel:
    """Base mixin class for MiniCPM-O models with audio support."""

    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(vllm_config=vllm_config, prefix=prefix)

        with self._mark_tower_model(vllm_config, "audio"):
            self.apm = self.init_audio_module(
                vllm_config=vllm_config, prefix=maybe_prefix(prefix, "apm")
            )
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(self, skip_prefixes=["tts"])
        return loader.load_weights(weights)
# ... omitted for brevity ...
    def _parse_and_validate_audio_input(
        self, **kwargs: object
    ) -> MiniCPMOAudioInputs | None:
        audio_features = kwargs.pop("audio_features", None)
        audio_embeds = kwargs.pop("audio_embeds", None)

        if audio_features is None and audio_embeds is None:
            return None

        if audio_embeds is not None:
            return MiniCPMOAudioEmbeddingInputs(
                type="audio_embeds",
                audio_embeds=audio_embeds,
            )

        audio_feature_lens = kwargs.pop("audio_feature_lens")

        return MiniCPMOAudioFeatureInputs(
            type="audio_features",
# ... omitted for brevity ...
    def _parse_and_validate_multimodal_inputs(self, **kwargs: object) -> dict:
        modalities = super()._parse_and_validate_multimodal_inputs(**kwargs)

        # Preserve the order of modalities if there are multiple of them
        # from the order of kwargs.
        for input_key in kwargs:
            if (
                input_key in ("audio_features", "audio_embeds")
                and "audios" not in modalities
            ):
                modalities["audios"] = self._parse_and_validate_audio_input(**kwargs)

        return modalities
```
**EN:** Defines `MiniCPMOBaseModel`, a backbone model container that orchestrates embeddings, layers, and output heads. Key methods such as `get_placeholder_str`, `__init__`, `init_audio_module`, `load_weights`, `subsequent_chunk_mask` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Base mixin class for MiniCPM-O models with audio support."
**CN:** 定义 `MiniCPMOBaseModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 `get_placeholder_str`, `__init__`, `init_audio_module`, `load_weights`, `subsequent_chunk_mask` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Base mixin class for MiniCPM-O models with audio support。”

### Class `MiniCPMO` (lines 845-891)
```python
@MULTIMODAL_REGISTRY.register_processor(
    MiniCPMOMultiModalProcessor,
    info=MiniCPMOProcessingInfo,
    dummy_inputs=MiniCPMODummyInputsBuilder,
)
class MiniCPMO(MiniCPMOBaseModel, MiniCPMV2_6):
    """
    MiniCPM-O model with audio support.
    Different versions use different LLM backbones:
    - Version 2.6: Uses Qwen2
    - Version 4.5: Uses Qwen3
    """

    def __new__(cls, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config

        # Determine version from config
        if hasattr(config, "version"):
            try:
                version_str = str(config.version)
                version_parts = version_str.split(".")
                version = tuple(int(x) for x in version_parts[:2])
            except (ValueError, TypeError) as e:
                raise ValueError(
                    f"Invalid model version format in config: {config.version}. "
                    "Expected a dot-separated version string like '4.5'."
                ) from e
        else:
            # Default to 2.6 for backward compatibility
            version = (2, 6)

        # Dispatch class based on version
        instance_cls = _MINICPMO_SUPPORT_VERSION.get(version)
        if instance_cls is None:
            supported_versions = ", ".join(
                [f"{v[0]}.{v[1]}" for v in sorted(_MINICPMO_SUPPORT_VERSION.keys())]
            )
            raise ValueError(
                f"Currently, MiniCPMO only supports versions "
                f"{supported_versions}. Got version: {version}"
            )

        return instance_cls(vllm_config=vllm_config, prefix=prefix)

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        # This __init__ won't be called due to __new__ returning a different class
        pass
```
**EN:** Defines `MiniCPMO`, a supporting module used by the surrounding model implementation. It inherits from MiniCPMOBaseModel, MiniCPMV2_6. Key methods such as `__new__`, `__init__` show where construction, forward execution, or weight adaptation happens. Docstring hint: "MiniCPM-O model with audio support."
**CN:** 定义 `MiniCPMO`，它是一个被周边模型实现复用的支撑模块。 它继承自 MiniCPMOBaseModel、MiniCPMV2_6。 `__new__`, `__init__` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“MiniCPM-O model with audio support。”

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。
- **Pooling and encoding**
  - **EN:** This file exposes encoder-style behavior, pooled outputs, or sequence-level representations beyond causal decoding.
  - **CN:** 该文件提供编码器式行为、池化输出，或超出因果解码的序列表征。

## Dependencies / 依赖关系
- **Standard library**: os, collections.abc, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, transformers, transformers.modeling_outputs, transformers.models.whisper.modeling_whisper
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.multimodal, vllm.multimodal.inputs, vllm.multimodal.parse, vllm.multimodal.processing, vllm.utils.tensor_schema
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .minicpmv, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
