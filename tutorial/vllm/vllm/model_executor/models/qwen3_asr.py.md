# qwen3_asr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/qwen3_asr.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Qwen3 ASR multimodal model adapter used for inference in vLLM. The module docstring summarizes it as: "Inference-only Qwen3-ASR model." / 实现 Qwen3 ASR 在 vLLM 中用于推理的多模态模型适配器。 模块文档字符串还将其概括为：“Inference-only Qwen3-ASR model。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-90)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright 2026 The Qwen team.
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
# Unless required by applicable law or agreed to in writing, software
# ... omitted for brevity ...
    BaseDummyInputsBuilder,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
)
from vllm.sequence import IntermediateTensors
from vllm.tokenizers import cached_tokenizer_from_config
from vllm.transformers_utils.configs.qwen3_asr import (
    Qwen3ASRConfig,
    Qwen3ASRThinkerConfig,
)
from vllm.transformers_utils.processor import cached_processor_from_config
from vllm.transformers_utils.processors.qwen3_asr import (
    Qwen3ASRProcessor,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, transformers.feature_extraction_utils, transformers.models.whisper supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.config.speech_to_text, vllm.inputs connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, transformers.feature_extraction_utils, transformers.models.whisper 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.config.speech_to_text, vllm.inputs 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger, _ASR_TEXT_TAG` (lines 91-92)
```python
logger = init_logger(__name__)
_ASR_TEXT_TAG = "<asr_text>"
```
**EN:** This assignment block centers on `logger, _ASR_TEXT_TAG` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `logger, _ASR_TEXT_TAG` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Function `_get_feat_extract_output_lengths` (lines 95-101)
```python
def _get_feat_extract_output_lengths(input_lengths: torch.Tensor):
    input_lengths_leave = input_lengths % 100
    feat_lengths = (input_lengths_leave - 1) // 2 + 1
    output_lengths = (
        ((feat_lengths - 1) // 2 + 1 - 1) // 2 + 1 + (input_lengths // 100) * 13
    )
    return output_lengths
```
**EN:** The function `_get_feat_extract_output_lengths` helps provide a reusable helper for the surrounding model code. Its main inputs are `input_lengths`.
**CN:** 函数 `_get_feat_extract_output_lengths` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `input_lengths`。

### Class `Qwen3ASRProcessingInfo` (lines 104-132)
```python
class Qwen3ASRProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(Qwen3ASRConfig).thinker_config

    def get_hf_processor(self, **kwargs: object) -> Qwen3ASRProcessor:
        processor = self.ctx.get_hf_processor(
            Qwen3ASRProcessor,
            use_fast=kwargs.pop("use_fast", True),
            **kwargs,
        )
        if not hasattr(processor, "audio_token"):
            processor.audio_token = "<|audio_pad|>"
        return processor

    def get_feature_extractor(self, **kwargs: object) -> WhisperFeatureExtractor:
        hf_processor = self.get_hf_processor(**kwargs)
        feature_extractor = hf_processor.feature_extractor
        assert isinstance(feature_extractor, WhisperFeatureExtractor)
        return feature_extractor

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"audio": None}

    def get_data_parser(self) -> MultiModalDataParser:
        feature_extractor = self.get_feature_extractor()
        return Qwen3ASRMultiModalDataParser(
            target_sr=feature_extractor.sampling_rate,
            expected_hidden_size=self._get_expected_hidden_size(),
        )
```
**EN:** Defines `Qwen3ASRProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseProcessingInfo. Key methods such as `get_hf_config`, `get_hf_processor`, `get_feature_extractor`, `get_supported_mm_limits`, `get_data_parser` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3ASRProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseProcessingInfo。 `get_hf_config`, `get_hf_processor`, `get_feature_extractor`, `get_supported_mm_limits`, `get_data_parser` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3ASRDummyInputsBuilder` (lines 135-170)
```python
class Qwen3ASRDummyInputsBuilder(BaseDummyInputsBuilder[Qwen3ASRProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_audios = mm_counts.get("audio", 0)

        hf_processor = self.info.get_hf_processor()
        audio_token = hf_processor.audio_token

        return audio_token * num_audios

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_audios = mm_counts.get("audio", 0)

        feature_extractor = self.info.get_feature_extractor()

        target_audio_length = (
            min(
                feature_extractor.chunk_length,
                30,
            )
            * feature_extractor.sampling_rate
        )

        audio_overrides = mm_options.get("audio")

        return {
            "audio": self._get_dummy_audios(
                length=target_audio_length,
                num_audios=num_audios,
                overrides=audio_overrides,
            ),
        }
```
**EN:** Defines `Qwen3ASRDummyInputsBuilder`, a supporting module used by the surrounding model implementation. It inherits from BaseDummyInputsBuilder[Qwen3ASRProcessingInfo]. Key methods such as `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3ASRDummyInputsBuilder`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseDummyInputsBuilder[Qwen3ASRProcessingInfo]。 `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Function `_qwen3asr_field_config` (lines 173-181)
```python
def _qwen3asr_field_config(hf_inputs: Mapping[str, torch.Tensor]):
    audio_feature_lengths = hf_inputs.get("audio_feature_lengths", torch.empty((0,)))
    return dict(
        input_audio_features=MultiModalFieldConfig.flat_from_sizes(
            "audio", audio_feature_lengths, dim=1
        ),
        feature_attention_mask=MultiModalFieldConfig.batched("audio"),
        audio_feature_lengths=MultiModalFieldConfig.batched("audio"),
    )
```
**EN:** The function `_qwen3asr_field_config` helps provide a reusable helper for the surrounding model code. Its main inputs are `hf_inputs`.
**CN:** 函数 `_qwen3asr_field_config` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `hf_inputs`。

### Class `Qwen3ASRMultiModalDataParser` (lines 184-197)
```python
class Qwen3ASRMultiModalDataParser(MultiModalDataParser):
    def _parse_audio_data(
        self,
        data: dict[str, torch.Tensor] | ModalityData[AudioItem],
    ) -> ModalityDataItems[Any, Any] | None:
        if isinstance(data, dict):
            return DictEmbeddingItems(
                data,
                modality="audio",
                required_fields={"input_audio_features", "audio_feature_lengths"},
                fields_factory=_qwen3asr_field_config,
            )

        return super()._parse_audio_data(data)
```
**EN:** Defines `Qwen3ASRMultiModalDataParser`, a supporting module used by the surrounding model implementation. It inherits from MultiModalDataParser. Key methods such as `_parse_audio_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3ASRMultiModalDataParser`，它是一个被周边模型实现复用的支撑模块。 它继承自 MultiModalDataParser。 `_parse_audio_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3ASRMultiModalProcessor` (lines 200-256)
```python
class Qwen3ASRMultiModalProcessor(
    Qwen3OmniMoeThinkerMultiModalProcessor,
):
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return _qwen3asr_field_config(hf_inputs)

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, Any],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        processor = self.info.get_hf_processor(**hf_processor_mm_kwargs)
        tokenizer = self.info.get_tokenizer()
        vocab = tokenizer.get_vocab()

        audio_token = processor.audio_token
        audio_token_id = vocab[audio_token]

        out_mm_data = out_mm_kwargs.get_data()
        audio_feature_lengths = out_mm_data.get("audio_feature_lengths")
        feature_attention_mask = out_mm_data.get("feature_attention_mask")
        if audio_feature_lengths is None and feature_attention_mask is None:
            audio_output_lengths = []
        elif audio_feature_lengths is not None:
            audio_output_lens = _get_feat_extract_output_lengths(audio_feature_lengths)
            audio_output_lengths = audio_output_lens.tolist()
        elif feature_attention_mask is not None:
            assert isinstance(feature_attention_mask, torch.Tensor)
            audio_output_lens = _get_feat_extract_output_lengths(
                feature_attention_mask.sum(-1)
            )
            audio_output_lengths = audio_output_lens.tolist()

        def get_replacement_qwen2_audio(item_idx: int):
            num_features = audio_output_lengths[item_idx]
            if num_features == 0:
                audios = mm_items.get_items("audio", AudioProcessorItems)
                audio = audios.get(item_idx)
                raise ValueError(
                    f"The audio {audio} (len={len(audio)}) is too short "
                    "to be represented inside the model"
                )

            return [audio_token_id] * num_features

        return [
            PromptReplacement(
                modality="audio",
                target=audio_token,
                replacement=get_replacement_qwen2_audio,
            ),
        ]
```
**EN:** Defines `Qwen3ASRMultiModalProcessor`, a supporting module used by the surrounding model implementation. It inherits from Qwen3OmniMoeThinkerMultiModalProcessor. Key methods such as `_get_mm_fields_config`, `_get_prompt_updates` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3ASRMultiModalProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 Qwen3OmniMoeThinkerMultiModalProcessor。 `_get_mm_fields_config`, `_get_prompt_updates` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3ASRForConditionalGeneration` (lines 259-601)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Qwen3ASRMultiModalProcessor,
    info=Qwen3ASRProcessingInfo,
    dummy_inputs=Qwen3ASRDummyInputsBuilder,
)
class Qwen3ASRForConditionalGeneration(
    nn.Module,
    SupportsMultiModal,
    SupportsPP,
    SupportsMRoPE,
    SupportsTranscription,
    SupportsLoRA,
):
    # LoRA support
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        self.vllm_config = vllm_config  # needed for torch compile forward context
        thinker_config: Qwen3ASRThinkerConfig = (
            vllm_config.model_config.hf_config.thinker_config
        )
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = thinker_config
        self.multimodal_config = multimodal_config
        self.quant_config = quant_config

        with self._mark_tower_model(vllm_config, "audio"):
            self.audio_tower = Qwen3OmniMoeAudioEncoder(
                thinker_config.audio_config,
                prefix=maybe_prefix(prefix, "audio_tower"),
            )

        with self._mark_language_model(vllm_config):
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        if intermediate_tensors is not None:
            inputs_embeds = None

        hidden_states = self.language_model.model(
            input_ids,
            positions,
            intermediate_tensors,
            inputs_embeds=inputs_embeds,
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
        loader = AutoWeightsLoader(
            self,
            skip_prefixes=["talker.", "code2wav."],
        )
        loaded_weights = loader.load_weights(weights, mapper=self.hf_to_vllm_mapper)

        return loaded_weights
```
**EN:** Defines `Qwen3ASRForConditionalGeneration`, a supporting module used by the surrounding model implementation. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `_parse_and_validate_audio_input`, `_parse_and_validate_multimodal_inputs`, `_process_audio_input` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3ASRForConditionalGeneration`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `_parse_and_validate_audio_input`, `_parse_and_validate_multimodal_inputs`, `_process_audio_input` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **Position encoding**
  - **EN:** Rotary or related positional logic appears in the file to preserve token order information efficiently.
  - **CN:** 文件中出现了 Rotary 等位置编码逻辑，用于高效保留 token 顺序信息。

## Dependencies / 依赖关系
- **Standard library**: collections.abc, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, transformers.feature_extraction_utils, transformers.models.whisper
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.config.speech_to_text, vllm.inputs, vllm.logger, vllm.model_executor.models.interfaces, vllm.model_executor.models.module_mapping, vllm.model_executor.models.qwen3
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
