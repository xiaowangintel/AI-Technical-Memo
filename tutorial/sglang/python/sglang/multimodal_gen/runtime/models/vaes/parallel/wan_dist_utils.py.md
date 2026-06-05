# wan_dist_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/vaes/parallel/wan_dist_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for wan dist utils within the multimodal runtime. Key symbols include `tensor_pad`, `tensor_chunk`, `split_for_parallel_encode`. / 该模块实现多模态运行时中与 wan dist utils 相关的模型构件。 关键符号包括 `tensor_pad`, `tensor_chunk`, `split_for_parallel_encode`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Imports and module setup / 导入与模块初始化
```python
import math

import torch
import torch.distributed as dist
import torch.nn as nn
import torch.nn.functional as F

from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    get_sp_group,
    get_sp_parallel_rank,
    get_sp_world_size,
)
from sglang.multimodal_gen.runtime.layers.activation import get_act_fn
from sglang.multimodal_gen.runtime.models.vaes.parallel.wan_common_utils import (
# ...
    residual_up_block_forward,
    up_block_forward,
)
from sglang.multimodal_gen.runtime.platforms import current_platform
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 31-45: Function `tensor_pad` / 函数 `tensor_pad`
```python
def tensor_pad(x: torch.Tensor, len_to_pad: int, dim: int = -2):
    x = torch.cat(
        [
            x,
            torch.zeros(
                *x.shape[:dim],
                len_to_pad,
                *x.shape[dim + 1 :],
                dtype=x.dtype,
                device=x.device,
            ),
        ],
        dim=dim,
    )
    return x
```
**EN:** This function drives `tensor_pad` with inputs such as `x`, `len_to_pad`, `dim`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `tensor_pad`，主要处理 `x`, `len_to_pad`, `dim` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 48-58: Function `tensor_chunk` / 函数 `tensor_chunk`
```python
def tensor_chunk(x: torch.Tensor, dim: int = -2, world_size: int = 1, rank: int = 0):
    if x is None:
        return None
    if world_size <= 1:
        return x
    len_to_padding = (int(math.ceil(x.shape[dim] / world_size)) * world_size) - x.shape[
        dim
    ]
    if len_to_padding != 0:
        x = tensor_pad(x, len_to_padding, dim=dim)
    return torch.chunk(x, world_size, dim=dim)[rank]
```
**EN:** This function drives `tensor_chunk` with inputs such as `x`, `dim`, `world_size`, `rank`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `tensor_chunk`，主要处理 `x`, `dim`, `world_size`, `rank` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 61-72: Function `split_for_parallel_encode` / 函数 `split_for_parallel_encode`
```python
def split_for_parallel_encode(
    x: torch.Tensor, downsample_count: int, world_size: int, rank: int
):
    orig_height = x.shape[-2]
    expected_height = orig_height // (2**downsample_count)
    factor = world_size * (2**downsample_count)
    pad_h = (factor - orig_height % factor) % factor
    if pad_h:
        x = F.pad(x, (0, 0, 0, pad_h, 0, 0))
    expected_local_height = (orig_height + pad_h) // (2**downsample_count) // world_size
    x = tensor_chunk(x, dim=-2, world_size=world_size, rank=rank)
    return x, expected_height, expected_local_height
```
**EN:** This function drives `split_for_parallel_encode` with inputs such as `x`, `downsample_count`, `world_size`, `rank`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `split_for_parallel_encode`，主要处理 `x`, `downsample_count`, `world_size`, `rank` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 75-83: Function `ensure_local_height` / 函数 `ensure_local_height`
```python
def ensure_local_height(x: torch.Tensor, expected_local_height: int | None):
    if expected_local_height is None:
        return x
    if x.shape[-2] < expected_local_height:
        pad = expected_local_height - x.shape[-2]
        return F.pad(x, (0, 0, 0, pad, 0, 0))
    if x.shape[-2] > expected_local_height:
        return x[..., :expected_local_height, :].contiguous()
    return x
```
**EN:** This function drives `ensure_local_height` with inputs such as `x`, `expected_local_height`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `ensure_local_height`，主要处理 `x`, `expected_local_height` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 86-91: Function `split_for_parallel_decode` / 函数 `split_for_parallel_decode`
```python
def split_for_parallel_decode(
    x: torch.Tensor, upsample_count: int, world_size: int, rank: int
):
    expected_height = x.shape[-2] * (2**upsample_count)
    x = tensor_chunk(x, dim=-2, world_size=world_size, rank=rank)
    return x, expected_height
```
**EN:** This function drives `split_for_parallel_decode` with inputs such as `x`, `upsample_count`, `world_size`, `rank`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `split_for_parallel_decode`，主要处理 `x`, `upsample_count`, `world_size`, `rank` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 94-100: Function `gather_and_trim_height` / 函数 `gather_and_trim_height`
```python
def gather_and_trim_height(x: torch.Tensor, expected_height: int | None):
    if expected_height is None:
        return x
    x = get_sp_group().all_gather(x, dim=-2)
    if x.shape[-2] != expected_height:
        x = x[..., :expected_height, :].contiguous()
    return x
```
**EN:** This function drives `gather_and_trim_height` with inputs such as `x`, `expected_height`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `gather_and_trim_height`，主要处理 `x`, `expected_height` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 103-113: Function `_ensure_recv_buf` / 函数 `_ensure_recv_buf`
```python
def _ensure_recv_buf(
    recv_buf: torch.Tensor | None, reference: torch.Tensor
) -> torch.Tensor:
    if (
        recv_buf is None
        or recv_buf.shape != reference.shape
        or recv_buf.dtype != reference.dtype
        or recv_buf.device != reference.device
    ):
        return torch.empty_like(reference)
    return recv_buf
```
**EN:** This function drives `_ensure_recv_buf` with inputs such as `recv_buf`, `reference`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_ensure_recv_buf`，主要处理 `recv_buf`, `reference` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 116-165: Function `halo_exchange` / 函数 `halo_exchange`
```python
def halo_exchange(
    x: torch.Tensor,
    height_halo_size: int = 1,
    recv_top_buf: torch.Tensor | None = None,
    recv_bottom_buf: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    if height_halo_size == 0:
        return x, recv_top_buf, recv_bottom_buf

    sp_group = get_sp_group()
    rank = get_sp_parallel_rank()
    world_size = get_sp_world_size()
    group = sp_group.device_group
    group_ranks = sp_group.ranks
# ...
        torch.concat([recv_top_buf, x, recv_bottom_buf], dim=-2),
        recv_top_buf,
        recv_bottom_buf,
    )
```
**EN:** This function drives `halo_exchange` with inputs such as `x`, `height_halo_size`, `recv_top_buf`, `recv_bottom_buf`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `halo_exchange`，主要处理 `x`, `height_halo_size`, `recv_top_buf`, `recv_bottom_buf` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 168-247: Class `WanDistConv2d` / 类 `WanDistConv2d`
```python
class WanDistConv2d(nn.Conv2d):
    def __init__(
        self,
        in_channels: int,
        out_channels: int,
        kernel_size: int | tuple[int, int, int],
        stride: int | tuple[int, int, int] = 1,
        padding: int | tuple[int, int, int] = 0,
        height_padding: tuple[int, int] | None = None,
    ):
        super().__init__(
            in_channels=in_channels,
            out_channels=out_channels,
            kernel_size=kernel_size,
# ...
        if start != 0 or end != out.shape[-2]:
            out = out[..., start:end, :]

        return out
```
**EN:** This class models `WanDistConv2d` as a specialization of `nn.Conv2d`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDistConv2d`，并继承/扩展 `nn.Conv2d`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 250-346: Class `WanDistCausalConv3d` / 类 `WanDistCausalConv3d`
```python
class WanDistCausalConv3d(nn.Conv3d):
    def __init__(
        self,
        in_channels: int,
        out_channels: int,
        kernel_size: int | tuple[int, int, int],
        stride: int | tuple[int, int, int] = 1,
        padding: int | tuple[int, int, int] = 0,
    ):
        super().__init__(
            in_channels=in_channels,
            out_channels=out_channels,
            kernel_size=kernel_size,
            stride=stride,
# ...
        if start != 0 or end != out.shape[-2]:
            out = out[..., start:end, :]

        return out
```
**EN:** This class models `WanDistCausalConv3d` as a specialization of `nn.Conv3d`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDistCausalConv3d`，并继承/扩展 `nn.Conv3d`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 349-365: Class `WanDistZeroPad2d` / 类 `WanDistZeroPad2d`
```python
class WanDistZeroPad2d(nn.Module):
    """Apply 2D padding once globally across sequence-parallel height splits."""

    def __init__(self, padding: tuple[int, int, int, int]) -> None:
        super().__init__()
        self.padding = padding  # (left, right, top, bottom)
        self.rank = get_sp_parallel_rank()
        self.world_size = get_sp_world_size()

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        left, right, top, bottom = self.padding
        if self.world_size <= 1:
            return F.pad(x, (left, right, top, bottom))
        # Only the first/last rank should contribute global top/bottom padding.
        top = top if self.rank == 0 else 0
        bottom = bottom if self.rank == self.world_size - 1 else 0
        return F.pad(x, (left, right, top, bottom))
```
**EN:** This class models `WanDistZeroPad2d` as a specialization of `nn.Module`. Apply 2D padding once globally across sequence-parallel height splits. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDistZeroPad2d`，并继承/扩展 `nn.Module`。 文档字符串指出：Apply 2D padding once globally across sequence-parallel height splits. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 368-423: Class `WanDistResample` / 类 `WanDistResample`
```python
class WanDistResample(nn.Module):
    r"""
    A custom resampling module for 2D and 3D data used for parallel decoding.

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
**EN:** This class models `WanDistResample` as a specialization of `nn.Module`. A custom resampling module for 2D and 3D data used for parallel decoding. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDistResample`，并继承/扩展 `nn.Module`。 文档字符串指出：A custom resampling module for 2D and 3D data used for parallel decoding. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 426-462: Class `WanDistResidualBlock` / 类 `WanDistResidualBlock`
```python
class WanDistResidualBlock(nn.Module):
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
**EN:** This class models `WanDistResidualBlock` as a specialization of `nn.Module`. A custom residual block module. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDistResidualBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A custom residual block module. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 465-493: Class `WanDistAttentionBlock` / 类 `WanDistAttentionBlock`
```python
class WanDistAttentionBlock(nn.Module):
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
# ...
        if self.world_size > 1:
            x = torch.chunk(x, self.world_size, dim=-2)[self.rank]

        return x
```
**EN:** This class models `WanDistAttentionBlock` as a specialization of `nn.Module`. Causal self-attention with a single head. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDistAttentionBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：Causal self-attention with a single head. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 496-528: Class `WanDistMidBlock` / 类 `WanDistMidBlock`
```python
class WanDistMidBlock(nn.Module):
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
**EN:** This class models `WanDistMidBlock` as a specialization of `nn.Module`. Middle block for WanVAE encoder and decoder. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDistMidBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：Middle block for WanVAE encoder and decoder. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 531-566: Class `WanDistResidualDownBlock` / 类 `WanDistResidualDownBlock`
```python
class WanDistResidualDownBlock(nn.Module):
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
        self.avg_shortcut = AvgDown3D(
# ...
            self.downsampler = None

    def forward(self, x):
        return residual_down_block_forward(self, x)
```
**EN:** This class models `WanDistResidualDownBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDistResidualDownBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 569-629: Class `WanDistResidualUpBlock` / 类 `WanDistResidualUpBlock`
```python
class WanDistResidualUpBlock(nn.Module):
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
**EN:** This class models `WanDistResidualUpBlock` as a specialization of `nn.Module`. A block that handles upsampling for the WanVAE decoder. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDistResidualUpBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A block that handles upsampling for the WanVAE decoder. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 632-680: Class `WanDistUpBlock` / 类 `WanDistUpBlock`
```python
class WanDistUpBlock(nn.Module):
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
**EN:** This class models `WanDistUpBlock` as a specialization of `nn.Module`. A block that handles upsampling for the WanVAE decoder. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `WanDistUpBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：A block that handles upsampling for the WanVAE decoder. 其中较重要的方法包括 `__init__`, `forward`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行
- Sampling parameter control / 采样参数控制

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.activation`, `sglang.multimodal_gen.runtime.models.vaes.parallel.wan_common_utils`, `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `torch`, `torch.distributed`, `torch.nn`, `torch.nn.functional`
- **Stdlib / 标准库**: `math`
