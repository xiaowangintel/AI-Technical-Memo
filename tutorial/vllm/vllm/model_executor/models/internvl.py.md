# internvl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/internvl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for internvl, including encoder/decoder glue and vLLM runtime adaptation. / 面向 internvl 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 10-62)
```python
from abc import abstractmethod
from collections.abc import Iterable, Mapping, Sequence
from functools import cached_property
from typing import Annotated, Literal, TypeAlias, TypeVar

import torch
import torch.nn as nn
from transformers import BatchFeature, PretrainedConfig

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.quantization.awq import AWQConfig
from vllm.model_executor.models.intern_vit import (
    InternVisionModel,
    InternVisionPatchModel,
)
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    BatchedTensorInputs,
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
)
from vllm.sequence import IntermediateTensors
from vllm.transformers_utils.processors.internvl import (
    InternVLImageProcessor,
    InternVLProcessor,
    InternVLVideoProcessor,
)
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from .utils import AutoWeightsLoader, init_vllm_registered_model, maybe_prefix
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 183-183)
```python
_I = TypeVar("_I", bound=BaseInternVLProcessingInfo)
```
**EN:** This block defines _I, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _I，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Class `InternVLImagePixelInputs` (lines 65-77)
```python
class InternVLImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - bnp: Batch size * number of images * (1 + num_patches)
        - c: Number of channels (3)
        - h: Height of each image patch
        - w: Width of each image patch
    """

    type: Literal["pixel_values"]
    pixel_values_flat: Annotated[torch.Tensor, TensorShape("bnp", 3, "h", "w")]
    num_patches: Annotated[torch.Tensor, TensorShape("bn")]
```
**EN:** Class `InternVLImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `InternVLImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `InternVLImageEmbeddingInputs` (lines 80-89)
```python
class InternVLImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - n: Number of images
        - f: Total image feature size
        - h: Hidden size (must match the hidden size of language model backbone)
    """

    type: Literal["image_embeds"]
    data: Annotated[torch.Tensor | list[torch.Tensor], TensorShape("n", "f", "h")]
```
**EN:** Class `InternVLImageEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `InternVLImageEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `InternVLVideoPixelInputs` (lines 95-107)
```python
class InternVLVideoPixelInputs(TensorSchema):
    """
    Dimensions:
        - bvf: Batch size * number of videos * num_frames
        - bn: Batch size * number of images
        - c: Number of channels (3)
        - h: Height of each video frame
        - w: Width of each video frame
    """

    type: Literal["pixel_values_videos"]
    pixel_values_flat: Annotated[torch.Tensor, TensorShape("bvf", 3, "h", "w")]
    num_patches: Annotated[torch.Tensor, TensorShape("bn")]
```
**EN:** Class `InternVLVideoPixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `InternVLVideoPixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `InternVLVideoEmbeddingInputs` (lines 110-119)
```python
class InternVLVideoEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - n: Number of videos
        - f: Total video feature size
        - h: Hidden size (must match the hidden size of language model backbone)
    """

    type: Literal["video_embeds"]
    data: Annotated[torch.Tensor | list[torch.Tensor], TensorShape("n", "f", "h")]
```
**EN:** Class `InternVLVideoEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `InternVLVideoEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `BaseInternVLProcessingInfo` (lines 125-180)
```python
class BaseInternVLProcessingInfo(BaseProcessingInfo):
    """Basic image-only ProcessingInfo for InternVL-style models."""

    @abstractmethod
    def get_hf_processor(self, **kwargs: object) -> InternVLProcessor:
        raise NotImplementedError

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
        processor: InternVLProcessor,
    ) -> int:
        return processor.get_num_image_tokens(
            image_width=image_width,
            image_height=image_height,
        )

    def get_image_size_with_most_features(self) -> ImageSize:
        processor = self.get_hf_processor()
        image_processor = processor.image_processor
```
**EN:** Class `BaseInternVLProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_processor, get_supported_mm_limits, get_num_image_tokens, get_image_size_with_most_features, get_max_image_tokens.
**CN:** 类 `BaseInternVLProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_processor, get_supported_mm_limits, get_num_image_tokens, get_image_size_with_most_features, get_max_image_tokens。

### Method `BaseInternVLProcessingInfo.get_hf_processor` (lines 129-130)
```python
    @abstractmethod
    def get_hf_processor(self, **kwargs: object) -> InternVLProcessor:
        raise NotImplementedError
```
**EN:** Method `BaseInternVLProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BaseInternVLProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `BaseInternVLProcessingInfo.get_supported_mm_limits` (lines 132-133)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Method `BaseInternVLProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BaseInternVLProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `BaseInternVLDummyInputsBuilder` (lines 186-212)
```python
class BaseInternVLDummyInputsBuilder(BaseDummyInputsBuilder[_I]):
    """Basic image-only DummyInputsBuilder for InternVL-style models."""

    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        return "<image>" * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        target_width, target_height = self.info.get_image_size_with_most_features()
        num_images = mm_counts.get("image", 0)

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=target_width,
                height=target_height,
                num_images=num_images,
                overrides=image_overrides,
```
**EN:** Class `BaseInternVLDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[_I]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `BaseInternVLDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[_I]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `BaseInternVLDummyInputsBuilder.get_dummy_text` (lines 189-192)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        return "<image>" * num_images
```
**EN:** Method `BaseInternVLDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BaseInternVLDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `BaseInternVLDummyInputsBuilder.get_dummy_mm_data` (lines 194-212)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        target_width, target_height = self.info.get_image_size_with_most_features()
        num_images = mm_counts.get("image", 0)

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
**EN:** Method `BaseInternVLDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BaseInternVLDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `BaseInternVLMultiModalProcessor` (lines 215-317)
```python
class BaseInternVLMultiModalProcessor(BaseMultiModalProcessor[_I]):
    """Basic image-only MultiModalProcessor for InternVL-style models."""

    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        processed_outputs = super()._call_hf_processor(
            prompt=prompt,
            mm_data=mm_data,
            mm_kwargs=mm_kwargs,
            tok_kwargs=tok_kwargs,
        )

        hf_processor = self.info.get_hf_processor(**mm_kwargs)
        image_token_id = hf_processor.ctx_image_token_id

        # Since there may be extra tokens in the feature placeholders,
        # we need to pass the image token ID to the model to select the
        # tokens to merge from the vision encoder outputs
        processed_outputs["image_token_id"] = torch.tensor(image_token_id)
```
**EN:** Class `BaseInternVLMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[_I]. Key methods include _call_hf_processor, _get_image_fields_config, _get_mm_fields_config, _get_prompt_repl_image, _get_prompt_updates.
**CN:** 类 `BaseInternVLMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[_I]。 关键方法包括 _call_hf_processor, _get_image_fields_config, _get_mm_fields_config, _get_prompt_repl_image, _get_prompt_updates。

### Class `InternVLProcessingInfo` (lines 320-406)
```python
class InternVLProcessingInfo(BaseInternVLProcessingInfo):
    """InternVL ProcessingInfo extended for video processing"""

    def get_image_processor(self, **kwargs):
        config = self.get_hf_config()
        vision_config = config.vision_config

        kwargs = self.ctx.get_merged_mm_kwargs(kwargs)
        kwargs.setdefault("image_size", vision_config.image_size)
        kwargs.setdefault("min_dynamic_patch", config.min_dynamic_patch)
        kwargs.setdefault("max_dynamic_patch", config.max_dynamic_patch)
        kwargs.setdefault("dynamic_image_size", config.dynamic_image_size)
        kwargs.setdefault("use_thumbnail", config.use_thumbnail)

        return InternVLImageProcessor(**kwargs)

    def get_video_processor(self, **kwargs):
        config = self.get_hf_config()
        vision_config = config.vision_config

        kwargs = self.ctx.get_merged_mm_kwargs(kwargs)
        kwargs.setdefault("image_size", vision_config.image_size)

        return InternVLVideoProcessor(**kwargs)
```
**EN:** Class `InternVLProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseInternVLProcessingInfo. Key methods include get_image_processor, get_video_processor, ctx_video_token, get_hf_processor, get_supported_mm_limits, get_num_frames_with_most_features.
**CN:** 类 `InternVLProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseInternVLProcessingInfo。 关键方法包括 get_image_processor, get_video_processor, ctx_video_token, get_hf_processor, get_supported_mm_limits, get_num_frames_with_most_features。

### Method `InternVLProcessingInfo.get_image_processor` (lines 323-334)
```python
    def get_image_processor(self, **kwargs):
        config = self.get_hf_config()
        vision_config = config.vision_config

        kwargs = self.ctx.get_merged_mm_kwargs(kwargs)
        kwargs.setdefault("image_size", vision_config.image_size)
        kwargs.setdefault("min_dynamic_patch", config.min_dynamic_patch)
        kwargs.setdefault("max_dynamic_patch", config.max_dynamic_patch)
        kwargs.setdefault("dynamic_image_size", config.dynamic_image_size)
        kwargs.setdefault("use_thumbnail", config.use_thumbnail)

        return InternVLImageProcessor(**kwargs)
```
**EN:** Method `InternVLProcessingInfo.get_image_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternVLProcessingInfo.get_image_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `InternVLProcessingInfo.get_video_processor` (lines 336-343)
```python
    def get_video_processor(self, **kwargs):
        config = self.get_hf_config()
        vision_config = config.vision_config

        kwargs = self.ctx.get_merged_mm_kwargs(kwargs)
        kwargs.setdefault("image_size", vision_config.image_size)

        return InternVLVideoProcessor(**kwargs)
```
**EN:** Method `InternVLProcessingInfo.get_video_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternVLProcessingInfo.get_video_processor` 封装了该模块中的一段可复用核心逻辑。

### Class `InternVLDummyInputsBuilder` (lines 409-445)
```python
class InternVLDummyInputsBuilder(
    BaseInternVLDummyInputsBuilder[InternVLProcessingInfo]
):
    """InternVL DummyInputsBuilder extended for video support"""

    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_videos = mm_counts.get("video", 0)

        return super().get_dummy_text(mm_counts) + "<video>" * num_videos

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        dummy_image = super().get_dummy_mm_data(seq_len, mm_counts, mm_options)
        if self.info.ctx_video_token:
            config = self.info.get_hf_config()
            image_size: int = config.vision_config.image_size
            target_num_frames = self.info.get_num_frames_with_most_features(
                seq_len, mm_counts
            )
            num_videos = mm_counts.get("video", 0)
            video_overrides = mm_options.get("video")
```
**EN:** Class `InternVLDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseInternVLDummyInputsBuilder[InternVLProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `InternVLDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseInternVLDummyInputsBuilder[InternVLProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `InternVLDummyInputsBuilder.get_dummy_text` (lines 414-417)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_videos = mm_counts.get("video", 0)

        return super().get_dummy_text(mm_counts) + "<video>" * num_videos
```
**EN:** Method `InternVLDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternVLDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `InternVLDummyInputsBuilder.get_dummy_mm_data` (lines 419-445)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        dummy_image = super().get_dummy_mm_data(seq_len, mm_counts, mm_options)
        if self.info.ctx_video_token:
            config = self.info.get_hf_config()
            image_size: int = config.vision_config.image_size
            target_num_frames = self.info.get_num_frames_with_most_features(
                seq_len, mm_counts
            )
            num_videos = mm_counts.get("video", 0)
            video_overrides = mm_options.get("video")
            dummy_video = {
                "video": self._get_dummy_videos(
                    width=image_size,
                    height=image_size,
                    num_frames=target_num_frames,
                    num_videos=num_videos,
                    overrides=video_overrides,
                )
            }
        else:
            dummy_video = {}
        return {**dummy_image, **dummy_video}
```
**EN:** Method `InternVLDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternVLDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `InternVLMultiModalProcessor` (lines 448-536)
```python
class InternVLMultiModalProcessor(
    BaseInternVLMultiModalProcessor[InternVLProcessingInfo]
):
    """InternVL MultiModalProcessor extended for video support"""

    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        processed_outputs = super()._call_hf_processor(
            prompt, mm_data, mm_kwargs, tok_kwargs
        )

        hf_processor = self.info.get_hf_processor(**mm_kwargs)
        if (video_token_id := hf_processor.ctx_video_token_id) is not None:
            processed_outputs["video_token_id"] = torch.tensor(video_token_id)

        return processed_outputs

    def _get_video_fields_config(self, hf_inputs: BatchFeature):
        video_num_patches = hf_inputs.get("video_num_patches", torch.empty(0))
        num_videos = len(video_num_patches)
```
**EN:** Class `InternVLMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseInternVLMultiModalProcessor[InternVLProcessingInfo]. Key methods include _call_hf_processor, _get_video_fields_config, _get_mm_fields_config, _get_prompt_repl_video, _get_prompt_updates.
**CN:** 类 `InternVLMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseInternVLMultiModalProcessor[InternVLProcessingInfo]。 关键方法包括 _call_hf_processor, _get_video_fields_config, _get_mm_fields_config, _get_prompt_repl_video, _get_prompt_updates。

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
- **Standard library / 标准库**: `from abc import abstractmethod`, `from collections.abc import Iterable, Mapping, Sequence`, `from functools import cached_property`, `from typing import Annotated, Literal, TypeAlias, TypeVar`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers import BatchFeature, PretrainedConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.quantization.awq import AWQConfig`, `from vllm.model_executor.models.intern_vit import (`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import (`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`
