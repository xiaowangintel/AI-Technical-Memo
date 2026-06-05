# llava_onevision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/llava_onevision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the LLaVA Onevision multimodal model adapter used for inference in vLLM. / 实现 LLaVA Onevision 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-51)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import math
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Final, Literal, Protocol, TypeAlias

import torch
import torch.nn as nn
from transformers import BatchFeature, LlavaOnevisionConfig, LlavaOnevisionProcessor
from transformers.models.llava_onevision.modeling_llava_onevision import (
    get_anyres_image_grid_shape,
    unpad_image,
)

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
# ... omitted for brevity ...
from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .llava import LlavaDummyInputsBuilder, init_vision_tower_for_llava
from .llava_next import (
    BaseLlavaNextMultiModalProcessor,
    LlavaNextLikeConfig,
    LlavaNextProcessingInfo,
)
from .siglip import SiglipVisionModel
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)

# For profile run
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, transformers, transformers.models.llava_onevision.modeling_llava_onevision supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.activation connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, transformers, transformers.models.llava_onevision.modeling_llava_onevision 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.activation 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `_MAX_FRAMES_PER_VIDEO` (lines 52-52)
```python
_MAX_FRAMES_PER_VIDEO = 16
```
**EN:** This assignment block centers on `_MAX_FRAMES_PER_VIDEO` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `_MAX_FRAMES_PER_VIDEO` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Class `LlavaOnevisionVideoPixelInputs` (lines 55-74)
```python
class LlavaOnevisionVideoPixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of videos
        - f: Number of frames
        - c: Number of channels (3)
        - h: Height
        - w: Width

        Note that `f` may be different for each batch, and 'num_frames'
        may be different for each video, in which case the data is passed as a
        list instead of a batched tensor.
    """

    type: Literal["pixel_values_videos"] = "pixel_values_videos"

    pixel_values_videos: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("bn", "f", 3, "h", "w", dynamic_dims={"f"}),
    ]
```
**EN:** Defines `LlavaOnevisionVideoPixelInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of videos - f: Number of frames - c: Number of channels (3) - h: Height - w: Width Note that `f` may be different for each batch, and 'num_frames' may be different for each video, in."
**CN:** 定义 `LlavaOnevisionVideoPixelInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of videos - f: Number of frames - c: Number of channels (3) - h: Height - w: Width Note that `f` may be different for each batch, and 'num_frames' may be different for each video, in。”

### Class `LlavaOnevisionImagePixelInputs` (lines 77-97)
```python
class LlavaOnevisionImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - np: Number of patches (1 + num_patches)
        - c: Number of channels (3)
        - h: Height
        - w: Width

        Note that `num_patches` may be different per batch and image,
        in which case the data is passed as a list instead of a batched tensor.
    """

    type: Literal["pixel_values"] = "pixel_values"

    pixel_values: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("bn", "np", 3, "h", "w", dynamic_dims={"np"}),
    ]

    image_sizes: Annotated[torch.Tensor | None, TensorShape("bn", 2)]
```
**EN:** Defines `LlavaOnevisionImagePixelInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - np: Number of patches (1 + num_patches) - c: Number of channels (3) - h: Height - w: Width Note that `num_patches` may be different per batch and image, in which case the."
**CN:** 定义 `LlavaOnevisionImagePixelInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - np: Number of patches (1 + num_patches) - c: Number of channels (3) - h: Height - w: Width Note that `num_patches` may be different per batch and image, in which case the。”

### Class `LlavaOnevisionImageEmbeddingInputs` (lines 100-113)
```python
class LlavaOnevisionImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - ifs: Image feature size
        - hs: Hidden size (must match language model backbone)
    """

    type: Literal["image_embeds"] = "image_embeds"

    data: Annotated[
        torch.Tensor,
        TensorShape("bn", "ifs", "hs"),
    ]
```
**EN:** Defines `LlavaOnevisionImageEmbeddingInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - ifs: Image feature size - hs: Hidden size (must match language model backbone)."
**CN:** 定义 `LlavaOnevisionImageEmbeddingInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - ifs: Image feature size - hs: Hidden size (must match language model backbone)。”

### Class `LlavaOnevisionLikeConfig` (lines 125-126)
```python
class LlavaOnevisionLikeConfig(LlavaNextLikeConfig, Protocol):
    video_token_index: Final[int]
```
**EN:** Defines `LlavaOnevisionLikeConfig`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from LlavaNextLikeConfig, Protocol.
**CN:** 定义 `LlavaOnevisionLikeConfig`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 LlavaNextLikeConfig、Protocol。

### Class `LlavaOnevisionProcessingInfo` (lines 129-259)
```python
class LlavaOnevisionProcessingInfo(LlavaNextProcessingInfo):
    def get_hf_config(self) -> LlavaOnevisionLikeConfig:
        return self.ctx.get_hf_config(LlavaOnevisionConfig)

    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(LlavaOnevisionProcessor, **kwargs)

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None, "video": None}
# ... omitted for brevity ...
    def _get_num_unpadded_features(
        self,
        *,
        original_height: int,
        original_width: int,
        npatches: int,
        num_patch_height: int,
        num_patch_width: int,
    ) -> tuple[int, int]:
        current_height = npatches * num_patch_height
        current_width = npatches * num_patch_width

        aspect_ratio = original_width / original_height
        current_aspect_ratio = current_width / current_height

        if aspect_ratio > current_aspect_ratio:
            new_height = int(
                round(original_height * (current_width / original_width), 7)
            )
```
**EN:** Defines `LlavaOnevisionProcessingInfo`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from LlavaNextProcessingInfo. Key methods such as `get_hf_config`, `get_hf_processor`, `get_supported_mm_limits`, `_get_num_unpadded_features`, `get_image_size_with_most_features` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaOnevisionProcessingInfo`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 LlavaNextProcessingInfo。 `get_hf_config`, `get_hf_processor`, `get_supported_mm_limits`, `_get_num_unpadded_features`, `get_image_size_with_most_features` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaOnevisionDummyInputsBuilder` (lines 262-306)
```python
class LlavaOnevisionDummyInputsBuilder(
    LlavaDummyInputsBuilder[LlavaOnevisionProcessingInfo]
):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.image_token
        video_token = processor.video_token

        return image_token * num_images + video_token * num_videos

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)

        target_width, target_height = self.info.get_image_size_with_most_features()
        target_num_frames = self.info.get_num_frames_with_most_features(
            seq_len, mm_counts
        )

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
                width=target_width,
                height=target_height,
                num_frames=target_num_frames,
                num_videos=num_videos,
                overrides=video_overrides,
            ),
        }
```
**EN:** Defines `LlavaOnevisionDummyInputsBuilder`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from LlavaDummyInputsBuilder[LlavaOnevisionProcessingInfo]. Key methods such as `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaOnevisionDummyInputsBuilder`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 LlavaDummyInputsBuilder[LlavaOnevisionProcessingInfo]。 `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaOnevisionMultiModalProcessor` (lines 309-450)
```python
class LlavaOnevisionMultiModalProcessor(
    BaseLlavaNextMultiModalProcessor[LlavaOnevisionProcessingInfo]
):
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return dict(
            pixel_values=MultiModalFieldConfig.batched("image"),
            image_sizes=MultiModalFieldConfig.batched("image"),
            image_embeds=MultiModalFieldConfig.batched("image"),
            pixel_values_videos=MultiModalFieldConfig.batched("video"),
        )
# ... omitted for brevity ...
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        mm_data = dict(mm_data)
        videos = mm_data.pop("videos", [])
        assert isinstance(videos, list)

        if not videos:
            return super()._call_hf_processor(
                prompt=prompt,
                mm_data=mm_data,
                mm_kwargs=mm_kwargs,
                tok_kwargs=tok_kwargs,
            )

# ... omitted for brevity ...
    def _hf_processor_applies_updates(
        self,
        prompt_text: str,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        tokenization_kwargs: Mapping[str, object],
    ) -> bool:
        base_result = super()._hf_processor_applies_updates(
            prompt_text=prompt_text,
            mm_items=mm_items,
            hf_processor_mm_kwargs=hf_processor_mm_kwargs,
            tokenization_kwargs=tokenization_kwargs,
        )

        return base_result and mm_items.get_count("video", strict=False) == 0
# ... omitted for brevity ...
    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        image_repls = super()._get_prompt_updates(
            mm_items=mm_items,
            hf_processor_mm_kwargs=hf_processor_mm_kwargs,
            out_mm_kwargs=out_mm_kwargs,
        )

        hf_config = self.info.get_hf_config()
        video_token_id = hf_config.video_token_index

        def get_video_replacement(item_idx: int):
            videos = mm_items.get_items(
                "video", (VideoEmbeddingItems, VideoProcessorItems)
            )
```
**EN:** Defines `LlavaOnevisionMultiModalProcessor`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from BaseLlavaNextMultiModalProcessor[LlavaOnevisionProcessingInfo]. Key methods such as `_get_mm_fields_config`, `_call_hf_processor`, `_hf_processor_applies_updates`, `_get_prompt_updates` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaOnevisionMultiModalProcessor`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 BaseLlavaNextMultiModalProcessor[LlavaOnevisionProcessingInfo]。 `_get_mm_fields_config`, `_call_hf_processor`, `_hf_processor_applies_updates`, `_get_prompt_updates` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaOnevisionMultiModalProjector` (lines 453-473)
```python
class LlavaOnevisionMultiModalProjector(nn.Module):
    def __init__(self, config: LlavaOnevisionConfig):
        super().__init__()

        self.linear_1 = nn.Linear(
            config.vision_config.hidden_size,
            config.text_config.hidden_size,
            bias=config.multimodal_projector_bias,
        )
        self.act = get_act_fn(config.projector_hidden_act)
        self.linear_2 = nn.Linear(
            config.text_config.hidden_size,
            config.text_config.hidden_size,
            bias=config.multimodal_projector_bias,
        )

    def forward(self, image_features: torch.Tensor) -> torch.Tensor:
        hidden_states = self.linear_1(image_features)
        hidden_states = self.act(hidden_states)
        hidden_states = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** Defines `LlavaOnevisionMultiModalProjector`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaOnevisionMultiModalProjector`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaOnevisionForConditionalGeneration` (lines 476-918)
```python
@MULTIMODAL_REGISTRY.register_processor(
    LlavaOnevisionMultiModalProcessor,
    info=LlavaOnevisionProcessingInfo,
    dummy_inputs=LlavaOnevisionDummyInputsBuilder,
)
class LlavaOnevisionForConditionalGeneration(nn.Module, SupportsMultiModal, SupportsPP):
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            # mapping for new names in checkpoint saved after transformers v4.52
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config

        self.config = config
        self.multimodal_config = multimodal_config

        with self._mark_tower_model(vllm_config, {"image", "video"}):
            # Initialize the vision tower only up to the required feature layer
            self.vision_tower = init_vision_tower_for_llava(
                config,
                quant_config=quant_config,
                require_post_norm=False,
                prefix=maybe_prefix(prefix, "vision_tower"),
            )
            self.image_newline = nn.Parameter(
                torch.empty(config.text_config.hidden_size)
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        """Run forward pass for LlaVA-Onevision.
        Args:
            input_ids: Flattened (concatenated) input_ids corresponding to a
                batch.
            pixel_values_videos: Pixels in each frames for each input videos.
        """
        if intermediate_tensors is not None:
            inputs_embeds = None

        hidden_states = self.language_model.model(
            input_ids, positions, intermediate_tensors, inputs_embeds=inputs_embeds
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.language_model.compute_logits(hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights, mapper=self.hf_to_vllm_mapper)
```
**EN:** Defines `LlavaOnevisionForConditionalGeneration`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `_parse_and_validate_image_input`, `_parse_and_validate_video_input`, `_parse_and_validate_multimodal_inputs` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaOnevisionForConditionalGeneration`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `_parse_and_validate_image_input`, `_parse_and_validate_video_input`, `_parse_and_validate_multimodal_inputs` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Quantized weights**
  - **EN:** Quantization hooks allow the model to load compressed checkpoints or specialized linear layers.
  - **CN:** 量化钩子使模型能够加载压缩检查点或专用线性层。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。
- **Pooling and encoding**
  - **EN:** This file exposes encoder-style behavior, pooled outputs, or sequence-level representations beyond causal decoding.
  - **CN:** 该文件提供编码器式行为、池化输出，或超出因果解码的序列表征。

## Dependencies / 依赖关系
- **Standard library**: math, collections.abc, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, transformers, transformers.models.llava_onevision.modeling_llava_onevision
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.activation, vllm.multimodal, vllm.multimodal.inputs, vllm.multimodal.parse, vllm.multimodal.processing
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .clip, .interfaces, .llava, .llava_next, .siglip, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
