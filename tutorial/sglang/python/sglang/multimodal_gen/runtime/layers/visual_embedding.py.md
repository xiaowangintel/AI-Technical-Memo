# visual_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/visual_embedding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `PatchEmbed`, `Timesteps`, and `CombinedTimestepGuidanceTextProjEmbeddings`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `PatchEmbed`、`Timesteps` 和 `CombinedTimestepGuidanceTextProjEmbeddings` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 5-31: module setup and imports / 模块初始化与导入
```python
import math

import torch
import torch.nn as nn
import torch.nn.functional as F
from diffusers.models.embeddings import (
    CombinedTimestepGuidanceTextProjEmbeddings as _CombinedTimestepGuidanceTextProjEmbeddings,
)
from diffusers.models.embeddings import (
    CombinedTimestepTextProjEmbeddings as _CombinedTimestepTextProjEmbeddings,
)
from diffusers.models.embeddings import (
    PixArtAlphaTextProjection,
    TimestepEmbedding,
)
from diffusers.models.embeddings import Timesteps as _Timesteps
from diffusers.models.embeddings import (
    get_timestep_embedding as timestep_embedding_diffusers,
)

from sglang.jit_kernel.timestep_embedding import (
    timestep_embedding as timestep_embedding_cuda,
)
from sglang.multimodal_gen.runtime.layers.activation import get_act_fn
from sglang.multimodal_gen.runtime.layers.linear import ColumnParallelLinear
from sglang.multimodal_gen.runtime.layers.mlp import MLP
from sglang.multimodal_gen.runtime.platforms import current_platform
```
**EN:** This block establishes the module context and imports `math`, `torch`, `torch.nn`, `torch.nn.functional`, `diffusers.models.embeddings`, and `sglang.jit_kernel.timestep_embedding`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `math`、`torch`、`torch.nn`、`torch.nn.functional`、`diffusers.models.embeddings` 和 `sglang.jit_kernel.timestep_embedding`。这些依赖为后续实现提供所需符号。

### Lines 33-33: supporting statements / 辅助语句
```python
_is_cuda = current_platform.is_cuda()
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_is_cuda`. The code collaborates with `current_platform.is_cuda`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_is_cuda` 等名称。 代码会与 `current_platform.is_cuda` 协同工作。

### Lines 36-49: `PatchEmbed` class overview / `PatchEmbed` 类概览
```python
class PatchEmbed(nn.Module):
    """2D Image to Patch Embedding

    Image to Patch Embedding using Conv2d

    A convolution based approach to patchifying a 2D image w/ embedding projection.

    Based on the impl in https://github.com/google-research/vision_transformer

    Hacked together by / Copyright 2020 Ross Wightman

    Remove the _assert function in forward function to be compatible with multi-resolution images.
    """
```
**EN:** This block defines class `PatchEmbed`. 2D Image to Patch Embedding Image to Patch Embedding using Conv2d A convolution based approach to patchifying a 2D image w/ embedding projection. Based on the impl in https://github.com/google-research/vision_transformer Hacked together by / Copyright 2020 Ross Wightman Remove the _assert function in forward function to be compatible with multi-resolution images. It inherits from `nn.Module`.
**CN:** 该代码块定义了类 `PatchEmbed`。 它用于封装 patch embed 相关行为。 它继承自 `nn.Module`。

### Lines 50-81: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        patch_size=16,
        in_chans=3,
        embed_dim=768,
        norm_layer=None,
        flatten=True,
        bias=True,
        dtype=None,
        prefix: str = "",
    ):
        super().__init__()
        if isinstance(patch_size, list | tuple):
            if len(patch_size) == 1:
                patch_size = (1, patch_size[0], patch_size[0])
            elif len(patch_size) == 2:
                patch_size = (1, patch_size[0], patch_size[1])
        else:
            patch_size = (1, patch_size, patch_size)

        self.patch_size = patch_size
        self.flatten = flatten

        self.proj = nn.Conv3d(
            in_chans,
            embed_dim,
            kernel_size=patch_size,
            stride=patch_size,
            bias=bias,
            dtype=dtype,
        )
        self.norm = norm_layer(embed_dim) if norm_layer else nn.Identity()
```
**EN:** This block defines method `__init__` on `PatchEmbed`. It initializes the instance state. Key calls include `super.__init__`, `isinstance`, `nn.Conv3d`, `norm_layer`, and `nn.Identity`. The implementation branches on conditions. Parameters such as `patch_size`, `in_chans`, `embed_dim`, `norm_layer`, and `flatten` drive the behavior in this section.
**CN:** 该代码块定义了 `PatchEmbed` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`isinstance`、`nn.Conv3d`、`norm_layer` 和 `nn.Identity`。 实现中包含条件分支。 本段逻辑主要由 `patch_size`、`in_chans`、`embed_dim`、`norm_layer` 和 `flatten` 等参数驱动。

### Lines 83-111: `forward` implementation / `forward` 实现
```python
    def forward(self, x):
        if x.dim() == 5:
            B, C, T, H, W = x.shape
            pt, ph, pw = self.patch_size

            if T % pt == 0 and H % ph == 0 and W % pw == 0:
                T_ = T // pt
                H_ = H // ph
                W_ = W // pw

                x = x.reshape(B, C, T_, pt, H_, ph, W_, pw)
                x = x.permute(0, 2, 4, 6, 1, 3, 5, 7).contiguous()
                x = x.reshape(B, T_ * H_ * W_, C * pt * ph * pw)

                w = self.proj.weight.reshape(self.proj.weight.shape[0], -1)
                x = F.linear(x, w, self.proj.bias)  # [B, T'*H'*W', embed_dim]

                if not self.flatten:
                    x = x.reshape(B, T_, H_, W_, -1).permute(0, 4, 1, 2, 3).contiguous()

                x = self.norm(x)
                return x

        # Fallback to Conv3d for non-5D input or indivisible spatial dims.
        x = self.proj(x)
        if self.flatten:
            x = x.flatten(2).transpose(1, 2)
        x = self.norm(x)
        return x
```
**EN:** This block defines method `forward` on `PatchEmbed`. It executes function. Key calls include `self.proj`, `self.norm`, `x.dim`, `x.flatten.transpose`, and `x.reshape`. The implementation branches on conditions. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `PatchEmbed` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `self.proj`、`self.norm`、`x.dim`、`x.flatten.transpose` 和 `x.reshape`。 实现中包含条件分支。 本段逻辑主要由 `x` 等参数驱动。

### Lines 114-114: `Timesteps` class overview / `Timesteps` 类概览
```python
class Timesteps(_Timesteps):
```
**EN:** This block defines class `Timesteps`. It encapsulates timesteps behavior. It inherits from `_Timesteps`.
**CN:** 该代码块定义了类 `Timesteps`。 它用于封装 timesteps 相关行为。 它继承自 `_Timesteps`。

### Lines 115-131: `forward` implementation / `forward` 实现
```python
    def forward(self, timesteps: torch.Tensor) -> torch.Tensor:
        if _is_cuda:
            return timestep_embedding_cuda(
                timesteps,
                self.num_channels,
                flip_sin_to_cos=self.flip_sin_to_cos,
                downscale_freq_shift=self.downscale_freq_shift,
                scale=self.scale,
            )
        else:
            return timestep_embedding_diffusers(
                timesteps,
                self.num_channels,
                flip_sin_to_cos=self.flip_sin_to_cos,
                downscale_freq_shift=self.downscale_freq_shift,
                scale=self.scale,
            )
```
**EN:** This block defines method `forward` on `Timesteps`. It executes function. Key calls include `timestep_embedding_cuda`, and `timestep_embedding_diffusers`. The implementation branches on conditions. Parameters such as `timesteps` drive the behavior in this section.
**CN:** 该代码块定义了 `Timesteps` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `timestep_embedding_cuda` 和 `timestep_embedding_diffusers`。 实现中包含条件分支。 本段逻辑主要由 `timesteps` 等参数驱动。

### Lines 134-136: `CombinedTimestepGuidanceTextProjEmbeddings` class overview / `CombinedTimestepGuidanceTextProjEmbeddings` 类概览
```python
class CombinedTimestepGuidanceTextProjEmbeddings(
    _CombinedTimestepGuidanceTextProjEmbeddings
):
```
**EN:** This block defines class `CombinedTimestepGuidanceTextProjEmbeddings`. It encapsulates combined timestep guidance text proj embeddings behavior. It inherits from `_CombinedTimestepGuidanceTextProjEmbeddings`.
**CN:** 该代码块定义了类 `CombinedTimestepGuidanceTextProjEmbeddings`。 它用于封装 combined timestep guidance text proj embeddings 相关行为。 它继承自 `_CombinedTimestepGuidanceTextProjEmbeddings`。

### Lines 137-153: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, embedding_dim, pooled_projection_dim):
        nn.Module.__init__(self)

        # use sgld op
        self.time_proj = Timesteps(
            num_channels=256, flip_sin_to_cos=True, downscale_freq_shift=0
        )
        # use diffusers op
        self.timestep_embedder = TimestepEmbedding(
            in_channels=256, time_embed_dim=embedding_dim
        )
        self.guidance_embedder = TimestepEmbedding(
            in_channels=256, time_embed_dim=embedding_dim
        )
        self.text_embedder = PixArtAlphaTextProjection(
            pooled_projection_dim, embedding_dim, act_fn="silu"
        )
```
**EN:** This block defines method `__init__` on `CombinedTimestepGuidanceTextProjEmbeddings`. It initializes the instance state. Key calls include `nn.Module.__init__`, `Timesteps`, `TimestepEmbedding`, and `PixArtAlphaTextProjection`. Parameters such as `embedding_dim`, and `pooled_projection_dim` drive the behavior in this section.
**CN:** 该代码块定义了 `CombinedTimestepGuidanceTextProjEmbeddings` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `nn.Module.__init__`、`Timesteps`、`TimestepEmbedding` 和 `PixArtAlphaTextProjection`。 本段逻辑主要由 `embedding_dim` 和 `pooled_projection_dim` 等参数驱动。

### Lines 156-156: `CombinedTimestepTextProjEmbeddings` class overview / `CombinedTimestepTextProjEmbeddings` 类概览
```python
class CombinedTimestepTextProjEmbeddings(_CombinedTimestepTextProjEmbeddings):
```
**EN:** This block defines class `CombinedTimestepTextProjEmbeddings`. It encapsulates combined timestep text proj embeddings behavior. It inherits from `_CombinedTimestepTextProjEmbeddings`.
**CN:** 该代码块定义了类 `CombinedTimestepTextProjEmbeddings`。 它用于封装 combined timestep text proj embeddings 相关行为。 它继承自 `_CombinedTimestepTextProjEmbeddings`。

### Lines 157-170: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, embedding_dim, pooled_projection_dim):
        nn.Module.__init__(self)

        # use sgld op
        self.time_proj = Timesteps(
            num_channels=256, flip_sin_to_cos=True, downscale_freq_shift=0
        )
        # use diffusers op
        self.timestep_embedder = TimestepEmbedding(
            in_channels=256, time_embed_dim=embedding_dim
        )
        self.text_embedder = PixArtAlphaTextProjection(
            pooled_projection_dim, embedding_dim, act_fn="silu"
        )
```
**EN:** This block defines method `__init__` on `CombinedTimestepTextProjEmbeddings`. It initializes the instance state. Key calls include `nn.Module.__init__`, `Timesteps`, `TimestepEmbedding`, and `PixArtAlphaTextProjection`. Parameters such as `embedding_dim`, and `pooled_projection_dim` drive the behavior in this section.
**CN:** 该代码块定义了 `CombinedTimestepTextProjEmbeddings` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `nn.Module.__init__`、`Timesteps`、`TimestepEmbedding` 和 `PixArtAlphaTextProjection`。 本段逻辑主要由 `embedding_dim` 和 `pooled_projection_dim` 等参数驱动。

### Lines 173-177: `TimestepEmbedder` class overview / `TimestepEmbedder` 类概览
```python
class TimestepEmbedder(nn.Module):
    """
    Embeds scalar timesteps into vector representations.
    """
```
**EN:** This block defines class `TimestepEmbedder`. Embeds scalar timesteps into vector representations. It inherits from `nn.Module`.
**CN:** 该代码块定义了类 `TimestepEmbedder`。 它用于封装 timestep embedder 相关行为。 它继承自 `nn.Module`。

### Lines 178-199: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        hidden_size,
        act_layer="silu",
        frequency_embedding_size=256,
        max_period=10000,
        dtype=None,
        freq_dtype=torch.float32,
        prefix: str = "",
    ):
        super().__init__()
        self.frequency_embedding_size = frequency_embedding_size
        self.max_period = max_period

        self.mlp = MLP(
            frequency_embedding_size,
            hidden_size,
            hidden_size,
            act_type=act_layer,
            dtype=dtype,
        )
        self.freq_dtype = freq_dtype
```
**EN:** This block defines method `__init__` on `TimestepEmbedder`. It initializes the instance state. Key calls include `super.__init__`, `MLP`, and `super`. Parameters such as `hidden_size`, `act_layer`, `frequency_embedding_size`, `max_period`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `TimestepEmbedder` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`MLP` 和 `super`。 本段逻辑主要由 `hidden_size`、`act_layer`、`frequency_embedding_size`、`max_period` 和 `dtype` 等参数驱动。

### Lines 201-215: `forward` implementation / `forward` 实现
```python
    def forward(
        self, t: torch.Tensor, timestep_seq_len: int | None = None
    ) -> torch.Tensor:
        t_freq = timestep_embedding(
            t, self.frequency_embedding_size, self.max_period, dtype=self.freq_dtype
        ).to(self.mlp.fc_in.weight.dtype)
        if timestep_seq_len is not None:
            assert (
                t_freq.shape[0] % timestep_seq_len == 0
            ), "timestep length is not divisible by timestep_seq_len"
            batch_size = t_freq.shape[0] // timestep_seq_len
            t_freq = t_freq.unflatten(0, (batch_size, timestep_seq_len))
        # t_freq = t_freq.to(self.mlp.fc_in.weight.dtype)
        t_emb = self.mlp(t_freq)
        return t_emb
```
**EN:** This block defines method `forward` on `TimestepEmbedder`. It executes function. Key calls include `timestep_embedding.to`, `self.mlp`, `t_freq.unflatten`, and `timestep_embedding`. The implementation branches on conditions. Parameters such as `t`, and `timestep_seq_len` drive the behavior in this section.
**CN:** 该代码块定义了 `TimestepEmbedder` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `timestep_embedding.to`、`self.mlp`、`t_freq.unflatten` 和 `timestep_embedding`。 实现中包含条件分支。 本段逻辑主要由 `t` 和 `timestep_seq_len` 等参数驱动。

### Lines 218-245: `timestep_embedding` implementation / `timestep_embedding` 实现
```python
def timestep_embedding(
    t: torch.Tensor,
    dim: int,
    max_period: int = 10000,
    dtype: torch.dtype = torch.float32,
) -> torch.Tensor:
    """
    Create sinusoidal timestep embeddings.

    Args:
        t: Tensor of shape [B] with timesteps
        dim: Embedding dimension
        max_period: Controls the minimum frequency of the embeddings

    Returns:
        Tensor of shape [B, dim] with embeddings
    """
    half = dim // 2
    freqs = torch.exp(
        -math.log(max_period)
        * torch.arange(start=0, end=half, dtype=dtype, device=t.device)
        / half
    )
    args = t[:, None].float() * freqs[None]
    embedding = torch.cat([torch.cos(args), torch.sin(args)], dim=-1)
    if dim % 2:
        embedding = torch.cat([embedding, torch.zeros_like(embedding[:, :1])], dim=-1)
    return embedding
```
**EN:** This block defines function `timestep_embedding`. Create sinusoidal timestep embeddings. Args: t: Tensor of shape [B] with timesteps dim: Embedding dimension max_period: Controls the minimum frequency of the embeddings Returns: Tensor of shape [B, dim] with embeddings Key calls include `torch.exp`, `torch.cat`, `t.float`, `torch.cos`, and `torch.sin`. The implementation branches on conditions. Parameters such as `t`, `dim`, `max_period`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了函数 `timestep_embedding`。 它用于处理 timestep embedding 相关逻辑。 关键调用包括 `torch.exp`、`torch.cat`、`t.float`、`torch.cos` 和 `torch.sin`。 实现中包含条件分支。 本段逻辑主要由 `t`、`dim`、`max_period` 和 `dtype` 等参数驱动。

### Lines 248-250: `ModulateProjection` class overview / `ModulateProjection` 类概览
```python
class ModulateProjection(nn.Module):
    """Modulation layer for DiT blocks."""
```
**EN:** This block defines class `ModulateProjection`. Modulation layer for DiT blocks. It inherits from `nn.Module`.
**CN:** 该代码块定义了类 `ModulateProjection`。 它用于封装 modulate projection 相关行为。 它继承自 `nn.Module`。

### Lines 251-269: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        hidden_size: int,
        factor: int = 2,
        act_layer: str = "silu",
        dtype: torch.dtype | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.factor = factor
        self.hidden_size = hidden_size
        self.linear = ColumnParallelLinear(
            hidden_size,
            hidden_size * factor,
            bias=True,
            gather_output=True,
            params_dtype=dtype,
        )
        self.act = get_act_fn(act_layer)
```
**EN:** This block defines method `__init__` on `ModulateProjection`. It initializes the instance state. Key calls include `super.__init__`, `ColumnParallelLinear`, `get_act_fn`, and `super`. Parameters such as `hidden_size`, `factor`, `act_layer`, `dtype`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `ModulateProjection` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`ColumnParallelLinear`、`get_act_fn` 和 `super`。 本段逻辑主要由 `hidden_size`、`factor`、`act_layer`、`dtype` 和 `prefix` 等参数驱动。

### Lines 271-274: `forward` implementation / `forward` 实现
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = self.act(x)
        x, _ = self.linear(x)
        return x
```
**EN:** This block defines method `forward` on `ModulateProjection`. It executes function. Key calls include `self.act`, and `self.linear`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `ModulateProjection` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `self.act` 和 `self.linear`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 277-298: `unpatchify` implementation / `unpatchify` 实现
```python
def unpatchify(x, t, h, w, patch_size, channels) -> torch.Tensor:
    """
    Convert patched representation back to image space.

    Args:
        x: Tensor of shape [B, T*H*W, C*P_t*P_h*P_w]
        t, h, w: Temporal and spatial dimensions

    Returns:
        Unpatchified tensor of shape [B, C, T*P_t, H*P_h, W*P_w]
    """
    assert x.ndim == 3, f"x.ndim: {x.ndim}"
    assert len(patch_size) == 3, f"patch_size: {patch_size}"
    assert t * h * w == x.shape[1], f"t * h * w: {t * h * w}, x.shape[1]: {x.shape[1]}"
    c = channels
    pt, ph, pw = patch_size

    x = x.reshape(shape=(x.shape[0], t, h, w, c, pt, ph, pw))
    x = torch.einsum("nthwcopq->nctohpwq", x)
    imgs = x.reshape(shape=(x.shape[0], c, t * pt, h * ph, w * pw))

    return imgs
```
**EN:** This block defines function `unpatchify`. Convert patched representation back to image space. Args: x: Tensor of shape [B, T*H*W, C*P_t*P_h*P_w] t, h, w: Temporal and spatial dimensions Returns: Unpatchified tensor of shape [B, C, T*P_t, H*P_h, W*P_w] Key calls include `x.reshape`, `torch.einsum`, and `len`. Parameters such as `x`, `t`, `h`, `w`, and `patch_size` drive the behavior in this section.
**CN:** 该代码块定义了函数 `unpatchify`。 它用于处理 unpatchify 相关逻辑。 关键调用包括 `x.reshape`、`torch.einsum` 和 `len`。 本段逻辑主要由 `x`、`t`、`h`、`w` 和 `patch_size` 等参数驱动。

## Key Concepts / 关键概念
- `PatchEmbed`: 2D Image to Patch Embedding Image to Patch Embedding using Conv2d A convolution based approach to patchifying a 2D image w/ embedding projection. / 核心类，用于封装 patch embed 相关行为。
- `Timesteps`: Primary class that encapsulates timesteps behavior. / 核心类，用于封装 timesteps 相关行为。
- `CombinedTimestepGuidanceTextProjEmbeddings`: Primary class that encapsulates combined timestep guidance text proj embeddings behavior. / 核心类，用于封装 combined timestep guidance text proj embeddings 相关行为。
- `CombinedTimestepTextProjEmbeddings`: Primary class that encapsulates combined timestep text proj embeddings behavior. / 核心类，用于封装 combined timestep text proj embeddings 相关行为。
- `TimestepEmbedder`: Embeds scalar timesteps into vector representations. / 核心类，用于封装 timestep embedder 相关行为。
- `timestep_embedding`: Create sinusoidal timestep embeddings. / 顶层函数，用于处理 timestep embedding 相关逻辑。
- `ModulateProjection`: Modulation layer for DiT blocks. / 核心类，用于封装 modulate projection 相关行为。
- `unpatchify`: Convert patched representation back to image space. / 顶层函数，用于处理 unpatchify 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `math`
- **Third-party / 第三方依赖**: `torch`, `torch.nn`, `torch.nn.functional`, `diffusers.models.embeddings`
- **Internal modules / 内部模块**: `sglang.jit_kernel.timestep_embedding`, `sglang.multimodal_gen.runtime.layers.activation`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.mlp`, `sglang.multimodal_gen.runtime.platforms`

- **Total lines / 总行数**: 298
