# qwen3_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen3_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only Qwen3-VL model compatible with HuggingFace weights. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 15-15: Module docstring
```python
"""Inference-only Qwen3-VL model compatible with HuggingFace weights."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 17-82: Module imports
```python
import logging
import re
from collections import defaultdict
from functools import lru_cache, partial
from typing import Callable, Iterable, List, Optional, Tuple, Union

import numpy as np
import torch
import torch.nn as nn
from einops import rearrange
from transformers.activations import ACT2FN

from sglang.srt.configs.qwen3_vl import Qwen3VLConfig, Qwen3VLVisionConfig
from sglang.srt.distributed import get_tensor_model_parallel_world_size
from sglang.srt.distributed.parallel_state import get_pp_group
from sglang.srt.environ import envs
from sglang.srt.layers.attention.vision import (
    BATCH_BUCKETS,
    FLASHINFER_MAX_SEQLEN_BUCKETS,
    FLASHINFER_WORKSPACE_SIZE_BYTES,
    VisionAttention,
)
from sglang.srt.layers.conv import Conv3dLayer
from sglang.srt.layers.dp_attention import (
    get_attention_tp_rank,
    get_attention_tp_size,
    is_dp_attention_enabled,
)
from sglang.srt.layers.linear import ColumnParallelLinear, RowParallelLinear
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.pooler import Pooler, PoolingType
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.rotary_embedding import get_rope
from sglang.srt.layers.utils import PPMissingLayer, get_layer_id
from sglang.srt.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.managers.mm_utils import (
    MultiModalityDataPaddingPatternMultimodalTokens,
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 84-84: Top-level assign
```python
_is_npu = is_npu()
```
**EN:** Defines or updates _is_npu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_npu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 85-85: Top-level assign
```python
graph_runners_dict = defaultdict(lambda: ViTCudaGraphRunner)
```
**EN:** Defines or updates graph_runners_dict, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 graph_runners_dict，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 86-91: Top-level if
```python
if _is_npu:
    from sglang.srt.hardware_backend.npu.graph_runner.vit_npu_graph_runner import (
        ViTNpuGraphRunner,
    )

    graph_runners_dict["npu"] = ViTNpuGraphRunner
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 94-94: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 96-96: Top-level assign
```python
_is_cpu_amx_available = cpu_has_amx_support()
```
**EN:** Defines or updates _is_cpu_amx_available, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cpu_amx_available，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 97-97: Top-level assign
```python
_is_cpu = is_cpu()
```
**EN:** Defines or updates _is_cpu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cpu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 100-101: Class `Qwen3_VisionMLP` overview
```python
class Qwen3_VisionMLP(nn.Module):
```
**EN:** Defines `Qwen3_VisionMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3_VisionMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 102-134: Method `Qwen3_VisionMLP.__init__`
```python
    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        bias: bool = True,
        hidden_act="silu",
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        use_data_parallel: bool = False,
    ):
        super().__init__()
        self.tp_size = 1 if use_data_parallel else get_attention_tp_size()
        self.tp_rank = 0 if use_data_parallel else get_attention_tp_rank()
        self.linear_fc1 = ColumnParallelLinear(
            in_features,
            hidden_features,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("linear_fc1", prefix),
            tp_size=self.tp_size,
            tp_rank=self.tp_rank,
        )
        self.linear_fc2 = RowParallelLinear(
            hidden_features,
            in_features,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("linear_fc2", prefix),
            tp_size=self.tp_size,
            tp_rank=self.tp_rank,
            use_dp_attention_reduce=is_dp_attention_enabled(),
        )
        self.act = ACT2FN[hidden_act]
```
**EN:** This method implements `__init__(in_features: ..., hidden_features: ..., bias: ...=..., hidden_act=..., quant_config: ...=..., prefix: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(in_features: ..., hidden_features: ..., bias: ...=..., hidden_act=..., quant_config: ...=..., prefix: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 136-139: Method `Qwen3_VisionMLP.forward`
```python
    def forward(self, x: torch.Tensor):
        x_fc1, _ = self.linear_fc1(x)
        mlp_output, _ = self.linear_fc2(self.act(x_fc1))
        return mlp_output
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 142-142: Class `Qwen3VLVisionPatchEmbed` overview
```python
class Qwen3VLVisionPatchEmbed(nn.Module):
```
**EN:** Defines `Qwen3VLVisionPatchEmbed` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3VLVisionPatchEmbed`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 143-157: Method `Qwen3VLVisionPatchEmbed.__init__`
```python
    def __init__(self, config) -> None:
        super().__init__()
        self.patch_size = config.patch_size
        self.temporal_patch_size = config.temporal_patch_size
        self.in_channels = config.in_channels
        self.embed_dim = config.hidden_size

        kernel_size = [self.temporal_patch_size, self.patch_size, self.patch_size]
        self.proj = Conv3dLayer(
            self.in_channels,
            self.embed_dim,
            kernel_size=kernel_size,
            stride=kernel_size,
            bias=True,
        )
```
**EN:** This method implements `__init__(config)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 159-171: Method `Qwen3VLVisionPatchEmbed.forward`
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        target_dtype = self.proj.weight.dtype
        hidden_states = hidden_states.view(
            -1,
            self.in_channels,
            self.temporal_patch_size,
            self.patch_size,
            self.patch_size,
        )
        hidden_states = self.proj(hidden_states.to(dtype=target_dtype)).view(
            -1, self.embed_dim
        )
        return hidden_states
```
**EN:** This method implements `forward(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 174-175: Class `Qwen3_VisionBlock` overview
```python
class Qwen3_VisionBlock(nn.Module):
```
**EN:** Defines `Qwen3_VisionBlock` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3_VisionBlock`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 176-217: Method `Qwen3_VisionBlock.__init__`
```python
    def __init__(
        self,
        dim: int,
        num_heads: int,
        intermediate_dim: int,
        head_size: Optional[int] = None,
        hidden_act="silu",
        norm_layer: Optional[Callable[[int], nn.Module]] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        use_data_parallel: bool = False,
        workspace_buffer: torch.Tensor | None = None,
    ) -> None:
        super().__init__()
        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm1 = norm_layer(dim)
        self.norm2 = norm_layer(dim)

        self.attn = VisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            head_size=head_size,
            projection_size=num_heads * head_size,
            use_qkv_parallel=True,
            proj_bias=True,
            flatten_batch=True,
            quant_config=quant_config,
            prefix=add_prefix("attn", prefix),
            use_data_parallel=use_data_parallel,
            use_dp_attention_reduce=is_dp_attention_enabled(),
            workspace_buffer=workspace_buffer,
        )
        self.mlp = Qwen3_VisionMLP(
            dim,
            intermediate_dim,
            hidden_act=hidden_act,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(dim: ..., num_heads: ..., intermediate_dim: ..., head_size: ...=..., hidden_act=..., norm_layer: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., num_heads: ..., intermediate_dim: ..., head_size: ...=..., hidden_act=..., norm_layer: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 219-245: Method `Qwen3_VisionBlock.forward`
```python
    def forward(
        self,
        x: torch.Tensor,
        cu_seqlens: torch.Tensor,
        rotary_pos_emb_cos: torch.Tensor,
        rotary_pos_emb_sin: torch.Tensor,
        output_ws: Optional[torch.Tensor] = None,
        max_seqlen: Optional[torch.Tensor] = None,
        sequence_lengths: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        hidden_states = self.norm1(x)
        hidden_states = rearrange(hidden_states, "s b ... -> b s ...")
        attn = self.attn(
            hidden_states,
            cu_seqlens=cu_seqlens,
            rotary_pos_emb_cos=rotary_pos_emb_cos,
            rotary_pos_emb_sin=rotary_pos_emb_sin,
            output_ws=output_ws,
            max_seqlen=max_seqlen,
            sequence_lengths=sequence_lengths,
        )
        attn = rearrange(attn, "b s ... -> s b ...")
        x += attn
        norm2 = self.norm2(x)
        mlp = self.mlp(norm2)
        x += mlp
        return x
```
**EN:** This method implements `forward(x: ..., cu_seqlens: ..., rotary_pos_emb_cos: ..., rotary_pos_emb_sin: ..., output_ws: ...=..., max_seqlen: ...=..., ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ..., cu_seqlens: ..., rotary_pos_emb_cos: ..., rotary_pos_emb_sin: ..., output_ws: ...=..., max_seqlen: ...=..., ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 248-249: Class `Qwen3VLMoeVisionPatchMerger` overview
```python
class Qwen3VLMoeVisionPatchMerger(nn.Module):
```
**EN:** Defines `Qwen3VLMoeVisionPatchMerger` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3VLMoeVisionPatchMerger`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 250-294: Method `Qwen3VLMoeVisionPatchMerger.__init__`
```python
    def __init__(
        self,
        dim: int,
        context_dim: int,
        padded_context_dim: int,
        norm_layer: Optional[Callable[[int], nn.Module]] = None,
        spatial_merge_size: int = 2,
        use_postshuffle_norm: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        use_data_parallel: bool = False,
    ) -> None:
        super().__init__()
        self.hidden_size = context_dim * (spatial_merge_size**2)
        self.padded_context_dim = padded_context_dim * (spatial_merge_size**2)

        self.use_postshuffle_norm = use_postshuffle_norm

        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm = norm_layer(
            self.hidden_size if use_postshuffle_norm else context_dim
        )
        self.tp_size = 1 if use_data_parallel else get_attention_tp_size()
        self.tp_rank = 0 if use_data_parallel else get_attention_tp_rank()
        self.linear_fc1 = ColumnParallelLinear(
            self.hidden_size,
            self.padded_context_dim,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("linear_fc1", prefix),
            tp_size=self.tp_size,
            tp_rank=self.tp_rank,
        )
        self.act_fn = nn.GELU()
        self.linear_fc2 = RowParallelLinear(
            self.padded_context_dim,
            dim,
            bias=True,
            quant_config=quant_config,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(dim: ..., context_dim: ..., padded_context_dim: ..., norm_layer: ...=..., spatial_merge_size: ...=..., use_postshuffle_norm: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., context_dim: ..., padded_context_dim: ..., norm_layer: ...=..., spatial_merge_size: ...=..., use_postshuffle_norm: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 296-305: Method `Qwen3VLMoeVisionPatchMerger.forward`
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        if self.use_postshuffle_norm:
            x = self.norm(x.view(-1, self.hidden_size))
        else:
            x = self.norm(x).view(-1, self.hidden_size)

        x_parallel, _ = self.linear_fc1(x)
        x_parallel = self.act_fn(x_parallel)
        out, _ = self.linear_fc2(x_parallel)
        return out
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 308-309: Class `Qwen3VLMoeVisionModel` overview
```python
class Qwen3VLMoeVisionModel(nn.Module, RotaryPosMixin):
```
**EN:** Defines `Qwen3VLMoeVisionModel` as a reusable runtime type derived from nn.Module, RotaryPosMixin. The class groups 20 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3VLMoeVisionModel`，其继承关系为 nn.Module, RotaryPosMixin。这个类组织了 20 个方法，用于实现模型相关行为。

### Lines 310-424: Method `Qwen3VLMoeVisionModel.__init__`
```python
    def __init__(
        self,
        vision_config: Qwen3VLVisionConfig,
        norm_eps: float = 1e-6,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        use_data_parallel: bool = False,
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.hidden_size = vision_config.hidden_size
        self.num_heads = vision_config.num_heads
        self.num_position_embeddings = vision_config.num_position_embeddings
        self.num_grid_per_side = int(self.num_position_embeddings**0.5)
        self.num_grid = self.num_grid_per_side * self.num_grid_per_side
        self.align_corners = (
            get_global_server_args().enable_precise_embedding_interpolation
        )
        self.patch_size = vision_config.patch_size
        self.spatial_merge_size = vision_config.spatial_merge_size
        self.spatial_merge_unit = self.spatial_merge_size**2
        self.temporal_patch_size = vision_config.temporal_patch_size
        self.use_data_parallel = use_data_parallel
        # layer indexes of which layer's output should be deep-stacked
        self.deepstack_visual_indexes = vision_config.deepstack_visual_indexes
        self.out_hidden_size = vision_config.out_hidden_size * (
            1 + len(self.deepstack_visual_indexes)
        )
        self.patch_embed = Qwen3VLVisionPatchEmbed(config=vision_config)
        if self.pp_group.is_first_rank:
            self.pos_embed = VocabParallelEmbedding(
                self.num_position_embeddings,
                self.hidden_size,
                quant_config=quant_config,
                enable_tp=not use_data_parallel,
                use_attn_tp_group=is_dp_attention_enabled() and not use_data_parallel,
                prefix=add_prefix("pos_embed", prefix),
            )
        else:
            self.pos_embed = PPMissingLayer()
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(vision_config: ..., norm_eps: ...=..., quant_config: ...=..., prefix: ...=..., use_data_parallel: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(vision_config: ..., norm_eps: ...=..., quant_config: ...=..., prefix: ...=..., use_data_parallel: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 426-428: Method `Qwen3VLMoeVisionModel.dtype`
```python
    @property
    def dtype(self) -> torch.dtype:
        return self.patch_embed.proj.weight.dtype
```
**EN:** This method implements `dtype()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLMoeVisionModel` Decorators: property.
**CN:** 这个方法实现了 `dtype()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLMoeVisionModel` 内部调用 装饰器：property。

### Lines 430-432: Method `Qwen3VLMoeVisionModel.device`
```python
    @property
    def device(self) -> torch.device:
        return self.patch_embed.proj.weight.device
```
**EN:** This method implements `device()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLMoeVisionModel` Decorators: property.
**CN:** 这个方法实现了 `device()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLMoeVisionModel` 内部调用 装饰器：property。

### Lines 434-451: Method `Qwen3VLMoeVisionModel.rot_pos_emb`
```python
    def rot_pos_emb(
        self, grid_thw: list[list[int]]
    ) -> tuple[torch.Tensor, torch.Tensor]:
        pos_ids = []
        for t, h, w in grid_thw:
            base = self.rot_pos_ids(h, w, self.spatial_merge_size)
            pos_ids.append(base if t == 1 else base.repeat(t, 1))

        pos_ids = torch.cat(pos_ids, dim=0).to(self.device, non_blocking=True)
        max_grid_size = max(max(h, w) for _, h, w in grid_thw)

        # Use pre-computed cos_sin_cache from RotaryEmbedding
        cos, sin = self.rotary_pos_emb.get_cos_sin(max_grid_size)

        cos_combined = cos[pos_ids].flatten(1)
        sin_combined = sin[pos_ids].flatten(1)

        return cos_combined, sin_combined
```
**EN:** This method implements `rot_pos_emb(grid_thw: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLMoeVisionModel`.
**CN:** 这个方法实现了 `rot_pos_emb(grid_thw: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLMoeVisionModel` 内部调用。

### Lines 453-468: Method `Qwen3VLMoeVisionModel._get_interpolation_indices`
```python
    def _get_interpolation_indices(self, dim_size: int) -> torch.Tensor:
        """
        Compute continuous interpolation indices for a single dimension.

        Returns continuous indices.
        """
        if self.align_corners:
            indices = np.linspace(
                0, self.num_grid_per_side - 1, dim_size, dtype=np.float32
            )
        else:
            indices = (np.arange(dim_size, dtype=np.float32) + 0.5) * (
                self.num_grid_per_side / dim_size
            ) - 0.5
            indices = np.clip(indices, 0, self.num_grid_per_side - 1)
        return indices
```
**EN:** This method implements `_get_interpolation_indices(dim_size: ...)` and Compute continuous interpolation indices for a single dimension.
**CN:** 这个方法实现了 `_get_interpolation_indices(dim_size: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 470-498: Method `Qwen3VLMoeVisionModel._calculate_indices_and_weights`
```python
    def _calculate_indices_and_weights(self, h_idxs, w_idxs):
        """
        Compute bilinear interpolation indices and weights.

        Returns tuple of (indices, weights), each as 4 numpy arrays for the 4 corner points.
        """
        h_f = np.floor(h_idxs).astype(np.int64)
        h_c = np.clip(h_f + 1, 0, self.num_grid_per_side - 1)
        dh = h_idxs - h_f

        w_f = np.floor(w_idxs).astype(np.int64)
        w_c = np.clip(w_f + 1, 0, self.num_grid_per_side - 1)
        dw = w_idxs - w_f

        side = self.num_grid_per_side

        indices = [
            (h_f[:, None] * side + w_f).flatten(),
            (h_f[:, None] * side + w_c).flatten(),
            (h_c[:, None] * side + w_f).flatten(),
            (h_c[:, None] * side + w_c).flatten(),
        ]
        weights = [
            ((1 - dh)[:, None] * (1 - dw)).flatten(),
            ((1 - dh)[:, None] * dw).flatten(),
            (dh[:, None] * (1 - dw)).flatten(),
            (dh[:, None] * dw).flatten(),
        ]
        return indices, weights
```
**EN:** This method implements `_calculate_indices_and_weights(h_idxs, w_idxs)` and Compute bilinear interpolation indices and weights.
**CN:** 这个方法实现了 `_calculate_indices_and_weights(h_idxs, w_idxs)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 500-521: Method `Qwen3VLMoeVisionModel._get_position_embedding`
```python
    def _get_position_embedding(self, patch_pos_embeds, grid_ts, grid_hs, grid_ws):
        """
        Tile and reorganize position embeddings to align with the token sequence.
        """
        result_parts = []
        merge_size = self.spatial_merge_size

        for pos_embed, t, h, w in zip(patch_pos_embeds, grid_ts, grid_hs, grid_ws):
            pos_embed = pos_embed.repeat(t, 1)

            h_merge = h // merge_size
            w_merge = w // merge_size

            pos_embed = (
                pos_embed.view(t, h_merge, merge_size, w_merge, merge_size, -1)
                .permute(0, 1, 3, 2, 4, 5)
                .flatten(0, 4)
            )

            result_parts.append(pos_embed)

        return torch.cat(result_parts, dim=0)
```
**EN:** This method implements `_get_position_embedding(patch_pos_embeds, grid_ts, grid_hs, grid_ws)` and Tile and reorganize position embeddings to align with the token sequence.
**CN:** 这个方法实现了 `_get_position_embedding(patch_pos_embeds, grid_ts, grid_hs, grid_ws)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 523-537: Method `Qwen3VLMoeVisionModel._torch_interp_indices`
```python
    def _torch_interp_indices(
        self, dim_size: int, device: torch.device
    ) -> torch.Tensor:
        side = self.num_grid_per_side
        if self.align_corners:
            # align_corners=True
            return torch.linspace(
                0, side - 1, dim_size, dtype=torch.float32, device=device
            )
        else:
            # align_corners=False  (match _get_interpolation_indices)
            idx = (torch.arange(dim_size, dtype=torch.float32, device=device) + 0.5) * (
                side / dim_size
            ) - 0.5
            return idx.clamp_(0, side - 1)
```
**EN:** This method implements `_torch_interp_indices(dim_size: ..., device: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLMoeVisionModel`.
**CN:** 这个方法实现了 `_torch_interp_indices(dim_size: ..., device: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLMoeVisionModel` 内部调用。

### Lines 539-597: Method `Qwen3VLMoeVisionModel.fast_pos_embed_interpolate_from_list`
```python
    def fast_pos_embed_interpolate_from_list(self, grid_thw):
        num_grid_per_side = self.num_grid_per_side
        m_size = self.spatial_merge_size
        hidden_dim = self.pos_embed.embedding_dim

        outputs = []
        for t, h, w in grid_thw:
            h_idxs = torch.linspace(
                0, num_grid_per_side - 1, h, dtype=torch.float32, device=self.device
            )
            w_idxs = torch.linspace(
                0, num_grid_per_side - 1, w, dtype=torch.float32, device=self.device
            )

            h_floor = h_idxs.to(torch.long)
            w_floor = w_idxs.to(torch.long)
            h_ceil = torch.clamp(h_floor + 1, max=num_grid_per_side - 1)
            w_ceil = torch.clamp(w_floor + 1, max=num_grid_per_side - 1)

            dh = h_idxs - h_floor
            dw = w_idxs - w_floor

            # Create meshgrid view for all h, w vars
            dh_grid, dw_grid = torch.meshgrid(dh, dw, indexing="ij")
            h_floor_grid, w_floor_grid = torch.meshgrid(h_floor, w_floor, indexing="ij")
            h_ceil_grid, w_ceil_grid = torch.meshgrid(h_ceil, w_ceil, indexing="ij")

            # original computation of weights
            # w00 = (1 - dh_grid) * (1 - dw_grid)
            # w01 = (1 - dh_grid) * dw_grid
            # w10 = dh_grid * (1 - dw_grid)
            # w11 = dh_grid * dw_grid
            # we reuse w11 here to avoid duplicate
            # dh_grid * dw_grid computation
            w11 = dh_grid * dw_grid
            w10 = dh_grid - w11
            w01 = dw_grid - w11
            w00 = 1 - dh_grid - w01

            h_grid = torch.stack([h_floor_grid, h_floor_grid, h_ceil_grid, h_ceil_grid])
# ... truncated for brevity ...
```
**EN:** This method implements `fast_pos_embed_interpolate_from_list(grid_thw)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `fast_pos_embed_interpolate_from_list(grid_thw)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 599-617: Method `Qwen3VLMoeVisionModel.add_padding_to_fi_seqlens`
```python
    def add_padding_to_fi_seqlens(
        self, seq: np.ndarray, batch_size: int, padding_value: int
    ) -> np.ndarray:
        batch_size_padded = next(
            (b for b in BATCH_BUCKETS if b >= batch_size),
            # For large batches (> max bucket), round up to a multiple of
            # the base bucket size to avoid negative pad length.
            round_up(batch_size, BATCH_BUCKETS[0]),
        )
        if batch_size_padded == batch_size:
            return seq
        return np.concatenate(
            [
                seq,
                np.full(
                    (batch_size_padded - batch_size,), padding_value, dtype=seq.dtype
                ),
            ]
        )
```
**EN:** This method implements `add_padding_to_fi_seqlens(seq: ..., batch_size: ..., padding_value: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLMoeVisionModel`.
**CN:** 这个方法实现了 `add_padding_to_fi_seqlens(seq: ..., batch_size: ..., padding_value: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLMoeVisionModel` 内部调用。

### Lines 619-627: Method `Qwen3VLMoeVisionModel.bucket_flashinfer_max_seqlen`
```python
    def bucket_flashinfer_max_seqlen(self, real_max_seqlen: int) -> int:
        if real_max_seqlen <= 0:
            return FLASHINFER_MAX_SEQLEN_BUCKETS[0]
        return next(
            (s for s in FLASHINFER_MAX_SEQLEN_BUCKETS if s >= real_max_seqlen),
            # For large sequences (> max bucket), round up to a multiple of
            # the largest bucket to avoid under-estimation.
            round_up(real_max_seqlen, FLASHINFER_MAX_SEQLEN_BUCKETS[-1]),
        )
```
**EN:** This method implements `bucket_flashinfer_max_seqlen(real_max_seqlen: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLMoeVisionModel`.
**CN:** 这个方法实现了 `bucket_flashinfer_max_seqlen(real_max_seqlen: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLMoeVisionModel` 内部调用。

### Lines 629-682: Method `Qwen3VLMoeVisionModel.fast_pos_embed_interpolate`
```python
    def fast_pos_embed_interpolate(self, grid_thw):
        """Interpolate position embeddings for (batch, 3) size input dimensions.

        Performs bilinear interpolation on spatial dimensions (height, width) and replicates
        along temporal dimension. The result is reorganized according to spatial_merge_size.

        Args:
            grid_thw: Tensor of shape [batch_size, 3] with (temporal, height, width) dimensions
                     in patches for each sample.

        Returns:
            Interpolated position embeddings tensor.
        """
        grid_thw_cpu = grid_thw.cpu().numpy()

        # transfer data to CPU before loop
        temporal_dims = grid_thw_cpu[:, 0].tolist()
        height_dims = grid_thw_cpu[:, 1].tolist()
        width_dims = grid_thw_cpu[:, 2].tolist()

        device = self.pos_embed.weight.device
        dtype = self.pos_embed.weight.dtype

        patches_size = [h * w for h, w in zip(height_dims, width_dims)]
        total_patches = sum(patches_size)
        all_indices_np = np.zeros((4, total_patches), dtype=np.int64)
        all_weights_np = np.zeros((4, total_patches), dtype=np.float32)

        current_idx = 0

        # calculate indices and weights on CPU
        for t, h, w in zip(temporal_dims, height_dims, width_dims):
            h_idxs = self._get_interpolation_indices(h)
            w_idxs = self._get_interpolation_indices(w)

            indices, weights = self._calculate_indices_and_weights(h_idxs, w_idxs)

            end_idx = current_idx + h * w
            for i in range(4):
                all_indices_np[i, current_idx:end_idx] = indices[i]
# ... truncated for brevity ...
```
**EN:** This method implements `fast_pos_embed_interpolate(grid_thw)` and Interpolate position embeddings for (batch, 3) size input dimensions.
**CN:** 这个方法实现了 `fast_pos_embed_interpolate(grid_thw)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 684-719: Method `Qwen3VLMoeVisionModel.compute_flashinfer_batch_offsets_packed`
```python
    def compute_flashinfer_batch_offsets_packed(
        self,
        token_cu_seqlens: np.ndarray,
        *,
        elem_per_token: int,
    ) -> np.ndarray:
        """
        Build packed *element* indptrs for FlashInfer cuDNN prefill.

        Input:
        token_cu_seqlens: (B+1,) token indptr
        elem_per_token: per-token element width on THIS TP rank
                        (usually hidden_size / attn_tp_size)

        Output:
        packed_offsets: (3 * (B_padded + 1),) int32
            [qk_indptr, v_indptr, o_indptr] concatenated,
            each indptr is (B_padded + 1,) in element units.
        """
        assert token_cu_seqlens.ndim == 1 and token_cu_seqlens.size >= 2
        B = int(token_cu_seqlens.size - 1)
        B_padded = self.bucket_flashinfer_batch_size(B)

        # token indptr -> pad to (B_padded+1,) by appending total_tokens for extra empty sequences
        token_indptr = token_cu_seqlens.astype(np.int64, copy=False)  # (B+1,)
        if B_padded != B:
            pad = np.full((B_padded - B,), token_indptr[-1], dtype=token_indptr.dtype)
            token_indptr = np.concatenate([token_indptr, pad], axis=0)  # (B_padded+1,)

        # convert token indptr -> element indptr
        elem_indptr = (token_indptr * int(elem_per_token)).astype(
            np.int32
        )  # (B_padded+1,)

        # q/k/v/o in this ViT path share the same indptr
        return np.concatenate([elem_indptr, elem_indptr, elem_indptr], axis=0)
```
**EN:** This method implements `compute_flashinfer_batch_offsets_packed(token_cu_seqlens: ..., *, elem_per_token: ...)` and Build packed *element* indptrs for FlashInfer cuDNN prefill.
**CN:** 这个方法实现了 `compute_flashinfer_batch_offsets_packed(token_cu_seqlens: ..., *, elem_per_token: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 721-726: Method `Qwen3VLMoeVisionModel.bucket_flashinfer_batch_size`
```python
    def bucket_flashinfer_batch_size(self, batch_size: int) -> int:
        """Bucketize batch size for cuDNN graph caching."""
        return next(
            (b for b in BATCH_BUCKETS if b >= batch_size),
            round_up(batch_size, BATCH_BUCKETS[0]),
        )
```
**EN:** This method implements `bucket_flashinfer_batch_size(batch_size: ...)` and Bucketize batch size for cuDNN graph caching.
**CN:** 这个方法实现了 `bucket_flashinfer_batch_size(batch_size: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 728-747: Method `Qwen3VLMoeVisionModel.compute_flashinfer_sequence_lengths_padded`
```python
    def compute_flashinfer_sequence_lengths_padded(
        self,
        token_cu_seqlens: np.ndarray,
    ) -> np.ndarray:
        """
        token_cu_seqlens: (B+1,) token indptr
        return: (B_padded,) token lengths (padded with 0)
        """
        assert token_cu_seqlens.ndim == 1 and token_cu_seqlens.size >= 2
        B = int(token_cu_seqlens.size - 1)

        seq_lens = (token_cu_seqlens[1:] - token_cu_seqlens[:-1]).astype(
            np.int32
        )  # (B,)

        B_padded = self.bucket_flashinfer_batch_size(B)
        if B_padded != B:
            pad = np.zeros((B_padded - B,), dtype=np.int32)
            seq_lens = np.concatenate([seq_lens, pad], axis=0)  # (B_padded,)
        return seq_lens
```
**EN:** This method implements `compute_flashinfer_sequence_lengths_padded(token_cu_seqlens: ...)` and token_cu_seqlens: (B+1,) token indptr.
**CN:** 这个方法实现了 `compute_flashinfer_sequence_lengths_padded(token_cu_seqlens: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 749-857: Method `Qwen3VLMoeVisionModel.forward`
```python
    def forward(
        self,
        x: torch.Tensor,
        grid_thw: torch.Tensor,
    ) -> torch.Tensor:
        if envs.SGLANG_VIT_ENABLE_CUDA_GRAPH.get():
            if _is_npu:
                return self.forward_with_npu_graph(x, grid_thw)
            return self.forward_with_cuda_graph(x, grid_thw)

        x = x.to(device=self.device, dtype=self.dtype)
        x = self.patch_embed(x)

        if isinstance(grid_thw, list):
            grid_thw_list = grid_thw
            grid_thw = np.array(grid_thw, dtype=np.int32)
        else:
            grid_thw_list = grid_thw.tolist()
            grid_thw = grid_thw.cpu().numpy()

        pos_embeds = self.fast_pos_embed_interpolate_from_list(grid_thw_list)
        x += pos_embeds

        rotary_pos_emb_cos, rotary_pos_emb_sin = self.rot_pos_emb(grid_thw_list)

        # ---- build token indptr (B+1,) ----
        token_cu_seqlens = np.repeat(
            grid_thw[:, 1] * grid_thw[:, 2], grid_thw[:, 0]
        ).cumsum(axis=0, dtype=np.int32)
        token_cu_seqlens = np.concatenate(
            [np.zeros(1, dtype=np.int32), token_cu_seqlens]
        )

        flashinfer_max_seqlen = 0
        cu_seqlens = None
        if get_global_server_args().mm_attention_backend == "flashinfer_cudnn":
            # real token lens (B,)
            real_seq_lens = token_cu_seqlens[1:] - token_cu_seqlens[:-1]
            flashinfer_max_seqlen = self.bucket_flashinfer_max_seqlen(
                int(real_seq_lens.max()) if real_seq_lens.size > 0 else 0
# ... truncated for brevity ...
```
**EN:** This method implements `forward(x: ..., grid_thw: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ..., grid_thw: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 859-878: Method `Qwen3VLMoeVisionModel.forward_with_npu_graph`
```python
    def forward_with_npu_graph(
        self,
        x: torch.Tensor,
        grid_thw: torch.Tensor,
    ) -> torch.Tensor:
        (
            x,
            cu_seqlens,
            rotary_pos_emb_cos,
            rotary_pos_emb_sin,
        ) = self._prepare_graph_inputs(x, grid_thw)

        cu_seqlens = cu_seqlens.to("cpu")
        return self.graph_runners.run(
            x=x,
            rotary_pos_emb_cos=rotary_pos_emb_cos,
            rotary_pos_emb_sin=rotary_pos_emb_sin,
            cu_seqlens=cu_seqlens,
            output_indices=None,
        )
```
**EN:** This method implements `forward_with_npu_graph(x: ..., grid_thw: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_with_npu_graph(x: ..., grid_thw: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 880-905: Method `Qwen3VLMoeVisionModel.forward_with_cuda_graph`
```python
    def forward_with_cuda_graph(
        self,
        x: torch.Tensor,
        grid_thw: torch.Tensor,
    ) -> torch.Tensor:
        (
            x,
            cu_seqlens,
            rotary_pos_emb_cos,
            rotary_pos_emb_sin,
        ) = self._prepare_graph_inputs(x, grid_thw)
        if not isinstance(cu_seqlens, torch.Tensor):
            cu_seqlens = torch.tensor(cu_seqlens, device=x.device, dtype=torch.int32)
        else:
            cu_seqlens = cu_seqlens.to(device=x.device, dtype=torch.int32)
        cu_seqlens = cu_seqlens.contiguous()

        return self.graph_runners.run(
            x=x,
            position_embeddings=None,
            rotary_pos_emb_cos=rotary_pos_emb_cos,
            rotary_pos_emb_sin=rotary_pos_emb_sin,
            cu_seqlens=cu_seqlens,
            cu_window_seqlens=None,
            output_indices=None,
        )
```
**EN:** This method implements `forward_with_cuda_graph(x: ..., grid_thw: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_with_cuda_graph(x: ..., grid_thw: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 907-932: Method `Qwen3VLMoeVisionModel.load_weights`
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("attn.qkv.", "attn.q.", "q"),
            ("attn.qkv.", "attn.k.", "k"),
            ("attn.qkv.", "attn.v.", "v"),
        ]
        params_dict = dict(self.named_parameters(remove_duplicate=False))
        loaded_params: set[str] = set()

        for name, loaded_weight in weights:
            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)

                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
            loaded_params.add(name)
        return loaded_params
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 934-958: Method `Qwen3VLMoeVisionModel._prepare_graph_inputs`
```python
    def _prepare_graph_inputs(self, x: torch.Tensor, grid_thw: torch.Tensor) -> tuple[
        torch.Tensor,
        torch.Tensor,
        torch.Tensor,
        torch.Tensor,
    ]:
        # patchify
        x = x.to(device=self.device, dtype=self.dtype)
        x = self.patch_embed(x)

        if isinstance(grid_thw, list):
            grid_thw_list = grid_thw
            grid_thw = torch.tensor(grid_thw, dtype=torch.int32)
        else:
            grid_thw_list = grid_thw.tolist()

        pos_embeds = self.fast_pos_embed_interpolate(grid_thw)
        x += pos_embeds

        # rotary embedding -> (cos, sin)
        rotary_pos_emb_cos, rotary_pos_emb_sin = self.rot_pos_emb(grid_thw_list)

        # compute cu_seqlens
        cu_seqlens = compute_cu_seqlens_from_grid_numpy(grid_thw)
        return x, cu_seqlens, rotary_pos_emb_cos, rotary_pos_emb_sin
```
**EN:** This method implements `_prepare_graph_inputs(x: ..., grid_thw: ...)` and prepares intermediate tensors, masks, or metadata before the main compute path.
**CN:** 这个方法实现了 `_prepare_graph_inputs(x: ..., grid_thw: ...)`，其作用是在主计算路径前准备中间张量、掩码或元数据。

### Lines 961-961: Top-level assign
```python
cached_get_processor = lru_cache(get_processor)
```
**EN:** Defines or updates cached_get_processor, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 cached_get_processor，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 964-965: Class `Qwen3LLMModel` overview
```python
class Qwen3LLMModel(Qwen3Model):
```
**EN:** Defines `Qwen3LLMModel` as a reusable runtime type derived from Qwen3Model. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3LLMModel`，其继承关系为 Qwen3Model。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 966-982: Method `Qwen3LLMModel.__init__`
```python
    def __init__(
        self,
        *,
        config: Qwen3VLConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__(config=config, quant_config=quant_config, prefix=prefix)
        if not self.pp_group.is_first_rank:
            assert self.start_layer >= len(
                config.vision_config.deepstack_visual_indexes
            ), "start_layer should be greater than or equal to len(deepstack_visual_indexes)"

        self.hidden_size = config.hidden_size
        self.deepstack_embed_to_decoder_layer = range(
            len(config.vision_config.deepstack_visual_indexes)
        )
```
**EN:** This method implements `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 984-994: Method `Qwen3LLMModel.get_deepstack_embeds`
```python
    def get_deepstack_embeds(
        self, layer_idx: int, input_deepstack_embeds: Optional[torch.Tensor]
    ) -> Optional[torch.Tensor]:
        """Get deepstack embeddings for a given layer index, or None if not applicable."""
        if (
            input_deepstack_embeds is None
            or layer_idx not in self.deepstack_embed_to_decoder_layer
        ):
            return None
        sep = self.hidden_size * layer_idx
        return input_deepstack_embeds[:, sep : sep + self.hidden_size]
```
**EN:** This method implements `get_deepstack_embeds(layer_idx: ..., input_deepstack_embeds: ...)` and Get deepstack embeddings for a given layer index, or None if not applicable.
**CN:** 这个方法实现了 `get_deepstack_embeds(layer_idx: ..., input_deepstack_embeds: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 996-1067: Method `Qwen3LLMModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
        input_deepstack_embeds: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, PPProxyTensors]:

        if self.pp_group.is_first_rank:
            if input_embeds is None:
                hidden_states = self.embed_tokens(input_ids)
            else:
                hidden_states = input_embeds
            residual = None
        else:
            assert pp_proxy_tensors is not None
            hidden_states = pp_proxy_tensors["hidden_states"]
            residual = pp_proxy_tensors["residual"]

        aux_hidden_states = []
        for layer_idx, layer in enumerate(
            self.layers[self.start_layer : self.end_layer]
        ):
            layer_idx = layer_idx + self.start_layer
            if layer_idx in self.layers_to_capture:
                aux_hidden_states.append(
                    hidden_states + residual if residual is not None else hidden_states
                )

            # SGLang applies residual at the START of the next layer, not at the END like HuggingFace.
            # See: https://github.com/huggingface/transformers/blob/v5.0.0rc0/src/transformers/models/qwen3_vl/modeling_qwen3_vl.py#L549
            # To match HF behavior, deepstack must be added AFTER residual: (hidden_states + residual) + deepstack
            # The order matters because addition with different tensors is not associative in practice.
            # Deepstack for prev_layer is applied at the start of current layer via post_residual_addition.
            deepstack_embeds = self.get_deepstack_embeds(
                layer_idx - 1, input_deepstack_embeds
            )
            hidden_states, residual = layer(
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=..., input_deepstack_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=..., input_deepstack_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 1070-1071: Class `Qwen3VLForConditionalGeneration` overview
```python
class Qwen3VLForConditionalGeneration(nn.Module):
    # To ensure correct weight loading and mapping.
```
**EN:** Defines `Qwen3VLForConditionalGeneration` as a reusable runtime type derived from nn.Module. The class groups 14 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3VLForConditionalGeneration`，其继承关系为 nn.Module。这个类组织了 14 个方法，用于实现模型相关行为。

### Lines 1072-1084: Class `Qwen3VLForConditionalGeneration` attributes
```python
    hf_to_sglang_mapper = WeightsMapper(
        orig_to_new_substr={
            "attn.qkv": "attn.qkv_proj",
        },
        orig_to_new_prefix={
            # mapping for new names in checkpoint saved after transformers v4.52
            "model.language_model.": "language_model.model.",
            "model.visual.": "visual.",
            # mapping for original checkpoint
            "lm_head.": "language_model.lm_head.",
            "model.": "language_model.model.",
        },
    )
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3VLForConditionalGeneration` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3VLForConditionalGeneration` 在运行时的行为。

### Lines 1086-1164: Method `Qwen3VLForConditionalGeneration.__init__`
```python
    def __init__(
        self,
        config: Qwen3VLConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        language_model_cls=Qwen3LLMModel,
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.quant_config = quant_config

        self.use_data_parallel = get_global_server_args().mm_enable_dp_encoder

        self.visual = Qwen3VLMoeVisionModel(
            config.vision_config,
            # NOTE: Qwen3-VL vision encoder currently supports BitsAndBytes 4-bit quantization.
            # Other quantization methods (e.g., GPTQ, AWQ) are untested and may not be supported.
            quant_config=None,
            norm_eps=getattr(config, "rms_norm_eps", 1e-6),
            prefix=add_prefix("model.visual", prefix),
            use_data_parallel=self.use_data_parallel,
        )

        # TODO: make it more elegant
        if language_model_cls is Qwen3LLMModel:
            self.config: Qwen3VLConfig = config  # for qwen3-vl
        else:
            self.config = config.text_config  # for qwen3-omni / qwen3-vl-moe
            self.config.encoder_only = getattr(config, "encoder_only", False)
            self.config.language_only = getattr(config, "language_only", False)
            # Propagate tie_word_embeddings from parent config. In transformers
            # v5.5.3+, Qwen3VLMoeTextConfig sets tie_word_embeddings=True by
            # default but the actual model checkpoint has a separate lm_head.
            # The parent Qwen3VLMoeConfig correctly has tie_word_embeddings=False.
            if hasattr(config, "tie_word_embeddings"):
                self.config.tie_word_embeddings = config.tie_word_embeddings

        if not hasattr(config, "encoder_only") or not config.encoder_only:
            self.model = language_model_cls(
                config=self.config,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., language_model_cls=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., language_model_cls=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1166-1174: Method `Qwen3VLForConditionalGeneration.separate_deepstack_embeds`
```python
    def separate_deepstack_embeds(self, embedding):
        assert (
            embedding.shape[-1] % (1 + self.num_deepstack_embeddings) == 0
        ), f"hidden_state of {embedding.shape} should be divisible by ({1 + self.num_deepstack_embeddings})"

        separate_index = self.config.hidden_size
        input_embeds = embedding[:, :separate_index]
        input_deepstack_embeds = embedding[:, separate_index:]
        return input_embeds, input_deepstack_embeds
```
**EN:** This method implements `separate_deepstack_embeds(embedding)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `separate_deepstack_embeds(embedding)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1176-1178: Method `Qwen3VLForConditionalGeneration.start_layer`
```python
    @property
    def start_layer(self) -> int:
        return getattr(getattr(self, "model", None), "start_layer", 0)
```
**EN:** This method implements `start_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLForConditionalGeneration` Decorators: property.
**CN:** 这个方法实现了 `start_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLForConditionalGeneration` 内部调用 装饰器：property。

### Lines 1180-1187: Method `Qwen3VLForConditionalGeneration.end_layer`
```python
    @property
    def end_layer(self) -> int:
        model = getattr(self, "model", None)
        end_layer = getattr(model, "end_layer", None)
        if end_layer is not None:
            return end_layer
        cfg = getattr(model, "config", None)
        return int(getattr(cfg, "num_hidden_layers", 0))
```
**EN:** This method implements `end_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLForConditionalGeneration` Decorators: property.
**CN:** 这个方法实现了 `end_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLForConditionalGeneration` 内部调用 装饰器：property。

### Lines 1189-1191: Method `Qwen3VLForConditionalGeneration.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], mm_inputs: MultimodalInputs):
        pattern = MultiModalityDataPaddingPatternMultimodalTokens()
        return pattern.pad_input_tokens(input_ids, mm_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLForConditionalGeneration`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLForConditionalGeneration` 内部调用。

### Lines 1193-1210: Method `Qwen3VLForConditionalGeneration.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        # in qwen-vl, last dim is the same
        pixel_values = torch.cat([item.feature for item in items], dim=0).type(
            self.visual.dtype
        )
        image_grid_thw = torch.concat([item.image_grid_thw for item in items], dim=0)
        assert pixel_values.dim() == 2, pixel_values.dim()
        assert image_grid_thw.dim() == 2, image_grid_thw.dim()

        if self.use_data_parallel:
            return run_dp_sharded_mrope_vision_model(
                self.visual,
                pixel_values,
                image_grid_thw.tolist(),
                rope_type="rope_3d",
            )
        else:
            return self.visual(pixel_values, grid_thw=image_grid_thw)
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1212-1226: Method `Qwen3VLForConditionalGeneration.get_video_feature`
```python
    def get_video_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        # in qwen-vl, last dim is the same
        pixel_values = torch.cat([item.feature for item in items], dim=0).type(
            self.visual.dtype
        )
        video_grid_thw = torch.concat([item.video_grid_thw for item in items], dim=0)
        assert pixel_values.dim() == 2, pixel_values.dim()
        assert video_grid_thw.dim() == 2, video_grid_thw.dim()
        if self.use_data_parallel:
            return run_dp_sharded_mrope_vision_model(
                self.visual, pixel_values, video_grid_thw.tolist(), rope_type="rope_3d"
            )
        else:
            video_embeds = self.visual(pixel_values, grid_thw=video_grid_thw)
        return video_embeds
```
**EN:** This method implements `get_video_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_video_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1228-1229: Method `Qwen3VLForConditionalGeneration.get_input_embeddings`
```python
    def get_input_embeddings(self):
        return self.model.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1231-1233: Class `Qwen3VLForConditionalGeneration` attributes
```python
    _lora_pattern = re.compile(
        r"^model\.layers\.(\d+)\.(?:self_attn|mlp)\.(?:qkv_proj|o_proj|down_proj|gate_up_proj)$"
    )
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3VLForConditionalGeneration` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3VLForConditionalGeneration` 在运行时的行为。

### Lines 1235-1236: Method `Qwen3VLForConditionalGeneration.should_apply_lora`
```python
    def should_apply_lora(self, module_name: str) -> bool:
        return bool(self._lora_pattern.match(module_name))
```
**EN:** This method implements `should_apply_lora(module_name: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLForConditionalGeneration`.
**CN:** 这个方法实现了 `should_apply_lora(module_name: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLForConditionalGeneration` 内部调用。

### Lines 1238-1298: Method `Qwen3VLForConditionalGeneration.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        get_embedding: bool = False,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ):
        """Run forward pass for Qwen3-VL.

        Args:
            input_ids: Flattened (concatenated) input_ids corresponding to a
                batch.
            positions: Flattened (concatenated) position ids corresponding to a
                batch.
                **NOTE**: If mrope is enabled (default setting for Qwen2-VL
                opensource models), the shape will be `(3, seq_len)`,
                otherwise it will be `(seq_len,).
                (Use input_metadata.mrope_positions to replace it)
        """
        if self.is_mrope_enabled:
            positions = forward_batch.mrope_positions

        if not (
            forward_batch.forward_mode.is_decode()
            or not forward_batch.contains_image_inputs()
        ):
            if self.is_mrope_enabled:
                assert positions.ndim == 2 and positions.size(0) == 3, (
                    "multimodal section rotary embedding requires "
                    f"(3, seq_len) positions, but got {positions.size()}"
                )

        hidden_states = general_mm_embed_routine(
            input_ids=input_ids,
            forward_batch=forward_batch,
            language_model=self.model,
            multimodal_model=self,
            positions=positions,
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., get_embedding: ...=..., pp_proxy_tensors: ...=...)` and Run forward pass for Qwen3-VL. Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., get_embedding: ...=..., pp_proxy_tensors: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：torch.no_grad()。

### Lines 1300-1308: Method `Qwen3VLForConditionalGeneration.set_dflash_layers_to_capture`
```python
    def set_dflash_layers_to_capture(self, layer_ids: List[int]):
        if not self.pp_group.is_last_rank:
            return
        if layer_ids is None:
            raise ValueError(
                "DFLASH requires explicit layer_ids for aux hidden capture."
            )
        self.capture_aux_hidden_states = True
        self.model.set_dflash_layers_to_capture([val + 1 for val in layer_ids])
```
**EN:** This method implements `set_dflash_layers_to_capture(layer_ids: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLForConditionalGeneration`.
**CN:** 这个方法实现了 `set_dflash_layers_to_capture(layer_ids: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLForConditionalGeneration` 内部调用。

### Lines 1310-1393: Method `Qwen3VLForConditionalGeneration.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            ("gate_up_proj", "up_proj", 1),
            ("gate_up_proj", "gate_proj", 0),
        ]
        params_dict = dict(self.named_parameters(remove_duplicate=False))
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            if "language_model" in name:
                name = name.replace(r"model.language_model.", r"model.")
            layer_id = get_layer_id(name)

            # Only copy embed_tokens to lm_head when tie_word_embeddings=True
            # For models with tie_word_embeddings=False (e.g. 8B), lm_head has independent weights
            if (
                self.pp_group.is_last_rank
                and "model.embed_tokens.weight" in name
                and self.config.tie_word_embeddings
            ):
                if "lm_head.weight" in params_dict:
                    lm_head_param = params_dict["lm_head.weight"]
                    weight_loader = getattr(
                        lm_head_param, "weight_loader", default_weight_loader
                    )
                    weight_loader(lm_head_param, loaded_weight)

            is_visual = "visual" in name
            if (
                not is_visual
                and layer_id is not None
                and hasattr(self, "model")
                and hasattr(self.model, "start_layer")
                and (
                    layer_id < self.model.start_layer
                    or layer_id >= self.model.end_layer
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1395-1396: Method `Qwen3VLForConditionalGeneration.get_embed_and_head`
```python
    def get_embed_and_head(self):
        return self.model.embed_tokens.weight, self.lm_head.weight
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1398-1409: Method `Qwen3VLForConditionalGeneration.set_eagle3_layers_to_capture`
```python
    def set_eagle3_layers_to_capture(self, layer_ids: Optional[List[int]] = None):
        self.capture_aux_hidden_states = True
        self.model.capture_aux_hidden_states = True
        if layer_ids is None:
            num_layers = self.config.num_hidden_layers
            self.model.layers_to_capture = [
                2,
                num_layers // 2,
                num_layers - 3,
            ]  # Specific layers for EAGLE3 support
        else:
            self.model.layers_to_capture = [val + 1 for val in layer_ids]
```
**EN:** This method implements `set_eagle3_layers_to_capture(layer_ids: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3VLForConditionalGeneration`.
**CN:** 这个方法实现了 `set_eagle3_layers_to_capture(layer_ids: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3VLForConditionalGeneration` 内部调用。

### Lines 1412-1412: Top-level assign
```python
EntryClass = Qwen3VLForConditionalGeneration
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `logging`
- `re`
- `collections.defaultdict`
- `functools.lru_cache`
- `functools.partial`
- `typing.Callable`
- `typing.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `typing.Union`
- `numpy`
- `torch`
- `torch.nn`
- `einops.rearrange`
- `transformers.activations.ACT2FN`
- `sglang.srt.configs.qwen3_vl.Qwen3VLConfig`
- `sglang.srt.configs.qwen3_vl.Qwen3VLVisionConfig`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.distributed.parallel_state.get_pp_group`
- `sglang.srt.environ.envs`
- `sglang.srt.layers.attention.vision.BATCH_BUCKETS`
- `sglang.srt.layers.attention.vision.FLASHINFER_MAX_SEQLEN_BUCKETS`
- `sglang.srt.layers.attention.vision.FLASHINFER_WORKSPACE_SIZE_BYTES`
- `sglang.srt.layers.attention.vision.VisionAttention`
- `sglang.srt.layers.conv.Conv3dLayer`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.pooler.Pooler`
- `sglang.srt.layers.pooler.PoolingType`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.utils.get_layer_id`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternMultimodalTokens`
- `sglang.srt.managers.mm_utils.general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch.Modality`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.qwen3.Qwen3Model`
- `sglang.srt.models.utils.RotaryPosMixin`
- `sglang.srt.models.utils.WeightsMapper`
- `sglang.srt.models.utils.compute_cu_seqlens_from_grid_numpy`
- `sglang.srt.multimodal.mm_utils.run_dp_sharded_mrope_vision_model`
- `sglang.srt.multimodal.vit_cuda_graph_runner.ViTCudaGraphRunner`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.cpu_has_amx_support`
- `sglang.srt.utils.is_cpu`
- `sglang.srt.utils.is_npu`
- `sglang.srt.utils.round_up`
- `sglang.srt.utils.hf_transformers_utils.get_processor`
- `sglang.srt.hardware_backend.npu.graph_runner.vit_npu_graph_runner.ViTNpuGraphRunner`
