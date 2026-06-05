# kda.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/kda.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops; KDA-related kernels and layer integration / 快速线性注意力自定义内核与工具算子；KDA 相关内核与层集成

## Line-by-Line Analysis / 逐行分析
### Lines 12-29 — imports and setup
```python
import torch
import torch.nn as nn

from vllm.model_executor.custom_op import CustomOp
from vllm.triton_utils import tl, triton
from vllm.utils.math_utils import cdiv, next_power_of_2

from .chunk_delta_h import chunk_gated_delta_rule_fwd_h
from .cumsum import chunk_local_cumsum
from .fused_recurrent import fused_recurrent_gated_delta_rule_fwd_kernel
from .index import prepare_chunk_indices
from .l2norm import l2norm_fwd
from .op import exp, log
from .solve_tril import solve_tril
from .utils import FLA_CHUNK_SIZE, is_amd

BT_LIST_AUTOTUNE = [32, 64, 128]
NUM_WARPS_AUTOTUNE = [2, 4, 8, 16] if is_amd else [4, 8, 16, 32]
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. It writes or updates `BT_LIST_AUTOTUNE`, `NUM_WARPS_AUTOTUNE`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 它会写入或更新 `BT_LIST_AUTOTUNE`, `NUM_WARPS_AUTOTUNE`。

### Lines 32-106 — function `fused_recurrent_kda_fwd`
```python
def fused_recurrent_kda_fwd(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    scale: float,
    initial_state: torch.Tensor,
    inplace_final_state: bool = True,
    cu_seqlens: torch.Tensor | None = None,
    ssm_state_indices: torch.Tensor | None = None,
    num_accepted_tokens: torch.Tensor | None = None,
    use_qk_l2norm_in_kernel: bool = False,
) -> tuple[torch.Tensor, torch.Tensor]:
    B, T, H, K, V = *k.shape, v.shape[-1]
    HV = v.shape[2]
    N = B if cu_seqlens is None else len(cu_seqlens) - 1
    BK, BV = next_power_of_2(K), min(next_power_of_2(V), 8)
    NK, NV = cdiv(K, BK), cdiv(V, BV)
    assert NK == 1, "NK > 1 is not supported yet"
    num_stages = 3
    num_warps = 1

    o = torch.empty_like(k)
    if inplace_final_state:
        final_state = initial_state
    else:
        final_state = q.new_empty(T, HV, V, K, dtype=initial_state.dtype)

    stride_init_state_token = initial_state.stride(0)
    stride_final_state_token = final_state.stride(0)

    if ssm_state_indices is None:
        stride_indices_seq, stride_indices_tok = 1, 1
    elif ssm_state_indices.ndim == 1:
        stride_indices_seq, stride_indices_tok = ssm_state_indices.stride(0), 1
    else:
        stride_indices_seq, stride_indices_tok = ssm_state_indices.stride()

    grid = (NK, NV, N * HV)
    fused_recurrent_gated_delta_rule_fwd_kernel[grid](
        q=q,
        k=k,
        v=v,
# ... omitted for brevity ...

    return o, final_state
```
**EN:** This function defines `fused_recurrent_kda_fwd`. It provides one of the file's main runtime building blocks. The main inputs are `q`, `k`, `v`, `g`, `beta`, `scale`. Key calls include `torch.empty_like`, `initial_state.stride`, `final_state.stride`, `fused_recurrent_gated_delta_rule_fwd_kernel`, `next_power_of_2`, `min`. It writes or updates `B`, `T`, `H`, `K`, `V`, `HV`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_recurrent_kda_fwd`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `q`, `k`, `v`, `g`, `beta`, `scale`。 关键调用包括 `torch.empty_like`, `initial_state.stride`, `final_state.stride`, `fused_recurrent_gated_delta_rule_fwd_kernel`, `next_power_of_2`, `min`。 它会写入或更新 `B`, `T`, `H`, `K`, `V`, `HV`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 109-145 — function `fused_recurrent_kda`
```python
def fused_recurrent_kda(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor = None,
    scale: float = None,
    initial_state: torch.Tensor = None,
    inplace_final_state: bool = True,
    use_qk_l2norm_in_kernel: bool = True,
    cu_seqlens: torch.Tensor | None = None,
    ssm_state_indices: torch.LongTensor | None = None,
    **kwargs,
) -> tuple[torch.Tensor, torch.Tensor]:
    if cu_seqlens is not None and q.shape[0] != 1:
        raise ValueError(
            f"The batch size is expected to be 1 rather than {q.shape[0]} when using `cu_seqlens`."
            f"Please flatten variable-length inputs before processing."
        )
    if scale is None:
        scale = k.shape[-1] ** -0.5

    o, final_state = fused_recurrent_kda_fwd(
        q=q.contiguous(),
        k=k.contiguous(),
        v=v.contiguous(),
        g=g.contiguous(),
        beta=beta.contiguous(),
        scale=scale,
        initial_state=initial_state,
        inplace_final_state=inplace_final_state,
        cu_seqlens=cu_seqlens,
        ssm_state_indices=ssm_state_indices,
        num_accepted_tokens=None,
        use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
    )
    return o, final_state
```
**EN:** This function defines `fused_recurrent_kda`. It provides one of the file's main runtime building blocks. The main inputs are `q`, `k`, `v`, `g`, `beta`, `scale`. Key calls include `fused_recurrent_kda_fwd`, `ValueError`, `q.contiguous`, `k.contiguous`, `v.contiguous`, `g.contiguous`. It writes or updates `o`, `final_state`, `scale`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_recurrent_kda`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `q`, `k`, `v`, `g`, `beta`, `scale`。 关键调用包括 `fused_recurrent_kda_fwd`, `ValueError`, `q.contiguous`, `k.contiguous`, `v.contiguous`, `g.contiguous`。 它会写入或更新 `o`, `final_state`, `scale`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 148-233 — function `layer_norm_gated_fwd_kernel`
```python
@triton.heuristics(
    {
        "STORE_RESIDUAL_OUT": lambda args: args["residual_out"] is not None,
        "HAS_RESIDUAL": lambda args: args["residual"] is not None,
        "HAS_WEIGHT": lambda args: args["w"] is not None,
        "HAS_BIAS": lambda args: args["b"] is not None,
    }
)
@triton.jit
def layer_norm_gated_fwd_kernel(
    x,  # pointer to the input
    g,  # pointer to the gate
    y,  # pointer to the output
    w,  # pointer to the weights
    b,  # pointer to the biases
    residual,  # pointer to the residual
    residual_out,  # pointer to the residual
    mean,  # pointer to the mean
    rstd,  # pointer to the 1/std
    eps,  # epsilon to avoid division by zero
    T,  # number of rows in x
    D: tl.constexpr,  # number of columns in x
    BT: tl.constexpr,
    BD: tl.constexpr,
    ACTIVATION: tl.constexpr,
    IS_RMS_NORM: tl.constexpr,
    STORE_RESIDUAL_OUT: tl.constexpr,
    HAS_RESIDUAL: tl.constexpr,
    HAS_WEIGHT: tl.constexpr,
    HAS_BIAS: tl.constexpr,
):
    i_t = tl.program_id(0)

    o_d = tl.arange(0, BD)
    m_d = o_d < D

    p_x = tl.make_block_ptr(x, (T, D), (D, 1), (i_t * BT, 0), (BT, BD), (1, 0))
    b_x = tl.load(p_x, boundary_check=(0, 1)).to(tl.float32)
    if HAS_RESIDUAL:
        p_res = tl.make_block_ptr(
            residual, (T, D), (D, 1), (i_t * BT, 0), (BT, BD), (1, 0)
        )
        b_x += tl.load(p_res, boundary_check=(0, 1)).to(tl.float32)
    if STORE_RESIDUAL_OUT:
        p_res_out = tl.make_block_ptr(
            residual_out, (T, D), (D, 1), (i_t * BT, 0), (BT, BD), (1, 0)
        )
        tl.store(p_res_out, b_x.to(p_res_out.dtype.element_ty), boundary_check=(0, 1))
    if not IS_RMS_NORM:
        b_mean = tl.sum(b_x, axis=1) / D
        p_mean = tl.make_block_ptr(mean, (T,), (1,), (i_t * BT,), (BT,), (0,))
        tl.store(p_mean, b_mean.to(p_mean.dtype.element_ty), boundary_check=(0,))
        b_xbar = tl.where(m_d[None, :], b_x - b_mean[:, None], 0.0)
        b_var = tl.sum(b_xbar * b_xbar, axis=1) / D
    else:
        b_xbar = tl.where(m_d[None, :], b_x, 0.0)
# ... omitted for brevity ...
    p_y = tl.make_block_ptr(y, (T, D), (D, 1), (i_t * BT, 0), (BT, BD), (1, 0))
    tl.store(p_y, b_y.to(p_y.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** This function defines `layer_norm_gated_fwd_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `x`, `g`, `y`, `w`, `b`, `residual`. Key calls include `triton.heuristics`, `tl.program_id`, `tl.arange`, `tl.make_block_ptr`, `tl.load.to`, `tl.store`. It writes or updates `i_t`, `o_d`, `m_d`, `p_x`, `b_x`, `b_rstd`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `layer_norm_gated_fwd_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `x`, `g`, `y`, `w`, `b`, `residual`。 关键调用包括 `triton.heuristics`, `tl.program_id`, `tl.arange`, `tl.make_block_ptr`, `tl.load.to`, `tl.store`。 它会写入或更新 `i_t`, `o_d`, `m_d`, `p_x`, `b_x`, `b_rstd`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 236-309 — function `layer_norm_gated_fwd_kernel1`
```python
@triton.heuristics(
    {
        "STORE_RESIDUAL_OUT": lambda args: args["residual_out"] is not None,
        "HAS_RESIDUAL": lambda args: args["residual"] is not None,
        "HAS_WEIGHT": lambda args: args["w"] is not None,
        "HAS_BIAS": lambda args: args["b"] is not None,
    }
)
@triton.jit
def layer_norm_gated_fwd_kernel1(
    x,  # pointer to the input
    g,  # pointer to the gate
    y,  # pointer to the output
    w,  # pointer to the weights
    b,  # pointer to the biases
    residual,  # pointer to the residual
    residual_out,  # pointer to the residual
    mean,  # pointer to the mean
    rstd,  # pointer to the 1/std
    eps,  # epsilon to avoid division by zero
    D: tl.constexpr,  # number of columns in x
    BD: tl.constexpr,
    ACTIVATION: tl.constexpr,
    IS_RMS_NORM: tl.constexpr,
    STORE_RESIDUAL_OUT: tl.constexpr,
    HAS_RESIDUAL: tl.constexpr,
    HAS_WEIGHT: tl.constexpr,
    HAS_BIAS: tl.constexpr,
):
    i_t = tl.program_id(0)
    x += i_t * D
    y += i_t * D
    g += i_t * D
    if HAS_RESIDUAL:
        residual += i_t * D
    if STORE_RESIDUAL_OUT:
        residual_out += i_t * D

    o_d = tl.arange(0, BD)
    m_d = o_d < D
    b_x = tl.load(x + o_d, mask=m_d, other=0.0).to(tl.float32)
    if HAS_RESIDUAL:
        b_x += tl.load(residual + o_d, mask=m_d, other=0.0).to(tl.float32)
    if STORE_RESIDUAL_OUT:
        tl.store(residual_out + o_d, b_x, mask=m_d)
    if not IS_RMS_NORM:
        b_mean = tl.sum(b_x, axis=0) / D
        tl.store(mean + i_t, b_mean)
        b_xbar = tl.where(m_d, b_x - b_mean, 0.0)
        b_var = tl.sum(b_xbar * b_xbar, axis=0) / D
    else:
        b_xbar = tl.where(m_d, b_x, 0.0)
        b_var = tl.sum(b_xbar * b_xbar, axis=0) / D
    b_rstd = 1 / tl.sqrt(b_var + eps)
    tl.store(rstd + i_t, b_rstd)

# ... omitted for brevity ...
    # Write output
    tl.store(y + o_d, b_y, mask=m_d)
```
**EN:** This function defines `layer_norm_gated_fwd_kernel1`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `x`, `g`, `y`, `w`, `b`, `residual`. Key calls include `triton.heuristics`, `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.store`, `tl.where`. It writes or updates `i_t`, `x`, `y`, `g`, `o_d`, `m_d`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `layer_norm_gated_fwd_kernel1`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `x`, `g`, `y`, `w`, `b`, `residual`。 关键调用包括 `triton.heuristics`, `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.store`, `tl.where`。 它会写入或更新 `i_t`, `x`, `y`, `g`, `o_d`, `m_d`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 312-394 — function `layer_norm_gated_fwd`
```python
def layer_norm_gated_fwd(
    x: torch.Tensor,
    g: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    activation: str = "swish",
    eps: float = 1e-5,
    residual: torch.Tensor = None,
    out_dtype: torch.dtype = None,
    residual_dtype: torch.dtype = None,
    is_rms_norm: bool = False,
):
    if residual is not None:
        residual_dtype = residual.dtype
    T, D = x.shape
    if residual is not None:
        assert residual.shape == (T, D)
    if weight is not None:
        assert weight.shape == (D,)
    if bias is not None:
        assert bias.shape == (D,)
    # allocate output
    y = x if out_dtype is None else torch.empty_like(x, dtype=out_dtype)
    if residual is not None or (
        residual_dtype is not None and residual_dtype != x.dtype
    ):
        residual_out = torch.empty(T, D, device=x.device, dtype=residual_dtype)
    else:
        residual_out = None
    mean = (
        torch.empty((T,), dtype=torch.float, device=x.device)
        if not is_rms_norm
        else None
    )
    rstd = torch.empty((T,), dtype=torch.float, device=x.device)
    # Less than 64KB per feature: enqueue fused kernel
    MAX_FUSED_SIZE = 65536 // x.element_size()
    BD = min(MAX_FUSED_SIZE, next_power_of_2(D))
    if D > BD:
        raise RuntimeError("This layer norm doesn't support feature dim >= 64KB.")
    # heuristics for number of warps

    if D <= 512:
        BT = 32
# ... omitted for brevity ...
    # residual_out is None if residual is None and residual_dtype == input_dtype
    return y, mean, rstd, residual_out if residual_out is not None else x
```
**EN:** This function defines `layer_norm_gated_fwd`. It provides one of the file's main runtime building blocks. The main inputs are `x`, `g`, `weight`, `bias`, `activation`, `eps`. Key calls include `torch.empty`, `min`, `torch.empty_like`, `x.element_size`, `next_power_of_2`, `RuntimeError`. It writes or updates `T`, `D`, `y`, `mean`, `rstd`, `MAX_FUSED_SIZE`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `layer_norm_gated_fwd`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `x`, `g`, `weight`, `bias`, `activation`, `eps`。 关键调用包括 `torch.empty`, `min`, `torch.empty_like`, `x.element_size`, `next_power_of_2`, `RuntimeError`。 它会写入或更新 `T`, `D`, `y`, `mean`, `rstd`, `MAX_FUSED_SIZE`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 397-432 — function `rms_norm_gated`
```python
def rms_norm_gated(
    x: torch.Tensor,
    g: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    activation: str = "swish",
    residual: torch.Tensor | None = None,
    prenorm: bool = False,
    residual_in_fp32: bool = False,
    eps: float = 1e-6,
):
    x_shape_og = x.shape
    # reshape input data into 2D tensor
    x = x.contiguous().reshape(-1, x.shape[-1])
    g = g.contiguous().reshape(-1, g.shape[-1])
    if residual is not None:
        assert residual.shape == x_shape_og
        residual = residual.contiguous().reshape(-1, residual.shape[-1])
    residual_dtype = (
        residual.dtype
        if residual is not None
        else (torch.float if residual_in_fp32 else None)
    )
    y, _, _, residual_out = layer_norm_gated_fwd(
        x=x,
        g=g,
        weight=weight,
        bias=bias,
        activation=activation,
        eps=eps,
        residual=residual,
        residual_dtype=residual_dtype,
        is_rms_norm=True,
    )
    y = y.reshape(x_shape_og)
    return y if not prenorm else (y, residual_out.reshape(x_shape_og))
```
**EN:** This function defines `rms_norm_gated`. It provides one of the file's main runtime building blocks. The main inputs are `x`, `g`, `weight`, `bias`, `activation`, `residual`. Key calls include `x.contiguous.reshape`, `g.contiguous.reshape`, `layer_norm_gated_fwd`, `y.reshape`, `residual.contiguous.reshape`, `x.contiguous`. It writes or updates `x_shape_og`, `x`, `g`, `residual_dtype`, `y`, `_`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `rms_norm_gated`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `x`, `g`, `weight`, `bias`, `activation`, `residual`。 关键调用包括 `x.contiguous.reshape`, `g.contiguous.reshape`, `layer_norm_gated_fwd`, `y.reshape`, `residual.contiguous.reshape`, `x.contiguous`。 它会写入或更新 `x_shape_og`, `x`, `g`, `residual_dtype`, `y`, `_`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 435-507 — class `FusedRMSNormGated`
```python
@CustomOp.register("fused_rms_norm_gated")
class FusedRMSNormGated(CustomOp):
    def __init__(
        self,
        hidden_size: int,
        elementwise_affine: bool = True,
        eps: float = 1e-5,
        activation: str = "swish",
        device: torch.device | None = None,
        dtype: torch.dtype | None = None,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()

# ... omitted for brevity ...
            residual_in_fp32=residual_in_fp32,
        )
```
**EN:** This class defines `FusedRMSNormGated`. It inherits from `CustomOp`. It provides one of the file's main runtime building blocks. Important methods include `__init__`, `forward_native`, `forward_cuda`. Key calls include `CustomOp.register`, `super.__init__`, `self.register_parameter`, `x.float`, `x_float.pow.mean`, `g.float`. It writes or updates `factory_kwargs`, `hidden_size`, `elementwise_affine`, `eps`, `activation`, `x_float`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `FusedRMSNormGated`。 它继承自 `CustomOp`。 它是该文件中的一个主要运行时构件。 重要方法包括 `__init__`, `forward_native`, `forward_cuda`。 关键调用包括 `CustomOp.register`, `super.__init__`, `self.register_parameter`, `x.float`, `x_float.pow.mean`, `g.float`。 它会写入或更新 `factory_kwargs`, `hidden_size`, `elementwise_affine`, `eps`, `activation`, `x_float`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 437-461 — method `FusedRMSNormGated.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        elementwise_affine: bool = True,
        eps: float = 1e-5,
        activation: str = "swish",
        device: torch.device | None = None,
        dtype: torch.dtype | None = None,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()

        self.hidden_size = hidden_size
        self.elementwise_affine = elementwise_affine
        self.eps = eps
        self.activation = activation

        if self.activation not in ["swish", "silu", "sigmoid"]:
            raise ValueError(f"Unsupported activation: {self.activation}")

        if elementwise_affine:
            self.weight = nn.Parameter(torch.empty(hidden_size, **factory_kwargs))
        else:
            self.register_parameter("weight", None)
        self.register_parameter("bias", None)
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `hidden_size`, `elementwise_affine`, `eps`, `activation`, `device`, `dtype`. Key calls include `super.__init__`, `self.register_parameter`, `ValueError`, `nn.Parameter`, `super`, `torch.empty`. It writes or updates `factory_kwargs`, `hidden_size`, `elementwise_affine`, `eps`, `activation`, `weight`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `hidden_size`, `elementwise_affine`, `eps`, `activation`, `device`, `dtype`。 关键调用包括 `super.__init__`, `self.register_parameter`, `ValueError`, `nn.Parameter`, `super`, `torch.empty`。 它会写入或更新 `factory_kwargs`, `hidden_size`, `elementwise_affine`, `eps`, `activation`, `weight`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 463-487 — method `FusedRMSNormGated.forward_native`
```python
    def forward_native(
        self,
        x: torch.Tensor,
        g: torch.Tensor,
        residual: torch.Tensor | None = None,
        prenorm: bool = False,
        residual_in_fp32: bool = False,
    ) -> torch.Tensor:
        """Decomposed PyTorch ops for torch.compile/inductor fusion."""
        # TODO(https://github.com/vllm-project/vllm/issues/36175): implement
        # native residual/prenorm path and unify with RMSNormGated.
        # For now, fall back to the triton kernel.
        if residual is not None or prenorm:
            return self.forward_cuda(x, g, residual, prenorm, residual_in_fp32)
        x_float = x.float()
        variance = x_float.pow(2).mean(dim=-1, keepdim=True)
        x_normed = x_float * torch.rsqrt(variance + self.eps)
        if self.weight is not None:
            x_normed = x_normed * self.weight.float()
        g_float = g.float()
        if self.activation in ("swish", "silu"):
            out = x_normed * g_float * torch.sigmoid(g_float)
        else:  # sigmoid
            out = x_normed * torch.sigmoid(g_float)
        return out.to(x.dtype)
```
**EN:** This method defines `forward_native`. Decomposed PyTorch ops for torch.compile/inductor fusion. The main inputs are `x`, `g`, `residual`, `prenorm`, `residual_in_fp32`. Key calls include `x.float`, `x_float.pow.mean`, `g.float`, `out.to`, `self.forward_cuda`, `torch.rsqrt`. It writes or updates `x_float`, `variance`, `x_normed`, `g_float`, `out`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward_native`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `x`, `g`, `residual`, `prenorm`, `residual_in_fp32`。 关键调用包括 `x.float`, `x_float.pow.mean`, `g.float`, `out.to`, `self.forward_cuda`, `torch.rsqrt`。 它会写入或更新 `x_float`, `variance`, `x_normed`, `g_float`, `out`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 489-507 — method `FusedRMSNormGated.forward_cuda`
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        g: torch.Tensor,
        residual: torch.Tensor | None = None,
        prenorm: bool = False,
        residual_in_fp32: bool = False,
    ) -> torch.Tensor:
        return rms_norm_gated(
            x,
            g,
            self.weight,
            self.bias,
            self.activation,
            residual=residual,
            eps=self.eps,
            prenorm=prenorm,
            residual_in_fp32=residual_in_fp32,
        )
```
**EN:** This method defines `forward_cuda`. It executes the main forward/runtime path for this component. The main inputs are `x`, `g`, `residual`, `prenorm`, `residual_in_fp32`. Key calls include `rms_norm_gated`.
**CN:** 该方法定义 `forward_cuda`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `x`, `g`, `residual`, `prenorm`, `residual_in_fp32`。 关键调用包括 `rms_norm_gated`。

### Lines 510-618 — function `chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_inter`
```python
@triton.heuristics({"IS_VARLEN": lambda args: args["cu_seqlens"] is not None})
@triton.autotune(
    configs=[
        triton.Config({"BK": BK}, num_warps=num_warps, num_stages=num_stages)
        for BK in [32, 64]
        for num_warps in [1, 2, 4, 8]
        for num_stages in [2, 3, 4]
    ],
    key=["BC"],
)
@triton.jit(do_not_specialize=["T"])
def chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_inter(
    q,
    k,
    g,
    beta,
    A,
    Aqk,
    scale,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    K: tl.constexpr,
    BT: tl.constexpr,
    BC: tl.constexpr,
    BK: tl.constexpr,
    NC: tl.constexpr,
    IS_VARLEN: tl.constexpr,
):
    i_t, i_c, i_bh = tl.program_id(0), tl.program_id(1), tl.program_id(2)
    i_b, i_h = i_bh // H, i_bh % H
    i_i, i_j = i_c // NC, i_c % NC
    if IS_VARLEN:
        i_n, i_t = (
            tl.load(chunk_indices + i_t * 2).to(tl.int32),
            tl.load(chunk_indices + i_t * 2 + 1).to(tl.int32),
        )
        bos, eos = (
            tl.load(cu_seqlens + i_n).to(tl.int32),
            tl.load(cu_seqlens + i_n + 1).to(tl.int32),
        )
        T = eos - bos
    else:
        bos, eos = i_b * T, i_b * T + T

    if i_t * BT + i_i * BC >= T:
        return
    if i_i <= i_j:
        return

    q += (bos * H + i_h) * K
    k += (bos * H + i_h) * K
    g += (bos * H + i_h) * K
    A += (bos * H + i_h) * BT
    Aqk += (bos * H + i_h) * BT
# ... omitted for brevity ...
    )
    tl.store(p_Aqk, b_Aqk.to(Aqk.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** This function defines `chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_inter`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `q`, `k`, `g`, `beta`, `A`, `Aqk`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.make_block_ptr`, `tl.load`, `tl.zeros`. It writes or updates `i_t`, `i_c`, `i_bh`, `i_b`, `i_h`, `i_i`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_inter`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `q`, `k`, `g`, `beta`, `A`, `Aqk`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.make_block_ptr`, `tl.load`, `tl.zeros`。 它会写入或更新 `i_t`, `i_c`, `i_bh`, `i_b`, `i_h`, `i_i`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 621-714 — function `chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_intra`
```python
@triton.heuristics({"IS_VARLEN": lambda args: args["cu_seqlens"] is not None})
@triton.autotune(
    configs=[triton.Config({}, num_warps=num_warps) for num_warps in [1, 2, 4, 8]],
    key=["BK", "BT"],
)
@triton.jit(do_not_specialize=["T"])
def chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_intra(
    q,
    k,
    g,
    beta,
    A,
    Aqk,
    scale,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    K: tl.constexpr,
    BT: tl.constexpr,
    BC: tl.constexpr,
    BK: tl.constexpr,
    IS_VARLEN: tl.constexpr,
):
    i_t, i_i, i_bh = tl.program_id(0), tl.program_id(1), tl.program_id(2)
    i_b, i_h = i_bh // H, i_bh % H
    if IS_VARLEN:
        i_n, i_t = (
            tl.load(chunk_indices + i_t * 2).to(tl.int32),
            tl.load(chunk_indices + i_t * 2 + 1).to(tl.int32),
        )
        bos, eos = (
            tl.load(cu_seqlens + i_n).to(tl.int32),
            tl.load(cu_seqlens + i_n + 1).to(tl.int32),
        )
        T = eos - bos
    else:
        bos, eos = i_b * T, i_b * T + T

    if i_t * BT + i_i * BC >= T:
        return

    o_i = tl.arange(0, BC)
    o_k = tl.arange(0, BK)
    m_k = o_k < K
    m_A = (i_t * BT + i_i * BC + o_i) < T
    o_A = (bos + i_t * BT + i_i * BC + o_i) * H * BT + i_h * BT + i_i * BC

    p_q = tl.make_block_ptr(
        q + (bos * H + i_h) * K,
        (T, K),
        (H * K, 1),
        (i_t * BT + i_i * BC, 0),
        (BC, BK),
        (1, 0),
    )
# ... omitted for brevity ...
        p_kt += H * K
        p_gk += H * K
```
**EN:** This function defines `chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_intra`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `q`, `k`, `g`, `beta`, `A`, `Aqk`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `tl.make_block_ptr`, `tl.load`. It writes or updates `i_t`, `i_i`, `i_bh`, `i_b`, `i_h`, `o_i`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_intra`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `q`, `k`, `g`, `beta`, `A`, `Aqk`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `tl.make_block_ptr`, `tl.load`。 它会写入或更新 `i_t`, `i_i`, `i_bh`, `i_b`, `i_h`, `o_i`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 717-798 — function `chunk_kda_scaled_dot_kkt_fwd`
```python
def chunk_kda_scaled_dot_kkt_fwd(
    q: torch.Tensor,
    k: torch.Tensor,
    gk: torch.Tensor | None = None,
    beta: torch.Tensor | None = None,
    scale: float | None = None,
    cu_seqlens: torch.Tensor | None = None,
    chunk_size: int = FLA_CHUNK_SIZE,
    output_dtype: torch.dtype = torch.float32,
) -> tuple[torch.Tensor, torch.Tensor]:
    r"""
    Compute beta * K * K^T.

    Args:
        k (torch.Tensor):
            The key tensor of shape `[B, T, H, K]`.
        beta (torch.Tensor):
            The beta tensor of shape `[B, T, H]`.
        gk (torch.Tensor):
            The cumulative sum of the gate tensor of shape `[B, T, H, K]` applied to the key tensor. Default: `None`.
        cu_seqlens (torch.Tensor):
            The cumulative sequence lengths of the input tensor.
            Default: None
        chunk_size (int):
            The chunk size. Default: 64.
        output_dtype (torch.dtype):
            The dtype of the output tensor. Default: `torch.float32`

    Returns:
        beta * K * K^T of shape `[B, T, H, BT]` where `BT` is the chunk size.
    """
    B, T, H, K = k.shape
    assert K <= 256
    BT = chunk_size
    chunk_indices = (
        prepare_chunk_indices(cu_seqlens, BT) if cu_seqlens is not None else None
    )
    NT = cdiv(T, BT) if cu_seqlens is None else len(chunk_indices)

    BC = min(16, BT)
    NC = cdiv(BT, BC)
    BK = max(next_power_of_2(K), 16)
    A = torch.zeros(B, T, H, BT, device=k.device, dtype=output_dtype)
    Aqk = torch.zeros(B, T, H, BT, device=k.device, dtype=output_dtype)
# ... omitted for brevity ...
    )
    return A, Aqk
```
**EN:** This function defines `chunk_kda_scaled_dot_kkt_fwd`. Compute beta * K * K^T. The main inputs are `q`, `k`, `gk`, `beta`, `scale`, `cu_seqlens`. Key calls include `min`, `cdiv`, `max`, `torch.zeros`, `chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_inter`, `chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_intra`. It writes or updates `B`, `T`, `H`, `K`, `BT`, `chunk_indices`.
**CN:** 该函数定义 `chunk_kda_scaled_dot_kkt_fwd`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `q`, `k`, `gk`, `beta`, `scale`, `cu_seqlens`。 关键调用包括 `min`, `cdiv`, `max`, `torch.zeros`, `chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_inter`, `chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_intra`。 它会写入或更新 `B`, `T`, `H`, `K`, `BT`, `chunk_indices`。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] Kda-related kernels and layer integration / [CN] KDA 相关内核与层集成
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `fused_recurrent_kda_fwd`, `fused_recurrent_kda`, `layer_norm_gated_fwd_kernel`, `layer_norm_gated_fwd_kernel1` / [CN] 核心符号：`fused_recurrent_kda_fwd`, `fused_recurrent_kda`, `layer_norm_gated_fwd_kernel`, `layer_norm_gated_fwd_kernel1`

## Dependencies / 依赖关系
- **External**: `torch`, `torch.nn`, `chunk_delta_h`, `cumsum`, `fused_recurrent`, `index`, `l2norm`, `op`, `solve_tril`, `utils` / **外部依赖**: `torch`, `torch.nn`, `chunk_delta_h`, `cumsum`, `fused_recurrent`, `index`, `l2norm`, `op`, `solve_tril`, `utils`
- **Internal**: `vllm.model_executor.custom_op`, `vllm.triton_utils`, `vllm.utils.math_utils` / **内部依赖**: `vllm.model_executor.custom_op`, `vllm.triton_utils`, `vllm.utils.math_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
