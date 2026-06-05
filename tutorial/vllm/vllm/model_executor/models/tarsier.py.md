# tarsier.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/tarsier.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Tarsier multimodal model adapter used for inference in vLLM. / 实现 Tarsier 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-59)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import math
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Final, Literal, Protocol, TypeAlias, TypeVar

import torch
import torch.nn as nn
from transformers import (
    BatchFeature,
    CLIPVisionConfig,
    PretrainedConfig,
    SiglipVisionConfig,
)
from transformers import LlavaConfig as HfLlavaConfig
from transformers.image_utils import ImageInput, get_image_size, to_numpy_array
from transformers.models.llava import LlavaProcessor
# ... omitted for brevity ...
from .clip import CLIPVisionModel
from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .siglip import SiglipVisionModel
from .utils import (
    AutoWeightsLoader,
    get_layer_index,
    init_vllm_registered_model,
    maybe_prefix,
)
from .vision import (
    VisionEncoderInfo,
    get_num_selected_vision_tokens,
    get_vision_encoder_info,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, transformers, transformers.image_utils supply framework primitives, while internal modules like vllm.config, vllm.model_executor.layers.activation, vllm.model_executor.layers.linear, vllm.model_executor.layers.quantization connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, transformers, transformers.image_utils 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.model_executor.layers.activation, vllm.model_executor.layers.linear, vllm.model_executor.layers.quantization 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Class `TarsierImagePixelInputs` (lines 60-70)
```python
class TarsierImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - c: Number of channels (3)
        - h: Height
        - w: Width
    """

    type: Literal["pixel_values"] = "pixel_values"
    pixel_values: Annotated[torch.Tensor, TensorShape("bn", 3, "h", "w")]
```
**EN:** Defines `TarsierImagePixelInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - c: Number of channels (3) - h: Height - w: Width."
**CN:** 定义 `TarsierImagePixelInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - c: Number of channels (3) - h: Height - w: Width。”

### Class `TarsierImageEmbeddingInputs` (lines 73-83)
```python
class TarsierImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - ifs: Image feature size
        - hs: Hidden size (must match the hidden size of language model
          backbone)
    """

    type: Literal["image_embeds"] = "image_embeds"
    data: Annotated[torch.Tensor, TensorShape("bn", "ifs", "hs")]
```
**EN:** Defines `TarsierImageEmbeddingInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - ifs: Image feature size - hs: Hidden size (must match the hidden size of language model backbone)."
**CN:** 定义 `TarsierImageEmbeddingInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - ifs: Image feature size - hs: Hidden size (must match the hidden size of language model backbone)。”

### Class `TarsierHfConfig` (lines 89-98)
```python
class TarsierHfConfig(Protocol):  # Based on the Tarsier's LlavaConfig
    vision_config: Final[PretrainedConfig]
    text_config: Final[PretrainedConfig]  # Added from Tarsier's LlavaConfig
    image_token_index: Final[int]
    vision_feature_select_strategy: Final[str]
    vision_feature_layer: Final[int | list[int]]
    projector_hidden_act: Final[str]
    image_newline_idx: Final[int]
    image_new_idx: Final[int]
    multimodal_projector_bias: bool = True
```
**EN:** Defines `TarsierHfConfig`, a supporting module used by the surrounding model implementation. It inherits from Protocol.
**CN:** 定义 `TarsierHfConfig`，它是一个被周边模型实现复用的支撑模块。 它继承自 Protocol。

### Class `TarsierProcessorKwargs` (lines 101-107)
```python
class TarsierProcessorKwargs(ProcessingKwargs, total=False):
    _defaults = {
        "text_kwargs": {
            "padding": False,
        },
        "images_kwargs": {},
    }
```
**EN:** Defines `TarsierProcessorKwargs`, a supporting module used by the surrounding model implementation. It inherits from ProcessingKwargs.
**CN:** 定义 `TarsierProcessorKwargs`，它是一个被周边模型实现复用的支撑模块。 它继承自 ProcessingKwargs。

### Class `TarsierProcessor` (lines 110-169)
```python
class TarsierProcessor(LlavaProcessor):
    def __call__(
        self,
        images: ImageInput = None,
        text: TextInput
        | PreTokenizedInput
        | list[TextInput]
        | list[PreTokenizedInput] = None,
        audio=None,
        videos=None,
        **kwargs: Unpack[TarsierProcessorKwargs],
    ) -> BatchFeature:
        if images is None and text is None:
            raise ValueError("You have to specify at least one of `images` or `text`.")

        output_kwargs = self._merge_kwargs(
            TarsierProcessorKwargs,
            tokenizer_init_kwargs=self.tokenizer.init_kwargs,
            **kwargs,
        )
        if images is not None:
            image_inputs = self.image_processor(
                images, **output_kwargs["images_kwargs"]
            )
        else:
            image_inputs = {}

        if isinstance(text, str):
            text = [text]
        elif not isinstance(text, list) and not isinstance(text[0], str):
            raise ValueError(
                "Invalid input text. Please provide a string, or a list of strings"
            )

        # try to expand inputs in processing if we have the necessary parts
        prompt_strings = text
        if image_inputs.get("pixel_values") is not None:
            # Replace the image token with the expanded image token sequence
            pixel_values = image_inputs["pixel_values"]
            height, width = get_image_size(to_numpy_array(pixel_values[0]))
            num_image_tokens = (
                (height // self.patch_size) * (width // self.patch_size + 1)
                + self.num_additional_image_tokens
                + 1
            )
            if self.vision_feature_select_strategy == "default":
                num_image_tokens -= 1

            prompt_strings = []
            for sample in text:
                sample = sample.replace(
                    self.image_token, self.image_token * num_image_tokens
                )
                prompt_strings.append(sample)

        return_tensors = output_kwargs["text_kwargs"].pop("return_tensors", None)
        text_inputs = self.tokenizer(prompt_strings, **output_kwargs["text_kwargs"])
        return BatchFeature(
            data={**text_inputs, **image_inputs}, tensor_type=return_tensors
        )
```
**EN:** Defines `TarsierProcessor`, a supporting module used by the surrounding model implementation. It inherits from LlavaProcessor. Key methods such as `__call__` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `TarsierProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 LlavaProcessor。 `__call__` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `TarsierMultiModalProjector` (lines 172-204)
```python
class TarsierMultiModalProjector(nn.Module):
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
**EN:** Defines `TarsierMultiModalProjector`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `TarsierMultiModalProjector`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `TarsierProcessingInfo` (lines 207-271)
```python
class TarsierProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self) -> TarsierHfConfig:
        return self.ctx.get_hf_config(HfLlavaConfig)

    def get_vision_encoder_info(self) -> VisionEncoderInfo:
        return get_vision_encoder_info(self.get_hf_config())

    def get_hf_processor(self, **kwargs: object) -> TarsierProcessor:
        vision_info = self.get_vision_encoder_info()

        kwargs.setdefault("patch_size", vision_info.get_patch_size())

        return self.ctx.get_hf_processor(TarsierProcessor, **kwargs)

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
        num_projected_patches = get_num_selected_vision_tokens(
            vision_encoder_info.get_num_image_tokens(
                image_width=image_width,
                image_height=image_height,
            ),
# ... omitted for brevity ...
                    image_width=default_size.width,
                    image_height=default_size.height,
                ),
                hf_config.vision_feature_select_strategy,
            )
            if num_projected_patches_default <= 0:
                raise ValueError("Could not determine a valid number of image patches.")
            num_projected_patches = num_projected_patches_default
        num_height_patches = int(math.sqrt(num_projected_patches))
        total_image_tokens_for_llm = num_projected_patches + num_height_patches + 1
        return total_image_tokens_for_llm

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

    def get_image_newline_idx(self) -> int:
        return self.get_hf_config().image_newline_idx

    def get_image_new_idx(self) -> int:
        return self.get_hf_config().image_new_idx
```
**EN:** Defines `TarsierProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseProcessingInfo. Key methods such as `get_hf_config`, `get_vision_encoder_info`, `get_hf_processor`, `get_supported_mm_limits`, `get_num_image_tokens` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `TarsierProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseProcessingInfo。 `get_hf_config`, `get_vision_encoder_info`, `get_hf_processor`, `get_supported_mm_limits`, `get_num_image_tokens` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `TarsierDummyInputsBuilder` (lines 277-278)
```python
class TarsierDummyInputsBuilder(LlavaDummyInputsBuilder[_I_Tarsier]):
    pass
```
**EN:** Defines `TarsierDummyInputsBuilder`, a supporting module used by the surrounding model implementation. It inherits from LlavaDummyInputsBuilder[_I_Tarsier].
**CN:** 定义 `TarsierDummyInputsBuilder`，它是一个被周边模型实现复用的支撑模块。 它继承自 LlavaDummyInputsBuilder[_I_Tarsier]。

### Class `TarsierMultiModalProcessor` (lines 281-326)
```python
class TarsierMultiModalProcessor(BaseMultiModalProcessor[_I_Tarsier]):
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return dict(
            pixel_values=MultiModalFieldConfig.batched("image"),
            image_embeds=MultiModalFieldConfig.batched("image"),
        )

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        hf_config = self.info.get_hf_config()
        image_token_id = hf_config.image_token_index  # The <IMAGE> token ID

        def get_replacement(item_idx: int):
            images = mm_items.get_items(
                "image", (ImageEmbeddingItems, ImageProcessorItems)
            )

            if isinstance(images, ImageEmbeddingItems):
                num_projected_patches = images.get_feature_size(item_idx)
                # This assumes num_projected_patches is a perfect square
                num_height_patches = int(math.sqrt(num_projected_patches))
                num_final_image_tokens = num_projected_patches + num_height_patches + 1
            else:
                image_size = images.get_image_size(item_idx)
                num_final_image_tokens = self.info.get_num_image_tokens(
                    image_width=image_size.width,
                    image_height=image_size.height,
                )

            return [image_token_id] * num_final_image_tokens

        return [
            PromptReplacement(
                modality="image",
                target=[image_token_id],  # Replace each single <IMAGE> token
                replacement=get_replacement,
            ),
        ]
```
**EN:** Defines `TarsierMultiModalProcessor`, a supporting module used by the surrounding model implementation. It inherits from BaseMultiModalProcessor[_I_Tarsier]. Key methods such as `_get_mm_fields_config`, `_get_prompt_updates` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `TarsierMultiModalProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseMultiModalProcessor[_I_Tarsier]。 `_get_mm_fields_config`, `_get_prompt_updates` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `TarsierForConditionalGeneration` (lines 375-591)
```python
@MULTIMODAL_REGISTRY.register_processor(
    TarsierMultiModalProcessor,
    info=TarsierProcessingInfo,
    dummy_inputs=TarsierDummyInputsBuilder,
)
class TarsierForConditionalGeneration(nn.Module, SupportsMultiModal, SupportsPP):
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()

        config: TarsierHfConfig = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config  # Storing the Tarsier-specific HF config

        with self._mark_tower_model(vllm_config, "image"):
            self.vision_tower = init_vision_tower_for_tarsier(
                config,
                quant_config=quant_config,
                require_post_norm=False,
                prefix=maybe_prefix(prefix, "vision_tower"),
            )
            projector_bias = getattr(config, "multimodal_projector_bias", True)

            self.multi_modal_projector = TarsierMultiModalProjector(
                vision_hidden_size=config.vision_config.hidden_size,
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        if intermediate_tensors is not None:
            inputs_embeds = None

        hidden_states = self.language_model.model(
            input_ids=input_ids,
            positions=positions,
            intermediate_tensors=intermediate_tensors,
            inputs_embeds=inputs_embeds,
        )
        return hidden_states
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.language_model.compute_logits(hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights)
```
**EN:** Defines `TarsierForConditionalGeneration`, a supporting module used by the surrounding model implementation. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `_parse_and_validate_image_input`, `_image_pixels_to_features`, `_add_tarsier_split_tokens` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `TarsierForConditionalGeneration`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `_parse_and_validate_image_input`, `_image_pixels_to_features`, `_add_tarsier_split_tokens` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **Standard library**: math, collections.abc, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, transformers, transformers.image_utils, transformers.models.llava, transformers.processing_utils, transformers.tokenization_utils_base
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.model_executor.layers.activation, vllm.model_executor.layers.linear, vllm.model_executor.layers.quantization, vllm.model_executor.models.llava, vllm.multimodal, vllm.multimodal.inputs, vllm.multimodal.parse
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .clip, .interfaces, .siglip, .utils, .vision
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
