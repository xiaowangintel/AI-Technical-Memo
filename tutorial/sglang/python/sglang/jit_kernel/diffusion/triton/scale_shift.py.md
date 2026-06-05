# scale_shift.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/triton/scale_shift.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import torch". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import torch”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and module setup
```python
import torch
import triton  # type: ignore
import triton.language as tl  # type: ignore

from sglang.multimodal_gen.runtime.platforms import current_platform


@triton.jit
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 9-102: Function `_fused_layernorm_scale_shift_gate_select01_kernel`
```python
def _fused_layernorm_scale_shift_gate_select01_kernel(
    output_ptr,
    gate_out_ptr,
    x_ptr,
    weight_ptr,
    bias_ptr,
    scale0_ptr,
    shift0_ptr,
    gate0_ptr,
    scale1_ptr,
    shift1_ptr,
    gate1_ptr,
    index_ptr,
    inner_dim,
    seq_len,
    stride_x_row,
    stride_out_row,
    stride_go_row,
    stride_w,
    stride_b,
    stride_s0_b,
    stride_s0_c,
    stride_sh0_b,
    stride_sh0_c,
    stride_g0_b,
    stride_g0_c,
    stride_s1_b,
    stride_s1_c,
    stride_sh1_b,
    stride_sh1_c,
    stride_g1_b,
    stride_g1_c,
    stride_i_b,
    stride_i_l,
    eps,
    HAS_WEIGHT: tl.constexpr,
    HAS_BIAS: tl.constexpr,
    BLOCK_N: tl.constexpr,
):
    row = tl.program_id(0)
    cols = tl.arange(0, BLOCK_N)
    mask = cols < inner_dim

    x_row_ptr = x_ptr + row * stride_x_row
    out_row_ptr = output_ptr + row * stride_out_row
    gate_row_ptr = gate_out_ptr + row * stride_go_row

    x = tl.load(x_row_ptr + cols, mask=mask, other=0.0).to(tl.float32)
    mean = tl.sum(x, axis=0) / inner_dim
    xbar = tl.where(mask, x - mean, 0.0)
    var = tl.sum(xbar * xbar, axis=0) / inner_dim
    rstd = tl.rsqrt(var + eps)
    x_hat = (x - mean) * rstd

    if HAS_WEIGHT:
        w = tl.load(weight_ptr + cols * stride_w, mask=mask, other=1.0).to(tl.float32)
        x_hat = x_hat * w
    if HAS_BIAS:
        b = tl.load(bias_ptr + cols * stride_b, mask=mask, other=0.0).to(tl.float32)
        x_hat = x_hat + b

    batch_idx = row // seq_len
    seq_idx = row % seq_len
    idx = tl.load(index_ptr + batch_idx * stride_i_b + seq_idx * stride_i_l).to(tl.int1)

    scale0_ptrs = scale0_ptr + batch_idx * stride_s0_b + cols * stride_s0_c
    shift0_ptrs = shift0_ptr + batch_idx * stride_sh0_b + cols * stride_sh0_c
    gate0_ptrs = gate0_ptr + batch_idx * stride_g0_b + cols * stride_g0_c

    scale1_ptrs = scale1_ptr + batch_idx * stride_s1_b + cols * stride_s1_c
    shift1_ptrs = shift1_ptr + batch_idx * stride_sh1_b + cols * stride_sh1_c
    gate1_ptrs = gate1_ptr + batch_idx * stride_g1_b + cols * stride_g1_c

    # Branch on scalar idx instead of using tl.where on pointers.
    # tl.where on pointers triggers an assertion in AMD Triton's
    # CanonicalizePointers pass (ConvertArithSelectOp) on gfx950.
    # This keeps it at 3 loads (not 6), avoids the pointer-level
    # tl.where entirely, and since idx is uniform across all threads
    # the branch has no divergence cost.
    if idx:
# ...
```
**EN:** This block defines `_fused_layernorm_scale_shift_gate_select01_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_fused_layernorm_scale_shift_gate_select01_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 103-220: Function `_fused_residual_layernorm_scale_shift_gate_select01_kernel`
```python
def _fused_residual_layernorm_scale_shift_gate_select01_kernel(
    output_ptr,
    residual_out_ptr,
    gate_out_ptr,
    x_ptr,
    residual_ptr,
    residual_gate_ptr,
    weight_ptr,
    bias_ptr,
    scale0_ptr,
    shift0_ptr,
    gate0_ptr,
    scale1_ptr,
    shift1_ptr,
    gate1_ptr,
    index_ptr,
    inner_dim,
    seq_len,
    stride_x_row,
    stride_res_row,
    stride_rg_row,
    stride_out_row,
    stride_res_out_row,
    stride_go_row,
    stride_w,
    stride_b,
    stride_s0_b,
    stride_s0_c,
    stride_sh0_b,
    stride_sh0_c,
    stride_g0_b,
    stride_g0_c,
    stride_s1_b,
    stride_s1_c,
    stride_sh1_b,
    stride_sh1_c,
    stride_g1_b,
    stride_g1_c,
    stride_i_b,
    stride_i_l,
    eps,
    HAS_WEIGHT: tl.constexpr,
    HAS_BIAS: tl.constexpr,
    BLOCK_N: tl.constexpr,
):
    row = tl.program_id(0)
    cols = tl.arange(0, BLOCK_N)
    mask = cols < inner_dim

    x_row_ptr = x_ptr + row * stride_x_row
    res_row_ptr = residual_ptr + row * stride_res_row
    rg_row_ptr = residual_gate_ptr + row * stride_rg_row
    out_row_ptr = output_ptr + row * stride_out_row
    res_out_row_ptr = residual_out_ptr + row * stride_res_out_row
    gate_row_ptr = gate_out_ptr + row * stride_go_row

    x = tl.load(x_row_ptr + cols, mask=mask, other=0.0).to(tl.float32)
    residual = tl.load(res_row_ptr + cols, mask=mask, other=0.0).to(tl.float32)
    residual_gate = tl.load(rg_row_ptr + cols, mask=mask, other=0.0).to(tl.float32)
    residual_out = residual + residual_gate * x
    tl.store(res_out_row_ptr + cols, residual_out, mask=mask)

    mean = tl.sum(residual_out, axis=0) / inner_dim
    xbar = tl.where(mask, residual_out - mean, 0.0)
    var = tl.sum(xbar * xbar, axis=0) / inner_dim
    rstd = tl.rsqrt(var + eps)
    x_hat = (residual_out - mean) * rstd

    if HAS_WEIGHT:
        w = tl.load(weight_ptr + cols * stride_w, mask=mask, other=1.0).to(tl.float32)
        x_hat = x_hat * w
    if HAS_BIAS:
        b = tl.load(bias_ptr + cols * stride_b, mask=mask, other=0.0).to(tl.float32)
        x_hat = x_hat + b

    batch_idx = row // seq_len
    seq_idx = row % seq_len
    idx = tl.load(index_ptr + batch_idx * stride_i_b + seq_idx * stride_i_l).to(tl.int1)

    scale0_ptrs = scale0_ptr + batch_idx * stride_s0_b + cols * stride_s0_c
# ...
```
**EN:** This block defines `_fused_residual_layernorm_scale_shift_gate_select01_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_fused_residual_layernorm_scale_shift_gate_select01_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 221-265: Function `_fused_scale_shift_4d_kernel`
```python
def _fused_scale_shift_4d_kernel(
    output_ptr,
    normalized_ptr,
    scale_ptr,
    shift_ptr,
    scale_constant: tl.constexpr,  # scale_constant is either 0 or 1.
    rows,
    inner_dim,
    seq_len,
    num_frames,
    frame_seqlen,
    BLOCK_N: tl.constexpr,
):
    pid_row = tl.program_id(0)
    pid_col = tl.program_id(1)

    col_offsets = pid_col * BLOCK_N + tl.arange(0, BLOCK_N)
    mask = col_offsets < inner_dim

    # Pointers for normalized and output
    row_base = pid_row * inner_dim
    norm_ptrs = normalized_ptr + row_base + col_offsets
    out_ptrs = output_ptr + row_base + col_offsets

    # Pointers for scale (per-frame) and shift (per-token)
    b_idx = pid_row // seq_len
    t_idx = pid_row % seq_len
    frame_idx_in_batch = t_idx // frame_seqlen

    scale_row_idx = b_idx * num_frames + frame_idx_in_batch
    scale_ptrs = scale_ptr + scale_row_idx * inner_dim + col_offsets
    # shift is per-token [B*L, C], indexed by pid_row directly
    shift_ptrs = shift_ptr + pid_row * inner_dim + col_offsets

    normalized = tl.load(norm_ptrs, mask=mask, other=0.0)
    scale = tl.load(scale_ptrs, mask=mask, other=0.0)
    shift = tl.load(shift_ptrs, mask=mask, other=0.0)

    scale_const_tensor = tl.full([BLOCK_N], scale_constant, dtype=scale.dtype)
    output = normalized * (scale_const_tensor + scale) + shift

    tl.store(out_ptrs, output, mask=mask)


@triton.jit
```
**EN:** This block defines `_fused_scale_shift_4d_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_fused_scale_shift_4d_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 266-332: Function `fuse_scale_shift_kernel_blc_opt`
```python
def fuse_scale_shift_kernel_blc_opt(
    x_ptr,
    shift_ptr,
    scale_ptr,
    scale_constant: tl.constexpr,  # scale_constant is either 0 or 1.,
    y_ptr,
    B,
    L,
    C,
    stride_x_b,
    stride_x_l,
    stride_x_c,
    stride_s_b,
    stride_s_l,
    stride_s_c,
    stride_sc_b,
    stride_sc_l,
    stride_sc_c,
    SCALE_IS_SCALAR: tl.constexpr,
    SHIFT_IS_SCALAR: tl.constexpr,
    BLOCK_L: tl.constexpr,
    BLOCK_C: tl.constexpr,
):
    pid_l = tl.program_id(0)
    pid_c = tl.program_id(1)
    pid_b = tl.program_id(2)

    l_offsets = pid_l * BLOCK_L + tl.arange(0, BLOCK_L)
    c_offsets = pid_c * BLOCK_C + tl.arange(0, BLOCK_C)

    mask_l = l_offsets < L
    mask_c = c_offsets < C
    mask = mask_l[:, None] & mask_c[None, :]

    x_off = (
        pid_b * stride_x_b
        + l_offsets[:, None] * stride_x_l
        + c_offsets[None, :] * stride_x_c
    )
    x = tl.load(x_ptr + x_off, mask=mask, other=0)

    if SHIFT_IS_SCALAR:
        shift_val = tl.load(shift_ptr)
        shift = tl.full((BLOCK_L, BLOCK_C), shift_val, dtype=shift_val.dtype)
    else:
        s_off = (
            pid_b * stride_s_b
            + l_offsets[:, None] * stride_s_l
            + c_offsets[None, :] * stride_s_c
        )
        shift = tl.load(shift_ptr + s_off, mask=mask, other=0)

    if SCALE_IS_SCALAR:
        scale_val = tl.load(scale_ptr)
        scale = tl.full((BLOCK_L, BLOCK_C), scale_val, dtype=scale_val.dtype)
    else:
        sc_off = (
            pid_b * stride_sc_b
            + l_offsets[:, None] * stride_sc_l
            + c_offsets[None, :] * stride_sc_c
        )
        scale = tl.load(scale_ptr + sc_off, mask=mask, other=0)

    y = x * (scale_constant + scale) + shift
    tl.store(y_ptr + x_off, y, mask=mask)
```
**EN:** This block defines `fuse_scale_shift_kernel_blc_opt`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fuse_scale_shift_kernel_blc_opt`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 333-454: Function `fuse_scale_shift_kernel`
```python
def fuse_scale_shift_kernel(
    x: torch.Tensor,
    scale: torch.Tensor,
    shift: torch.Tensor,
    scale_constant: float = 1.0,
    block_l: int = 128,
    block_c: int = 128,
):
    assert (x.is_cuda and scale.is_cuda) or (x.is_xpu and scale.is_xpu)
    assert x.is_contiguous()

    B, L, C = x.shape
    output = torch.empty_like(x)

    if scale.dim() == 4:
        # scale/shift: [B, F, 1, C]
        rows = B * L
        x_2d = x.view(rows, C)
        output_2d = output.view(rows, C)

        def grid(meta):
            return (rows, triton.cdiv(C, meta["BLOCK_N"]))

        num_frames = scale.shape[1]
        assert (
            L % num_frames == 0
        ), "seq_len must be divisible by num_frames for 4D scale/shift"
        frame_seqlen = L // num_frames

        # Compact scale [B, F, 1, C] -> [B*F, C] (per-frame)
        scale_reshaped = scale.squeeze(2).reshape(-1, C).contiguous()
        # shift is per-token [B, L, C] -> [B*L, C]
        shift_reshaped = shift.reshape(rows, C).contiguous()

        _fused_scale_shift_4d_kernel[grid](
            output_2d,
            x_2d,
            scale_reshaped,
            shift_reshaped,
            scale_constant,
            rows,
            C,
            L,
            num_frames,
            frame_seqlen,
        )
    else:
        # 2D: [B, C] or [1, C]  -> treat as [B, 1, C] and broadcast over L
        # 3D: [B, L, C] (or broadcastable variants like [B, 1, C], [1, L, C], [1, 1, C])
        # Also support scalar (0D or 1-element)
        if scale.dim() == 0 or (scale.dim() == 1 and scale.numel() == 1):
            scale_blc = scale.reshape(1)
        elif scale.dim() == 2:
            scale_blc = scale[:, None, :]
        elif scale.dim() == 3:
            scale_blc = scale
        else:
            raise ValueError("scale must be 0D/1D(1)/2D/3D or 4D")

        if shift.dim() == 0 or (shift.dim() == 1 and shift.numel() == 1):
            shift_blc = shift.reshape(1)
        elif shift.dim() == 2:
            shift_blc = shift[:, None, :]
        elif shift.dim() == 3:
            shift_blc = shift
        else:
            # broadcast later via expand if possible
            shift_blc = shift

        need_scale_scalar = scale_blc.dim() == 1 and scale_blc.numel() == 1
        need_shift_scalar = shift_blc.dim() == 1 and shift_blc.numel() == 1

        if not need_scale_scalar:
            scale_exp = scale_blc.expand(B, L, C)
            s_sb, s_sl, s_sc = scale_exp.stride()
        else:
            s_sb = s_sl = s_sc = 0

        if not need_shift_scalar:
            shift_exp = shift_blc.expand(B, L, C)
# ...
```
**EN:** This block defines `fuse_scale_shift_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fuse_scale_shift_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 455-546: Function `fuse_layernorm_scale_shift_gate_select01_kernel`
```python
def fuse_layernorm_scale_shift_gate_select01_kernel(
    x: torch.Tensor,
    weight: torch.Tensor | None,
    bias: torch.Tensor | None,
    scale0: torch.Tensor,
    shift0: torch.Tensor,
    gate0: torch.Tensor,
    scale1: torch.Tensor,
    shift1: torch.Tensor,
    gate1: torch.Tensor,
    index: torch.Tensor,
    eps: float,
):
    assert x.is_cuda
    assert x.is_contiguous()
    B, L, C = x.shape
    output = torch.empty_like(x)
    gate_out = torch.empty_like(x)

    if (
        scale0.dim() != 2
        or shift0.dim() != 2
        or gate0.dim() != 2
        or scale1.dim() != 2
        or shift1.dim() != 2
        or gate1.dim() != 2
    ):
        raise ValueError("scale0/shift0/gate0/scale1/shift1/gate1 must be 2D [B, C]")
    if index.dim() != 2:
        raise ValueError("index must be 2D [B, L]")
    if weight is not None and (weight.dim() != 1 or weight.shape[0] != C):
        raise ValueError("weight must be 1D [C]")
    if bias is not None and (bias.dim() != 1 or bias.shape[0] != C):
        raise ValueError("bias must be 1D [C]")

    x_2d = x.view(B * L, C)
    output_2d = output.view(B * L, C)
    gate_out_2d = gate_out.view(B * L, C)
    weight = weight.contiguous() if weight is not None else x_2d
    bias = bias.contiguous() if bias is not None else x_2d

    MAX_FUSED_SIZE = 65536 // x_2d.element_size()
    BLOCK_N = min(MAX_FUSED_SIZE, triton.next_power_of_2(C))
    if C > BLOCK_N:
        raise RuntimeError("This layer norm doesn't support feature dim >= 64KB.")
    num_warps, num_stages = 4, 4

    grid = (B * L,)
    _fused_layernorm_scale_shift_gate_select01_kernel[grid](
        output_2d,
        gate_out_2d,
        x_2d,
        weight,
        bias,
        scale0.contiguous(),
        shift0.contiguous(),
        gate0.contiguous(),
        scale1.contiguous(),
        shift1.contiguous(),
        gate1.contiguous(),
        index.contiguous(),
        C,
        L,
        x_2d.stride(0),
        output_2d.stride(0),
        gate_out_2d.stride(0),
        weight.stride(0) if weight.dim() == 1 else 0,
        bias.stride(0) if bias.dim() == 1 else 0,
        scale0.stride(0),
        scale0.stride(1),
        shift0.stride(0),
        shift0.stride(1),
        gate0.stride(0),
        gate0.stride(1),
        scale1.stride(0),
        scale1.stride(1),
        shift1.stride(0),
        shift1.stride(1),
        gate1.stride(0),
        gate1.stride(1),
# ...
```
**EN:** This block defines `fuse_layernorm_scale_shift_gate_select01_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fuse_layernorm_scale_shift_gate_select01_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 547-677: Function `fuse_residual_layernorm_scale_shift_gate_select01_kernel`
```python
def fuse_residual_layernorm_scale_shift_gate_select01_kernel(
    x: torch.Tensor,
    residual: torch.Tensor,
    residual_gate: torch.Tensor,
    weight: torch.Tensor | None,
    bias: torch.Tensor | None,
    scale0: torch.Tensor,
    shift0: torch.Tensor,
    gate0: torch.Tensor,
    scale1: torch.Tensor,
    shift1: torch.Tensor,
    gate1: torch.Tensor,
    index: torch.Tensor,
    eps: float,
):
    assert x.is_cuda
    assert x.is_contiguous()
    assert residual.is_contiguous()
    assert residual_gate.is_contiguous()
    B, L, C = x.shape
    output = torch.empty_like(x)
    residual_out = torch.empty_like(x)
    gate_out = torch.empty_like(x)

    if residual.shape != x.shape:
        raise ValueError("residual must have the same shape as x")
    if residual_gate.shape != x.shape:
        raise ValueError("residual_gate must have the same shape as x")
    if (
        scale0.dim() != 2
        or shift0.dim() != 2
        or gate0.dim() != 2
        or scale1.dim() != 2
        or shift1.dim() != 2
        or gate1.dim() != 2
    ):
        raise ValueError("scale0/shift0/gate0/scale1/shift1/gate1 must be 2D [B, C]")
    if index.dim() != 2:
        raise ValueError("index must be 2D [B, L]")
    if weight is not None and (weight.dim() != 1 or weight.shape[0] != C):
        raise ValueError("weight must be 1D [C]")
    if bias is not None and (bias.dim() != 1 or bias.shape[0] != C):
        raise ValueError("bias must be 1D [C]")

    x_2d = x.view(B * L, C)
    residual_2d = residual.view(B * L, C)
    residual_gate_2d = residual_gate.view(B * L, C)
    output_2d = output.view(B * L, C)
    residual_out_2d = residual_out.view(B * L, C)
    gate_out_2d = gate_out.view(B * L, C)
    weight = weight.contiguous() if weight is not None else x_2d
    bias = bias.contiguous() if bias is not None else x_2d

    MAX_FUSED_SIZE = 65536 // x_2d.element_size()
    BLOCK_N = min(MAX_FUSED_SIZE, triton.next_power_of_2(C))
    if C > BLOCK_N:
        raise RuntimeError("This layer norm doesn't support feature dim >= 64KB.")
    num_warps, num_stages = 4, 4

    grid = (B * L,)
    _fused_residual_layernorm_scale_shift_gate_select01_kernel[grid](
        output_2d,
        residual_out_2d,
        gate_out_2d,
        x_2d,
        residual_2d,
        residual_gate_2d,
        weight,
        bias,
        scale0.contiguous(),
        shift0.contiguous(),
        gate0.contiguous(),
        scale1.contiguous(),
        shift1.contiguous(),
        gate1.contiguous(),
        index.contiguous(),
        C,
        L,
        x_2d.stride(0),
        residual_2d.stride(0),
# ...
```
**EN:** This block defines `fuse_residual_layernorm_scale_shift_gate_select01_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fuse_residual_layernorm_scale_shift_gate_select01_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language as tl`
- `sglang.multimodal_gen.runtime.platforms -> current_platform`
- `.npu_fallback -> fuse_scale_shift_native`
- `.mps_fallback -> fuse_scale_shift_kernel_native`
- `.torch_fallback -> fuse_scale_shift_kernel_native`
- `.torch_fallback -> (`
