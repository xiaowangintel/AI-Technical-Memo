# conv.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/conv.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements linear projections and GEMM backend integration with convolutional layer support for the SGLang SRT runtime. It exposes symbols such as `_tuplify`, `_check_enable_linear`, `_reverse_repeat_tuple`, and `_compute_same_padding_for_pad` and connects them to backend-specific paths such as `CUDA`. / 该模块为 SGLang 的 SRT 运行时实现了线性投影与 GEMM 后端集成，并结合卷积层支持。它提供了 `_tuplify`、`_check_enable_linear`、`_reverse_repeat_tuple` 以及 `_compute_same_padding_for_pad` 等符号，并把这些符号连接到 `CUDA` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports, constants, and runtime setup
```python
"""
Conv2d/Conv3d layers with unfold+linear optimization for patch embeddings.

When kernel_size == stride, padding == 0, dilation == 1, groups == 1, the conv
is equivalent to unfold + F.linear, which is significantly faster on CUDA and
also avoids the PyTorch 2.9.1 + CuDNN < 9.15 Conv3d bug
(https://github.com/pytorch/pytorch/issues/168167).
"""

import math
from typing import Tuple, Union

import torch
import torch.nn as nn
import torch.nn.functional as F

from sglang.srt.layers.utils.multi_platform import MultiPlatformOp

_VALID_PADDING_STRINGS = {"same", "valid"}
_VALID_PADDING_MODES = {"zeros", "reflect", "replicate", "circular"}
```
**EN:** This section prepares the module namespace. It imports `math`, `typing.Tuple`, `typing.Union`, `torch`, `torch.nn`, and `torch.nn.functional`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_VALID_PADDING_STRINGS` and `_VALID_PADDING_MODES` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `math`、`typing.Tuple`、`typing.Union`、`torch`、`torch.nn` 以及 `torch.nn.functional`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_VALID_PADDING_STRINGS` 和 `_VALID_PADDING_MODES` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 23-29: Internal helper `_tuplify`
```python
def _tuplify(val, n: int) -> tuple:
    if isinstance(val, (list, tuple)):
        assert len(val) == n
        return tuple(val)
    return (val,) * n
```
**EN:** This block defines `_tuplify` and contains the main logic for this step. It mainly invokes `isinstance`, `tuple`, and `len`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_tuplify`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance`、`tuple` 以及 `len`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 30-45: Internal helper `_check_enable_linear`
```python
def _check_enable_linear(
    kernel_size: tuple,
    stride: tuple,
    padding: tuple,
    dilation: tuple,
    groups: int,
) -> bool:
    """Check if conv can be replaced with unfold + F.linear."""
    return (
        kernel_size == stride
        and all(p == 0 for p in padding)
        and all(d == 1 for d in dilation)
        and groups == 1
    )
```
**EN:** This block defines `_check_enable_linear` and contains the main logic for this step. It mainly invokes `all`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_check_enable_linear`，并承载这一阶段的核心逻辑。 它主要调用 `all`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 46-50: Internal helper `_reverse_repeat_tuple`
```python
def _reverse_repeat_tuple(t: tuple) -> tuple:
    """(1, 2, 3) -> (3, 3, 2, 2, 1, 1). Used for F.pad with non-zeros padding_mode."""
    return tuple(x for x in reversed(t) for _ in range(2))
```
**EN:** This block defines `_reverse_repeat_tuple` and contains the main logic for this step. It mainly invokes `tuple`, `reversed`, and `range`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_reverse_repeat_tuple`，并承载这一阶段的核心逻辑。 它主要调用 `tuple`、`reversed` 以及 `range`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 51-64: Internal helper `_compute_same_padding_for_pad`
```python
def _compute_same_padding_for_pad(kernel_size: tuple, dilation: tuple) -> tuple:
    """Compute _reversed_padding_repeated_twice for padding='same'.

    This mirrors PyTorch's nn.Conv*d behavior: pre-compute the exact pad
    amounts so that F.pad can be called before F.conv*d(padding=0).
    """
    pad = []
    for k, d in zip(reversed(kernel_size), reversed(dilation)):
        total = d * (k - 1)
        pad.append(total // 2)
        pad.append(total - total // 2)
    return tuple(pad)
```
**EN:** This block defines `_compute_same_padding_for_pad` and contains the main logic for this step. It mainly invokes `zip`, `tuple`, `reversed`, and `pad.append`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pad` and `total` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_compute_same_padding_for_pad`，并承载这一阶段的核心逻辑。 它主要调用 `zip`、`tuple`、`reversed` 以及 `pad.append`，说明该流程会编排底层辅助函数或计算内核。 像 `pad` 和 `total` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 65-93: Internal helper `_validate_conv_args`
```python
def _validate_conv_args(
    in_channels: int,
    out_channels: int,
    groups: int,
    padding,
    padding_mode: str,
    stride: tuple,
) -> None:
    if in_channels % groups != 0:
        raise ValueError(
            f"in_channels ({in_channels}) must be divisible by groups ({groups})"
        )
    if out_channels % groups != 0:
        raise ValueError(
            f"out_channels ({out_channels}) must be divisible by groups ({groups})"
        )
    if padding_mode not in _VALID_PADDING_MODES:
        raise ValueError(
            f"padding_mode must be one of {_VALID_PADDING_MODES}, got '{padding_mode}'"
        )
    if isinstance(padding, str):
        if padding not in _VALID_PADDING_STRINGS:
            raise ValueError(
                f"padding must be one of {_VALID_PADDING_STRINGS}, got '{padding}'"
            )
        if padding == "same" and any(s != 1 for s in stride):
            raise ValueError("padding='same' is not supported for strided convolutions")
```
**EN:** This block defines `_validate_conv_args` and contains the main logic for this step. It mainly invokes `isinstance`, `ValueError`, and `any`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_validate_conv_args`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance`、`ValueError` 以及 `any`，说明该流程会编排底层辅助函数或计算内核。

### Lines 94-96: Class `Conv2dLayer` declaration and shared state
```python
class Conv2dLayer(MultiPlatformOp):
    """Drop-in replacement for nn.Conv2d. Linear optimization disabled by default."""
```
**EN:** This block introduces class `Conv2dLayer` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Drop-in replacement for nn.Conv2d.
**CN:** 该代码块引入类 `Conv2dLayer`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 97-152: `Conv2dLayer` initialization and state setup
```python
    def __init__(
        self,
        in_channels: int,
        out_channels: int,
        kernel_size: Union[int, Tuple[int, int]],
        stride: Union[int, Tuple[int, int]] = 1,
        padding: Union[int, Tuple[int, int], str] = 0,
        dilation: Union[int, Tuple[int, int]] = 1,
        groups: int = 1,
        bias: bool = True,
        padding_mode: str = "zeros",
        disable_linear: bool = True,
    ):
        super().__init__()
        self.in_channels = in_channels
        self.out_channels = out_channels
        self.kernel_size = _tuplify(kernel_size, 2)
        self.stride = _tuplify(stride, 2)
        self.dilation = _tuplify(dilation, 2)
        self.groups = groups
        self.padding_mode = padding_mode

        _validate_conv_args(
            in_channels, out_channels, groups, padding, padding_mode, self.stride
        )

        if isinstance(padding, str):
            self.padding = (0, 0) if padding == "valid" else padding
        else:
            self.padding = _tuplify(padding, 2)

        # Pre-compute pad tuple for padding_mode != "zeros" (mirrors nn.Conv2d).
        # When padding="same", we need numeric values for F.pad;
        # when padding is already numeric, _reverse_repeat_tuple handles it.
        if isinstance(self.padding, str):
            self._reversed_padding_repeated_twice = _compute_same_padding_for_pad(
                self.kernel_size, self.dilation
            )
        else:
            self._reversed_padding_repeated_twice = _reverse_repeat_tuple(self.padding)

        padding_tuple = self.padding if isinstance(self.padding, tuple) else (1, 1)
        self.enable_linear = not disable_linear and _check_enable_linear(
            self.kernel_size, self.stride, padding_tuple, self.dilation, groups
        )

        self.weight = nn.Parameter(
            torch.empty(out_channels, in_channels // groups, *self.kernel_size)
        )
        if bias:
            self.bias = nn.Parameter(torch.empty(out_channels))
        else:
            self.register_parameter("bias", None)

        self._reset_parameters()
```
**EN:** This block defines `Conv2dLayer.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `_tuplify`, `_validate_conv_args`, `isinstance`, and `nn.Parameter`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.in_channels`, `self.out_channels`, `self.kernel_size`, `self.stride`, and `self.dilation` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Conv2dLayer.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`_tuplify`、`_validate_conv_args`、`isinstance` 以及 `nn.Parameter`，说明该流程会编排底层辅助函数或计算内核。 像 `self.in_channels`、`self.out_channels`、`self.kernel_size`、`self.stride` 以及 `self.dilation` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 153-159: Internal helper `Conv2dLayer._reset_parameters`
```python
    def _reset_parameters(self):
        nn.init.kaiming_uniform_(self.weight, a=math.sqrt(5))
        if self.bias is not None:
            fan_in = nn.init._calculate_correct_fan(self.weight, "fan_in")
            bound = 1 / math.sqrt(fan_in) if fan_in > 0 else 0
            nn.init.uniform_(self.bias, -bound, bound)
```
**EN:** This block defines `Conv2dLayer._reset_parameters` and contains the main logic for this step. It mainly invokes `nn.init.kaiming_uniform_`, `nn.init._calculate_correct_fan`, `nn.init.uniform_`, and `math.sqrt`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `fan_in` and `bound` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Conv2dLayer._reset_parameters`，并承载这一阶段的核心逻辑。 它主要调用 `nn.init.kaiming_uniform_`、`nn.init._calculate_correct_fan`、`nn.init.uniform_` 以及 `math.sqrt`，说明该流程会编排底层辅助函数或计算内核。 像 `fan_in` 和 `bound` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 160-167: Internal helper `Conv2dLayer._forward_mulmat`
```python
    def _forward_mulmat(self, x: torch.Tensor) -> torch.Tensor:
        K1, K2 = self.kernel_size
        x = x.unfold(2, K1, K1).unfold(3, K2, K2)
        N, _, Hp, Wp = x.shape[:4]
        x = x.permute(0, 2, 3, 1, 4, 5).reshape(N, Hp, Wp, -1)
        x = F.linear(x, self.weight.reshape(self.out_channels, -1), self.bias)
        return x.permute(0, 3, 1, 2)
```
**EN:** This block defines `Conv2dLayer._forward_mulmat` and contains the main logic for this step. It mainly invokes `x.unfold.unfold`, `x.permute.reshape`, `F.linear`, `x.permute`, and `self.weight.reshape`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `K1`, `K2`, `x`, `N`, and `_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Conv2dLayer._forward_mulmat`，并承载这一阶段的核心逻辑。 它主要调用 `x.unfold.unfold`、`x.permute.reshape`、`F.linear`、`x.permute` 以及 `self.weight.reshape`，说明该流程会编排底层辅助函数或计算内核。 像 `K1`、`K2`、`x`、`N` 以及 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 168-188: Internal helper `Conv2dLayer._forward_conv`
```python
    def _forward_conv(self, x: torch.Tensor) -> torch.Tensor:
        if self.padding_mode != "zeros":
            return F.conv2d(
                F.pad(x, self._reversed_padding_repeated_twice, mode=self.padding_mode),
                self.weight,
                self.bias,
                self.stride,
                (0, 0),
                self.dilation,
                self.groups,
            )
        return F.conv2d(
            x,
            self.weight,
            self.bias,
            self.stride,
            self.padding,
            self.dilation,
            self.groups,
        )
```
**EN:** This block defines `Conv2dLayer._forward_conv` and contains the main logic for this step. It mainly invokes `F.conv2d` and `F.pad`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Conv2dLayer._forward_conv`，并承载这一阶段的核心逻辑。 它主要调用 `F.conv2d` 和 `F.pad`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 189-193: `Conv2dLayer.forward_native` NATIVE execution path
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        if self.enable_linear:
            return self._forward_mulmat(x)
        return self._forward_conv(x)
```
**EN:** This block defines `Conv2dLayer.forward_native` and contains the main logic for this step. It mainly invokes `self._forward_conv` and `self._forward_mulmat`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Conv2dLayer.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `self._forward_conv` 和 `self._forward_mulmat`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 194-199: `Conv2dLayer.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, x: torch.Tensor) -> torch.Tensor:
        if self.enable_linear:
            return self._forward_mulmat(x)
        return self._forward_conv(x)
```
**EN:** This block defines `Conv2dLayer.forward_cuda` and contains the main logic for this step. It mainly invokes `self._forward_conv` and `self._forward_mulmat`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Conv2dLayer.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self._forward_conv` 和 `self._forward_mulmat`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 200-202: Class `Conv3dLayer` declaration and shared state
```python
class Conv3dLayer(MultiPlatformOp):
    """Drop-in replacement for nn.Conv3d with automatic linear optimization."""
```
**EN:** This block introduces class `Conv3dLayer` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Drop-in replacement for nn.Conv3d with automatic linear optimization.
**CN:** 该代码块引入类 `Conv3dLayer`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 203-255: `Conv3dLayer` initialization and state setup
```python
    def __init__(
        self,
        in_channels: int,
        out_channels: int,
        kernel_size: Union[int, Tuple[int, int, int]],
        stride: Union[int, Tuple[int, int, int]] = 1,
        padding: Union[int, Tuple[int, int, int], str] = 0,
        dilation: Union[int, Tuple[int, int, int]] = 1,
        groups: int = 1,
        bias: bool = True,
        padding_mode: str = "zeros",
        disable_linear: bool = False,
    ):
        super().__init__()
        self.in_channels = in_channels
        self.out_channels = out_channels
        self.kernel_size = _tuplify(kernel_size, 3)
        self.stride = _tuplify(stride, 3)
        self.dilation = _tuplify(dilation, 3)
        self.groups = groups
        self.padding_mode = padding_mode

        _validate_conv_args(
            in_channels, out_channels, groups, padding, padding_mode, self.stride
        )

        if isinstance(padding, str):
            self.padding = (0, 0, 0) if padding == "valid" else padding
        else:
            self.padding = _tuplify(padding, 3)

        if isinstance(self.padding, str):
            self._reversed_padding_repeated_twice = _compute_same_padding_for_pad(
                self.kernel_size, self.dilation
            )
        else:
            self._reversed_padding_repeated_twice = _reverse_repeat_tuple(self.padding)

        padding_tuple = self.padding if isinstance(self.padding, tuple) else (1, 1, 1)
        self.enable_linear = not disable_linear and _check_enable_linear(
            self.kernel_size, self.stride, padding_tuple, self.dilation, groups
        )

        self.weight = nn.Parameter(
            torch.empty(out_channels, in_channels // groups, *self.kernel_size)
        )
        if bias:
            self.bias = nn.Parameter(torch.empty(out_channels))
        else:
            self.register_parameter("bias", None)

        self._reset_parameters()
```
**EN:** This block defines `Conv3dLayer.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `_tuplify`, `_validate_conv_args`, `isinstance`, and `nn.Parameter`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.in_channels`, `self.out_channels`, `self.kernel_size`, `self.stride`, and `self.dilation` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Conv3dLayer.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`_tuplify`、`_validate_conv_args`、`isinstance` 以及 `nn.Parameter`，说明该流程会编排底层辅助函数或计算内核。 像 `self.in_channels`、`self.out_channels`、`self.kernel_size`、`self.stride` 以及 `self.dilation` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 256-262: Internal helper `Conv3dLayer._reset_parameters`
```python
    def _reset_parameters(self):
        nn.init.kaiming_uniform_(self.weight, a=math.sqrt(5))
        if self.bias is not None:
            fan_in = nn.init._calculate_correct_fan(self.weight, "fan_in")
            bound = 1 / math.sqrt(fan_in) if fan_in > 0 else 0
            nn.init.uniform_(self.bias, -bound, bound)
```
**EN:** This block defines `Conv3dLayer._reset_parameters` and contains the main logic for this step. It mainly invokes `nn.init.kaiming_uniform_`, `nn.init._calculate_correct_fan`, `nn.init.uniform_`, and `math.sqrt`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `fan_in` and `bound` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Conv3dLayer._reset_parameters`，并承载这一阶段的核心逻辑。 它主要调用 `nn.init.kaiming_uniform_`、`nn.init._calculate_correct_fan`、`nn.init.uniform_` 以及 `math.sqrt`，说明该流程会编排底层辅助函数或计算内核。 像 `fan_in` 和 `bound` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 263-270: Internal helper `Conv3dLayer._forward_mulmat`
```python
    def _forward_mulmat(self, x: torch.Tensor) -> torch.Tensor:
        K1, K2, K3 = self.kernel_size
        x = x.unfold(2, K1, K1).unfold(3, K2, K2).unfold(4, K3, K3)
        N, Dp, Hp, Wp = x.shape[0], x.shape[2], x.shape[3], x.shape[4]
        x = x.permute(0, 2, 3, 4, 1, 5, 6, 7).reshape(N, Dp, Hp, Wp, -1)
        x = F.linear(x, self.weight.reshape(self.out_channels, -1), self.bias)
        return x.permute(0, 4, 1, 2, 3)
```
**EN:** This block defines `Conv3dLayer._forward_mulmat` and contains the main logic for this step. It mainly invokes `x.unfold.unfold.unfold`, `x.permute.reshape`, `F.linear`, `x.permute`, and `self.weight.reshape`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `K1`, `K2`, `K3`, `x`, and `N` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Conv3dLayer._forward_mulmat`，并承载这一阶段的核心逻辑。 它主要调用 `x.unfold.unfold.unfold`、`x.permute.reshape`、`F.linear`、`x.permute` 以及 `self.weight.reshape`，说明该流程会编排底层辅助函数或计算内核。 像 `K1`、`K2`、`K3`、`x` 以及 `N` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 271-291: Internal helper `Conv3dLayer._forward_conv`
```python
    def _forward_conv(self, x: torch.Tensor) -> torch.Tensor:
        if self.padding_mode != "zeros":
            return F.conv3d(
                F.pad(x, self._reversed_padding_repeated_twice, mode=self.padding_mode),
                self.weight,
                self.bias,
                self.stride,
                (0, 0, 0),
                self.dilation,
                self.groups,
            )
        return F.conv3d(
            x,
            self.weight,
            self.bias,
            self.stride,
            self.padding,
            self.dilation,
            self.groups,
        )
```
**EN:** This block defines `Conv3dLayer._forward_conv` and contains the main logic for this step. It mainly invokes `F.conv3d` and `F.pad`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Conv3dLayer._forward_conv`，并承载这一阶段的核心逻辑。 它主要调用 `F.conv3d` 和 `F.pad`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 292-296: `Conv3dLayer.forward_native` NATIVE execution path
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        if self.enable_linear:
            return self._forward_mulmat(x)
        return self._forward_conv(x)
```
**EN:** This block defines `Conv3dLayer.forward_native` and contains the main logic for this step. It mainly invokes `self._forward_conv` and `self._forward_mulmat`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Conv3dLayer.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `self._forward_conv` 和 `self._forward_mulmat`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 297-300: `Conv3dLayer.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, x: torch.Tensor) -> torch.Tensor:
        if self.enable_linear:
            return self._forward_mulmat(x)
        return self._forward_conv(x)
```
**EN:** This block defines `Conv3dLayer.forward_cuda` and contains the main logic for this step. It mainly invokes `self._forward_conv` and `self._forward_mulmat`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Conv3dLayer.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self._forward_conv` 和 `self._forward_mulmat`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_tuplify`, `_check_enable_linear`, `_reverse_repeat_tuple`, `_compute_same_padding_for_pad`, and `_validate_conv_args`. / **主要符号**：核心入口包括 `_tuplify`、`_check_enable_linear`、`_reverse_repeat_tuple`、`_compute_same_padding_for_pad` 以及 `_validate_conv_args`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。
- **Embedding pipeline**: Describes how IDs or features are mapped into model-space tensors. / **嵌入流水线**：说明 ID 或特征如何映射到模型空间张量。

## Dependencies / 依赖关系
- **Standard library**: `math`, `typing.Tuple`, and `typing.Union` / **标准库**：`math`、`typing.Tuple` 以及 `typing.Union`
- **Third-party**: `torch`, `torch.nn`, and `torch.nn.functional` / **第三方依赖**：`torch`、`torch.nn` 以及 `torch.nn.functional`
- **Internal SGLang modules**: `sglang.srt.layers.utils.multi_platform.MultiPlatformOp` / **SGLang 内部模块**：`sglang.srt.layers.utils.multi_platform.MultiPlatformOp`
