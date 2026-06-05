# llava_next_video.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/llava_next_video.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the LLaVA Next Video multimodal model adapter used for inference in vLLM. / 实现 LLaVA Next Video 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-50)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import math
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Literal

import torch
import torch.nn as nn
from transformers import BatchFeature, LlavaNextVideoConfig, LlavaNextVideoProcessor

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.activation import get_act_fn
from vllm.model_executor.models.clip import CLIPVisionModel
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
# ... omitted for brevity ...
from vllm.sequence import IntermediateTensors
from vllm.utils.collection_utils import is_list_of
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .llava import init_vision_tower_for_llava
from .siglip import SiglipVisionModel
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
from .vision import get_vision_encoder_info
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, transformers supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.activation connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, transformers 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.activation 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Class `LlavaNextVideoPixelInputs` (lines 51-71)
```python
class LlavaNextVideoPixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of videos
        - f: Number of frames
        - c: Number of channels (3)
        - h: Height of each frame
        - w: Width of each frame

    Note that `f` may be different for each batch, in which case
    the data is passed as a list instead of a batched tensor.

    Note that it only supports one video input for one batch.
    """

    type: Literal["pixel_values_videos"] = "pixel_values_videos"

    pixel_values_videos: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("bn", "f", 3, "h", "w", dynamic_dims={"f"}),
    ]
```
**EN:** Defines `LlavaNextVideoPixelInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of videos - f: Number of frames - c: Number of channels (3) - h: Height of each frame - w: Width of each frame Note that `f` may be different for each batch, in which case the data is passed as a list instead of a batched tensor."
**CN:** 定义 `LlavaNextVideoPixelInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of videos - f: Number of frames - c: Number of channels (3) - h: Height of each frame - w: Width of each frame Note that `f` may be different for each batch, in which case the data is passed as a list instead of a batched tensor。”

### Class `LlavaNextVideoProcessingInfo` (lines 74-150)
```python
class LlavaNextVideoProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(LlavaNextVideoConfig)

    def get_vision_encoder_info(self):
        return get_vision_encoder_info(self.get_hf_config())

    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(LlavaNextVideoProcessor, **kwargs)
# ... omitted for brevity ...
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"video": 1}
```
**EN:** Defines `LlavaNextVideoProcessingInfo`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from BaseProcessingInfo. Key methods such as `get_hf_config`, `get_vision_encoder_info`, `get_hf_processor`, `get_supported_mm_limits`, `get_image_size_with_most_features` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaNextVideoProcessingInfo`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 BaseProcessingInfo。 `get_hf_config`, `get_vision_encoder_info`, `get_hf_processor`, `get_supported_mm_limits`, `get_image_size_with_most_features` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaNextVideoDummyInputsBuilder` (lines 153-187)
```python
class LlavaNextVideoDummyInputsBuilder(
    BaseDummyInputsBuilder[LlavaNextVideoProcessingInfo]
):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_videos = mm_counts.get("video", 0)

        processor = self.info.get_hf_processor()
        video_token = processor.video_token

        return video_token * num_videos

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_videos = mm_counts.get("video", 0)

        target_width, target_height = self.info.get_image_size_with_most_features()
        target_num_frames = self.info.get_num_frames_with_most_features(
            seq_len, mm_counts
        )

        video_overrides = mm_options.get("video")

        return {
            "video": self._get_dummy_videos(
                width=target_width,
                height=target_height,
                num_frames=target_num_frames,
                num_videos=num_videos,
                overrides=video_overrides,
            )
        }
```
**EN:** Defines `LlavaNextVideoDummyInputsBuilder`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from BaseDummyInputsBuilder[LlavaNextVideoProcessingInfo]. Key methods such as `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaNextVideoDummyInputsBuilder`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 BaseDummyInputsBuilder[LlavaNextVideoProcessingInfo]。 `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaNextVideoMultiModalProcessor` (lines 190-232)
```python
class LlavaNextVideoMultiModalProcessor(
    BaseMultiModalProcessor[LlavaNextVideoProcessingInfo]
):
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return dict(pixel_values_videos=MultiModalFieldConfig.batched("video"))

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        hf_config = self.info.get_hf_config()
        video_token_id = hf_config.video_token_index

        def get_replacement(item_idx: int):
            videos = mm_items.get_items(
                "video", (VideoEmbeddingItems, VideoProcessorItems)
            )

            if isinstance(videos, VideoEmbeddingItems):
                num_video_tokens = videos.get_feature_size(item_idx)
            else:
                image_size = videos.get_frame_size(item_idx)
                num_video_tokens = self.info.get_num_video_tokens(
                    image_width=image_size.width,
                    image_height=image_size.height,
                    num_frames=videos.get_num_frames(item_idx),
                )

            return [video_token_id] * num_video_tokens

        return [
            PromptReplacement(
                modality="video",
                target=[video_token_id],
                replacement=get_replacement,
            ),
        ]
```
**EN:** Defines `LlavaNextVideoMultiModalProcessor`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from BaseMultiModalProcessor[LlavaNextVideoProcessingInfo]. Key methods such as `_get_mm_fields_config`, `_get_prompt_updates` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaNextVideoMultiModalProcessor`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 BaseMultiModalProcessor[LlavaNextVideoProcessingInfo]。 `_get_mm_fields_config`, `_get_prompt_updates` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaNextVideoPooler` (lines 236-268)
```python
class LlavaNextVideoPooler(nn.Module):
    def __init__(self, config: LlavaNextVideoConfig):
        super().__init__()

        mode = config.spatial_pool_mode
        stride = config.spatial_pool_stride
        image_size = config.vision_config.image_size
        patch_size = config.vision_config.patch_size
        self.image_size = image_size // patch_size**2

        if mode == "average":
            self.pool = nn.AvgPool2d(kernel_size=stride, stride=stride)
        elif mode == "max":
            self.pool = nn.MaxPool2d(kernel_size=stride, stride=stride)
        else:
            # TODO: Support Conv2d pooling layer, need to load weights
            raise ValueError(
                f"Unknown pooling mode: {mode}. Expected [`average`, `max`]"
            )

    def forward(self, image_features: torch.Tensor):
        ori_width = int(
            math.sqrt(image_features.shape[1] * self.image_size // self.image_size)
        )
        ori_height = int(ori_width * self.image_size // self.image_size)

        batch_size, _, dim = image_features.shape
        image_features_spatial = image_features.view(
            batch_size, ori_height, ori_height, dim
        ).permute(0, 3, 1, 2)
        image_features_spatial = self.pool(image_features_spatial)

        return image_features_spatial.flatten(2).transpose(1, 2).contiguous()
```
**EN:** Defines `LlavaNextVideoPooler`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaNextVideoPooler`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaNextMultiModalProjector` (lines 271-293)
```python
class LlavaNextMultiModalProjector(nn.Module):
    def __init__(
        self,
        vision_hidden_size: int,
        text_hidden_size: int,
        projector_hidden_act: str,
        multimodal_projector_bias: bool,
    ):
        super().__init__()

        self.linear_1 = nn.Linear(
            vision_hidden_size, text_hidden_size, bias=multimodal_projector_bias
        )
        self.act = get_act_fn(projector_hidden_act)
        self.linear_2 = nn.Linear(
            text_hidden_size, text_hidden_size, bias=multimodal_projector_bias
        )

    def forward(self, image_features: torch.Tensor) -> torch.Tensor:
        hidden_states = self.linear_1(image_features)
        hidden_states = self.act(hidden_states)
        hidden_states = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** Defines `LlavaNextMultiModalProjector`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaNextMultiModalProjector`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaNextVideoForConditionalGeneration` (lines 296-462)
```python
@MULTIMODAL_REGISTRY.register_processor(
    LlavaNextVideoMultiModalProcessor,
    info=LlavaNextVideoProcessingInfo,
    dummy_inputs=LlavaNextVideoDummyInputsBuilder,
)
class LlavaNextVideoForConditionalGeneration(nn.Module, SupportsMultiModal, SupportsPP):
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

        with self._mark_tower_model(vllm_config, "video"):
            # Initialize the vision tower only up to the required feature layer
            self.vision_tower = init_vision_tower_for_llava(
                config,
                quant_config=quant_config,
                require_post_norm=False,
                prefix=maybe_prefix(prefix, "vision_tower"),
            )
            self.vision_resampler = LlavaNextVideoPooler(config)
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        """Run forward pass for LlaVA-NeXT-Video.
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
        loader = AutoWeightsLoader(
            self,
            # This model doesn't support images for now
            ignore_unexpected_prefixes=["image_newline"],
        )
        return loader.load_weights(weights, mapper=self.hf_to_vllm_mapper)
```
**EN:** Defines `LlavaNextVideoForConditionalGeneration`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `_parse_and_validate_video_input`, `_video_pixels_to_features`, `_process_video_pixels` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaNextVideoForConditionalGeneration`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `_parse_and_validate_video_input`, `_video_pixels_to_features`, `_process_video_pixels` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **External libraries**: torch, torch.nn, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.activation, vllm.model_executor.models.clip, vllm.multimodal, vllm.multimodal.inputs, vllm.multimodal.parse
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .llava, .siglip, .utils, .vision
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
