# gemma3n_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/gemma3n_mm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for gemma3n_mm, including architecture wrappers and weight loading logic. / 面向推理的 gemma3n_mm vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-64)
```python
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Any, Literal

import torch
from torch import nn
from transformers import AutoModel, BatchFeature
from transformers.models.gemma3n import (
    Gemma3nAudioConfig,
    Gemma3nAudioFeatureExtractor,
    Gemma3nConfig,
    Gemma3nProcessor,
    Gemma3nTextConfig,
    Gemma3nVisionConfig,
)
from transformers.models.siglip import SiglipImageProcessorFast

from vllm.config import ModelConfig, SpeechToTextConfig, VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.config.speech_to_text import SpeechToTextParams
from vllm.inputs import MultiModalDataDict, PromptType, TextPrompt
from vllm.logger import init_logger
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import RowParallelLinear
from vllm.model_executor.layers.vocab_parallel_embedding import VocabParallelEmbedding
from vllm.model_executor.models.gemma3n import Gemma3nForCausalLM
from vllm.model_executor.models.gemma3n_audio_utils import (
    adjust_audio_features_to_expected_length,
)
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.model_executor.models.whisper import ISO639_1_SUPPORTED_LANGS
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import (
    ImageProcessorItems,
    MultiModalDataItems,
    MultiModalDataParser,
)
from vllm.multimodal.processing import BaseDummyInputsBuilder
from vllm.multimodal.processing.processor import (
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    MultiModalPromptUpdates,
    MultiModalPromptUpdatesApplyResult,
    PlaceholderFeaturesInfo,
    PromptReplacement,
    PromptUpdate,
    PromptUpdateDetails,
    replace_token_matches,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsTranscription
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 69-69)
```python
TOKENS_PER_IMAGE = 256
```
**EN:** This block defines TOKENS_PER_IMAGE, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 TOKENS_PER_IMAGE，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 70-70)
```python
TOKENS_PER_AUDIO = 188
```
**EN:** This block defines TOKENS_PER_AUDIO, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 TOKENS_PER_AUDIO，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Class `Gemma3nImagePixelInputs` (lines 73-83)
```python
class Gemma3nImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - c: Number of channels (3)
        - h: Height of each patch
        - w: Width of each patch
    """

    type: Literal["pixel_values"] = "pixel_values"
    pixel_values: Annotated[torch.Tensor, TensorShape("bn", 3, "h", "w")]
```
**EN:** Class `Gemma3nImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Gemma3nImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Gemma3nAudioInputs` (lines 86-96)
```python
class Gemma3nAudioInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of audios
        - s: seq_length
        - f: num_features
    """

    type: Literal["audio"] = "audio"
    input_features_padded: Annotated[torch.Tensor, TensorShape("bn", "s", "f")]
    input_features_mask: Annotated[torch.Tensor, TensorShape("bn", "s")]
```
**EN:** Class `Gemma3nAudioInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Gemma3nAudioInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Gemma3nProcessingInfo` (lines 102-159)
```python
class Gemma3nProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(Gemma3nConfig)

    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(Gemma3nProcessor, **kwargs)

    def get_feature_extractor(self, **kwargs: object) -> Gemma3nAudioFeatureExtractor:
        return self.get_hf_processor(**kwargs).feature_extractor

    def get_data_parser(self):
        feature_extractor = self.get_feature_extractor()

        return MultiModalDataParser(
            target_sr=feature_extractor.sampling_rate,
            expected_hidden_size=self._get_expected_hidden_size(),
        )

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None, "audio": None}

    def get_max_tokens_per_item(
        self, seq_len: int, mm_counts: Mapping[str, int]
    ) -> Mapping[str, int] | None:
        return {"image": TOKENS_PER_IMAGE, "audio": TOKENS_PER_AUDIO}
```
**EN:** Class `Gemma3nProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_hf_processor, get_feature_extractor, get_data_parser, get_supported_mm_limits, get_max_tokens_per_item.
**CN:** 类 `Gemma3nProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor, get_feature_extractor, get_data_parser, get_supported_mm_limits, get_max_tokens_per_item。

### Method `Gemma3nProcessingInfo.get_hf_config` (lines 103-104)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config(Gemma3nConfig)
```
**EN:** Method `Gemma3nProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3nProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `Gemma3nProcessingInfo.get_hf_processor` (lines 106-107)
```python
    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(Gemma3nProcessor, **kwargs)
```
**EN:** Method `Gemma3nProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3nProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Class `Gemma3nDummyInputsBuilder` (lines 162-205)
```python
class Gemma3nDummyInputsBuilder(BaseDummyInputsBuilder[Gemma3nProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        num_audios = mm_counts.get("audio", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.image_token
        audio_token = processor.audio_token

        return image_token * num_images + audio_token * num_audios

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        num_audios = mm_counts.get("audio", 0)
        processor = self.info.get_hf_processor()
        audio_feature_extractor: Gemma3nAudioFeatureExtractor = (
            processor.feature_extractor
        )
        audio_len = audio_feature_extractor.fft_length
        image_processor: SiglipImageProcessorFast = processor.image_processor
```
**EN:** Class `Gemma3nDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[Gemma3nProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `Gemma3nDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[Gemma3nProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `Gemma3nDummyInputsBuilder.get_dummy_text` (lines 163-171)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        num_audios = mm_counts.get("audio", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.image_token
        audio_token = processor.audio_token

        return image_token * num_images + audio_token * num_audios
```
**EN:** Method `Gemma3nDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3nDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `Gemma3nDummyInputsBuilder.get_dummy_mm_data` (lines 173-205)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        num_audios = mm_counts.get("audio", 0)
        processor = self.info.get_hf_processor()
        audio_feature_extractor: Gemma3nAudioFeatureExtractor = (
            processor.feature_extractor
        )
        audio_len = audio_feature_extractor.fft_length
        image_processor: SiglipImageProcessorFast = processor.image_processor
        img_width = image_processor.size.get("width", 224)
        img_height = image_processor.size.get("height", 224)

        image_overrides = mm_options.get("image")
        audio_overrides = mm_options.get("audio")

        return {
            "image": self._get_dummy_images(
                width=img_width,
                height=img_height,
                num_images=num_images,
                overrides=image_overrides,
            ),
            "audio": self._get_dummy_audios(
                length=audio_len,
                num_audios=num_audios,
                overrides=audio_overrides,
            ),
        }
```
**EN:** Method `Gemma3nDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3nDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `Gemma3nMultiModalProcessor` (lines 208-385)
```python
class Gemma3nMultiModalProcessor(BaseMultiModalProcessor[Gemma3nProcessingInfo]):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        # HF Transformers audio processor no longer accepts `audios` key.
        # We pop `audios` and replace it with `audio` key to suppress
        # the warning.
        if "audios" in mm_data:
            mm_data["audio"] = mm_data.pop("audios")
        processed_outputs = super()._call_hf_processor(
            prompt,
            mm_data,
            mm_kwargs,
            tok_kwargs,
        )

        if "input_features" in processed_outputs:
            # Padding enables audio_tower to run in batched mode
            processed_outputs["input_features_padded"] = processed_outputs[
                "input_features"
            ]
```
**EN:** Class `Gemma3nMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[Gemma3nProcessingInfo]. Key methods include _call_hf_processor, _get_mm_fields_config, _get_prompt_updates, _apply_token_matches, _find_mm_placeholders.
**CN:** 类 `Gemma3nMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[Gemma3nProcessingInfo]。 关键方法包括 _call_hf_processor, _get_mm_fields_config, _get_prompt_updates, _apply_token_matches, _find_mm_placeholders。

### Class `Gemma3nMultimodalEmbedder` (lines 388-460)
```python
class Gemma3nMultimodalEmbedder(nn.Module):
    """Embeds token ids or soft tokens for multimodal content into language
    model space."""

    def __init__(
        self,
        multimodal_config: Gemma3nAudioConfig | Gemma3nVisionConfig,
        text_config: Gemma3nTextConfig,
    ):
        super().__init__()

        self.multimodal_hidden_size = multimodal_config.hidden_size
        self.eps = multimodal_config.rms_norm_eps
        self.vocab_offset = multimodal_config.vocab_offset
        self.vocab_size = multimodal_config.vocab_size
        self.text_hidden_size = text_config.hidden_size

        self.embedding = VocabParallelEmbedding(
            self.vocab_size,
            self.multimodal_hidden_size,
        )

        self.hard_embedding_norm = RMSNorm(
            self.multimodal_hidden_size,
            eps=self.eps,
```
**EN:** Class `Gemma3nMultimodalEmbedder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma3nMultimodalEmbedder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma3nMultimodalEmbedder.__init__` (lines 392-431)
```python
    def __init__(
        self,
        multimodal_config: Gemma3nAudioConfig | Gemma3nVisionConfig,
        text_config: Gemma3nTextConfig,
    ):
        super().__init__()

        self.multimodal_hidden_size = multimodal_config.hidden_size
        self.eps = multimodal_config.rms_norm_eps
        self.vocab_offset = multimodal_config.vocab_offset
        self.vocab_size = multimodal_config.vocab_size
        self.text_hidden_size = text_config.hidden_size

        self.embedding = VocabParallelEmbedding(
            self.vocab_size,
            self.multimodal_hidden_size,
        )

        self.hard_embedding_norm = RMSNorm(
            self.multimodal_hidden_size,
            eps=self.eps,
        )

        self.soft_embedding_norm = RMSNorm(
            self.multimodal_hidden_size,
            eps=self.eps,
        )

        self.embedding_projection = RowParallelLinear(
            self.multimodal_hidden_size,
            self.text_hidden_size,
            bias=False,
            input_is_parallel=False,  # scatter the full-width input internally
        )

        self.embedding_post_projection_norm = RMSNorm(
            self.text_hidden_size,
            eps=self.eps,
            has_weight=False,
        )
```
**EN:** Method `Gemma3nMultimodalEmbedder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nMultimodalEmbedder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nMultimodalEmbedder.forward` (lines 433-460)
```python
    def forward(
        self,
        input_ids: torch.LongTensor | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """Embeds token ids or soft tokens for multimodal content into language model space.

        Args:
            input_ids: A torch.LongTensor containing the token ids to embed. Values should be in the range
                `[vocab_offset, vocab_offset + vocab_size)`.
            inputs_embeds: A torch.Tensor containing the soft tokens to embed.

        Returns:
            A torch.Tensor of embeddings with  shape `[batch_size, seq_len, self.config.text_config.hidden_size]`.
        """  # noqa: E501
        if (input_ids is None) ^ (inputs_embeds is not None):
            raise ValueError(
                "You must specify exactly one of input_ids or inputs_embeds"
            )

        if inputs_embeds is not None:
            emb_norm = self.soft_embedding_norm(inputs_embeds)
        else:
            hard_emb = self.embedding(input_ids - self.vocab_offset)
            emb_norm = self.hard_embedding_norm(hard_emb)

        emb_norm_proj, _ = self.embedding_projection(emb_norm)
        return self.embedding_post_projection_norm(emb_norm_proj)
```
**EN:** Method `Gemma3nMultimodalEmbedder.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Embeds token ids or soft tokens for multimodal content into language model space.
**CN:** Method `Gemma3nMultimodalEmbedder.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Embeds token ids or soft tokens for multimodal content into language model space。

### Class `Gemma3nForConditionalGeneration` (lines 468-821)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Gemma3nMultiModalProcessor,
    info=Gemma3nProcessingInfo,
    dummy_inputs=Gemma3nDummyInputsBuilder,
)
class Gemma3nForConditionalGeneration(
    nn.Module, SupportsMultiModal, SupportsTranscription
):
    supported_languages = ISO639_1_SUPPORTED_LANGS

    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
        "gate_up_proj": [
            "gate_proj",
            "up_proj",
        ],
    }

    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            # mapping for new names in checkpoint saved after transformers v4.52
```
**EN:** Class `Gemma3nForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsTranscription. Key methods include __init__, _parse_and_validate_image_input, _parse_and_validate_audio_input, _parse_and_validate_multimodal_inputs, _process_image_input, _process_audio_input.
**CN:** 类 `Gemma3nForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsTranscription。 关键方法包括 __init__, _parse_and_validate_image_input, _parse_and_validate_audio_input, _parse_and_validate_multimodal_inputs, _process_image_input, _process_audio_input。

### Method `Gemma3nForConditionalGeneration.__init__` (lines 499-537)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = config
        self.quant_config = quant_config
        self.multimodal_config = multimodal_config
        self.vocab_size = config.text_config.vocab_size

        with self._mark_tower_model(vllm_config, "image"):
            self.vision_tower = AutoModel.from_config(config=config.vision_config)
            self.embed_vision = Gemma3nMultimodalEmbedder(
                config.vision_config, config.text_config
            )

        with self._mark_tower_model(vllm_config, "audio"):
            self.audio_tower = AutoModel.from_config(config=config.audio_config)
            self.embed_audio = Gemma3nMultimodalEmbedder(
                config.audio_config, config.text_config
            )

        with self._mark_language_model(vllm_config):
            self.language_model: Gemma3nForCausalLM = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=config.text_config,
                prefix=maybe_prefix(prefix, "language_model"),
                architectures=["Gemma3nForCausalLM"],
            )

            # NOTE (NickLucche) In order to be compatible with cudagraph, the
            # buffer needs to be consistent, so we pre-allocate here.
            self.per_layer_embeddings = torch.zeros(
                vllm_config.scheduler_config.max_num_batched_tokens,
                self.config.text_config.num_hidden_layers,
                self.config.text_config.hidden_size_per_layer_input,
                device=self.language_model.model.embed_tokens.weight.device,
                dtype=self.language_model.model.embed_tokens.weight.dtype,
            )
```
**EN:** Method `Gemma3nForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nForConditionalGeneration.embed_multimodal` (lines 663-680)
```python
    def embed_multimodal(self, **kwargs: object) -> MultiModalEmbeddings:
        mm_input_by_modality = self._parse_and_validate_multimodal_inputs(**kwargs)
        if mm_input_by_modality is None:
            return []

        multimodal_embeddings: list[torch.Tensor] = []

        # NOTE: It is important to iterate over the keys in this dictionary
        # to preserve the order of the modalities.
        for modality in mm_input_by_modality:
            multimodal_input = mm_input_by_modality[modality]
            if modality == "image":
                vision_embeddings = self._process_image_input(multimodal_input)
                multimodal_embeddings.extend(vision_embeddings)
            if modality == "audio":
                audio_embeddings = self._process_audio_input(multimodal_input)
                multimodal_embeddings.extend(audio_embeddings)
        return multimodal_embeddings
```
**EN:** Method `Gemma3nForConditionalGeneration.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma3nForConditionalGeneration.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

## Key Concepts / 关键概念
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
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Any, Literal`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import AutoModel, BatchFeature`, `from transformers.models.gemma3n import (`, `from transformers.models.siglip import SiglipImageProcessorFast`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import ModelConfig, SpeechToTextConfig, VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.config.speech_to_text import SpeechToTextParams`, `from vllm.inputs import MultiModalDataDict, PromptType, TextPrompt`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import RowParallelLinear`, `from vllm.model_executor.layers.vocab_parallel_embedding import VocabParallelEmbedding`, `from vllm.model_executor.models.gemma3n import Gemma3nForCausalLM`, `from vllm.model_executor.models.gemma3n_audio_utils import (`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.model_executor.models.whisper import ISO639_1_SUPPORTED_LANGS`
