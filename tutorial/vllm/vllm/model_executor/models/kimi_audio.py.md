# kimi_audio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/kimi_audio.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for kimi_audio, including encoder/decoder glue and vLLM runtime adaptation. / 面向 kimi_audio 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 6-56)
```python
from collections.abc import Iterable, Mapping, Sequence
from typing import Any, ClassVar, Literal

import numpy as np
import torch
import torch.nn as nn
from transformers import BatchFeature
from transformers import WhisperConfig as HFWhisperConfig

from vllm.config import ModelConfig, SpeechToTextConfig, VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.config.speech_to_text import SpeechToTextParams
from vllm.inputs import PromptType, TokensPrompt
from vllm.model_executor.model_loader import DefaultModelLoader
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.model_executor.models.interfaces import (
    SupportsMultiModal,
    SupportsPP,
    SupportsTranscription,
)
from vllm.model_executor.models.utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
from vllm.model_executor.models.whisper import WhisperEncoder
from vllm.model_executor.models.whisper_utils import ISO639_1_SUPPORTED_LANGS
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import MultiModalFieldConfig
from vllm.multimodal.parse import (
    AudioItem,
    DictEmbeddingItems,
    ModalityData,
    ModalityDataItems,
    MultiModalDataParser,
)
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseProcessingInfo,
    PromptReplacement,
)
from vllm.multimodal.processing.processor import (
    BaseMultiModalProcessor,
    ProcessorInputs,
)
from vllm.sequence import IntermediateTensors
from vllm.tokenizers import cached_get_tokenizer
from vllm.tokenizers.kimi_audio import KimiAudioTokenizer
from vllm.transformers_utils.processor import cached_feature_extractor_from_config
from vllm.transformers_utils.processors.kimi_audio import KimiAudioProcessor
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 59-59)
```python
KIMIA_WHISPER_SUBFOLDER = "whisper-large-v3"
```
**EN:** This block defines KIMIA_WHISPER_SUBFOLDER, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 KIMIA_WHISPER_SUBFOLDER，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 224-227)
```python
_KIMIAUDIO_FIELD_CONFIG = {
    "whisper_input_features": MultiModalFieldConfig.batched("audio"),
    "feature_attention_mask": MultiModalFieldConfig.batched("audio"),
}
```
**EN:** This block defines _KIMIAUDIO_FIELD_CONFIG, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _KIMIAUDIO_FIELD_CONFIG，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Function `_get_feat_extract_output_lengths` (lines 62-73)
```python
def _get_feat_extract_output_lengths(input_lengths: torch.Tensor) -> torch.Tensor:
    """Compute output lengths after Whisper feature extraction.

    Whisper processes audio through multiple conv layers with stride=2,
    producing 13 output features per 100 input samples.
    """
    input_lengths_leave = input_lengths % 100
    feat_lengths = (input_lengths_leave - 1) // 2 + 1
    output_lengths = (
        ((feat_lengths - 1) // 2 + 1 - 1) // 2 + 1 + (input_lengths // 100) * 13
    )
    return output_lengths
```
**EN:** Function `_get_feat_extract_output_lengths` encapsulates a focused piece of reusable logic inside this module. The docstring says: Compute output lengths after Whisper feature extraction.
**CN:** Function `_get_feat_extract_output_lengths` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Compute output lengths after Whisper feature extraction。

### Class `KimiAudioWhisperEncoder` (lines 76-135)
```python
class KimiAudioWhisperEncoder(WhisperEncoder):
    """WhisperEncoder for Kimi-Audio with packed_modules_mapping."""

    # packed_modules_mapping for Q/K/V fusion during weight loading
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
    }

    def __init__(
        self, *, vllm_config: VllmConfig, prefix: str = "", init_in_fp32: bool = False
    ):
        # Load Whisper config from subfolder (authoritative source)
        # Kimi-Audio stores Whisper config in whisper-large-v3/config.json
        model_path = vllm_config.model_config.model

        # Load WhisperConfig from the subfolder
        whisper_config = HFWhisperConfig.from_pretrained(
            model_path,
            subfolder=KIMIA_WHISPER_SUBFOLDER,
            revision=vllm_config.model_config.revision,
        )

        super().__init__(
            vllm_config=vllm_config.with_hf_config(whisper_config),
            prefix=prefix,
```
**EN:** Class `KimiAudioWhisperEncoder` organizes related behavior for this model family or helper component. It inherits from WhisperEncoder. Key methods include __init__, load_weights.
**CN:** 类 `KimiAudioWhisperEncoder` 用于组织该模型族或辅助组件的相关行为。 它继承自 WhisperEncoder。 关键方法包括 __init__, load_weights。

### Method `KimiAudioWhisperEncoder.__init__` (lines 84-102)
```python
    def __init__(
        self, *, vllm_config: VllmConfig, prefix: str = "", init_in_fp32: bool = False
    ):
        # Load Whisper config from subfolder (authoritative source)
        # Kimi-Audio stores Whisper config in whisper-large-v3/config.json
        model_path = vllm_config.model_config.model

        # Load WhisperConfig from the subfolder
        whisper_config = HFWhisperConfig.from_pretrained(
            model_path,
            subfolder=KIMIA_WHISPER_SUBFOLDER,
            revision=vllm_config.model_config.revision,
        )

        super().__init__(
            vllm_config=vllm_config.with_hf_config(whisper_config),
            prefix=prefix,
            init_in_fp32=init_in_fp32,
        )
```
**EN:** Method `KimiAudioWhisperEncoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KimiAudioWhisperEncoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KimiAudioWhisperEncoder.load_weights` (lines 104-135)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue

                param = params_dict[name]
                weight_loader = param.weight_loader
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
**EN:** Method `KimiAudioWhisperEncoder.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `KimiAudioWhisperEncoder.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `KimiAudioProcessingInfo` (lines 144-171)
```python
class KimiAudioProcessingInfo(BaseProcessingInfo):
    """Processing info for vLLM registry."""

    def get_hf_processor(self, **kwargs: object) -> KimiAudioProcessor:
        feature_extractor = cached_feature_extractor_from_config(
            self.ctx.model_config,
            subfolder=KIMIA_WHISPER_SUBFOLDER,
        )

        return KimiAudioProcessor(
            feature_extractor=feature_extractor,
            tokenizer=self.get_tokenizer(),
        )

    def get_feature_extractor(self, **kwargs: object):
        return cached_feature_extractor_from_config(
            self.ctx.model_config, subfolder=KIMIA_WHISPER_SUBFOLDER
        )

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"audio": 1}

    def get_data_parser(self) -> "KimiAudioMultiModalDataParser":
        feature_extractor = self.get_feature_extractor()
        return KimiAudioMultiModalDataParser(
```
**EN:** Class `KimiAudioProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_processor, get_feature_extractor, get_supported_mm_limits, get_data_parser.
**CN:** 类 `KimiAudioProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_processor, get_feature_extractor, get_supported_mm_limits, get_data_parser。

### Method `KimiAudioProcessingInfo.get_hf_processor` (lines 147-156)
```python
    def get_hf_processor(self, **kwargs: object) -> KimiAudioProcessor:
        feature_extractor = cached_feature_extractor_from_config(
            self.ctx.model_config,
            subfolder=KIMIA_WHISPER_SUBFOLDER,
        )

        return KimiAudioProcessor(
            feature_extractor=feature_extractor,
            tokenizer=self.get_tokenizer(),
        )
```
**EN:** Method `KimiAudioProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiAudioProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiAudioProcessingInfo.get_feature_extractor` (lines 158-161)
```python
    def get_feature_extractor(self, **kwargs: object):
        return cached_feature_extractor_from_config(
            self.ctx.model_config, subfolder=KIMIA_WHISPER_SUBFOLDER
        )
```
**EN:** Method `KimiAudioProcessingInfo.get_feature_extractor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiAudioProcessingInfo.get_feature_extractor` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiAudioProcessingInfo.get_supported_mm_limits` (lines 163-164)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"audio": 1}
```
**EN:** Method `KimiAudioProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiAudioProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiAudioProcessingInfo.get_data_parser` (lines 166-171)
```python
    def get_data_parser(self) -> "KimiAudioMultiModalDataParser":
        feature_extractor = self.get_feature_extractor()
        return KimiAudioMultiModalDataParser(
            target_sr=feature_extractor.sampling_rate,
            expected_hidden_size=self._get_expected_hidden_size(),
        )
```
**EN:** Method `KimiAudioProcessingInfo.get_data_parser` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiAudioProcessingInfo.get_data_parser` 封装了该模块中的一段可复用核心逻辑。

### Class `KimiAudioDummyInputsBuilder` (lines 174-220)
```python
class KimiAudioDummyInputsBuilder(BaseDummyInputsBuilder[KimiAudioProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        return ""

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, Any] | None = None,
    ) -> dict[str, Any]:
        num_audios = mm_counts.get("audio", 0)
        if num_audios == 0:
            return {}

        feature_extractor = self.info.get_feature_extractor()
        target_audio_length = (
            min(feature_extractor.chunk_length, 30) * feature_extractor.sampling_rate
        )

        return {
            "audio": self._get_dummy_audios(
                length=target_audio_length, num_audios=num_audios
            ),
        }
```
**EN:** Class `KimiAudioDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[KimiAudioProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data, get_dummy_processor_inputs.
**CN:** 类 `KimiAudioDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[KimiAudioProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data, get_dummy_processor_inputs。

### Method `KimiAudioDummyInputsBuilder.get_dummy_text` (lines 175-176)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        return ""
```
**EN:** Method `KimiAudioDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiAudioDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiAudioDummyInputsBuilder.get_dummy_mm_data` (lines 178-197)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, Any] | None = None,
    ) -> dict[str, Any]:
        num_audios = mm_counts.get("audio", 0)
        if num_audios == 0:
            return {}

        feature_extractor = self.info.get_feature_extractor()
        target_audio_length = (
            min(feature_extractor.chunk_length, 30) * feature_extractor.sampling_rate
        )

        return {
            "audio": self._get_dummy_audios(
                length=target_audio_length, num_audios=num_audios
            ),
        }
```
**EN:** Method `KimiAudioDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiAudioDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiAudioDummyInputsBuilder.get_dummy_processor_inputs` (lines 199-220)
```python
    def get_dummy_processor_inputs(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> ProcessorInputs:
        dummy_mm_data = self.get_dummy_mm_data(seq_len, mm_counts, mm_options)
        dummy_mm_items = self.info.parse_mm_data(dummy_mm_data)

        num_audios = mm_counts.get("audio", 0)
        dummy_tokens = (
            [198]
            if num_audios == 0
            else [
                KimiAudioProcessor.KIMIA_MEDIA_BEGIN,
                KimiAudioProcessor.KIMIA_TEXT_BLANK,
                KimiAudioProcessor.KIMIA_MEDIA_END,
            ]
            * num_audios
        )

        return ProcessorInputs(prompt=dummy_tokens, mm_data_items=dummy_mm_items)
```
**EN:** Method `KimiAudioDummyInputsBuilder.get_dummy_processor_inputs` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiAudioDummyInputsBuilder.get_dummy_processor_inputs` 封装了该模块中的一段可复用核心逻辑。

### Class `KimiAudioMultiModalDataParser` (lines 230-245)
```python
class KimiAudioMultiModalDataParser(MultiModalDataParser):
    """Custom data parser for Kimi-Audio multimodal data."""

    def _parse_audio_data(
        self,
        data: dict[str, torch.Tensor] | ModalityData[AudioItem],
    ) -> ModalityDataItems[Any, Any] | None:
        if isinstance(data, dict):
            return DictEmbeddingItems(
                data,
                modality="audio",
                required_fields={"whisper_input_features", "feature_attention_mask"},
                fields_factory=lambda hf_inputs: _KIMIAUDIO_FIELD_CONFIG,
            )

        return super()._parse_audio_data(data)
```
**EN:** Class `KimiAudioMultiModalDataParser` organizes related behavior for this model family or helper component. It inherits from MultiModalDataParser. Key methods include _parse_audio_data.
**CN:** 类 `KimiAudioMultiModalDataParser` 用于组织该模型族或辅助组件的相关行为。 它继承自 MultiModalDataParser。 关键方法包括 _parse_audio_data。

### Class `KimiAudioMultiModalProcessor` (lines 248-329)
```python
class KimiAudioMultiModalProcessor(BaseMultiModalProcessor[KimiAudioProcessingInfo]):
    """vLLM multi-modal processor wrapper for Kimi-Audio."""

    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        """Call the HuggingFace processor."""
        # Convert mm_data format: {'audios': [...]} -> {'audio': ...}
        mm_data = dict(mm_data)
        audios = mm_data.pop("audios", [])

        # Convert audio format: [(array, sr), ...] -> [array, ...]
        # KimiAudioProcessor expects raw numpy arrays
        if audios:
            audio_arrays = []
            for aud in audios:
                if isinstance(aud, (tuple, list)) and len(aud) == 2:
                    # Format: (audio_array, sampling_rate)
                    audio_arrays.append(aud[0])
                elif isinstance(aud, np.ndarray):
                    audio_arrays.append(aud)
```
**EN:** Class `KimiAudioMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[KimiAudioProcessingInfo]. Key methods include _call_hf_processor, _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `KimiAudioMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[KimiAudioProcessingInfo]。 关键方法包括 _call_hf_processor, _get_mm_fields_config, _get_prompt_updates。

### Class `KimiAudioMultiModalProjector` (lines 337-368)
```python
class KimiAudioMultiModalProjector(nn.Module):
    """Projects Whisper features to LLM embedding space.

    Kimi-Audio VQ-Adaptor architecture:
    Custom Whisper (5120) → Linear[5120→3584] → Linear[3584→3584] → LayerNorm
    """

    def __init__(
        self,
        whisper_dim: int = 5120,  # Kimi-Audio custom Whisper encoder dim
        llm_dim: int = 3584,
        prefix: str = "",
    ):
        super().__init__()
        self.whisper_dim = whisper_dim
        self.llm_dim = llm_dim

        # VQ-Adaptor layers (exact checkpoint structure)
        # layers.0: Linear[5120 → 3584]
        self.vq_adaptor_layers_0 = nn.Linear(whisper_dim, llm_dim)
        # layers.3: Linear[3584 → 3584]
        self.vq_adaptor_layers_3 = nn.Linear(llm_dim, llm_dim)
        # layers.4: LayerNorm[3584]
        self.vq_adaptor_layers_4 = nn.LayerNorm(llm_dim)
```
**EN:** Class `KimiAudioMultiModalProjector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `KimiAudioMultiModalProjector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `KimiAudioMultiModalProjector.__init__` (lines 344-360)
```python
    def __init__(
        self,
        whisper_dim: int = 5120,  # Kimi-Audio custom Whisper encoder dim
        llm_dim: int = 3584,
        prefix: str = "",
    ):
        super().__init__()
        self.whisper_dim = whisper_dim
        self.llm_dim = llm_dim

        # VQ-Adaptor layers (exact checkpoint structure)
        # layers.0: Linear[5120 → 3584]
        self.vq_adaptor_layers_0 = nn.Linear(whisper_dim, llm_dim)
        # layers.3: Linear[3584 → 3584]
        self.vq_adaptor_layers_3 = nn.Linear(llm_dim, llm_dim)
        # layers.4: LayerNorm[3584]
        self.vq_adaptor_layers_4 = nn.LayerNorm(llm_dim)
```
**EN:** Method `KimiAudioMultiModalProjector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KimiAudioMultiModalProjector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KimiAudioMultiModalProjector.forward` (lines 362-368)
```python
    def forward(self, audio_features: torch.Tensor) -> torch.Tensor:
        # Project: [B, T, 5120] → [B, T, 3584]
        hidden = self.vq_adaptor_layers_0(audio_features)
        hidden = torch.nn.functional.gelu(hidden)
        hidden = self.vq_adaptor_layers_3(hidden)
        hidden = self.vq_adaptor_layers_4(hidden)
        return hidden
```
**EN:** Method `KimiAudioMultiModalProjector.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `KimiAudioMultiModalProjector.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `KimiAudioForConditionalGeneration` (lines 376-674)
```python
@MULTIMODAL_REGISTRY.register_processor(
    KimiAudioMultiModalProcessor,
    info=KimiAudioProcessingInfo,
    dummy_inputs=KimiAudioDummyInputsBuilder,
)
class KimiAudioForConditionalGeneration(
    nn.Module,
    SupportsMultiModal,
    SupportsPP,
    SupportsTranscription,
):
    """Kimi-Audio model for ASR transcription."""

    # Kimi-Audio supports a subset of Whisper's supported languages
    supported_languages: ClassVar[Mapping[str, str]] = {
        k: ISO639_1_SUPPORTED_LANGS[k]
        for k in ["zh", "en", "ja", "ko", "de", "fr", "es", "it", "pt", "ru", "ar"]
    }
    supports_transcription: ClassVar[Literal[True]] = True

    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            # audio tower
            "model.encoder.": "audio_tower.",
            # Audio projector (VQ-Adaptor)
```
**EN:** Class `KimiAudioForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP, SupportsTranscription. Key methods include get_placeholder_str, __init__, _parse_and_validate_audio_input, _process_audio_input, embed_multimodal, embed_input_ids.
**CN:** 类 `KimiAudioForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP、SupportsTranscription。 关键方法包括 get_placeholder_str, __init__, _parse_and_validate_audio_input, _process_audio_input, embed_multimodal, embed_input_ids。

### Method `KimiAudioForConditionalGeneration.get_placeholder_str` (lines 416-417)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        return cls.AUDIO_PLACEHOLDER if modality.startswith("audio") else None
```
**EN:** Method `KimiAudioForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiAudioForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiAudioForConditionalGeneration.__init__` (lines 419-455)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        self.config = vllm_config.model_config.hf_config
        self.quant_config = vllm_config.quant_config
        self.multimodal_config = vllm_config.model_config.multimodal_config
        self.model_path = vllm_config.model_config.model

        self.secondary_weights = [
            DefaultModelLoader.Source(
                model_or_path=vllm_config.model_config.model,
                subfolder="whisper-large-v3",
                revision=vllm_config.model_config.revision,
            )
        ]

        with self._mark_tower_model(vllm_config, "audio"):
            self.audio_tower = KimiAudioWhisperEncoder(
                vllm_config=vllm_config,
                prefix=maybe_prefix(prefix, "audio_tower"),
            )
            self.multi_modal_projector = KimiAudioMultiModalProjector(
                whisper_dim=getattr(self.config, "kimia_adaptor_input_dim", 5120),
                llm_dim=self.config.hidden_size,
                prefix=maybe_prefix(prefix, "multi_modal_projector"),
            )

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config.with_hf_config(
                    self.config, architectures=["Qwen2ForCausalLM"]
                ),
                prefix=maybe_prefix(prefix, "language_model"),
            )

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )
```
**EN:** Method `KimiAudioForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KimiAudioForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KimiAudioForConditionalGeneration.embed_multimodal` (lines 491-505)
```python
    def embed_multimodal(self, **kwargs: object) -> list[torch.Tensor] | None:
        audio_input = self._parse_and_validate_audio_input(**kwargs)
        if audio_input is None:
            return []

        audio_embeds = self._process_audio_input(audio_input)

        # audio_embeds shape: [batch_size, seq_len, hidden_dim]
        # Return as list of 2D tensors, one per batch item
        if audio_embeds.dim() == 3:
            # Unbind batch dimension: [B, T, D] -> list of B tensors [T, D]
            return list(audio_embeds.unbind(dim=0))
        else:
            # Single sample: [T, D] -> wrap in list
            return [audio_embeds]
```
**EN:** Method `KimiAudioForConditionalGeneration.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `KimiAudioForConditionalGeneration.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `KimiAudioForConditionalGeneration.embed_input_ids` (lines 507-569)
```python
    def embed_input_ids(
        self,
        input_ids: torch.Tensor,
        multimodal_embeddings: tuple[torch.Tensor, ...] | None = None,
        *,
        is_multimodal: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """Embed input IDs and fuse with audio embeddings.

        Kimi-Audio fusion: inputs_embeds = (text_emb + audio_emb) × √2

        For PP compatibility, we use the is_multimodal mask from vLLM engine
        which is correctly computed per pipeline stage.
        """
        # Get text embeddings
        inputs_embeds = self.language_model.model.embed_tokens(input_ids)

        if multimodal_embeddings is None or len(multimodal_embeddings) == 0:
            return inputs_embeds

        # is_multimodal must be provided for PP to work correctly
        if is_multimodal is None or not is_multimodal.any():
            return inputs_embeds

        # multimodal_embeddings[0] contains audio embeddings
        audio_embeds = multimodal_embeddings[0]

        # Handle different tensor structures
        if isinstance(audio_embeds, (list, tuple)):
            audio_embeds = torch.cat(audio_embeds, dim=0)
        elif audio_embeds.dim() == 3:
            audio_embeds = audio_embeds.reshape(-1, audio_embeds.shape[-1])

        # In PP, audio_embeds count should match is_multimodal.sum()
        # For now, use embeddings sequentially
        # (works for non-PP, PP needs vLLM infra fix)
        num_mm_tokens = is_multimodal.sum().item()
        num_audio_embeds = audio_embeds.shape[0]

        # Use the minimum of available embeddings and positions
        # This ensures we don't access out-of-bounds
        num_to_use = min(num_audio_embeds, num_mm_tokens)

        # Get positions for the tokens we'll actually process
        mm_positions = is_multimodal.nonzero(as_tuple=True)[0]
        actual_mm_mask = torch.zeros_like(is_multimodal)
        actual_mm_mask[mm_positions[:num_to_use]] = True

        # Use corresponding embeddings
        used_audio_embeds = audio_embeds[:num_to_use]

        # Save text embeddings at multimodal positions
        text_at_mm_positions = inputs_embeds[actual_mm_mask].clone()

        # Replace text with audio at multimodal positions
        inputs_embeds[actual_mm_mask] = used_audio_embeds.to(dtype=inputs_embeds.dtype)

        # Apply Kimi-Audio's unique fusion formula: (text + audio) × √2
        inputs_embeds[actual_mm_mask] = (
            inputs_embeds[actual_mm_mask] + text_at_mm_positions
# ... truncated for analysis ...
```
**EN:** Method `KimiAudioForConditionalGeneration.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline. The docstring says: Embed input IDs and fuse with audio embeddings.
**CN:** Method `KimiAudioForConditionalGeneration.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。 文档字符串表达的核心意思是：Embed input IDs and fuse with audio embeddings。

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
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Any, ClassVar, Literal`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `import torch.nn as nn`, `from transformers import BatchFeature`, `from transformers import WhisperConfig as HFWhisperConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import ModelConfig, SpeechToTextConfig, VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.config.speech_to_text import SpeechToTextParams`, `from vllm.inputs import PromptType, TokensPrompt`, `from vllm.model_executor.model_loader import DefaultModelLoader`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`, `from vllm.model_executor.models.interfaces import (`, `from vllm.model_executor.models.utils import (`, `from vllm.model_executor.models.whisper import WhisperEncoder`, `from vllm.model_executor.models.whisper_utils import ISO639_1_SUPPORTED_LANGS`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import MultiModalFieldConfig`
- **Module note / 模块说明**: **EN:** Inference-only Kimi-Audio model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only Kimi-Audio model compatible with HuggingFace weights.。
