# hyperclovax_vision_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/hyperclovax_vision_v2.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for hyperclovax_vision_v2, including encoder/decoder glue and vLLM runtime adaptation. / 面向 hyperclovax_vision_v2 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 13-49)
```python
from collections.abc import Iterable, Mapping, Sequence
from functools import partial
from typing import Annotated, Literal

import torch
import torch.nn as nn
from transformers import BatchFeature

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.forward_context import set_forward_context
from vllm.inputs import MultiModalDataDict
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import ImageSize, MultiModalDataItems
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    ProcessorInputs,
    PromptReplacement,
    PromptUpdate,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .qwen2_5_vl import Qwen2_5_VisionTransformer
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 55-55)
```python
V2_IMAGE_TOKEN: str = "<|image_start|><|IMAGE_PAD|><|image_end|>"
```
**EN:** This block defines V2_IMAGE_TOKEN, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 V2_IMAGE_TOKEN，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 56-56)
```python
V2_VIDEO_TOKEN: str = "<|video_start|><|VIDEO_PAD|><|video_end|>"
```
**EN:** This block defines V2_VIDEO_TOKEN, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 V2_VIDEO_TOKEN，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Class `HCXVisionV2ImagePixelInputs` (lines 59-71)
```python
class HCXVisionV2ImagePixelInputs(TensorSchema):
    """
    V2 Image inputs using Qwen2.5-VL style grid_thw format.

    Dimensions:
        - np: Number of patches
        - ni: Number of images
        - cps: Number of channels * patch_size * patch_size
    """

    type: Literal["pixel_values"] = "pixel_values"
    pixel_values: Annotated[torch.Tensor, TensorShape("np", "cps")]
    image_grid_thw: Annotated[torch.Tensor, TensorShape("ni", 3)]
```
**EN:** Class `HCXVisionV2ImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `HCXVisionV2ImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `HCXVisionV2ImageEmbeddingInputs` (lines 74-86)
```python
class HCXVisionV2ImageEmbeddingInputs(TensorSchema):
    """
    V2 Image embedding inputs.

    Dimensions:
        - nf: Number of image features
        - hs: Hidden size
        - ni: Number of images
    """

    type: Literal["image_embeds"] = "image_embeds"
    image_embeds: Annotated[torch.Tensor, TensorShape("nf", "hs")]
    image_grid_thw: Annotated[torch.Tensor, TensorShape("ni", 3)]
```
**EN:** Class `HCXVisionV2ImageEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `HCXVisionV2ImageEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `HCXVisionV2VideoPixelInputs` (lines 92-104)
```python
class HCXVisionV2VideoPixelInputs(TensorSchema):
    """
    V2 Video inputs using Qwen2.5-VL style grid_thw format.

    Dimensions:
        - np: Number of patches
        - nv: Number of videos
        - ctps: Number of channels * temporal_patch_size * patch_size * patch_size
    """

    type: Literal["pixel_values_videos"] = "pixel_values_videos"
    pixel_values_videos: Annotated[torch.Tensor, TensorShape("np", "ctps")]
    video_grid_thw: Annotated[torch.Tensor, TensorShape("nv", 3)]
```
**EN:** Class `HCXVisionV2VideoPixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `HCXVisionV2VideoPixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `HCXVisionV2VideoEmbeddingInputs` (lines 107-119)
```python
class HCXVisionV2VideoEmbeddingInputs(TensorSchema):
    """
    V2 Video embedding inputs.

    Dimensions:
        - nf: Number of video features
        - hs: Hidden size
        - nv: Number of videos
    """

    type: Literal["video_embeds"] = "video_embeds"
    video_embeds: Annotated[torch.Tensor, TensorShape("nf", "hs")]
    video_grid_thw: Annotated[torch.Tensor, TensorShape("nv", 3)]
```
**EN:** Class `HCXVisionV2VideoEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `HCXVisionV2VideoEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `HCXVisionV2ProcessingInfo` (lines 125-178)
```python
class HCXVisionV2ProcessingInfo(BaseProcessingInfo):
    """Processing info for HyperCLOVAX V2 (32B Think model)."""

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None, "video": None}

    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        hf_config = self.get_hf_config()
        vision_config = hf_config.vision_config
        patch_size = vision_config.patch_size
        spatial_merge_size = vision_config.spatial_merge_size

        grid_h = image_height // patch_size
        grid_w = image_width // patch_size

        return (grid_h * grid_w) // (spatial_merge_size**2)

    def get_num_video_tokens(
        self,
        *,
```
**EN:** Class `HCXVisionV2ProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_supported_mm_limits, get_num_image_tokens, get_num_video_tokens, get_image_size_with_most_features, get_max_image_tokens.
**CN:** 类 `HCXVisionV2ProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_supported_mm_limits, get_num_image_tokens, get_num_video_tokens, get_image_size_with_most_features, get_max_image_tokens。

### Method `HCXVisionV2ProcessingInfo.get_supported_mm_limits` (lines 128-129)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None, "video": None}
```
**EN:** Method `HCXVisionV2ProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionV2ProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Method `HCXVisionV2ProcessingInfo.get_num_image_tokens` (lines 131-145)
```python
    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        hf_config = self.get_hf_config()
        vision_config = hf_config.vision_config
        patch_size = vision_config.patch_size
        spatial_merge_size = vision_config.spatial_merge_size

        grid_h = image_height // patch_size
        grid_w = image_width // patch_size

        return (grid_h * grid_w) // (spatial_merge_size**2)
```
**EN:** Method `HCXVisionV2ProcessingInfo.get_num_image_tokens` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionV2ProcessingInfo.get_num_image_tokens` 封装了该模块中的一段可复用核心逻辑。

### Method `HCXVisionV2ProcessingInfo.get_num_video_tokens` (lines 147-164)
```python
    def get_num_video_tokens(
        self,
        *,
        video_width: int,
        video_height: int,
        num_frames: int,
    ) -> int:
        hf_config = self.get_hf_config()
        vision_config = hf_config.vision_config
        patch_size = vision_config.patch_size
        temporal_patch_size = vision_config.temporal_patch_size
        spatial_merge_size = vision_config.spatial_merge_size

        grid_t = num_frames // temporal_patch_size
        grid_h = video_height // patch_size
        grid_w = video_width // patch_size

        return (grid_t * grid_h * grid_w) // (spatial_merge_size**2)
```
**EN:** Method `HCXVisionV2ProcessingInfo.get_num_video_tokens` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionV2ProcessingInfo.get_num_video_tokens` 封装了该模块中的一段可复用核心逻辑。

### Method `HCXVisionV2ProcessingInfo.get_image_size_with_most_features` (lines 166-171)
```python
    def get_image_size_with_most_features(self) -> ImageSize:
        hf_config = self.get_hf_config()
        vision_config = hf_config.vision_config
        # Use a reasonable default size
        size = getattr(vision_config, "image_size", 448)
        return ImageSize(width=size, height=size)
```
**EN:** Method `HCXVisionV2ProcessingInfo.get_image_size_with_most_features` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionV2ProcessingInfo.get_image_size_with_most_features` 封装了该模块中的一段可复用核心逻辑。

### Class `HCXVisionV2DummyInputsBuilder` (lines 181-251)
```python
class HCXVisionV2DummyInputsBuilder(BaseDummyInputsBuilder[HCXVisionV2ProcessingInfo]):
    """Dummy inputs builder for HyperCLOVAX V2 memory profiling."""

    def get_dummy_text(
        self,
        mm_counts: Mapping[str, int],
    ) -> str:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)
        return V2_IMAGE_TOKEN * num_images + V2_VIDEO_TOKEN * num_videos

    def get_dummy_processor_inputs(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions] | None = None,
        mm_processor_kwargs: Mapping[str, object] | None = None,
    ) -> ProcessorInputs:
        """Build dummy processor inputs for memory profiling."""
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)
        prompt_text = V2_IMAGE_TOKEN * num_images + V2_VIDEO_TOKEN * num_videos

        dummy_mm_data = self.get_dummy_mm_data(
            seq_len,
```
**EN:** Class `HCXVisionV2DummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[HCXVisionV2ProcessingInfo]. Key methods include get_dummy_text, get_dummy_processor_inputs, get_dummy_mm_data.
**CN:** 类 `HCXVisionV2DummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[HCXVisionV2ProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_processor_inputs, get_dummy_mm_data。

### Method `HCXVisionV2DummyInputsBuilder.get_dummy_text` (lines 184-190)
```python
    def get_dummy_text(
        self,
        mm_counts: Mapping[str, int],
    ) -> str:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)
        return V2_IMAGE_TOKEN * num_images + V2_VIDEO_TOKEN * num_videos
```
**EN:** Method `HCXVisionV2DummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionV2DummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `HCXVisionV2DummyInputsBuilder.get_dummy_processor_inputs` (lines 192-217)
```python
    def get_dummy_processor_inputs(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions] | None = None,
        mm_processor_kwargs: Mapping[str, object] | None = None,
    ) -> ProcessorInputs:
        """Build dummy processor inputs for memory profiling."""
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)
        prompt_text = V2_IMAGE_TOKEN * num_images + V2_VIDEO_TOKEN * num_videos

        dummy_mm_data = self.get_dummy_mm_data(
            seq_len,
            mm_counts,
            mm_options,
            mm_processor_kwargs=mm_processor_kwargs,
        )
        dummy_mm_items = self.info.parse_mm_data(dummy_mm_data, validate=False)

        return ProcessorInputs(
            prompt=prompt_text,
            mm_data_items=dummy_mm_items,
            hf_processor_mm_kwargs=mm_processor_kwargs or {},
            tokenization_kwargs={"truncation": False},
        )
```
**EN:** Method `HCXVisionV2DummyInputsBuilder.get_dummy_processor_inputs` encapsulates a focused piece of reusable logic inside this module. The docstring says: Build dummy processor inputs for memory profiling.
**CN:** Method `HCXVisionV2DummyInputsBuilder.get_dummy_processor_inputs` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Build dummy processor inputs for memory profiling。

### Method `HCXVisionV2DummyInputsBuilder.get_dummy_mm_data` (lines 219-251)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions] | None = None,
        mm_processor_kwargs: Mapping[str, object] | None = None,
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)

        target_width, target_height = self.info.get_image_size_with_most_features()
        target_num_frames = 16  # Default for video

        image_overrides = mm_options.get("image") if mm_options else None
        video_overrides = mm_options.get("video") if mm_options else None

        result: MultiModalDataDict = {
            "image": self._get_dummy_images(
                width=target_width,
                height=target_height,
                num_images=num_images,
                overrides=image_overrides,  # type: ignore
            ),
            "video": self._get_dummy_videos(
                width=target_width,
                height=target_height,
                num_frames=target_num_frames,
                num_videos=num_videos,
                overrides=video_overrides,  # type: ignore
            ),
        }

        return result
```
**EN:** Method `HCXVisionV2DummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionV2DummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `HCXVisionV2MultiModalProcessor` (lines 254-410)
```python
class HCXVisionV2MultiModalProcessor(
    BaseMultiModalProcessor[HCXVisionV2ProcessingInfo]
):
    """Multimodal processor for HyperCLOVAX V2 (32B Think model)."""

    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        images = mm_data.get("images")
        videos = mm_data.get("videos")

        # Get the HF processor
        hf_processor = self.info.get_hf_processor(**mm_kwargs)

        # Build data dict for HF processor (images/videos only)
        # NOTE: We pass the prompt as-is without token normalization.
        # Token expansion is handled by vLLM via _get_prompt_updates since
        # _hf_processor_applies_updates returns False.
        data: dict[str, object] = dict(
            text=prompt,
            images=images,
```
**EN:** Class `HCXVisionV2MultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[HCXVisionV2ProcessingInfo]. Key methods include _call_hf_processor, _hf_processor_applies_updates, _get_prompt_updates, _get_mm_fields_config.
**CN:** 类 `HCXVisionV2MultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[HCXVisionV2ProcessingInfo]。 关键方法包括 _call_hf_processor, _hf_processor_applies_updates, _get_prompt_updates, _get_mm_fields_config。

### Class `HCXVisionV2ForCausalLM` (lines 418-681)
```python
@MULTIMODAL_REGISTRY.register_processor(
    HCXVisionV2MultiModalProcessor,
    info=HCXVisionV2ProcessingInfo,
    dummy_inputs=HCXVisionV2DummyInputsBuilder,
)
class HCXVisionV2ForCausalLM(nn.Module, SupportsMultiModal, SupportsPP):
    """
    HyperCLOVAX-SEED Vision-Language Model (V2 architecture).

    Supports:
    - HyperCLOVAX-SEED-Think-32B: Vision + Text

    Uses Qwen2.5 Vision Transformer as the vision encoder.
    """

    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
        "qkv": ["qkv"],  # For vision tower
    }

    # Weight mapping for loading HuggingFace checkpoints
    # NOTE: Order matters! Ignores (None) should come before renames to prevent
    # partial matches
    hf_to_vllm_mapper = WeightsMapper(
```
**EN:** Class `HCXVisionV2ForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods include __init__, get_placeholder_str, _parse_and_validate_image_input, _parse_and_validate_video_input, _process_image_input, _process_video_input.
**CN:** 类 `HCXVisionV2ForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 关键方法包括 __init__, get_placeholder_str, _parse_and_validate_image_input, _parse_and_validate_video_input, _process_image_input, _process_video_input。

### Method `HCXVisionV2ForCausalLM.__init__` (lines 448-504)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
    ) -> None:
        super().__init__()

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        # Text config
        text_config = config.text_config
        if text_config.model_type in ["gpt2", "hyperclovax", "llama"]:
            text_config._attn_implementation = "sdpa"
        if text_config.model_type != "hyperclovax":
            text_config.logits_scaling = 1.0

        # Vision config
        vision_config = config.vision_config

        self.config = config
        self.vision_config = vision_config
        self.text_config = text_config
        self.vllm_config = vllm_config

        # Linear projector (vision_hidden_size -> text_hidden_size)
        # For V2 model: mm_projector_type is "linear"
        vision_hidden_size = vision_config.hidden_size
        text_hidden_size = text_config.hidden_size

        # Check if out_hidden_size is defined (Qwen2.5-VL style)
        # The merger in Qwen2.5 VisionTransformer handles projection to out_hidden_size
        if hasattr(vision_config, "out_hidden_size"):
            out_hidden = vision_config.out_hidden_size
        else:
            out_hidden = vision_hidden_size

        with self._mark_tower_model(vllm_config, {"image", "video"}):
            self.visual = Qwen2_5_VisionTransformer(
                vision_config=vision_config,
                norm_eps=getattr(config, "rms_norm_eps", 1e-6),
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "visual"),
            )
            self.mm_projector = nn.Linear(out_hidden, text_hidden_size)

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=text_config,
                prefix=maybe_prefix(prefix, "language_model"),
            )

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )
```
**EN:** Method `HCXVisionV2ForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `HCXVisionV2ForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `HCXVisionV2ForCausalLM.get_placeholder_str` (lines 507-513)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return V2_IMAGE_TOKEN
        if modality.startswith("video"):
            return V2_VIDEO_TOKEN

        raise ValueError("Only image or video modality is supported")
```
**EN:** Method `HCXVisionV2ForCausalLM.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionV2ForCausalLM.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `HCXVisionV2ForCausalLM.embed_multimodal` (lines 630-652)
```python
    def embed_multimodal(
        self,
        **kwargs: object,
    ) -> MultiModalEmbeddings:
        modalities = self._parse_and_validate_multimodal_inputs(**kwargs)
        if not modalities:
            return []

        multimodal_embeddings: tuple[torch.Tensor, ...] = ()

        for modality in modalities:
            if modality == "image":
                image_input = modalities["image"]
                if image_input is not None:
                    image_embeddings = self._process_image_input(image_input)
                    multimodal_embeddings += tuple(image_embeddings)
            if modality == "video":
                video_input = modalities["video"]
                if video_input is not None:
                    video_embeddings = self._process_video_input(video_input)
                    multimodal_embeddings += tuple(video_embeddings)

        return multimodal_embeddings
```
**EN:** Method `HCXVisionV2ForCausalLM.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `HCXVisionV2ForCausalLM.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `HCXVisionV2ForCausalLM.forward` (lines 654-668)
```python
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
            input_ids, positions, intermediate_tensors, inputs_embeds=inputs_embeds
        )
        return hidden_states
```
**EN:** Method `HCXVisionV2ForCausalLM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `HCXVisionV2ForCausalLM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping, Sequence`, `from functools import partial`, `from typing import Annotated, Literal`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers import BatchFeature`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.forward_context import set_forward_context`, `from vllm.inputs import MultiModalDataDict`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import ImageSize, MultiModalDataItems`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`, `from vllm.utils.tensor_schema import TensorSchema, TensorShape`, `from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP`, `from .qwen2_5_vl import Qwen2_5_VisionTransformer`
- **Module note / 模块说明**: **EN:** HyperCLOVAX V2 (32B Think Model) Implementation.  This module contains the V2 architecture that uses Qwen2.5 Vision Transformer instead of CLIP/SigLIP used in V1.  Supports: - HyperCLOVAX-SEED-Think-32B: Vision + Text **CN:** 模块文档字符串给出的原始说明是：HyperCLOVAX V2 (32B Think Model) Implementation.  This module contains the V2 architecture that uses Qwen2.5 Vision Transformer instead of CLIP/SigLIP used in V1.  Supports: - HyperCLOVAX-SEED-Think-32B: Vision + Text。
