# radio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/radio.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the radio model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 radio 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 18-37: Module imports
```python
import logging
import math
from collections.abc import Iterable
from itertools import repeat
from typing import TypeAlias

import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange
from transformers import PretrainedConfig
from transformers.modeling_outputs import BaseModelOutput

from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.model_loader.weight_utils import (
    default_weight_loader,
    replace_prefix,
    replace_substrings,
)
from sglang.srt.models.internvl import InternVisionEncoder
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 39-39: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 41-41: Top-level annassign
```python
input_dim_t: TypeAlias = int | tuple[int, int]
```
**EN:** Defines or updates input_dim_t, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 input_dim_t，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 42-42: Top-level annassign
```python
norm_t: TypeAlias = tuple[float, float, float] | torch.Tensor
```
**EN:** Defines or updates norm_t, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 norm_t，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 45-51: Function `_ntuple`
```python
def _ntuple(n):
    def parse(x):
        if isinstance(x, Iterable) and not isinstance(x, str):
            return tuple(x)
        return tuple(repeat(x, n))

    return parse
```
**EN:** This function implements `_ntuple(n)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `_ntuple(n)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 54-54: Top-level assign
```python
to_1tuple = _ntuple(1)
```
**EN:** Defines or updates to_1tuple, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 to_1tuple，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 55-55: Top-level assign
```python
to_2tuple = _ntuple(2)
```
**EN:** Defines or updates to_2tuple, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 to_2tuple，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 56-56: Top-level assign
```python
to_3tuple = _ntuple(3)
```
**EN:** Defines or updates to_3tuple, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 to_3tuple，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 57-57: Top-level assign
```python
to_4tuple = _ntuple(4)
```
**EN:** Defines or updates to_4tuple, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 to_4tuple，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 58-58: Top-level assign
```python
to_ntuple = _ntuple
```
**EN:** Defines or updates to_ntuple, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 to_ntuple，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 61-61: Class `ClsToken` overview
```python
class ClsToken(nn.Module):
```
**EN:** Defines `ClsToken` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ClsToken`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 62-92: Method `ClsToken.__init__`
```python
    def __init__(
        self,
        ndim: int,
        num_tokens: int = 1,
        enabled: bool = True,
        register_multiple: int | None = None,
        num_registers: int | None = None,
    ):
        super().__init__()

        self.ndim = ndim
        self.enabled = enabled
        self.num_registers = 0
        self.num_tokens = num_tokens
        if enabled:
            if num_registers:
                self.num_registers = num_registers
            elif register_multiple:
                self.num_registers = register_multiple - (
                    num_tokens % register_multiple
                )

            scale = ndim**-0.5
            self.token = nn.Parameter(
                torch.randn(num_tokens + self.num_registers, ndim) * scale
            )

        else:
            self.token = None

        self.num_patches = self.num_tokens + self.num_registers
```
**EN:** This method implements `__init__(ndim: ..., num_tokens: ...=..., enabled: ...=..., register_multiple: ...=..., num_registers: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(ndim: ..., num_tokens: ...=..., enabled: ...=..., register_multiple: ...=..., num_registers: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 94-107: Method `ClsToken.forward`
```python
    def forward(self, x: torch.Tensor):
        if self.token is None:
            return x

        token = self.token.unsqueeze(0).expand(x.shape[0], -1, -1)
        x = torch.cat(
            [
                token,
                x,
            ],
            dim=1,
        )

        return x
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 110-110: Class `ViTPatchGenerator` overview
```python
class ViTPatchGenerator(nn.Module):
```
**EN:** Defines `ViTPatchGenerator` as a reusable runtime type derived from nn.Module. The class groups 14 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ViTPatchGenerator`，其继承关系为 nn.Module。这个类组织了 14 个方法，用于实现模型相关行为。

### Lines 111-192: Method `ViTPatchGenerator.__init__`
```python
    def __init__(
        self,
        patch_size: int,
        embed_dim: int,
        input_dims: input_dim_t,
        abs_pos: bool = True,
        normalize_patches: bool = False,
        cls_token: bool = False,
        max_input_dims: input_dim_t | None = None,
        pos_dropout: float = 0.0,
        return_pos_enc: bool = False,
        num_cls_tokens: int = 1,
        register_multiple: int | None = None,
        num_registers: int | None = None,
        patch_bias: bool = False,
        video_temporal_patch_size: int = 1,
        separate_video_embedder: bool = True,
        device=None,
        dtype=None,
    ):
        super().__init__()
        if isinstance(input_dims, int):
            input_dims = (input_dims, input_dims)

        if max_input_dims is None:
            max_input_dims = input_dims
        if isinstance(max_input_dims, int):
            max_input_dims = (max_input_dims, max_input_dims)

        max_input_dims = tuple(
            int(math.ceil(d / patch_size) * patch_size) for d in max_input_dims
        )

        self.cpe_mode = max_input_dims != input_dims
        self.pos_dropout = pos_dropout
        self.return_pos_enc = return_pos_enc

        factory = dict(device=device, dtype=dtype)

        self.patch_size = patch_size
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(patch_size: ..., embed_dim: ..., input_dims: ..., abs_pos: ...=..., normalize_patches: ...=..., cls_token: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(patch_size: ..., embed_dim: ..., input_dims: ..., abs_pos: ...=..., normalize_patches: ...=..., cls_token: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 194-201: Method `ViTPatchGenerator.forward`
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        patches = self.embed_patches(x)
        patches, pos_enc = self.apply_pos_enc(patches, input_size=x.shape[2:])
        patches = self.cls_token(patches)
        patches = self.patch_normalizer(patches)
        if self.return_pos_enc:
            return patches, pos_enc
        return patches
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 203-235: Method `ViTPatchGenerator.forward_video`
```python
    def forward_video(self, x: torch.Tensor, temporal_patch_size: int) -> torch.Tensor:
        """Embed video frames with temporal compression via tubelet grouping."""
        assert (
            self.video_embedder is not None
        ), "video_embedder is required for temporal compression"
        T = temporal_patch_size
        num_frames = x.shape[0]

        if num_frames % T != 0:
            pad = T - (num_frames % T)
            x = torch.cat(
                [x, x[-1:].expand(pad, -1, -1, -1)],
                dim=0,
            )

        padded_frames = x.shape[0]
        num_tubelets = padded_frames // T

        patches = self.im_to_patches(x)
        num_spatial = patches.shape[1]
        feat_dim = patches.shape[2]

        patches = patches.reshape(num_tubelets, T, num_spatial, feat_dim)
        patches = patches.permute(0, 2, 1, 3).reshape(
            num_tubelets, num_spatial, T * feat_dim
        )

        patches = self.video_embedder(patches)

        patches, _ = self.apply_pos_enc(patches, input_size=x.shape[2:])
        patches = self.cls_token(patches)
        patches = self.patch_normalizer(patches)
        return patches
```
**EN:** This method implements `forward_video(x: ..., temporal_patch_size: ...)` and Embed video frames with temporal compression via tubelet grouping.
**CN:** 这个方法实现了 `forward_video(x: ..., temporal_patch_size: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 237-239: Method `ViTPatchGenerator.apply_cls_token`
```python
    @property
    def apply_cls_token(self):
        return self.cls_token.enabled
```
**EN:** This method implements `apply_cls_token()` and implements a focused helper that supports the surrounding runtime flow inside `ViTPatchGenerator` Decorators: property.
**CN:** 这个方法实现了 `apply_cls_token()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ViTPatchGenerator` 内部调用 装饰器：property。

### Lines 241-243: Method `ViTPatchGenerator.num_cls_tokens`
```python
    @property
    def num_cls_tokens(self):
        return self.cls_token.num_tokens
```
**EN:** This method implements `num_cls_tokens()` and implements a focused helper that supports the surrounding runtime flow inside `ViTPatchGenerator` Decorators: property.
**CN:** 这个方法实现了 `num_cls_tokens()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ViTPatchGenerator` 内部调用 装饰器：property。

### Lines 245-247: Method `ViTPatchGenerator.num_cls_patches`
```python
    @property
    def num_cls_patches(self):
        return self.cls_token.num_patches
```
**EN:** This method implements `num_cls_patches()` and implements a focused helper that supports the surrounding runtime flow inside `ViTPatchGenerator` Decorators: property.
**CN:** 这个方法实现了 `num_cls_patches()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ViTPatchGenerator` 内部调用 装饰器：property。

### Lines 249-251: Method `ViTPatchGenerator.num_registers`
```python
    @property
    def num_registers(self):
        return self.cls_token.num_registers
```
**EN:** This method implements `num_registers()` and registers metadata or implementation classes into a global lookup table Decorators: property.
**CN:** 这个方法实现了 `num_registers()`，其作用是把元数据或实现类注册到全局查找表中 装饰器：property。

### Lines 253-255: Method `ViTPatchGenerator.num_skip`
```python
    @property
    def num_skip(self):
        return self.num_cls_tokens + self.num_registers
```
**EN:** This method implements `num_skip()` and implements a focused helper that supports the surrounding runtime flow inside `ViTPatchGenerator` Decorators: property.
**CN:** 这个方法实现了 `num_skip()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ViTPatchGenerator` 内部调用 装饰器：property。

### Lines 257-276: Method `ViTPatchGenerator._load_embed`
```python
    def _load_embed(self, src_embed: torch.Tensor, targ_embed: nn.Parameter):
        if src_embed.shape != targ_embed.shape:
            src_size = int(math.sqrt(src_embed.shape[1]))

            assert (
                src_size**2 == src_embed.shape[1]
            ), "Unable to interpolate non-square embedding"

            src_embed = rearrange(
                src_embed, "b (h w) c -> b c h w", h=src_size, w=src_size
            )
            src_embed = F.interpolate(
                src_embed,
                size=(self.num_rows, self.num_cols),
                mode="bicubic",
                align_corners=True,
                antialias=False,
            )
            src_embed = rearrange(src_embed, "b c h w -> b (h w) c")
        targ_embed.data.copy_(src_embed)
```
**EN:** This method implements `_load_embed(src_embed: ..., targ_embed: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `_load_embed(src_embed: ..., targ_embed: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 278-303: Method `ViTPatchGenerator._load_projection`
```python
    def _load_projection(
        self, src_proj_weight: torch.Tensor, targ_proj_weight: torch.Tensor
    ):
        if src_proj_weight.shape != targ_proj_weight.shape:
            src_patch_size = int(math.sqrt(src_proj_weight.shape[1] // 3))

            assert (src_patch_size**2) * 3 == src_proj_weight.shape[
                1
            ], "Unable to interpolate non-square patch size"

            src_proj_weight = rearrange(
                src_proj_weight,
                "b (c h w) -> b c h w",
                c=3,
                h=src_patch_size,
                w=src_patch_size,
            )
            src_proj_weight = F.interpolate(
                src_proj_weight,
                size=(self.patch_size, self.patch_size),
                mode="bicubic",
                align_corners=True,
                antialias=False,
            )
            src_proj_weight = rearrange(src_proj_weight, "b c h w -> b (c h w)")
        targ_proj_weight.data.copy_(src_proj_weight)
```
**EN:** This method implements `_load_projection(src_proj_weight: ..., targ_proj_weight: ...)` and implements a focused helper that supports the surrounding runtime flow inside `ViTPatchGenerator`.
**CN:** 这个方法实现了 `_load_projection(src_proj_weight: ..., targ_proj_weight: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ViTPatchGenerator` 内部调用。

### Lines 305-308: Method `ViTPatchGenerator.embed_patches`
```python
    def embed_patches(self, x: torch.Tensor) -> torch.Tensor:
        patches = self.im_to_patches(x)
        patches = self.embedder(patches)
        return patches
```
**EN:** This method implements `embed_patches(x: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `embed_patches(x: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 310-332: Method `ViTPatchGenerator.apply_pos_enc`
```python
    def apply_pos_enc(
        self,
        patches: torch.Tensor,
        patch_idxs: torch.Tensor | None = None,
        input_size: tuple[int, int] | None = None,
    ) -> torch.Tensor:
        if not self.abs_pos:
            return patches

        pos_enc = self.get_pos_enc(patches.shape[0], patch_idxs, input_size)

        if self.training and self.pos_dropout > 0:
            keeps = (
                torch.rand(
                    patches.shape[0], 1, 1, dtype=pos_enc.dtype, device=pos_enc.device
                )
                > self.pos_dropout
            )
            pos_enc_drop = torch.where(keeps, pos_enc, 0)
        else:
            pos_enc_drop = pos_enc

        return patches + pos_enc_drop, pos_enc
```
**EN:** This method implements `apply_pos_enc(patches: ..., patch_idxs: ...=..., input_size: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `ViTPatchGenerator`.
**CN:** 这个方法实现了 `apply_pos_enc(patches: ..., patch_idxs: ...=..., input_size: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ViTPatchGenerator` 内部调用。

### Lines 334-355: Method `ViTPatchGenerator.get_pos_enc`
```python
    def get_pos_enc(
        self,
        batch_size: int,
        patch_idxs: torch.Tensor | None = None,
        input_size: tuple[int, int] | None = None,
    ) -> torch.Tensor:
        if input_size is None:
            input_dims = self.input_dims
        else:
            input_dims = tuple(d // self.patch_size for d in input_size)

        pos_embed = self._get_pos_embeddings(batch_size, input_dims)

        if patch_idxs is None:
            return pos_embed

        exp_patch_idxs = patch_idxs.unsqueeze(-1).expand(-1, -1, pos_embed.shape[-1])

        pos_embed = torch.gather(
            pos_embed.expand(patch_idxs.shape[0], -1, -1), dim=1, index=exp_patch_idxs
        )
        return pos_embed
```
**EN:** This method implements `get_pos_enc(batch_size: ..., patch_idxs: ...=..., input_size: ...=...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_pos_enc(batch_size: ..., patch_idxs: ...=..., input_size: ...=...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 357-392: Method `ViTPatchGenerator._get_pos_embeddings`
```python
    def _get_pos_embeddings(self, batch_size: int, input_dims: tuple[int, int]):
        if (self.num_rows, self.num_cols) == input_dims:
            return self.pos_embed

        pos_embed = self.pos_embed.reshape(1, self.num_rows, self.num_cols, -1).permute(
            0, 3, 1, 2
        )

        def window_select(pos_embed):
            if input_dims[0] < pos_embed.shape[-2]:
                pos_embed = pos_embed[..., : input_dims[0], :]
            if input_dims[1] < pos_embed.shape[-1]:
                pos_embed = pos_embed[..., :, : input_dims[1]]
            return pos_embed

        if self.cpe_mode:
            max_dim = max(input_dims)
            pos_embed = F.interpolate(
                pos_embed.float(),
                size=(max_dim, max_dim),
                align_corners=False,
                mode="bilinear",
            ).to(pos_embed.dtype)

            pos_embed = window_select(pos_embed)
        else:
            pos_embed = window_select(pos_embed)

        if pos_embed.shape[-2:] != input_dims:
            pos_embed = F.interpolate(
                pos_embed.float(), size=input_dims, align_corners=False, mode="bilinear"
            ).to(pos_embed.dtype)

        pos_embed = pos_embed.flatten(2).permute(0, 2, 1)

        return pos_embed
```
**EN:** This method implements `_get_pos_embeddings(batch_size: ..., input_dims: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `_get_pos_embeddings(batch_size: ..., input_dims: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 395-395: Class `Im2Patches` overview
```python
class Im2Patches(nn.Module):
```
**EN:** Defines `Im2Patches` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Im2Patches`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 396-398: Method `Im2Patches.__init__`
```python
    def __init__(self, patch_size: int):
        super().__init__()
        self.patch_size = patch_size
```
**EN:** This method implements `__init__(patch_size: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(patch_size: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 400-416: Method `Im2Patches.forward`
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        if self.patch_size == 1:
            patches = x.flatten(2)
            patches = patches.permute(0, 2, 1)
            return patches

        py = x.shape[-2] // self.patch_size
        px = x.shape[-1] // self.patch_size
        patches = rearrange(
            x,
            "b c (py yy) (px xx) -> b (py px) (c yy xx)",
            py=py,
            yy=self.patch_size,
            px=px,
            xx=self.patch_size,
        )
        return patches
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 419-419: Class `ViTPatchLinear` overview
```python
class ViTPatchLinear(nn.Linear):
```
**EN:** Defines `ViTPatchLinear` as a reusable runtime type derived from nn.Linear. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ViTPatchLinear`，其继承关系为 nn.Linear。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 420-422: Method `ViTPatchLinear.__init__`
```python
    def __init__(self, patch_size: int, embed_dim: int, bias: bool = False, **factory):
        super().__init__(3 * (patch_size**2), embed_dim, bias=bias, **factory)
        self.patch_size = patch_size
```
**EN:** This method implements `__init__(patch_size: ..., embed_dim: ..., bias: ...=..., **factory)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(patch_size: ..., embed_dim: ..., bias: ...=..., **factory)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 425-425: Class `RadioInternVisionModel` overview
```python
class RadioInternVisionModel(nn.Module):
```
**EN:** Defines `RadioInternVisionModel` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `RadioInternVisionModel`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 426-428: Class `RadioInternVisionModel` attributes
```python
    packed_modules_mapping = {
        "qkv": ["qkv"],
    }
```
**EN:** Defines class-level attributes and metadata that shape how `RadioInternVisionModel` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `RadioInternVisionModel` 在运行时的行为。

### Lines 430-458: Method `RadioInternVisionModel.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig = None,
        quant_config: QuantizationConfig | None = None,
    ) -> None:
        super().__init__()

        self.config = config
        self.img_size, self.grid_size, self.num_patches = self._init_img_size(
            to_2tuple(config.patch_size), config.image_size
        )
        max_img_size = int(
            round(config.max_img_size / config.patch_size) * config.patch_size
        )
        video_temporal_patch_size = getattr(config, "video_temporal_patch_size", 1)
        separate_video_embedder = getattr(config, "separate_video_embedder", True)

        self.patch_generator = ViTPatchGenerator(
            config.patch_size,
            config.hidden_size,
            input_dims=self.img_size,
            max_input_dims=max_img_size,
            cls_token=True,
            register_multiple=config.reg_tokens,
            video_temporal_patch_size=video_temporal_patch_size,
            separate_video_embedder=separate_video_embedder,
        )

        self.encoder = InternVisionEncoder(config=config, quant_config=quant_config)
```
**EN:** This method implements `__init__(config: ...=..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...=..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 460-466: Method `RadioInternVisionModel._init_img_size`
```python
    def _init_img_size(self, patch_size, img_size: int | tuple[int, int]):
        if img_size is None:
            return None, None, None
        img_size = to_2tuple(img_size)
        grid_size = tuple([s // p for s, p in zip(img_size, patch_size)])
        num_patches = grid_size[0] * grid_size[1]
        return img_size, grid_size, num_patches
```
**EN:** This method implements `_init_img_size(patch_size, img_size: ...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `_init_img_size(patch_size, img_size: ...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 468-469: Method `RadioInternVisionModel.get_input_embeddings`
```python
    def get_input_embeddings(self):
        return self.embeddings
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 471-476: Method `RadioInternVisionModel.forward`
```python
    def forward(self, x: torch.Tensor) -> torch.FloatTensor:
        assert self.patch_generator is not None
        hidden_states = self.patch_generator(x)
        encoder_outputs = self.encoder.forward(inputs_embeds=hidden_states)
        assert isinstance(encoder_outputs, BaseModelOutput)
        return encoder_outputs.last_hidden_state
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 479-479: Class `RadioModel` overview
```python
class RadioModel(nn.Module):
```
**EN:** Defines `RadioModel` as a reusable runtime type derived from nn.Module. The class groups 6 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `RadioModel`，其继承关系为 nn.Module。这个类组织了 6 个方法，用于实现模型相关行为。

### Lines 480-482: Class `RadioModel` attributes
```python
    packed_modules_mapping = {
        "qkv": ["qkv"],
    }
```
**EN:** Defines class-level attributes and metadata that shape how `RadioModel` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `RadioModel` 在运行时的行为。

### Lines 484-495: Method `RadioModel.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
    ) -> None:
        super().__init__()

        self.config = config
        self.model = RadioInternVisionModel(
            config=config,
            quant_config=quant_config,
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 497-510: Method `RadioModel.forward`
```python
    def forward(
        self,
        pixel_values: torch.Tensor | list[torch.Tensor] | None = None,
        num_frames: int | None = None,
    ) -> torch.FloatTensor:
        if (
            num_frames is not None
            and getattr(self.config, "video_temporal_patch_size", 1) > 1
        ):
            return self._forward_video_temporal(pixel_values, num_frames)
        if isinstance(pixel_values, list):
            return self._forward_dynamic(pixel_values)
        y = self.model(pixel_values)
        return self._extract_final(y)
```
**EN:** This method implements `forward(pixel_values: ...=..., num_frames: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(pixel_values: ...=..., num_frames: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 512-544: Method `RadioModel._forward_dynamic`
```python
    def _forward_dynamic(
        self, images: list[torch.Tensor]
    ) -> tuple[torch.Tensor, list[int]]:
        """Process variable-size images with ragged packing via cu_seqlens."""
        patch_gen = self.model.patch_generator
        all_patches = []
        seqlens = [0]

        for img in images:
            patches = patch_gen(img)
            seq_len = patches.shape[1]
            all_patches.append(patches.squeeze(0))
            seqlens.append(seqlens[-1] + seq_len)

        hidden = torch.cat(all_patches, dim=0).unsqueeze(0)
        cu_seqlens = torch.tensor(seqlens, dtype=torch.int32, device=hidden.device)

        out = self.model.encoder.forward(inputs_embeds=hidden, cu_seqlens=cu_seqlens)
        features = out.last_hidden_state

        num_skip = patch_gen.num_skip
        per_image_features = []
        num_patches_list = []
        for i in range(len(images)):
            start = seqlens[i] + num_skip
            end = seqlens[i + 1]
            per_image_features.append(features[0, start:end])
            num_patches_list.append(end - start)

        return (
            torch.cat(per_image_features, dim=0).unsqueeze(0),
            num_patches_list,
        )
```
**EN:** This method implements `_forward_dynamic(images: ...)` and Process variable-size images with ragged packing via cu_seqlens.
**CN:** 这个方法实现了 `_forward_dynamic(images: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 546-570: Method `RadioModel._forward_video_temporal`
```python
    def _forward_video_temporal(
        self, pixel_values: torch.Tensor, num_frames: int
    ) -> torch.Tensor:
        """Process video frames with temporal compression (tubelet grouping)."""
        T = self.config.video_temporal_patch_size
        patch_gen = self.model.patch_generator

        patches = patch_gen.forward_video(pixel_values, T)
        num_tubelets = patches.shape[0]
        seq_per_tubelet = patches.shape[1]

        cu_seqlens = torch.arange(
            0,
            (num_tubelets + 1) * seq_per_tubelet,
            seq_per_tubelet,
            dtype=torch.int32,
            device=patches.device,
        )
        packed = patches.reshape(1, -1, patches.shape[-1])

        out = self.model.encoder.forward(inputs_embeds=packed, cu_seqlens=cu_seqlens)
        features = out.last_hidden_state.reshape(num_tubelets, seq_per_tubelet, -1)

        num_skip = patch_gen.num_skip
        return features[:, num_skip:]
```
**EN:** This method implements `_forward_video_temporal(pixel_values: ..., num_frames: ...)` and Process video frames with temporal compression (tubelet grouping).
**CN:** 这个方法实现了 `_forward_video_temporal(pixel_values: ..., num_frames: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 572-604: Method `RadioModel.load_weights`
```python
    def load_weights(self, weights) -> set[str]:
        remap_substrings = {
            "attn": "attn.attn",
            "qkv": "qkv_proj",
            "blocks": "encoder.layers",
        }
        remap_prefixes = {
            "radio_model.": "",
        }

        loaded_params: set[str] = set()
        params_dict = dict(self.named_parameters())

        if isinstance(weights, dict):
            weights_list = list(weights.items())
        else:
            weights_list = list(weights)

        for name, weight in weights_list:
            if not name.startswith("radio_model."):
                # Skip non-radio weights
                continue
            name = replace_substrings(name, remap_substrings)
            name = replace_prefix(name, remap_prefixes)
            if name and name in params_dict:
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, weight)
                loaded_params.add(name)
                if "video_embedder" in name:
                    self.model.patch_generator._video_embedder_loaded = True

        return loaded_params
```
**EN:** This method implements `load_weights(weights)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 606-612: Method `RadioModel._extract_final`
```python
    def _extract_final(self, y: torch.Tensor):
        # Remove CLS + REGISTERS tokens
        patch_gen = getattr(self.model, "patch_generator", None)
        if patch_gen is not None:
            all_feat = y[:, patch_gen.num_skip :]

        return all_feat
```
**EN:** This method implements `_extract_final(y: ...)` and implements a focused helper that supports the surrounding runtime flow inside `RadioModel`.
**CN:** 这个方法实现了 `_extract_final(y: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `RadioModel` 内部调用。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `logging`
- `math`
- `collections.abc.Iterable`
- `itertools.repeat`
- `typing.TypeAlias`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `einops.rearrange`
- `transformers.PretrainedConfig`
- `transformers.modeling_outputs.BaseModelOutput`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.model_loader.weight_utils.replace_prefix`
- `sglang.srt.model_loader.weight_utils.replace_substrings`
- `sglang.srt.models.internvl.InternVisionEncoder`
