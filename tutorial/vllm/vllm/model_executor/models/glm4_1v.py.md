# glm4_1v.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/glm4_1v.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for glm4_1v, including architecture wrappers and weight loading logic. / 面向推理的 glm4_1v vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 30-118)
```python
import math
from collections.abc import Callable, Iterable, Iterator, Mapping, Sequence
from functools import partial
from typing import Annotated, Any, Literal, TypeAlias

import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange
from transformers import BatchFeature, Glm4vProcessor
from transformers.models.glm4v.configuration_glm4v import (
    Glm4vTextConfig,
    Glm4vVisionConfig,
)
from transformers.models.glm4v.image_processing_glm4v import (
    Glm4vImageProcessor,
    smart_resize,
)
from transformers.models.glm4v.video_processing_glm4v import Glm4vVideoProcessor
from transformers.video_utils import VideoMetadata

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions, VideoDummyOptions
from vllm.distributed import get_tensor_model_parallel_world_size, parallel_state
from vllm.distributed import utils as dist_utils
from vllm.inputs import MultiModalDataDict
from vllm.logger import init_logger
from vllm.model_executor.layers.attention import (
    MMEncoderAttention,
)
from vllm.model_executor.layers.conv import Conv2dLayer, Conv3dLayer
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.quantization.compressed_tensors import (
    compressed_tensors,
)
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.rotary_embedding.common import (
    ApplyRotaryEmb,
)
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalFieldConfig,
    MultiModalKwargsItems,
    VideoItem,
)
from vllm.multimodal.parse import ImageSize, MultiModalDataItems, MultiModalDataParser
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
from vllm.v1.attention.backends.registry import AttentionBackendEnum

from ..layers.activation import SiluAndMul
from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMRoPE,
    SupportsMultiModal,
    SupportsPP,
)
from .qwen2_vl import _create_qwen2vl_field_factory
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
from .vision import (
    get_vit_attn_backend,
    is_vit_use_data_parallel,
    run_dp_sharded_mrope_vision_model,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 123-123)
```python
_MAX_FRAMES_PER_VIDEO = 600
```
**EN:** This block defines _MAX_FRAMES_PER_VIDEO, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _MAX_FRAMES_PER_VIDEO，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Function `all_gather_interleave` (lines 235-253)
```python
def all_gather_interleave(local_tensor, hidden_size: int, tp_size: int):
    """All-gather the input tensor interleavely across model parallel group."""
    import torch.distributed as dist

    gathered_tensors = [torch.zeros_like(local_tensor) for _ in range(tp_size)]
    dist.all_gather(
        gathered_tensors,
        local_tensor,
        group=parallel_state.get_tp_group().device_group,
    )

    gathered_tensors_split = [
        torch.split(tensor, hidden_size // tp_size, -1) for tensor in gathered_tensors
    ]
    ordered_tensors = [
        tensor for pair in zip(*gathered_tensors_split) for tensor in pair
    ]
    result_tensor = torch.cat(ordered_tensors, dim=-1)
    return result_tensor
```
**EN:** Function `all_gather_interleave` encapsulates a focused piece of reusable logic inside this module. The docstring says: All-gather the input tensor interleavely across model parallel group.
**CN:** Function `all_gather_interleave` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：All-gather the input tensor interleavely across model parallel group。

### Class `Glm4vImagePixelInputs` (lines 128-140)
```python
class Glm4vImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - np: Number of patches
        - cpp: Number of channels * patch_size * patch_size
        - ni: Number of images
        - g: Grid dimensions (3 for grid_t, grid_h, grid_w)
    """

    type: Literal["pixel_values"] = "pixel_values"

    pixel_values: Annotated[torch.Tensor, TensorShape("np", "cpp")]
    image_grid_thw: Annotated[torch.Tensor, TensorShape("ni", 3)]
```
**EN:** Class `Glm4vImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Glm4vImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Glm4vImageEmbeddingInputs` (lines 143-155)
```python
class Glm4vImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - f: Number of image features (varies based on image resolution)
        - h: Hidden size (must match language model backbone)
        - n: Number of images
        - g: Grid dimensions (3 for grid_t, grid_h, grid_w)
    """

    type: Literal["image_embeds"] = "image_embeds"

    image_embeds: Annotated[torch.Tensor, TensorShape("f", "h")]
    image_grid_thw: Annotated[torch.Tensor, TensorShape("n", 3)]
```
**EN:** Class `Glm4vImageEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Glm4vImageEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Glm4vVideoPixelInputs` (lines 161-175)
```python
class Glm4vVideoPixelInputs(TensorSchema):
    """
    Dimensions:
        - np: Number of patches
        - ctpp: Number of channels * temporal_patch_size *
            patch_size * patch_size
        - f: Number of frames
        - g: Grid dimensions (3 for grid_t which is usually 1 for processed
          video, grid_h, grid_w)
    """

    type: Literal["pixel_values_videos"] = "pixel_values_videos"

    pixel_values_videos: Annotated[torch.Tensor, TensorShape("np", "ctpp")]
    video_grid_thw: Annotated[torch.Tensor, TensorShape("f", 3)]
```
**EN:** Class `Glm4vVideoPixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Glm4vVideoPixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Glm4vVideoEmbeddingInputs` (lines 178-191)
```python
class Glm4vVideoEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - p: Number of video patches across all frames
        - h: Hidden size (must match language model backbone)
        - f: Number of frames
        - g: Grid dimensions (3 for grid_t which is usually 1 for processed
          video, grid_h, grid_w)
    """

    type: Literal["video_embeds"] = "video_embeds"

    video_embeds: Annotated[torch.Tensor, TensorShape("p", "h")]
    video_grid_thw: Annotated[torch.Tensor, TensorShape("f", 3)]
```
**EN:** Class `Glm4vVideoEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Glm4vVideoEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Glm4vVisionMLP` (lines 199-232)
```python
class Glm4vVisionMLP(nn.Module):
    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        bias: bool = False,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        use_data_parallel = is_vit_use_data_parallel()
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=in_features,
            output_sizes=[hidden_features] * 2,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
            disable_tp=use_data_parallel,
        )
        self.down_proj = RowParallelLinear(
            hidden_features,
            in_features,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
```
**EN:** Class `Glm4vVisionMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Glm4vVisionMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Glm4vVisionMLP.__init__` (lines 200-226)
```python
    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        bias: bool = False,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        use_data_parallel = is_vit_use_data_parallel()
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=in_features,
            output_sizes=[hidden_features] * 2,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
            disable_tp=use_data_parallel,
        )
        self.down_proj = RowParallelLinear(
            hidden_features,
            in_features,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
            disable_tp=use_data_parallel,
        )
        self.act_fn = SiluAndMul()
```
**EN:** Method `Glm4vVisionMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4vVisionMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4vVisionMLP.forward` (lines 228-232)
```python
    def forward(self, x: torch.Tensor):
        x, _ = self.gate_up_proj(x)
        x = self.act_fn(x)
        x, _ = self.down_proj(x)
        return x
```
**EN:** Method `Glm4vVisionMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4vVisionMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Glm4vVisionAttention` (lines 256-364)
```python
class Glm4vVisionAttention(nn.Module):
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        projection_size: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        # Per attention head and per partition values.
        use_data_parallel = is_vit_use_data_parallel()
        self.tp_size = (
            1 if use_data_parallel else get_tensor_model_parallel_world_size()
        )
        self.tp_rank = (
            0 if use_data_parallel else parallel_state.get_tensor_model_parallel_rank()
        )
        self.hidden_size_per_attention_head = dist_utils.divide(
            projection_size, num_heads
        )
        self.num_attention_heads_per_partition = dist_utils.divide(
            num_heads, self.tp_size
        )
```
**EN:** Class `Glm4vVisionAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, split_qkv, forward.
**CN:** 类 `Glm4vVisionAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, split_qkv, forward。

### Method `Glm4vVisionAttention.__init__` (lines 257-310)
```python
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        projection_size: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        # Per attention head and per partition values.
        use_data_parallel = is_vit_use_data_parallel()
        self.tp_size = (
            1 if use_data_parallel else get_tensor_model_parallel_world_size()
        )
        self.tp_rank = (
            0 if use_data_parallel else parallel_state.get_tensor_model_parallel_rank()
        )
        self.hidden_size_per_attention_head = dist_utils.divide(
            projection_size, num_heads
        )
        self.num_attention_heads_per_partition = dist_utils.divide(
            num_heads, self.tp_size
        )

        self.qkv = QKVParallelLinear(
            hidden_size=embed_dim,
            head_size=self.hidden_size_per_attention_head,
            total_num_heads=num_heads,
            total_num_kv_heads=num_heads,
            bias=False,
            quant_config=quant_config,
            # Change qkv prefix to align with GLM-4.5V-FP8 quantization cfg
            prefix=f"{prefix}.qkv_proj"
            if isinstance(quant_config, compressed_tensors.CompressedTensorsConfig)
            else f"{prefix}.qkv",
            disable_tp=use_data_parallel,
        )
        self.proj = RowParallelLinear(
            input_size=projection_size,
            output_size=embed_dim,
            quant_config=quant_config,
            prefix=f"{prefix}.proj",
            bias=False,
            disable_tp=use_data_parallel,
        )

        self.attn = MMEncoderAttention(
            num_heads=self.num_attention_heads_per_partition,
            head_size=self.hidden_size_per_attention_head,
            scale=self.hidden_size_per_attention_head**-0.5,
            prefix=f"{prefix}.attn",
        )

        self.apply_rotary_emb = ApplyRotaryEmb(enforce_enable=True)
```
**EN:** Method `Glm4vVisionAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4vVisionAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4vVisionAttention.split_qkv` (lines 312-327)
```python
    def split_qkv(self, qkv: torch.Tensor) -> tuple[torch.Tensor, ...]:
        # [s, b, 3 * head * head_dim]
        seq_len, bs, _ = qkv.shape

        # [s, b, 3 * head * head_dim] -> 3 * [s, b, head * head_dim]
        q, k, v = qkv.chunk(3, dim=2)

        # 3 * [s, b, head * head_dim] -> 3 * [s, b, head, head_dim]
        new_shape = (
            seq_len,
            bs,
            self.num_attention_heads_per_partition,
            self.hidden_size_per_attention_head,
        )
        q, k, v = (x.view(*new_shape) for x in (q, k, v))
        return q, k, v
```
**EN:** Method `Glm4vVisionAttention.split_qkv` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Glm4vVisionAttention.split_qkv` 封装了该模块中的一段可复用核心逻辑。

### Class `Glm4vVisionBlock` (lines 367-415)
```python
class Glm4vVisionBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        num_heads: int,
        mlp_hidden_dim: int,
        norm_layer: Callable[[int], nn.Module] | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm1 = norm_layer(dim)
        self.norm2 = norm_layer(dim)
        self.attn = Glm4vVisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            projection_size=dim,
            quant_config=quant_config,
            prefix=f"{prefix}.attn",
        )
        self.mlp = Glm4vVisionMLP(
            dim,
            mlp_hidden_dim,
```
**EN:** Class `Glm4vVisionBlock` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Glm4vVisionBlock` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Glm4vVisionBlock.__init__` (lines 368-395)
```python
    def __init__(
        self,
        dim: int,
        num_heads: int,
        mlp_hidden_dim: int,
        norm_layer: Callable[[int], nn.Module] | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm1 = norm_layer(dim)
        self.norm2 = norm_layer(dim)
        self.attn = Glm4vVisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            projection_size=dim,
            quant_config=quant_config,
            prefix=f"{prefix}.attn",
        )
        self.mlp = Glm4vVisionMLP(
            dim,
            mlp_hidden_dim,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )
```
**EN:** Method `Glm4vVisionBlock.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4vVisionBlock.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4vVisionBlock.forward` (lines 397-415)
```python
    def forward(
        self,
        x: torch.Tensor,
        cu_seqlens: torch.Tensor,
        rotary_pos_emb_cos: torch.Tensor,
        rotary_pos_emb_sin: torch.Tensor,
        max_seqlen: int | None = None,  # Only used for Flash Attention
    ) -> torch.Tensor:
        x_attn = self.attn(
            self.norm1(x),
            cu_seqlens=cu_seqlens,
            rotary_pos_emb_cos=rotary_pos_emb_cos,
            rotary_pos_emb_sin=rotary_pos_emb_sin,
            max_seqlen=max_seqlen,
        )
        x_fused_norm, residual = self.norm2(x, residual=x_attn)
        x = residual + self.mlp(x_fused_norm)

        return x
```
**EN:** Method `Glm4vVisionBlock.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4vVisionBlock.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Glm4vVisionPatchEmbed` (lines 418-444)
```python
class Glm4vVisionPatchEmbed(nn.Module):
    def __init__(
        self,
        patch_size: int = 14,
        temporal_patch_size: int = 1,
        in_channels: int = 3,
        hidden_size: int = 1536,
    ) -> None:
        super().__init__()
        self.patch_size = patch_size
        self.temporal_patch_size = temporal_patch_size
        self.hidden_size = hidden_size

        kernel_size = (temporal_patch_size, patch_size, patch_size)
        self.proj = Conv3dLayer(
            in_channels,
            hidden_size,
            kernel_size=kernel_size,
            stride=kernel_size,
            bias=True,
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        L, C = x.shape
        x = x.view(L, -1, self.temporal_patch_size, self.patch_size, self.patch_size)
```
**EN:** Class `Glm4vVisionPatchEmbed` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Glm4vVisionPatchEmbed` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Glm4vVisionPatchEmbed.__init__` (lines 419-438)
```python
    def __init__(
        self,
        patch_size: int = 14,
        temporal_patch_size: int = 1,
        in_channels: int = 3,
        hidden_size: int = 1536,
    ) -> None:
        super().__init__()
        self.patch_size = patch_size
        self.temporal_patch_size = temporal_patch_size
        self.hidden_size = hidden_size

        kernel_size = (temporal_patch_size, patch_size, patch_size)
        self.proj = Conv3dLayer(
            in_channels,
            hidden_size,
            kernel_size=kernel_size,
            stride=kernel_size,
            bias=True,
        )
```
**EN:** Method `Glm4vVisionPatchEmbed.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4vVisionPatchEmbed.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4vVisionPatchEmbed.forward` (lines 440-444)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        L, C = x.shape
        x = x.view(L, -1, self.temporal_patch_size, self.patch_size, self.patch_size)
        x = self.proj(x).view(L, self.hidden_size)
        return x
```
**EN:** Method `Glm4vVisionPatchEmbed.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4vVisionPatchEmbed.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Glm4vPatchMerger` (lines 447-494)
```python
class Glm4vPatchMerger(nn.Module):
    def __init__(
        self,
        d_model: int,
        context_dim: int,
        quant_config: QuantizationConfig | None = None,
        bias: bool = False,
        prefix: str = "",
    ) -> None:
        super().__init__()
        use_data_parallel = is_vit_use_data_parallel()
        self.hidden_size = d_model
        self.proj = ColumnParallelLinear(
            self.hidden_size,
            self.hidden_size,
            bias=bias,
            gather_output=True,
            quant_config=quant_config,
            prefix=f"{prefix}.proj",
            disable_tp=use_data_parallel,
        )
        self.post_projection_norm = nn.LayerNorm(self.hidden_size)
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=self.hidden_size,
            output_sizes=[context_dim] * 2,
```
**EN:** Class `Glm4vPatchMerger` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Glm4vPatchMerger` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Glm4vPatchMerger.__init__` (lines 448-486)
```python
    def __init__(
        self,
        d_model: int,
        context_dim: int,
        quant_config: QuantizationConfig | None = None,
        bias: bool = False,
        prefix: str = "",
    ) -> None:
        super().__init__()
        use_data_parallel = is_vit_use_data_parallel()
        self.hidden_size = d_model
        self.proj = ColumnParallelLinear(
            self.hidden_size,
            self.hidden_size,
            bias=bias,
            gather_output=True,
            quant_config=quant_config,
            prefix=f"{prefix}.proj",
            disable_tp=use_data_parallel,
        )
        self.post_projection_norm = nn.LayerNorm(self.hidden_size)
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=self.hidden_size,
            output_sizes=[context_dim] * 2,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
            disable_tp=use_data_parallel,
        )
        self.down_proj = RowParallelLinear(
            context_dim,
            self.hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
            disable_tp=use_data_parallel,
        )
        self.act_fn = SiluAndMul()
        self.extra_activation_func = nn.GELU()
```
**EN:** Method `Glm4vPatchMerger.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4vPatchMerger.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4vPatchMerger.forward` (lines 488-494)
```python
    def forward(self, x: torch.Tensor):
        x, _ = self.proj(x)
        x = self.extra_activation_func(self.post_projection_norm(x))
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** Method `Glm4vPatchMerger.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4vPatchMerger.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Glm4vVisionEmbeddings` (lines 497-604)
```python
class Glm4vVisionEmbeddings(nn.Module):
    def __init__(self, config: Glm4vVisionConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = config.image_size
        self.patch_size = config.patch_size

        self.num_patches = (self.image_size // self.patch_size) ** 2
        self.num_positions = self.num_patches
        self.position_embedding = nn.Embedding(self.num_positions, self.embed_dim)
        self.register_buffer(
            "position_ids",
            torch.arange(self.num_positions).expand((1, -1)),
            persistent=False,
        )

    def forward(
        self, embeddings, lengths, image_shapes, h_coords, w_coords
    ) -> torch.Tensor:
        pos_embed_weight = self.position_embedding.weight
        hidden_size = pos_embed_weight.shape[1]
        total_seq = h_coords.shape[0]
        device = pos_embed_weight.device
```
**EN:** Class `Glm4vVisionEmbeddings` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Glm4vVisionEmbeddings` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Glm4vVisionEmbeddings.__init__` (lines 498-512)
```python
    def __init__(self, config: Glm4vVisionConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = config.image_size
        self.patch_size = config.patch_size

        self.num_patches = (self.image_size // self.patch_size) ** 2
        self.num_positions = self.num_patches
        self.position_embedding = nn.Embedding(self.num_positions, self.embed_dim)
        self.register_buffer(
            "position_ids",
            torch.arange(self.num_positions).expand((1, -1)),
            persistent=False,
        )
```
**EN:** Method `Glm4vVisionEmbeddings.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4vVisionEmbeddings.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4vVisionEmbeddings.forward` (lines 514-604)
```python
    def forward(
        self, embeddings, lengths, image_shapes, h_coords, w_coords
    ) -> torch.Tensor:
        pos_embed_weight = self.position_embedding.weight
        hidden_size = pos_embed_weight.shape[1]
        total_seq = h_coords.shape[0]
        device = pos_embed_weight.device

        # Move coordinates to correct device
        h_coords, w_coords = h_coords.to(device), w_coords.to(device)

        # Handle empty sequence case
        if total_seq == 0:
            adapted_pos_embed = torch.empty(
                0, hidden_size, device=device, dtype=pos_embed_weight.dtype
            )
        else:
            # Convert inputs to tensors if needed
            if isinstance(lengths, list):
                lengths = torch.tensor(lengths, device=device, dtype=torch.long)
            if not isinstance(image_shapes, torch.Tensor):
                image_shapes = torch.tensor(
                    image_shapes, device=device, dtype=torch.long
                )

            # Prepare 2D position embedding
            orig_size_sq = pos_embed_weight.shape[0]
            orig_size = int(orig_size_sq**0.5)
            pos_embed_2d = (
                pos_embed_weight.view(orig_size, orig_size, hidden_size)
                .permute(2, 0, 1)
                .unsqueeze(0)
                .to(device=device, dtype=torch.float32)
            )

            # Calculate target dimensions for each patch
            # Add bounds checking for data parallel mode
            if len(lengths) > image_shapes.shape[0]:
                # In data parallel mode, some GPUs might not have all
                # image shapes
                # Use available image shapes, cycling if necessary
                target_h_list = []
                target_w_list = []
                for i in range(len(lengths)):
                    # Cycle through available shapes
                    shape_idx = i % image_shapes.shape[0]
                    target_h_list.append(image_shapes[shape_idx, 1].repeat(lengths[i]))
                    target_w_list.append(image_shapes[shape_idx, 2].repeat(lengths[i]))
                target_h = torch.cat(target_h_list).to(
                    device=device, dtype=torch.float32
                )
                target_w = torch.cat(target_w_list).to(
                    device=device, dtype=torch.float32
                )
            else:
                target_h = torch.cat(
                    [image_shapes[i, 1].repeat(lengths[i]) for i in range(len(lengths))]
                ).to(device=device, dtype=torch.float32)
                target_w = torch.cat(
                    [image_shapes[i, 2].repeat(lengths[i]) for i in range(len(lengths))]
# ... truncated for analysis ...
```
**EN:** Method `Glm4vVisionEmbeddings.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4vVisionEmbeddings.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

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
- **Standard library / 标准库**: `import math`, `from collections.abc import Callable, Iterable, Iterator, Mapping, Sequence`, `from functools import partial`, `from typing import Annotated, Any, Literal, TypeAlias`, `from einops import rearrange`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `import torch.nn as nn`, `import torch.nn.functional as F`, `from transformers import BatchFeature, Glm4vProcessor`, `from transformers.models.glm4v.configuration_glm4v import (`, `from transformers.models.glm4v.image_processing_glm4v import (`, `from transformers.models.glm4v.video_processing_glm4v import Glm4vVideoProcessor`, `from transformers.video_utils import VideoMetadata`, `import torch.distributed as dist`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions, VideoDummyOptions`, `from vllm.distributed import get_tensor_model_parallel_world_size, parallel_state`, `from vllm.distributed import utils as dist_utils`, `from vllm.inputs import MultiModalDataDict`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.attention import (`, `from vllm.model_executor.layers.conv import Conv2dLayer, Conv3dLayer`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.quantization.compressed_tensors import (`
- **Module note / 模块说明**: **EN:** Inference-only GLM-4.1V & GLM-4.6V-Flash, AutoGLM-Phone-9B model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only GLM-4.1V & GLM-4.6V-Flash, AutoGLM-Phone-9B model compatible with HuggingFace weights.。
