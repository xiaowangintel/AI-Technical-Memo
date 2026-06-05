# llava.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/llava.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the LLaVA multimodal model adapter used for inference in vLLM. / 实现 LLaVA 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-74)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from abc import abstractmethod
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Final, Literal, Protocol, TypeAlias, TypeVar

import torch
import torch.nn as nn
from transformers import (
    BatchFeature,
    CLIPVisionConfig,
    LlavaConfig,
    PixtralVisionConfig,
    PretrainedConfig,
    SiglipVisionConfig,
)
from transformers.models.llava import LlavaProcessor
# ... omitted for brevity ...
    SupportsMultiModal,
    SupportsPP,
)
from .module_mapping import MultiModelKeys
from .pixtral import PixtralHFEncoderInfo, PixtralHFVisionModel
from .siglip import SiglipVisionModel
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    get_layer_index,
    init_vllm_registered_model,
    maybe_prefix,
)
from .vision import get_num_selected_vision_tokens, get_vision_encoder_info
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, transformers, transformers.models.llava supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.activation connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, transformers, transformers.models.llava 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.activation 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Class `LlavaImagePixelInputs` (lines 75-88)
```python
class LlavaImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - c: Number of channels (3)
        - h: Height
        - w: Width

    Note that `height` or `width` may be different per batch and image,
    in which case the data is passed as a list instead of a batched tensor.
    """

    type: Literal["pixel_values"] = "pixel_values"
    pixel_values: Annotated[torch.Tensor, TensorShape("bn", 3, "h", "w")]
```
**EN:** Defines `LlavaImagePixelInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - c: Number of channels (3) - h: Height - w: Width Note that `height` or `width` may be different per batch and image, in which case the data is passed as a list instead of."
**CN:** 定义 `LlavaImagePixelInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - c: Number of channels (3) - h: Height - w: Width Note that `height` or `width` may be different per batch and image, in which case the data is passed as a list instead of。”

### Class `PixtralHFImagePixelInputs` (lines 91-107)
```python
class PixtralHFImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - c: Number of channels
        - h: Height
        - w: Width

    Note that `height` or `width` may be different per batch and image,
    in which case the data is passed as a list instead of a batched tensor.
    """

    type: Literal["pixel_values_pixtral"] = "pixel_values_pixtral"
    pixel_values: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("bn", "c", "h", "w", dynamic_dims={"h", "w"}),
    ]
```
**EN:** Defines `PixtralHFImagePixelInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - c: Number of channels - h: Height - w: Width Note that `height` or `width` may be different per batch and image, in which case the data is passed as a list instead of a b."
**CN:** 定义 `PixtralHFImagePixelInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - c: Number of channels - h: Height - w: Width Note that `height` or `width` may be different per batch and image, in which case the data is passed as a list instead of a b。”

### Class `LlavaImageEmbeddingInputs` (lines 110-119)
```python
class LlavaImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - ifs: Image feature size
        - hs: Hidden size (must match language model backbone)
    """

    type: Literal["image_embeds"] = "image_embeds"
    data: Annotated[torch.Tensor, TensorShape("bn", "ifs", "hs")]
```
**EN:** Defines `LlavaImageEmbeddingInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - ifs: Image feature size - hs: Hidden size (must match language model backbone)."
**CN:** 定义 `LlavaImageEmbeddingInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - ifs: Image feature size - hs: Hidden size (must match language model backbone)。”

### Class `LlavaMultiModalProjector` (lines 128-160)
```python
class LlavaMultiModalProjector(nn.Module):
    def __init__(
        self,
        vision_hidden_size: int,
        text_hidden_size: int,
        projector_hidden_act: str,
        multimodal_projector_bias: bool,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.linear_1 = ColumnParallelLinear(
            vision_hidden_size,
            text_hidden_size,
            bias=multimodal_projector_bias,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_1",
        )
        self.act = get_act_fn(projector_hidden_act)
        self.linear_2 = RowParallelLinear(
            text_hidden_size,
            text_hidden_size,
            bias=multimodal_projector_bias,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_2",
        )

    def forward(self, image_features: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.linear_1(image_features)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** Defines `LlavaMultiModalProjector`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaMultiModalProjector`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaLikeConfig` (lines 163-167)
```python
class LlavaLikeConfig(Protocol):
    vision_config: Final[PretrainedConfig]
    image_token_index: Final[int]
    vision_feature_select_strategy: Final[str]
    vision_feature_layer: Final[int | list[int]]
```
**EN:** Defines `LlavaLikeConfig`, a supporting module used by the surrounding model implementation. It inherits from Protocol.
**CN:** 定义 `LlavaLikeConfig`，它是一个被周边模型实现复用的支撑模块。 它继承自 Protocol。

### Class `LlavaLikeProcessor` (lines 170-171)
```python
class LlavaLikeProcessor(Protocol):
    image_token: Final[str]
```
**EN:** Defines `LlavaLikeProcessor`, a supporting module used by the surrounding model implementation. It inherits from Protocol.
**CN:** 定义 `LlavaLikeProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 Protocol。

### Class `BaseLlavaProcessingInfo` (lines 174-216)
```python
class BaseLlavaProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self) -> LlavaLikeConfig:
        return self.ctx.get_hf_config(LlavaConfig)

    def get_vision_encoder_info(self):
        return get_vision_encoder_info(self.get_hf_config())

    @abstractmethod
    def get_hf_processor(self, **kwargs: object) -> LlavaLikeProcessor:
        raise NotImplementedError

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        hf_config = self.get_hf_config()
        vision_encoder_info = self.get_vision_encoder_info()

        return get_num_selected_vision_tokens(
            vision_encoder_info.get_num_image_tokens(
                image_width=image_width,
                image_height=image_height,
            ),
            hf_config.vision_feature_select_strategy,
        )

    def get_image_size_with_most_features(self) -> ImageSize:
        vision_encoder_info = self.get_vision_encoder_info()
        width = height = vision_encoder_info.get_image_size()
        return ImageSize(width=width, height=height)

    def get_max_image_tokens(self) -> int:
        target_width, target_height = self.get_image_size_with_most_features()

        return self.get_num_image_tokens(
            image_width=target_width,
            image_height=target_height,
        )
```
**EN:** Defines `BaseLlavaProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseProcessingInfo. Key methods such as `get_hf_config`, `get_vision_encoder_info`, `get_hf_processor`, `get_supported_mm_limits`, `get_num_image_tokens` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `BaseLlavaProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseProcessingInfo。 `get_hf_config`, `get_vision_encoder_info`, `get_hf_processor`, `get_supported_mm_limits`, `get_num_image_tokens` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaDummyInputsBuilder` (lines 222-250)
```python
class LlavaDummyInputsBuilder(BaseDummyInputsBuilder[_I]):
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
**EN:** Defines `LlavaDummyInputsBuilder`, a supporting module used by the surrounding model implementation. It inherits from BaseDummyInputsBuilder[_I]. Key methods such as `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaDummyInputsBuilder`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseDummyInputsBuilder[_I]。 `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlavaProcessingInfo` (lines 253-261)
```python
class LlavaProcessingInfo(BaseLlavaProcessingInfo):
    def get_hf_processor(self, **kwargs: object):
        hf_processor = self.ctx.get_hf_processor(LlavaProcessor, **kwargs)
        # In case patch_size is omitted from `processor_config.json`
        # e.g. for E5-V: https://huggingface.co/royokong/e5-v
        if hf_processor.patch_size is None:
            patch_size = self.get_vision_encoder_info().get_patch_size()
            hf_processor.patch_size = patch_size
        return hf_processor
```
**EN:** Defines `LlavaProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseLlavaProcessingInfo. Key methods such as `get_hf_processor` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlavaProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseLlavaProcessingInfo。 `get_hf_processor` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MantisForConditionalGeneration` (lines 844-850)
```python
@MULTIMODAL_REGISTRY.register_processor(
    MantisMultiModalProcessor,
    info=MantisProcessingInfo,
    dummy_inputs=LlavaDummyInputsBuilder,
)
class MantisForConditionalGeneration(LlavaForConditionalGeneration):
    pass
```
**EN:** Defines `MantisForConditionalGeneration`, a supporting module used by the surrounding model implementation. It inherits from LlavaForConditionalGeneration.
**CN:** 定义 `MantisForConditionalGeneration`，它是一个被周边模型实现复用的支撑模块。 它继承自 LlavaForConditionalGeneration。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Quantized weights**
  - **EN:** Quantization hooks allow the model to load compressed checkpoints or specialized linear layers.
  - **CN:** 量化钩子使模型能够加载压缩检查点或专用线性层。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。

## Dependencies / 依赖关系
- **Standard library**: abc, collections.abc, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, transformers, transformers.models.llava, transformers.models.pixtral
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.activation, vllm.model_executor.layers.linear, vllm.model_executor.layers.quantization, vllm.multimodal, vllm.multimodal.cache
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .clip, .interfaces, .module_mapping, .pixtral, .siglip, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
