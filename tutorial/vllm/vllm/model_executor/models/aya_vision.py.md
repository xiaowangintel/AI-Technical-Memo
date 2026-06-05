# aya_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/aya_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for aya_vision, including encoder/decoder glue and vLLM runtime adaptation. / 面向 aya_vision 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-46)
```python
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Literal

import torch
from torch import nn
from transformers import BatchFeature, GotOcr2ImageProcessor
from transformers.activations import ACT2FN
from transformers.image_processing_utils import get_size_dict
from transformers.models.aya_vision import AyaVisionConfig
from transformers.models.aya_vision.processing_aya_vision import AyaVisionProcessor
from transformers.models.got_ocr2.image_processing_got_ocr2 import (
    get_optimal_tiled_canvas,
)

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import ImageProcessorItems, ImageSize, MultiModalDataItems
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
    PromptUpdateDetails,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .siglip import SiglipVisionModel
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    get_layer_index,
    init_vllm_registered_model,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `_get_num_hidden_layers` (lines 296-307)
```python
def _get_num_hidden_layers(hf_config: AyaVisionConfig) -> int:
    feature_layers = hf_config.vision_feature_layer
    num_hidden_layers = hf_config.vision_config.num_hidden_layers
    # If we have one feature layer, initialize up to that layer
    if isinstance(feature_layers, int):
        return get_layer_index(feature_layers, num_hidden_layers)
    # If we have multiple feature layers, initialize up to the deepest m
    elif isinstance(feature_layers, (list, tuple)):
        return max(get_layer_index(idx, num_hidden_layers) for idx in feature_layers)
    raise TypeError(
        f"vision_layer_feature type: {type(feature_layers)} is not supported"
    )
```
**EN:** Function `_get_num_hidden_layers` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_get_num_hidden_layers` 封装了该模块中的一段可复用核心逻辑。

### Class `AyaVisionImagePixelInputs` (lines 49-70)
```python
class AyaVisionImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - np: The total number of patches over each image over each prompt in
              the batch
        - c: Number of channels
        - h: Height of each image patch
        - w: Width of each image patch
        - bn: Batch size * number of images
    """

    type: Literal["pixel_values"]

    pixel_values: Annotated[
        torch.Tensor,
        TensorShape("np", 3, "h", "w"),
    ]

    num_patches: Annotated[
        torch.Tensor,
        TensorShape("bn"),
    ]
```
**EN:** Class `AyaVisionImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `AyaVisionImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `AyaVisionMultiModalProjector` (lines 73-133)
```python
class AyaVisionMultiModalProjector(nn.Module):
    def __init__(self, config: AyaVisionConfig):
        super().__init__()
        self.config = config
        self.downsample_factor = config.downsample_factor
        self.alignment_intermediate_size = getattr(
            config, "alignment_intermediate_size", config.text_config.hidden_size
        )
        self.layernorm = nn.LayerNorm(
            config.vision_config.hidden_size * (config.downsample_factor**2),
            eps=config.adapter_layer_norm_eps,
        )

        self.linear_1 = nn.Linear(
            config.vision_config.hidden_size * (config.downsample_factor**2),
            self.alignment_intermediate_size,
            bias=True,
        )

        self.act = ACT2FN["silu"]  # SwiGLU uses SiLU activation
        # For SwiGLU, project down to half size since we split intermediate dim
        self.linear_2 = nn.Linear(
            self.alignment_intermediate_size // 2,
            config.text_config.hidden_size,
            bias=True,
```
**EN:** Class `AyaVisionMultiModalProjector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward, pixel_shuffle.
**CN:** 类 `AyaVisionMultiModalProjector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward, pixel_shuffle。

### Method `AyaVisionMultiModalProjector.__init__` (lines 74-98)
```python
    def __init__(self, config: AyaVisionConfig):
        super().__init__()
        self.config = config
        self.downsample_factor = config.downsample_factor
        self.alignment_intermediate_size = getattr(
            config, "alignment_intermediate_size", config.text_config.hidden_size
        )
        self.layernorm = nn.LayerNorm(
            config.vision_config.hidden_size * (config.downsample_factor**2),
            eps=config.adapter_layer_norm_eps,
        )

        self.linear_1 = nn.Linear(
            config.vision_config.hidden_size * (config.downsample_factor**2),
            self.alignment_intermediate_size,
            bias=True,
        )

        self.act = ACT2FN["silu"]  # SwiGLU uses SiLU activation
        # For SwiGLU, project down to half size since we split intermediate dim
        self.linear_2 = nn.Linear(
            self.alignment_intermediate_size // 2,
            config.text_config.hidden_size,
            bias=True,
        )
```
**EN:** Method `AyaVisionMultiModalProjector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `AyaVisionMultiModalProjector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `AyaVisionMultiModalProjector.forward` (lines 100-110)
```python
    def forward(self, image_features: torch.Tensor) -> torch.Tensor:
        image_features = self.pixel_shuffle(image_features)
        image_features = self.layernorm(image_features)
        hidden_states = self.linear_1(image_features)

        # Split along last dimension and apply SwiGLU
        x, gate = hidden_states.chunk(2, dim=-1)
        hidden_states = self.act(gate) * x

        hidden_states = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** Method `AyaVisionMultiModalProjector.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `AyaVisionMultiModalProjector.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `AyaVisionMultiModalProjector.pixel_shuffle` (lines 112-133)
```python
    def pixel_shuffle(self, image_features: torch.Tensor) -> torch.Tensor:  # B, S, D
        batch_size, seq_length, _ = image_features.shape
        height = width = int(seq_length**0.5)
        image_features = image_features.reshape(
            image_features.shape[0], width, height, -1
        )
        channels = image_features.shape[-1]
        image_features = image_features.reshape(
            batch_size,
            width,
            int(height / self.downsample_factor),
            int(channels * self.downsample_factor),
        )
        image_features = image_features.permute(0, 2, 1, 3)
        image_features = image_features.reshape(
            batch_size,
            int(height / self.downsample_factor),
            int(width / self.downsample_factor),
            -1,
        )
        image_features = image_features.permute(0, 2, 1, 3)
        return image_features
```
**EN:** Method `AyaVisionMultiModalProjector.pixel_shuffle` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AyaVisionMultiModalProjector.pixel_shuffle` 封装了该模块中的一段可复用核心逻辑。

### Class `AyaVisionProcessingInfo` (lines 136-178)
```python
class AyaVisionProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self) -> AyaVisionConfig:
        return self.ctx.get_hf_config(AyaVisionConfig)

    def get_hf_processor(self, **kwargs: object) -> AyaVisionProcessor:
        return self.ctx.get_hf_processor(AyaVisionProcessor, **kwargs)

    def get_image_processor(self, **kwargs: object) -> GotOcr2ImageProcessor:
        return self.get_hf_processor(**kwargs).image_processor

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_image_size_with_most_features(self) -> ImageSize:
        image_processor = self.get_image_processor()
        height = image_processor.size["height"]
        width = image_processor.size["width"]
        max_patches = image_processor.max_patches
        return ImageSize(height=height * max_patches, width=width * max_patches)

    def get_num_patches(
        self,
        *,
        image_width: int,
        image_height: int,
```
**EN:** Class `AyaVisionProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_hf_processor, get_image_processor, get_supported_mm_limits, get_image_size_with_most_features, get_num_patches.
**CN:** 类 `AyaVisionProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor, get_image_processor, get_supported_mm_limits, get_image_size_with_most_features, get_num_patches。

### Method `AyaVisionProcessingInfo.get_hf_config` (lines 137-138)
```python
    def get_hf_config(self) -> AyaVisionConfig:
        return self.ctx.get_hf_config(AyaVisionConfig)
```
**EN:** Method `AyaVisionProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AyaVisionProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `AyaVisionProcessingInfo.get_hf_processor` (lines 140-141)
```python
    def get_hf_processor(self, **kwargs: object) -> AyaVisionProcessor:
        return self.ctx.get_hf_processor(AyaVisionProcessor, **kwargs)
```
**EN:** Method `AyaVisionProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AyaVisionProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `AyaVisionProcessingInfo.get_image_processor` (lines 143-144)
```python
    def get_image_processor(self, **kwargs: object) -> GotOcr2ImageProcessor:
        return self.get_hf_processor(**kwargs).image_processor
```
**EN:** Method `AyaVisionProcessingInfo.get_image_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AyaVisionProcessingInfo.get_image_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `AyaVisionProcessingInfo.get_supported_mm_limits` (lines 146-147)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Method `AyaVisionProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AyaVisionProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `AyaVisionDummyInputsBuilder` (lines 181-208)
```python
class AyaVisionDummyInputsBuilder(BaseDummyInputsBuilder[AyaVisionProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.image_token

        return image_token * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        image_size = self.info.get_image_size_with_most_features()

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=image_size.width,
                height=image_size.height,
                num_images=num_images,
```
**EN:** Class `AyaVisionDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[AyaVisionProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `AyaVisionDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[AyaVisionProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `AyaVisionDummyInputsBuilder.get_dummy_text` (lines 182-188)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.image_token

        return image_token * num_images
```
**EN:** Method `AyaVisionDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AyaVisionDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `AyaVisionDummyInputsBuilder.get_dummy_mm_data` (lines 190-208)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        image_size = self.info.get_image_size_with_most_features()

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=image_size.width,
                height=image_size.height,
                num_images=num_images,
                overrides=image_overrides,
            )
        }
```
**EN:** Method `AyaVisionDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AyaVisionDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `AyaVisionMultiModalProcessor` (lines 211-293)
```python
class AyaVisionMultiModalProcessor(BaseMultiModalProcessor[AyaVisionProcessingInfo]):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        processed_outputs = super()._call_hf_processor(
            prompt,
            mm_data,
            mm_kwargs,
            tok_kwargs,
        )
        hf_processor = self.info.get_hf_processor(**mm_kwargs)
        image_processor = hf_processor.image_processor

        # HF processor pops the `num_patches` kwarg, which is needed by vLLM
        if (images := mm_data.get("images")) is not None:
            mm_items = self.info.parse_mm_data({"image": images}, validate=False)
            parsed_images = mm_items.get_items("image", ImageProcessorItems)
            image_sizes = [
                parsed_images.get_image_size(i) for i in range(len(parsed_images))
            ]
```
**EN:** Class `AyaVisionMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[AyaVisionProcessingInfo]. Key methods include _call_hf_processor, _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `AyaVisionMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[AyaVisionProcessingInfo]。 关键方法包括 _call_hf_processor, _get_mm_fields_config, _get_prompt_updates。

### Class `AyaVisionForConditionalGeneration` (lines 315-441)
```python
@MULTIMODAL_REGISTRY.register_processor(
    AyaVisionMultiModalProcessor,
    info=AyaVisionProcessingInfo,
    dummy_inputs=AyaVisionDummyInputsBuilder,
)
class AyaVisionForConditionalGeneration(nn.Module, SupportsMultiModal, SupportsPP):
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            # mapping for new names in checkpoint saved after transformers v4.52
            "model.language_model.": "language_model.model.",
            "model.vision_tower.": "vision_tower.",
            "model.multi_modal_projector.": "multi_modal_projector.",
            "lm_head.": "language_model.lm_head.",
        }
    )

    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"

        raise ValueError("Only image modality is supported")

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
```
**EN:** Class `AyaVisionForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods include get_placeholder_str, __init__, dtype, load_weights, _image_pixels_to_features, _process_image_input.
**CN:** 类 `AyaVisionForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 关键方法包括 get_placeholder_str, __init__, dtype, load_weights, _image_pixels_to_features, _process_image_input。

### Method `AyaVisionForConditionalGeneration.get_placeholder_str` (lines 327-331)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"

        raise ValueError("Only image modality is supported")
```
**EN:** Method `AyaVisionForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AyaVisionForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `AyaVisionForConditionalGeneration.__init__` (lines 333-359)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config: AyaVisionConfig = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        num_hidden_layers = _get_num_hidden_layers(config)
        self.config = config
        self.quant_config = quant_config
        self.multimodal_config = multimodal_config

        with self._mark_tower_model(vllm_config, "image"):
            self.vision_tower = SiglipVisionModel(
                config.vision_config,
                quant_config,
                num_hidden_layers_override=num_hidden_layers,
                prefix=maybe_prefix(prefix, "vision_model"),
            )
            self.multi_modal_projector = AyaVisionMultiModalProjector(config)

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=config.text_config,
                prefix=maybe_prefix(prefix, "model"),
                # Cohere2ForCausalLM and CohereForCausalLM are the same on vllm
                architectures=["Cohere2ForCausalLM"],
            )
```
**EN:** Method `AyaVisionForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `AyaVisionForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `AyaVisionForConditionalGeneration.dtype` (lines 362-363)
```python
    @property
    def dtype(self):
        return next(self.parameters()).dtype
```
**EN:** Method `AyaVisionForConditionalGeneration.dtype` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AyaVisionForConditionalGeneration.dtype` 封装了该模块中的一段可复用核心逻辑。

### Method `AyaVisionForConditionalGeneration.load_weights` (lines 365-367)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights, mapper=self.hf_to_vllm_mapper)
```
**EN:** Method `AyaVisionForConditionalGeneration.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `AyaVisionForConditionalGeneration.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Literal`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import BatchFeature, GotOcr2ImageProcessor`, `from transformers.activations import ACT2FN`, `from transformers.image_processing_utils import get_size_dict`, `from transformers.models.aya_vision import AyaVisionConfig`, `from transformers.models.aya_vision.processing_aya_vision import AyaVisionProcessor`, `from transformers.models.got_ocr2.image_processing_got_ocr2 import (`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import ImageProcessorItems, ImageSize, MultiModalDataItems`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`, `from vllm.utils.tensor_schema import TensorSchema, TensorShape`, `from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP`, `from .siglip import SiglipVisionModel`, `from .utils import (`
