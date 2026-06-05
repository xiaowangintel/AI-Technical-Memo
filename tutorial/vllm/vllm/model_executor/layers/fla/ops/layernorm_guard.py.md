# layernorm_guard.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/layernorm_guard.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops; normalization layers and fused normalization kernels / 快速线性注意力自定义内核与工具算子；归一化层与融合归一化内核

## Line-by-Line Analysis / 逐行分析
### Lines 16-25 — imports and setup
```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange

from vllm.triton_utils import tl, triton
from vllm.utils.math_utils import cdiv, next_power_of_2
from vllm.utils.platform_utils import num_compute_units

from .utils import input_guard
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 28-57 — function `rms_norm_ref`
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
**EN:** This function defines `rms_norm_ref`. It provides one of the file's main runtime building blocks. The main inputs are `x`, `weight`, `bias`, `z`, `eps`, `group_size`. Key calls include `weight.float`, `out.to`, `bias.float`, `x.float`, `rearrange`, `F.silu`. It writes or updates `dtype`, `weight`, `bias`, `x`, `z`, `rstd`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `rms_norm_ref`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `x`, `weight`, `bias`, `z`, `eps`, `group_size`。 关键调用包括 `weight.float`, `out.to`, `bias.float`, `x.float`, `rearrange`, `F.silu`。 它会写入或更新 `dtype`, `weight`, `bias`, `x`, `z`, `rstd`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 60-168 — function `layer_norm_fwd_kernel`
```python
@triton.heuristics(
    {
        "HAS_BIAS": lambda args: args["B"] is not None,
        "HAS_Z": lambda args: args["Z"] is not None,
    }
)
@triton.jit
def layer_norm_fwd_kernel(
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
    row_offsets = rows[:, None] * stride_x_row
    col_offsets = cols[None, :] + group * N

    # Base pointers
    X_base = X + row_offsets + col_offsets
    Y_base = Y + rows[:, None] * stride_y_row + col_offsets

    # Create mask for valid rows and columns
    row_mask = rows[:, None] < M
    col_mask = cols[None, :] < N
    mask = row_mask & col_mask

    # Load input data with 2D tile
    x = tl.load(X_base, mask=mask, other=0.0).to(tl.float32)

    if HAS_Z and not NORM_BEFORE_GATE:
        Z_base = Z + rows[:, None] * stride_z_row + col_offsets
        z = tl.load(Z_base, mask=mask, other=0.0).to(tl.float32)
# ... omitted for brevity ...
    # Write output
    tl.store(Y_base, y, mask=mask)
```
**EN:** This function defines `layer_norm_fwd_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `X`, `Y`, `W`, `B`, `Z`, `Mean`. Key calls include `triton.heuristics`, `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.rsqrt`, `tl.store`. It writes or updates `row_start`, `group`, `rows`, `cols`, `row_offsets`, `col_offsets`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `layer_norm_fwd_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `X`, `Y`, `W`, `B`, `Z`, `Mean`。 关键调用包括 `triton.heuristics`, `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.rsqrt`, `tl.store`。 它会写入或更新 `row_start`, `group`, `rows`, `cols`, `row_offsets`, `col_offsets`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 171-175 — function `calc_rows_per_block`
```python
def calc_rows_per_block(M: int, device: torch.device) -> int:
    sm_count = num_compute_units(device.index)
    rows_per_block = next_power_of_2(cdiv(M, 2 * sm_count))
    rows_per_block = min(rows_per_block, 4)
    return rows_per_block
```
**EN:** This function defines `calc_rows_per_block`. It provides one of the file's main runtime building blocks. The main inputs are `M`, `device`. Key calls include `num_compute_units`, `next_power_of_2`, `min`, `cdiv`. It writes or updates `sm_count`, `rows_per_block`.
**CN:** 该函数定义 `calc_rows_per_block`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `M`, `device`。 关键调用包括 `num_compute_units`, `next_power_of_2`, `min`, `cdiv`。 它会写入或更新 `sm_count`, `rows_per_block`。

### Lines 178-250 — function `layer_norm_fwd`
```python
def layer_norm_fwd(
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    eps: float,
    z: torch.Tensor = None,
    out: torch.Tensor = None,
    group_size: int = None,
    norm_before_gate: bool = True,
    is_rms_norm: bool = False,
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
    mean = (
        torch.empty((ngroups * M,), dtype=torch.float32, device=x.device)
        if not is_rms_norm
        else None
    )
    rstd = torch.empty((ngroups * M,), dtype=torch.float32, device=x.device)
    # Less than 64KB per feature: enqueue fused kernel
    MAX_FUSED_SIZE = 65536 // x.element_size()
    BLOCK_N = min(MAX_FUSED_SIZE, triton.next_power_of_2(group_size))
    if group_size > BLOCK_N:
        raise RuntimeError("This layer norm doesn't support feature dim >= 64KB.")
    # heuristics for number of warps
# ... omitted for brevity ...
    )
    return out, mean, rstd
```
**EN:** This function defines `layer_norm_fwd`. It provides one of the file's main runtime building blocks. The main inputs are `x`, `weight`, `bias`, `eps`, `z`, `out`. Key calls include `torch.empty`, `min`, `calc_rows_per_block`, `layer_norm_fwd_kernel`, `x.stride`, `weight.stride`. It writes or updates `M`, `N`, `ngroups`, `mean`, `rstd`, `MAX_FUSED_SIZE`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `layer_norm_fwd`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `x`, `weight`, `bias`, `eps`, `z`, `out`。 关键调用包括 `torch.empty`, `min`, `calc_rows_per_block`, `layer_norm_fwd_kernel`, `x.stride`, `weight.stride`。 它会写入或更新 `M`, `N`, `ngroups`, `mean`, `rstd`, `MAX_FUSED_SIZE`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 253-298 — function `_layer_norm_fn_impl`
```python
def _layer_norm_fn_impl(
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
    """Triton layer/RMS norm with optional gating.

    If z is not None, computes norm(x) * silu(z) when norm_before_gate,
    else norm(x * silu(z)).

    This calls the triton kernel directly. The original code wrapped this
    in a torch.autograd.Function (LayerNormFn) to save tensors for a
    backward pass, but vLLM is inference-only so there is no backward pass.
    The autograd wrapper also prevented torch.compile/dynamo from tracing
    through the function due to its @staticmethod forward.
    """
    x_shape_og = x.shape
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
    y, _, _ = layer_norm_fwd(
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
**EN:** This function defines `_layer_norm_fn_impl`. Triton layer/RMS norm with optional gating. The main inputs are `x`, `weight`, `bias`, `z`, `eps`, `group_size`. Key calls include `x.reshape`, `weight.contiguous`, `layer_norm_fwd`, `y.reshape`, `x.stride`, `x.contiguous`. It writes or updates `x_shape_og`, `x`, `weight`, `y`, `_`, `z`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_layer_norm_fn_impl`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `x`, `weight`, `bias`, `z`, `eps`, `group_size`。 关键调用包括 `x.reshape`, `weight.contiguous`, `layer_norm_fwd`, `y.reshape`, `x.stride`, `x.contiguous`。 它会写入或更新 `x_shape_og`, `x`, `weight`, `y`, `_`, `z`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 301-315 — function `layernorm_fn`
```python
@input_guard
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
    return _layer_norm_fn_impl(
        x, weight, bias, z, eps, group_size, norm_before_gate, is_rms_norm, activation
    )
```
**EN:** This function defines `layernorm_fn`. It provides one of the file's main runtime building blocks. The main inputs are `x`, `weight`, `bias`, `z`, `eps`, `group_size`. Key calls include `_layer_norm_fn_impl`.
**CN:** 该函数定义 `layernorm_fn`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `x`, `weight`, `bias`, `z`, `eps`, `group_size`。 关键调用包括 `_layer_norm_fn_impl`。

### Lines 318-331 — function `rmsnorm_fn`
```python
@input_guard
def rmsnorm_fn(
    x,
    weight,
    bias,
    z=None,
    eps=1e-6,
    group_size=None,
    norm_before_gate=True,
    activation: str = "swish",
):
    return _layer_norm_fn_impl(
        x, weight, bias, z, eps, group_size, norm_before_gate, True, activation
    )
```
**EN:** This function defines `rmsnorm_fn`. It provides one of the file's main runtime building blocks. The main inputs are `x`, `weight`, `bias`, `z`, `eps`, `group_size`. Key calls include `_layer_norm_fn_impl`.
**CN:** 该函数定义 `rmsnorm_fn`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `x`, `weight`, `bias`, `z`, `eps`, `group_size`。 关键调用包括 `_layer_norm_fn_impl`。

### Lines 334-371 — class `LayerNormGated`
```python
class LayerNormGated(nn.Module):
    def __init__(
        self,
        hidden_size,
        eps: float = 1e-5,
        group_size: int | None = None,
        norm_before_gate: bool = True,
        device: torch.device | None = None,
        dtype: torch.dtype | None = None,
    ):
        """If group_size is not None, we do GroupNorm with each group having group_size elements.
        group_size=None is equivalent to group_size=hidden_size (i.e. there's only 1 group).
        """

# ... omitted for brevity ...
            norm_before_gate=self.norm_before_gate,
        )
```
**EN:** This class defines `LayerNormGated`. It inherits from `nn.Module`. It provides one of the file's main runtime building blocks. Important methods include `__init__`, `forward`, `reset_parameters`. Key calls include `super.__init__`, `nn.Parameter`, `self.reset_parameters`, `torch.nn.init.ones_`, `torch.nn.init.zeros_`, `layernorm_fn`. It writes or updates `factory_kwargs`, `eps`, `weight`, `bias`, `group_size`, `norm_before_gate`.
**CN:** 该类定义了 `LayerNormGated`。 它继承自 `nn.Module`。 它是该文件中的一个主要运行时构件。 重要方法包括 `__init__`, `forward`, `reset_parameters`。 关键调用包括 `super.__init__`, `nn.Parameter`, `self.reset_parameters`, `torch.nn.init.ones_`, `torch.nn.init.zeros_`, `layernorm_fn`。 它会写入或更新 `factory_kwargs`, `eps`, `weight`, `bias`, `group_size`, `norm_before_gate`。

### Lines 335-355 — method `LayerNormGated.__init__`
```python
    def __init__(
        self,
        hidden_size,
        eps: float = 1e-5,
        group_size: int | None = None,
        norm_before_gate: bool = True,
        device: torch.device | None = None,
        dtype: torch.dtype | None = None,
    ):
        """If group_size is not None, we do GroupNorm with each group having group_size elements.
        group_size=None is equivalent to group_size=hidden_size (i.e. there's only 1 group).
        """

        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.eps = eps
        self.weight = nn.Parameter(torch.empty(hidden_size, **factory_kwargs))
        self.bias = nn.Parameter(torch.empty(hidden_size, **factory_kwargs))
        self.group_size = group_size
        self.norm_before_gate = norm_before_gate
        self.reset_parameters()
```
**EN:** This method defines `__init__`. If group_size is not None, we do GroupNorm with each group having group_size elements. The main inputs are `hidden_size`, `eps`, `group_size`, `norm_before_gate`, `device`, `dtype`. Key calls include `super.__init__`, `nn.Parameter`, `self.reset_parameters`, `torch.empty`, `super`. It writes or updates `factory_kwargs`, `eps`, `weight`, `bias`, `group_size`, `norm_before_gate`.
**CN:** 该方法定义 `__init__`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_size`, `eps`, `group_size`, `norm_before_gate`, `device`, `dtype`。 关键调用包括 `super.__init__`, `nn.Parameter`, `self.reset_parameters`, `torch.empty`, `super`。 它会写入或更新 `factory_kwargs`, `eps`, `weight`, `bias`, `group_size`, `norm_before_gate`。

### Lines 361-371 — method `LayerNormGated.forward`
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
        )
```
**EN:** This method defines `forward`. If z is not None, we do norm(x) * silu(z) if norm_before_gate, else norm(x * silu(z)) The main inputs are `x`, `z`. Key calls include `layernorm_fn`.
**CN:** 该方法定义 `forward`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `x`, `z`。 关键调用包括 `layernorm_fn`。

### Lines 357-359 — method `LayerNormGated.reset_parameters`
```python
    def reset_parameters(self):
        torch.nn.init.ones_(self.weight)
        torch.nn.init.zeros_(self.bias)
```
**EN:** This method defines `reset_parameters`. It provides one of the file's main runtime building blocks. Key calls include `torch.nn.init.ones_`, `torch.nn.init.zeros_`.
**CN:** 该方法定义 `reset_parameters`。 它是该文件中的一个主要运行时构件。 关键调用包括 `torch.nn.init.ones_`, `torch.nn.init.zeros_`。

### Lines 374-412 — class `RMSNormGated`
```python
class RMSNormGated(nn.Module):
    def __init__(
        self,
        hidden_size,
        eps: float = 1e-5,
        group_size: int | None = None,
        norm_before_gate: bool = False,
        device: torch.device | None = None,
        dtype: torch.dtype | None = None,
        activation: str = "swish",
    ):
        """If group_size is not None, we do GroupNorm with each group having group_size elements.
        group_size=None is equivalent to group_size=hidden_size (i.e. there's only 1 group).
        """
# ... omitted for brevity ...
            activation=self.activation,
        )
```
**EN:** This class defines `RMSNormGated`. It inherits from `nn.Module`. It provides one of the file's main runtime building blocks. Important methods include `__init__`, `forward`, `reset_parameters`. Key calls include `super.__init__`, `nn.Parameter`, `self.register_parameter`, `self.reset_parameters`, `torch.nn.init.ones_`, `rmsnorm_fn`. It writes or updates `factory_kwargs`, `eps`, `activation`, `weight`, `group_size`, `norm_before_gate`.
**CN:** 该类定义了 `RMSNormGated`。 它继承自 `nn.Module`。 它是该文件中的一个主要运行时构件。 重要方法包括 `__init__`, `forward`, `reset_parameters`。 关键调用包括 `super.__init__`, `nn.Parameter`, `self.register_parameter`, `self.reset_parameters`, `torch.nn.init.ones_`, `rmsnorm_fn`。 它会写入或更新 `factory_kwargs`, `eps`, `activation`, `weight`, `group_size`, `norm_before_gate`。

### Lines 375-396 — method `RMSNormGated.__init__`
```python
    def __init__(
        self,
        hidden_size,
        eps: float = 1e-5,
        group_size: int | None = None,
        norm_before_gate: bool = False,
        device: torch.device | None = None,
        dtype: torch.dtype | None = None,
        activation: str = "swish",
    ):
        """If group_size is not None, we do GroupNorm with each group having group_size elements.
        group_size=None is equivalent to group_size=hidden_size (i.e. there's only 1 group).
        """
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.eps = eps
        self.activation = activation
        self.weight = nn.Parameter(torch.empty(hidden_size, **factory_kwargs))
        self.register_parameter("bias", None)
        self.group_size = group_size
        self.norm_before_gate = norm_before_gate
        self.reset_parameters()
```
**EN:** This method defines `__init__`. If group_size is not None, we do GroupNorm with each group having group_size elements. The main inputs are `hidden_size`, `eps`, `group_size`, `norm_before_gate`, `device`, `dtype`. Key calls include `super.__init__`, `nn.Parameter`, `self.register_parameter`, `self.reset_parameters`, `torch.empty`, `super`. It writes or updates `factory_kwargs`, `eps`, `activation`, `weight`, `group_size`, `norm_before_gate`.
**CN:** 该方法定义 `__init__`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_size`, `eps`, `group_size`, `norm_before_gate`, `device`, `dtype`。 关键调用包括 `super.__init__`, `nn.Parameter`, `self.register_parameter`, `self.reset_parameters`, `torch.empty`, `super`。 它会写入或更新 `factory_kwargs`, `eps`, `activation`, `weight`, `group_size`, `norm_before_gate`。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] Normalization layers and fused normalization kernels / [CN] 归一化层与融合归一化内核
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `rms_norm_ref`, `layer_norm_fwd_kernel`, `calc_rows_per_block`, `layer_norm_fwd` / [CN] 核心符号：`rms_norm_ref`, `layer_norm_fwd_kernel`, `calc_rows_per_block`, `layer_norm_fwd`

## Dependencies / 依赖关系
- **External**: `torch`, `torch.nn`, `torch.nn.functional`, `einops`, `utils` / **外部依赖**: `torch`, `torch.nn`, `torch.nn.functional`, `einops`, `utils`
- **Internal**: `vllm.triton_utils`, `vllm.utils.math_utils`, `vllm.utils.platform_utils` / **内部依赖**: `vllm.triton_utils`, `vllm.utils.math_utils`, `vllm.utils.platform_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
