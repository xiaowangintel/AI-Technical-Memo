# wanvae.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/vaes/wanvae.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for wanvae within the multimodal runtime. Key symbols include `forward_context`, `WanResample`, `WanResidualBlock`. / 该模块实现多模态运行时中与 wanvae 相关的模型构件。 关键符号包括 `forward_context`, `WanResample`, `WanResidualBlock`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-74: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0

# Copyright 2025 The Wan Team and The HuggingFace Team. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# ...
feat_idx = contextvars.ContextVar("feat_idx", default=0)
first_chunk = contextvars.ContextVar("first_chunk", default=None)

bind_context(is_first_frame, feat_cache, feat_idx, CACHE_T, first_chunk)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 77-91: Function `forward_context` / 函数 `forward_context`
```python
@contextmanager
def forward_context(
    first_frame_arg=False, feat_cache_arg=None, feat_idx_arg=None, first_chunk_arg=None
):
    is_first_frame_token = is_first_frame.set(first_frame_arg)
    feat_cache_token = feat_cache.set(feat_cache_arg)
    feat_idx_token = feat_idx.set(feat_idx_arg)
    first_chunk_token = first_chunk.set(first_chunk_arg)
    try:
        yield
    finally:
        is_first_frame.reset(is_first_frame_token)
        feat_cache.reset(feat_cache_token)
        feat_idx.reset(feat_idx_token)
        first_chunk.reset(first_chunk_token)
```
**EN:** This function drives `forward_context` with inputs such as `first_frame_arg`, `feat_cache_arg`, `feat_idx_arg`, `first_chunk_arg`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `forward_context`，主要处理 `first_frame_arg`, `feat_cache_arg`, `feat_idx_arg`, `first_chunk_arg` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 94-146: Class `WanResample` / 类 `WanResample`
```python
class WanResample(nn.Module):
    r"""
    A custom resampling module for 2D and 3D data.

    Args:
        dim (int): The number of input/output channels.
        mode (str): The resampling mode. Must be one of:
            - 'none': No resampling (identity operation).
            - 'upsample2d': 2D upsampling with nearest-exact interpolation and convolution.
            - 'upsample3d': 3D upsampling with nearest-exact interpolation, convolution, and causal 3D convolution.
            - 'downsample2d': 2D downsampling with zero-padding and convolution.
            - 'downsample3d': 3D downsampling with zero-padding, convolution, and causal 3D convolution.
    """

# ...
            self.resample = nn.Identity()

    def forward(self, x):
        return resample_forward(self, x)
```
**EN:** This class models `WanResample` as a specialization of `nn.Module`. A custom resampling module for 2D and 3D data. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanResample`，并继承/扩展 `nn.Module`。 文档字符串指出：A custom resampling module for 2D and 3D data. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 149-183: Class `WanResidualBlock` / 类 `WanResidualBlock`
```python
class WanResidualBlock(nn.Module):
    r"""
    A custom residual block module.

    Args:
        in_dim (int): Number of input channels.
        out_dim (int): Number of output channels.
        dropout (float, optional): Dropout rate for the dropout layer. Default is 0.0.
        non_linearity (str, optional): Type of non-linearity to use. Default is "silu".
    """

    def __init__(
        self,
        in_dim: int,
# ...
        )

    def forward(self, x):
        return residual_block_forward(self, x)
```
**EN:** This class models `WanResidualBlock` as a specialization of `nn.Module`. A custom residual block module. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanResidualBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A custom residual block module. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 186-204: Class `WanAttentionBlock` / 类 `WanAttentionBlock`
```python
class WanAttentionBlock(nn.Module):
    r"""
    Causal self-attention with a single head.

    Args:
        dim (int): The number of channels in the input tensor.
    """

    def __init__(self, dim) -> None:
        super().__init__()
        self.dim = dim

        # layers
        self.norm = WanRMS_norm(dim)
        self.to_qkv = nn.Conv2d(dim, dim * 3, 1)
        self.proj = nn.Conv2d(dim, dim, 1)

    def forward(self, x):
        return attention_block_forward(self, x)
```
**EN:** This class models `WanAttentionBlock` as a specialization of `nn.Module`. Causal self-attention with a single head. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanAttentionBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：Causal self-attention with a single head. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 207-239: Class `WanMidBlock` / 类 `WanMidBlock`
```python
class WanMidBlock(nn.Module):
    """
    Middle block for WanVAE encoder and decoder.

    Args:
        dim (int): Number of input/output channels.
        dropout (float): Dropout rate.
        non_linearity (str): Type of non-linearity to use.
    """

    def __init__(
        self,
        dim: int,
        dropout: float = 0.0,
# ...
        self.gradient_checkpointing = False

    def forward(self, x):
        return mid_block_forward(self, x)
```
**EN:** This class models `WanMidBlock` as a specialization of `nn.Module`. Middle block for WanVAE encoder and decoder. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanMidBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：Middle block for WanVAE encoder and decoder. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 242-278: Class `WanResidualDownBlock` / 类 `WanResidualDownBlock`
```python
class WanResidualDownBlock(nn.Module):

    def __init__(
        self,
        in_dim,
        out_dim,
        dropout,
        num_res_blocks,
        temperal_downsample=False,
        down_flag=False,
    ):
        super().__init__()

        # Shortcut path with downsample
# ...
            self.downsampler = None

    def forward(self, x):
        return residual_down_block_forward(self, x)
```
**EN:** This class models `WanResidualDownBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanResidualDownBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 281-463: Class `WanEncoder3d` / 类 `WanEncoder3d`
```python
class WanEncoder3d(nn.Module):
    r"""
    A 3D encoder module.

    Args:
        dim (int): The base number of channels in the first layer.
        z_dim (int): The dimensionality of the latent space.
        dim_mult (list of int): Multipliers for the number of channels in each block.
        num_res_blocks (int): Number of residual blocks in each block.
        attn_scales (list of float): Scales at which to apply attention mechanisms.
        temperal_downsample (list of bool): Whether to downsample temporally in each block.
        dropout (float): Dropout rate for the dropout layers.
        non_linearity (str): Type of non-linearity to use.
    """
# ...

        if self.use_parallel_encode and self.world_size > 1:
            x = gather_and_trim_height(x, expected_height)
        return x
```
**EN:** This class models `WanEncoder3d` as a specialization of `nn.Module`. A 3D encoder module. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanEncoder3d`，并继承/扩展 `nn.Module`。 文档字符串指出：A 3D encoder module. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 467-527: Class `WanResidualUpBlock` / 类 `WanResidualUpBlock`
```python
class WanResidualUpBlock(nn.Module):
    """
    A block that handles upsampling for the WanVAE decoder.
    Args:
        in_dim (int): Input dimension
        out_dim (int): Output dimension
        num_res_blocks (int): Number of residual blocks
        dropout (float): Dropout rate
        temperal_upsample (bool): Whether to upsample on temporal dimension
        up_flag (bool): Whether to upsample or not
        non_linearity (str): Type of non-linearity to use
    """

    def __init__(
# ...
        self.gradient_checkpointing = False

    def forward(self, x):
        return residual_up_block_forward(self, x)
```
**EN:** This class models `WanResidualUpBlock` as a specialization of `nn.Module`. A block that handles upsampling for the WanVAE decoder. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanResidualUpBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A block that handles upsampling for the WanVAE decoder. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 530-576: Class `WanUpBlock` / 类 `WanUpBlock`
```python
class WanUpBlock(nn.Module):
    """
    A block that handles upsampling for the WanVAE decoder.

    Args:
        in_dim (int): Input dimension
        out_dim (int): Output dimension
        num_res_blocks (int): Number of residual blocks
        dropout (float): Dropout rate
        upsample_mode (str, optional): Mode for upsampling ('upsample2d' or 'upsample3d')
        non_linearity (str): Type of non-linearity to use
    """

    def __init__(
# ...
        self.gradient_checkpointing = False

    def forward(self, x):
        return up_block_forward(self, x)
```
**EN:** This class models `WanUpBlock` as a specialization of `nn.Module`. A block that handles upsampling for the WanVAE decoder. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanUpBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A block that handles upsampling for the WanVAE decoder. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 579-766: Class `WanDecoder3d` / 类 `WanDecoder3d`
```python
class WanDecoder3d(nn.Module):
    r"""
    A 3D decoder module.

    Args:
        dim (int): The base number of channels in the first layer.
        z_dim (int): The dimensionality of the latent space.
        dim_mult (list of int): Multipliers for the number of channels in each block.
        num_res_blocks (int): Number of residual blocks in each block.
        attn_scales (list of float): Scales at which to apply attention mechanisms.
        temperal_upsample (list of bool): Whether to upsample temporally in each block.
        dropout (float): Dropout rate for the dropout layers.
        non_linearity (str): Type of non-linearity to use.
    """
# ...

        if self.use_parallel_decode and self.world_size > 1:
            x = gather_and_trim_height(x, expected_height)
        return x
```
**EN:** This class models `WanDecoder3d` as a specialization of `nn.Module`. A 3D decoder module. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDecoder3d`，并继承/扩展 `nn.Module`。 文档字符串指出：A 3D decoder module. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 769-785: Function `patchify` / 函数 `patchify`
```python
def patchify(x, patch_size):
    if patch_size == 1:
        return x

    if x.dim() == 4:
        x = rearrange(x, "b c (h q) (w r) -> b (c r q) h w", q=patch_size, r=patch_size)
    elif x.dim() == 5:
        x = rearrange(
            x,
            "b c f (h q) (w r) -> b (c r q) f h w",
            q=patch_size,
            r=patch_size,
        )
    else:
        raise ValueError(f"Invalid input shape: {x.shape}")

    return x
```
**EN:** This function drives `patchify` with inputs such as `x`, `patch_size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `patchify`，主要处理 `x`, `patch_size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 788-802: Function `unpatchify` / 函数 `unpatchify`
```python
def unpatchify(x, patch_size):
    if patch_size == 1:
        return x

    if x.dim() == 4:
        x = rearrange(x, "b (c r q) h w -> b c (h q) (w r)", q=patch_size, r=patch_size)
    elif x.dim() == 5:
        x = rearrange(
            x,
            "b (c r q) f h w -> b c f (h q) (w r)",
            q=patch_size,
            r=patch_size,
        )

    return x
```
**EN:** This function drives `unpatchify` with inputs such as `x`, `patch_size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `unpatchify`，主要处理 `x`, `patch_size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 805-1021: Class `AutoencoderKLWan` / 类 `AutoencoderKLWan`
```python
class AutoencoderKLWan(ParallelTiledVAE):
    r"""
    A VAE model with KL loss for encoding videos into latents and decoding latent representations into videos.
    Introduced in [Wan 2.1].
    """

    _supports_gradient_checkpointing = False

    def __init__(
        self,
        config: WanVAEConfig,
    ) -> None:
        nn.Module.__init__(self)
        ParallelTiledVAE.__init__(self, config)
# ...
        else:
            z = posterior.mode()
        dec = self.decode(z)
        return dec
```
**EN:** This class models `AutoencoderKLWan` as a specialization of `ParallelTiledVAE`. A VAE model with KL loss for encoding videos into latents and decoding latent representations into videos. Important methods include `__init__`, `clear_cache`, `encode`, `_encode`.
**CN:** 该类实现 `AutoencoderKLWan`，并继承/扩展 `ParallelTiledVAE`。 文档字符串指出：A VAE model with KL loss for encoding videos into latents and decoding latent representations into videos. 其中较重要的方法包括 `__init__`, `clear_cache`, `encode`, `_encode`。

### Lines 1022-1024: Top-level configuration / 顶层配置
```python


EntryClass = AutoencoderKLWan
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.vaes`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.activation`, `sglang.multimodal_gen.runtime.models.vaes.common`, `sglang.multimodal_gen.runtime.models.vaes.parallel.wan_common_utils`, `sglang.multimodal_gen.runtime.models.vaes.parallel.wan_dist_utils`
- **External / 外部**: `contextvars`, `torch`, `torch.distributed`, `torch.nn`, `einops`
- **Stdlib / 标准库**: `contextlib`
