# nano_nemotron_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/nano_nemotron_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Nano Nemotron VL multimodal model adapter used for inference in vLLM. / 实现 Nano Nemotron VL 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-95)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# --------------------------------------------------------
# Adapted from
# https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/models/internvl.py
# under Apache-2.0 License
#     LICENSE is in root directory.
# --------------------------------------------------------

import math
import warnings
from collections.abc import Iterable, Mapping, Sequence
from functools import cached_property
from io import BytesIO
from typing import Annotated, Literal, TypeAlias

import torch
import torch.nn as nn
# ... omitted for brevity ...
from vllm.transformers_utils.configs.radio import RadioConfig
from vllm.transformers_utils.processors.internvl import get_internvl_target_ratios
from vllm.transformers_utils.processors.nano_nemotron_vl import (
    AUDIO_CONTEXT,
    IMG_CONTEXT,
    IMG_END,
    IMG_START,
    BaseNanoNemotronVLProcessor,
    DynamicResolutionImageTiler,
    NanoNemotronVLProcessor,
    get_video_target_size_and_feature_size,
)
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .utils import _merge_multimodal_embeddings
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, transformers supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.logger connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, transformers 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.logger 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger, MAX_AUDIO_LEN_S` (lines 96-98)
```python
logger = init_logger(__name__)

MAX_AUDIO_LEN_S = 10 * 60  # 10 minutes
```
**EN:** This assignment block centers on `logger, MAX_AUDIO_LEN_S` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `logger, MAX_AUDIO_LEN_S` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Class `NanoNemotronVLAudioFeatureInputs` (lines 101-113)
```python
class NanoNemotronVLAudioFeatureInputs(TensorSchema):
    """
    Dimensions:
        - c: Number of audio clips (possibly flattened across audio items)
        - b: Number of original audio items
        - t: Audio feature length
        - f: Feature size (mel bins)
    """

    type: Literal["audio_features"] = "audio_features"
    input_audio_features: Annotated[torch.Tensor, TensorShape("c", "t", "f")]
    feature_attention_mask: Annotated[torch.Tensor, TensorShape("c", "t")]
    audio_num_clips: list[int]
```
**EN:** Defines `NanoNemotronVLAudioFeatureInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - c: Number of audio clips (possibly flattened across audio items) - b: Number of original audio items - t: Audio feature length - f: Feature size (mel bins)."
**CN:** 定义 `NanoNemotronVLAudioFeatureInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - c: Number of audio clips (possibly flattened across audio items) - b: Number of original audio items - t: Audio feature length - f: Feature size (mel bins)。”

### Class `NanoNemotronVLImagePixelInputs` (lines 116-128)
```python
class NanoNemotronVLImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - bnp: Batch size * number of images * (1 + num_patches)
        - c: Number of channels (3)
        - h: Height of each image patch
        - w: Width of each image patch
    """

    type: Literal["pixel_values"] = "pixel_values"
    pixel_values_flat: Annotated[torch.Tensor, TensorShape("bnp", 3, "h", "w")]
    num_patches: Annotated[torch.Tensor, TensorShape("bn")]
```
**EN:** Defines `NanoNemotronVLImagePixelInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - bnp: Batch size * number of images * (1 + num_patches) - c: Number of channels (3) - h: Height of each image patch - w: Width of each image patch."
**CN:** 定义 `NanoNemotronVLImagePixelInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - bnp: Batch size * number of images * (1 + num_patches) - c: Number of channels (3) - h: Height of each image patch - w: Width of each image patch。”

### Class `NanoNemotronVLImagePixelInputsDynamic` (lines 131-142)
```python
class NanoNemotronVLImagePixelInputsDynamic(TensorSchema):
    """
    Dynamic-resolution image inputs.

    imgs_sizes: per-image (height, width) in pixels.
    num_tokens_per_image: per-image number of embedding tokens (post downsample).
    """

    type: Literal["pixel_values_dynamic"] = "pixel_values_dynamic"
    pixel_values_flat: Annotated[torch.Tensor, TensorShape("bn", "h", "w")]
    imgs_sizes: list[tuple[int, int]]
    num_tokens_per_image: list[int]
```
**EN:** Defines `NanoNemotronVLImagePixelInputsDynamic`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dynamic-resolution image inputs."
**CN:** 定义 `NanoNemotronVLImagePixelInputsDynamic`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dynamic-resolution image inputs。”

### Class `NanoNemotronVLImageEmbeddingInputs` (lines 145-154)
```python
class NanoNemotronVLImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - n: Number of images
        - f: Total image feature size
        - h: Hidden size (must match the hidden size of language model backbone)
    """

    type: Literal["image_embeds"]
    data: Annotated[torch.Tensor | list[torch.Tensor], TensorShape("n", "f", "h")]
```
**EN:** Defines `NanoNemotronVLImageEmbeddingInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - n: Number of images - f: Total image feature size - h: Hidden size (must match the hidden size of language model backbone)."
**CN:** 定义 `NanoNemotronVLImageEmbeddingInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - n: Number of images - f: Total image feature size - h: Hidden size (must match the hidden size of language model backbone)。”

### Class `NanoNemotronVLVideoPixelInputs` (lines 164-179)
```python
class NanoNemotronVLVideoPixelInputs(TensorSchema):
    """
    Dimensions:
        - bvf: Batch size * number of videos * num_frames
        - bn: Batch size * number of videos
        - f: Number of frames
        - c: Number of channels (3)
        - h: Height of each video frame
        - w: Width of each video frame
    """

    type: Literal["pixel_values_videos"]
    pixel_values_flat: Annotated[torch.Tensor, TensorShape("bvf", 3, "h", "w")]
    num_patches: Annotated[torch.Tensor, TensorShape("bn")]
    frames_indices: Annotated[torch.Tensor, TensorShape("bvf")]
    frame_duration_ms: Annotated[torch.Tensor, TensorShape("bn")]
```
**EN:** Defines `NanoNemotronVLVideoPixelInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - bvf: Batch size * number of videos * num_frames - bn: Batch size * number of videos - f: Number of frames - c: Number of channels (3) - h: Height of each video frame - w: Width of each video frame."
**CN:** 定义 `NanoNemotronVLVideoPixelInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - bvf: Batch size * number of videos * num_frames - bn: Batch size * number of videos - f: Number of frames - c: Number of channels (3) - h: Height of each video frame - w: Width of each video frame。”

### Class `NanoNemotronVLVideoEmbeddingInputs` (lines 182-191)
```python
class NanoNemotronVLVideoEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - n: Number of videos
        - f: Total video feature size
        - h: Hidden size (must match the hidden size of language model backbone)
    """

    type: Literal["video_embeds"]
    data: Annotated[torch.Tensor | list[torch.Tensor], TensorShape("n", "f", "h")]
```
**EN:** Defines `NanoNemotronVLVideoEmbeddingInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - n: Number of videos - f: Total video feature size - h: Hidden size (must match the hidden size of language model backbone)."
**CN:** 定义 `NanoNemotronVLVideoEmbeddingInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - n: Number of videos - f: Total video feature size - h: Hidden size (must match the hidden size of language model backbone)。”

### Class `NemotronH_Nano_VL_V2` (lines 896-1626)
```python
@MULTIMODAL_REGISTRY.register_processor(
    NanoNemotronVLMultiModalProcessor,
    info=NanoNemotronVLProcessingInfo,
    dummy_inputs=NanoNemotronVLDummyInputsBuilder,
)
class NemotronH_Nano_VL_V2(
    nn.Module, HasInnerState, IsHybrid, SupportsMultiModal, SupportsMultiModalPruning
):
    requires_sequential_video_encoding = True
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        model_config = vllm_config.model_config
        config = model_config.hf_config
        multimodal_config = model_config.multimodal_config
        image_size = config.force_image_size
        patch_size = config.patch_size
        self.patch_size = patch_size
        self.template = config.template
        self.num_image_token = int(
            (image_size // patch_size) ** 2 * (config.downsample_ratio**2)
        )
        self.downsample_ratio = config.downsample_ratio
        self.ps_version = config.ps_version
        self.image_tag_type = config.image_tag_type
        self.video_pruning_rate = multimodal_config.video_pruning_rate

        vision_config = getattr(config, "vision_config", config)
        self.video_temporal_patch_size: int = getattr(
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
# ... omitted for brevity ...
        hidden_states = self.language_model(
            input_ids=input_ids,
            positions=positions,
            intermediate_tensors=intermediate_tensors,
            inputs_embeds=inputs_embeds,
            **kwargs,
        )

# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.language_model.compute_logits(hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        mm_config = self.model_config.multimodal_config
        load_multimodal_weights = not all(
            mm_config.get_limit_per_prompt(modality) == 0
            for modality in ("image", "video", "audio")
        )
        adapter_dict = dict(self.mlp1.named_parameters())

        def is_llm(name: str) -> bool:
            return name.startswith("language_model")

        def is_adapter_weights(weight: tuple[str, torch.Tensor]):
            return weight[0].startswith("mlp1")

        def is_vision_weights(name: str) -> bool:
            return name.startswith("vision_model.radio_model.")

        def is_sound_weights(name: str) -> bool:
            return name.startswith("sound")
```
**EN:** Defines `NemotronH_Nano_VL_V2`, a supporting module used by the surrounding model implementation. It inherits from nn.Module, HasInnerState, IsHybrid. Key methods such as `get_placeholder_str`, `__init__`, `pixel_shuffle`, `pixel_shuffle_dynamic_res`, `extract_feature_dynamic` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `NemotronH_Nano_VL_V2`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module、HasInnerState、IsHybrid。 `get_placeholder_str`, `__init__`, `pixel_shuffle`, `pixel_shuffle_dynamic_res`, `extract_feature_dynamic` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。
- **Position encoding**
  - **EN:** Rotary or related positional logic appears in the file to preserve token order information efficiently.
  - **CN:** 文件中出现了 Rotary 等位置编码逻辑，用于高效保留 token 顺序信息。

## Dependencies / 依赖关系
- **Standard library**: math, warnings, collections.abc, functools, io, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.logger, vllm.model_executor.layers.activation, vllm.model_executor.layers.layernorm, vllm.model_executor.model_loader.weight_utils, vllm.model_executor.models.interfaces
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
