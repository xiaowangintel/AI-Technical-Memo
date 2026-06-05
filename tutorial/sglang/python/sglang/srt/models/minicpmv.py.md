# minicpmv.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/minicpmv.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only MiniCPM-V model compatible with HuggingFace weights. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 22-22: Module docstring
```python
"""Inference-only MiniCPM-V model compatible with HuggingFace weights."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 24-71: Module imports
```python
import types
from functools import partial
from itertools import chain
from typing import (
    Any,
    Callable,
    Iterable,
    List,
    Literal,
    Optional,
    Tuple,
    TypedDict,
    Union,
)

import numpy as np
import torch
import torch.types
from PIL import Image
from torch import nn
from torch.nn.init import trunc_normal_
from transformers import PretrainedConfig

from sglang.srt.layers.linear import ReplicatedLinear
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.managers.mm_utils import (
    MultiModalityDataPaddingPatternTokenPairs,
    general_mm_embed_routine,
)
from sglang.srt.managers.schedule_batch import (
    MultimodalDataItem,
    MultimodalInputFormat,
    MultimodalInputs,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.utils import set_default_torch_dtype
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.idefics2 import Idefics2VisionTransformer
from sglang.srt.models.llama import LlamaConfig, LlamaForCausalLM
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 73-73: Top-level assign
```python
RawImageType = Union[Image.Image, torch.Tensor]
```
**EN:** Defines or updates RawImageType, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 RawImageType，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 78-102: Function `get_1d_sincos_pos_embed_from_grid`
```python
def get_1d_sincos_pos_embed_from_grid(
    embed_dim: int, pos: np.ndarray, version: Tuple[int, int] = (2, 0)
) -> torch.Tensor:
    """
    embed_dim: output dimension for each position
    pos: a list of positions to be encoded: size (M,) / (H, W)
    out: (M, D) / (H, W, D)
    """
    assert embed_dim % 2 == 0
    omega = np.arange(embed_dim // 2, dtype=np.float32)
    omega /= embed_dim / 2.0
    omega = 1.0 / 10000**omega  # (D/2,)

    if version == (2, 0):
        pos = pos.reshape(-1)  # (M,)
        out = np.einsum("m,d->md", pos, omega)  # (M, D/2), outer product
        emb_sin = np.sin(out)  # (M, D/2)
        emb_cos = np.cos(out)  # (M, D/2)
        emb = np.concatenate([emb_sin, emb_cos], axis=1)  # (M, D)
    else:
        out = np.einsum("hw,d->hwd", pos, omega)  # (H, W, D/2), outer product
        emb_sin = np.sin(out)  # (H, W, D/2)
        emb_cos = np.cos(out)  # (H, W, D/2)
        emb = np.concatenate([emb_sin, emb_cos], axis=-1)  # (H, W, D)
    return emb
```
**EN:** This function implements `get_1d_sincos_pos_embed_from_grid(embed_dim: ..., pos: ..., version: ...=...)` and embed_dim: output dimension for each position.
**CN:** 这个函数实现了 `get_1d_sincos_pos_embed_from_grid(embed_dim: ..., pos: ..., version: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 105-122: Function `get_2d_sincos_pos_embed_from_grid`
```python
def get_2d_sincos_pos_embed_from_grid(
    embed_dim: int, grid: np.ndarray, version: Tuple[int, int] = (2, 0)
) -> torch.Tensor:
    assert embed_dim % 2 == 0

    # use half of dimensions to encode grid_h
    emb_h = get_1d_sincos_pos_embed_from_grid(
        embed_dim // 2, grid[0], version
    )  # (H*W, D/2) or (H, W, D/2)
    emb_w = get_1d_sincos_pos_embed_from_grid(
        embed_dim // 2, grid[1], version
    )  # (H*W, D/2) or (H, W, D/2)

    if version == (2, 0):
        emb = np.concatenate([emb_h, emb_w], axis=1)  # (H*W, D)
    else:
        emb = np.concatenate([emb_h, emb_w], axis=-1)  # (H, W, D)
    return emb
```
**EN:** This function implements `get_2d_sincos_pos_embed_from_grid(embed_dim: ..., grid: ..., version: ...=...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个函数实现了 `get_2d_sincos_pos_embed_from_grid(embed_dim: ..., grid: ..., version: ...=...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 125-155: Function `get_2d_sincos_pos_embed`
```python
def get_2d_sincos_pos_embed(
    embed_dim: int,
    grid_size: Union[int, Tuple[int, int]],
    cls_token: bool = False,
    version: Tuple[int, int] = (2, 0),
) -> torch.Tensor:
    """
    grid_size: int of the grid height and width
    return:
    pos_embed: [grid_size*grid_size, embed_dim] or
                [1+grid_size*grid_size, embed_dim] (w/ or w/o cls_token)
    """
    if isinstance(grid_size, int):
        grid_h_size, grid_w_size = grid_size, grid_size
    else:
        grid_h_size, grid_w_size = grid_size[0], grid_size[1]

    grid_h = np.arange(grid_h_size, dtype=np.float32)
    grid_w = np.arange(grid_w_size, dtype=np.float32)
    grid = np.meshgrid(grid_w, grid_h)  # here w goes first
    grid = np.stack(grid, axis=0)
    assert isinstance(grid, np.ndarray) and grid.shape == (2, grid_h_size, grid_w_size)

    if version == (2, 0):
        grid = grid.reshape([2, 1, grid_h_size, grid_w_size])
        pos_embed = get_2d_sincos_pos_embed_from_grid(embed_dim, grid, version)
        if cls_token:
            pos_embed = np.concatenate([np.zeros([1, embed_dim]), pos_embed], axis=0)
    else:
        pos_embed = get_2d_sincos_pos_embed_from_grid(embed_dim, grid, version)
    return pos_embed
```
**EN:** This function implements `get_2d_sincos_pos_embed(embed_dim: ..., grid_size: ..., cls_token: ...=..., version: ...=...)` and grid_size: int of the grid height and width.
**CN:** 这个函数实现了 `get_2d_sincos_pos_embed(embed_dim: ..., grid_size: ..., cls_token: ...=..., version: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 158-158: Class `MiniCPMVImagePixelInputs` overview
```python
class MiniCPMVImagePixelInputs(TypedDict):
```
**EN:** Defines `MiniCPMVImagePixelInputs` as a reusable runtime type derived from TypedDict. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMVImagePixelInputs`，其继承关系为 TypedDict。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 159-175: Class `MiniCPMVImagePixelInputs` attributes
```python
    type: Literal["pixel_values"]
    data: List[torch.Tensor]
    """
    Shape: `(batch_size * num_images, num_channels, height, width)`

    Note that the image size may vary, so we pass it as a list
    instead of a batched tensor.
    """

    image_bounds: torch.Tensor
    """
    Shape: `(batch_size * num_images, 2)`

    This should be in `(start, stop)` format.
    """

    tgt_sizes: torch.Tensor
```
**EN:** Defines class-level attributes and metadata that shape how `MiniCPMVImagePixelInputs` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiniCPMVImagePixelInputs` 在运行时的行为。

### Lines 183-183: Class `MiniCPMVImageEmbeddingInputs` overview
```python
class MiniCPMVImageEmbeddingInputs(TypedDict):
```
**EN:** Defines `MiniCPMVImageEmbeddingInputs` as a reusable runtime type derived from TypedDict. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMVImageEmbeddingInputs`，其继承关系为 TypedDict。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 184-193: Class `MiniCPMVImageEmbeddingInputs` attributes
```python
    type: Literal["image_embeds"]
    data: torch.Tensor
    """
    Shape: `(batch_size * num_images, image_feature_size, hidden_size)`

    `hidden_size` must match the hidden size of language model backbone.
    instead of a batched tensor.
    """

    image_bounds: torch.Tensor
```
**EN:** Defines class-level attributes and metadata that shape how `MiniCPMVImageEmbeddingInputs` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiniCPMVImageEmbeddingInputs` 在运行时的行为。

### Lines 201-201: Top-level assign
```python
MiniCPMVImageInputs = Union[MiniCPMVImagePixelInputs, MiniCPMVImageEmbeddingInputs]
```
**EN:** Defines or updates MiniCPMVImageInputs, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 MiniCPMVImageInputs，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 203-203: Top-level assign
```python
DEFAULT_LN = partial(nn.LayerNorm, eps=1e-6)
```
**EN:** Defines or updates DEFAULT_LN, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 DEFAULT_LN，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 206-213: Class `BaseResampler` overview
```python
class BaseResampler(nn.Module):
    """
    A 2D perceiver-resampler network with one cross attention layers by
        (grid_size**2) learnable queries and 2d sincos pos_emb.
    Outputs:
        A tensor with the shape of (grid_size**2, embed_dim)
    """
```
**EN:** Defines `BaseResampler` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `BaseResampler`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 214-256: Method `BaseResampler.__init__`
```python
    def __init__(
        self,
        num_queries: int,
        embed_dim: int,
        num_heads: int,
        kv_dim: Optional[int] = None,
        norm_layer: Callable[[int], nn.LayerNorm] = DEFAULT_LN,
        do_post_projection: bool = True,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.num_queries = num_queries
        self.embed_dim = embed_dim
        self.num_heads = num_heads

        self.query = nn.Parameter(torch.zeros(self.num_queries, embed_dim))
        trunc_normal_(self.query, std=0.02)
        if kv_dim is not None and kv_dim != embed_dim:
            self.kv_proj = ReplicatedLinear(
                kv_dim,
                embed_dim,
                bias=False,
                quant_config=quant_config,
                prefix=add_prefix("kv_proj", prefix),
            )
        else:
            # Maintain the same return value with ReplicatedLinear.forward
            self.kv_proj = lambda *args, **kwargs: (  # type: ignore # noqa
                nn.Identity()(*args, **kwargs),
                None,
            )
        self.attn = nn.MultiheadAttention(embed_dim, num_heads)
        self.ln_q = norm_layer(embed_dim)
        self.ln_kv = norm_layer(embed_dim)
        self.do_post_projection = do_post_projection
        self.ln_post = norm_layer(embed_dim) if do_post_projection else None
        self.proj = (
            nn.Parameter((embed_dim**-0.5) * torch.randn(embed_dim, embed_dim))
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(num_queries: ..., embed_dim: ..., num_heads: ..., kv_dim: ...=..., norm_layer: ...=..., do_post_projection: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(num_queries: ..., embed_dim: ..., num_heads: ..., kv_dim: ...=..., norm_layer: ...=..., do_post_projection: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 258-265: Method `BaseResampler._init_weights`
```python
    def _init_weights(self, m: nn.Module) -> None:
        if isinstance(m, nn.Linear):
            trunc_normal_(m.weight, std=0.02)
            if isinstance(m, nn.Linear) and m.bias is not None:
                nn.init.constant_(m.bias, 0)
        elif isinstance(m, nn.LayerNorm):
            nn.init.constant_(m.bias, 0)
            nn.init.constant_(m.weight, 1.0)
```
**EN:** This method implements `_init_weights(m: ...)` and handles weight mapping, filtering, or loading for this model component.
**CN:** 这个方法实现了 `_init_weights(m: ...)`，其作用是处理该模型组件的权重映射、筛选或加载逻辑。

### Lines 267-268: Method `BaseResampler._repeat`
```python
    def _repeat(self, query, N: int):
        return query.unsqueeze(1).repeat(1, N, 1)
```
**EN:** This method implements `_repeat(query, N: ...)` and implements a focused helper that supports the surrounding runtime flow inside `BaseResampler`.
**CN:** 这个方法实现了 `_repeat(query, N: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `BaseResampler` 内部调用。

### Lines 271-272: Class `Resampler2_5` overview
```python
class Resampler2_5(BaseResampler):
```
**EN:** Defines `Resampler2_5` as a reusable runtime type derived from BaseResampler. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Resampler2_5`，其继承关系为 BaseResampler。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 273-297: Method `Resampler2_5.__init__`
```python
    def __init__(
        self,
        num_queries: int,
        embed_dim: int,
        num_heads: int,
        kv_dim: Optional[int] = None,
        norm_layer: Callable[[int], nn.LayerNorm] = DEFAULT_LN,
        max_size: Tuple[int, int] = (70, 70),
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__(
            num_queries,
            embed_dim,
            num_heads,
            kv_dim,
            norm_layer,
            quant_config=quant_config,
            prefix=prefix,
        )

        self.max_size = max_size
        self._set_2d_pos_cache(self.max_size)

        self.apply(self._init_weights)
```
**EN:** This method implements `__init__(num_queries: ..., embed_dim: ..., num_heads: ..., kv_dim: ...=..., norm_layer: ...=..., max_size: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(num_queries: ..., embed_dim: ..., num_heads: ..., kv_dim: ...=..., norm_layer: ...=..., max_size: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 299-306: Method `Resampler2_5._set_2d_pos_cache`
```python
    def _set_2d_pos_cache(
        self, max_size: Tuple[int, int], device: torch.types.Device = "cpu"
    ) -> None:
        pos_embed_arr = get_2d_sincos_pos_embed(
            self.embed_dim, max_size, version=(2, 5)
        )
        pos_embed = torch.from_numpy(pos_embed_arr).float().to(device)
        self.register_buffer("pos_embed", pos_embed, persistent=False)
```
**EN:** This method implements `_set_2d_pos_cache(max_size: ..., device: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `Resampler2_5`.
**CN:** 这个方法实现了 `_set_2d_pos_cache(max_size: ..., device: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Resampler2_5` 内部调用。

### Lines 308-320: Method `Resampler2_5._adjust_pos_cache`
```python
    def _adjust_pos_cache(
        self, tgt_sizes: torch.Tensor, device: torch.types.Device
    ) -> None:
        max_h = tgt_sizes[:, 0].max().item()
        max_w = tgt_sizes[:, 1].max().item()
        assert isinstance(max_h, int) and isinstance(max_w, int)

        if max_h > self.max_size[0] or max_w > self.max_size[1]:
            self.max_size = (
                max(max_h, self.max_size[0]),
                max(max_w, self.max_size[1]),
            )
            self._set_2d_pos_cache(self.max_size, device)
```
**EN:** This method implements `_adjust_pos_cache(tgt_sizes: ..., device: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Resampler2_5`.
**CN:** 这个方法实现了 `_adjust_pos_cache(tgt_sizes: ..., device: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Resampler2_5` 内部调用。

### Lines 322-368: Method `Resampler2_5.forward`
```python
    def forward(self, x: torch.Tensor, tgt_sizes: torch.Tensor) -> torch.Tensor:
        assert x.shape[0] == tgt_sizes.shape[0]
        bs = x.shape[0]

        device = x.device
        dtype = x.dtype

        patch_len = tgt_sizes[:, 0] * tgt_sizes[:, 1]

        self._adjust_pos_cache(tgt_sizes, device=device)

        max_patch_len = patch_len.max().item()
        assert isinstance(max_patch_len, int)

        key_padding_mask = torch.zeros(
            (bs, max_patch_len), dtype=torch.bool, device=device
        )

        pos_embed = []
        for i in range(bs):
            tgt_h, tgt_w = tgt_sizes[i].tolist()
            pos_embed.append(
                self.pos_embed[:tgt_h, :tgt_w, :].reshape((tgt_h * tgt_w, -1)).to(dtype)
            )  # patches * D
            key_padding_mask[i, patch_len[i] :] = True
        pos_embed = torch.nn.utils.rnn.pad_sequence(
            pos_embed, batch_first=True, padding_value=0.0
        ).permute(
            1, 0, 2
        )  # BLD => L * B * D
        x, _ = self.kv_proj(x)  # B * L * D
        x = self.ln_kv(x).permute(1, 0, 2)  # L * B * D

        q = self.ln_q(self.query)  # Q * D

        out = self.attn(
            self._repeat(q, bs),  # Q * B * D
            x + pos_embed,  # L * B * D +  L * B * D
            x,
            key_padding_mask=key_padding_mask,
# ... truncated for brevity ...
```
**EN:** This method implements `forward(x: ..., tgt_sizes: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ..., tgt_sizes: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 371-372: Class `Resampler4_5` overview
```python
class Resampler4_5(BaseResampler):
```
**EN:** Defines `Resampler4_5` as a reusable runtime type derived from BaseResampler. The class groups 7 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Resampler4_5`，其继承关系为 BaseResampler。这个类组织了 7 个方法，用于实现模型相关行为。

### Lines 373-400: Method `Resampler4_5.__init__`
```python
    def __init__(
        self,
        num_queries: int,
        embed_dim: int,
        num_heads: int,
        kv_dim: Optional[int] = None,
        norm_layer: Callable[[int], nn.LayerNorm] = DEFAULT_LN,
        max_size: tuple[int, int] = (70, 70),
        max_temporal_size=36000,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__(
            num_queries,
            embed_dim,
            num_heads,
            kv_dim,
            norm_layer,
            quant_config=quant_config,
            prefix=prefix,
        )

        self.max_size = max_size
        self.max_temporal_size = max_temporal_size

        self._set_2d_pos_cache(self.max_size)
        self._set_temporal_pos_cache(self.max_temporal_size)
        self.apply(self._init_weights)
```
**EN:** This method implements `__init__(num_queries: ..., embed_dim: ..., num_heads: ..., kv_dim: ...=..., norm_layer: ...=..., max_size: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(num_queries: ..., embed_dim: ..., num_heads: ..., kv_dim: ...=..., norm_layer: ...=..., max_size: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 402-422: Method `Resampler4_5.get_1d_sincos_pos_embed_from_temporal_size`
```python
    def get_1d_sincos_pos_embed_from_temporal_size(
        self, embed_dim: int, pos: np.ndarray
    ):
        """
        embed_dim: output dimension for each position
        pos: a list of positions to be encoded: size (M,)
        out: (M, D)
        """
        assert embed_dim % 2 == 0
        omega = np.arange(embed_dim // 2, dtype=np.float32)
        omega /= embed_dim / 2.0
        omega = 1.0 / 10000**omega  # (D/2,)

        pos = pos.reshape(-1)  # (M,)
        out = np.einsum("m,d->md", pos, omega)  # (M, D/2), outer product

        emb_sin = np.sin(out)  # (M, D/2)
        emb_cos = np.cos(out)  # (M, D/2)

        emb = np.concatenate([emb_sin, emb_cos], axis=1)  # (M, D)
        return emb
```
**EN:** This method implements `get_1d_sincos_pos_embed_from_temporal_size(embed_dim: ..., pos: ...)` and embed_dim: output dimension for each position.
**CN:** 这个方法实现了 `get_1d_sincos_pos_embed_from_temporal_size(embed_dim: ..., pos: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 424-431: Method `Resampler4_5._set_2d_pos_cache`
```python
    def _set_2d_pos_cache(
        self, max_size: tuple[int, int], device: torch.types.Device = "cpu"
    ) -> None:
        pos_embed_arr = get_2d_sincos_pos_embed(
            self.embed_dim, max_size, version=(2, 5)
        )
        pos_embed = torch.from_numpy(pos_embed_arr).float().to(device)
        self.register_buffer("pos_embed", pos_embed, persistent=False)
```
**EN:** This method implements `_set_2d_pos_cache(max_size: ..., device: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `Resampler4_5`.
**CN:** 这个方法实现了 `_set_2d_pos_cache(max_size: ..., device: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Resampler4_5` 内部调用。

### Lines 433-445: Method `Resampler4_5._adjust_pos_cache`
```python
    def _adjust_pos_cache(
        self, tgt_sizes: torch.Tensor, device: torch.types.Device
    ) -> None:
        max_h = tgt_sizes[:, 0].max().item()
        max_w = tgt_sizes[:, 1].max().item()
        assert isinstance(max_h, int) and isinstance(max_w, int)

        if max_h > self.max_size[0] or max_w > self.max_size[1]:
            self.max_size = (
                max(max_h, self.max_size[0]),
                max(max_w, self.max_size[1]),
            )
            self._set_2d_pos_cache(self.max_size, device)
```
**EN:** This method implements `_adjust_pos_cache(tgt_sizes: ..., device: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Resampler4_5`.
**CN:** 这个方法实现了 `_adjust_pos_cache(tgt_sizes: ..., device: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Resampler4_5` 内部调用。

### Lines 447-460: Method `Resampler4_5._set_temporal_pos_cache`
```python
    def _set_temporal_pos_cache(
        self, max_temporal_size: int, device: torch.types.Device = "cpu"
    ) -> None:
        temporal_size = np.arange(max_temporal_size, dtype=np.float32)
        pos_embed = (
            torch.from_numpy(
                self.get_1d_sincos_pos_embed_from_temporal_size(
                    self.embed_dim, temporal_size
                )
            )
            .float()
            .to(device)
        )
        self.register_buffer("temporal_pos_embed", pos_embed, persistent=False)
```
**EN:** This method implements `_set_temporal_pos_cache(max_temporal_size: ..., device: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `Resampler4_5`.
**CN:** 这个方法实现了 `_set_temporal_pos_cache(max_temporal_size: ..., device: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Resampler4_5` 内部调用。

### Lines 462-467: Method `Resampler4_5._adjust_temporal_pos_cache`
```python
    def _adjust_temporal_pos_cache(
        self, max_temporal_size: int, device: torch.types.Device = "cpu"
    ):
        if max_temporal_size > self.max_temporal_size:
            self.max_temporal_size = max_temporal_size
            self._set_temporal_pos_cache(self.max_temporal_size, device)
```
**EN:** This method implements `_adjust_temporal_pos_cache(max_temporal_size: ..., device: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `Resampler4_5`.
**CN:** 这个方法实现了 `_adjust_temporal_pos_cache(max_temporal_size: ..., device: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Resampler4_5` 内部调用。

### Lines 469-580: Method `Resampler4_5.forward`
```python
    def forward(
        self, x: torch.Tensor, tgt_sizes: torch.Tensor, temporal_ids=None
    ) -> torch.Tensor:
        assert x.shape[0] == tgt_sizes.shape[0]
        bs = x.shape[0]

        device = x.device
        dtype = x.dtype

        patch_len = tgt_sizes[:, 0] * tgt_sizes[:, 1]

        self._adjust_pos_cache(tgt_sizes, device=device)

        temporal_pos_emb = False
        temporal_ids_flatten = None
        if temporal_ids is not None:
            # example: [[-1], [-1], [2, 6, 9]]
            temporal_ids_flatten = list(chain.from_iterable(temporal_ids))
            max_temporal_size = max(temporal_ids_flatten)
            if max_temporal_size > -1:
                temporal_pos_emb = True
            if max_temporal_size > self.max_temporal_size:
                self._adjust_temporal_pos_cache(max_temporal_size, device)

        max_patch_len = patch_len.max().item()
        assert isinstance(max_patch_len, int)

        key_padding_mask = torch.zeros(
            (bs, max_patch_len), dtype=torch.bool, device=device
        )

        x, _ = self.kv_proj(x)  # B * L * D
        x = self.ln_kv(x).permute(1, 0, 2)  # L * B * D
        q = self.ln_q(self.query)  # Q * D

        pos_embed_2d = []
        pos_embed_temporal = []
        for i in range(bs):
            tgt_h, tgt_w = tgt_sizes[i]
            if temporal_pos_emb:
# ... truncated for brevity ...
```
**EN:** This method implements `forward(x: ..., tgt_sizes: ..., temporal_ids=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ..., tgt_sizes: ..., temporal_ids=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 583-598: Function `get_version_by_config`
```python
def get_version_by_config(config: PretrainedConfig) -> Tuple[int, ...]:
    # 4.6 ships its own ``model_type`` instead of a numeric ``version``.
    if getattr(config, "model_type", None) == "minicpmv4_6":
        return 4, 6

    version_float = getattr(config, "version", None)

    # The old configs do not include version number
    # TODO: Remove this after the HF repos are updated
    if version_float is None:
        if config.hidden_size == 2304 and config.query_num == 64:
            return 2, 0
        return 2, 5

    version_str = str(version_float)
    return tuple(int(x) for x in version_str.split("."))
```
**EN:** This function implements `get_version_by_config(config: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个函数实现了 `get_version_by_config(config: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 601-606: Class `MiniCPMBaseModel` overview
```python
class MiniCPMBaseModel(nn.Module):
    """
    The abstract class of MiniCPMV can only be inherited, but cannot be
    instantiated.
    """
```
**EN:** Defines `MiniCPMBaseModel` as a reusable runtime type derived from nn.Module. The class groups 11 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMBaseModel`，其继承关系为 nn.Module。这个类组织了 11 个方法，用于实现模型相关行为。

### Lines 607-642: Method `MiniCPMBaseModel.__init__`
```python
    def __init__(
        self,
        *,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        # All MiniCPM-V models disable `tie_word_embeddings` but
        # `PretrainedConfig.tie_word_embeddings` defaults to True; we cannot
        # check `tie_word_embeddings` until SGLang integrate MiniCPM-V model
        # and config class
        self.config = config

        self.version = get_version_by_config(self.config)
        self.llm = self.init_llm(
            config=config, quant_config=quant_config, prefix=add_prefix("llm", prefix)
        )
        self.vpm = self.init_vision_module(
            config, quant_config, add_prefix("vpm", prefix)
        )
        self.vision_dim = (
            self.vpm.embed_dim
            if self.version == (2, 0)
            else self.vpm.embeddings.embed_dim
        )
        self.embed_dim = self.config.hidden_size

        self.resampler = self.init_resampler(
            self.embed_dim,
            self.vision_dim,
            quant_config=quant_config,
            prefix=add_prefix("resampler", prefix),
        )

        self.logits_processor = LogitsProcessor(config)
```
**EN:** This method implements `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 644-701: Method `MiniCPMBaseModel._get_image_bounds`
```python
    def _get_image_bounds(
        self,
        input_ids: torch.Tensor,
        pad_values: List[int],
        im_start_id: int,
        im_end_id: int,
        slice_start_id: Optional[int] = None,
        slice_end_id: Optional[int] = None,
    ) -> torch.Tensor:
        """
        Returns a tensor indicating the bounds (start and end token ids) of the images
        """
        # All the images in the batch should share the same special image
        # bound token ids.
        start_cond = input_ids == im_start_id
        end_cond = input_ids == im_end_id
        if slice_start_id is not None:
            start_cond |= input_ids == slice_start_id
            end_cond |= input_ids == slice_end_id

        (image_start_tokens,) = torch.where(start_cond)
        image_start_tokens += 1
        (image_end_tokens,) = torch.where(end_cond)

        # the im_start_id sometimes can be cached as prefix, but it is needed for the embedding of the images
        if len(image_start_tokens) != len(image_end_tokens):
            if (
                len(image_start_tokens) + 1 == len(image_end_tokens)
                and input_ids[0] in pad_values
                and len(image_start_tokens) != 0
                and len(image_end_tokens) != 0
                and image_end_tokens[0] < image_start_tokens[0]
            ):
                image_start_tokens = torch.cat(
                    [
                        torch.tensor([0], device=image_start_tokens.device),
                        image_start_tokens,
                    ]
                )
        valid_image_nums = min(len(image_start_tokens), len(image_end_tokens))
# ... truncated for brevity ...
```
**EN:** This method implements `_get_image_bounds(input_ids: ..., pad_values: ..., im_start_id: ..., im_end_id: ..., slice_start_id: ...=..., slice_end_id: ...=...)` and Returns a tensor indicating the bounds (start and end token ids) of the images.
**CN:** 这个方法实现了 `_get_image_bounds(input_ids: ..., pad_values: ..., im_start_id: ..., im_end_id: ..., slice_start_id: ...=..., slice_end_id: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 703-754: Method `MiniCPMBaseModel._parse_and_validate_inputs`
```python
    def _parse_and_validate_inputs(
        self,
        input_ids: torch.Tensor,
        **kwargs: object,
    ) -> Optional[MiniCPMVImageInputs]:
        pixel_values = kwargs.pop("pixel_values", [])
        tgt_sizes = kwargs.pop("tgt_sizes", [])
        im_start_id = kwargs.pop("im_start_id", None)
        im_end_id = kwargs.pop("im_end_id", None)
        slice_start_id = kwargs.pop("slice_start_id", None)
        slice_end_id = kwargs.pop("slice_end_id", None)
        image_embeds = kwargs.pop("image_embeds", None)
        pad_values = kwargs.pop("pad_values", None)

        if image_embeds is not None:
            image_bounds = self._get_image_bounds(
                input_ids=input_ids,
                pad_values=pad_values,
                im_start_id=im_start_id,
                im_end_id=im_end_id,
                slice_start_id=slice_start_id,
                slice_end_id=slice_end_id,
            )
            if not isinstance(image_embeds, (torch.Tensor, list)):
                raise ValueError(
                    f"Incorrect type of image embeds. "
                    f"Got type: {type(image_embeds)}"
                )

            if isinstance(image_embeds, list):
                image_embeds = torch.cat(image_embeds)

            return MiniCPMVImageEmbeddingInputs(
                image_bounds=image_bounds,
                data=image_embeds,
                type="image_embeds",
            )

        image_bounds = self._get_image_bounds(
            input_ids=input_ids,
# ... truncated for brevity ...
```
**EN:** This method implements `_parse_and_validate_inputs(input_ids: ..., **kwargs)` and implements a focused helper that supports the surrounding runtime flow inside `MiniCPMBaseModel`.
**CN:** 这个方法实现了 `_parse_and_validate_inputs(input_ids: ..., **kwargs)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiniCPMBaseModel` 内部调用。

### Lines 756-790: Method `MiniCPMBaseModel.get_embedding`
```python
    def get_embedding(
        self,
        input_ids: torch.Tensor,
        image_inputs: Optional[MiniCPMVImageInputs],
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        vlm_embedding: torch.Tensor = self.llm.get_input_embeddings(input_ids)

        if image_inputs is None:  # No image
            vision_hidden_states = torch.tensor([], device=input_ids.device)
        else:
            if image_inputs["type"] == "image_embeds":
                vision_hidden_states = (
                    image_inputs["data"]
                    .type(vlm_embedding.dtype)
                    .to(vlm_embedding.device)
                )
            else:
                vision_hidden_states = self.get_vision_hidden_states(image_inputs)
            # See NOTE in _parse_and_validate_inputs
            image_bounds = image_inputs["image_bounds"]
            if len(image_bounds) > 0:
                image_indices = torch.stack(
                    [
                        torch.arange(start, end, dtype=torch.long)
                        for start, end in image_bounds.tolist()
                    ]
                ).to(vlm_embedding.device)

                vlm_embedding.scatter_(
                    0,
                    image_indices.view(-1, 1).repeat(1, vlm_embedding.shape[-1]),
                    vision_hidden_states.view(-1, vision_hidden_states.shape[-1]),
                )

        return vlm_embedding, vision_hidden_states
```
**EN:** This method implements `get_embedding(input_ids: ..., image_inputs: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embedding(input_ids: ..., image_inputs: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 792-793: Method `MiniCPMBaseModel.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Embedding:
        return self.llm.get_input_embeddings()
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 795-809: Method `MiniCPMBaseModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        **kwargs: Any,
    ) -> torch.Tensor:
        hidden_states = general_mm_embed_routine(
            input_ids=input_ids,
            forward_batch=forward_batch,
            multimodal_model=self,
            language_model=self.llm,
            positions=positions,
        )
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 811-817: Method `MiniCPMBaseModel.init_llm`
```python
    def init_llm(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        raise NotImplementedError
```
**EN:** This method implements `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 819-825: Method `MiniCPMBaseModel.init_vision_module`
```python
    def init_vision_module(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig],
        prefix: str = "",
    ) -> nn.Module:
        raise NotImplementedError
```
**EN:** This method implements `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 827-834: Method `MiniCPMBaseModel.init_resampler`
```python
    def init_resampler(
        self,
        embed_dim: int,
        vision_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        raise NotImplementedError
```
**EN:** This method implements `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 836-842: Method `MiniCPMBaseModel.get_vision_embedding`
```python
    def get_vision_embedding(
        self,
        pixel_values: List[torch.Tensor],
        patch_attn_mask: Optional[torch.Tensor] = None,
        tgt_sizes: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** This method implements `get_vision_embedding(pixel_values: ..., patch_attn_mask: ...=..., tgt_sizes: ...=...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_vision_embedding(pixel_values: ..., patch_attn_mask: ...=..., tgt_sizes: ...=...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 844-845: Method `MiniCPMBaseModel.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 848-848: Class `MiniCPMV2_6` overview
```python
class MiniCPMV2_6(MiniCPMBaseModel):
```
**EN:** Defines `MiniCPMV2_6` as a reusable runtime type derived from MiniCPMBaseModel. The class groups 7 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV2_6`，其继承关系为 MiniCPMBaseModel。这个类组织了 7 个方法，用于实现模型相关行为。

### Lines 849-886: Class `MiniCPMV2_6` attributes
```python
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
    # LoRA specific attributes
    supported_lora_modules = [
        # vision encoder
        "fc1",
        "fc2",
        "out_proj",
        # language model
        "qkv_proj",  # same name with vision encoder
        "o_proj",
        "gate_up_proj",
        "down_proj",
        # resampler
        "kv_proj",
    ]

    # BitandBytes specific attributes
    bitsandbytes_stacked_params_mapping = {
        # shard_name, weight_name, index
        "q_proj": ("qkv_proj", 0),
        "k_proj": ("qkv_proj", 1),
        "v_proj": ("qkv_proj", 2),
        "gate_proj": ("gate_up_proj", 0),
        "up_proj": ("gate_up_proj", 1),
    }

    embedding_modules = {}
    embedding_padding_modules = []
```
**EN:** Defines class-level attributes and metadata that shape how `MiniCPMV2_6` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiniCPMV2_6` 在运行时的行为。

### Lines 888-895: Method `MiniCPMV2_6.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__(config=config, quant_config=quant_config, prefix=prefix)
        assert self.version == (2, 6)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 897-903: Method `MiniCPMV2_6.init_llm`
```python
    def init_llm(
        self,
        config: Qwen2Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        return Qwen2ForCausalLM(config=config, quant_config=quant_config, prefix=prefix)
```
**EN:** This method implements `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 905-919: Method `MiniCPMV2_6.init_vision_module`
```python
    def init_vision_module(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig],
        prefix: str = "",
    ) -> nn.Module:
        model = Idefics2VisionTransformer(
            config=config.vision_config, quant_config=quant_config, prefix=prefix
        )
        if self.config.drop_vision_last_layer:
            model.encoder.layers = model.encoder.layers[:-1]

        setattr(model, "embed_dim", model.embeddings.embed_dim)
        setattr(model, "patch_size", model.embeddings.patch_size)
        return model
```
**EN:** This method implements `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 921-939: Method `MiniCPMV2_6.init_resampler`
```python
    def init_resampler(
        self,
        embed_dim: int,
        vision_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        with set_default_torch_dtype(torch.float16):
            # The resampler in 2.6 remains consistent with the one in 2.5.
            resampler = Resampler2_5(
                num_queries=self.config.query_num,
                embed_dim=embed_dim,
                num_heads=embed_dim // 128,
                kv_dim=vision_dim,
                quant_config=quant_config,
                prefix=prefix,
            )

        return resampler.to(device="cuda", dtype=torch.get_default_dtype())
```
**EN:** This method implements `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 941-952: Method `MiniCPMV2_6.get_vision_embedding`
```python
    def get_vision_embedding(
        self,
        pixel_values: List[torch.Tensor],
        patch_attn_mask: Optional[torch.Tensor] = None,
        tgt_sizes: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        vision_embedding = self.vpm(
            pixel_values,
            patch_attention_mask=patch_attn_mask,
            tgt_sizes=tgt_sizes,
        )
        return vision_embedding
```
**EN:** This method implements `get_vision_embedding(pixel_values: ..., patch_attn_mask: ...=..., tgt_sizes: ...=...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_vision_embedding(pixel_values: ..., patch_attn_mask: ...=..., tgt_sizes: ...=...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 954-995: Method `MiniCPMV2_6.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        if items and items[0].format == MultimodalInputFormat.PRECOMPUTED_EMBEDDING:
            result = torch.cat([item.feature for item in items])
            return result.reshape(-1, result.shape[-1])

        # list of tensors
        pixel_values = flatten_nested_list([item.feature for item in items])
        tgt_sizes = torch.stack(
            flatten_nested_list([item.tgt_size for item in items]), dim=0
        )
        assert len(pixel_values) == tgt_sizes.shape[0]

        device = self.vpm.embeddings.position_embedding.weight.device
        dtype = self.vpm.embeddings.position_embedding.weight.dtype
        all_pixel_values_lst = [
            i.flatten(end_dim=1).permute(1, 0) for i in pixel_values
        ]

        max_patches = (tgt_sizes[:, 0] * tgt_sizes[:, 1]).max().item()
        assert isinstance(max_patches, int)
        all_pixel_values = torch.nn.utils.rnn.pad_sequence(
            all_pixel_values_lst, batch_first=True, padding_value=0.0
        )

        B, L, _ = all_pixel_values.shape
        all_pixel_values = all_pixel_values.permute(0, 2, 1).reshape(B, 3, -1, L)
        patch_attn_mask = torch.zeros(
            (B, 1, max_patches), dtype=torch.bool, device=device
        )

        tgt_sizes_tensor = tgt_sizes.clone().to(device=patch_attn_mask.device)
        mask_shapes = tgt_sizes_tensor[:, 0] * tgt_sizes_tensor[:, 1]
        patch_attn_mask[:, 0, :] = torch.arange(
            patch_attn_mask.size(2), device=patch_attn_mask.device
        ).unsqueeze(0) < mask_shapes.unsqueeze(1)

        vision_embedding = self.vpm(
            all_pixel_values.type(dtype),
            patch_attention_mask=patch_attn_mask,
            tgt_sizes=tgt_sizes,
# ... truncated for brevity ...
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 997-1011: Method `MiniCPMV2_6.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], image_inputs: MultimodalInputs):
        # Get all special token IDs
        im_start_id: int = image_inputs.im_start_id
        im_end_id: int = image_inputs.im_end_id
        slice_start_id: int = image_inputs.slice_start_id
        slice_end_id: int = image_inputs.slice_end_id

        media_token_pairs = [(im_start_id, im_end_id), (slice_start_id, slice_end_id)]
        # Only increment data_idx on im_start (not slice_start) so all slices
        # within one image share the same pad_value for per-image caching.
        pattern = MultiModalityDataPaddingPatternTokenPairs(
            media_token_pairs, data_start_token_ids=[im_start_id]
        )

        return pattern.pad_input_tokens(input_ids, image_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., image_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiniCPMV2_6`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., image_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiniCPMV2_6` 内部调用。

### Lines 1014-1014: Class `MiniCPMV4_0` overview
```python
class MiniCPMV4_0(MiniCPMBaseModel):
```
**EN:** Defines `MiniCPMV4_0` as a reusable runtime type derived from MiniCPMBaseModel. The class groups 7 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV4_0`，其继承关系为 MiniCPMBaseModel。这个类组织了 7 个方法，用于实现模型相关行为。

### Lines 1015-1052: Class `MiniCPMV4_0` attributes
```python
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
    # LoRA specific attributes
    supported_lora_modules = [
        # vision encoder
        "fc1",
        "fc2",
        "out_proj",
        # language model
        "qkv_proj",  # same name with vision encoder
        "o_proj",
        "gate_up_proj",
        "down_proj",
        # resampler
        "kv_proj",
    ]

    # BitandBytes specific attributes
    bitsandbytes_stacked_params_mapping = {
        # shard_name, weight_name, index
        "q_proj": ("qkv_proj", 0),
        "k_proj": ("qkv_proj", 1),
        "v_proj": ("qkv_proj", 2),
        "gate_proj": ("gate_up_proj", 0),
        "up_proj": ("gate_up_proj", 1),
    }

    embedding_modules = {}
    embedding_padding_modules = []
```
**EN:** Defines class-level attributes and metadata that shape how `MiniCPMV4_0` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiniCPMV4_0` 在运行时的行为。

### Lines 1054-1061: Method `MiniCPMV4_0.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__(config=config, quant_config=quant_config, prefix=prefix)
        assert self.version == (4, 0)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1063-1069: Method `MiniCPMV4_0.init_llm`
```python
    def init_llm(
        self,
        config: LlamaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        return LlamaForCausalLM(config=config, quant_config=quant_config, prefix=prefix)
```
**EN:** This method implements `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1071-1085: Method `MiniCPMV4_0.init_vision_module`
```python
    def init_vision_module(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig],
        prefix: str = "",
    ) -> nn.Module:
        model = Idefics2VisionTransformer(
            config=config.vision_config, quant_config=quant_config, prefix=prefix
        )
        if self.config.drop_vision_last_layer:
            model.encoder.layers = model.encoder.layers[:-1]

        setattr(model, "embed_dim", model.embeddings.embed_dim)
        setattr(model, "patch_size", model.embeddings.patch_size)
        return model
```
**EN:** This method implements `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1087-1105: Method `MiniCPMV4_0.init_resampler`
```python
    def init_resampler(
        self,
        embed_dim: int,
        vision_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        with set_default_torch_dtype(torch.float16):
            # The resampler in 2.6 remains consistent with the one in 2.5.
            resampler = Resampler2_5(
                num_queries=self.config.query_num,
                embed_dim=embed_dim,
                num_heads=embed_dim // 128,
                kv_dim=vision_dim,
                quant_config=quant_config,
                prefix=prefix,
            )

        return resampler.to(device="cuda", dtype=torch.get_default_dtype())
```
**EN:** This method implements `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1107-1118: Method `MiniCPMV4_0.get_vision_embedding`
```python
    def get_vision_embedding(
        self,
        pixel_values: List[torch.Tensor],
        patch_attn_mask: Optional[torch.Tensor] = None,
        tgt_sizes: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        vision_embedding = self.vpm(
            pixel_values,
            patch_attention_mask=patch_attn_mask,
            tgt_sizes=tgt_sizes,
        )
        return vision_embedding
```
**EN:** This method implements `get_vision_embedding(pixel_values: ..., patch_attn_mask: ...=..., tgt_sizes: ...=...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_vision_embedding(pixel_values: ..., patch_attn_mask: ...=..., tgt_sizes: ...=...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1120-1161: Method `MiniCPMV4_0.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        if items and items[0].format == MultimodalInputFormat.PRECOMPUTED_EMBEDDING:
            result = torch.cat([item.feature for item in items])
            return result.reshape(-1, result.shape[-1])

        # list of tensors
        pixel_values = flatten_nested_list([item.feature for item in items])
        tgt_sizes = torch.stack(
            flatten_nested_list([item.tgt_size for item in items]), dim=0
        )
        assert len(pixel_values) == tgt_sizes.shape[0]

        device = self.vpm.embeddings.position_embedding.weight.device
        dtype = self.vpm.embeddings.position_embedding.weight.dtype
        all_pixel_values_lst = [
            i.flatten(end_dim=1).permute(1, 0) for i in pixel_values
        ]

        max_patches = (tgt_sizes[:, 0] * tgt_sizes[:, 1]).max().item()
        assert isinstance(max_patches, int)
        all_pixel_values = torch.nn.utils.rnn.pad_sequence(
            all_pixel_values_lst, batch_first=True, padding_value=0.0
        )

        B, L, _ = all_pixel_values.shape
        all_pixel_values = all_pixel_values.permute(0, 2, 1).reshape(B, 3, -1, L)
        patch_attn_mask = torch.zeros(
            (B, 1, max_patches), dtype=torch.bool, device=device
        )

        tgt_sizes_tensor = tgt_sizes.clone().to(device=patch_attn_mask.device)
        mask_shapes = tgt_sizes_tensor[:, 0] * tgt_sizes_tensor[:, 1]
        patch_attn_mask[:, 0, :] = torch.arange(
            patch_attn_mask.size(2), device=patch_attn_mask.device
        ).unsqueeze(0) < mask_shapes.unsqueeze(1)

        vision_embedding = self.vpm(
            all_pixel_values.type(dtype),
            patch_attention_mask=patch_attn_mask,
            tgt_sizes=tgt_sizes,
# ... truncated for brevity ...
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1163-1177: Method `MiniCPMV4_0.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], image_inputs: MultimodalInputs):
        # Get all special token IDs
        im_start_id: int = image_inputs.im_start_id
        im_end_id: int = image_inputs.im_end_id
        slice_start_id: int = image_inputs.slice_start_id
        slice_end_id: int = image_inputs.slice_end_id

        media_token_pairs = [(im_start_id, im_end_id), (slice_start_id, slice_end_id)]
        # Only increment data_idx on im_start (not slice_start) so all slices
        # within one image share the same pad_value for per-image caching.
        pattern = MultiModalityDataPaddingPatternTokenPairs(
            media_token_pairs, data_start_token_ids=[im_start_id]
        )

        return pattern.pad_input_tokens(input_ids, image_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., image_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiniCPMV4_0`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., image_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiniCPMV4_0` 内部调用。

### Lines 1180-1180: Class `MiniCPMV4_5` overview
```python
class MiniCPMV4_5(MiniCPMBaseModel):
```
**EN:** Defines `MiniCPMV4_5` as a reusable runtime type derived from MiniCPMBaseModel. The class groups 8 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV4_5`，其继承关系为 MiniCPMBaseModel。这个类组织了 8 个方法，用于实现模型相关行为。

### Lines 1181-1218: Class `MiniCPMV4_5` attributes
```python
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
    # LoRA specific attributes
    supported_lora_modules = [
        # vision encoder
        "fc1",
        "fc2",
        "out_proj",
        # language model
        "qkv_proj",  # same name with vision encoder
        "o_proj",
        "gate_up_proj",
        "down_proj",
        # resampler
        "kv_proj",
    ]

    # BitandBytes specific attributes
    bitsandbytes_stacked_params_mapping = {
        # shard_name, weight_name, index
        "q_proj": ("qkv_proj", 0),
        "k_proj": ("qkv_proj", 1),
        "v_proj": ("qkv_proj", 2),
        "gate_proj": ("gate_up_proj", 0),
        "up_proj": ("gate_up_proj", 1),
    }

    embedding_modules = {}
    embedding_padding_modules = []
```
**EN:** Defines class-level attributes and metadata that shape how `MiniCPMV4_5` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiniCPMV4_5` 在运行时的行为。

### Lines 1220-1227: Method `MiniCPMV4_5.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__(config=config, quant_config=quant_config, prefix=prefix)
        assert self.version == (4, 5)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1229-1239: Method `MiniCPMV4_5.init_llm`
```python
    def init_llm(
        self,
        config: Qwen3Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        llm = Qwen3ForCausalLM(config=config, quant_config=quant_config, prefix=prefix)
        llm.get_input_embeddings = types.MethodType(
            lambda self: self.model.get_input_embeddings(), llm
        )
        return llm
```
**EN:** This method implements `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1241-1255: Method `MiniCPMV4_5.init_vision_module`
```python
    def init_vision_module(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig],
        prefix: str = "",
    ) -> nn.Module:
        model = Idefics2VisionTransformer(
            config=config.vision_config, quant_config=quant_config, prefix=prefix
        )
        if self.config.drop_vision_last_layer:
            model.encoder.layers = model.encoder.layers[:-1]

        setattr(model, "embed_dim", model.embeddings.embed_dim)
        setattr(model, "patch_size", model.embeddings.patch_size)
        return model
```
**EN:** This method implements `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1257-1275: Method `MiniCPMV4_5.init_resampler`
```python
    def init_resampler(
        self,
        embed_dim: int,
        vision_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        with set_default_torch_dtype(torch.float16):
            # The resampler in 2.6 remains consistent with the one in 2.5.
            resampler = Resampler4_5(
                num_queries=self.config.query_num,
                embed_dim=embed_dim,
                num_heads=embed_dim // 128,
                kv_dim=vision_dim,
                quant_config=quant_config,
                prefix=prefix,
            )

        return resampler.to(device="cuda", dtype=torch.get_default_dtype())
```
**EN:** This method implements `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1277-1288: Method `MiniCPMV4_5.get_vision_embedding`
```python
    def get_vision_embedding(
        self,
        pixel_values: List[torch.Tensor],
        patch_attn_mask: Optional[torch.Tensor] = None,
        tgt_sizes: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        vision_embedding = self.vpm(
            pixel_values,
            patch_attention_mask=patch_attn_mask,
            tgt_sizes=tgt_sizes,
        )
        return vision_embedding
```
**EN:** This method implements `get_vision_embedding(pixel_values: ..., patch_attn_mask: ...=..., tgt_sizes: ...=...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_vision_embedding(pixel_values: ..., patch_attn_mask: ...=..., tgt_sizes: ...=...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1290-1331: Method `MiniCPMV4_5.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        if items and items[0].format == MultimodalInputFormat.PRECOMPUTED_EMBEDDING:
            result = torch.cat([item.feature for item in items])
            return result.reshape(-1, result.shape[-1])

        # list of tensors
        pixel_values = flatten_nested_list([item.feature for item in items])
        tgt_sizes = torch.stack(
            flatten_nested_list([item.tgt_size for item in items]), dim=0
        )
        assert len(pixel_values) == tgt_sizes.shape[0]

        device = self.vpm.embeddings.position_embedding.weight.device
        dtype = self.vpm.embeddings.position_embedding.weight.dtype
        all_pixel_values_lst = [
            i.flatten(end_dim=1).permute(1, 0) for i in pixel_values
        ]

        max_patches = (tgt_sizes[:, 0] * tgt_sizes[:, 1]).max().item()
        assert isinstance(max_patches, int)
        all_pixel_values = torch.nn.utils.rnn.pad_sequence(
            all_pixel_values_lst, batch_first=True, padding_value=0.0
        )

        B, L, _ = all_pixel_values.shape
        all_pixel_values = all_pixel_values.permute(0, 2, 1).reshape(B, 3, -1, L)
        patch_attn_mask = torch.zeros(
            (B, 1, max_patches), dtype=torch.bool, device=device
        )

        tgt_sizes_tensor = tgt_sizes.clone().to(device=patch_attn_mask.device)
        mask_shapes = tgt_sizes_tensor[:, 0] * tgt_sizes_tensor[:, 1]
        patch_attn_mask[:, 0, :] = torch.arange(
            patch_attn_mask.size(2), device=patch_attn_mask.device
        ).unsqueeze(0) < mask_shapes.unsqueeze(1)

        vision_embedding = self.vpm(
            all_pixel_values.type(dtype),
            patch_attention_mask=patch_attn_mask,
            tgt_sizes=tgt_sizes,
# ... truncated for brevity ...
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1333-1347: Method `MiniCPMV4_5.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], image_inputs: MultimodalInputs):
        # Get all special token IDs
        im_start_id: int = image_inputs.im_start_id
        im_end_id: int = image_inputs.im_end_id
        slice_start_id: int = image_inputs.slice_start_id
        slice_end_id: int = image_inputs.slice_end_id

        media_token_pairs = [(im_start_id, im_end_id), (slice_start_id, slice_end_id)]
        # Only increment data_idx on im_start (not slice_start) so all slices
        # within one image share the same pad_value for per-image caching.
        pattern = MultiModalityDataPaddingPatternTokenPairs(
            media_token_pairs, data_start_token_ids=[im_start_id]
        )

        return pattern.pad_input_tokens(input_ids, image_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., image_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiniCPMV4_5`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., image_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiniCPMV4_5` 内部调用。

### Lines 1349-1351: Method `MiniCPMV4_5.eval`
```python
    def eval(self):
        super().eval()
        return self
```
**EN:** This method implements `eval()` and implements a focused helper that supports the surrounding runtime flow inside `MiniCPMV4_5`.
**CN:** 这个方法实现了 `eval()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiniCPMV4_5` 内部调用。

### Lines 1354-1366: Class `MiniCPMV4_6` overview
```python
class MiniCPMV4_6(MiniCPMBaseModel):
    """MiniCPM-V 4.6.

    Differences vs 4.5:
      * mid-ViT compression (``MiniCPMV_VisionTransformer`` fires a 2x2 window
        attention + 2x2 fold at ``config.insert_layer_id``);
      * post-encoder connector is a pure MLP chain (``MiniCPMV_Merger``),
        not a Perceiver resampler;
      * LLM backbone is Qwen3.5;
      * ``config.downsample_mode`` toggles ``"16x"`` (mid-ViT + post merger)
        vs ``"4x"`` (skip mid-ViT, keep 4x more visual tokens).
    """
```
**EN:** Defines `MiniCPMV4_6` as a reusable runtime type derived from MiniCPMBaseModel. The class groups 9 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV4_6`，其继承关系为 MiniCPMBaseModel。这个类组织了 9 个方法，用于实现模型相关行为。

### Lines 1367-1401: Class `MiniCPMV4_6` attributes
```python
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
    supported_lora_modules = [
        # vision encoder + mid-ViT merger
        "fc1",
        "fc2",
        "out_proj",
        "linear_1",
        "linear_2",
        # language model
        "qkv_proj",
        "o_proj",
        "gate_up_proj",
        "down_proj",
    ]

    bitsandbytes_stacked_params_mapping = {
        "q_proj": ("qkv_proj", 0),
        "k_proj": ("qkv_proj", 1),
        "v_proj": ("qkv_proj", 2),
        "gate_proj": ("gate_up_proj", 0),
        "up_proj": ("gate_up_proj", 1),
    }

    embedding_modules = {}
    embedding_padding_modules = []
```
**EN:** Defines class-level attributes and metadata that shape how `MiniCPMV4_6` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiniCPMV4_6` 在运行时的行为。

### Lines 1403-1426: Method `MiniCPMV4_6.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__(config=config, quant_config=quant_config, prefix=prefix)
        assert self.version == (4, 6)
        # ``Qwen3_5ForCausalLM`` returns plain hidden states (body only, no LM
        # head, no LogitsProcessor). Add them here so the downstream sampler
        # sees a ``LogitsProcessorOutput``. With ``tie_word_embeddings=True``
        # (4.6 default) the head shares weights with the embedding.
        text_config = config.text_config
        if getattr(text_config, "tie_word_embeddings", False):
            self.lm_head = self.llm.embed_tokens
        else:
            from sglang.srt.layers.vocab_parallel_embedding import ParallelLMHead

            self.lm_head = ParallelLMHead(
                text_config.vocab_size,
                text_config.hidden_size,
                quant_config=quant_config,
                prefix=add_prefix("lm_head", prefix),
            )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1428-1437: Method `MiniCPMV4_6.init_llm`
```python
    def init_llm(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        # 4.6 nests the LLM config under ``text_config``.
        return Qwen3_5ForCausalLM(
            config=config.text_config, quant_config=quant_config, prefix=prefix
        )
```
**EN:** This method implements `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1439-1457: Method `MiniCPMV4_6.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        **kwargs: Any,
    ) -> torch.Tensor:
        # Apply our lm_head + LogitsProcessor on top of the base routine; the
        # 4.6 LLM body (``Qwen3_5ForCausalLM``) returns plain hidden states,
        # unlike the ``Qwen3ForCausalLM`` 4.5 used.
        hidden_states = super().forward(
            input_ids=input_ids,
            positions=positions,
            forward_batch=forward_batch,
            **kwargs,
        )
        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch
        )
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 1459-1476: Method `MiniCPMV4_6.init_vision_module`
```python
    def init_vision_module(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig],
        prefix: str = "",
    ) -> nn.Module:
        model = MiniCPMV_VisionTransformer(
            config=config.vision_config, quant_config=quant_config, prefix=prefix
        )
        if getattr(self.config, "drop_vision_last_layer", False):
            # The mid-ViT merger sits on the transformer (not encoder.layers),
            # so popping the last encoder layer leaves it untouched — same
            # behaviour as 4.5.
            model.encoder.layers = model.encoder.layers[:-1]

        setattr(model, "embed_dim", model.embeddings.embed_dim)
        setattr(model, "patch_size", model.embeddings.patch_size)
        return model
```
**EN:** This method implements `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1478-1493: Method `MiniCPMV4_6.init_resampler`
```python
    def init_resampler(
        self,
        embed_dim: int,
        vision_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        # 4.6 replaces Resampler4_5 with a pure MLP. Method name kept so
        # ``MiniCPMBaseModel.__init__`` doesn't need to branch.
        with set_default_torch_dtype(torch.float16):
            merger = MiniCPMV_Merger(
                config=self.config,
                quant_config=quant_config,
                prefix=prefix,
            )
        return merger.to(device="cuda", dtype=torch.get_default_dtype())
```
**EN:** This method implements `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1495-1506: Method `MiniCPMV4_6.get_vision_embedding`
```python
    def get_vision_embedding(
        self,
        pixel_values: List[torch.Tensor],
        patch_attn_mask: Optional[torch.Tensor] = None,
        tgt_sizes: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        hidden, _ = self.vpm(
            pixel_values,
            patch_attention_mask=patch_attn_mask,
            target_sizes=tgt_sizes,
        )
        return hidden
```
**EN:** This method implements `get_vision_embedding(pixel_values: ..., patch_attn_mask: ...=..., tgt_sizes: ...=...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_vision_embedding(pixel_values: ..., patch_attn_mask: ...=..., tgt_sizes: ...=...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1508-1551: Method `MiniCPMV4_6.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        if items and items[0].format == MultimodalInputFormat.PRECOMPUTED_EMBEDDING:
            result = torch.cat([item.feature for item in items])
            return result.reshape(-1, result.shape[-1])

        pixel_values = flatten_nested_list([item.feature for item in items])
        tgt_sizes = torch.stack(
            flatten_nested_list([item.tgt_size for item in items]), dim=0
        )
        assert len(pixel_values) == tgt_sizes.shape[0]

        device = self.vpm.embeddings.position_embedding.weight.device
        dtype = self.vpm.embeddings.position_embedding.weight.dtype
        all_pixel_values_lst = [
            i.flatten(end_dim=1).permute(1, 0) for i in pixel_values
        ]

        max_patches = (tgt_sizes[:, 0] * tgt_sizes[:, 1]).max().item()
        assert isinstance(max_patches, int)
        all_pixel_values = torch.nn.utils.rnn.pad_sequence(
            all_pixel_values_lst, batch_first=True, padding_value=0.0
        )

        B, L, _ = all_pixel_values.shape
        all_pixel_values = all_pixel_values.permute(0, 2, 1).reshape(B, 3, -1, L)
        patch_attn_mask = torch.zeros(
            (B, 1, max_patches), dtype=torch.bool, device=device
        )

        tgt_sizes_tensor = tgt_sizes.clone().to(device=patch_attn_mask.device)
        mask_shapes = tgt_sizes_tensor[:, 0] * tgt_sizes_tensor[:, 1]
        patch_attn_mask[:, 0, :] = torch.arange(
            patch_attn_mask.size(2), device=patch_attn_mask.device
        ).unsqueeze(0) < mask_shapes.unsqueeze(1)

        use_vit_merger = getattr(self.config, "downsample_mode", "16x") != "4x"

        vision_embedding, tgt_sizes_out = self.vpm(
            all_pixel_values.type(dtype),
            patch_attention_mask=patch_attn_mask,
# ... truncated for brevity ...
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1556-1556: Class `MiniCPMV4_6` attributes
```python
    get_video_feature = get_image_feature
```
**EN:** Defines class-level attributes and metadata that shape how `MiniCPMV4_6` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiniCPMV4_6` 在运行时的行为。

### Lines 1558-1568: Method `MiniCPMV4_6.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], image_inputs: MultimodalInputs):
        im_start_id: int = image_inputs.im_start_id
        im_end_id: int = image_inputs.im_end_id
        slice_start_id: int = image_inputs.slice_start_id
        slice_end_id: int = image_inputs.slice_end_id

        media_token_pairs = [(im_start_id, im_end_id), (slice_start_id, slice_end_id)]
        pattern = MultiModalityDataPaddingPatternTokenPairs(
            media_token_pairs, data_start_token_ids=[im_start_id]
        )
        return pattern.pad_input_tokens(input_ids, image_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., image_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiniCPMV4_6`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., image_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiniCPMV4_6` 内部调用。

### Lines 1570-1616: Method `MiniCPMV4_6.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        """Remap 4.6 prefixes (``model.{vision_tower,merger,language_model}``)
        to sglang's (``vpm`` / ``resampler`` / ``llm``) and delegate the LLM
        portion to ``Qwen3_5ForCausalLM.load_weights`` — the Qwen3.5 hybrid
        backbone has its own stacked-param logic (``in_proj_a/b -> in_proj_ba``,
        ``in_proj_qkv/z -> in_proj_qkvz``) the legacy loader doesn't know.
        Vision-side still needs QKV stacking + ``out_proj -> proj`` rename.
        """

        llm_weights: List[Tuple[str, torch.Tensor]] = []
        vision_weights: List[Tuple[str, torch.Tensor]] = []
        for name, w in weights:
            if name.startswith("model.language_model."):
                llm_weights.append((name[len("model.language_model.") :], w))
                continue
            if name.startswith("model.vision_tower."):
                name = "vpm." + name[len("model.vision_tower.") :]
            elif name.startswith("model.merger."):
                name = "resampler." + name[len("model.merger.") :]
            vision_weights.append((name, w))

        self.llm.load_weights(iter(llm_weights))

        stacked_params_mapping = [
            ("self_attn.qkv_proj", "self_attn.q_proj", "q"),
            ("self_attn.qkv_proj", "self_attn.k_proj", "k"),
            ("self_attn.qkv_proj", "self_attn.v_proj", "v"),
        ]
        params_dict = dict(self.named_parameters())
        for name, loaded_weight in vision_weights:
            name = name.replace("self_attn.out_proj", "self_attn.proj")

            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                target = name.replace(weight_name, param_name)
                if target not in params_dict:
                    continue
                param = params_dict[target]
                param.weight_loader(param, loaded_weight, shard_id)
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and Remap 4.6 prefixes (``model.{vision_tower,merger,language_model}``).
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1619-1624: Top-level assign
```python
_SUPPORT_VERSION = {
    (2, 6): MiniCPMV2_6,
    (4, 0): MiniCPMV4_0,
    (4, 5): MiniCPMV4_5,
    (4, 6): MiniCPMV4_6,
}
```
**EN:** Defines or updates _SUPPORT_VERSION, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _SUPPORT_VERSION，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 1627-1635: Class `MiniCPMV` overview
```python
class MiniCPMV:
    """
    Different versions of MiniCPMV use different visual encoders and LLMs,
    which is not conducive to the current integration logic of LoRA and
    bitsandbytes in SGLang. Therefore, it is necessary to separate them.
    """

    # Ensure that the LoRA support check passes when the class is not
    # initialized, but set all these attributes to empty.
```
**EN:** Defines `MiniCPMV` as a reusable runtime type derived from no explicit base class. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV`，其继承关系为 no explicit base class。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 1636-1641: Class `MiniCPMV` attributes
```python
    packed_modules_mapping = {}
    supported_lora_modules = []
    embedding_modules = {}
    embedding_padding_modules = []

    minicpmv: nn.Module
```
**EN:** Defines class-level attributes and metadata that shape how `MiniCPMV` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiniCPMV` 在运行时的行为。

### Lines 1643-1680: Method `MiniCPMV.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        # 4.6 carries ``model_type == "minicpmv4_6"`` instead of a numeric
        # ``config.version``; older versionless configs keep the legacy
        # ``(2, 6)`` default.
        if getattr(config, "model_type", None) == "minicpmv4_6":
            version = (4, 6)
        elif not hasattr(config, "version"):
            version = (2, 6)
        else:
            version = str(config.version).split(".")
            version = tuple([int(x) for x in version])
        # Dispatch class based on version
        instance_class = _SUPPORT_VERSION.get(version)
        if instance_class is None:
            supported_versions = ", ".join(
                [f"{v[0]}.{v[1]}" for v in sorted(_SUPPORT_VERSION.keys())]
            )
            raise ValueError(
                f"Currently, MiniCPMV only supports versions "
                f"{supported_versions}. Got version: {version}"
            )

        try:
            minicpmv = instance_class(
                config=config, quant_config=quant_config, prefix=prefix
            )
            self.minicpmv = minicpmv
        except Exception as e:
            print(f"Failed to instantiate MiniCPMV: {e}")
            raise e
        self.config = config
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1682-1685: Method `MiniCPMV.__getattr__`
```python
    def __getattr__(self, name):
        if name == "minicpmv":
            return None
        return getattr(self.minicpmv, name)
```
**EN:** This method implements `__getattr__(name)` and implements a focused helper that supports the surrounding runtime flow inside `MiniCPMV`.
**CN:** 这个方法实现了 `__getattr__(name)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiniCPMV` 内部调用。

### Lines 1687-1688: Method `MiniCPMV.__call__`
```python
    def __call__(self, *args, **kwargs):
        return self.minicpmv(*args, **kwargs)
```
**EN:** This method implements `__call__(*args, **kwargs)` and implements a focused helper that supports the surrounding runtime flow inside `MiniCPMV`.
**CN:** 这个方法实现了 `__call__(*args, **kwargs)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiniCPMV` 内部调用。

### Lines 1690-1746: Method `MiniCPMV.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        # Defer to the version-specific subclass loader if it overrides the
        # base (4.6 does — it needs prefix remap + Qwen3.5 LLM delegation).
        sub_loader = getattr(type(self.minicpmv), "load_weights", None)
        base_loader = getattr(MiniCPMBaseModel, "load_weights", None)
        if sub_loader is not None and sub_loader is not base_loader:
            return self.minicpmv.load_weights(weights)

        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        params_dict = dict(self.minicpmv.named_parameters())
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq~" in name or "projector" in name:
                continue
            if "rotary_emb.cos_cached" in name or "rotary_emb.sin_cached" in name:
                # Models trained using ColossalAI may include these tensors in
                # the checkpoint. Skip them.
                continue
            if name.startswith("model.vision_tower") and name not in params_dict:
                continue

            # adapt to VisionAttention
            name = name.replace(r"self_attn.out_proj", r"self_attn.proj")

            if "sampler" in name:
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
                continue

            for param_name, weight_name, shard_id in stacked_params_mapping:
                # replace the name and load with customized loader
                if weight_name not in name:
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1752-1752: Class `MiniCPMV4_6ForConditionalGeneration` overview
```python
class MiniCPMV4_6ForConditionalGeneration(MiniCPMV):
```
**EN:** Defines `MiniCPMV4_6ForConditionalGeneration` as a reusable runtime type derived from MiniCPMV. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV4_6ForConditionalGeneration`，其继承关系为 MiniCPMV。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1753-1753: Class `MiniCPMV4_6ForConditionalGeneration` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1756-1756: Top-level assign
```python
EntryClass = [MiniCPMV, MiniCPMV4_6ForConditionalGeneration]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `types`
- `functools.partial`
- `itertools.chain`
- `typing.Any`
- `typing.Callable`
- `typing.Iterable`
- `typing.List`
- `typing.Literal`
- `typing.Optional`
- `typing.Tuple`
- `typing.TypedDict`
- `typing.Union`
- `numpy`
- `torch`
- `torch.types`
- `PIL.Image`
- `torch.nn`
- `torch.nn.init.trunc_normal_`
- `transformers.PretrainedConfig`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternTokenPairs`
- `sglang.srt.managers.mm_utils.general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputFormat`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.utils.set_default_torch_dtype`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.idefics2.Idefics2VisionTransformer`
- `sglang.srt.models.llama.LlamaConfig`
- `sglang.srt.models.llama.LlamaForCausalLM`
- `sglang.srt.models.minicpmv_vit.MiniCPMV_Merger`
- `sglang.srt.models.minicpmv_vit.MiniCPMV_VisionTransformer`
- `sglang.srt.models.qwen2.Qwen2Config`
- `sglang.srt.models.qwen2.Qwen2ForCausalLM`
- `sglang.srt.models.qwen3.Qwen3Config`
- `sglang.srt.models.qwen3.Qwen3ForCausalLM`
- `sglang.srt.models.qwen3_5.Qwen3_5ForCausalLM`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.flatten_nested_list`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
