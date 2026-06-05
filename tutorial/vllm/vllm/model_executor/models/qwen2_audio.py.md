# qwen2_audio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/qwen2_audio.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Qwen2 Audio multimodal model adapter used for inference in vLLM. The module docstring summarizes it as: "Inference-only Qwen2-Audio model compatible with HuggingFace weights." / 实现 Qwen2 Audio 在 vLLM 中用于推理的多模态模型适配器。 模块文档字符串还将其概括为：“Inference-only Qwen2-Audio model compatible with HuggingFace weights。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-69)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# Copyright 2024 The Qwen team.
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
#
# ... omitted for brevity ...
)
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .utils import AutoWeightsLoader, init_vllm_registered_model, maybe_prefix


# # === Audio Inputs === #
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, transformers, transformers.models.qwen2_audio supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.multimodal connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, transformers, transformers.models.qwen2_audio 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.multimodal 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Class `Qwen2AudioFeatureInputs` (lines 70-86)
```python
class Qwen2AudioFeatureInputs(TensorSchema):
    """
    Dimensions:
        - na: Number of audios
        - nmb: Number of mel bins
    """

    type: Literal["audio_features"]
    input_features: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("na", "nmb", 3000),
    ]

    feature_attention_mask: Annotated[
        torch.Tensor,
        TensorShape("na", 3000),
    ]
```
**EN:** Defines `Qwen2AudioFeatureInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - na: Number of audios - nmb: Number of mel bins."
**CN:** 定义 `Qwen2AudioFeatureInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - na: Number of audios - nmb: Number of mel bins。”

### Class `Qwen2AudioEmbeddingInputs` (lines 89-103)
```python
class Qwen2AudioEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size
        - naf: Number of audio features
        - hs: Hidden size (must match the hidden size of language model
          backbone)
    """

    type: Literal["audio_embeds"] = "audio_embeds"

    audio_embeds: Annotated[
        list[torch.Tensor],
        TensorShape("bn", "naf", "hs", dynamic_dims={"naf"}),
    ]
```
**EN:** Defines `Qwen2AudioEmbeddingInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size - naf: Number of audio features - hs: Hidden size (must match the hidden size of language model backbone)."
**CN:** 定义 `Qwen2AudioEmbeddingInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size - naf: Number of audio features - hs: Hidden size (must match the hidden size of language model backbone)。”

### Class `Qwen2AudioMultiModalProjector` (lines 111-118)
```python
class Qwen2AudioMultiModalProjector(nn.Module):
    def __init__(self, audio_hidden_size: int, text_hidden_size: int):
        super().__init__()
        self.linear = nn.Linear(audio_hidden_size, text_hidden_size, bias=True)

    def forward(self, audio_features):
        hidden_states = self.linear(audio_features)
        return hidden_states
```
**EN:** Defines `Qwen2AudioMultiModalProjector`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen2AudioMultiModalProjector`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Function `_get_feat_extract_output_lengths` (lines 122-125)
```python
def _get_feat_extract_output_lengths(input_lengths: torch.Tensor):
    feat_lengths = (input_lengths - 1) // 2 + 1
    output_lengths = (feat_lengths - 2) // 2 + 1
    return feat_lengths, output_lengths
```
**EN:** The function `_get_feat_extract_output_lengths` helps provide a reusable helper for the surrounding model code. Its main inputs are `input_lengths`.
**CN:** 函数 `_get_feat_extract_output_lengths` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `input_lengths`。

### Function `_qwen2audio_field_config` (lines 128-133)
```python
def _qwen2audio_field_config(hf_inputs: Mapping[str, torch.Tensor]):
    return dict(
        audio_embeds=MultiModalFieldConfig.batched("audio"),
        input_features=MultiModalFieldConfig.batched("audio"),
        feature_attention_mask=MultiModalFieldConfig.batched("audio"),
    )
```
**EN:** The function `_qwen2audio_field_config` helps provide a reusable helper for the surrounding model code. Its main inputs are `hf_inputs`.
**CN:** 函数 `_qwen2audio_field_config` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `hf_inputs`。

### Class `Qwen2AudioMultiModalDataParser` (lines 136-149)
```python
class Qwen2AudioMultiModalDataParser(MultiModalDataParser):
    def _parse_audio_data(
        self,
        data: dict[str, torch.Tensor] | ModalityData[AudioItem],
    ) -> ModalityDataItems[Any, Any] | None:
        if isinstance(data, dict):
            return DictEmbeddingItems(
                data,
                modality="audio",
                required_fields={"audio_embeds"},
                fields_factory=_qwen2audio_field_config,
            )

        return super()._parse_audio_data(data)
```
**EN:** Defines `Qwen2AudioMultiModalDataParser`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from MultiModalDataParser. Key methods such as `_parse_audio_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen2AudioMultiModalDataParser`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 MultiModalDataParser。 `_parse_audio_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen2AudioProcessingInfo` (lines 152-199)
```python
class Qwen2AudioProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(Qwen2AudioConfig)

    def get_hf_processor(self, **kwargs: object) -> Qwen2AudioProcessor:
        return self.ctx.get_hf_processor(Qwen2AudioProcessor, **kwargs)

    def get_feature_extractor(self, **kwargs: object) -> WhisperFeatureExtractor:
        hf_processor = self.get_hf_processor(**kwargs)
        feature_extractor = hf_processor.feature_extractor  # type: ignore
        assert isinstance(feature_extractor, WhisperFeatureExtractor)
        return feature_extractor

    def get_data_parser(self):
        feature_extractor = self.get_feature_extractor()

        return Qwen2AudioMultiModalDataParser(
            target_sr=feature_extractor.sampling_rate,
            target_channels=self.get_target_channels(),
            expected_hidden_size=self._get_expected_hidden_size(),
        )

    def get_target_channels(self) -> int:
        """Return target audio channels for Qwen2 Audio models (mono)."""
        return 1

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"audio": None}

    def get_mm_max_tokens_per_item(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int] | None = None,
    ) -> Mapping[str, int]:
        mm_counts = mm_counts or {}
        if mm_counts.get("audio", 0) <= 0:
            return {}

        feature_extractor = self.get_feature_extractor()
        chunk_length = min(feature_extractor.chunk_length, 30)
        audio_len = int(chunk_length * feature_extractor.sampling_rate)
        hop_length = feature_extractor.hop_length
        max_mel_seq_len = audio_len // hop_length

        input_lengths = torch.tensor([max_mel_seq_len], dtype=torch.long)
        _, output_lengths = _get_feat_extract_output_lengths(input_lengths)

        return {"audio": int(output_lengths.item())}
```
**EN:** Defines `Qwen2AudioProcessingInfo`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from BaseProcessingInfo. Key methods such as `get_hf_config`, `get_hf_processor`, `get_feature_extractor`, `get_data_parser`, `get_target_channels` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen2AudioProcessingInfo`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 BaseProcessingInfo。 `get_hf_config`, `get_hf_processor`, `get_feature_extractor`, `get_data_parser`, `get_target_channels` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen2AudioDummyInputsBuilder` (lines 202-233)
```python
class Qwen2AudioDummyInputsBuilder(BaseDummyInputsBuilder[Qwen2AudioProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_audios = mm_counts.get("audio", 0)

        hf_processor = self.info.get_hf_processor()
        audio_token = hf_processor.audio_token
        audio_bos_token = hf_processor.audio_bos_token
        audio_eos_token = hf_processor.audio_eos_token

        return (audio_bos_token + audio_token + audio_eos_token) * num_audios

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        feature_extractor = self.info.get_feature_extractor()

        sampling_rate = feature_extractor.sampling_rate
        audio_len = feature_extractor.chunk_length * sampling_rate
        num_audios = mm_counts.get("audio", 0)

        audio_overrides = mm_options.get("audio")

        return {
            "audio": self._get_dummy_audios(
                length=audio_len,
                num_audios=num_audios,
                overrides=audio_overrides,
            )
        }
```
**EN:** Defines `Qwen2AudioDummyInputsBuilder`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from BaseDummyInputsBuilder[Qwen2AudioProcessingInfo]. Key methods such as `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen2AudioDummyInputsBuilder`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 BaseDummyInputsBuilder[Qwen2AudioProcessingInfo]。 `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen2AudioMultiModalProcessor` (lines 236-323)
```python
class Qwen2AudioMultiModalProcessor(BaseMultiModalProcessor[Qwen2AudioProcessingInfo]):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, Any],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        # NOTE - we rename audios -> audio in mm data because transformers has
        # deprecated audios for the qwen2audio processor and will remove
        # support for it in transformers 4.54.
        audios = mm_data.pop("audios", [])
        if audios:
            mm_data["audio"] = audios

        # Text-only input not supported in composite processor
        if not mm_data.get("audio", []):
            prompt_ids = self.info.get_tokenizer().encode(prompt)
            prompt_ids = self._apply_hf_processor_tokens_only(prompt_ids)
            return BatchFeature(dict(input_ids=[prompt_ids]), tensor_type="pt")
# ... omitted for brevity ...
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return _qwen2audio_field_config(hf_inputs)
# ... omitted for brevity ...
    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        processor = self.info.get_hf_processor(**hf_processor_mm_kwargs)
        audio_token_id = processor.audio_token_id

        out_mm_data = out_mm_kwargs.get_data()
        feature_attention_mask = out_mm_data.get("feature_attention_mask")
        if feature_attention_mask is None:
            audio_output_lengths = []
        else:
            assert isinstance(feature_attention_mask, torch.Tensor)
            _, audio_output_lens = _get_feat_extract_output_lengths(
                feature_attention_mask.sum(-1)
            )
```
**EN:** Defines `Qwen2AudioMultiModalProcessor`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from BaseMultiModalProcessor[Qwen2AudioProcessingInfo]. Key methods such as `_call_hf_processor`, `_get_mm_fields_config`, `_get_prompt_updates` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen2AudioMultiModalProcessor`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 BaseMultiModalProcessor[Qwen2AudioProcessingInfo]。 `_call_hf_processor`, `_get_mm_fields_config`, `_get_prompt_updates` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen2AudioForConditionalGeneration` (lines 326-485)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Qwen2AudioMultiModalProcessor,
    info=Qwen2AudioProcessingInfo,
    dummy_inputs=Qwen2AudioDummyInputsBuilder,
)
class Qwen2AudioForConditionalGeneration(nn.Module, SupportsMultiModal, SupportsPP):
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("audio"):
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = config
        self.multimodal_config = multimodal_config
        self.quant_config = quant_config

        with self._mark_tower_model(vllm_config, "audio"):
            self.audio_tower = Qwen2AudioEncoder(config.audio_config)
            self.multi_modal_projector = Qwen2AudioMultiModalProjector(
                config.audio_config.d_model, config.text_config.hidden_size
            )

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=config.text_config,
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        if intermediate_tensors is not None:
            inputs_embeds = None

        hidden_states = self.language_model.model(
            input_ids, positions, intermediate_tensors, inputs_embeds=inputs_embeds
        )
        return hidden_states
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.language_model.compute_logits(hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights)
```
**EN:** Defines `Qwen2AudioForConditionalGeneration`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `_parse_and_validate_audio_input`, `_process_audio_input`, `embed_multimodal` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen2AudioForConditionalGeneration`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `_parse_and_validate_audio_input`, `_process_audio_input`, `embed_multimodal` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Quantized weights**
  - **EN:** Quantization hooks allow the model to load compressed checkpoints or specialized linear layers.
  - **CN:** 量化钩子使模型能够加载压缩检查点或专用线性层。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。

## Dependencies / 依赖关系
- **Standard library**: collections.abc, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, transformers, transformers.models.qwen2_audio, transformers.models.whisper
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.multimodal, vllm.multimodal.inputs, vllm.multimodal.parse, vllm.multimodal.processing, vllm.sequence
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
