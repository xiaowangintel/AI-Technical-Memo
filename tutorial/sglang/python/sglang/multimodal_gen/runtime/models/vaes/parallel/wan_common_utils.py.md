# wan_common_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/vaes/parallel/wan_common_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for wan common utils within the multimodal runtime. Key symbols include `AvgDown3D`, `DupUp3D`, `WanCausalConv3d`. / 该模块实现多模态运行时中与 wan common utils 相关的模型构件。 关键符号包括 `AvgDown3D`, `DupUp3D`, `WanCausalConv3d`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup / 导入与模块初始化
```python
from __future__ import annotations

import torch
import torch.nn as nn
import torch.nn.functional as F

from sglang.multimodal_gen.runtime.platforms import current_platform
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 10-60: Class `AvgDown3D` / 类 `AvgDown3D`
```python
class AvgDown3D(nn.Module):
    def __init__(
        self,
        in_channels,
        out_channels,
        factor_t,
        factor_s=1,
    ):
        super().__init__()
        self.in_channels = in_channels
        self.out_channels = out_channels
        self.factor_t = factor_t
        self.factor_s = factor_s
        self.factor = self.factor_t * self.factor_s * self.factor_s
# ...
            W // self.factor_s,
        )
        x = x.mean(dim=2)
        return x
```
**EN:** This class models `AvgDown3D` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `AvgDown3D`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 63-106: Class `DupUp3D` / 类 `DupUp3D`
```python
class DupUp3D(nn.Module):
    def __init__(
        self,
        in_channels: int,
        out_channels: int,
        factor_t,
        factor_s=1,
    ):
        super().__init__()
        self.in_channels = in_channels
        self.out_channels = out_channels

        self.factor_t = factor_t
        self.factor_s = factor_s
# ...
        _first_chunk = first_chunk.get() if first_chunk is not None else None
        if _first_chunk:
            x = x[:, :, self.factor_t - 1 :, :, :]
        return x
```
**EN:** This class models `DupUp3D` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `DupUp3D`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 109-154: Class `WanCausalConv3d` / 类 `WanCausalConv3d`
```python
class WanCausalConv3d(nn.Conv3d):
    r"""
    A custom 3D causal convolution layer with feature caching support.

    This layer extends the standard Conv3D layer by ensuring causality in the time dimension and handling feature
    caching for efficient inference.
    """

    def __init__(
        self,
        in_channels: int,
        out_channels: int,
        kernel_size: int | tuple[int, int, int],
        stride: int | tuple[int, int, int] = 1,
# ...
        x = (
            x if current_platform.is_amp_supported() else x.to(self.weight.dtype)
        )  # casting needed if amp isn't supported
        return super().forward(x)
```
**EN:** This class models `WanCausalConv3d` as a specialization of `nn.Conv3d`. A custom 3D causal convolution layer with feature caching support. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanCausalConv3d`，并继承/扩展 `nn.Conv3d`。 文档字符串指出：A custom 3D causal convolution layer with feature caching support. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 157-184: Class `WanRMS_norm` / 类 `WanRMS_norm`
```python
class WanRMS_norm(nn.Module):
    r"""
    A custom RMS normalization layer.
    """

    def __init__(
        self,
        dim: int,
        channel_first: bool = True,
        images: bool = True,
        bias: bool = False,
    ) -> None:
        super().__init__()
        broadcastable_dims = (1, 1, 1) if not images else (1, 1)
# ...
            * self.scale
            * self.gamma
            + self.bias
        )
```
**EN:** This class models `WanRMS_norm` as a specialization of `nn.Module`. A custom RMS normalization layer. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanRMS_norm`，并继承/扩展 `nn.Module`。 文档字符串指出：A custom RMS normalization layer. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 187-193: Class `WanUpsample` / 类 `WanUpsample`
```python
class WanUpsample(nn.Upsample):
    r"""
    Perform upsampling while ensuring the output tensor has the same data type as the input.
    """

    def forward(self, x):
        return super().forward(x.float()).type_as(x)
```
**EN:** This class models `WanUpsample` as a specialization of `nn.Upsample`. Perform upsampling while ensuring the output tensor has the same data type as the input. Important methods include `forward`.
**CN:** 该类实现 `WanUpsample`，并继承/扩展 `nn.Upsample`。 文档字符串指出：Perform upsampling while ensuring the output tensor has the same data type as the input. 其中较重要的方法包括 `forward`。

### Lines 194-200: Top-level configuration / 顶层配置
```python


is_first_frame = None
feat_cache = None
feat_idx = None
cache_t = None
first_chunk = None
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 203-219: Function `bind_context` / 函数 `bind_context`
```python
def bind_context(
    is_first_frame_var,
    feat_cache_var,
    feat_idx_var,
    cache_t_value,
    first_chunk_var,
):
    global is_first_frame
    global feat_cache
    global feat_idx
    global cache_t
    global first_chunk
    is_first_frame = is_first_frame_var
    feat_cache = feat_cache_var
    feat_idx = feat_idx_var
    cache_t = cache_t_value
    first_chunk = first_chunk_var
```
**EN:** This function drives `bind_context` with inputs such as `is_first_frame_var`, `feat_cache_var`, `feat_idx_var`, `cache_t_value`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `bind_context`，主要处理 `is_first_frame_var`, `feat_cache_var`, `feat_idx_var`, `cache_t_value` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 222-230: Function `_ensure_bound` / 函数 `_ensure_bound`
```python
def _ensure_bound():
    if (
        is_first_frame is None
        or feat_cache is None
        or feat_idx is None
        or cache_t is None
        or first_chunk is None
    ):
        raise RuntimeError("common_utils.bind_context() must be called before use.")
```
**EN:** This function drives `_ensure_bound`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_ensure_bound`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 233-312: Function `resample_forward` / 函数 `resample_forward`
```python
def resample_forward(self, x):
    _ensure_bound()
    b, c, t, h, w = x.size()
    first_frame = is_first_frame.get()
    if first_frame:
        assert t == 1
    _feat_cache = feat_cache.get()
    _feat_idx = feat_idx.get()
    if self.mode == "upsample3d":
        if _feat_cache is not None:
            idx = _feat_idx
            if _feat_cache[idx] is None:
                _feat_cache[idx] = "Rep"
                _feat_idx += 1
# ...
            feat_idx.set(_feat_idx)
        elif not first_frame and hasattr(self, "time_conv"):
            x = self.time_conv(x)
    return x
```
**EN:** This function drives `resample_forward` with inputs such as `x`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `resample_forward`，主要处理 `x` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 315-376: Function `residual_block_forward` / 函数 `residual_block_forward`
```python
def residual_block_forward(self, x):
    _ensure_bound()
    # Apply shortcut connection
    h = self.conv_shortcut(x)

    # First normalization and activation
    x = self.norm1(x)
    x = self.nonlinearity(x)

    _feat_cache = feat_cache.get()
    _feat_idx = feat_idx.get()
    if _feat_cache is not None:
        idx = _feat_idx
        cache_x = x[:, :, -cache_t:, :, :].clone()
# ...
        x = self.conv2(x)

    # Add residual connection
    return x + h
```
**EN:** This function drives `residual_block_forward` with inputs such as `x`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `residual_block_forward`，主要处理 `x` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 379-408: Function `attention_block_forward` / 函数 `attention_block_forward`
```python
def attention_block_forward(self, x):
    identity = x
    batch_size, channels, num_frames, height, width = x.size()
    x = x.permute(0, 2, 1, 3, 4).reshape(
        batch_size * num_frames, channels, height, width
    )
    x = self.norm(x)

    # compute query, key, value
    qkv = self.to_qkv(x)
    qkv = qkv.reshape(batch_size * num_frames, 1, channels * 3, -1)
    qkv = qkv.permute(0, 1, 3, 2).contiguous()
    q, k, v = qkv.chunk(3, dim=-1)

# ...
    x = x.view(batch_size, num_frames, channels, height, width)
    x = x.permute(0, 2, 1, 3, 4)

    return x + identity
```
**EN:** This function drives `attention_block_forward` with inputs such as `x`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `attention_block_forward`，主要处理 `x` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 411-422: Function `mid_block_forward` / 函数 `mid_block_forward`
```python
def mid_block_forward(self, x):
    # First residual block
    x = self.resnets[0](x)

    # Process through attention and residual blocks
    for attn, resnet in zip(self.attentions, self.resnets[1:], strict=True):
        if attn is not None:
            x = attn(x)

        x = resnet(x)

    return x
```
**EN:** This function drives `mid_block_forward` with inputs such as `x`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `mid_block_forward`，主要处理 `x` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 425-432: Function `residual_down_block_forward` / 函数 `residual_down_block_forward`
```python
def residual_down_block_forward(self, x):
    x_copy = x
    for resnet in self.resnets:
        x = resnet(x)
    if self.downsampler is not None:
        x = self.downsampler(x)

    return x + self.avg_shortcut(x_copy)
```
**EN:** This function drives `residual_down_block_forward` with inputs such as `x`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `residual_down_block_forward`，主要处理 `x` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 435-448: Function `residual_up_block_forward` / 函数 `residual_up_block_forward`
```python
def residual_up_block_forward(self, x):
    if self.avg_shortcut is not None:
        x_copy = x

    for resnet in self.resnets:
        x = resnet(x)

    if self.upsampler is not None:
        x = self.upsampler(x)

    if self.avg_shortcut is not None:
        x = x + self.avg_shortcut(x_copy)

    return x
```
**EN:** This function drives `residual_up_block_forward` with inputs such as `x`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `residual_up_block_forward`，主要处理 `x` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 451-457: Function `up_block_forward` / 函数 `up_block_forward`
```python
def up_block_forward(self, x):
    for resnet in self.resnets:
        x = resnet(x)

    if self.upsamplers is not None:
        x = self.upsamplers[0](x)
    return x
```
**EN:** This function drives `up_block_forward` with inputs such as `x`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `up_block_forward`，主要处理 `x` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Caching strategy / 缓存策略
- Sampling parameter control / 采样参数控制
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `__future__`, `torch`, `torch.nn`, `torch.nn.functional`
