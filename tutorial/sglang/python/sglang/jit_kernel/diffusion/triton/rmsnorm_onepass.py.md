# rmsnorm_onepass.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/triton/rmsnorm_onepass.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import torch". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import torch”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and module setup
```python
import torch
import triton  # type: ignore
import triton.language as tl  # type: ignore

from sglang.kernel_api_logging import debug_kernel_api
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.srt.utils.custom_op import register_custom_op


# Adapted from https://github.com/ModelTC/LightX2V/blob/main/lightx2v/common/ops/norm/triton_ops.py#L905-L956
@triton.jit
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 12-40: Function `_rms_norm_tiled_onepass`
```python
def _rms_norm_tiled_onepass(
    y_ptr,
    x_ptr,
    w_ptr,
    SEQ: tl.constexpr,
    DIM: tl.constexpr,
    EPS: tl.constexpr,
    BLOCK_SIZE_SEQ: tl.constexpr,
    BLOCK_SIZE_DIM: tl.constexpr,
):
    seq_blk_id = tl.program_id(0)
    seq_id = seq_blk_id * BLOCK_SIZE_SEQ

    seq_offset = seq_id + tl.arange(0, BLOCK_SIZE_SEQ)[:, None]
    s_mask = seq_offset < SEQ
    d_offset = tl.arange(0, BLOCK_SIZE_DIM)[None, :]
    d_mask = d_offset < DIM
    y_blk = y_ptr + seq_offset * DIM + d_offset
    x_blk = x_ptr + seq_offset * DIM + d_offset
    mask = s_mask & d_mask

    x = tl.load(x_blk, mask=mask, other=0.0).to(tl.float32)
    mean_square = tl.sum(x * x, axis=1, keep_dims=True) / DIM
    rstd = tl.math.rsqrt(mean_square + EPS)
    w = tl.load(w_ptr + d_offset, mask=d_mask)
    tl.store(y_blk, x * rstd * w, mask=mask)


@register_custom_op(op_name="triton_one_pass_rms_norm_cuda", out_shape="x")
```
**EN:** This block defines `_rms_norm_tiled_onepass`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_rms_norm_tiled_onepass`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 41-67: Function `_triton_one_pass_rms_norm_cuda`
```python
def _triton_one_pass_rms_norm_cuda(
    x: torch.Tensor, w: torch.Tensor, eps: float = 1e-6
) -> torch.Tensor:
    shape = x.shape
    x = x.contiguous()
    y = torch.empty_like(x)
    x_view = x.reshape(-1, shape[-1])
    y_view = y.reshape(-1, shape[-1])
    S, D = x_view.shape

    block_size_seq = min(16, triton.next_power_of_2(max(1, S // 512)))
    grid = (triton.cdiv(S, block_size_seq),)

    with torch.get_device_module().device(x.device):
        _rms_norm_tiled_onepass[grid](
            y_view,
            x_view,
            w,
            S,
            D,
            eps,
            BLOCK_SIZE_DIM=triton.next_power_of_2(D),
            BLOCK_SIZE_SEQ=block_size_seq,
        )
    return y
```
**EN:** This block defines `_triton_one_pass_rms_norm_cuda`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_triton_one_pass_rms_norm_cuda`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 68-83: Imports and module setup
```python
def triton_one_pass_rms_norm(x: torch.Tensor, w: torch.Tensor, eps: float = 1e-6):
    return _triton_one_pass_rms_norm_cuda(x, w, eps)


if current_platform.is_mps():
    from .mps_fallback import triton_one_pass_rms_norm_native

    @debug_kernel_api
    def triton_one_pass_rms_norm(x: torch.Tensor, w: torch.Tensor, eps: float = 1e-6):
        return triton_one_pass_rms_norm_native(x, w, eps)


if current_platform.is_cpu():
    from .torch_fallback import triton_one_pass_rms_norm_native

    triton_one_pass_rms_norm = triton_one_pass_rms_norm_native
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language as tl`
- `sglang.kernel_api_logging -> debug_kernel_api`
- `sglang.multimodal_gen.runtime.platforms -> current_platform`
- `sglang.srt.utils.custom_op -> register_custom_op`
- `.mps_fallback -> triton_one_pass_rms_norm_native`
- `.torch_fallback -> triton_one_pass_rms_norm_native`
