# gemma3_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/gemma3_mm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for gemma3_mm, including architecture wrappers and weight loading logic. / 面向推理的 gemma3_mm vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-52)
```python
import math
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Any, Literal

import torch
from torch import nn
from transformers import BatchFeature, Gemma3Config, Gemma3Processor
from transformers.models.gemma3.image_processing_gemma3 import Gemma3ImageProcessor
from transformers.models.gemma3.processing_gemma3 import Gemma3ProcessorKwargs

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.logger import init_logger
from vllm.model_executor.layers.layernorm import GemmaRMSNorm
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import ImageProcessorItems, ImageSize, MultiModalDataItems
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

from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from .siglip import SiglipVisionModel
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `Gemma3ImagePixelInputs` (lines 57-72)
```python
class Gemma3ImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - p: Number of patches total (over each image over each prompt in the
          batch)
        - c: Number of channels (3)
        - h: Height of each patch
        - w: Width of each patch
        - bn: Batch size * number of images
    """

    type: Literal["pixel_values"] = "pixel_values"

    pixel_values: Annotated[torch.Tensor, TensorShape("p", 3, "h", "w")]

    num_patches: Annotated[torch.Tensor, TensorShape("bn")]
```
**EN:** Class `Gemma3ImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Gemma3ImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Gemma3ProcessingInfo` (lines 78-228)
```python
class Gemma3ProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(Gemma3Config)

    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(Gemma3Processor, **kwargs)

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_num_crops(
        self,
        *,
        image_width: int,
        image_height: int,
        processor: Gemma3Processor,
        mm_kwargs: Mapping[str, object],
    ) -> int:
        image_processor: Gemma3ImageProcessor = processor.image_processor

        images_kwargs = processor._merge_kwargs(
            Gemma3ProcessorKwargs,
            tokenizer_init_kwargs=processor.tokenizer.init_kwargs,
            **self.ctx.get_merged_mm_kwargs(mm_kwargs),
        )["images_kwargs"]
```
**EN:** Class `Gemma3ProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_hf_processor, get_supported_mm_limits, get_num_crops, get_image_repl, get_num_image_tokens.
**CN:** 类 `Gemma3ProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor, get_supported_mm_limits, get_num_crops, get_image_repl, get_num_image_tokens。

### Method `Gemma3ProcessingInfo.get_hf_config` (lines 79-80)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config(Gemma3Config)
```
**EN:** Method `Gemma3ProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3ProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `Gemma3ProcessingInfo.get_hf_processor` (lines 82-83)
```python
    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(Gemma3Processor, **kwargs)
```
**EN:** Method `Gemma3ProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3ProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `Gemma3ProcessingInfo.get_supported_mm_limits` (lines 85-86)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Method `Gemma3ProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3ProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Method `Gemma3ProcessingInfo.get_num_crops` (lines 88-158)
```python
    def get_num_crops(
        self,
        *,
        image_width: int,
        image_height: int,
        processor: Gemma3Processor,
        mm_kwargs: Mapping[str, object],
    ) -> int:
        image_processor: Gemma3ImageProcessor = processor.image_processor

        images_kwargs = processor._merge_kwargs(
            Gemma3ProcessorKwargs,
            tokenizer_init_kwargs=processor.tokenizer.init_kwargs,
            **self.ctx.get_merged_mm_kwargs(mm_kwargs),
        )["images_kwargs"]

        do_pan_and_scan = images_kwargs.get(
            "do_pan_and_scan", image_processor.do_pan_and_scan
        )
        pan_and_scan_min_crop_size = images_kwargs.get(
            "pan_and_scan_min_crop_size", image_processor.pan_and_scan_min_crop_size
        )
        pan_and_scan_max_num_crops = images_kwargs.get(
            "pan_and_scan_max_num_crops", image_processor.pan_and_scan_max_num_crops
        )
        pan_and_scan_min_ratio_to_activate = images_kwargs.get(
            "pan_and_scan_min_ratio_to_activate",
            image_processor.pan_and_scan_min_ratio_to_activate,
        )

        if not do_pan_and_scan:
            return 0

        logger.warning_once(
            "`do_pan_and_scan=True` has suboptimal results on V1 "
            "because of the simplified attention pattern being used."
        )

        # Based on Gemma3ImageProcessor.pan_and_scan
        if image_width >= image_height:
            if image_width / image_height < pan_and_scan_min_ratio_to_activate:
                return 0

            num_crops_w = min(
                int(math.floor(image_width / pan_and_scan_min_crop_size)),
                int(math.floor(image_width / image_height + 0.5)),
            )

            num_crops_w = max(2, num_crops_w)
            num_crops_w = min(pan_and_scan_max_num_crops, num_crops_w)
            num_crops_h = 1
        else:
            if image_height / image_width < pan_and_scan_min_ratio_to_activate:
                return 0

            num_crops_h = min(
                int(math.floor(image_height / pan_and_scan_min_crop_size)),
                int(math.floor(image_height / image_width + 0.5)),
            )

# ... truncated for analysis ...
```
**EN:** Method `Gemma3ProcessingInfo.get_num_crops` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3ProcessingInfo.get_num_crops` 封装了该模块中的一段可复用核心逻辑。

### Class `Gemma3DummyInputsBuilder` (lines 231-259)
```python
class Gemma3DummyInputsBuilder(BaseDummyInputsBuilder[Gemma3ProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.boi_token

        return image_token * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)

        target_width, target_height = self.info.get_image_size_with_most_features()

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=target_width,
                height=target_height,
```
**EN:** Class `Gemma3DummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[Gemma3ProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `Gemma3DummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[Gemma3ProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `Gemma3DummyInputsBuilder.get_dummy_text` (lines 232-238)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.boi_token

        return image_token * num_images
```
**EN:** Method `Gemma3DummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3DummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `Gemma3DummyInputsBuilder.get_dummy_mm_data` (lines 240-259)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)

        target_width, target_height = self.info.get_image_size_with_most_features()

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=target_width,
                height=target_height,
                num_images=num_images,
                overrides=image_overrides,
            )
        }
```
**EN:** Method `Gemma3DummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3DummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `Gemma3MultiModalProcessor` (lines 262-417)
```python
class Gemma3MultiModalProcessor(BaseMultiModalProcessor[Gemma3ProcessingInfo]):
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

        # HF processor pops the `num_crops` kwarg, which is needed by vLLM
        if (images := mm_data.get("images")) is not None:
            mm_items = self.info.parse_mm_data({"image": images}, validate=False)
            parsed_images = mm_items.get_items("image", ImageProcessorItems)
            image_sizes = [
                parsed_images.get_image_size(i) for i in range(len(parsed_images))
            ]
            hf_processor = self.info.get_hf_processor(**mm_kwargs)

            num_crops = [
```
**EN:** Class `Gemma3MultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[Gemma3ProcessingInfo]. Key methods include _call_hf_processor, _get_mm_fields_config, _get_prompt_updates, _apply_token_matches, _find_mm_placeholders.
**CN:** 类 `Gemma3MultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[Gemma3ProcessingInfo]。 关键方法包括 _call_hf_processor, _get_mm_fields_config, _get_prompt_updates, _apply_token_matches, _find_mm_placeholders。

### Class `Gemma3MultiModalProjector` (lines 420-461)
```python
class Gemma3MultiModalProjector(nn.Module):
    def __init__(self, config: Gemma3Config):
        super().__init__()

        self.mm_input_projection_weight = nn.Parameter(
            torch.zeros(
                config.vision_config.hidden_size, config.text_config.hidden_size
            )
        )

        self.mm_soft_emb_norm = GemmaRMSNorm(
            config.vision_config.hidden_size, eps=config.vision_config.layer_norm_eps
        )

        self.patches_per_image = int(
            config.vision_config.image_size // config.vision_config.patch_size
        )
        self.tokens_per_side = int(config.mm_tokens_per_image**0.5)
        self.kernel_size = self.patches_per_image // self.tokens_per_side
        self.avg_pool = nn.AvgPool2d(
            kernel_size=self.kernel_size, stride=self.kernel_size
        )

    def forward(self, vision_outputs: torch.Tensor):
        batch_size, _, seq_length = vision_outputs.shape
```
**EN:** Class `Gemma3MultiModalProjector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma3MultiModalProjector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma3MultiModalProjector.__init__` (lines 421-441)
```python
    def __init__(self, config: Gemma3Config):
        super().__init__()

        self.mm_input_projection_weight = nn.Parameter(
            torch.zeros(
                config.vision_config.hidden_size, config.text_config.hidden_size
            )
        )

        self.mm_soft_emb_norm = GemmaRMSNorm(
            config.vision_config.hidden_size, eps=config.vision_config.layer_norm_eps
        )

        self.patches_per_image = int(
            config.vision_config.image_size // config.vision_config.patch_size
        )
        self.tokens_per_side = int(config.mm_tokens_per_image**0.5)
        self.kernel_size = self.patches_per_image // self.tokens_per_side
        self.avg_pool = nn.AvgPool2d(
            kernel_size=self.kernel_size, stride=self.kernel_size
        )
```
**EN:** Method `Gemma3MultiModalProjector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3MultiModalProjector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3MultiModalProjector.forward` (lines 443-461)
```python
    def forward(self, vision_outputs: torch.Tensor):
        batch_size, _, seq_length = vision_outputs.shape

        reshaped_vision_outputs = vision_outputs.transpose(1, 2)
        reshaped_vision_outputs = reshaped_vision_outputs.reshape(
            batch_size, seq_length, self.patches_per_image, self.patches_per_image
        )
        reshaped_vision_outputs = reshaped_vision_outputs.contiguous()

        pooled_vision_outputs = self.avg_pool(reshaped_vision_outputs)
        pooled_vision_outputs = pooled_vision_outputs.flatten(2)
        pooled_vision_outputs = pooled_vision_outputs.transpose(1, 2)

        normed_vision_outputs = self.mm_soft_emb_norm(pooled_vision_outputs)

        projected_vision_outputs = torch.matmul(
            normed_vision_outputs, self.mm_input_projection_weight
        )
        return projected_vision_outputs.type_as(vision_outputs)
```
**EN:** Method `Gemma3MultiModalProjector.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma3MultiModalProjector.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma3ForConditionalGeneration` (lines 469-684)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Gemma3MultiModalProcessor,
    info=Gemma3ProcessingInfo,
    dummy_inputs=Gemma3DummyInputsBuilder,
)
class Gemma3ForConditionalGeneration(
    nn.Module, SupportsMultiModal, SupportsPP, SupportsLoRA
):
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
            "model.language_model.": "language_model.model.",
            "model.vision_tower.": "vision_tower.",
```
**EN:** Class `Gemma3ForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP, SupportsLoRA. Key methods include get_placeholder_str, __init__, dtype, _parse_and_validate_image_input, _image_pixels_to_features, _process_image_input.
**CN:** 类 `Gemma3ForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP、SupportsLoRA。 关键方法包括 get_placeholder_str, __init__, dtype, _parse_and_validate_image_input, _image_pixels_to_features, _process_image_input。

### Method `Gemma3ForConditionalGeneration.get_placeholder_str` (lines 495-499)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<start_of_image>"

        raise ValueError("Only image modality is supported")
```
**EN:** Method `Gemma3ForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3ForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `Gemma3ForConditionalGeneration.__init__` (lines 501-536)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = config
        self.quant_config = quant_config
        self.multimodal_config = multimodal_config

        self.configure_mm_token_handling(
            vocab_size=config.text_config.vocab_size,
            mm_token_ids=[config.image_token_index],
        )

        with self._mark_tower_model(vllm_config, "image"):
            self.vision_tower = SiglipVisionModel(
                config.vision_config,
                quant_config,
                prefix=maybe_prefix(prefix, "vision_tower"),
            )
            self.multi_modal_projector = Gemma3MultiModalProjector(config)

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=config.text_config,
                prefix=maybe_prefix(prefix, "language_model"),
                architectures=["Gemma3ForCausalLM"],
            )

            logit_scale = getattr(config, "logit_scale", 1.0)
            self.language_model.logits_processor.scale *= logit_scale

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )
```
**EN:** Method `Gemma3ForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3ForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3ForConditionalGeneration.dtype` (lines 539-540)
```python
    @property
    def dtype(self):
        return next(self.parameters()).dtype
```
**EN:** Method `Gemma3ForConditionalGeneration.dtype` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3ForConditionalGeneration.dtype` 封装了该模块中的一段可复用核心逻辑。

### Method `Gemma3ForConditionalGeneration.embed_multimodal` (lines 582-587)
```python
    def embed_multimodal(self, **kwargs: object) -> MultiModalEmbeddings:
        image_input = self._parse_and_validate_image_input(**kwargs)
        if image_input is None:
            return []

        return self._process_image_input(image_input)
```
**EN:** Method `Gemma3ForConditionalGeneration.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma3ForConditionalGeneration.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **LoRA support / LoRA 支持**
  - **EN:** The file is aware of low-rank adaptation interfaces or compatible parameter layouts.
  - **CN:** 该文件考虑了 LoRA 接口或兼容的低秩参数布局。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Any, Literal`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import BatchFeature, Gemma3Config, Gemma3Processor`, `from transformers.models.gemma3.image_processing_gemma3 import Gemma3ImageProcessor`, `from transformers.models.gemma3.processing_gemma3 import Gemma3ProcessorKwargs`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.layernorm import GemmaRMSNorm`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import ImageProcessorItems, ImageSize, MultiModalDataItems`, `from vllm.multimodal.processing import BaseDummyInputsBuilder`, `from vllm.multimodal.processing.processor import (`, `from vllm.sequence import IntermediateTensors`
