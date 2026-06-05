# layernorm_gated.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/layernorm_gated.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the main layernorm gated classes used by the SGLang attention stack, together with helper routines that support runtime execution. / 该模块定义 SGLang 注意力栈使用的主要 layernorm gated 类，并提供支撑运行时执行的辅助例程。
## Line-by-Line Analysis / 逐行分析
### Lines 9-25: imports
```python
from functools import lru_cache

import torch
import torch.nn.functional as F
import triton
import triton.language as tl
from einops import rearrange

from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import (
    cdiv,
    cpu_has_amx_support,
    device_context,
    is_cpu,
    is_npu,
    next_power_of_2,
)
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 27-31: module constants
```python
_is_npu = is_npu()
_use_cpu = is_cpu() and cpu_has_amx_support()

# Maximum rows per Triton block for layernorm gated kernel
MAX_ROWS_PER_BLOCK = 4
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 34-64: function rms_norm_ref
```python
def rms_norm_ref(
    x,
    weight,
    bias,
    z=None,
    eps=1e-6,
    group_size=None,
    norm_before_gate=True,
    upcast=True,
):
    dtype = x.dtype
    N = x.shape[-1]
    weight = weight.float()
    bias = bias.float() if bias is not None else None
    if upcast:
        x = x.float()
        z = z.float() if z is not None else z
    if z is not None and not norm_before_gate:
        x = x * F.silu(z)
    if group_size is None:
        rstd = 1 / torch.sqrt((x.square()).mean(dim=-1, keepdim=True) + eps)
        out = (x * rstd * weight) + bias if bias is not None else (x * rstd * weight)
    else:
        x_group = rearrange(x, "... (g d) -> ... g d", d=group_size)
        rstd = 1 / torch.sqrt((x_group.square()).mean(dim=-1, keepdim=True) + eps)
        out = rearrange(x_group * rstd, "... g d -> ... (g d)") * weight
        if bias is not None:
            out = out + bias
    if z is not None and norm_before_gate:
        out *= F.silu(z)
    return out.to(dtype)
```
**EN:** Implements the rms norm ref routine used by this attention module.
**CN:** 实现该注意力模块使用的 rms norm ref 例程。

### Lines 67-169: function _layer_norm_fwd_1pass_kernel
```python
@triton.jit
def _layer_norm_fwd_1pass_kernel(
    X,  # pointer to the input
    Y,  # pointer to the output
    W,  # pointer to the weights
    B,  # pointer to the biases
    Z,  # pointer to the other branch
    Mean,  # pointer to the mean
    Rstd,  # pointer to the 1/std
    stride_x_row,  # how much to increase the pointer when moving by 1 row
    stride_y_row,
    stride_z_row,
    M,  # number of rows in X
    N: tl.constexpr,  # number of columns in X
    eps,  # epsilon to avoid division by zero
    BLOCK_N: tl.constexpr,
    ROWS_PER_BLOCK: tl.constexpr,
    HAS_BIAS: tl.constexpr,
    HAS_Z: tl.constexpr,
    NORM_BEFORE_GATE: tl.constexpr,
    IS_RMS_NORM: tl.constexpr,
    ACTIVATION: tl.constexpr,
):
    # Map the program id to the starting row of X and Y it should compute.
    row_start = tl.program_id(0) * ROWS_PER_BLOCK
    group = tl.program_id(1)

    # Create 2D tile: [ROWS_PER_BLOCK, BLOCK_N]
    rows = row_start + tl.arange(0, ROWS_PER_BLOCK)
    cols = tl.arange(0, BLOCK_N)

    # Compute offsets for 2D tile
# ... omitted 59 lines ...
    y = x_hat * w[None, :] + b[None, :] if HAS_BIAS else x_hat * w[None, :]

    if HAS_Z and NORM_BEFORE_GATE:
        Z_base = Z + rows[:, None] * stride_z_row + col_offsets
        z = tl.load(Z_base, mask=mask, other=0.0).to(tl.float32)
        if ACTIVATION == "swish" or ACTIVATION == "silu":
            y *= z * tl.sigmoid(z)
        elif ACTIVATION == "sigmoid":
            y *= tl.sigmoid(z)

    # Write output
    tl.store(Y_base, y, mask=mask)
```
**EN:** Implements the layer norm fwd 1pass kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 layer norm fwd 1pass kernel 例程。

### Lines 172-179: function _get_sm_count
```python
@lru_cache
def _get_sm_count(device: torch.device) -> int:
    """Get and cache the SM count for a given device."""
    if device.type == "xpu":
        assert torch.xpu.is_available(), "XPU device is not available"
        return torch.xpu.get_device_properties(device).gpu_subslice_count
    props = torch.cuda.get_device_properties(device)
    return props.multi_processor_count
```
**EN:** Implements the get sm count routine used by this attention module.
**CN:** 实现该注意力模块使用的 get sm count 例程。

### Lines 182-194: function calc_rows_per_block
```python
def calc_rows_per_block(M: int, device: torch.device) -> int:
    # When piecewise cuda graph is enabled, use a constant value to avoid
    # torch.compile creating guards on the dynamic batch dimension.
    try:
        if not get_global_server_args().disable_piecewise_cuda_graph:
            return MAX_ROWS_PER_BLOCK
    except ValueError:
        # Global server args not initialized (e.g., in unit tests)
        pass
    sm_count = _get_sm_count(device)
    rows_per_block = next_power_of_2(cdiv(M, 2 * sm_count))
    rows_per_block = min(rows_per_block, MAX_ROWS_PER_BLOCK)
    return rows_per_block
```
**EN:** Implements the calc rows per block routine used by this attention module.
**CN:** 实现该注意力模块使用的 calc rows per block 例程。

### Lines 197-270: function _layer_norm_fwd
```python
def _layer_norm_fwd(
    x,
    weight,
    bias,
    eps,
    z=None,
    out=None,
    group_size=None,
    norm_before_gate=True,
    is_rms_norm=False,
    activation: str = "swish",
):
    M, N = x.shape
    if group_size is None:
        group_size = N
    assert N % group_size == 0
    ngroups = N // group_size
    assert x.stride(-1) == 1
    if z is not None:
        assert z.stride(-1) == 1
        assert z.shape == (M, N)
    assert weight.shape == (N,)
    assert weight.stride(-1) == 1
    if bias is not None:
        assert bias.stride(-1) == 1
        assert bias.shape == (N,)
    # allocate output
    if out is not None:
        assert out.shape == x.shape
    else:
        out = torch.empty_like(x)
    assert out.stride(-1) == 1
# ... omitted 30 lines ...
            group_size,
            eps,
            BLOCK_N=BLOCK_N,
            ROWS_PER_BLOCK=rows_per_block,
            HAS_BIAS=bias is not None,
            HAS_Z=z is not None,
            NORM_BEFORE_GATE=norm_before_gate,
            IS_RMS_NORM=is_rms_norm,
            num_warps=num_warps,
            ACTIVATION=activation,
        )
    return out, mean, rstd
```
**EN:** Implements the layer norm fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 layer norm fwd 例程。

### Lines 273-274: conditional branch
```python
if _is_npu:
    from sgl_kernel_npu.fla.layernorm_gated import layer_norm_fwd_npu as _layer_norm_fwd
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 277-317: function rms_norm_gated
```python
def rms_norm_gated(
    *,
    x,
    weight,
    bias,
    z=None,
    eps=1e-6,
    group_size=None,
    norm_before_gate=True,
    is_rms_norm=False,
    activation: str = "swish",
):
    """If z is not None, we do norm(x) * silu(z) if norm_before_gate, else norm(x * silu(z))"""

    x_shape_og = x.shape
    # reshape input data into 2D tensor
    x = x.reshape(-1, x.shape[-1])
    if x.stride(-1) != 1:
        x = x.contiguous()
    if z is not None:
        assert z.shape == x_shape_og
        z = z.reshape(-1, z.shape[-1])
        if z.stride(-1) != 1:
            z = z.contiguous()
    weight = weight.contiguous()
    if bias is not None:
        bias = bias.contiguous()
    if _is_npu:
        assert activation == "swish", "NPU only supports swish activation"
    y, mean, rstd = _layer_norm_fwd(
        x,
        weight,
        bias,
        eps,
        z=z,
        group_size=group_size,
        norm_before_gate=norm_before_gate,
        is_rms_norm=is_rms_norm,
        activation=activation,
    )
    return y.reshape(x_shape_og)
```
**EN:** Implements the rms norm gated routine used by this attention module.
**CN:** 实现该注意力模块使用的 rms norm gated 例程。

### Lines 320-321: class LayerNormFn
```python
class LayerNormFn(torch.autograd.Function):
```
**EN:** Defines the layer norm fn type and the state it exposes to the rest of the attention stack.
**CN:** 定义 layer norm fn 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 322-345: method LayerNormFn.forward
```python
    @staticmethod
    def forward(
        ctx,
        x,
        weight,
        bias,
        z=None,
        eps=1e-6,
        group_size=None,
        norm_before_gate=True,
        is_rms_norm=False,
        activation: str = "swish",
    ):
        return rms_norm_gated(
            x=x,
            weight=weight,
            bias=bias,
            eps=eps,
            z=z,
            group_size=group_size,
            norm_before_gate=norm_before_gate,
            is_rms_norm=is_rms_norm,
            activation=activation,
        )
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 348-361: function layernorm_fn
```python
def layernorm_fn(
    x,
    weight,
    bias,
    z=None,
    eps=1e-6,
    group_size=None,
    norm_before_gate=True,
    is_rms_norm=False,
    activation: str = "swish",
):
    return LayerNormFn.apply(
        x, weight, bias, z, eps, group_size, norm_before_gate, is_rms_norm, activation
    )
```
**EN:** Implements the layernorm fn routine used by this attention module.
**CN:** 实现该注意力模块使用的 layernorm fn 例程。

### Lines 364-365: class LayerNorm
```python
class LayerNorm(torch.nn.Module):
```
**EN:** Defines the layer norm type and the state it exposes to the rest of the attention stack.
**CN:** 定义 layer norm 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 366-386: method LayerNorm.__init__
```python
    def __init__(
        self,
        hidden_size,
        eps=1e-5,
        group_size=None,
        norm_before_gate=True,
        device=None,
        dtype=None,
    ):
        """If group_size is not None, we do GroupNorm with each group having group_size elements.
        group_size=None is equivalent to group_size=hidden_size (i.e. there's only 1 group).
        """

        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.eps = eps
        self.weight = torch.nn.Parameter(torch.empty(hidden_size, **factory_kwargs))
        self.bias = torch.nn.Parameter(torch.empty(hidden_size, **factory_kwargs))
        self.group_size = group_size
        self.norm_before_gate = norm_before_gate
        self.reset_parameters()
```
**EN:** Initializes the LayerNorm instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 LayerNorm 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 388-390: method LayerNorm.reset_parameters
```python
    def reset_parameters(self):
        torch.nn.init.ones_(self.weight)
        torch.nn.init.zeros_(self.bias)
```
**EN:** Implements the reset parameters routine used by this attention module.
**CN:** 实现该注意力模块使用的 reset parameters 例程。

### Lines 392-403: method LayerNorm.forward
```python
    def forward(self, x, z=None):
        """If z is not None, we do norm(x) * silu(z) if norm_before_gate, else norm(x * silu(z))"""
        return layernorm_fn(
            x,
            self.weight,
            self.bias,
            z=z,
            group_size=self.group_size,
            eps=self.eps,
            norm_before_gate=self.norm_before_gate,
            is_rms_norm=False,
        )
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 406-407: class RMSNorm
```python
class RMSNorm(torch.nn.Module):
```
**EN:** Defines the rmsnorm type and the state it exposes to the rest of the attention stack.
**CN:** 定义 rmsnorm 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 408-429: method RMSNorm.__init__
```python
    def __init__(
        self,
        hidden_size,
        eps=1e-5,
        group_size=None,
        norm_before_gate=True,
        device=None,
        dtype=None,
        activation: str = "swish",
    ):
        """If group_size is not None, we do GroupNorm with each group having group_size elements.
        group_size=None is equivalent to group_size=hidden_size (i.e. there's only 1 group).
        """
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.eps = eps
        self.activation = activation
        self.weight = torch.nn.Parameter(torch.empty(hidden_size, **factory_kwargs))
        self.register_parameter("bias", None)
        self.group_size = group_size
        self.norm_before_gate = norm_before_gate
        self.reset_parameters()
```
**EN:** Initializes the RMSNorm instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 RMSNorm 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 431-432: method RMSNorm.reset_parameters
```python
    def reset_parameters(self):
        torch.nn.init.ones_(self.weight)
```
**EN:** Implements the reset parameters routine used by this attention module.
**CN:** 实现该注意力模块使用的 reset parameters 例程。

### Lines 434-456: method RMSNorm.forward
```python
    def forward(self, x, z=None):
        """If z is not None, we do norm(x) * silu(z) if norm_before_gate, else norm(x * silu(z))"""
        if _use_cpu:
            assert (
                self.norm_before_gate
                and self.group_size is None
                and self.activation == "swish"
            ), "CPU rmsnorm_gated currently only supports norm before gate without group size or activation other than swish"
            return torch.ops.sgl_kernel.fused_rmsnorm_gated_cpu(
                x, self.weight, z, self.eps
            )
        else:
            return layernorm_fn(
                x,
                self.weight,
                self.bias,
                z=z,
                eps=self.eps,
                group_size=self.group_size,
                norm_before_gate=self.norm_before_gate,
                is_rms_norm=True,
                activation=self.activation,
            )
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `functools.lru_cache`
- `torch`
- `torch.nn.functional`
- `triton`
- `triton.language`
- `einops.rearrange`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.cdiv`
- `sglang.srt.utils.cpu_has_amx_support`
- `sglang.srt.utils.device_context`
- `sglang.srt.utils.is_cpu`
- `sglang.srt.utils.is_npu`
- `sglang.srt.utils.next_power_of_2`
- `sgl_kernel_npu.fla.layernorm_gated.layer_norm_fwd_npu`
