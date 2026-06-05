# isaac.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/isaac.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for isaac, including architecture wrappers and weight loading logic. / 面向推理的 isaac vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-75)
```python
from __future__ import annotations

from collections.abc import Iterable, Iterator, Mapping, Sequence
from typing import Annotated, Any

import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange
from transformers.image_processing_utils import BatchFeature

from vllm.config import ModelConfig, VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.distributed import parallel_state
from vllm.distributed import utils as dist_utils
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.attention import MMEncoderAttention
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.model_loader.weight_utils import (
    default_weight_loader,
)
from vllm.model_executor.models.interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMRoPE,
    SupportsMultiModal,
    SupportsPP,
)
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.model_executor.models.siglip import SiglipMLP
from vllm.model_executor.models.utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import ImageSize, MultiModalDataItems
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
    PromptUpdateDetails,
)
from vllm.sequence import IntermediateTensors
from vllm.tokenizers import cached_tokenizer_from_config
from vllm.transformers_utils.config import patch_rope_parameters
from vllm.transformers_utils.configs.isaac import (
    IsaacConfig,
    PixelShuffleSiglip2VisionConfig,
)
from vllm.transformers_utils.processors.isaac import (
    IsaacImageProcessor,
    IsaacProcessor,
    get_image_size_for_max_num_patches,
)
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .vision import is_vit_use_data_parallel
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `create_cumulative_seq_lengths` (lines 78-89)
```python
def create_cumulative_seq_lengths(
    seq_sizes: torch.Tensor, device: torch.device
) -> tuple[torch.Tensor, torch.Tensor]:
    """Create cumulative sequence lengths for variable-length attention."""
    cu_seqlens = torch.zeros(len(seq_sizes) + 1, dtype=torch.int32, device=device)
    cu_seqlens[1:] = seq_sizes.cumsum(0)
    max_seqlen = (
        seq_sizes.max()
        if len(seq_sizes) > 0
        else torch.tensor(0, dtype=torch.int32, device=device)
    )
    return cu_seqlens, max_seqlen
```
**EN:** Function `create_cumulative_seq_lengths` encapsulates a focused piece of reusable logic inside this module. The docstring says: Create cumulative sequence lengths for variable-length attention.
**CN:** Function `create_cumulative_seq_lengths` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Create cumulative sequence lengths for variable-length attention。

### Function `create_pixel_shuffle_index_map` (lines 175-238)
```python
def create_pixel_shuffle_index_map(
    seq_sizes: torch.Tensor,
    token_grids: torch.Tensor,
    scale_factor: int = 1,
    device: torch.device | None = None,
) -> torch.Tensor:
    """
    Build a gather-index map that tells us, for every *output* token after
    pixel-shuffle, which `scale_factor**2` *input* tokens are being merged.

    Args
    ----
    seq_sizes     : (num_images,)  - #patches in each image (row-major order)
    token_grids   : (num_images,2) - (height, width) for every image
    scale_factor  : spatial down-scale factor (≥2)
    device        : (optional) overrides `seq_sizes.device`

    Returns
    -------
    gather_idx : (new_total_seq_len, scale_factor**2) int64 tensor.
                 gather_idx[i, j] is the *flat* index into the *original*
                 packed sequence for the j-th sub-patch that forms the
                 i-th output token.
    """
    if device is None:
        device = seq_sizes.device

    r = int(scale_factor)
    if r < 2:
        raise ValueError("`scale_factor` must be ≥ 2")

    # Safety: all spatial dims must be divisible by r
    # Cannot run under torch compile fullgraph mode hence
    if not torch.compiler.is_compiling() and not (
        (token_grids[:, 0] % r == 0).all() and (token_grids[:, 1] % r == 0).all()
    ):
        raise AssertionError(
            "Every (H,W) in `token_grids` must be divisible by "
            f"scale_factor={r}, got {token_grids.tolist()}"
        )

    gather_chunks: list[torch.Tensor] = []
    tok_offset = 0

    for seq_len, (h, w) in zip(seq_sizes.tolist(), token_grids.tolist(), strict=False):
        # Build the (H, W) grid of flat indices for this image
        grid = torch.arange(seq_len, device=device, dtype=torch.int64) + tok_offset
        grid = grid.view(h, w)  # (H, W)

        # -------- identical ordering to your fixed-res routine --------
        # Step 1: split width into blocks of r
        grid = grid.view(h, w // r, r)  # (H, W/r, r)
        # Step 2: now split height into blocks of r
        grid = grid.view(h // r, r, w // r, r)  # (H/r, r, W/r, r)
        # Step 3: final permutation to (H/r, W/r, r, r)
        grid = grid.permute(0, 2, 1, 3).contiguous()  # (H/r, W/r, r, r)
        # Step 4: each (r, r) block forms one output token
        gather_chunks.append(grid.reshape(-1, r * r))  # (H*W / r², r²)

        tok_offset += seq_len
# ... truncated for analysis ...
```
**EN:** Function `create_pixel_shuffle_index_map` encapsulates a focused piece of reusable logic inside this module. The docstring says: Build a gather-index map that tells us, for every *output* token after pixel-shuffle, which `scale_factor**2` *input* tokens are being merged.
**CN:** Function `create_pixel_shuffle_index_map` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Build a gather-index map that tells us, for every *output* token after pixel-shuffle, which `scale_factor**2` *input* tokens are being merged。

### Function `pixel_shuffle_varlen` (lines 241-302)
```python
def pixel_shuffle_varlen(
    x: torch.Tensor,
    token_grids: torch.Tensor,
    scale_factor: int = 1,
) -> torch.Tensor:
    r"""Apply pixel shuffle to a packed vision sequence without unpacking per image.

    Args:
        x (`torch.Tensor`):
            Concatenated vision embeddings. Accepts `(seq_len, hidden_size)` or
            `(1, seq_len, hidden_size)` shapes produced by stacking image
            patches.
        token_grids (`torch.Tensor`):
            Integer tensor of shape `(num_images, 2)` whose rows give the
            `(height, width)` patch grid sizes corresponding to each image
            segment inside `x`.
        scale_factor (`int`, *optional*, defaults to 1):
            Spatial down-sampling factor specific to pixel shuffle. Values
            greater than one merge `scale_factor**2` neighboring patches into a
            single embedding channel-group.

    Returns:
        `torch.Tensor`: Pixel-shuffled embeddings with shape matching the input
        convention: `(seq_len, hidden_size * scale_factor**2)` when the input
        was 2D, or `(1, seq_len, hidden_size * scale_factor**2)` if the
        singleton batch dimension was present.

    Raises:
        ValueError: If more than one batch item is provided.
    """
    keep_batch_dim = x.dim() == 3
    if keep_batch_dim:
        if x.size(0) != 1:
            raise AssertionError("Packed sequence is expected to have batch_size == 1")
        x_ = x.squeeze(0)  # (seq, embed)
    else:
        x_ = x  # (seq, embed)

    embed_dim = x_.size(-1)
    r = int(scale_factor)

    # Calculate seq_sizes from token_grids
    seq_sizes = torch.prod(token_grids, dim=-1)

    # Build index map and gather in one go
    gather_idx = create_pixel_shuffle_index_map(
        seq_sizes=seq_sizes,
        token_grids=token_grids,
        scale_factor=r,
        device=x_.device,
    )  # (new_seq, r²)

    # Gather → (new_seq, r², embed_dim)
    gathered = x_[gather_idx]  # fancy indexing keeps gradient

    # Merge the r² group dimension into channels to finish the shuffle
    out = gathered.reshape(gathered.size(0), embed_dim * r * r)

    # Restore batch dimension if needed
    if keep_batch_dim:
# ... truncated for analysis ...
```
**EN:** Function `pixel_shuffle_varlen` encapsulates a focused piece of reusable logic inside this module. The docstring says: Apply pixel shuffle to a packed vision sequence without unpacking per image.
**CN:** Function `pixel_shuffle_varlen` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Apply pixel shuffle to a packed vision sequence without unpacking per image。

### Class `Siglip2VariableSequenceEmbeddings` (lines 92-172)
```python
class Siglip2VariableSequenceEmbeddings(nn.Module):
    def __init__(self, config: PixelShuffleSiglip2VisionConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.patch_size = config.patch_size

        self.patch_embedding = ReplicatedLinear(
            input_size=config.num_channels * self.patch_size * self.patch_size,
            output_size=self.embed_dim,
            return_bias=False,
        )

        self.num_patches = config.num_patches
        self.position_embedding_size = int(self.num_patches**0.5)
        self.position_embedding = nn.Embedding(self.num_patches, self.embed_dim)

    def positional_embeddings(
        self, packed_seq_patches: tuple[torch.Tensor, torch.Tensor, torch.Tensor]
    ) -> torch.Tensor:
        # Prepare positional embeddings grid: (1, embed_dim, h, w)
        positional_embeddings = (
            self.position_embedding.weight.reshape(
                self.position_embedding_size, self.position_embedding_size, -1
            )
```
**EN:** Class `Siglip2VariableSequenceEmbeddings` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, positional_embeddings, forward.
**CN:** 类 `Siglip2VariableSequenceEmbeddings` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, positional_embeddings, forward。

### Method `Siglip2VariableSequenceEmbeddings.__init__` (lines 93-107)
```python
    def __init__(self, config: PixelShuffleSiglip2VisionConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.patch_size = config.patch_size

        self.patch_embedding = ReplicatedLinear(
            input_size=config.num_channels * self.patch_size * self.patch_size,
            output_size=self.embed_dim,
            return_bias=False,
        )

        self.num_patches = config.num_patches
        self.position_embedding_size = int(self.num_patches**0.5)
        self.position_embedding = nn.Embedding(self.num_patches, self.embed_dim)
```
**EN:** Method `Siglip2VariableSequenceEmbeddings.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Siglip2VariableSequenceEmbeddings.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Siglip2VariableSequenceEmbeddings.positional_embeddings` (lines 109-152)
```python
    def positional_embeddings(
        self, packed_seq_patches: tuple[torch.Tensor, torch.Tensor, torch.Tensor]
    ) -> torch.Tensor:
        # Prepare positional embeddings grid: (1, embed_dim, h, w)
        positional_embeddings = (
            self.position_embedding.weight.reshape(
                self.position_embedding_size, self.position_embedding_size, -1
            )
            .permute(2, 0, 1)
            .unsqueeze(0)
        )

        _seq_patches, _seq_sizes, spatial_shapes = packed_seq_patches
        pos_embeds_list = []
        mode = "bilinear"
        align_corners = False
        antialias = True
        for spatial_shape in spatial_shapes:
            height, width = int(spatial_shape[0]), int(spatial_shape[1])
            # Guard to ensure height and width are positive for torch.compile
            if height > 0 and width > 0:
                resized_pos_embed = F.interpolate(
                    positional_embeddings,
                    size=(height, width),
                    mode=mode,
                    align_corners=align_corners,
                    antialias=antialias,
                )
                # Reshape from (1, embed_dim, height, width) to
                # (height*width, embed_dim)
                resized_pos_embed = resized_pos_embed.reshape(
                    self.embed_dim, height * width
                ).transpose(0, 1)
            else:
                # Fallback - should never happen in practice
                resized_pos_embed = positional_embeddings.reshape(
                    self.embed_dim,
                    self.position_embedding_size * self.position_embedding_size,
                ).transpose(0, 1)[: height * width]
            pos_embeds_list.append(resized_pos_embed)

        # Concatenate all positional embeddings along the sequence dimension
        pos_embeds = torch.cat(pos_embeds_list, dim=0)
        return pos_embeds
```
**EN:** Method `Siglip2VariableSequenceEmbeddings.positional_embeddings` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Siglip2VariableSequenceEmbeddings.positional_embeddings` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Class `IsaacProcessingInfo` (lines 310-373)
```python
class IsaacProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self) -> IsaacConfig:
        if hasattr(self.ctx, "get_hf_config"):
            original_config = self.ctx.get_hf_config()
            # Map HF config parameters to our vLLM config parameters
            return IsaacConfig(
                # Vision parameters - map from HF names
                vision_config=getattr(original_config, "vision_config", None),
                vision_patch_size=getattr(original_config, "video_patch_size", 16),
                vision_max_num_patches=getattr(
                    original_config, "vision_max_num_patches", 256
                ),
                vision_min_num_patches=getattr(
                    original_config, "vision_min_num_patches", None
                ),
                pixel_shuffle_scale=getattr(original_config, "pixel_shuffle_scale", 1),
                max_sequence_length=getattr(
                    original_config, "max_sequence_length", 16384
                ),
                vision_token=getattr(original_config, "vision_token", "<image>"),
                vision_attn_implementation=getattr(
                    original_config, "vision_attn_implementation", None
                ),
            )
        return IsaacConfig()
```
**EN:** Class `IsaacProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_image_processor, get_hf_processor, get_image_size_with_most_features, get_supported_mm_limits, get_mm_max_tokens_per_item.
**CN:** 类 `IsaacProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_image_processor, get_hf_processor, get_image_size_with_most_features, get_supported_mm_limits, get_mm_max_tokens_per_item。

### Method `IsaacProcessingInfo.get_hf_config` (lines 311-334)
```python
    def get_hf_config(self) -> IsaacConfig:
        if hasattr(self.ctx, "get_hf_config"):
            original_config = self.ctx.get_hf_config()
            # Map HF config parameters to our vLLM config parameters
            return IsaacConfig(
                # Vision parameters - map from HF names
                vision_config=getattr(original_config, "vision_config", None),
                vision_patch_size=getattr(original_config, "video_patch_size", 16),
                vision_max_num_patches=getattr(
                    original_config, "vision_max_num_patches", 256
                ),
                vision_min_num_patches=getattr(
                    original_config, "vision_min_num_patches", None
                ),
                pixel_shuffle_scale=getattr(original_config, "pixel_shuffle_scale", 1),
                max_sequence_length=getattr(
                    original_config, "max_sequence_length", 16384
                ),
                vision_token=getattr(original_config, "vision_token", "<image>"),
                vision_attn_implementation=getattr(
                    original_config, "vision_attn_implementation", None
                ),
            )
        return IsaacConfig()
```
**EN:** Method `IsaacProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `IsaacProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `IsaacProcessingInfo.get_image_processor` (lines 336-337)
```python
    def get_image_processor(self, **kwargs) -> IsaacImageProcessor:
        return IsaacImageProcessor(**kwargs)
```
**EN:** Method `IsaacProcessingInfo.get_image_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `IsaacProcessingInfo.get_image_processor` 封装了该模块中的一段可复用核心逻辑。

### Class `IsaacDummyInputsBuilder` (lines 376-403)
```python
class IsaacDummyInputsBuilder(BaseDummyInputsBuilder[IsaacProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        hf_processor = self.info.get_hf_processor()
        image_token: str = hf_processor.image_token

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
```
**EN:** Class `IsaacDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[IsaacProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `IsaacDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[IsaacProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `IsaacDummyInputsBuilder.get_dummy_text` (lines 377-383)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        hf_processor = self.info.get_hf_processor()
        image_token: str = hf_processor.image_token

        return image_token * num_images
```
**EN:** Method `IsaacDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `IsaacDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `IsaacDummyInputsBuilder.get_dummy_mm_data` (lines 385-403)
```python
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
            ),
        }
```
**EN:** Method `IsaacDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `IsaacDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `IsaacImagePixelInputs` (lines 406-429)
```python
class IsaacImagePixelInputs(TensorSchema):
    """
    Schema for validating Isaac image inputs.

    Dimensions:
        - np: Number of patches
        - d: Patch dimension
        - ni: Number of images

    The schema enforces:
        - pixel_values must be 2D: (num_patches, patch_dim)
        - image_grid_thw must be 2D: (num_images, 3)
          where 3 represents [T, H, W]
    """

    pixel_values: Annotated[
        torch.Tensor,
        TensorShape("np", "d"),
    ]

    image_grid_thw: Annotated[
        torch.Tensor,
        TensorShape("ni", 3),
    ]
```
**EN:** Class `IsaacImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `IsaacImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `IsaacMultiModalProcessor` (lines 432-475)
```python
class IsaacMultiModalProcessor(BaseMultiModalProcessor):
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        # Configure multimodal fields for Isaac model
        image_grid_thw = hf_inputs.get("image_grid_thw", torch.empty((0, 3)))
        image_grid_sizes = image_grid_thw.prod(-1)

        return {
            "pixel_values": MultiModalFieldConfig.flat_from_sizes(
                "image", image_grid_sizes
            ),
            "image_grid_thw": MultiModalFieldConfig.batched("image"),
        }

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, Any],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        image_processor = self.info.get_image_processor(**hf_processor_mm_kwargs)
```
**EN:** Class `IsaacMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor. Key methods include _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `IsaacMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor。 关键方法包括 _get_mm_fields_config, _get_prompt_updates。

### Class `Siglip2VisionAttention` (lines 478-565)
```python
class Siglip2VisionAttention(nn.Module):
    def __init__(
        self,
        config: PixelShuffleSiglip2VisionConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        prefix: str = "",
    ) -> None:
        super().__init__()

        use_data_parallel = is_vit_use_data_parallel()
        self.tp_size = (
            1
            if use_data_parallel
            else parallel_state.get_tensor_model_parallel_world_size()
        )
        self.tp_rank = parallel_state.get_tensor_model_parallel_rank()
        self.hidden_size_per_attention_head = dist_utils.divide(
            config.hidden_size, config.num_attention_heads
        )
        self.num_attention_heads_per_partition = dist_utils.divide(
            config.num_attention_heads, self.tp_size
        )

        self.qkv_proj = QKVParallelLinear(
```
**EN:** Class `Siglip2VisionAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, split_qkv, forward.
**CN:** 类 `Siglip2VisionAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, split_qkv, forward。

### Method `Siglip2VisionAttention.__init__` (lines 479-525)
```python
    def __init__(
        self,
        config: PixelShuffleSiglip2VisionConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        prefix: str = "",
    ) -> None:
        super().__init__()

        use_data_parallel = is_vit_use_data_parallel()
        self.tp_size = (
            1
            if use_data_parallel
            else parallel_state.get_tensor_model_parallel_world_size()
        )
        self.tp_rank = parallel_state.get_tensor_model_parallel_rank()
        self.hidden_size_per_attention_head = dist_utils.divide(
            config.hidden_size, config.num_attention_heads
        )
        self.num_attention_heads_per_partition = dist_utils.divide(
            config.num_attention_heads, self.tp_size
        )

        self.qkv_proj = QKVParallelLinear(
            hidden_size=config.hidden_size,
            head_size=self.hidden_size_per_attention_head,
            total_num_heads=config.num_attention_heads,
            total_num_kv_heads=config.num_attention_heads,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
            disable_tp=use_data_parallel,
        )
        self.out_proj = RowParallelLinear(
            input_size=config.hidden_size,
            output_size=config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.out_proj",
            disable_tp=use_data_parallel,
        )

        self.attn = MMEncoderAttention(
            num_heads=self.num_attention_heads_per_partition,
            head_size=self.hidden_size_per_attention_head,
            scale=self.hidden_size_per_attention_head**-0.5,
            prefix=f"{prefix}.attn",
        )
```
**EN:** Method `Siglip2VisionAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Siglip2VisionAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Siglip2VisionAttention.split_qkv` (lines 527-537)
```python
    def split_qkv(self, qkv: torch.Tensor) -> tuple[torch.Tensor, ...]:
        seq_len, bs, _ = qkv.shape
        q, k, v = qkv.chunk(3, dim=2)
        new_shape = (
            seq_len,
            bs,
            self.num_attention_heads_per_partition,
            self.hidden_size_per_attention_head,
        )
        q, k, v = (x.view(*new_shape) for x in (q, k, v))
        return q, k, v
```
**EN:** Method `Siglip2VisionAttention.split_qkv` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Siglip2VisionAttention.split_qkv` 封装了该模块中的一段可复用核心逻辑。

### Class `Siglip2EncoderLayer` (lines 568-613)
```python
class Siglip2EncoderLayer(nn.Module):
    def __init__(
        self,
        config: PixelShuffleSiglip2VisionConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.embed_dim = config.hidden_size
        self.layer_norm1 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)
        self.self_attn = Siglip2VisionAttention(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.layer_norm2 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)
        self.mlp = SiglipMLP(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )

    def forward(
        self,
```
**EN:** Class `Siglip2EncoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Siglip2EncoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Siglip2EncoderLayer.__init__` (lines 569-589)
```python
    def __init__(
        self,
        config: PixelShuffleSiglip2VisionConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.embed_dim = config.hidden_size
        self.layer_norm1 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)
        self.self_attn = Siglip2VisionAttention(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.layer_norm2 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)
        self.mlp = SiglipMLP(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )
```
**EN:** Method `Siglip2EncoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Siglip2EncoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Siglip2EncoderLayer.forward` (lines 591-613)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        *,
        cu_seqlens: torch.Tensor,
        max_seqlen: torch.Tensor | None,
    ) -> torch.Tensor:
        residual = hidden_states

        hidden_states = self.layer_norm1(hidden_states)
        hidden_states = self.self_attn(
            hidden_states=hidden_states,
            cu_seqlens=cu_seqlens,
            max_seqlen=max_seqlen,
        )
        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.layer_norm2(hidden_states)
        hidden_states = self.mlp(hidden_states)
        hidden_states = residual + hidden_states

        return hidden_states
```
**EN:** Method `Siglip2EncoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Siglip2EncoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Siglip2Encoder` (lines 616-651)
```python
class Siglip2Encoder(nn.Module):
    def __init__(
        self,
        config: PixelShuffleSiglip2VisionConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.layers = nn.ModuleList(
            [
                Siglip2EncoderLayer(
                    config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layers.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )

    def forward(
        self,
        inputs_embeds: torch.Tensor,
        *,
```
**EN:** Class `Siglip2Encoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Siglip2Encoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Siglip2Encoder.__init__` (lines 617-635)
```python
    def __init__(
        self,
        config: PixelShuffleSiglip2VisionConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.layers = nn.ModuleList(
            [
                Siglip2EncoderLayer(
                    config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layers.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )
```
**EN:** Method `Siglip2Encoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Siglip2Encoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Siglip2Encoder.forward` (lines 637-651)
```python
    def forward(
        self,
        inputs_embeds: torch.Tensor,
        *,
        cu_seqlens: torch.Tensor | None = None,
        max_seqlen: torch.Tensor | None = None,
    ) -> torch.Tensor:
        hidden_states = inputs_embeds
        for encoder_layer in self.layers:
            hidden_states = encoder_layer(
                hidden_states,
                cu_seqlens=cu_seqlens,
                max_seqlen=max_seqlen,
            )
        return hidden_states
```
**EN:** Method `Siglip2Encoder.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Siglip2Encoder.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Siglip2VisionTransformer` (lines 654-742)
```python
class Siglip2VisionTransformer(nn.Module):
    def __init__(
        self,
        config: PixelShuffleSiglip2VisionConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        embed_dim = config.hidden_size

        self.embeddings = Siglip2VariableSequenceEmbeddings(config)
        self.pixel_shuffle_scale_factor = config.pixel_shuffle_scale_factor
        self.encoder = Siglip2Encoder(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.encoder",
        )
        self.post_layernorm = nn.LayerNorm(embed_dim, eps=config.layer_norm_eps)

    def forward(
        self,
        packed_seq_patches: tuple[torch.Tensor, torch.Tensor],
    ) -> torch.Tensor:
```
**EN:** Class `Siglip2VisionTransformer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward, load_weights.
**CN:** 类 `Siglip2VisionTransformer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward, load_weights。

### Method `Siglip2VisionTransformer.__init__` (lines 655-673)
```python
    def __init__(
        self,
        config: PixelShuffleSiglip2VisionConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        embed_dim = config.hidden_size

        self.embeddings = Siglip2VariableSequenceEmbeddings(config)
        self.pixel_shuffle_scale_factor = config.pixel_shuffle_scale_factor
        self.encoder = Siglip2Encoder(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.encoder",
        )
        self.post_layernorm = nn.LayerNorm(embed_dim, eps=config.layer_norm_eps)
```
**EN:** Method `Siglip2VisionTransformer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Siglip2VisionTransformer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Siglip2VisionTransformer.forward` (lines 675-715)
```python
    def forward(
        self,
        packed_seq_patches: tuple[torch.Tensor, torch.Tensor],
    ) -> torch.Tensor:
        r"""
        spatial_shapes (`torch.LongTensor` of shape `(batch_size, 2)`):
            Tensor containing the spatial dimensions (height, width)
            of the input images.
        """

        seq_patches, token_grids = packed_seq_patches
        seq_sizes = torch.prod(token_grids, dim=-1)

        # Get embeddings from packed sequence
        hidden_states = self.embeddings((seq_patches, seq_sizes, token_grids))

        # Add a pseudo batch dimension for the encoder
        hidden_states = hidden_states.unsqueeze(0)

        cu_seqlens, max_seqlen = create_cumulative_seq_lengths(
            seq_sizes, hidden_states.device
        )

        hidden_states = self.encoder(
            inputs_embeds=hidden_states,
            cu_seqlens=cu_seqlens,
            max_seqlen=max_seqlen,
        )
        hidden_states = self.post_layernorm(hidden_states)

        if self.pixel_shuffle_scale_factor > 1:
            hidden_states = pixel_shuffle_varlen(
                x=hidden_states,
                token_grids=token_grids,
                scale_factor=self.pixel_shuffle_scale_factor,
            )
        # Remove the pseudo batch dimension we added earlier
        hidden_states = hidden_states.squeeze(0)

        # return last_hidden_state
        return hidden_states
```
**EN:** Method `Siglip2VisionTransformer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: spatial_shapes (`torch.LongTensor` of shape `(batch_size, 2)`): Tensor containing the spatial dimensions (height, width) of the input images.
**CN:** Method `Siglip2VisionTransformer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：spatial_shapes (`torch.LongTensor` of shape `(batch_size, 2)`): Tensor containing the spatial dimensions (height, width) of the input images。

### Class `IsaacVisionEmbedding` (lines 751-791)
```python
class IsaacVisionEmbedding(nn.Module):
    def __init__(
        self,
        vision_cfg: PixelShuffleSiglip2VisionConfig,
        hidden_dim: int,
        output_dim: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.transformer = Siglip2VisionTransformer(
            vision_cfg,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "0"),
        )
        self.linear_fc1 = ColumnParallelLinear(
            hidden_dim,
            4 * hidden_dim,
            bias=False,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "1"),
            return_bias=False,
        )
        self.act = nn.SiLU()
        self.linear_fc2 = RowParallelLinear(
```
**EN:** Class `IsaacVisionEmbedding` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `IsaacVisionEmbedding` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `IsaacVisionEmbedding.__init__` (lines 752-782)
```python
    def __init__(
        self,
        vision_cfg: PixelShuffleSiglip2VisionConfig,
        hidden_dim: int,
        output_dim: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.transformer = Siglip2VisionTransformer(
            vision_cfg,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "0"),
        )
        self.linear_fc1 = ColumnParallelLinear(
            hidden_dim,
            4 * hidden_dim,
            bias=False,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "1"),
            return_bias=False,
        )
        self.act = nn.SiLU()
        self.linear_fc2 = RowParallelLinear(
            4 * hidden_dim,
            output_dim,
            bias=False,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "3"),
            return_bias=False,
        )
```
**EN:** Method `IsaacVisionEmbedding.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `IsaacVisionEmbedding.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `IsaacVisionEmbedding.forward` (lines 784-791)
```python
    def forward(
        self, packed_seq_patches: tuple[torch.Tensor, torch.Tensor]
    ) -> torch.Tensor:
        hidden_states = self.transformer(packed_seq_patches)
        hidden_states = self.linear_fc1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states = self.linear_fc2(hidden_states)
        return hidden_states
```
**EN:** Method `IsaacVisionEmbedding.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `IsaacVisionEmbedding.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

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
- **Standard library / 标准库**: `from __future__ import annotations`, `from collections.abc import Iterable, Iterator, Mapping, Sequence`, `from typing import Annotated, Any`, `from einops import rearrange`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `import torch.nn as nn`, `import torch.nn.functional as F`, `from transformers.image_processing_utils import BatchFeature`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import ModelConfig, VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.distributed import parallel_state`, `from vllm.distributed import utils as dist_utils`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.attention import MMEncoderAttention`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.model_loader.weight_utils import (`, `from vllm.model_executor.models.interfaces import (`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.model_executor.models.siglip import SiglipMLP`
