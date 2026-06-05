# paddleocr_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/paddleocr_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the paddleocr vl multimodal runtime stack in SGLang, including model blocks, adapters, and inference helpers. / 该模块实现 SGLang 中 paddleocr vl 的多模态运行时堆栈，包括模型模块、适配器与推理辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 17-40: Module imports
```python
from collections.abc import Iterable
from typing import List, Optional, Set, Tuple, Union

import numpy as np
import torch
import torch.nn as nn
from einops import rearrange
from transformers.activations import GELUActivation
from transformers.utils import torch_int

from sglang.srt.layers.activation import get_act_fn
from sglang.srt.layers.attention.vision import VisionAttention
from sglang.srt.layers.conv import Conv2dLayer
from sglang.srt.layers.linear import ColumnParallelLinear, RowParallelLinear
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.managers.mm_utils import (
    MultiModalityDataPaddingPatternMultimodalTokens,
    general_mm_embed_routine,
)
from sglang.srt.managers.schedule_batch import MultimodalDataItem, MultimodalInputs
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.ernie4 import Ernie4_5_ForCausalLM
from sglang.srt.utils import add_prefix, is_npu
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 43-44: Class `Projector` overview
```python
class Projector(nn.Module):
```
**EN:** Defines `Projector` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Projector`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 45-67: Method `Projector.__init__`
```python
    def __init__(
        self,
        text_config,
        vision_config,
        prefix: str = "",
    ):
        super().__init__()
        self.text_config = text_config
        self.vision_config = vision_config
        self.merge_kernel_size = (2, 2)

        self.hidden_size = (
            self.vision_config.hidden_size
            * self.merge_kernel_size[0]
            * self.merge_kernel_size[1]
        )

        self.pre_norm = torch.nn.LayerNorm(self.vision_config.hidden_size, eps=1e-05)
        self.linear_1 = nn.Linear(self.hidden_size, self.hidden_size, bias=True)
        self.act = GELUActivation()
        self.linear_2 = nn.Linear(
            self.hidden_size, self.text_config.hidden_size, bias=True
        )
```
**EN:** This method implements `__init__(text_config, vision_config, prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(text_config, vision_config, prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 69-105: Method `Projector.forward`
```python
    def forward(
        self,
        image_features: torch.Tensor,
        image_grid_thw: List[Tuple[int, int, int]],
    ) -> torch.Tensor:
        m1, m2 = self.merge_kernel_size
        if isinstance(image_features, (list, tuple)):
            processed_features = list()
            for image_feature, image_grid in zip(image_features, image_grid_thw):
                image_feature = self.pre_norm(image_feature)
                t, h, w = image_grid

                image_feature = rearrange(
                    image_feature,
                    "(t h p1 w p2) d -> (t h w) (p1 p2 d)",
                    t=t,
                    h=h // m1,
                    p1=m1,
                    w=w // m2,
                    p2=m2,
                )
                hidden_states = self.linear_1(image_feature)
                hidden_states = self.act(hidden_states)
                hidden_states = self.linear_2(hidden_states)
                processed_features.append(hidden_states)

            return processed_features

        dims = image_features.shape[:-1]
        dim = image_features.shape[-1]
        image_features = image_features.view(np.prod(dims), dim)
        hidden_states = self.pre_norm(image_features).view(-1, self.hidden_size)
        hidden_states = self.linear_1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states = self.linear_2(hidden_states)

        return hidden_states.view(*dims, -1)
```
**EN:** This method implements `forward(image_features: ..., image_grid_thw: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(image_features: ..., image_grid_thw: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 108-109: Class `SiglipVisionEmbeddings` overview
```python
class SiglipVisionEmbeddings(nn.Module):
```
**EN:** Defines `SiglipVisionEmbeddings` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SiglipVisionEmbeddings`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 110-136: Method `SiglipVisionEmbeddings.__init__`
```python
    def __init__(self, config):
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
**EN:** This method implements `__init__(config)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 138-173: Method `SiglipVisionEmbeddings.interpolate_pos_encoding`
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
**EN:** This method implements `interpolate_pos_encoding(embeddings: ..., height: ..., width: ..., is_after_patchify: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `SiglipVisionEmbeddings`.
**CN:** 这个方法实现了 `interpolate_pos_encoding(embeddings: ..., height: ..., width: ..., is_after_patchify: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SiglipVisionEmbeddings` 内部调用。

### Lines 175-192: Method `SiglipVisionEmbeddings.fetch_position_embedding_lfu_cache`
```python
    def fetch_position_embedding_lfu_cache(self, embeddings, h, w, max_cache: int = 20):
        grid = (h, w)
        if grid in self.cache_position_embedding:
            self.cache_position_count[grid] += 1
            return self.cache_position_embedding[grid]

        if len(self.cache_position_embedding) >= max_cache:
            min_hit_grid = min(
                self.cache_position_count,
                key=self.cache_position_count.get,
            )
            self.cache_position_count.pop(min_hit_grid)
            self.cache_position_embedding.pop(min_hit_grid)

        position_embedding = self.interpolate_pos_encoding(embeddings, h, w, True)
        self.cache_position_count[grid] = 1
        self.cache_position_embedding[grid] = position_embedding
        return position_embedding
```
**EN:** This method implements `fetch_position_embedding_lfu_cache(embeddings, h, w, max_cache: ...=...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `fetch_position_embedding_lfu_cache(embeddings, h, w, max_cache: ...=...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 194-250: Method `SiglipVisionEmbeddings.forward`
```python
    def forward(
        self,
        pixel_values: torch.FloatTensor,
        position_ids: Optional[torch.Tensor] = None,
        image_grid_thw: Optional[
            List[
                Union[
                    Tuple[int, int, int],
                    List[Tuple[int, int, int]],
                ]
            ]
        ] = None,
        interpolate_pos_encoding=False,
    ) -> torch.Tensor:
        if pixel_values.dim() == 4:
            pixel_values = pixel_values.unsqueeze(0)
        if pixel_values.dim() == 5:
            if position_ids is None:
                raise ValueError(
                    "position_ids cannot be None when pixel_values.dim() is 5."
                )
            (
                batch_size,
                squence_len,
                channel,
                height,
                width,
            ) = pixel_values.shape
            target_dtype = self.patch_embedding.weight.dtype
            pixel_values = rearrange(pixel_values, "b l c h w -> (b l) c h w")
            patch_embeds = self.patch_embedding(pixel_values.to(dtype=target_dtype))
            embeddings = patch_embeds.flatten(-2).squeeze(-1)

            if interpolate_pos_encoding and image_grid_thw is not None:
                start = 0
                tmp_embeddings = list()
                for image_grid in image_grid_thw:
                    t, h, w = image_grid
                    end = start + t * h * w
                    image_embeddings = embeddings[start:end, :]
# ... truncated for brevity ...
```
**EN:** This method implements `forward(pixel_values: ..., position_ids: ...=..., image_grid_thw: ...=..., interpolate_pos_encoding=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(pixel_values: ..., position_ids: ...=..., image_grid_thw: ...=..., interpolate_pos_encoding=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 253-254: Class `SigLIPRotaryEmbedding` overview
```python
class SigLIPRotaryEmbedding(nn.Module):
```
**EN:** Defines `SigLIPRotaryEmbedding` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SigLIPRotaryEmbedding`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 255-259: Method `SigLIPRotaryEmbedding.__init__`
```python
    def __init__(self, dim: int, theta: float = 10000.0) -> None:
        super().__init__()
        self.dim = dim
        self.theta = theta
        self.rope_init()
```
**EN:** This method implements `__init__(dim: ..., theta: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., theta: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 261-265: Method `SigLIPRotaryEmbedding.rope_init`
```python
    def rope_init(self):
        inv_freq = 1.0 / (
            self.theta ** (torch.arange(0, self.dim, 2, dtype=torch.float) / self.dim)
        )
        self.register_buffer("inv_freq", inv_freq, persistent=False)
```
**EN:** This method implements `rope_init()` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `rope_init()`，其作用是执行后续运行时所需的初始化工作。

### Lines 267-274: Method `SigLIPRotaryEmbedding.forward`
```python
    def forward(self, seqlen: int) -> torch.Tensor:
        seq = torch.arange(
            seqlen,
            device=self.inv_freq.device,
            dtype=self.inv_freq.dtype,
        )
        freqs = torch.outer(seq, self.inv_freq)
        return freqs
```
**EN:** This method implements `forward(seqlen: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(seqlen: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 277-278: Class `SiglipMLP` overview
```python
class SiglipMLP(nn.Module):
```
**EN:** Defines `SiglipMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SiglipMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 279-306: Method `SiglipMLP.__init__`
```python
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.activation_fn = get_act_fn(config.hidden_act)
        if quant_config and quant_config.get_name() in ["bitsandbytes", "torchao"]:
            quantizable = True
        else:
            quantizable = (
                config.hidden_size % 64 == 0 and config.intermediate_size % 64 == 0
            )
        self.fc1 = ColumnParallelLinear(
            config.hidden_size,
            config.intermediate_size,
            quant_config=quant_config if quantizable else None,
            prefix=add_prefix("fc1", prefix),
        )
        self.fc2 = RowParallelLinear(
            config.intermediate_size,
            config.hidden_size,
            quant_config=quant_config if quantizable else None,
            prefix=add_prefix("fc2", prefix),
        )
```
**EN:** This method implements `__init__(config, quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 308-312: Method `SiglipMLP.forward`
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.fc1(hidden_states)
        hidden_states = self.activation_fn(hidden_states)
        hidden_states, _ = self.fc2(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 315-316: Class `SiglipEncoderLayer` overview
```python
class SiglipEncoderLayer(nn.Module):
```
**EN:** Defines `SiglipEncoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SiglipEncoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 317-341: Method `SiglipEncoderLayer.__init__`
```python
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.embed_dim = config.hidden_size
        self.layer_norm1 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)

        self.self_attn = VisionAttention(
            embed_dim=self.embed_dim,
            num_heads=config.num_attention_heads,
            projection_size=self.embed_dim,
            use_qkv_parallel=True,
            qkv_bias=True,
            flatten_batch=True,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )

        self.layer_norm2 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)
        self.mlp = SiglipMLP(
            config, quant_config=quant_config, prefix=add_prefix("mlp", prefix)
        )
```
**EN:** This method implements `__init__(config, quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 343-368: Method `SiglipEncoderLayer.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: Optional[List[torch.Tensor]] = None,
        rope_emb: Optional[Tuple[torch.Tensor, torch.Tensor]] = None,
    ) -> Tuple[torch.FloatTensor]:

        residual = hidden_states

        hidden_states = self.layer_norm1(hidden_states)

        hidden_states = self.self_attn(
            hidden_states,
            cu_seqlens=cu_seqlens,
            position_embeddings=rope_emb,
        )

        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.layer_norm2(hidden_states)
        hidden_states = self.mlp(hidden_states)

        hidden_states = residual + hidden_states

        return hidden_states
```
**EN:** This method implements `forward(hidden_states: ..., cu_seqlens: ...=..., rope_emb: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., cu_seqlens: ...=..., rope_emb: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 371-372: Class `SiglipEncoder` overview
```python
class SiglipEncoder(nn.Module):
```
**EN:** Defines `SiglipEncoder` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SiglipEncoder`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 373-394: Method `SiglipEncoder.__init__`
```python
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        embed_dim = config.hidden_size
        num_heads = config.num_attention_heads
        head_dim = embed_dim // num_heads
        self.layers = nn.ModuleList(
            [
                SiglipEncoderLayer(
                    config,
                    quant_config=quant_config,
                    prefix=add_prefix(f"layers.{layer_idx}", prefix),
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )
        self.rotary_pos_emb = SigLIPRotaryEmbedding(head_dim // 2)
```
**EN:** This method implements `__init__(config, quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 396-404: Method `SiglipEncoder.flatten_list`
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
**EN:** This method implements `flatten_list()` and implements a focused helper that supports the surrounding runtime flow inside `SiglipEncoder` Decorators: staticmethod.
**CN:** 这个方法实现了 `flatten_list()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SiglipEncoder` 内部调用 装饰器：staticmethod。

### Lines 406-458: Method `SiglipEncoder.forward`
```python
    def forward(
        self,
        inputs_embeds,
        cu_seqlens: Optional[List[torch.Tensor]] = None,
        image_grid_thw: Optional[
            List[
                Union[
                    Tuple[int, int, int],
                    List[Tuple[int, int, int]],
                ]
            ]
        ] = None,
        height_position_ids: Optional[torch.Tensor] = None,
        width_position_ids: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        device = inputs_embeds.device
        hidden_states = inputs_embeds
        flatten_image_grid_thw = self.flatten_list(image_grid_thw)

        if width_position_ids is None or height_position_ids is None:
            split_hids = list()
            split_wids = list()
            for t, h, w in flatten_image_grid_thw:
                image_pids = torch.arange(t * h * w, device=device) % (h * w)
                sample_hids = image_pids // w
                sample_wids = image_pids % w
                split_hids.append(sample_hids)
                split_wids.append(sample_wids)
            width_position_ids = torch.concat(split_wids, dim=0)
            height_position_ids = torch.concat(split_hids, dim=0)

        pids = torch.stack(
            [height_position_ids, width_position_ids],
            dim=-1,
        )
        max_grid_size = pids.max() + 1
        rope_emb_max_grid = self.rotary_pos_emb(max_grid_size)
        rope_emb = rope_emb_max_grid[pids].flatten(1)
        rope_emb = rope_emb.repeat(1, 2)
        rope_emb = (rope_emb.cos(), rope_emb.sin())
# ... truncated for brevity ...
```
**EN:** This method implements `forward(inputs_embeds, cu_seqlens: ...=..., image_grid_thw: ...=..., height_position_ids: ...=..., width_position_ids: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(inputs_embeds, cu_seqlens: ...=..., image_grid_thw: ...=..., height_position_ids: ...=..., width_position_ids: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 461-462: Class `SiglipVisionTransformer` overview
```python
class SiglipVisionTransformer(nn.Module):
```
**EN:** Defines `SiglipVisionTransformer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SiglipVisionTransformer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 463-479: Method `SiglipVisionTransformer.__init__`
```python
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        embed_dim = config.hidden_size

        self.embeddings = SiglipVisionEmbeddings(config)
        self.encoder = SiglipEncoder(
            config,
            quant_config=quant_config,
            prefix=add_prefix("encoder", prefix),
        )
        self.post_layernorm = nn.LayerNorm(embed_dim, eps=config.layer_norm_eps)
```
**EN:** This method implements `__init__(config, quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 481-528: Method `SiglipVisionTransformer.forward`
```python
    def forward(
        self,
        pixel_values,
        interpolate_pos_encoding: Optional[bool] = False,
        position_ids: Optional[torch.Tensor] = None,
        height_position_ids: Optional[torch.Tensor] = None,
        width_position_ids: Optional[torch.Tensor] = None,
        cu_seqlens: Optional[List[torch.Tensor]] = None,
        image_grid_thw: Optional[
            List[
                Union[
                    Tuple[int, int, int],
                    List[Tuple[int, int, int]],
                ]
            ]
        ] = None,
    ) -> list[torch.Tensor]:

        hidden_states = self.embeddings(
            pixel_values,
            interpolate_pos_encoding=interpolate_pos_encoding,
            position_ids=position_ids,
            image_grid_thw=image_grid_thw,
        )

        last_hidden_state = self.encoder(
            inputs_embeds=hidden_states,
            cu_seqlens=cu_seqlens,
            image_grid_thw=image_grid_thw,
            height_position_ids=height_position_ids,
            width_position_ids=width_position_ids,
        )

        last_hidden_state = self.post_layernorm(last_hidden_state)

        sample_hidden_state = list()
        if cu_seqlens is None:
            raise ValueError(
                "cu_seqlens cannot be None for "
                "SiglipVisionTransformer output processing."
# ... truncated for brevity ...
```
**EN:** This method implements `forward(pixel_values, interpolate_pos_encoding: ...=..., position_ids: ...=..., height_position_ids: ...=..., width_position_ids: ...=..., cu_seqlens: ...=..., ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(pixel_values, interpolate_pos_encoding: ...=..., position_ids: ...=..., height_position_ids: ...=..., width_position_ids: ...=..., cu_seqlens: ...=..., ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 531-531: Class `SiglipVisionModel` overview
```python
class SiglipVisionModel(nn.Module):
```
**EN:** Defines `SiglipVisionModel` as a reusable runtime type derived from nn.Module. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SiglipVisionModel`，其继承关系为 nn.Module。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 532-533: Class `SiglipVisionModel` attributes
```python
    config_class = "PaddleOCRVisionConfig"
    main_input_name = "pixel_values"
```
**EN:** Defines class-level attributes and metadata that shape how `SiglipVisionModel` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `SiglipVisionModel` 在运行时的行为。

### Lines 535-548: Method `SiglipVisionModel.__init__`
```python
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.vision_model = SiglipVisionTransformer(
            config,
            quant_config=quant_config,
            prefix=add_prefix("vision_model", prefix),
        )
        self.quant_config = quant_config
```
**EN:** This method implements `__init__(config, quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 550-552: Method `SiglipVisionModel.dtype`
```python
    @property
    def dtype(self) -> torch.dtype:
        return self.vision_model.embeddings.patch_embedding.weight.dtype
```
**EN:** This method implements `dtype()` and implements a focused helper that supports the surrounding runtime flow inside `SiglipVisionModel` Decorators: property.
**CN:** 这个方法实现了 `dtype()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SiglipVisionModel` 内部调用 装饰器：property。

### Lines 554-556: Method `SiglipVisionModel.device`
```python
    @property
    def device(self) -> torch.device:
        return self.vision_model.embeddings.patch_embedding.weight.device
```
**EN:** This method implements `device()` and implements a focused helper that supports the surrounding runtime flow inside `SiglipVisionModel` Decorators: property.
**CN:** 这个方法实现了 `device()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SiglipVisionModel` 内部调用 装饰器：property。

### Lines 558-559: Method `SiglipVisionModel.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Module:
        return self.vision_model.embeddings.patch_embedding
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 561-583: Method `SiglipVisionModel.forward`
```python
    def forward(
        self,
        pixel_values,
        interpolate_pos_encoding: bool = False,
        position_ids: Optional[torch.Tensor] = None,
        image_grid_thw: Optional[
            List[
                Union[
                    Tuple[int, int, int],
                    List[Tuple[int, int, int]],
                ]
            ]
        ] = None,
        cu_seqlens: Optional[List[torch.Tensor]] = None,
    ) -> list[torch.Tensor]:

        return self.vision_model(
            pixel_values=pixel_values,
            interpolate_pos_encoding=interpolate_pos_encoding,
            position_ids=position_ids,
            image_grid_thw=image_grid_thw,
            cu_seqlens=cu_seqlens,
        )
```
**EN:** This method implements `forward(pixel_values, interpolate_pos_encoding: ...=..., position_ids: ...=..., image_grid_thw: ...=..., cu_seqlens: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(pixel_values, interpolate_pos_encoding: ...=..., position_ids: ...=..., image_grid_thw: ...=..., cu_seqlens: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 586-587: Class `PaddleOCRVLForConditionalGeneration` overview
```python
class PaddleOCRVLForConditionalGeneration(Ernie4_5_ForCausalLM):
```
**EN:** Defines `PaddleOCRVLForConditionalGeneration` as a reusable runtime type derived from Ernie4_5_ForCausalLM. The class groups 7 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PaddleOCRVLForConditionalGeneration`，其继承关系为 Ernie4_5_ForCausalLM。这个类组织了 7 个方法，用于实现模型相关行为。

### Lines 588-604: Method `PaddleOCRVLForConditionalGeneration.__init__`
```python
    def __init__(self, *, config, quant_config=None, prefix: str = ""):
        super().__init__(config=config, prefix=prefix)
        config = self.config

        self.mlp_AR = Projector(
            config, config.vision_config, prefix=add_prefix("mlp_AR", prefix)
        )
        self.visual = SiglipVisionModel(
            config=config.vision_config, prefix=add_prefix("visual", prefix)
        )
        if not hasattr(self.model, "get_input_embeddings"):
            import types

            self.model.get_input_embeddings = types.MethodType(
                get_input_embeddings, self.model
            )
        self.is_mrope_enabled = "mrope_section" in self.config.rope_scaling
```
**EN:** This method implements `__init__(*, config, quant_config=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, config, quant_config=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 606-608: Method `PaddleOCRVLForConditionalGeneration.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], mm_inputs: MultimodalInputs):
        pattern = MultiModalityDataPaddingPatternMultimodalTokens()
        return pattern.pad_input_tokens(input_ids, mm_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `PaddleOCRVLForConditionalGeneration`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `PaddleOCRVLForConditionalGeneration` 内部调用。

### Lines 610-611: Method `PaddleOCRVLForConditionalGeneration.get_input_embeddings`
```python
    def get_input_embeddings(self):
        return self.model.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 613-643: Method `PaddleOCRVLForConditionalGeneration.encode_image`
```python
    def encode_image(self, pixel_values, image_grid_thw):
        pixel_values = pixel_values.type(self.visual.dtype)
        siglip_position_ids = list()
        image_grid_hws = list()
        cu_seqlens = [0]

        for idx, grid_thw in enumerate(image_grid_thw):
            thw_tuple = tuple(grid_thw.detach().cpu().numpy().tolist())
            numel = np.prod(thw_tuple)
            image_grid_hws.append(thw_tuple)
            image_position_ids = torch.arange(numel) % np.prod(thw_tuple[1:])
            siglip_position_ids.append(image_position_ids)
            cu_seqlens.append(cu_seqlens[-1] + numel)

        siglip_position_ids = torch.concat(siglip_position_ids, dim=0).to(
            pixel_values.device
        )
        cu_seqlens = torch.tensor(cu_seqlens, dtype=torch.int32).to(pixel_values.device)
        vision_outputs = self.visual(
            pixel_values=pixel_values,
            image_grid_thw=image_grid_hws,
            position_ids=siglip_position_ids,
            interpolate_pos_encoding=True,
            cu_seqlens=cu_seqlens,
        )
        image_embeds = self.mlp_AR(vision_outputs, image_grid_thw)

        # image_embeds = torch.stack(image_embeds, dim=0)
        image_embeds = torch.cat(image_embeds, dim=0)

        return image_embeds
```
**EN:** This method implements `encode_image(pixel_values, image_grid_thw)` and implements a focused helper that supports the surrounding runtime flow inside `PaddleOCRVLForConditionalGeneration`.
**CN:** 这个方法实现了 `encode_image(pixel_values, image_grid_thw)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `PaddleOCRVLForConditionalGeneration` 内部调用。

### Lines 645-652: Method `PaddleOCRVLForConditionalGeneration.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        pixel_values = torch.cat([item.feature for item in items], dim=0).type(
            self.visual.dtype
        )
        image_grid_thw = torch.concat([item.image_grid_thw for item in items], dim=0)
        image_embeds = self.encode_image(pixel_values, image_grid_thw)

        return image_embeds
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 654-683: Method `PaddleOCRVLForConditionalGeneration.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        get_embedding: bool = False,
    ):
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
        )

        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch
        )
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., get_embedding: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., get_embedding: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 685-722: Method `PaddleOCRVLForConditionalGeneration.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]) -> Set[str]:
        stacked_params_mapping = [
            # (param_name, weight_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            (".gate_up_proj", ".gate_proj", 0),
            (".gate_up_proj", ".up_proj", 1),
        ]
        params_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            if "head.attention" in name or "head.layernorm" in name:
                continue
            if "head.mlp" in name or "head.probe" in name:
                continue

            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)
                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                if "vision_model" in name and "out_proj" in name:
                    # adapt to VisionAttention
                    name = name.replace(".self_attn.out_proj", ".self_attn.proj")
                if name in params_dict.keys():
                    param = params_dict[name]
                    weight_loader = getattr(
                        param, "weight_loader", default_weight_loader
                    )
                    weight_loader(param, loaded_weight)
                else:
                    raise KeyError(f"Parameter '{name}' not found in model.")
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 726-727: Function `get_input_embeddings`
```python
def get_input_embeddings(self) -> nn.Embedding:
    return self.embed_tokens
```
**EN:** This function implements `get_input_embeddings(self)` and prepares or accesses embedding representations used by the model.
**CN:** 这个函数实现了 `get_input_embeddings(self)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 730-730: Top-level assign
```python
EntryClass = [PaddleOCRVLForConditionalGeneration]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `collections.abc.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Set`
- `typing.Tuple`
- `typing.Union`
- `numpy`
- `torch`
- `torch.nn`
- `einops.rearrange`
- `transformers.activations.GELUActivation`
- `transformers.utils.torch_int`
- `sglang.srt.layers.activation.get_act_fn`
- `sglang.srt.layers.attention.vision.VisionAttention`
- `sglang.srt.layers.conv.Conv2dLayer`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternMultimodalTokens`
- `sglang.srt.managers.mm_utils.general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.ernie4.Ernie4_5_ForCausalLM`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.is_npu`
- `types`
