# hunyuanvae.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/vaes/hunyuanvae.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for hunyuanvae within the multimodal runtime. Key symbols include `prepare_causal_attention_mask`, `HunyuanVAEAttention`, `HunyuanVideoCausalConv3d`. / 该模块实现多模态运行时中与 hunyuanvae 相关的模型构件。 关键符号包括 `prepare_causal_attention_mask`, `HunyuanVAEAttention`, `HunyuanVideoCausalConv3d`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Adapted from diffusers

# Copyright 2024 The Hunyuan Team, The HuggingFace Team and The sglang-diffusion Team. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# ...
from sglang.jit_kernel.diffusion.group_norm_silu import apply_group_norm_silu
from sglang.multimodal_gen.configs.models.vaes import HunyuanVAEConfig
from sglang.multimodal_gen.runtime.layers.activation import get_act_fn
from sglang.multimodal_gen.runtime.models.vaes.common import ParallelTiledVAE
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 31-45: Function `prepare_causal_attention_mask` / 函数 `prepare_causal_attention_mask`
```python
def prepare_causal_attention_mask(
    num_frames: int,
    height_width: int,
    dtype: torch.dtype,
    device: torch.device,
    batch_size: int | None = None,
) -> torch.Tensor:
    indices = torch.arange(1, num_frames + 1, dtype=torch.int32, device=device)
    indices_blocks = indices.repeat_interleave(height_width)
    x, y = torch.meshgrid(indices_blocks, indices_blocks, indexing="xy")
    mask = torch.where(x <= y, 0, -float("inf")).to(dtype=dtype)

    if batch_size is not None:
        mask = mask.unsqueeze(0).expand(batch_size, -1, -1)
    return mask
```
**EN:** This function drives `prepare_causal_attention_mask` with inputs such as `num_frames`, `height_width`, `dtype`, `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `prepare_causal_attention_mask`，主要处理 `num_frames`, `height_width`, `dtype`, `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 48-112: Class `HunyuanVAEAttention` / 类 `HunyuanVAEAttention`
```python
class HunyuanVAEAttention(nn.Module):

    def __init__(
        self, in_channels, heads, dim_head, eps, norm_num_groups, bias
    ) -> None:
        super().__init__()
        self.in_channels = in_channels
        self.heads = heads
        self.dim_head = dim_head
        self.eps = eps
        self.norm_num_groups = norm_num_groups
        self.bias = bias

        inner_dim = heads * dim_head
# ...
        # Residual connection and rescale
        hidden_states = hidden_states + residual

        return hidden_states
```
**EN:** This class models `HunyuanVAEAttention` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanVAEAttention`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 115-154: Class `HunyuanVideoCausalConv3d` / 类 `HunyuanVideoCausalConv3d`
```python
class HunyuanVideoCausalConv3d(nn.Module):

    def __init__(
        self,
        in_channels: int,
        out_channels: int,
        kernel_size: int | tuple[int, int, int] = 3,
        stride: int | tuple[int, int, int] = 1,
        padding: int | tuple[int, int, int] = 0,
        dilation: int | tuple[int, int, int] = 1,
        bias: bool = True,
        pad_mode: str = "replicate",
    ) -> None:
        super().__init__()
# ...
        hidden_states = F.pad(
            hidden_states, self.time_causal_padding, mode=self.pad_mode
        )
        return self.conv(hidden_states)
```
**EN:** This class models `HunyuanVideoCausalConv3d` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanVideoCausalConv3d`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 157-203: Class `HunyuanVideoUpsampleCausal3D` / 类 `HunyuanVideoUpsampleCausal3D`
```python
class HunyuanVideoUpsampleCausal3D(nn.Module):

    def __init__(
        self,
        in_channels: int,
        out_channels: int | None = None,
        kernel_size: int = 3,
        stride: int = 1,
        bias: bool = True,
        upsample_factor: tuple[int, ...] = (2, 2, 2),
    ) -> None:
        super().__init__()

        out_channels = out_channels or in_channels
# ...
            hidden_states = first_frame

        hidden_states = self.conv(hidden_states)
        return hidden_states
```
**EN:** This class models `HunyuanVideoUpsampleCausal3D` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanVideoUpsampleCausal3D`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 206-226: Class `HunyuanVideoDownsampleCausal3D` / 类 `HunyuanVideoDownsampleCausal3D`
```python
class HunyuanVideoDownsampleCausal3D(nn.Module):

    def __init__(
        self,
        channels: int,
        out_channels: int | None = None,
        padding: int = 1,
        kernel_size: int = 3,
        bias: bool = True,
        stride=2,
    ) -> None:
        super().__init__()
        out_channels = out_channels or channels

# ...

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = self.conv(hidden_states)
        return hidden_states
```
**EN:** This class models `HunyuanVideoDownsampleCausal3D` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanVideoDownsampleCausal3D`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 229-277: Class `HunyuanVideoResnetBlockCausal3D` / 类 `HunyuanVideoResnetBlockCausal3D`
```python
class HunyuanVideoResnetBlockCausal3D(nn.Module):

    def __init__(
        self,
        in_channels: int,
        out_channels: int | None = None,
        dropout: float = 0.0,
        groups: int = 32,
        eps: float = 1e-6,
        non_linearity: str = "silu",
    ) -> None:
        super().__init__()
        out_channels = out_channels or in_channels

# ...
            residual = self.conv_shortcut(residual)

        hidden_states = hidden_states + residual
        return hidden_states
```
**EN:** This class models `HunyuanVideoResnetBlockCausal3D` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanVideoResnetBlockCausal3D`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 280-392: Class `HunyuanVideoMidBlock3D` / 类 `HunyuanVideoMidBlock3D`
```python
class HunyuanVideoMidBlock3D(nn.Module):

    def __init__(
        self,
        in_channels: int,
        dropout: float = 0.0,
        num_layers: int = 1,
        resnet_eps: float = 1e-6,
        resnet_act_fn: str = "silu",
        resnet_groups: int = 32,
        add_attention: bool = True,
        attention_head_dim: int = 1,
    ) -> None:
        super().__init__()
# ...

                hidden_states = resnet(hidden_states)

        return hidden_states
```
**EN:** This class models `HunyuanVideoMidBlock3D` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanVideoMidBlock3D`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 395-456: Class `HunyuanVideoDownBlock3D` / 类 `HunyuanVideoDownBlock3D`
```python
class HunyuanVideoDownBlock3D(nn.Module):

    def __init__(
        self,
        in_channels: int,
        out_channels: int,
        dropout: float = 0.0,
        num_layers: int = 1,
        resnet_eps: float = 1e-6,
        resnet_act_fn: str = "silu",
        resnet_groups: int = 32,
        add_downsample: bool = True,
        downsample_stride: tuple[int, ...] | int = 2,
        downsample_padding: int = 1,
# ...
            for downsampler in self.downsamplers:
                hidden_states = downsampler(hidden_states)

        return hidden_states
```
**EN:** This class models `HunyuanVideoDownBlock3D` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanVideoDownBlock3D`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 459-520: Class `HunyuanVideoUpBlock3D` / 类 `HunyuanVideoUpBlock3D`
```python
class HunyuanVideoUpBlock3D(nn.Module):

    def __init__(
        self,
        in_channels: int,
        out_channels: int,
        dropout: float = 0.0,
        num_layers: int = 1,
        resnet_eps: float = 1e-6,
        resnet_act_fn: str = "silu",
        resnet_groups: int = 32,
        add_upsample: bool = True,
        upsample_scale_factor: tuple[int, ...] = (2, 2, 2),
    ) -> None:
# ...
            for upsampler in self.upsamplers:
                hidden_states = upsampler(hidden_states)

        return hidden_states
```
**EN:** This class models `HunyuanVideoUpBlock3D` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanVideoUpBlock3D`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 523-642: Class `HunyuanVideoEncoder3D` / 类 `HunyuanVideoEncoder3D`
```python
class HunyuanVideoEncoder3D(nn.Module):
    r"""
    Causal encoder for 3D video-like data introduced in [Hunyuan Video](https://huggingface.co/papers/2412.03603).
    """

    def __init__(
        self,
        in_channels: int = 3,
        out_channels: int = 3,
        down_block_types: tuple[str, ...] = (
            "HunyuanVideoDownBlock3D",
            "HunyuanVideoDownBlock3D",
            "HunyuanVideoDownBlock3D",
            "HunyuanVideoDownBlock3D",
# ...
        )
        hidden_states = self.conv_out(hidden_states)

        return hidden_states
```
**EN:** This class models `HunyuanVideoEncoder3D` as a specialization of `nn.Module`. Causal encoder for 3D video-like data introduced in [Hunyuan Video](https://huggingface.co/papers/2412.03603). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanVideoEncoder3D`，并继承/扩展 `nn.Module`。 文档字符串指出：Causal encoder for 3D video-like data introduced in [Hunyuan Video](https://huggingface.co/papers/2412.03603). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 645-765: Class `HunyuanVideoDecoder3D` / 类 `HunyuanVideoDecoder3D`
```python
class HunyuanVideoDecoder3D(nn.Module):
    r"""
    Causal decoder for 3D video-like data introduced in [Hunyuan Video](https://huggingface.co/papers/2412.03603).
    """

    def __init__(
        self,
        in_channels: int = 3,
        out_channels: int = 3,
        up_block_types: tuple[str, ...] = (
            "HunyuanVideoUpBlock3D",
            "HunyuanVideoUpBlock3D",
            "HunyuanVideoUpBlock3D",
            "HunyuanVideoUpBlock3D",
# ...
        )
        hidden_states = self.conv_out(hidden_states)

        return hidden_states
```
**EN:** This class models `HunyuanVideoDecoder3D` as a specialization of `nn.Module`. Causal decoder for 3D video-like data introduced in [Hunyuan Video](https://huggingface.co/papers/2412.03603). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanVideoDecoder3D`，并继承/扩展 `nn.Module`。 文档字符串指出：Causal decoder for 3D video-like data introduced in [Hunyuan Video](https://huggingface.co/papers/2412.03603). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 768-854: Class `AutoencoderKLHunyuanVideo` / 类 `AutoencoderKLHunyuanVideo`
```python
class AutoencoderKLHunyuanVideo(ParallelTiledVAE):
    r"""
    A VAE model with KL loss for encoding videos into latents and decoding latent representations into videos.
    Introduced in [HunyuanVideo](https://huggingface.co/papers/2412.03603).

    This model inherits from [`ModelMixin`]. Check the superclass documentation for it's generic methods implemented
    for all models (such as downloading or saving).
    """

    _supports_gradient_checkpointing = True

    def __init__(
        self,
        config: HunyuanVAEConfig,
# ...
        else:
            z = posterior.mode()
        dec = self.decode(z)
        return dec
```
**EN:** This class models `AutoencoderKLHunyuanVideo` as a specialization of `ParallelTiledVAE`. A VAE model with KL loss for encoding videos into latents and decoding latent representations into videos. Important methods include `__init__`, `_encode`, `_decode`, `forward`.
**CN:** 该类实现 `AutoencoderKLHunyuanVideo`，并继承/扩展 `ParallelTiledVAE`。 文档字符串指出：A VAE model with KL loss for encoding videos into latents and decoding latent representations into videos. 其中较重要的方法包括 `__init__`, `_encode`, `_decode`, `forward`。

### Lines 855-857: Top-level configuration / 顶层配置
```python


EntryClass = AutoencoderKLHunyuanVideo
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Distributed execution / 分布式执行
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.jit_kernel.diffusion.group_norm_silu`, `sglang.multimodal_gen.configs.models.vaes`, `sglang.multimodal_gen.runtime.layers.activation`, `sglang.multimodal_gen.runtime.models.vaes.common`
- **External / 外部**: `numpy`, `torch`, `torch.nn`, `torch.nn.functional`
