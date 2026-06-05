# paligemma.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/paligemma.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Paligemma multimodal model adapter used for inference in vLLM. / 实现 Paligemma 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-54)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Literal, TypeAlias

import torch
from torch import nn
from transformers import BatchFeature, PaliGemmaConfig

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict, MultiModalInput
from vllm.logger import init_logger
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
# ... omitted for brevity ...
from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from .module_mapping import MultiModelKeys
from .siglip import SiglipVisionModel
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
from .vision import get_vision_encoder_info
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, transformers supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.logger connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, transformers 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.logger 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger` (lines 55-55)
```python
logger = init_logger(__name__)
```
**EN:** This assignment initializes the module-level logger used for diagnostics, backend selection messages, and compatibility warnings.
**CN:** 这个赋值语句初始化了模块级日志记录器，用于输出诊断信息、后端选择消息和兼容性警告。

### Class `PaliGemmaImagePixelInputs` (lines 58-68)
```python
class PaliGemmaImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - c: Number of channels (3)
        - h: Height
        - w: Width
    """

    type: Literal["pixel_values"] = "pixel_values"
    data: Annotated[torch.Tensor, TensorShape("bn", 3, "h", "w")]
```
**EN:** Defines `PaliGemmaImagePixelInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - c: Number of channels (3) - h: Height - w: Width."
**CN:** 定义 `PaliGemmaImagePixelInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - c: Number of channels (3) - h: Height - w: Width。”

### Class `PaliGemmaImageEmbeddingInputs` (lines 71-80)
```python
class PaliGemmaImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - ifs: Image feature size
        - hs: Hidden size (must match language model backbone)
    """

    type: Literal["image_embeds"] = "image_embeds"
    data: Annotated[torch.Tensor, TensorShape("bn", "ifs", "hs")]
```
**EN:** Defines `PaliGemmaImageEmbeddingInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - ifs: Image feature size - hs: Hidden size (must match language model backbone)."
**CN:** 定义 `PaliGemmaImageEmbeddingInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - ifs: Image feature size - hs: Hidden size (must match language model backbone)。”

### Top-level mapping `PaliGemmaImageInputs` (lines 83-85)
```python
PaliGemmaImageInputs: TypeAlias = (
    PaliGemmaImagePixelInputs | PaliGemmaImageEmbeddingInputs
)
```
**EN:** This assignment block centers on `PaliGemmaImageInputs` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `PaliGemmaImageInputs` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Class `PaliGemmaMultiModalProjector` (lines 88-96)
```python
class PaliGemmaMultiModalProjector(nn.Module):
    def __init__(self, vision_hidden_size: int, projection_dim: int):
        super().__init__()

        self.linear = nn.Linear(vision_hidden_size, projection_dim, bias=True)

    def forward(self, image_features: torch.Tensor) -> torch.Tensor:
        hidden_states = self.linear(image_features)
        return hidden_states
```
**EN:** Defines `PaliGemmaMultiModalProjector`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `PaliGemmaMultiModalProjector`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `PaliGemmaProcessingInfo` (lines 99-123)
```python
class PaliGemmaProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(PaliGemmaConfig)

    def get_vision_encoder_info(self):
        return get_vision_encoder_info(self.get_hf_config())

    def get_default_tok_params(self) -> TokenizeParams:
        return super().get_default_tok_params().with_kwargs(add_special_tokens=False)

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": 1}

    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        vision_encoder_info = self.get_vision_encoder_info()

        return vision_encoder_info.get_num_image_tokens(
            image_width=image_width,
            image_height=image_height,
        )
```
**EN:** Defines `PaliGemmaProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseProcessingInfo. Key methods such as `get_hf_config`, `get_vision_encoder_info`, `get_default_tok_params`, `get_supported_mm_limits`, `get_num_image_tokens` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `PaliGemmaProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseProcessingInfo。 `get_hf_config`, `get_vision_encoder_info`, `get_default_tok_params`, `get_supported_mm_limits`, `get_num_image_tokens` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `PaliGemmaDummyInputsBuilder` (lines 126-151)
```python
class PaliGemmaDummyInputsBuilder(BaseDummyInputsBuilder[PaliGemmaProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        return ""

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        hf_config = self.info.get_hf_config()
        vision_config = hf_config.vision_config
        max_image_size = vision_config.image_size

        num_images = mm_counts.get("image", 0)

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=max_image_size,
                height=max_image_size,
                num_images=num_images,
                overrides=image_overrides,
            )
        }
```
**EN:** Defines `PaliGemmaDummyInputsBuilder`, a supporting module used by the surrounding model implementation. It inherits from BaseDummyInputsBuilder[PaliGemmaProcessingInfo]. Key methods such as `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `PaliGemmaDummyInputsBuilder`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseDummyInputsBuilder[PaliGemmaProcessingInfo]。 `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `PaliGemmaMultiModalProcessor` (lines 154-246)
```python
class PaliGemmaMultiModalProcessor(BaseMultiModalProcessor[PaliGemmaProcessingInfo]):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        tokenizer = self.info.get_tokenizer()
        if not mm_data:
            prompt_ids = tokenizer.encode(prompt, add_special_tokens=False)
            return BatchFeature(dict(input_ids=[prompt_ids]), tensor_type="pt")

        return super()._call_hf_processor(
            prompt=prompt,
            mm_data=mm_data,
            mm_kwargs=mm_kwargs,
            tok_kwargs=tok_kwargs,
        )
# ... omitted for brevity ...
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return dict(pixel_values=MultiModalFieldConfig.batched("image"))
# ... omitted for brevity ...
    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        hf_config = self.info.get_hf_config()
        image_token_id = hf_config.image_token_index

        tokenizer = self.info.get_tokenizer()

        bos_token_id = tokenizer.bos_token_id
        assert isinstance(bos_token_id, int)

        def get_insertion(item_idx: int):
            images = mm_items.get_items(
                "image", (ImageEmbeddingItems, ImageProcessorItems)
            )

# ... omitted for brevity ...
    def apply(
        self,
        inputs: ProcessorInputs,
        timing_ctx: TimingContext,
    ) -> MultiModalInput:
        mm_inputs = super().apply(inputs, timing_ctx)
        prompt_token_ids = mm_inputs["prompt_token_ids"]

        tokenizer = self.info.get_tokenizer()
        newline_prompt = "\n"
        newline_token_id = tokenizer.encode(newline_prompt)[-1]  # 108
        # Force to add newline at the end of prompt for paligemma's format
        # This step can NOT be replacemented by current PromptUpdate methods
        if len(prompt_token_ids) and prompt_token_ids[-1] != newline_token_id:
            prompt_token_ids.append(newline_token_id)
            mm_inputs["prompt_token_ids"] = prompt_token_ids

        return mm_inputs
```
**EN:** Defines `PaliGemmaMultiModalProcessor`, a supporting module used by the surrounding model implementation. It inherits from BaseMultiModalProcessor[PaliGemmaProcessingInfo]. Key methods such as `_call_hf_processor`, `_get_mm_fields_config`, `_get_prompt_updates`, `apply` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `PaliGemmaMultiModalProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseMultiModalProcessor[PaliGemmaProcessingInfo]。 `_call_hf_processor`, `_get_mm_fields_config`, `_get_prompt_updates`, `apply` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `PaliGemmaForConditionalGeneration` (lines 249-423)
```python
@MULTIMODAL_REGISTRY.register_processor(
    PaliGemmaMultiModalProcessor,
    info=PaliGemmaProcessingInfo,
    dummy_inputs=PaliGemmaDummyInputsBuilder,
)
class PaliGemmaForConditionalGeneration(
    nn.Module, SupportsLoRA, SupportsMultiModal, SupportsPP
):
    packed_modules_mapping = {
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = config
        self.multimodal_config = multimodal_config
        self.quant_config = quant_config

        with self._mark_tower_model(vllm_config, "image"):
            self.vision_tower = SiglipVisionModel(
                config.vision_config,
                quant_config,
                prefix=maybe_prefix(prefix, "vision_tower"),
            )
            self.multi_modal_projector = PaliGemmaMultiModalProjector(
                vision_hidden_size=config.vision_config.hidden_size,
                projection_dim=config.vision_config.projection_dim,
            )
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> IntermediateTensors:
        if intermediate_tensors is not None:
            inputs_embeds = None

        hidden_states = self.language_model.model(
            input_ids, positions, intermediate_tensors, inputs_embeds=inputs_embeds
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
        return loader.load_weights(weights, mapper=self.hf_to_vllm_mapper)
```
**EN:** Defines `PaliGemmaForConditionalGeneration`, a supporting module used by the surrounding model implementation. It inherits from nn.Module, SupportsLoRA, SupportsMultiModal. Key methods such as `get_placeholder_str`, `__init__`, `_parse_and_validate_image_input`, `_image_pixels_to_features`, `_process_image_input` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `PaliGemmaForConditionalGeneration`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module、SupportsLoRA、SupportsMultiModal。 `get_placeholder_str`, `__init__`, `_parse_and_validate_image_input`, `_image_pixels_to_features`, `_process_image_input` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **Standard library**: collections.abc, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.logger, vllm.multimodal, vllm.multimodal.inputs, vllm.multimodal.parse, vllm.multimodal.processing
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .module_mapping, .siglip, .utils, .vision
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
