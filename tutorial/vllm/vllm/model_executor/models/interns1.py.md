# interns1.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/interns1.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for interns1, including architecture wrappers and weight loading logic. / 面向推理的 interns1 vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 9-64)
```python
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Literal, TypeAlias

import regex as re
import torch
import torch.nn as nn
from transformers import BatchFeature, InternVLProcessor, PretrainedConfig
from transformers.activations import ACT2FN
from transformers.models.got_ocr2.image_processing_got_ocr2_fast import (
    GotOcr2ImageProcessorFast,
)
from transformers.models.internvl.video_processing_internvl import (
    InternVLVideoProcessor,
)

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.models.interns1_vit import InternS1VisionModel
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import (
    ImageEmbeddingItems,
    ImageProcessorItems,
    ImageSize,
    MultiModalDataItems,
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
from vllm.transformers_utils.processor import cached_video_processor_from_config
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `resolve_interns1_min_max_num` (lines 150-162)
```python
def resolve_interns1_min_max_num(
    min_dynamic_patch: int,
    max_dynamic_patch: int,
    dynamic_image_size: bool,
    use_thumbnail: bool,
) -> tuple[int, int]:
    min_dynamic_patch = min_dynamic_patch if dynamic_image_size else 1
    max_dynamic_patch = max_dynamic_patch if dynamic_image_size else 1

    if use_thumbnail and max_dynamic_patch != 1:
        max_dynamic_patch += 1

    return min_dynamic_patch, max_dynamic_patch
```
**EN:** Function `resolve_interns1_min_max_num` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `resolve_interns1_min_max_num` 封装了该模块中的一段可复用核心逻辑。

### Function `get_interns1_target_ratios` (lines 165-176)
```python
def get_interns1_target_ratios(
    min_num: int,
    max_num: int,
) -> list[tuple[int, int]]:
    target_ratios = {
        (i, j)
        for n in range(min_num, max_num + 1)
        for i in range(1, n + 1)
        for j in range(1, n + 1)
        if min_num <= i * j <= max_num
    }
    return sorted(target_ratios, key=lambda x: x[0] * x[1])
```
**EN:** Function `get_interns1_target_ratios` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `get_interns1_target_ratios` 封装了该模块中的一段可复用核心逻辑。

### Class `InternS1MultiModalProjector` (lines 67-87)
```python
class InternS1MultiModalProjector(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.layer_norm = nn.LayerNorm(
            config.vision_config.hidden_size * int(1 / config.downsample_ratio) ** 2
        )
        self.linear_1 = nn.Linear(
            config.vision_config.hidden_size * int(1 / config.downsample_ratio) ** 2,
            config.text_config.hidden_size,
        )
        self.act = ACT2FN[config.projector_hidden_act]
        self.linear_2 = nn.Linear(
            config.text_config.hidden_size, config.text_config.hidden_size
        )

    def forward(self, image_features):
        hidden_states = self.layer_norm(image_features)
        hidden_states = self.linear_1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** Class `InternS1MultiModalProjector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `InternS1MultiModalProjector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `InternS1MultiModalProjector.__init__` (lines 68-80)
```python
    def __init__(self, config):
        super().__init__()
        self.layer_norm = nn.LayerNorm(
            config.vision_config.hidden_size * int(1 / config.downsample_ratio) ** 2
        )
        self.linear_1 = nn.Linear(
            config.vision_config.hidden_size * int(1 / config.downsample_ratio) ** 2,
            config.text_config.hidden_size,
        )
        self.act = ACT2FN[config.projector_hidden_act]
        self.linear_2 = nn.Linear(
            config.text_config.hidden_size, config.text_config.hidden_size
        )
```
**EN:** Method `InternS1MultiModalProjector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternS1MultiModalProjector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternS1MultiModalProjector.forward` (lines 82-87)
```python
    def forward(self, image_features):
        hidden_states = self.layer_norm(image_features)
        hidden_states = self.linear_1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** Method `InternS1MultiModalProjector.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternS1MultiModalProjector.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternS1ImagePixelInputs` (lines 90-102)
```python
class InternS1ImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bnp: Batch size * number of images * (1 + num_patches)
        - c: Number of channels (3)
        - h: Height
        - w: Width
        - bn: Batch size * number of images
    """

    type: Literal["pixel_values"] = "pixel_values"
    pixel_values: Annotated[torch.Tensor, TensorShape("bnp", 3, "h", "w")]
    num_patches: Annotated[torch.Tensor, TensorShape("bn")]
```
**EN:** Class `InternS1ImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `InternS1ImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `InternS1ImageEmbeddingInputs` (lines 105-114)
```python
class InternS1ImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - ni: Number of images
        - tifs: Total image feature size
        - hs: Hidden size (must match language model backbone)
    """

    type: Literal["image_embeds"] = "image_embeds"
    data: Annotated[torch.Tensor | list[torch.Tensor], TensorShape("ni", "tifs", "hs")]
```
**EN:** Class `InternS1ImageEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `InternS1ImageEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `InternS1VideoPixelInputs` (lines 120-132)
```python
class InternS1VideoPixelInputs(TensorSchema):
    """
    Dimensions:
        - bnv: Batch size * number of videos * number of frames
        - bn: Batch size * number of images
        - c: Number of channels (3)
        - h: Height
        - w: Width
    """

    type: Literal["pixel_values_videos"] = "pixel_values_videos"
    pixel_values: Annotated[torch.Tensor, TensorShape("bnv", 3, "h", "w")]
    num_patches: Annotated[torch.Tensor, TensorShape("bn")]
```
**EN:** Class `InternS1VideoPixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `InternS1VideoPixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `InternS1VideoEmbeddingInputs` (lines 135-144)
```python
class InternS1VideoEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - nv: Number of videos
        - tvfs: Total video feature size
        - hs: Hidden size (must match language model backbone)
    """

    type: Literal["video_embeds"] = "video_embeds"
    data: Annotated[torch.Tensor | list[torch.Tensor], TensorShape("nv", "tvfs", "hs")]
```
**EN:** Class `InternS1VideoEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `InternS1VideoEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `InternS1ProcessingInfo` (lines 179-282)
```python
class InternS1ProcessingInfo(BaseProcessingInfo):
    """ProcessingInfo for InternS1-style models."""

    def get_hf_processor(self, **kwargs: object) -> InternVLProcessor:
        hf_processor = self.ctx.get_hf_processor(InternVLProcessor, **kwargs)
        hf_processor.video_processor = cached_video_processor_from_config(
            self.ctx.model_config,
            processor_cls=InternVLVideoProcessor,
            size=hf_processor.image_processor.size,
            **kwargs,
        )
        return hf_processor

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None, "video": None}

    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
        processor: InternVLProcessor,
        mm_kwargs: Mapping[str, object],
    ) -> int:
        image_processor: GotOcr2ImageProcessorFast = processor.image_processor
```
**EN:** Class `InternS1ProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_processor, get_supported_mm_limits, get_num_image_tokens, resolve_target_ratios, get_image_size_with_most_features, get_max_image_tokens.
**CN:** 类 `InternS1ProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_processor, get_supported_mm_limits, get_num_image_tokens, resolve_target_ratios, get_image_size_with_most_features, get_max_image_tokens。

### Method `InternS1ProcessingInfo.get_hf_processor` (lines 182-190)
```python
    def get_hf_processor(self, **kwargs: object) -> InternVLProcessor:
        hf_processor = self.ctx.get_hf_processor(InternVLProcessor, **kwargs)
        hf_processor.video_processor = cached_video_processor_from_config(
            self.ctx.model_config,
            processor_cls=InternVLVideoProcessor,
            size=hf_processor.image_processor.size,
            **kwargs,
        )
        return hf_processor
```
**EN:** Method `InternS1ProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternS1ProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `InternS1ProcessingInfo.get_supported_mm_limits` (lines 192-193)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None, "video": None}
```
**EN:** Method `InternS1ProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternS1ProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `InternS1DummyInputsBuilder` (lines 285-329)
```python
class InternS1DummyInputsBuilder(BaseDummyInputsBuilder[InternS1ProcessingInfo]):
    """DummyInputsBuilder for InternS1-style models."""

    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)
        image_token = self.info.get_hf_processor().image_token
        video_token = self.info.get_hf_processor().video_token

        return image_token * num_images + video_token * num_videos

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        target_width, target_height = self.info.get_image_size_with_most_features()
        target_num_frames = self.info.get_num_frames_with_most_features(
            seq_len, mm_counts
        )
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)

        config = self.info.get_hf_config()
```
**EN:** Class `InternS1DummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[InternS1ProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `InternS1DummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[InternS1ProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `InternS1DummyInputsBuilder.get_dummy_text` (lines 288-294)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)
        image_token = self.info.get_hf_processor().image_token
        video_token = self.info.get_hf_processor().video_token

        return image_token * num_images + video_token * num_videos
```
**EN:** Method `InternS1DummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternS1DummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `InternS1DummyInputsBuilder.get_dummy_mm_data` (lines 296-329)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        target_width, target_height = self.info.get_image_size_with_most_features()
        target_num_frames = self.info.get_num_frames_with_most_features(
            seq_len, mm_counts
        )
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)

        config = self.info.get_hf_config()
        image_size_h, image_size_w = config.vision_config.image_size

        image_overrides = mm_options.get("image")
        video_overrides = mm_options.get("video")

        return {
            "image": self._get_dummy_images(
                width=target_width,
                height=target_height,
                num_images=num_images,
                overrides=image_overrides,
            ),
            "video": self._get_dummy_videos(
                width=image_size_w,
                height=image_size_h,
                num_frames=target_num_frames,
                num_videos=num_videos,
                overrides=video_overrides,
            ),
        }
```
**EN:** Method `InternS1DummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternS1DummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `InternS1MultiModalProcessor` (lines 332-501)
```python
class InternS1MultiModalProcessor(BaseMultiModalProcessor[InternS1ProcessingInfo]):
    """Basic image-only MultiModalProcessor for InternS1-style models."""

    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        mm_data = dict(mm_data)
        videos = mm_data.pop("videos", [])
        images = mm_data.pop("images", [])
        assert isinstance(videos, list)
        assert isinstance(images, list)

        hf_processor = self.info.get_hf_processor(**mm_kwargs)
        tokenizer = hf_processor.tokenizer
        video_token_id = tokenizer.encode(
            hf_processor.video_token, add_special_tokens=False
        )
        assert len(video_token_id) == 1
        video_token_id = video_token_id[0]

        prompt = re.sub(hf_processor.image_token, "<image_placeholder>", prompt)
```
**EN:** Class `InternS1MultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[InternS1ProcessingInfo]. Key methods include _call_hf_processor, _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `InternS1MultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[InternS1ProcessingInfo]。 关键方法包括 _call_hf_processor, _get_mm_fields_config, _get_prompt_updates。

### Class `InternS1ForConditionalGeneration` (lines 509-820)
```python
@MULTIMODAL_REGISTRY.register_processor(
    InternS1MultiModalProcessor,
    info=InternS1ProcessingInfo,
    dummy_inputs=InternS1DummyInputsBuilder,
)
class InternS1ForConditionalGeneration(
    nn.Module, SupportsMultiModal, SupportsPP, SupportsLoRA
):
    # To ensure correct weight loading and mapping.
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            "lm_head.": "language_model.lm_head.",
            "model.language_model.": "language_model.model.",
            "model.vision_tower.": "vision_tower.",
            "model.multi_modal_projector.": "multi_modal_projector.",
        }
    )

    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        # transformers InternVLProcessor uses <IMG_CONTEXT> as the separator
        # refer to https://github.com/huggingface/transformers/blob/f90de364c2484c7c325bbe05befdcf487bd75b63/src/transformers/models/internvl/processing_internvl.py#L116
        if modality.startswith("image"):
            return "<IMG_CONTEXT>"
        if modality.startswith("video"):
```
**EN:** Class `InternS1ForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP, SupportsLoRA. Key methods include get_placeholder_str, __init__, _init_vision_model, _init_mlp1, pixel_shuffle, extract_feature.
**CN:** 类 `InternS1ForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP、SupportsLoRA。 关键方法包括 get_placeholder_str, __init__, _init_vision_model, _init_mlp1, pixel_shuffle, extract_feature。

### Method `InternS1ForConditionalGeneration.get_placeholder_str` (lines 523-531)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        # transformers InternVLProcessor uses <IMG_CONTEXT> as the separator
        # refer to https://github.com/huggingface/transformers/blob/f90de364c2484c7c325bbe05befdcf487bd75b63/src/transformers/models/internvl/processing_internvl.py#L116
        if modality.startswith("image"):
            return "<IMG_CONTEXT>"
        if modality.startswith("video"):
            return "<video>"

        raise ValueError("Only image or video modality is supported")
```
**EN:** Method `InternS1ForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternS1ForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `InternS1ForConditionalGeneration.__init__` (lines 533-571)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config

        self.config = config
        self.multimodal_config = multimodal_config

        image_size = config.vision_config.image_size[0]
        patch_size = config.vision_config.patch_size[0]
        self.patch_size = patch_size
        self.num_image_token = int(
            (image_size // patch_size) ** 2 * (config.downsample_ratio**2)
        )
        self.downsample_ratio = config.downsample_ratio

        with self._mark_tower_model(vllm_config, {"image", "video"}):
            self.vision_tower = self._init_vision_model(
                config,
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "vision_tower"),
            )
            self.multi_modal_projector = self._init_mlp1(config)

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=config.text_config,
                prefix=maybe_prefix(prefix, "language_model"),
            )

        self.img_context_token_id = None
        self.video_context_token_id = None

        self.visual_token_mask = None
        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )
```
**EN:** Method `InternS1ForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternS1ForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

## Key Concepts / 关键概念
- **LoRA support / LoRA 支持**
  - **EN:** The file is aware of low-rank adaptation interfaces or compatible parameter layouts.
  - **CN:** 该文件考虑了 LoRA 接口或兼容的低秩参数布局。
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
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Literal, TypeAlias`, `import regex as re`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers import BatchFeature, InternVLProcessor, PretrainedConfig`, `from transformers.activations import ACT2FN`, `from transformers.models.got_ocr2.image_processing_got_ocr2_fast import (`, `from transformers.models.internvl.video_processing_internvl import (`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.models.interns1_vit import InternS1VisionModel`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import (`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`, `from vllm.transformers_utils.processor import cached_video_processor_from_config`
