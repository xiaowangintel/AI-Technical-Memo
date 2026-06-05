# skyworkr1v.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/skyworkr1v.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Skyworkr1v multimodal model adapter used for inference in vLLM. / 实现 Skyworkr1v 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-44)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# adapted from https://huggingface.co/Skywork/Skywork-R1V-38B/blob/main/modeling_skywork_chat.py
# --------------------------------------------------------
# SkyworkR1V
# Copyright (c) 2025 Skywork
# Licensed under The MIT License [see LICENSE for details]
# --------------------------------------------------------
from collections.abc import Iterable, Mapping
from typing import Annotated, Literal, TypeAlias

import torch
import torch.nn as nn
from transformers import PretrainedConfig

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
# ... omitted for brevity ...
from vllm.sequence import IntermediateTensors
from vllm.transformers_utils.processors.internvl import (
    InternVLImageProcessor,
    InternVLProcessor,
)
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .internvl import (
    BaseInternVLDummyInputsBuilder,
    BaseInternVLMultiModalProcessor,
    BaseInternVLProcessingInfo,
)
from .utils import AutoWeightsLoader, init_vllm_registered_model, maybe_prefix
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, transformers supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.linear connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, transformers 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.linear 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Class `SkyworkR1VImagePixelInputs` (lines 45-65)
```python
class SkyworkR1VImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bnp: Batch size * number of images * (1 + num_patches)
        - c: Number of channels (3)
        - h: Height
        - w: Width
        - bn: Batch size * number of images
    """

    type: Literal["pixel_values"] = "pixel_values"

    pixel_values_flat: Annotated[
        torch.Tensor,
        TensorShape("bnp", 3, "h", "w"),
    ]

    num_patches: Annotated[
        torch.Tensor,
        TensorShape("bn"),
    ]
```
**EN:** Defines `SkyworkR1VImagePixelInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bnp: Batch size * number of images * (1 + num_patches) - c: Number of channels (3) - h: Height - w: Width - bn: Batch size * number of images."
**CN:** 定义 `SkyworkR1VImagePixelInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bnp: Batch size * number of images * (1 + num_patches) - c: Number of channels (3) - h: Height - w: Width - bn: Batch size * number of images。”

### Class `SkyworkR1VImageEmbeddingInputs` (lines 68-82)
```python
class SkyworkR1VImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - ni: Number of images
        - ifs: Image feature size
        - hs: Hidden size (must match the hidden size of language model
          backbone)
    """

    type: Literal["image_embeds"] = "image_embeds"

    data: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("ni", "ifs", "hs"),
    ]
```
**EN:** Defines `SkyworkR1VImageEmbeddingInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - ni: Number of images - ifs: Image feature size - hs: Hidden size (must match the hidden size of language model backbone)."
**CN:** 定义 `SkyworkR1VImageEmbeddingInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - ni: Number of images - ifs: Image feature size - hs: Hidden size (must match the hidden size of language model backbone)。”

### Top-level mapping `SkyworkR1VImageInputs` (lines 85-87)
```python
SkyworkR1VImageInputs: TypeAlias = (
    SkyworkR1VImagePixelInputs | SkyworkR1VImageEmbeddingInputs
)
```
**EN:** This assignment block centers on `SkyworkR1VImageInputs` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `SkyworkR1VImageInputs` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Class `SkyworkR1VProcessingInfo` (lines 90-118)
```python
class SkyworkR1VProcessingInfo(BaseInternVLProcessingInfo):
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

    def get_hf_processor(self, **kwargs: object) -> InternVLProcessor:
        config = self.get_hf_config()
        vision_config = config.vision_config

        image_processor = self.get_image_processor(**kwargs)
        image_size = image_processor.image_size
        patch_size = vision_config.patch_size
        downsample_ratio = config.downsample_ratio
        image_seq_length = int((image_size // patch_size) ** 2 * (downsample_ratio**2))

        return InternVLProcessor(
            tokenizer=self.get_tokenizer(),
            image_processor=image_processor,
            image_seq_length=image_seq_length,
        )
```
**EN:** Defines `SkyworkR1VProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseInternVLProcessingInfo. Key methods such as `get_image_processor`, `get_hf_processor` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SkyworkR1VProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseInternVLProcessingInfo。 `get_image_processor`, `get_hf_processor` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SkyworkR1VDummyInputsBuilder` (lines 121-145)
```python
class SkyworkR1VDummyInputsBuilder(BaseDummyInputsBuilder[SkyworkR1VProcessingInfo]):
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
            )
        }
```
**EN:** Defines `SkyworkR1VDummyInputsBuilder`, a supporting module used by the surrounding model implementation. It inherits from BaseDummyInputsBuilder[SkyworkR1VProcessingInfo]. Key methods such as `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SkyworkR1VDummyInputsBuilder`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseDummyInputsBuilder[SkyworkR1VProcessingInfo]。 `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SkyworkR1VChatModel` (lines 148-449)
```python
@MULTIMODAL_REGISTRY.register_processor(
    BaseInternVLMultiModalProcessor,
    info=SkyworkR1VProcessingInfo,
    dummy_inputs=BaseInternVLDummyInputsBuilder,
)
class SkyworkR1VChatModel(nn.Module, SupportsMultiModal, SupportsPP):
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config

        self.config = config
        self.multimodal_config = multimodal_config
        self._patch_quant_config(config, quant_config)

        image_size = config.force_image_size or config.vision_config.image_size
        patch_size = config.vision_config.patch_size
        self.patch_size = patch_size
        self.num_image_token = int(
            (image_size // patch_size) ** 2 * (config.downsample_ratio**2)
        )
        self.downsample_ratio = config.downsample_ratio
        self.ps_version = config.ps_version
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
# ... omitted for brevity ...
            inputs_embeds = None

        forward_kwargs = {
            "input_ids": input_ids,
            "positions": positions,
            "intermediate_tensors": intermediate_tensors,
            "inputs_embeds": inputs_embeds,
        }

        # Only required if the model is mono-architecture
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.language_model.compute_logits(hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        skip_prefixes = [
            "action_embed",
            "temporal_embed",
            "track_embed",
            "track_embed_decoder",
            "box_token",
            "cg_criterion",
            "cg_model",
            "loc_encoder",
            "loc_decoder",
            "sam",
            "temporal_token",
            "track_token",
        ]
        loader = AutoWeightsLoader(self, skip_prefixes=skip_prefixes)
        return loader.load_weights(weights)
```
**EN:** Defines `SkyworkR1VChatModel`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `_patch_quant_config`, `_init_vision_model`, `_init_mlp1` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SkyworkR1VChatModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `_patch_quant_config`, `_init_vision_model`, `_init_mlp1` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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

## Dependencies / 依赖关系
- **Standard library**: collections.abc, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.linear, vllm.model_executor.layers.quantization, vllm.model_executor.layers.quantization.awq, vllm.model_executor.models.intern_vit, vllm.multimodal
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .internvl, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
