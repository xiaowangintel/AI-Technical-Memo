# ernie45_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/ernie45_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for ernie45_vl, including encoder/decoder glue and vLLM runtime adaptation. / 面向 ernie45_vl 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 26-86)
```python
import math
from collections.abc import Callable, Iterable, Iterator, Mapping, Sequence
from functools import partial
from typing import Annotated, Any, Literal

import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange
from transformers import BaseImageProcessor, BatchFeature

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions, VideoDummyOptions
from vllm.distributed import parallel_state
from vllm.distributed import utils as dist_utils
from vllm.inputs import MultiModalDataDict
from vllm.logger import init_logger
from vllm.model_executor.layers.activation import QuickGELU
from vllm.model_executor.layers.attention import (
    MMEncoderAttention,
)
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding.common import (
    ApplyRotaryEmb,
)
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import ImageSize, MultiModalDataItems, MultiModalDataParser
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape
from vllm.v1.attention.backends.registry import AttentionBackendEnum

from .ernie45_vl_moe import Ernie4_5_VLMoeForCausalLM
from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMRoPE,
    SupportsMultiModal,
    SupportsPP,
)
from .utils import AutoWeightsLoader, WeightsMapper, maybe_prefix
from .vision import get_vit_attn_backend
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `all_gather_interleave` (lines 93-109)
```python
def all_gather_interleave(local_tensor, hidden_size: int, tp_size: int):
    """All-gather the input tensor interleavely across model parallel group."""
    import torch.distributed as dist

    gathered_tensors = [torch.zeros_like(local_tensor) for _ in range(tp_size)]
    dist.all_gather(
        gathered_tensors, local_tensor, group=parallel_state.get_tp_group().device_group
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

### Function `round_by_factor` (lines 550-551)
```python
def round_by_factor(number: int | float, factor: int) -> int:
    return round(number / factor) * factor
```
**EN:** Function `round_by_factor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `round_by_factor` 封装了该模块中的一段可复用核心逻辑。

### Function `ceil_by_factor` (lines 554-555)
```python
def ceil_by_factor(number: int | float, factor: int) -> int:
    return math.ceil(number / factor) * factor
```
**EN:** Function `ceil_by_factor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `ceil_by_factor` 封装了该模块中的一段可复用核心逻辑。

### Function `floor_by_factor` (lines 558-559)
```python
def floor_by_factor(number: int | float, factor: int) -> int:
    return math.floor(number / factor) * factor
```
**EN:** Function `floor_by_factor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `floor_by_factor` 封装了该模块中的一段可复用核心逻辑。

### Function `smart_resize` (lines 562-599)
```python
def smart_resize(
    height: int,
    width: int,
    factor: int = 28,
    min_pixels: int = 4 * 28 * 28,
    max_pixels: int = 16384 * 28 * 28,
):
    MAX_RATIO = 200
    if max(height, width) / min(height, width) > MAX_RATIO:
        if height > width:
            new_width = max(factor, round_by_factor(width, factor))
            new_height = floor_by_factor(new_width * MAX_RATIO, factor)
        else:
            new_height = max(factor, round_by_factor(height, factor))
            new_width = floor_by_factor(new_height * MAX_RATIO, factor)

        height = new_height
        width = new_width

    h_bar = max(factor, round_by_factor(height, factor))
    w_bar = max(factor, round_by_factor(width, factor))
    if h_bar * w_bar > max_pixels:
        beta = math.sqrt((height * width) / max_pixels)
        h_bar = floor_by_factor(height / beta, factor)
        w_bar = floor_by_factor(width / beta, factor)
    elif h_bar * w_bar < min_pixels:
        beta = math.sqrt(min_pixels / (height * width))
        h_bar = ceil_by_factor(height * beta, factor)
        w_bar = ceil_by_factor(width * beta, factor)

    if min_pixels > h_bar * w_bar or h_bar * w_bar > max_pixels:
        raise ValueError(
            f"Invalid h_bar={h_bar}, w_bar={w_bar}: "
            f"h_bar * w_bar must be >= min_pixels ({min_pixels}) "
            f"and <= max_pixels ({max_pixels})."
        )

    return h_bar, w_bar
```
**EN:** Function `smart_resize` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `smart_resize` 封装了该模块中的一段可复用核心逻辑。

### Class `Ernie4_5_VisionAttention` (lines 112-223)
```python
class Ernie4_5_VisionAttention(nn.Module):
    """VisionAttention using VLLM framework APIs"""

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
        self.tp_size = parallel_state.get_tensor_model_parallel_world_size()
        self.tp_rank = parallel_state.get_tensor_model_parallel_rank()
        self.hidden_size_per_attention_head = dist_utils.divide(
            projection_size, num_heads
        )
        self.num_attention_heads_per_partition = dist_utils.divide(
            num_heads, self.tp_size
        )

        self.qkv = QKVParallelLinear(
            hidden_size=embed_dim,
            head_size=self.hidden_size_per_attention_head,
```
**EN:** Class `Ernie4_5_VisionAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, split_qkv, forward.
**CN:** 类 `Ernie4_5_VisionAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, split_qkv, forward。

### Method `Ernie4_5_VisionAttention.__init__` (lines 115-160)
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
        self.tp_size = parallel_state.get_tensor_model_parallel_world_size()
        self.tp_rank = parallel_state.get_tensor_model_parallel_rank()
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
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv",
        )
        self.proj = RowParallelLinear(
            input_size=projection_size,
            output_size=embed_dim,
            quant_config=quant_config,
            prefix=f"{prefix}.proj",
        )

        self.attn = MMEncoderAttention(
            num_heads=self.num_attention_heads_per_partition,
            head_size=self.hidden_size_per_attention_head,
            scale=self.hidden_size_per_attention_head**-0.5,
            prefix=f"{prefix}.attn",
        )

        self.apply_rotary_emb = ApplyRotaryEmb(
            enforce_enable=True,
            enable_fp32_compute=True,
        )
```
**EN:** Method `Ernie4_5_VisionAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Ernie4_5_VisionAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Ernie4_5_VisionAttention.split_qkv` (lines 162-188)
```python
    def split_qkv(self, qkv: torch.Tensor) -> tuple[torch.Tensor, ...]:
        # [s, b, 3 * head * head_dim]
        seq_len, bs, _ = qkv.shape
        if self.tp_size > 1:
            qkv = all_gather_interleave(qkv, self.qkv.hidden_size, self.tp_size)

        # [s, b, 3 * head * head_dim] -> 3 * [s, b, head * head_dim]
        q, k, v = qkv.chunk(3, dim=2)

        # 3 * [s, b, head * head_dim]
        if self.tp_size > 1:
            splitter = partial(
                dist_utils.split_tensor_along_last_dim, num_partitions=self.tp_size
            )
            q = splitter(q)[self.tp_rank]
            k = splitter(k)[self.tp_rank]
            v = splitter(v)[self.tp_rank]

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
**EN:** Method `Ernie4_5_VisionAttention.split_qkv` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Ernie4_5_VisionAttention.split_qkv` 封装了该模块中的一段可复用核心逻辑。

### Class `Ernie4_5_VisionMLP` (lines 226-254)
```python
class Ernie4_5_VisionMLP(nn.Module):
    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        act_layer: type[nn.Module] = QuickGELU,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.fc1 = ColumnParallelLinear(
            in_features,
            hidden_features,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
        )
        self.act = act_layer()
        self.fc2 = RowParallelLinear(
            hidden_features,
            in_features,
            quant_config=quant_config,
            prefix=f"{prefix}.fc2",
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
```
**EN:** Class `Ernie4_5_VisionMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Ernie4_5_VisionMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Ernie4_5_VisionMLP.__init__` (lines 227-248)
```python
    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        act_layer: type[nn.Module] = QuickGELU,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.fc1 = ColumnParallelLinear(
            in_features,
            hidden_features,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
        )
        self.act = act_layer()
        self.fc2 = RowParallelLinear(
            hidden_features,
            in_features,
            quant_config=quant_config,
            prefix=f"{prefix}.fc2",
        )
```
**EN:** Method `Ernie4_5_VisionMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Ernie4_5_VisionMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Ernie4_5_VisionMLP.forward` (lines 250-254)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x_parallel, _ = self.fc1(x)
        x_parallel = self.act(x_parallel)
        x, _ = self.fc2(x_parallel)
        return x
```
**EN:** Method `Ernie4_5_VisionMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Ernie4_5_VisionMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Ernie4_5_VisionBlock` (lines 257-306)
```python
class Ernie4_5_VisionBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        num_heads: int,
        mlp_ratio: float,
        act_layer: type[nn.Module] = QuickGELU,
        norm_layer: Callable[[int], nn.Module] | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm1 = norm_layer(dim)
        self.norm2 = norm_layer(dim)
        mlp_hidden_dim = int(dim * mlp_ratio)

        self.attn = Ernie4_5_VisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            projection_size=dim,
            quant_config=quant_config,
            prefix=f"{prefix}.attn",
```
**EN:** Class `Ernie4_5_VisionBlock` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Ernie4_5_VisionBlock` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Ernie4_5_VisionBlock.__init__` (lines 258-290)
```python
    def __init__(
        self,
        dim: int,
        num_heads: int,
        mlp_ratio: float,
        act_layer: type[nn.Module] = QuickGELU,
        norm_layer: Callable[[int], nn.Module] | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm1 = norm_layer(dim)
        self.norm2 = norm_layer(dim)
        mlp_hidden_dim = int(dim * mlp_ratio)

        self.attn = Ernie4_5_VisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            projection_size=dim,
            quant_config=quant_config,
            prefix=f"{prefix}.attn",
        )

        self.mlp = Ernie4_5_VisionMLP(
            dim,
            mlp_hidden_dim,
            act_layer=act_layer,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )
```
**EN:** Method `Ernie4_5_VisionBlock.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Ernie4_5_VisionBlock.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Ernie4_5_VisionBlock.forward` (lines 292-306)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        rotary_pos_emb: torch.Tensor,
        max_seqlen: torch.Tensor | None = None,  # Only used for Flash Attention
    ) -> torch.Tensor:
        hidden_states = hidden_states + self.attn(
            self.norm1(hidden_states),
            cu_seqlens=cu_seqlens,
            rotary_pos_emb=rotary_pos_emb,
            max_seqlen=max_seqlen,
        )
        hidden_states = hidden_states + self.mlp(self.norm2(hidden_states))
        return hidden_states
```
**EN:** Method `Ernie4_5_VisionBlock.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Ernie4_5_VisionBlock.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Ernie4_5_VisionPatchEmbed` (lines 309-331)
```python
class Ernie4_5_VisionPatchEmbed(nn.Module):
    def __init__(
        self,
        patch_size: int = 14,
        in_channels: int = 3,
        embed_dim: int = 1280,
        prefix="",
    ) -> None:
        super().__init__()
        self.patch_size = patch_size
        self.in_channels = in_channels
        self.embed_dim = embed_dim

        self.proj = nn.Linear(
            in_channels * patch_size * patch_size, embed_dim, bias=False
        )

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        target_dtype = self.proj.weight.dtype
        hidden_states = hidden_states.to(target_dtype)
        hidden_states = self.proj(hidden_states)

        return hidden_states
```
**EN:** Class `Ernie4_5_VisionPatchEmbed` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Ernie4_5_VisionPatchEmbed` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Ernie4_5_VisionPatchEmbed.__init__` (lines 310-324)
```python
    def __init__(
        self,
        patch_size: int = 14,
        in_channels: int = 3,
        embed_dim: int = 1280,
        prefix="",
    ) -> None:
        super().__init__()
        self.patch_size = patch_size
        self.in_channels = in_channels
        self.embed_dim = embed_dim

        self.proj = nn.Linear(
            in_channels * patch_size * patch_size, embed_dim, bias=False
        )
```
**EN:** Method `Ernie4_5_VisionPatchEmbed.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Ernie4_5_VisionPatchEmbed.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Ernie4_5_VisionPatchEmbed.forward` (lines 326-331)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        target_dtype = self.proj.weight.dtype
        hidden_states = hidden_states.to(target_dtype)
        hidden_states = self.proj(hidden_states)

        return hidden_states
```
**EN:** Method `Ernie4_5_VisionPatchEmbed.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Ernie4_5_VisionPatchEmbed.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Ernie4_5_VisionRotaryEmbedding` (lines 334-346)
```python
class Ernie4_5_VisionRotaryEmbedding(nn.Module):
    def __init__(self, dim: int, theta: float = 10000.0) -> None:
        super().__init__()
        self.inv_freq = 1.0 / theta ** (
            torch.arange(start=0, end=dim, step=2, dtype=torch.float32) / dim
        )

    def forward(self, seqlen: int) -> torch.Tensor:
        seq = torch.arange(
            seqlen, device=self.inv_freq.device, dtype=self.inv_freq.dtype
        )
        freqs = torch.outer(input=seq, vec2=self.inv_freq)
        return freqs
```
**EN:** Class `Ernie4_5_VisionRotaryEmbedding` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Ernie4_5_VisionRotaryEmbedding` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Ernie4_5_VisionRotaryEmbedding.__init__` (lines 335-339)
```python
    def __init__(self, dim: int, theta: float = 10000.0) -> None:
        super().__init__()
        self.inv_freq = 1.0 / theta ** (
            torch.arange(start=0, end=dim, step=2, dtype=torch.float32) / dim
        )
```
**EN:** Method `Ernie4_5_VisionRotaryEmbedding.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Ernie4_5_VisionRotaryEmbedding.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Ernie4_5_VisionRotaryEmbedding.forward` (lines 341-346)
```python
    def forward(self, seqlen: int) -> torch.Tensor:
        seq = torch.arange(
            seqlen, device=self.inv_freq.device, dtype=self.inv_freq.dtype
        )
        freqs = torch.outer(input=seq, vec2=self.inv_freq)
        return freqs
```
**EN:** Method `Ernie4_5_VisionRotaryEmbedding.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Ernie4_5_VisionRotaryEmbedding.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Ernie4_5_VisionTransformer` (lines 349-506)
```python
class Ernie4_5_VisionTransformer(nn.Module):
    def __init__(
        self,
        vision_config,
        norm_eps: float = 1e-6,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        patch_size = vision_config.patch_size
        spatial_merge_size = vision_config.spatial_merge_size
        in_channels = vision_config.in_channels
        hidden_size = vision_config.hidden_size
        embed_dim = vision_config.embed_dim
        depth = vision_config.depth
        num_heads = vision_config.num_heads
        mlp_ratio = vision_config.mlp_ratio

        self.spatial_merge_size = spatial_merge_size
        self.num_heads = num_heads
        self.embed_dim = embed_dim

        self.patch_embed = Ernie4_5_VisionPatchEmbed(
            patch_size=patch_size,
            in_channels=in_channels,
```
**EN:** Class `Ernie4_5_VisionTransformer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, dtype, device, rot_pos_emb, compute_attn_mask_seqlen, forward.
**CN:** 类 `Ernie4_5_VisionTransformer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, dtype, device, rot_pos_emb, compute_attn_mask_seqlen, forward。

### Method `Ernie4_5_VisionTransformer.__init__` (lines 350-404)
```python
    def __init__(
        self,
        vision_config,
        norm_eps: float = 1e-6,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        patch_size = vision_config.patch_size
        spatial_merge_size = vision_config.spatial_merge_size
        in_channels = vision_config.in_channels
        hidden_size = vision_config.hidden_size
        embed_dim = vision_config.embed_dim
        depth = vision_config.depth
        num_heads = vision_config.num_heads
        mlp_ratio = vision_config.mlp_ratio

        self.spatial_merge_size = spatial_merge_size
        self.num_heads = num_heads
        self.embed_dim = embed_dim

        self.patch_embed = Ernie4_5_VisionPatchEmbed(
            patch_size=patch_size,
            in_channels=in_channels,
            embed_dim=embed_dim,
            prefix=f"{prefix}.patch_embed",
        )

        norm_layer = partial(nn.LayerNorm, eps=norm_eps)
        head_dim = embed_dim // num_heads
        self.rotary_pos_emb = Ernie4_5_VisionRotaryEmbedding(head_dim // 2)

        self.blocks = nn.ModuleList(
            [
                Ernie4_5_VisionBlock(
                    dim=embed_dim,
                    num_heads=num_heads,
                    mlp_ratio=mlp_ratio,
                    norm_layer=norm_layer,
                    quant_config=quant_config,
                    prefix=f"{prefix}.blocks.{layer_idx}",
                )
                for layer_idx in range(depth)
            ]
        )

        assert hidden_size == embed_dim, (
            "vit's config.hidden must be equal to config.embed_dim"
        )
        self.ln = nn.LayerNorm(hidden_size, eps=1e-6)

        self.attn_backend = get_vit_attn_backend(
            head_size=head_dim,
            dtype=torch.get_default_dtype(),
        )
```
**EN:** Method `Ernie4_5_VisionTransformer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Ernie4_5_VisionTransformer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Ernie4_5_VisionTransformer.dtype` (lines 407-408)
```python
    @property
    def dtype(self) -> torch.dtype:
        return self.patch_embed.proj.weight.dtype
```
**EN:** Method `Ernie4_5_VisionTransformer.dtype` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Ernie4_5_VisionTransformer.dtype` 封装了该模块中的一段可复用核心逻辑。

### Class `Ernie4_5_VLImagePixelInputs` (lines 512-524)
```python
class Ernie4_5_VLImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - np: The total number of patches over each image over each prompt in
              the batch
        - ni: Number of images
        - cps: Number of channels * patch_size * patch_size
    """

    type: Literal["pixel_values"]

    pixel_values: Annotated[torch.Tensor, TensorShape("np", "cps")]
    image_grid_thw: Annotated[torch.Tensor, TensorShape("ni", 3)]
```
**EN:** Class `Ernie4_5_VLImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Ernie4_5_VLImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Ernie4_5_VLVideoPixelInputs` (lines 530-542)
```python
class Ernie4_5_VLVideoPixelInputs(TensorSchema):
    """
    Dimensions:
        - np: The total number of patches over each image over each prompt in
              the batch
        - ni: Number of images
        - cps: Number of channels * temporal_patch_size * patch_size *
              patch_size
    """

    type: Literal["pixel_values_videos"]
    pixel_values_videos: Annotated[torch.Tensor, TensorShape("np", "cps")]
    video_grid_thw: Annotated[torch.Tensor, TensorShape("ni", 3)]
```
**EN:** Class `Ernie4_5_VLVideoPixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Ernie4_5_VLVideoPixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `VariableResolutionResamplerModel` (lines 602-783)
```python
class VariableResolutionResamplerModel(nn.Module):
    def __init__(
        self,
        in_dim,
        out_dim,
        spatial_conv_size,
        temporal_conv_size,
        config,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.in_dim = in_dim
        self.out_dim = out_dim
        self.config = config
        self.spatial_conv_size = spatial_conv_size
        self.temporal_conv_size = temporal_conv_size
        self.use_temporal_conv = config.use_temporal_conv

        # compress 2d conv(picture) to 1d
        self.spatial_dim = self.in_dim * self.spatial_conv_size * self.spatial_conv_size
        # compress 3d conv(video) to 1d
        self.temporal_dim = (
            self.in_dim
            * self.spatial_conv_size
            * self.spatial_conv_size
```
**EN:** Class `VariableResolutionResamplerModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, spatial_conv_reshape, forward, load_weights.
**CN:** 类 `VariableResolutionResamplerModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, spatial_conv_reshape, forward, load_weights。

### Method `VariableResolutionResamplerModel.__init__` (lines 603-686)
```python
    def __init__(
        self,
        in_dim,
        out_dim,
        spatial_conv_size,
        temporal_conv_size,
        config,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.in_dim = in_dim
        self.out_dim = out_dim
        self.config = config
        self.spatial_conv_size = spatial_conv_size
        self.temporal_conv_size = temporal_conv_size
        self.use_temporal_conv = config.use_temporal_conv

        # compress 2d conv(picture) to 1d
        self.spatial_dim = self.in_dim * self.spatial_conv_size * self.spatial_conv_size
        # compress 3d conv(video) to 1d
        self.temporal_dim = (
            self.in_dim
            * self.spatial_conv_size
            * self.spatial_conv_size
            * self.temporal_conv_size
        )

        self.spatial_linear1 = ColumnParallelLinear(
            self.spatial_dim,
            self.spatial_dim,
            bias=True,
            gather_output=True,
            quant_config=getattr(config, "quant_config", None),
            prefix=f"{prefix}.spatial_linear1",
        )

        self.spatial_gelu = nn.GELU()

        self.spatial_linear2 = ColumnParallelLinear(
            self.spatial_dim,
            self.spatial_dim,
            bias=True,
            gather_output=True,
            quant_config=getattr(config, "quant_config", None),
            prefix=f"{prefix}.spatial_linear2",
        )

        self.spatial_norm = nn.LayerNorm(self.spatial_dim, eps=1e-6)

        if self.use_temporal_conv:
            self.temporal_linear1 = ColumnParallelLinear(
                self.temporal_dim,
                self.spatial_dim,
                bias=True,
                gather_output=True,
                quant_config=getattr(config, "quant_config", None),
                prefix=f"{prefix}.temporal_linear1",
            )

            self.temporal_gelu = nn.GELU()
# ... truncated for analysis ...
```
**EN:** Method `VariableResolutionResamplerModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `VariableResolutionResamplerModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `VariableResolutionResamplerModel.spatial_conv_reshape` (lines 688-691)
```python
    def spatial_conv_reshape(self, x, spatial_conv_size):
        S, C = x.shape
        x = x.reshape([-1, C * (spatial_conv_size**2)])
        return x
```
**EN:** Method `VariableResolutionResamplerModel.spatial_conv_reshape` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `VariableResolutionResamplerModel.spatial_conv_reshape` 封装了该模块中的一段可复用核心逻辑。

### Class `Ernie4_5_VLProcessingInfo` (lines 786-982)
```python
class Ernie4_5_VLProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.model_config.hf_config

    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(use_fast=True, **kwargs)

    def get_image_processor(self, **kwargs: object):
        return self.get_hf_processor(**kwargs).image_processor

    def get_data_parser(self):
        return MultiModalDataParser(
            video_needs_metadata=True,
            expected_hidden_size=self._get_expected_hidden_size(),
        )

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None, "video": None}

    def get_mm_max_tokens_per_item(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
    ) -> Mapping[str, int]:
        max_image_tokens = self.get_max_image_tokens()
```
**EN:** Class `Ernie4_5_VLProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_hf_processor, get_image_processor, get_data_parser, get_supported_mm_limits, get_mm_max_tokens_per_item.
**CN:** 类 `Ernie4_5_VLProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor, get_image_processor, get_data_parser, get_supported_mm_limits, get_mm_max_tokens_per_item。

### Method `Ernie4_5_VLProcessingInfo.get_hf_config` (lines 787-788)
```python
    def get_hf_config(self):
        return self.ctx.model_config.hf_config
```
**EN:** Method `Ernie4_5_VLProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Ernie4_5_VLProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `Ernie4_5_VLProcessingInfo.get_hf_processor` (lines 790-791)
```python
    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(use_fast=True, **kwargs)
```
**EN:** Method `Ernie4_5_VLProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Ernie4_5_VLProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

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
- **Standard library / 标准库**: `import math`, `from collections.abc import Callable, Iterable, Iterator, Mapping, Sequence`, `from functools import partial`, `from typing import Annotated, Any, Literal`, `from einops import rearrange`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `import torch.nn as nn`, `import torch.nn.functional as F`, `from transformers import BaseImageProcessor, BatchFeature`, `import torch.distributed as dist`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions, VideoDummyOptions`, `from vllm.distributed import parallel_state`, `from vllm.distributed import utils as dist_utils`, `from vllm.inputs import MultiModalDataDict`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.activation import QuickGELU`, `from vllm.model_executor.layers.attention import (`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding.common import (`
- **Module note / 模块说明**: **EN:** Inference-only Ernie VL model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only Ernie VL model compatible with HuggingFace weights.。
