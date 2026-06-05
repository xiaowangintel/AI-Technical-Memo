# kimi_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/kimi_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for kimi_vl, including encoder/decoder glue and vLLM runtime adaptation. / 面向 kimi_vl 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 45-84)
```python
import math
from collections.abc import Iterable, Mapping, Sequence
from dataclasses import dataclass
from typing import Annotated, Any, Literal

import torch
from torch import nn
from transformers import BatchFeature
from transformers.activations import GELUActivation

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.linear import ReplicatedLinear
from vllm.model_executor.models.interfaces import SupportsMultiModal, SupportsPP
from vllm.model_executor.models.moonvit import MoonVitPretrainedModel
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
    NestedTensors,
)
from vllm.multimodal.parse import (
    ImageEmbeddingItems,
    ImageProcessorItems,
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
from vllm.transformers_utils.configs.kimi_vl import KimiVLConfig, MoonViTConfig
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .utils import AutoWeightsLoader, init_vllm_registered_model, maybe_prefix
from .vision import is_vit_use_data_parallel, run_dp_sharded_mrope_vision_model
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `MaxImageTokenMeta` (lines 89-91)
```python
@dataclass
class MaxImageTokenMeta:
    width: int = 1024
    height: int = 1024
```
**EN:** Class `MaxImageTokenMeta` organizes related behavior for this model family or helper component.
**CN:** 类 `MaxImageTokenMeta` 用于组织该模型族或辅助组件的相关行为。

### Class `KimiVLMultiModalProjector` (lines 94-129)
```python
class KimiVLMultiModalProjector(nn.Module):
    def __init__(
        self,
        config: KimiVLConfig,
        prefix: str = "",
    ):
        super().__init__()
        self.use_data_parallel = is_vit_use_data_parallel()

        self.hidden_size = (
            config.vision_config.hidden_size
            * config.vision_config.merge_kernel_size[0]
            * config.vision_config.merge_kernel_size[1]
        )

        self.pre_norm = torch.nn.LayerNorm(config.vision_config.hidden_size, eps=1e-5)
        self.linear_1 = ReplicatedLinear(
            self.hidden_size,
            self.hidden_size,
            bias=True,
            prefix=maybe_prefix(prefix, "linear_1"),
        )
        self.linear_2 = ReplicatedLinear(
            self.hidden_size,
            config.text_config.hidden_size,
```
**EN:** Class `KimiVLMultiModalProjector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `KimiVLMultiModalProjector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `KimiVLMultiModalProjector.__init__` (lines 95-122)
```python
    def __init__(
        self,
        config: KimiVLConfig,
        prefix: str = "",
    ):
        super().__init__()
        self.use_data_parallel = is_vit_use_data_parallel()

        self.hidden_size = (
            config.vision_config.hidden_size
            * config.vision_config.merge_kernel_size[0]
            * config.vision_config.merge_kernel_size[1]
        )

        self.pre_norm = torch.nn.LayerNorm(config.vision_config.hidden_size, eps=1e-5)
        self.linear_1 = ReplicatedLinear(
            self.hidden_size,
            self.hidden_size,
            bias=True,
            prefix=maybe_prefix(prefix, "linear_1"),
        )
        self.linear_2 = ReplicatedLinear(
            self.hidden_size,
            config.text_config.hidden_size,
            bias=True,
            prefix=maybe_prefix(prefix, "linear_2"),
        )
        self.act = GELUActivation()
```
**EN:** Method `KimiVLMultiModalProjector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KimiVLMultiModalProjector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KimiVLMultiModalProjector.forward` (lines 124-129)
```python
    def forward(self, image_features: torch.Tensor) -> torch.Tensor:
        hidden_states = self.pre_norm(image_features).view(-1, self.hidden_size)
        hidden_states, _ = self.linear_1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** Method `KimiVLMultiModalProjector.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `KimiVLMultiModalProjector.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `KimiVLImagePixelInputs` (lines 132-148)
```python
class KimiVLImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - nc: Number of channels
        - np: Number of patches
        - ps: Patch size
        - ni: Number of images
    """

    type: Literal["pixel_values"] = "pixel_values"

    pixel_values: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("np", 3, "ps", "ps"),
    ]

    image_grid_hws: Annotated[torch.Tensor, TensorShape("ni", 2)]
```
**EN:** Class `KimiVLImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `KimiVLImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `KimiVLProcessingInfo` (lines 156-202)
```python
class KimiVLProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(KimiVLConfig)

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        hf_processor = self.get_hf_processor()
        patch_size = hf_processor.image_processor.patch_size
        kernel_size = hf_processor.image_processor.merge_kernel_size
        in_token_limit = hf_processor.image_processor.in_token_limit
        height = image_height
        width = image_width
        assert isinstance(height, int), f"height must be int, current height {height}"
        assert isinstance(width, int), f"width must be int, current width {width}"
        assert kernel_size is not None, "kernel_size must be specified"

        if (width // patch_size) * (height // patch_size) > in_token_limit:
            scale = math.sqrt(
```
**EN:** Class `KimiVLProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_supported_mm_limits, get_num_image_tokens, image_token_id.
**CN:** 类 `KimiVLProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_supported_mm_limits, get_num_image_tokens, image_token_id。

### Method `KimiVLProcessingInfo.get_hf_config` (lines 157-158)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config(KimiVLConfig)
```
**EN:** Method `KimiVLProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiVLProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiVLProcessingInfo.get_supported_mm_limits` (lines 160-161)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Method `KimiVLProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiVLProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiVLProcessingInfo.get_num_image_tokens` (lines 163-198)
```python
    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        hf_processor = self.get_hf_processor()
        patch_size = hf_processor.image_processor.patch_size
        kernel_size = hf_processor.image_processor.merge_kernel_size
        in_token_limit = hf_processor.image_processor.in_token_limit
        height = image_height
        width = image_width
        assert isinstance(height, int), f"height must be int, current height {height}"
        assert isinstance(width, int), f"width must be int, current width {width}"
        assert kernel_size is not None, "kernel_size must be specified"

        if (width // patch_size) * (height // patch_size) > in_token_limit:
            scale = math.sqrt(
                in_token_limit / ((width // patch_size) * (height // patch_size))
            )
            new_w, new_h = int(width * scale), int(height * scale)
            width, height = new_w, new_h

        kernel_height, kernel_width = kernel_size

        pad_height = (
            kernel_height * patch_size - height % (kernel_height * patch_size)
        ) % (kernel_height * patch_size)
        pad_width = (
            kernel_width * patch_size - width % (kernel_width * patch_size)
        ) % (kernel_width * patch_size)

        # Calculate new dimensions after padding and patching
        token_height = (height + pad_height) // (kernel_size[0] * patch_size)
        token_width = (width + pad_width) // (kernel_size[1] * patch_size)
        return int(token_height * token_width)
```
**EN:** Method `KimiVLProcessingInfo.get_num_image_tokens` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiVLProcessingInfo.get_num_image_tokens` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiVLProcessingInfo.image_token_id` (lines 201-202)
```python
    @property
    def image_token_id(self) -> int:
        return self.get_hf_config().media_placeholder_token_id
```
**EN:** Method `KimiVLProcessingInfo.image_token_id` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiVLProcessingInfo.image_token_id` 封装了该模块中的一段可复用核心逻辑。

### Class `KimiVLDummyInputsBuilder` (lines 205-231)
```python
class KimiVLDummyInputsBuilder(BaseDummyInputsBuilder[KimiVLProcessingInfo]):
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

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=MaxImageTokenMeta.width,
                height=MaxImageTokenMeta.height,
                num_images=num_images,
                overrides=image_overrides,
```
**EN:** Class `KimiVLDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[KimiVLProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `KimiVLDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[KimiVLProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `KimiVLDummyInputsBuilder.get_dummy_text` (lines 206-212)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.image_token

        return image_token * num_images
```
**EN:** Method `KimiVLDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiVLDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiVLDummyInputsBuilder.get_dummy_mm_data` (lines 214-231)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=MaxImageTokenMeta.width,
                height=MaxImageTokenMeta.height,
                num_images=num_images,
                overrides=image_overrides,
            )
        }
```
**EN:** Method `KimiVLDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiVLDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `KimiVLMultiModalProcessor` (lines 234-282)
```python
class KimiVLMultiModalProcessor(BaseMultiModalProcessor[KimiVLProcessingInfo]):
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        image_grid_hws = hf_inputs.get("image_grid_hws", torch.empty((0, 2)))
        image_grid_sizes = image_grid_hws.prod(-1)

        # pixel_values is merged as a single large tensor
        # image_grid_hws is shapes for each subtensor in pixel_values
        return dict(
            pixel_values=MultiModalFieldConfig.flat_from_sizes(
                "image", image_grid_sizes
            ),
            image_grid_hws=MultiModalFieldConfig.batched("image"),
        )

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, Any],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        image_token_id = self.info.image_token_id
```
**EN:** Class `KimiVLMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[KimiVLProcessingInfo]. Key methods include _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `KimiVLMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[KimiVLProcessingInfo]。 关键方法包括 _get_mm_fields_config, _get_prompt_updates。

### Class `KimiVLForConditionalGeneration` (lines 290-416)
```python
@MULTIMODAL_REGISTRY.register_processor(
    KimiVLMultiModalProcessor,
    info=KimiVLProcessingInfo,
    dummy_inputs=KimiVLDummyInputsBuilder,
)
class KimiVLForConditionalGeneration(nn.Module, SupportsMultiModal, SupportsPP):
    supports_encoder_tp_data = True

    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<|media_start|>image<|media_content|><|media_pad|><|media_end|>"

        raise ValueError("Only image modality is supported")

    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
    ) -> None:
        super().__init__()
        model_config = vllm_config.model_config
        config: KimiVLConfig = model_config.hf_config
        quant_config = vllm_config.quant_config
```
**EN:** Class `KimiVLForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods include get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_pixels, _process_image_input, embed_multimodal.
**CN:** 类 `KimiVLForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 关键方法包括 get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_pixels, _process_image_input, embed_multimodal。

### Method `KimiVLForConditionalGeneration.get_placeholder_str` (lines 294-298)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<|media_start|>image<|media_content|><|media_pad|><|media_end|>"

        raise ValueError("Only image modality is supported")
```
**EN:** Method `KimiVLForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KimiVLForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `KimiVLForConditionalGeneration.__init__` (lines 300-341)
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
    ) -> None:
        super().__init__()
        model_config = vllm_config.model_config
        config: KimiVLConfig = model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config

        assert isinstance(config.vision_config, MoonViTConfig)
        self.use_data_parallel = (
            model_config.multimodal_config.mm_encoder_tp_mode == "data"
        )
        self.hidden_size = config.text_config.hidden_size

        with self._mark_tower_model(vllm_config, "image"):
            self.vision_tower = MoonVitPretrainedModel(
                config.vision_config,
                prefix=maybe_prefix(prefix, "vision_tower"),
            )
            self.multi_modal_projector = KimiVLMultiModalProjector(
                config=config,
                prefix=maybe_prefix(prefix, "multi_modal_projector"),
            )

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=config.text_config,
                prefix=maybe_prefix(prefix, "language_model"),
                architectures=["DeepseekV2ForCausalLM"],
            )

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )

        self.media_placeholder: int = self.config.media_placeholder_token_id
```
**EN:** Method `KimiVLForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KimiVLForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KimiVLForConditionalGeneration.embed_multimodal` (lines 381-389)
```python
    def embed_multimodal(self, **kwargs: object) -> NestedTensors | None:
        # Validate the multimodal input keyword arguments
        image_input = self._parse_and_validate_image_input(**kwargs)
        if image_input is None:
            return None

        # Run multimodal inputs through encoder and projector
        vision_embeddings = self._process_image_input(image_input)
        return vision_embeddings
```
**EN:** Method `KimiVLForConditionalGeneration.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `KimiVLForConditionalGeneration.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `KimiVLForConditionalGeneration.forward` (lines 391-409)
```python
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

        hidden_states = self.language_model(
            input_ids=input_ids,
            positions=positions,
            intermediate_tensors=intermediate_tensors,
            inputs_embeds=inputs_embeds,
        )

        return hidden_states
```
**EN:** Method `KimiVLForConditionalGeneration.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `KimiVLForConditionalGeneration.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable, Mapping, Sequence`, `from dataclasses import dataclass`, `from typing import Annotated, Any, Literal`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import BatchFeature`, `from transformers.activations import GELUActivation`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.linear import ReplicatedLinear`, `from vllm.model_executor.models.interfaces import SupportsMultiModal, SupportsPP`, `from vllm.model_executor.models.moonvit import MoonVitPretrainedModel`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import (`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`, `from vllm.transformers_utils.configs.kimi_vl import KimiVLConfig, MoonViTConfig`
