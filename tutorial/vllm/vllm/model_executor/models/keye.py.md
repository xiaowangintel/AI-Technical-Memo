# keye.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/keye.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for keye, including architecture wrappers and weight loading logic. / 面向推理的 keye vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-82)
```python
import math
from abc import abstractmethod
from collections.abc import Iterable, Iterator, Mapping, Sequence
from functools import partial
from typing import Annotated, Any, Literal, TypeAlias, TypeVar

import numpy as np
import torch
import torch.nn as nn
from einops import rearrange
from transformers import BaseImageProcessor, PretrainedConfig
from transformers.activations import GELUActivation
from transformers.feature_extraction_utils import BatchFeature
from transformers.modeling_outputs import BaseModelOutput, BaseModelOutputWithPooling
from transformers.utils import torch_int

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.inputs import ModalityData, MultiModalDataDict
from vllm.logger import init_logger
from vllm.model_executor.layers.attention import (
    MMEncoderAttention,
)
from vllm.model_executor.layers.conv import Conv2dLayer
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding.common import (
    ApplyRotaryEmb,
)
from vllm.model_executor.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    ImageItem,
    MultiModalFeatureSpec,
    MultiModalFieldConfig,
    MultiModalKwargsItems,
    VideoItem,
)
from vllm.multimodal.parse import (
    DictEmbeddingItems,
    ImageSize,
    ModalityDataItems,
    MultiModalDataItems,
    MultiModalDataParser,
)
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMRoPE,
    SupportsMultiModal,
    SupportsPP,
)
from .siglip import SiglipMLP
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    is_pp_missing_parameter,
    maybe_prefix,
)
from .vision import is_vit_use_data_parallel
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 1160-1160)
```python
_I = TypeVar("_I", bound=KeyeProcessingInfo)
```
**EN:** This block defines _I, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _I，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Function `smart_resize` (lines 87-127)
```python
def smart_resize(
    height: int,
    width: int,
    factor: int,
    min_pixels: int,
    max_pixels: int,
):
    if height < factor:
        logger.warning(
            "smart_resize: height=%s < factor=%s, reset height=factor",
            height,
            factor,
        )
        width = round((width * factor) / height)
        height = factor

    if width < factor:
        logger.warning(
            "smart_resize: width=%s < factor=%s, reset width=factor",
            width,
            factor,
        )
        height = round((height * factor) / width)
        width = factor

    if max(height, width) / min(height, width) > 200:
        raise ValueError(
            "absolute aspect ratio must be smaller than 200, got "
            "{max(height, width) / min(height, width)}"
        )
    h_bar = round(height / factor) * factor
    w_bar = round(width / factor) * factor
    if h_bar * w_bar > max_pixels:
        beta = math.sqrt((height * width) / max_pixels)
        h_bar = math.floor(height / beta / factor) * factor
        w_bar = math.floor(width / beta / factor) * factor
    elif h_bar * w_bar < min_pixels:
        beta = math.sqrt(min_pixels / (height * width))
        h_bar = math.ceil(height * beta / factor) * factor
        w_bar = math.ceil(width * beta / factor) * factor
    return h_bar, w_bar
```
**EN:** Function `smart_resize` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `smart_resize` 封装了该模块中的一段可复用核心逻辑。

### Function `apply_rotary_pos_emb_flashatt` (lines 337-350)
```python
def apply_rotary_pos_emb_flashatt(
    q: torch.Tensor,
    k: torch.Tensor,
    cos: torch.Tensor,
    sin: torch.Tensor,
    apply_rotary_emb: ApplyRotaryEmb,
) -> tuple[torch.Tensor, torch.Tensor]:
    cos = cos.chunk(2, dim=-1)[0].contiguous()
    sin = sin.chunk(2, dim=-1)[0].contiguous()

    q_embed = apply_rotary_emb(q, cos, sin)
    k_embed = apply_rotary_emb(k, cos, sin)

    return q_embed, k_embed
```
**EN:** Function `apply_rotary_pos_emb_flashatt` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `apply_rotary_pos_emb_flashatt` 封装了该模块中的一段可复用核心逻辑。

### Class `KeyeImagePixelInputs` (lines 130-144)
```python
class KeyeImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bnp: Batch size * Number of patches
        - c: Number of channels
        - ps: Patch size
        - ni: Number of images
        - g: Grid dimensions (3 for t, h, w)
    """

    type: Literal["pixel_values"]
    pixel_values: Annotated[
        torch.Tensor, TensorShape("bnp", 3, "ps", "ps", dynamic_dims={"bnp"})
    ]
    image_grid_thw: Annotated[torch.Tensor, TensorShape("ni", 3)]
```
**EN:** Class `KeyeImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `KeyeImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `KeyeImageEmbeddingInputs` (lines 147-159)
```python
class KeyeImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - nf: Number of image features
        - hs: Hidden size (must match the hidden size of language model
          backbone)
        - ni: Number of images
        - g: Grid dimensions (3 for t, h, w)
    """

    type: Literal["image_embeds"]
    image_embeds: Annotated[torch.Tensor, TensorShape("nf", "hs")]
    image_grid_thw: Annotated[torch.Tensor, TensorShape("ni", 3)]
```
**EN:** Class `KeyeImageEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `KeyeImageEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `KeyeVideoPixelInputs` (lines 165-179)
```python
class KeyeVideoPixelInputs(TensorSchema):
    """
    Dimensions:
        - bnp: Batch size * Number of patches
        - c: Number of channels
        - ps: Patch size
        - ni: Number of images
        - g: Grid dimensions (3 for t, h, w)
    """

    type: Literal["pixel_values_videos"]
    pixel_values_videos: Annotated[
        torch.Tensor, TensorShape("bnp", 3, "ps", "ps", dynamic_dims={"bnp"})
    ]
    video_grid_thw: Annotated[torch.Tensor, TensorShape("nv", 3)]
```
**EN:** Class `KeyeVideoPixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `KeyeVideoPixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `KeyeVideoEmbeddingInputs` (lines 182-194)
```python
class KeyeVideoEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - nf: Number of video features
        - hs: Hidden size (must match the hidden size of language model
          backbone)
        - nv: Number of videos
        - g: Grid dimensions (3 for t, h, w)
    """

    type: Literal["video_embeds"]
    video_embeds: Annotated[torch.Tensor, TensorShape("nf", "hs")]
    video_grid_thw: Annotated[torch.Tensor, TensorShape("nv", 3)]
```
**EN:** Class `KeyeVideoEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `KeyeVideoEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `KeyeVisionEmbeddings` (lines 200-334)
```python
class KeyeVisionEmbeddings(nn.Module):
    def __init__(self, config: PretrainedConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = config.image_size
        self.patch_size = config.patch_size

        self.patch_embedding = Conv2dLayer(
            in_channels=config.num_channels,
            out_channels=self.embed_dim,
            kernel_size=self.patch_size,
            stride=self.patch_size,
            padding="valid",
        )

        self.num_patches = (self.image_size // self.patch_size) ** 2
        self.num_positions = self.num_patches
        self.cache_position_embedding = dict()
        self.cache_position_count = dict()
        self.position_embedding = nn.Embedding(self.num_positions, self.embed_dim)
        self.packing_position_embedding = nn.Embedding(32768, self.embed_dim)

        self.register_buffer(
            "position_ids",
```
**EN:** Class `KeyeVisionEmbeddings` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, interpolate_pos_encoding, fetch_position_embedding_lfu_cache, forward.
**CN:** 类 `KeyeVisionEmbeddings` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, interpolate_pos_encoding, fetch_position_embedding_lfu_cache, forward。

### Method `KeyeVisionEmbeddings.__init__` (lines 201-227)
```python
    def __init__(self, config: PretrainedConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = config.image_size
        self.patch_size = config.patch_size

        self.patch_embedding = Conv2dLayer(
            in_channels=config.num_channels,
            out_channels=self.embed_dim,
            kernel_size=self.patch_size,
            stride=self.patch_size,
            padding="valid",
        )

        self.num_patches = (self.image_size // self.patch_size) ** 2
        self.num_positions = self.num_patches
        self.cache_position_embedding = dict()
        self.cache_position_count = dict()
        self.position_embedding = nn.Embedding(self.num_positions, self.embed_dim)
        self.packing_position_embedding = nn.Embedding(32768, self.embed_dim)

        self.register_buffer(
            "position_ids",
            torch.arange(self.num_positions).expand((1, -1)),
            persistent=False,
        )
```
**EN:** Method `KeyeVisionEmbeddings.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KeyeVisionEmbeddings.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KeyeVisionEmbeddings.interpolate_pos_encoding` (lines 229-263)
```python
    def interpolate_pos_encoding(
        self,
        embeddings: torch.Tensor,
        height: int,
        width: int,
        is_after_patchify: bool = False,
    ) -> torch.Tensor:
        num_positions = self.position_embedding.weight.shape[0]

        patch_pos_embed = self.position_embedding.weight.unsqueeze(0)

        dim = embeddings.shape[-1]

        if is_after_patchify:
            new_height = height
            new_width = width
        else:
            new_height = height // self.patch_size
            new_width = width // self.patch_size

        sqrt_num_positions = torch_int(num_positions**0.5)
        patch_pos_embed = patch_pos_embed.reshape(
            1, sqrt_num_positions, sqrt_num_positions, dim
        )
        patch_pos_embed = patch_pos_embed.permute(0, 3, 1, 2)

        patch_pos_embed = nn.functional.interpolate(
            patch_pos_embed,
            size=(new_height, new_width),
            mode="bilinear",
            align_corners=False,
        )

        patch_pos_embed = patch_pos_embed.permute(0, 2, 3, 1).view(1, -1, dim)
        return patch_pos_embed
```
**EN:** Method `KeyeVisionEmbeddings.interpolate_pos_encoding` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KeyeVisionEmbeddings.interpolate_pos_encoding` 封装了该模块中的一段可复用核心逻辑。

### Class `KeyeSiglipAttention` (lines 353-468)
```python
class KeyeSiglipAttention(nn.Module):
    """Multi-headed attention from 'Attention Is All You
    Need' paper."""

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        hidden_size = config.hidden_size
        self.hidden_size = config.hidden_size
        use_data_parallel = is_vit_use_data_parallel()
        tp_size = 1 if use_data_parallel else get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_attention_heads
        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0
```
**EN:** Class `KeyeSiglipAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `KeyeSiglipAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `KeyeSiglipAttention.__init__` (lines 357-411)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        hidden_size = config.hidden_size
        self.hidden_size = config.hidden_size
        use_data_parallel = is_vit_use_data_parallel()
        tp_size = 1 if use_data_parallel else get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_attention_heads
        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = config.hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scale = self.head_dim**-0.5

        self.qkv_proj = QKVParallelLinear(
            hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )
        self.out_proj = RowParallelLinear(
            input_size=hidden_size,
            output_size=hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.out_proj",
        )

        self.attn = MMEncoderAttention(
            num_heads=self.num_heads,
            head_size=self.head_dim,
            scale=self.scale,
            num_kv_heads=self.num_kv_heads,
            prefix=f"{prefix}.attn",
        )

        self.apply_rotary_emb = ApplyRotaryEmb(
            enforce_enable=True,
            enable_fp32_compute=True,
        )
```
**EN:** Method `KeyeSiglipAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KeyeSiglipAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KeyeSiglipAttention.forward` (lines 413-468)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        attention_mask: torch.Tensor | None = None,
        output_attentions: bool | None = False,
        cu_seqlens: list[torch.Tensor] | None = None,
        rope_emb: tuple[torch.Tensor, torch.Tensor] | None = None,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split(
            [self.q_size, self.kv_size, self.kv_size],
            dim=-1,
        )

        max_seqlen = (cu_seqlens[1:] - cu_seqlens[:-1]).max()

        if rope_emb is None:
            q = q.view(*q.shape[:-1], self.num_heads, self.head_dim)
            k = k.view(
                *k.shape[:-1],
                self.num_kv_heads,
                self.head_dim,
            )
            v = v.view(
                *v.shape[:-1],
                self.num_kv_heads,
                self.head_dim,
            )
        else:
            if cu_seqlens is None:
                raise ValueError("cu_seqlens cannot be None when rope_emb is not None.")
            cos, sin = rope_emb
            q = q.view(*q.shape[:-1], self.num_heads, self.head_dim)
            k = k.view(
                *k.shape[:-1],
                self.num_kv_heads,
                self.head_dim,
            )
            q, k = apply_rotary_pos_emb_flashatt(q, k, cos, sin, self.apply_rotary_emb)
            v = v.view(
                *v.shape[:-1],
                self.num_kv_heads,
                self.head_dim,
            )

        context_layer = self.attn(
            query=q,
            key=k,
            value=v,
            cu_seqlens=cu_seqlens,
            max_seqlen=max_seqlen,
        )
        context_layer = rearrange(context_layer, "b s h d -> b s (h d)")

        output, _ = self.out_proj(context_layer)
        return output
```
**EN:** Method `KeyeSiglipAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `KeyeSiglipAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `SigLIPRotaryEmbedding` (lines 471-491)
```python
class SigLIPRotaryEmbedding(nn.Module):
    def __init__(self, dim: int, theta: float = 10000.0) -> None:
        super().__init__()
        self.dim = dim
        self.theta = theta
        self.rope_init()

    def rope_init(self):
        inv_freq = 1.0 / (
            self.theta ** (torch.arange(0, self.dim, 2, dtype=torch.float) / self.dim)
        )
        self.register_buffer("inv_freq", inv_freq, persistent=False)

    def forward(self, seqlen: int) -> torch.Tensor:
        seq = torch.arange(
            seqlen,
            device=self.inv_freq.device,
            dtype=self.inv_freq.dtype,
        )
        freqs = torch.outer(seq, self.inv_freq)
        return freqs
```
**EN:** Class `SigLIPRotaryEmbedding` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, rope_init, forward.
**CN:** 类 `SigLIPRotaryEmbedding` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, rope_init, forward。

### Method `SigLIPRotaryEmbedding.__init__` (lines 472-476)
```python
    def __init__(self, dim: int, theta: float = 10000.0) -> None:
        super().__init__()
        self.dim = dim
        self.theta = theta
        self.rope_init()
```
**EN:** Method `SigLIPRotaryEmbedding.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `SigLIPRotaryEmbedding.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `SigLIPRotaryEmbedding.rope_init` (lines 478-482)
```python
    def rope_init(self):
        inv_freq = 1.0 / (
            self.theta ** (torch.arange(0, self.dim, 2, dtype=torch.float) / self.dim)
        )
        self.register_buffer("inv_freq", inv_freq, persistent=False)
```
**EN:** Method `SigLIPRotaryEmbedding.rope_init` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `SigLIPRotaryEmbedding.rope_init` 封装了该模块中的一段可复用核心逻辑。

### Class `KeyeSiglipEncoderLayer` (lines 494-543)
```python
class KeyeSiglipEncoderLayer(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.embed_dim = config.hidden_size
        self.layer_norm1 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)
        self.self_attn = KeyeSiglipAttention(
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
        hidden_states: torch.Tensor,
```
**EN:** Class `KeyeSiglipEncoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `KeyeSiglipEncoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `KeyeSiglipEncoderLayer.__init__` (lines 495-514)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.embed_dim = config.hidden_size
        self.layer_norm1 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)
        self.self_attn = KeyeSiglipAttention(
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
**EN:** Method `KeyeSiglipEncoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KeyeSiglipEncoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KeyeSiglipEncoderLayer.forward` (lines 516-543)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        attention_mask: torch.Tensor,
        output_attentions: bool | None = False,
        cu_seqlens: list[torch.Tensor] | None = None,
        rope_emb: tuple[torch.Tensor, torch.Tensor] | None = None,
    ) -> tuple[torch.FloatTensor]:
        residual = hidden_states

        hidden_states = self.layer_norm1(hidden_states)
        hidden_states = self.self_attn(
            hidden_states=hidden_states,
            attention_mask=attention_mask,
            output_attentions=output_attentions,
            cu_seqlens=cu_seqlens,
            rope_emb=rope_emb,
        )

        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.layer_norm2(hidden_states)
        hidden_states = self.mlp(hidden_states)

        hidden_states = residual + hidden_states

        return hidden_states
```
**EN:** Method `KeyeSiglipEncoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `KeyeSiglipEncoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `KeyeSiglipEncoder` (lines 546-636)
```python
class KeyeSiglipEncoder(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        embed_dim = config.hidden_size
        num_heads = config.num_attention_heads
        head_dim = embed_dim // num_heads
        self.layers = nn.ModuleList(
            [
                KeyeSiglipEncoderLayer(
                    config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layers.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )
        self.rotary_pos_emb = SigLIPRotaryEmbedding(head_dim // 2)

    @staticmethod
```
**EN:** Class `KeyeSiglipEncoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, flatten_list, forward.
**CN:** 类 `KeyeSiglipEncoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, flatten_list, forward。

### Method `KeyeSiglipEncoder.__init__` (lines 547-568)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        embed_dim = config.hidden_size
        num_heads = config.num_attention_heads
        head_dim = embed_dim // num_heads
        self.layers = nn.ModuleList(
            [
                KeyeSiglipEncoderLayer(
                    config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layers.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )
        self.rotary_pos_emb = SigLIPRotaryEmbedding(head_dim // 2)
```
**EN:** Method `KeyeSiglipEncoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KeyeSiglipEncoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KeyeSiglipEncoder.flatten_list` (lines 571-578)
```python
    @staticmethod
    def flatten_list(image_grid_thw):
        tmp_image_grid_thw = list()
        for image_grid in image_grid_thw:
            if isinstance(image_grid, list):
                tmp_image_grid_thw.extend(image_grid)
            else:
                tmp_image_grid_thw.append(image_grid)
        return tmp_image_grid_thw
```
**EN:** Method `KeyeSiglipEncoder.flatten_list` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KeyeSiglipEncoder.flatten_list` 封装了该模块中的一段可复用核心逻辑。

### Class `KeyeSiglipVisionTransformer` (lines 639-714)
```python
class KeyeSiglipVisionTransformer(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        embed_dim = config.hidden_size

        self.embeddings = KeyeVisionEmbeddings(config)
        self.encoder = KeyeSiglipEncoder(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.encoder",
        )
        self.post_layernorm = nn.LayerNorm(embed_dim, eps=config.layer_norm_eps)

    def forward(
        self,
        pixel_values,
        output_attentions: bool | None = None,
        output_hidden_states: bool | None = None,
        interpolate_pos_encoding: bool | None = False,
```
**EN:** Class `KeyeSiglipVisionTransformer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `KeyeSiglipVisionTransformer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `KeyeSiglipVisionTransformer.__init__` (lines 640-656)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        embed_dim = config.hidden_size

        self.embeddings = KeyeVisionEmbeddings(config)
        self.encoder = KeyeSiglipEncoder(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.encoder",
        )
        self.post_layernorm = nn.LayerNorm(embed_dim, eps=config.layer_norm_eps)
```
**EN:** Method `KeyeSiglipVisionTransformer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KeyeSiglipVisionTransformer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KeyeSiglipVisionTransformer.forward` (lines 658-714)
```python
    def forward(
        self,
        pixel_values,
        output_attentions: bool | None = None,
        output_hidden_states: bool | None = None,
        interpolate_pos_encoding: bool | None = False,
        attention_mask: torch.Tensor | None = None,
        sample_indices: torch.Tensor | None = None,
        image_indices: torch.Tensor | None = None,
        position_ids: torch.Tensor | None = None,
        height_position_ids: torch.Tensor | None = None,
        width_position_ids: torch.Tensor | None = None,
        cu_seqlens: list[torch.Tensor] | None = None,
        padding_mask: torch.Tensor | None = None,
        vision_return_embed_list: bool | None = False,
        image_grid_thw: list[tuple[int, int, int] | list[tuple[int, int, int]]]
        | None = None,
        return_pooler_output: bool | None = True,
        use_rope: bool | None = False,
        window_size: bool | None = -1,
    ) -> BaseModelOutputWithPooling:
        hidden_states = self.embeddings(
            pixel_values,
            interpolate_pos_encoding=interpolate_pos_encoding,
            position_ids=position_ids,
            image_grid_thw=image_grid_thw,
        )

        last_hidden_state = self.encoder(
            inputs_embeds=hidden_states,
            output_attentions=output_attentions,
            output_hidden_states=output_hidden_states,
            attention_mask=attention_mask,
            cu_seqlens=cu_seqlens,
            image_grid_thw=image_grid_thw,
            use_rope=use_rope,
            height_position_ids=height_position_ids,
            width_position_ids=width_position_ids,
            window_size=window_size,
            vision_or_text="vision",
        )

        last_hidden_state = self.post_layernorm(last_hidden_state)

        sample_hidden_state = list()
        if cu_seqlens is None:
            raise ValueError(
                "cu_seqlens cannot be None for "
                "SiglipVisionTransformer output processing."
            )
        for i in range(cu_seqlens.shape[0] - 1):
            start = cu_seqlens[i]
            end = cu_seqlens[i + 1]
            tensor = last_hidden_state[:, start:end, :].squeeze(0)
            sample_hidden_state.append(tensor)

        return sample_hidden_state
```
**EN:** Method `KeyeSiglipVisionTransformer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `KeyeSiglipVisionTransformer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

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
- **Standard library / 标准库**: `import math`, `from abc import abstractmethod`, `from collections.abc import Iterable, Iterator, Mapping, Sequence`, `from functools import partial`, `from typing import Annotated, Any, Literal, TypeAlias, TypeVar`, `from einops import rearrange`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `import torch.nn as nn`, `from transformers import BaseImageProcessor, PretrainedConfig`, `from transformers.activations import GELUActivation`, `from transformers.feature_extraction_utils import BatchFeature`, `from transformers.modeling_outputs import BaseModelOutput, BaseModelOutputWithPooling`, `from transformers.utils import torch_int`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.distributed import get_tensor_model_parallel_world_size`, `from vllm.inputs import ModalityData, MultiModalDataDict`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.attention import (`, `from vllm.model_executor.layers.conv import Conv2dLayer`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding.common import (`, `from vllm.model_executor.model_loader.weight_utils import (`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`
