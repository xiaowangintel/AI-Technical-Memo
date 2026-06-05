# bagel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/bagel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for bagel, including architecture wrappers and weight loading logic. / 面向推理的 bagel vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 10-55)
```python
from collections.abc import Iterable, Mapping, Sequence
from typing import Any, Literal, TypeAlias

import torch
import torch.nn as nn

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.logger import init_logger
from vllm.model_executor.layers.activation import get_act_fn
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import MultiModalDataItems
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
)
from vllm.sequence import IntermediateTensors
from vllm.transformers_utils.processors.bagel import BagelProcessor
from vllm.utils.tensor_schema import TensorSchema

from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from .siglip import SiglipVisionModel
from .utils import (
    AutoWeightsLoader,
    StageMissingLayer,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `BagelImagePixelInputs` (lines 60-70)
```python
class BagelImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - c: Number of channels (3)
        - h: Height of each image
        - w: Width of each image
    """

    type: Literal["pixel_values"]
    pixel_values: torch.Tensor  # Shape: (bn, 3, h, w)
```
**EN:** Class `BagelImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `BagelImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `BagelVisionMLP` (lines 76-109)
```python
class BagelVisionMLP(nn.Module):
    """MLP connector for vision features."""

    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        out_features: int,
        act_layer: str = "gelu_pytorch_tanh",
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.fc1 = ColumnParallelLinear(
            in_features,
            hidden_features,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
        )
        self.act = get_act_fn(act_layer)
        self.fc2 = RowParallelLinear(
            hidden_features,
            out_features,
            bias=True,
```
**EN:** Class `BagelVisionMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BagelVisionMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BagelVisionMLP.__init__` (lines 79-103)
```python
    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        out_features: int,
        act_layer: str = "gelu_pytorch_tanh",
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.fc1 = ColumnParallelLinear(
            in_features,
            hidden_features,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
        )
        self.act = get_act_fn(act_layer)
        self.fc2 = RowParallelLinear(
            hidden_features,
            out_features,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.fc2",
        )
```
**EN:** Method `BagelVisionMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BagelVisionMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BagelVisionMLP.forward` (lines 105-109)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x, _ = self.fc1(x)
        x = self.act(x)
        x, _ = self.fc2(x)
        return x
```
**EN:** Method `BagelVisionMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BagelVisionMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `PositionEmbedding` (lines 112-187)
```python
class PositionEmbedding(nn.Module):
    """2D position embedding for vision tokens using sin-cos embeddings."""

    def __init__(self, max_num_patch_per_side: int, hidden_size: int):
        super().__init__()
        self.max_num_patch_per_side = max_num_patch_per_side
        self.hidden_size = hidden_size

        # Create learnable 2D position embeddings (frozen sin-cos)
        pos_embed = self._get_2d_sincos_pos_embed(hidden_size, max_num_patch_per_side)
        self.register_buffer(
            "pos_embed",
            torch.from_numpy(pos_embed).float(),
            persistent=False,
        )

    @staticmethod
    def _get_2d_sincos_pos_embed(embed_dim: int, grid_size: int):
        """Generate 2D sin-cos position embeddings."""
        import numpy as np

        grid_h = np.arange(grid_size, dtype=np.float32)
        grid_w = np.arange(grid_size, dtype=np.float32)
        grid = np.meshgrid(grid_w, grid_h)  # w goes first
        grid = np.stack(grid, axis=0)
```
**EN:** Class `PositionEmbedding` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, _get_2d_sincos_pos_embed, _get_2d_sincos_pos_embed_from_grid, _get_1d_sincos_pos_embed_from_grid, forward.
**CN:** 类 `PositionEmbedding` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, _get_2d_sincos_pos_embed, _get_2d_sincos_pos_embed_from_grid, _get_1d_sincos_pos_embed_from_grid, forward。

### Method `PositionEmbedding.__init__` (lines 115-126)
```python
    def __init__(self, max_num_patch_per_side: int, hidden_size: int):
        super().__init__()
        self.max_num_patch_per_side = max_num_patch_per_side
        self.hidden_size = hidden_size

        # Create learnable 2D position embeddings (frozen sin-cos)
        pos_embed = self._get_2d_sincos_pos_embed(hidden_size, max_num_patch_per_side)
        self.register_buffer(
            "pos_embed",
            torch.from_numpy(pos_embed).float(),
            persistent=False,
        )
```
**EN:** Method `PositionEmbedding.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `PositionEmbedding.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `PositionEmbedding.forward` (lines 177-187)
```python
    def forward(self, position_ids: torch.Tensor) -> torch.Tensor:
        """
        Args:
            position_ids: Flattened position IDs, shape (N,) where each ID
                         corresponds to a position in the flattened grid
        Returns:
            Position embeddings of shape (N, hidden_size)
        """
        # Ensure position_ids are on the same device as pos_embed
        position_ids = position_ids.to(self.pos_embed.device)
        return self.pos_embed[position_ids]
```
**EN:** Method `PositionEmbedding.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Args: position_ids: Flattened position IDs, shape (N,) where each ID corresponds to a position in the flattened grid Returns: Position embeddings of shape (N, hidden_size).
**CN:** Method `PositionEmbedding.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Args: position_ids: Flattened position IDs, shape (N,) where each ID corresponds to a position in the flattened grid Returns: Position embeddings of shape (N, hidden_size)。

### Class `BagelProcessingInfo` (lines 190-237)
```python
class BagelProcessingInfo(BaseProcessingInfo):
    """Processing information for BAGEL model."""

    def get_hf_processor(self, **kwargs: object) -> BagelProcessor:
        from vllm.transformers_utils.processor import cached_get_image_processor

        image_processor = cached_get_image_processor(
            self.ctx.model_config.model,
            revision=self.ctx.model_config.revision,
            trust_remote_code=self.ctx.model_config.trust_remote_code,
        )

        tokenizer = self.get_tokenizer()

        return BagelProcessor(
            image_processor=image_processor,
            tokenizer=tokenizer,
            **kwargs,
        )

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_mm_max_tokens_per_item(
        self,
```
**EN:** Class `BagelProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_processor, get_supported_mm_limits, get_mm_max_tokens_per_item, get_num_image_tokens.
**CN:** 类 `BagelProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_processor, get_supported_mm_limits, get_mm_max_tokens_per_item, get_num_image_tokens。

### Method `BagelProcessingInfo.get_hf_processor` (lines 193-208)
```python
    def get_hf_processor(self, **kwargs: object) -> BagelProcessor:
        from vllm.transformers_utils.processor import cached_get_image_processor

        image_processor = cached_get_image_processor(
            self.ctx.model_config.model,
            revision=self.ctx.model_config.revision,
            trust_remote_code=self.ctx.model_config.trust_remote_code,
        )

        tokenizer = self.get_tokenizer()

        return BagelProcessor(
            image_processor=image_processor,
            tokenizer=tokenizer,
            **kwargs,
        )
```
**EN:** Method `BagelProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BagelProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `BagelProcessingInfo.get_supported_mm_limits` (lines 210-211)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Method `BagelProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BagelProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Method `BagelProcessingInfo.get_mm_max_tokens_per_item` (lines 213-222)
```python
    def get_mm_max_tokens_per_item(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
    ) -> Mapping[str, int]:
        hf_config = self.get_hf_config()
        # Calculate max tokens per image
        # For BAGEL: (vit_max_num_patch_per_side) ** 2
        max_num_patches = hf_config.vit_max_num_patch_per_side**2
        return {"image": max_num_patches}
```
**EN:** Method `BagelProcessingInfo.get_mm_max_tokens_per_item` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BagelProcessingInfo.get_mm_max_tokens_per_item` 封装了该模块中的一段可复用核心逻辑。

### Method `BagelProcessingInfo.get_num_image_tokens` (lines 224-237)
```python
    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        hf_config = self.get_hf_config()
        vit_config = hf_config.vit_config
        patch_size = vit_config.patch_size

        # Calculate number of patches
        num_patches_h = image_height // patch_size
        num_patches_w = image_width // patch_size
        return num_patches_h * num_patches_w
```
**EN:** Method `BagelProcessingInfo.get_num_image_tokens` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BagelProcessingInfo.get_num_image_tokens` 封装了该模块中的一段可复用核心逻辑。

### Class `BagelDummyInputsBuilder` (lines 240-269)
```python
class BagelDummyInputsBuilder(BaseDummyInputsBuilder[BagelProcessingInfo]):
    """Build dummy inputs for BAGEL model profiling."""

    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        # Use a simple placeholder for each image
        return "<|image_pad|>" * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        hf_config = self.info.get_hf_config()
        vit_config = hf_config.vit_config

        # Use the configured image size
        image_size = vit_config.image_size
        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=image_size,
```
**EN:** Class `BagelDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[BagelProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `BagelDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[BagelProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `BagelDummyInputsBuilder.get_dummy_text` (lines 243-246)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        # Use a simple placeholder for each image
        return "<|image_pad|>" * num_images
```
**EN:** Method `BagelDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BagelDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `BagelDummyInputsBuilder.get_dummy_mm_data` (lines 248-269)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        hf_config = self.info.get_hf_config()
        vit_config = hf_config.vit_config

        # Use the configured image size
        image_size = vit_config.image_size
        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=image_size,
                height=image_size,
                num_images=num_images,
                overrides=image_overrides,
            ),
        }
```
**EN:** Method `BagelDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BagelDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `BagelMultiModalProcessor` (lines 272-322)
```python
class BagelMultiModalProcessor(BaseMultiModalProcessor[BagelProcessingInfo]):
    """Multimodal processor for BAGEL model."""

    def _hf_processor_applies_updates(
        self,
        prompt_text: str,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        tokenization_kwargs: Mapping[str, object],
    ) -> bool:
        return False

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, Any],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptReplacement]:
        """Replace image placeholders with the correct number of tokens."""
        hf_config = self.info.get_hf_config()

        # Get the tokenizer to look up the image token ID
        tokenizer = self.info.get_tokenizer()
        image_token_id = tokenizer.get_vocab().get("<|image_pad|>")
        if image_token_id is None:
```
**EN:** Class `BagelMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[BagelProcessingInfo]. Key methods include _hf_processor_applies_updates, _get_prompt_updates, _get_mm_fields_config.
**CN:** 类 `BagelMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[BagelProcessingInfo]。 关键方法包括 _hf_processor_applies_updates, _get_prompt_updates, _get_mm_fields_config。

### Class `BagelForConditionalGeneration` (lines 330-584)
```python
@MULTIMODAL_REGISTRY.register_processor(
    BagelMultiModalProcessor,
    info=BagelProcessingInfo,
    dummy_inputs=BagelDummyInputsBuilder,
)
class BagelForConditionalGeneration(
    nn.Module, SupportsMultiModal, SupportsLoRA, SupportsPP
):
    """
    BAGEL: A unified multimodal model for image understanding and generation.

    For vLLM, we focus on the image understanding (vision-to-text) capabilities.
    The image generation part is not supported in vLLM.
    """

    # Weight mapping from HF to vLLM
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            "language_model.": "language_model.",
            "vit_model.": "vit_model.",
            "connector.": "connector.",
            "vit_pos_embed.": "vit_pos_embed.",
        }
    )
```
**EN:** Class `BagelForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsLoRA, SupportsPP. Key methods include get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_input, embed_multimodal, forward.
**CN:** 类 `BagelForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsLoRA、SupportsPP。 关键方法包括 get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_input, embed_multimodal, forward。

### Method `BagelForConditionalGeneration.get_placeholder_str` (lines 351-355)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<|image_pad|>"

        raise ValueError("Only image modality is supported")
```
**EN:** Method `BagelForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `BagelForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `BagelForConditionalGeneration.__init__` (lines 357-435)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config

        # Ensure we have a BagelConfig (check by name to handle trust_remote_code)
        # When trust_remote_code=True, the config comes from transformers_modules
        if type(config).__name__ != "BagelConfig":
            raise ValueError(
                f"Expected BagelConfig, got {type(config).__name__}. "
                "Make sure the model config is properly loaded."
            )

        self.config = config
        self.multimodal_config = multimodal_config

        # Initialize language model (Qwen2)
        # Pass the llm_config from BagelConfig to initialize Qwen2 properly
        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=config.llm_config,
                prefix=maybe_prefix(prefix, "language_model"),
                architectures=["Qwen2ForCausalLM"],
            )

        # Initialize vision model (SigLIP) if visual understanding is enabled
        if config.visual_und:
            # Fix vit_config: checkpoint has 26 layers (0-25) but config says 27
            # Also disable head as it's not in checkpoint
            vit_config = config.vit_config
            if vit_config.num_hidden_layers == 27:
                logger.warning(
                    "Overriding vit_config.num_hidden_layers from 27 to 26 "
                    "to match the Bagel model checkpoint."
                )
                vit_config.num_hidden_layers = 26
            if not hasattr(vit_config, "vision_use_head"):
                logger.warning(
                    "Setting vit_config.vision_use_head to False as it is not "
                    "present in the Bagel model checkpoint."
                )
                vit_config.vision_use_head = False

            with self._mark_tower_model(vllm_config, "image"):
                self.vit_model = SiglipVisionModel(
                    config=vit_config,
                    quant_config=quant_config,
                    prefix=maybe_prefix(prefix, "vit_model"),
                )

                # Initialize connector (MLP)
                vit_hidden_size = config.vit_config.hidden_size
                llm_hidden_size = config.llm_config.hidden_size

                self.connector = BagelVisionMLP(
                    in_features=vit_hidden_size,
                    hidden_features=llm_hidden_size,
# ... truncated for analysis ...
```
**EN:** Method `BagelForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BagelForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BagelForConditionalGeneration.embed_multimodal` (lines 500-506)
```python
    def embed_multimodal(self, **kwargs: object) -> MultiModalEmbeddings:
        """Get multimodal embeddings from input."""
        image_input = self._parse_and_validate_image_input(**kwargs)
        if image_input is None:
            return []

        return self._process_image_input(image_input)
```
**EN:** Method `BagelForConditionalGeneration.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline. The docstring says: Get multimodal embeddings from input.
**CN:** Method `BagelForConditionalGeneration.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。 文档字符串表达的核心意思是：Get multimodal embeddings from input。

### Method `BagelForConditionalGeneration.forward` (lines 508-533)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        """Run forward pass for BAGEL.

        Args:
            input_ids: Flattened (concatenated) input_ids corresponding to a batch.
            positions: Flattened (concatenated) position ids corresponding to a batch.
            intermediate_tensors: Intermediate tensors from prior forward pass.
            inputs_embeds: Optional tensor of input embeddings.
        """
        if intermediate_tensors is not None:
            inputs_embeds = None

        hidden_states = self.language_model.model(
            input_ids=input_ids,
            positions=positions,
            intermediate_tensors=intermediate_tensors,
            inputs_embeds=inputs_embeds,
        )
        return hidden_states
```
**EN:** Method `BagelForConditionalGeneration.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Run forward pass for BAGEL.
**CN:** Method `BagelForConditionalGeneration.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Run forward pass for BAGEL。

## Key Concepts / 关键概念
- **LoRA support / LoRA 支持**
  - **EN:** The file is aware of low-rank adaptation interfaces or compatible parameter layouts.
  - **CN:** 该文件考虑了 LoRA 接口或兼容的低秩参数布局。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。
- **KV cache / KV 缓存**
  - **EN:** The runtime stores key/value state so decoding can reuse prior computation efficiently.
  - **CN:** 运行时会保存 key/value 状态，以便解码阶段高效复用历史计算。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Any, Literal, TypeAlias`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `import numpy as np`, `import numpy as np`, `import numpy as np`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.activation import get_act_fn`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import MultiModalDataItems`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`
- **Module note / 模块说明**: **EN:** Inference-only BAGEL model compatible with HuggingFace weights.  BAGEL is a unified multimodal model for image understanding and generation. For vLLM, we focus on the image understanding (vision-to-text) capabilities. **CN:** 模块文档字符串给出的原始说明是：Inference-only BAGEL model compatible with HuggingFace weights.  BAGEL is a unified multimodal model for image understanding and generation. For vLLM, we focus on the image understanding (vision-to-text) capabilities.。
