# idefics3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/idefics3.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for idefics3, including architecture wrappers and weight loading logic. / 面向推理的 idefics3 vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 19-65)
```python
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Literal, TypeAlias

import torch
from torch import nn
from transformers import (
    BatchFeature,
    Idefics3Config,
    Idefics3ImageProcessor,
    Idefics3Processor,
)

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.linear import ReplicatedLinear
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import ImageProcessorItems, MultiModalDataItems
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

from .idefics2_vision_model import (
    Idefics2VisionTransformer as Idefics3VisionTransformer,
)
from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
)
from .llama import LlamaModel
from .utils import AutoWeightsLoader, maybe_prefix
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `Idefics3ImagePixelInputs` (lines 68-81)
```python
class Idefics3ImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - bnp: Batch size * number of images * number of patches
        - c: Number of channels (3)
        - h: Height
        - w: Width
    """

    type: Literal["pixel_values"]
    pixel_values: Annotated[torch.Tensor, TensorShape("bnp", 3, "h", "w")]
    pixel_attention_mask: Annotated[torch.Tensor, TensorShape("bnp", "h", "w")]
    num_patches: Annotated[torch.Tensor, TensorShape("bn")]
```
**EN:** Class `Idefics3ImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Idefics3ImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Idefics3ImageEmbeddingInputs` (lines 84-93)
```python
class Idefics3ImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - f: Image feature size
        - h: Hidden size (must match the hidden size of language model backbone)
    """

    type: Literal["image_embeds"]
    data: Annotated[torch.Tensor, TensorShape("bn", "f", "h")]
```
**EN:** Class `Idefics3ImageEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Idefics3ImageEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Idefics3ProcessingInfo` (lines 99-264)
```python
class Idefics3ProcessingInfo(BaseProcessingInfo):
    def get_hf_processor(self, **kwargs: object) -> Idefics3Processor:
        return self.ctx.get_hf_processor(Idefics3Processor, **kwargs)

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def _resize_output_size(
        self,
        *,
        height: int,
        width: int,
        max_len: int | None = None,
        min_len: int = 1,
        max_size: int | None = None,
    ) -> tuple[int, int]:
        # Set default value for max_len if not provided
        max_len = max(height, width) if max_len is None else max_len
        aspect_ratio = width / height

        # Handle the maximum size constraint
        if max_size is not None:
            max_len = min(max_len, max_size)

        # Adjust dimensions according to the aspect ratio
```
**EN:** Class `Idefics3ProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_processor, get_supported_mm_limits, _resize_output_size, _get_resize_output_image_size, _get_image_feature_grid_size, get_num_patches.
**CN:** 类 `Idefics3ProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_processor, get_supported_mm_limits, _resize_output_size, _get_resize_output_image_size, _get_image_feature_grid_size, get_num_patches。

### Method `Idefics3ProcessingInfo.get_hf_processor` (lines 100-101)
```python
    def get_hf_processor(self, **kwargs: object) -> Idefics3Processor:
        return self.ctx.get_hf_processor(Idefics3Processor, **kwargs)
```
**EN:** Method `Idefics3ProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Idefics3ProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `Idefics3ProcessingInfo.get_supported_mm_limits` (lines 103-104)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Method `Idefics3ProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Idefics3ProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `Idefics3DummyInputsBuilder` (lines 267-296)
```python
class Idefics3DummyInputsBuilder(BaseDummyInputsBuilder[Idefics3ProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token, _, _ = self.info._get_image_token(processor)

        return image_token * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        hf_processor = self.info.get_hf_processor()
        image_processor: Idefics3ImageProcessor = hf_processor.image_processor
        longest_edge = image_processor.max_image_size["longest_edge"]

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=longest_edge,
```
**EN:** Class `Idefics3DummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[Idefics3ProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `Idefics3DummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[Idefics3ProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `Idefics3DummyInputsBuilder.get_dummy_text` (lines 268-274)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token, _, _ = self.info._get_image_token(processor)

        return image_token * num_images
```
**EN:** Method `Idefics3DummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Idefics3DummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `Idefics3DummyInputsBuilder.get_dummy_mm_data` (lines 276-296)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        hf_processor = self.info.get_hf_processor()
        image_processor: Idefics3ImageProcessor = hf_processor.image_processor
        longest_edge = image_processor.max_image_size["longest_edge"]

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=longest_edge,
                height=longest_edge,
                num_images=num_images,
                overrides=image_overrides,
            )
        }
```
**EN:** Method `Idefics3DummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Idefics3DummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `Idefics3MultiModalProcessor` (lines 299-393)
```python
class Idefics3MultiModalProcessor(BaseMultiModalProcessor[Idefics3ProcessingInfo]):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        # Text-only input not supported in composite processor
        if not (images := mm_data.get("images", [])):
            prompt_ids = self.info.get_tokenizer().encode(prompt)
            prompt_ids = self._apply_hf_processor_tokens_only(prompt_ids)
            return BatchFeature(dict(input_ids=[prompt_ids]), tensor_type="pt")

        mm_kwargs = {"input_data_format": "channels_last", **mm_kwargs}
        processed_outputs = super()._call_hf_processor(
            prompt,
            mm_data,
            mm_kwargs,
            tok_kwargs,
        )

        mm_items = self.info.parse_mm_data({"image": images}, validate=False)
        parsed_images = mm_items.get_items("image", ImageProcessorItems)
        image_sizes = [
```
**EN:** Class `Idefics3MultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[Idefics3ProcessingInfo]. Key methods include _call_hf_processor, _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `Idefics3MultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[Idefics3ProcessingInfo]。 关键方法包括 _call_hf_processor, _get_mm_fields_config, _get_prompt_updates。

### Class `Idefics3SimpleMLP` (lines 396-416)
```python
class Idefics3SimpleMLP(nn.Module):
    def __init__(
        self,
        config: Idefics3Config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        input_size = config.vision_config.hidden_size * (config.scale_factor**2)
        output_size = config.text_config.hidden_size
        self.proj = ReplicatedLinear(
            input_size,
            output_size,
            bias=False,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "proj"),
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        out, _ = self.proj(x)
        return out
```
**EN:** Class `Idefics3SimpleMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Idefics3SimpleMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Idefics3SimpleMLP.__init__` (lines 397-412)
```python
    def __init__(
        self,
        config: Idefics3Config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        input_size = config.vision_config.hidden_size * (config.scale_factor**2)
        output_size = config.text_config.hidden_size
        self.proj = ReplicatedLinear(
            input_size,
            output_size,
            bias=False,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "proj"),
        )
```
**EN:** Method `Idefics3SimpleMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Idefics3SimpleMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Idefics3SimpleMLP.forward` (lines 414-416)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        out, _ = self.proj(x)
        return out
```
**EN:** Method `Idefics3SimpleMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Idefics3SimpleMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Idefics3Connector` (lines 419-453)
```python
class Idefics3Connector(nn.Module):
    def __init__(
        self,
        config: Idefics3Config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.scale_factor = config.scale_factor
        self.modality_projection = Idefics3SimpleMLP(
            config,
            quant_config,
            prefix=maybe_prefix(prefix, "modality_projection"),
        )

    def pixel_shuffle(self, x: torch.Tensor, scale_factor: int = 2) -> torch.Tensor:
        bsz, seq, embed_dim = x.size()
        height = width = int(seq**0.5)
        x = x.view(bsz, height, width, embed_dim)
        x = x.view(bsz, height, int(width / scale_factor), embed_dim * scale_factor)
        x = x.permute(0, 2, 1, 3)
        x = x.reshape(
            bsz,
            int(width / scale_factor),
            int(height / scale_factor),
```
**EN:** Class `Idefics3Connector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, pixel_shuffle, forward.
**CN:** 类 `Idefics3Connector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, pixel_shuffle, forward。

### Method `Idefics3Connector.__init__` (lines 420-432)
```python
    def __init__(
        self,
        config: Idefics3Config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.scale_factor = config.scale_factor
        self.modality_projection = Idefics3SimpleMLP(
            config,
            quant_config,
            prefix=maybe_prefix(prefix, "modality_projection"),
        )
```
**EN:** Method `Idefics3Connector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Idefics3Connector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Idefics3Connector.pixel_shuffle` (lines 434-448)
```python
    def pixel_shuffle(self, x: torch.Tensor, scale_factor: int = 2) -> torch.Tensor:
        bsz, seq, embed_dim = x.size()
        height = width = int(seq**0.5)
        x = x.view(bsz, height, width, embed_dim)
        x = x.view(bsz, height, int(width / scale_factor), embed_dim * scale_factor)
        x = x.permute(0, 2, 1, 3)
        x = x.reshape(
            bsz,
            int(width / scale_factor),
            int(height / scale_factor),
            embed_dim * (scale_factor**2),
        )
        x = x.permute(0, 2, 1, 3)
        x = x.reshape(bsz, int(seq / (scale_factor**2)), embed_dim * (scale_factor**2))
        return x
```
**EN:** Method `Idefics3Connector.pixel_shuffle` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Idefics3Connector.pixel_shuffle` 封装了该模块中的一段可复用核心逻辑。

### Class `Idefics3Model` (lines 456-541)
```python
class Idefics3Model(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config: Idefics3Config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.vocab_size = self.config.text_config.vocab_size
        self.vision_model = Idefics3VisionTransformer(
            config.vision_config,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "vision_model"),
        )
        self.connector = Idefics3Connector(
            config,
            quant_config,
            prefix=maybe_prefix(prefix, "connector"),
        )
        self.text_model = LlamaModel(
            vllm_config=vllm_config.with_hf_config(config.text_config),
            prefix=maybe_prefix(prefix, "text_model"),
        )

        self.image_seq_len = int(
```
**EN:** Class `Idefics3Model` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, image_pixels_to_features, embed_input_ids, forward.
**CN:** 类 `Idefics3Model` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, image_pixels_to_features, embed_input_ids, forward。

### Method `Idefics3Model.__init__` (lines 457-484)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config: Idefics3Config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.vocab_size = self.config.text_config.vocab_size
        self.vision_model = Idefics3VisionTransformer(
            config.vision_config,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "vision_model"),
        )
        self.connector = Idefics3Connector(
            config,
            quant_config,
            prefix=maybe_prefix(prefix, "connector"),
        )
        self.text_model = LlamaModel(
            vllm_config=vllm_config.with_hf_config(config.text_config),
            prefix=maybe_prefix(prefix, "text_model"),
        )

        self.image_seq_len = int(
            ((config.vision_config.image_size // config.vision_config.patch_size) ** 2)
            / (config.scale_factor**2)
        )
        self.image_token_id = self.config.image_token_id
```
**EN:** Method `Idefics3Model.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Idefics3Model.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Idefics3Model.image_pixels_to_features` (lines 486-523)
```python
    def image_pixels_to_features(
        self,
        pixel_values: torch.Tensor,
        pixel_attention_mask: torch.Tensor,
    ) -> torch.Tensor:
        # NOTE: we skip the step to select the vision feature layer since
        # this is already done inside the vision tower
        pixel_values = pixel_values.to(
            dtype=self.vision_model.embeddings.patch_embedding.weight.dtype
        )  # fp16 compatibility

        # Remove padding images - padding images are full 0.
        nb_values_per_image = pixel_values.shape[1:].numel()
        real_images_inds = (pixel_values == 0.0).sum(
            dim=(-1, -2, -3)
        ) != nb_values_per_image
        pixel_values = pixel_values[real_images_inds].contiguous()

        # Handle the vision attention mask
        # Remove padding images from the mask
        pixel_attention_mask = pixel_attention_mask[real_images_inds].contiguous()

        patch_size = self.config.vision_config.patch_size
        patches_subgrid = pixel_attention_mask.unfold(
            dimension=1, size=patch_size, step=patch_size
        )
        patches_subgrid = patches_subgrid.unfold(
            dimension=2, size=patch_size, step=patch_size
        )
        patch_attention_mask = (patches_subgrid.sum(dim=(-1, -2)) > 0).bool()

        # Get sequence from the vision encoder
        image_hidden_states = self.vision_model(
            pixel_values=pixel_values,
            patch_attention_mask=patch_attention_mask,
        )

        return image_hidden_states
```
**EN:** Method `Idefics3Model.image_pixels_to_features` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Idefics3Model.image_pixels_to_features` 封装了该模块中的一段可复用核心逻辑。

### Class `Idefics3ForConditionalGeneration` (lines 549-709)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Idefics3MultiModalProcessor,
    info=Idefics3ProcessingInfo,
    dummy_inputs=Idefics3DummyInputsBuilder,
)
class Idefics3ForConditionalGeneration(nn.Module, SupportsMultiModal, SupportsLoRA):
    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
        "gate_up_proj": [
            "gate_proj",
            "up_proj",
        ],
    }

    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"

        raise ValueError("Only image modality is supported")
```
**EN:** Class `Idefics3ForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsLoRA. Key methods include get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_pixels, _process_image_input, embed_multimodal.
**CN:** 类 `Idefics3ForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsLoRA。 关键方法包括 get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_pixels, _process_image_input, embed_multimodal。

### Method `Idefics3ForConditionalGeneration.get_placeholder_str` (lines 563-567)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"

        raise ValueError("Only image modality is supported")
```
**EN:** Method `Idefics3ForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Idefics3ForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `Idefics3ForConditionalGeneration.__init__` (lines 569-599)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config

        self.config = config
        self.multimodal_config = multimodal_config

        with self._mark_composite_model(
            vllm_config,
            language_targets=LlamaModel,
            tower_targets={"image": (Idefics3VisionTransformer, Idefics3Connector)},
        ):
            self.model = Idefics3Model(
                vllm_config=vllm_config,
                prefix=maybe_prefix(prefix, "model"),
            )

        self.image_token_id = self.config.image_token_id

        self.lm_head = ParallelLMHead(
            config.text_config.vocab_size,
            config.text_config.hidden_size,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "lm_head"),
        )
        if self.config.text_config.tie_word_embeddings:
            self.lm_head.weight = self.model.text_model.embed_tokens.weight
        self.logits_processor = LogitsProcessor(config.text_config.vocab_size)
```
**EN:** Method `Idefics3ForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Idefics3ForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **LoRA support / LoRA 支持**
  - **EN:** The file is aware of low-rank adaptation interfaces or compatible parameter layouts.
  - **CN:** 该文件考虑了 LoRA 接口或兼容的低秩参数布局。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Literal, TypeAlias`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import (`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.linear import ReplicatedLinear`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import ImageProcessorItems, MultiModalDataItems`, `from vllm.multimodal.processing import (`
- **Module note / 模块说明**: **EN:** Inference-only Idefics3 model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only Idefics3 model compatible with HuggingFace weights.。
