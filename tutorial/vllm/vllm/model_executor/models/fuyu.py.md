# fuyu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/fuyu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for fuyu, including architecture wrappers and weight loading logic. / 面向推理的 fuyu vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 21-52)
```python
import math
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Literal

import torch
import torch.nn as nn
from transformers import BatchFeature, FuyuConfig, FuyuImageProcessor, FuyuProcessor

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.linear import ColumnParallelLinear
from vllm.model_executor.models.persimmon import PersimmonForCausalLM
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import ImageProcessorItems, ImageSize, MultiModalDataItems
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
    PromptUpdateDetails,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .utils import AutoWeightsLoader, WeightsMapper, flatten_bn, maybe_prefix
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 55-55)
```python
_IMAGE_TOKEN_ID = 71011
```
**EN:** This block defines _IMAGE_TOKEN_ID, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _IMAGE_TOKEN_ID，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 56-56)
```python
_NEWLINE_TOKEN_ID = 71019
```
**EN:** This block defines _NEWLINE_TOKEN_ID, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _NEWLINE_TOKEN_ID，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Class `FuyuImagePatchInputs` (lines 59-77)
```python
class FuyuImagePatchInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - bnp: Batch size * number of images * number of patches
        - fn: patch_size_x * patch_size_y * num_channels
    """

    type: Literal["image_patches"] = "image_patches"

    image_patches_flat: Annotated[torch.Tensor, TensorShape("bnp", "fn")]

    patches_per_image: Annotated[list[int], TensorShape("bn")]
    """
    The number of total patches for each image in the batch.
    
    This is used to split the embeddings which has the first two dimensions
    flattened just like `image_patches_flat`.
    """
```
**EN:** Class `FuyuImagePatchInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `FuyuImagePatchInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `FuyuProcessingInfo` (lines 80-134)
```python
class FuyuProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(FuyuConfig)

    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(FuyuProcessor, **kwargs)

    def get_image_processor(self, **kwargs: object) -> FuyuImageProcessor:
        return self.get_hf_processor(**kwargs).image_processor

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": 1}

    def get_image_feature_grid_size(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> tuple[int, int]:
        image_processor = self.get_image_processor()
        target_width = image_processor.size["width"]
        target_height = image_processor.size["height"]
        patch_width = image_processor.patch_size["width"]
        patch_height = image_processor.patch_size["height"]
```
**EN:** Class `FuyuProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_hf_processor, get_image_processor, get_supported_mm_limits, get_image_feature_grid_size, get_num_image_tokens.
**CN:** 类 `FuyuProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor, get_image_processor, get_supported_mm_limits, get_image_feature_grid_size, get_num_image_tokens。

### Method `FuyuProcessingInfo.get_hf_config` (lines 81-82)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config(FuyuConfig)
```
**EN:** Method `FuyuProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FuyuProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `FuyuProcessingInfo.get_hf_processor` (lines 84-85)
```python
    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(FuyuProcessor, **kwargs)
```
**EN:** Method `FuyuProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FuyuProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `FuyuProcessingInfo.get_image_processor` (lines 87-88)
```python
    def get_image_processor(self, **kwargs: object) -> FuyuImageProcessor:
        return self.get_hf_processor(**kwargs).image_processor
```
**EN:** Method `FuyuProcessingInfo.get_image_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FuyuProcessingInfo.get_image_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `FuyuProcessingInfo.get_supported_mm_limits` (lines 90-91)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": 1}
```
**EN:** Method `FuyuProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FuyuProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `FuyuDummyInputsBuilder` (lines 137-159)
```python
class FuyuDummyInputsBuilder(BaseDummyInputsBuilder[FuyuProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        return ""

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
**EN:** Class `FuyuDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[FuyuProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `FuyuDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[FuyuProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `FuyuDummyInputsBuilder.get_dummy_text` (lines 138-139)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        return ""
```
**EN:** Method `FuyuDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FuyuDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `FuyuDummyInputsBuilder.get_dummy_mm_data` (lines 141-159)
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
**EN:** Method `FuyuDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FuyuDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `FuyuMultiModalProcessor` (lines 162-254)
```python
class FuyuMultiModalProcessor(BaseMultiModalProcessor[FuyuProcessingInfo]):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        if not mm_data:
            # Avoid warning from HF logger for text-only input
            prompt_ids = self.info.get_tokenizer().encode(prompt)
            prompt_ids = self._apply_hf_processor_tokens_only(prompt_ids)
            return BatchFeature(dict(input_ids=[prompt_ids]), tensor_type="pt")

        processed_outputs = super()._call_hf_processor(
            prompt=prompt,
            mm_data=mm_data,
            mm_kwargs=mm_kwargs,
            tok_kwargs=tok_kwargs,
        )

        image_patches = processed_outputs["image_patches"]
        processed_outputs["image_patches"] = flatten_bn(image_patches)
        processed_outputs["patches_per_image"] = torch.tensor(
            [len(p) for p in image_patches]
```
**EN:** Class `FuyuMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[FuyuProcessingInfo]. Key methods include _call_hf_processor, _apply_hf_processor_tokens_only, _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `FuyuMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[FuyuProcessingInfo]。 关键方法包括 _call_hf_processor, _apply_hf_processor_tokens_only, _get_mm_fields_config, _get_prompt_updates。

### Class `FuyuForCausalLM` (lines 262-368)
```python
@MULTIMODAL_REGISTRY.register_processor(
    FuyuMultiModalProcessor,
    info=FuyuProcessingInfo,
    dummy_inputs=FuyuDummyInputsBuilder,
)
class FuyuForCausalLM(nn.Module, SupportsMultiModal, SupportsPP):
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            "model.vision_embed_tokens.": "vision_embed_tokens.",
            "model.language_model.": "language_model.model.",
            "lm_head.": "language_model.lm_head.",
        }
    )

    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return None

        raise ValueError("Only image modality is supported")

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
```
**EN:** Class `FuyuForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods include get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_input, embed_multimodal, forward.
**CN:** 类 `FuyuForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 关键方法包括 get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_input, embed_multimodal, forward。

### Method `FuyuForCausalLM.get_placeholder_str` (lines 272-276)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return None

        raise ValueError("Only image modality is supported")
```
**EN:** Method `FuyuForCausalLM.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FuyuForCausalLM.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `FuyuForCausalLM.__init__` (lines 278-306)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = config
        self.multimodal_config = multimodal_config

        self.vocab_size = config.text_config.vocab_size
        self.image_token_id = _IMAGE_TOKEN_ID
        self.image_feature_size = config.patch_size**2 * config.num_channels

        with self._mark_tower_model(vllm_config, "image"):
            self.vision_embed_tokens = ColumnParallelLinear(
                self.image_feature_size,
                config.hidden_size,
                quant_config=quant_config,
                gather_output=True,
            )

        with self._mark_language_model(vllm_config):
            self.language_model = PersimmonForCausalLM(
                vllm_config=vllm_config.with_hf_config(config.text_config),
                prefix=maybe_prefix(prefix, "language_model"),
            )

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )
```
**EN:** Method `FuyuForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FuyuForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FuyuForCausalLM.embed_multimodal` (lines 334-339)
```python
    def embed_multimodal(self, **kwargs: object) -> MultiModalEmbeddings:
        image_input = self._parse_and_validate_image_input(**kwargs)
        if image_input is None:
            return []

        return self._process_image_input(image_input)
```
**EN:** Method `FuyuForCausalLM.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `FuyuForCausalLM.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `FuyuForCausalLM.forward` (lines 341-358)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ):
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
**EN:** Method `FuyuForCausalLM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FuyuForCausalLM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Literal`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers import BatchFeature, FuyuConfig, FuyuImageProcessor, FuyuProcessor`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.linear import ColumnParallelLinear`, `from vllm.model_executor.models.persimmon import PersimmonForCausalLM`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import ImageProcessorItems, ImageSize, MultiModalDataItems`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`, `from vllm.utils.tensor_schema import TensorSchema, TensorShape`, `from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP`
- **Module note / 模块说明**: **EN:** PyTorch Fuyu model. **CN:** 模块文档字符串给出的原始说明是：PyTorch Fuyu model.。
