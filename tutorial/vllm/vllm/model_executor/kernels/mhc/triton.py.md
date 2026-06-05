# triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/mhc/triton.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Triton MHC kernel path and wrapper logic. / 实现 Triton 的 MHC 内核路径与包装逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-8)
```python
import torch
import torch.nn.functional as F
from torch import Tensor

from vllm.triton_utils import tl, triton
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This import block loads `torch`, `torch.nn.functional`, `vllm.triton_utils`, `vllm.utils.torch_utils`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `triton.py`.
**CN:** 该导入代码块加载了 `torch`, `torch.nn.functional`, `vllm.triton_utils`, `vllm.utils.torch_utils`，为 `triton.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Function `_rmsnorm_nw_kernel` (lines 12-36)
```python
def _rmsnorm_nw_kernel(
    x_ptr,
    out_ptr,
    stride_row,
    D,
    eps,
    RBLOCK: tl.constexpr,
):
    """Weight-free RMSNorm Triton kernel: out = x * rsqrt(mean(x², -1) + eps)."""
    row = tl.program_id(0)
    cols = tl.arange(0, RBLOCK)
    mask = cols < D

    x = tl.load(
        x_ptr + row * stride_row + cols,
        mask=mask,
        other=0.0,
        eviction_policy="evict_first",
    ).to(tl.float32)

    var = tl.sum(x * x, 0) / D
    rstd = tl.rsqrt(var + eps)

    out = (x * rstd).to(out_ptr.dtype.element_ty)
    tl.store(out_ptr + row * D + cols, out, mask=mask, eviction_policy="evict_first")
```
**EN:** This helper implements `_rmsnorm_nw_kernel`. Weight-free RMSNorm Triton kernel: out = x * rsqrt(mean(x², -1) + eps). Internally it relies on calls such as `program_id`, `arange`, `to`, `rsqrt`.
**CN:** 该函数 `_rmsnorm_nw_kernel` 封装了此模块中的一段关键运行时逻辑，重点处理 rmsnorm nw kernel 相关工作。 它内部会调用 `program_id`, `arange`, `to`, `rsqrt` 等例程。

### Function `rmsnorm_nw` (lines 39-62)
```python
def rmsnorm_nw(x: Tensor, eps: float) -> Tensor:
    """Weight-free RMSNorm over the last dimension.

    Treats *x* as ``[num_rows, D]`` where ``num_rows = product(shape[:-1])``.
    Returns a contiguous tensor with the same shape and dtype as *x*.
    """
    orig_shape = x.shape
    D = orig_shape[-1]
    x_2d = x.reshape(-1, D)
    num_rows = x_2d.shape[0]

    out = torch.empty_like(x_2d)
    RBLOCK = triton.next_power_of_2(D)

    _rmsnorm_nw_kernel[(num_rows,)](
        x_2d,
        out,
        x_2d.stride(0),
        D,
        eps,
        RBLOCK=RBLOCK,
        num_warps=1 if RBLOCK <= 512 else (4 if RBLOCK <= 4096 else 8),
    )
    return out.view(orig_shape)
```
**EN:** This helper implements `rmsnorm_nw`. Weight-free RMSNorm over the last dimension. Internally it relies on calls such as `reshape`, `empty_like`, `next_power_of_2`, `view`.
**CN:** 该函数 `rmsnorm_nw` 封装了此模块中的一段关键运行时逻辑，重点处理 rmsnorm nw 相关工作。 它内部会调用 `reshape`, `empty_like`, `next_power_of_2`, `view` 等例程。

### Function `_hc_head_reduce_store_kernel` (lines 66-105)
```python
def _hc_head_reduce_store_kernel(
    pre_ptr,
    x_ptr,
    out_ptr,
    hidden_size: tl.constexpr,
    hc_mult: tl.constexpr,
    pre_stride_t: tl.constexpr,
    pre_stride_m: tl.constexpr,
    x_stride_t: tl.constexpr,
    x_stride_m: tl.constexpr,
    x_stride_h: tl.constexpr,
    out_stride_t: tl.constexpr,
    out_stride_h: tl.constexpr,
    BLOCK_H: tl.constexpr,
):
    token_idx = tl.program_id(0)
    block_idx = tl.program_id(1)
    offsets = block_idx * BLOCK_H + tl.arange(0, BLOCK_H)
    mask = offsets < hidden_size

    acc = tl.zeros((BLOCK_H,), dtype=tl.float32)
    for mix_idx in tl.static_range(0, hc_mult):
        pre = tl.load(pre_ptr + token_idx * pre_stride_t + mix_idx * pre_stride_m).to(
            tl.float32
        )
        x = tl.load(
            x_ptr
            + token_idx * x_stride_t
            + mix_idx * x_stride_m
            + offsets * x_stride_h,
            mask=mask,
            other=0.0,
        ).to(tl.float32)
        acc += pre * x

    tl.store(
        out_ptr + token_idx * out_stride_t + offsets * out_stride_h,
        acc,
        mask=mask,
    )
```
**EN:** This helper implements `_hc_head_reduce_store_kernel`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `program_id`, `zeros`, `static_range`, `store`.
**CN:** 该函数 `_hc_head_reduce_store_kernel` 封装了此模块中的一段关键运行时逻辑，重点处理 hc head reduce store kernel 相关工作。 它内部会调用 `program_id`, `zeros`, `static_range`, `store` 等例程。

### Function `hc_head_reduce_triton_kernel` (lines 108-140)
```python
def hc_head_reduce_triton_kernel(
    x: torch.Tensor,
    hc_fn: torch.Tensor,
    hc_scale: torch.Tensor,
    hc_base: torch.Tensor,
    out: torch.Tensor,
    norm_eps: float,
    hc_eps: float,
) -> None:
    x_flat = x.flatten(-2)
    x_normed = rmsnorm_nw(x_flat, norm_eps)
    mixes = F.linear(x_normed.float(), hc_fn)
    pre = torch.sigmoid(mixes * hc_scale + hc_base) + hc_eps

    hidden_size = x.shape[-1]
    hc_mult = x.shape[-2]
    block_h = 1024
    _hc_head_reduce_store_kernel[(x.shape[0], (hidden_size + block_h - 1) // block_h)](
        pre,
        x,
        out,
        hidden_size,
        hc_mult,
        pre.stride(0),
        pre.stride(1),
        x.stride(0),
        x.stride(1),
        x.stride(2),
        out.stride(0),
        out.stride(1),
        BLOCK_H=block_h,
        num_warps=4,
    )
```
**EN:** This helper implements `hc_head_reduce_triton_kernel`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `flatten`, `rmsnorm_nw`, `linear`, `float`.
**CN:** 该函数 `hc_head_reduce_triton_kernel` 封装了此模块中的一段关键运行时逻辑，重点处理 hc head reduce triton kernel 相关工作。 它内部会调用 `flatten`, `rmsnorm_nw`, `linear`, `float` 等例程。

### Function `_hc_head_triton` (lines 143-167)
```python
def _hc_head_triton(
    hs_flat: torch.Tensor,
    fn: torch.Tensor,
    hc_scale: torch.Tensor,
    hc_base: torch.Tensor,
    out: torch.Tensor,
    hidden_size: int,
    rms_eps: float,
    hc_eps: float,
    hc_mult: int,
) -> None:
    """Fill pre-allocated `out` (T, H) in-place with the hc_head result."""
    if hs_flat.shape[0] == 0:
        return

    hc_head_reduce_triton_kernel(
        hs_flat,
        fn,
        hc_scale,
        hc_base,
        out,
        rms_eps,
        hc_eps,
    )
    return
```
**EN:** This helper implements `_hc_head_triton`. Fill pre-allocated `out` (T, H) in-place with the hc_head result. Internally it relies on calls such as `hc_head_reduce_triton_kernel`.
**CN:** 该函数 `_hc_head_triton` 封装了此模块中的一段关键运行时逻辑，重点处理 hc head triton 相关工作。 它内部会调用 `hc_head_reduce_triton_kernel` 等例程。

### Top-level block (lines 170-174)
```python
direct_register_custom_op(
    op_name="hc_head_triton",
    op_func=_hc_head_triton,
    mutates_args=["out"],
)
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

## Key Concepts / 关键概念
- **Triton kernels / Triton 内核**
  - **EN:** The implementation relies on Triton-generated GPU kernels for low-level tensor work.
  - **CN:** 该实现依赖 Triton 生成的 GPU 内核完成底层张量计算。
- **Linear kernel abstraction / 线性内核抽象**
  - **EN:** The module defines abstractions or selectors for quantized linear operators.
  - **CN:** 该模块定义量化线性算子的抽象或选择器。
- **MHC kernels / MHC 内核**
  - **EN:** The implementation provides backend-specific MHC kernels and wrappers.
  - **CN:** 该实现提供面向不同后端的 MHC 内核与包装层。
- **Kernel registration / 内核注册**
  - **EN:** The file registers operators, backends, or preset configurations with PyTorch/vLLM.
  - **CN:** 该文件向 PyTorch/vLLM 注册算子、后端或预设配置。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.triton_utils`, `vllm.utils.torch_utils`
- **External / 外部依赖**: `torch`, `torch.nn.functional`
