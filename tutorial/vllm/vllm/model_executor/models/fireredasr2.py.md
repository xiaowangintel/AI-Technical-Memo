# fireredasr2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/fireredasr2.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for fireredasr2, including encoder/decoder glue and vLLM runtime adaptation. / 面向 fireredasr2 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-60)
```python
import math
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, cast

import torch
from torch import nn
from transformers import (
    BatchFeature,
    Qwen2Config,
)

from vllm.config import ModelConfig, SpeechToTextConfig, VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.config.speech_to_text import SpeechToTextParams
from vllm.inputs import MultiModalDataDict, PromptType
from vllm.logger import init_logger
from vllm.model_executor.layers.activation import _ACTIVATION_REGISTRY
from vllm.model_executor.layers.linear import (
    ReplicatedLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.models.whisper_utils import (
    ISO639_1_SUPPORTED_LANGS,
)
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import MultiModalDataItems, MultiModalDataParser
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
    PromptUpdateDetails,
)
from vllm.transformers_utils.processor import cached_processor_from_config
from vllm.transformers_utils.processors.fireredasr2 import (
    FireRedASR2FeatureExtractor,
)
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .conformer_encoder import ConformerEncoder
from .interfaces import (
    MultiModalEmbeddings,
    SupportsMultiModal,
    SupportsTranscription,
    _require_is_multimodal,
)
from .qwen2 import Qwen2ForCausalLM
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    _merge_multimodal_embeddings,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `FireRedASR2AudioInputs` (lines 65-84)
```python
class FireRedASR2AudioInputs(TensorSchema):
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
    speech_lengths: Annotated[
        list[torch.Tensor] | None,
        TensorShape("b"),
    ]
    fake_token_lengths: Annotated[
        list[torch.Tensor] | None,
        TensorShape("b"),
    ]
```
**EN:** Class `FireRedASR2AudioInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `FireRedASR2AudioInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `FireRedASR2Adapter` (lines 87-118)
```python
class FireRedASR2Adapter(nn.Module):
    def __init__(self, encoder_dim: int, llm_dim: int, downsample_rate: int = 2):
        super().__init__()
        self.ds = downsample_rate
        self.linear1 = ReplicatedLinear(
            input_size=encoder_dim * downsample_rate,
            output_size=llm_dim,
            bias=True,
        )
        self.relu = _ACTIVATION_REGISTRY["relu"]
        self.linear2 = ReplicatedLinear(
            input_size=llm_dim,
            output_size=llm_dim,
            bias=True,
        )

    def forward(self, x, x_lens):
        batch_size, seq_len, feat_dim = x.size()
        num_frames_to_discard = seq_len % self.ds
        if num_frames_to_discard > 0:
            x = x[:, :-num_frames_to_discard, :]
        seq_len = x.size(1)

        x = x.contiguous()
        x = x.view(batch_size, seq_len // self.ds, feat_dim * self.ds)
```
**EN:** Class `FireRedASR2Adapter` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `FireRedASR2Adapter` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `FireRedASR2Adapter.__init__` (lines 88-101)
```python
    def __init__(self, encoder_dim: int, llm_dim: int, downsample_rate: int = 2):
        super().__init__()
        self.ds = downsample_rate
        self.linear1 = ReplicatedLinear(
            input_size=encoder_dim * downsample_rate,
            output_size=llm_dim,
            bias=True,
        )
        self.relu = _ACTIVATION_REGISTRY["relu"]
        self.linear2 = ReplicatedLinear(
            input_size=llm_dim,
            output_size=llm_dim,
            bias=True,
        )
```
**EN:** Method `FireRedASR2Adapter.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FireRedASR2Adapter.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FireRedASR2Adapter.forward` (lines 103-118)
```python
    def forward(self, x, x_lens):
        batch_size, seq_len, feat_dim = x.size()
        num_frames_to_discard = seq_len % self.ds
        if num_frames_to_discard > 0:
            x = x[:, :-num_frames_to_discard, :]
        seq_len = x.size(1)

        x = x.contiguous()
        x = x.view(batch_size, seq_len // self.ds, feat_dim * self.ds)

        x, _ = self.linear1(x)
        x = self.relu(x)
        x, _ = self.linear2(x)

        new_x_lens = torch.clamp(x_lens, max=seq_len) // self.ds
        return x, new_x_lens
```
**EN:** Method `FireRedASR2Adapter.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FireRedASR2Adapter.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `FireRedASR2Encoder` (lines 121-130)
```python
class FireRedASR2Encoder(nn.Module):
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
    ):
        super().__init__()
        self.audio_encoder = ConformerEncoder(
            **vllm_config.model_config.hf_config.audio_encoder_conf
        )
```
**EN:** Class `FireRedASR2Encoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__.
**CN:** 类 `FireRedASR2Encoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__。

### Method `FireRedASR2Encoder.__init__` (lines 122-130)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
    ):
        super().__init__()
        self.audio_encoder = ConformerEncoder(
            **vllm_config.model_config.hf_config.audio_encoder_conf
        )
```
**EN:** Method `FireRedASR2Encoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FireRedASR2Encoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `FireRedASR2Model` (lines 133-173)
```python
class FireRedASR2Model(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        self.encoder = FireRedASR2Encoder(
            vllm_config=vllm_config,
        )
        encoder_dim = self.encoder.audio_encoder.odim
        llm_dim = vllm_config.model_config.hf_config.hidden_size
        self.encoder_projector = FireRedASR2Adapter(
            encoder_dim,
            llm_dim,
            vllm_config.model_config.hf_config.encoder_downsample_rate,
        )

        self.decoder = Qwen2ForCausalLM(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "decoder")
        )

    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        decoder_outputs = self.decoder(
```
**EN:** Class `FireRedASR2Model` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward, get_encoder_outputs.
**CN:** 类 `FireRedASR2Model` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward, get_encoder_outputs。

### Method `FireRedASR2Model.__init__` (lines 134-149)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        self.encoder = FireRedASR2Encoder(
            vllm_config=vllm_config,
        )
        encoder_dim = self.encoder.audio_encoder.odim
        llm_dim = vllm_config.model_config.hf_config.hidden_size
        self.encoder_projector = FireRedASR2Adapter(
            encoder_dim,
            llm_dim,
            vllm_config.model_config.hf_config.encoder_downsample_rate,
        )

        self.decoder = Qwen2ForCausalLM(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "decoder")
        )
```
**EN:** Method `FireRedASR2Model.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FireRedASR2Model.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FireRedASR2Model.forward` (lines 151-162)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        decoder_outputs = self.decoder(
            input_ids=input_ids,
            positions=positions,
            inputs_embeds=inputs_embeds,
        )
        return decoder_outputs
```
**EN:** Method `FireRedASR2Model.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FireRedASR2Model.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `FireRedASR2Model.get_encoder_outputs` (lines 164-173)
```python
    def get_encoder_outputs(
        self,
        speech: torch.Tensor | list[torch.Tensor] | None,
        speech_lengths: torch.Tensor | list[torch.Tensor] | None,
    ) -> torch.Tensor | None:
        encoder_outs, enc_lengths, enc_mask = self.encoder.audio_encoder(
            speech, speech_lengths
        )
        speech_features, speech_lens = self.encoder_projector(encoder_outs, enc_lengths)
        return speech_features
```
**EN:** Method `FireRedASR2Model.get_encoder_outputs` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2Model.get_encoder_outputs` 封装了该模块中的一段可复用核心逻辑。

### Class `FireRedASR2ProcessingInfo` (lines 176-197)
```python
class FireRedASR2ProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self) -> Qwen2Config:
        return self.ctx.get_hf_config(Qwen2Config)

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"audio": 1}

    def get_feature_extractor(self, **kwargs: object) -> FireRedASR2FeatureExtractor:
        hf_processor = self.get_hf_processor(**kwargs)
        feature_extractor = hf_processor.feature_extractor  # type: ignore
        assert isinstance(feature_extractor, FireRedASR2FeatureExtractor)
        return feature_extractor

    def get_data_parser(self) -> MultiModalDataParser:
        feature_extractor = self.get_feature_extractor()
        return MultiModalDataParser(
            target_sr=feature_extractor.sampling_rate,
            target_channels=self.get_target_channels(),
        )

    def get_target_channels(self) -> int:
        return 1
```
**EN:** Class `FireRedASR2ProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_supported_mm_limits, get_feature_extractor, get_data_parser, get_target_channels.
**CN:** 类 `FireRedASR2ProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_supported_mm_limits, get_feature_extractor, get_data_parser, get_target_channels。

### Method `FireRedASR2ProcessingInfo.get_hf_config` (lines 177-178)
```python
    def get_hf_config(self) -> Qwen2Config:
        return self.ctx.get_hf_config(Qwen2Config)
```
**EN:** Method `FireRedASR2ProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2ProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `FireRedASR2ProcessingInfo.get_supported_mm_limits` (lines 180-181)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"audio": 1}
```
**EN:** Method `FireRedASR2ProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2ProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Method `FireRedASR2ProcessingInfo.get_feature_extractor` (lines 183-187)
```python
    def get_feature_extractor(self, **kwargs: object) -> FireRedASR2FeatureExtractor:
        hf_processor = self.get_hf_processor(**kwargs)
        feature_extractor = hf_processor.feature_extractor  # type: ignore
        assert isinstance(feature_extractor, FireRedASR2FeatureExtractor)
        return feature_extractor
```
**EN:** Method `FireRedASR2ProcessingInfo.get_feature_extractor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2ProcessingInfo.get_feature_extractor` 封装了该模块中的一段可复用核心逻辑。

### Method `FireRedASR2ProcessingInfo.get_data_parser` (lines 189-194)
```python
    def get_data_parser(self) -> MultiModalDataParser:
        feature_extractor = self.get_feature_extractor()
        return MultiModalDataParser(
            target_sr=feature_extractor.sampling_rate,
            target_channels=self.get_target_channels(),
        )
```
**EN:** Method `FireRedASR2ProcessingInfo.get_data_parser` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2ProcessingInfo.get_data_parser` 封装了该模块中的一段可复用核心逻辑。

### Class `FireRedASR2DummyInputsBuilder` (lines 200-225)
```python
class FireRedASR2DummyInputsBuilder(BaseDummyInputsBuilder[FireRedASR2ProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_audios = mm_counts.get("audio", 0)

        return "<|AUDIO|>" * num_audios

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

        ret = {
            "audio": self._get_dummy_audios(
                length=audio_len, num_audios=num_audios, overrides=audio_overrides
            )
        }
```
**EN:** Class `FireRedASR2DummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[FireRedASR2ProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `FireRedASR2DummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[FireRedASR2ProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `FireRedASR2DummyInputsBuilder.get_dummy_text` (lines 201-204)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_audios = mm_counts.get("audio", 0)

        return "<|AUDIO|>" * num_audios
```
**EN:** Method `FireRedASR2DummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2DummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `FireRedASR2DummyInputsBuilder.get_dummy_mm_data` (lines 206-225)
```python
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

        ret = {
            "audio": self._get_dummy_audios(
                length=audio_len, num_audios=num_audios, overrides=audio_overrides
            )
        }
        return ret
```
**EN:** Method `FireRedASR2DummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2DummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `FireRedASR2MultiModalProcessor` (lines 228-307)
```python
class FireRedASR2MultiModalProcessor(
    BaseMultiModalProcessor[FireRedASR2ProcessingInfo]
):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        if mm_data:
            feature_extractor = self.info.get_feature_extractor(**mm_kwargs)
            mm_data = dict(audio=mm_data.pop("audios"))
            mm_kwargs = dict(
                **mm_kwargs,
                sampling_rate=feature_extractor.sampling_rate,
            )
        processed_outputs = super()._call_hf_processor(
            prompt=prompt,
            mm_data=mm_data,
            mm_kwargs=mm_kwargs,
            tok_kwargs=tok_kwargs,
        )
        if "labels" in processed_outputs:
            processed_outputs["input_ids"] = processed_outputs.pop("labels")
```
**EN:** Class `FireRedASR2MultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[FireRedASR2ProcessingInfo]. Key methods include _call_hf_processor, _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `FireRedASR2MultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[FireRedASR2ProcessingInfo]。 关键方法包括 _call_hf_processor, _get_mm_fields_config, _get_prompt_updates。

### Class `FireRedASR2ForConditionalGeneration` (lines 315-484)
```python
@MULTIMODAL_REGISTRY.register_processor(
    FireRedASR2MultiModalProcessor,
    info=FireRedASR2ProcessingInfo,
    dummy_inputs=FireRedASR2DummyInputsBuilder,
)
class FireRedASR2ForConditionalGeneration(
    nn.Module, SupportsTranscription, SupportsMultiModal
):
    packed_modules_mapping = {
        "self_attn.qkv_proj": [
            "self_attn.q_proj",
            "self_attn.k_proj",
            "self_attn.v_proj",
        ],
        "encoder_attn.kv_proj": ["encoder_attn.k_proj", "encoder_attn.v_proj"],
    }

    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_substr={
            "llm.": "model.decoder.",
            "encoder.": "model.encoder.audio_encoder.",
            "encoder_projector.": "model.encoder_projector.",
            "net.0": "pre_layer_norm",
            "net.1": "linear_expand",
            "net.4": "linear_project",
```
**EN:** Class `FireRedASR2ForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsTranscription, SupportsMultiModal. Key methods include validate_language, get_generation_prompt, get_speech_to_text_config, get_num_audio_tokens, __init__, forward.
**CN:** 类 `FireRedASR2ForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsTranscription、SupportsMultiModal。 关键方法包括 validate_language, get_generation_prompt, get_speech_to_text_config, get_num_audio_tokens, __init__, forward。

### Method `FireRedASR2ForConditionalGeneration.validate_language` (lines 343-354)
```python
    @classmethod
    def validate_language(cls, language: str | None) -> str | None:
        if language is None:
            # TODO language should be optional and can be guessed.
            # For now we default to en. See
            # https://github.com/huggingface/transformers/blob/main/src/transformers/models/whisper/generation_whisper.py#L1520
            logger.warning(
                "Defaulting to language='en'. If you wish to transcribe "
                "audio in a different language, pass the `language` field "
                "in the TranscriptionRequest."
            )
            language = "en"
        return super().validate_language(language)
```
**EN:** Method `FireRedASR2ForConditionalGeneration.validate_language` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2ForConditionalGeneration.validate_language` 封装了该模块中的一段可复用核心逻辑。

### Method `FireRedASR2ForConditionalGeneration.get_generation_prompt` (lines 357-377)
```python
    @classmethod
    def get_generation_prompt(
        cls,
        stt_params: SpeechToTextParams,
    ) -> PromptType:
        audio = stt_params.audio
        stt_config = stt_params.stt_config
        language = stt_params.language

        if language is None:
            raise ValueError(
                "Language must be specified when creating the fireredasr2 prompt"
            )

        prompt_str = "<|im_start|>user\n<|AUDIO|>请转写音频为文字<|im_end|>\n<|im_start|>assistant\n"  # noqa: E501
        prompt = {
            "prompt": prompt_str,
            "multi_modal_data": {
                "audio": (audio, stt_config.sample_rate),
            },
        }
        return cast(PromptType, prompt)
```
**EN:** Method `FireRedASR2ForConditionalGeneration.get_generation_prompt` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2ForConditionalGeneration.get_generation_prompt` 封装了该模块中的一段可复用核心逻辑。

### Method `FireRedASR2ForConditionalGeneration.get_speech_to_text_config` (lines 380-388)
```python
    @classmethod
    def get_speech_to_text_config(
        cls, model_config: ModelConfig, task_type: str
    ) -> SpeechToTextConfig:
        processor = cached_processor_from_config(model_config)

        return SpeechToTextConfig(
            max_audio_clip_s=processor.feature_extractor.chunk_length,
            sample_rate=processor.feature_extractor.sampling_rate,
        )
```
**EN:** Method `FireRedASR2ForConditionalGeneration.get_speech_to_text_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2ForConditionalGeneration.get_speech_to_text_config` 封装了该模块中的一段可复用核心逻辑。

### Method `FireRedASR2ForConditionalGeneration.get_num_audio_tokens` (lines 391-400)
```python
    @classmethod
    def get_num_audio_tokens(
        cls,
        audio_duration_s: float,
        stt_config: SpeechToTextConfig,
        model_config: ModelConfig,
    ) -> int | None:
        processor = cached_processor_from_config(model_config)
        hop_length = processor.feature_extractor.hop_length
        assert hop_length is not None
        return math.ceil(audio_duration_s * stt_config.sample_rate / hop_length)
```
**EN:** Method `FireRedASR2ForConditionalGeneration.get_num_audio_tokens` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FireRedASR2ForConditionalGeneration.get_num_audio_tokens` 封装了该模块中的一段可复用核心逻辑。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **KV cache / KV 缓存**
  - **EN:** The runtime stores key/value state so decoding can reuse prior computation efficiently.
  - **CN:** 运行时会保存 key/value 状态，以便解码阶段高效复用历史计算。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, cast`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import (`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import ModelConfig, SpeechToTextConfig, VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.config.speech_to_text import SpeechToTextParams`, `from vllm.inputs import MultiModalDataDict, PromptType`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.activation import _ACTIVATION_REGISTRY`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.models.whisper_utils import (`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import MultiModalDataItems, MultiModalDataParser`
