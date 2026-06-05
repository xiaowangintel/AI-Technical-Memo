# scale_residual_norm_scale_shift.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/cutedsl/scale_residual_norm_scale_shift.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from typing import Optional, Tuple, Union". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from typing import Optional, Tuple, Union”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
from typing import Optional, Tuple, Union

import cuda.bindings.driver as cuda
import cutlass
import cutlass.cute as cute
import torch

from sglang.jit_kernel.diffusion.cutedsl.common.norm_fusion import (
    apply_norm_cta,
    broadcast_tensor_for_bsfd,
    tensor_slice_for_bsfd,
)
from sglang.jit_kernel.diffusion.cutedsl.utils import TORCH_TO_CUTE_DTYPE, WARP_SIZE

_COMPILE_CACHE = {}
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 18-41: Function `to_cute_arg`
```python
def to_cute_arg(
    t,
    *,
    assume_aligned: Optional[int] = 32,
    use_32bit_stride: bool = False,
    enable_tvm_ffi: bool = True,
):
    """
    Convert a Python value into a CuTeDSL value.
    """
    if isinstance(t, torch.Tensor):
        return cute.runtime.from_dlpack(
            t,
            assumed_align=assume_aligned,
            use_32bit_stride=use_32bit_stride,
            enable_tvm_ffi=enable_tvm_ffi,
        )
    if isinstance(t, int):
        return cutlass.Int32(t)
    if isinstance(t, float):
        return cutlass.Float32(t)
    return t
```
**EN:** This block defines `to_cute_arg`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `to_cute_arg`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 42-56: Function `to_fake_cute_args`
```python
def to_fake_cute_args(t: torch.Tensor):
    if isinstance(t, torch.Tensor):
        # Only keep the last dim as compile-time value to maximum compiled kernel reuse
        # e.g. (1,2,1536):(3027,1536,1) -> (?,?,1536):(?,?,1)
        D = t.shape[-1]
        dtype = TORCH_TO_CUTE_DTYPE[t.dtype]
        shape = (*(cute.sym_int() for _ in range(t.ndim - 1)), D)
        stride = (*(cute.sym_int(divisibility=D) for _ in range(t.ndim - 1)), 1)
        fake_t = cute.runtime.make_fake_tensor(
            dtype, shape, stride, memspace=cute.AddressSpace.gmem, assumed_align=32
        )
        return fake_t
    return to_cute_arg(t)
```
**EN:** This block defines `to_fake_cute_args`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `to_fake_cute_args`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 57-227: Class `ScaleResidualNormScaleShift`
```python
class ScaleResidualNormScaleShift:
    @classmethod
    def make_hash_key(cls, *inputs):
        """
        Compile-time values:
          - D: hidden dimension (size of the last dimension)
          - norm_type: layer norm or RMS norm
          - tensor dtype
          - tensor rank (i.e., tensor.ndim)

        Runtime values:
          - all other inputs

        This hash key defines the compile-time specialization boundary for
        ScaleResidualNormScaleShift kernels.
        """

        def _sig(val):
            if isinstance(val, torch.Tensor):
                return (val.dtype, val.ndim, val.shape[-1])
            return val

        return tuple(_sig(val) for val in inputs)

    def __init__(self, D: int, norm_type: str):
        self.D = D
        self.norm_type = norm_type  # "layer" or "rms"
        self.num_warps = self.D // 256  # num of warps per cta
        self.num_threads = self.num_warps * WARP_SIZE  # num of threads per cta

    @cute.jit
    def __call__(
        self,
        mY,
        mResOut,
        mRes,
        mX,
        mGate,
        mWeight,
        mBias,
        mScale,
        mShift,
        eps: cutlass.Float32 = cutlass.Float32(1e-5),
        stream: cuda.CUstream = cuda.CUstream(cuda.CUstream_flags.CU_STREAM_DEFAULT),
    ):
        # Tensor shapes
        B, S, _ = mX.shape  # (batch, seq_len, hidden_dim)
        # Vectorized copy configuration
        num_vectorized = 8  # maximum num of elem per copy
        atom_copy = cute.make_copy_atom(
            cute.nvgpu.CopyUniversalOp(),
            mX.element_type,
            num_bits_per_copy=128,
        )
        # Thread/value layouts for tiled copy
        t_layout = cute.make_layout(self.num_threads)  # thread layout within a CTA
        v_layout = cute.make_layout(num_vectorized)  # per-thread vector layout
        tiled_copy = cute.make_tiled_copy_tv(atom_copy, t_layout, v_layout)

        self.kernel(
            mY,
            mResOut,
            mRes,
            mX,
            mGate,
            mWeight,
            mBias,
            mScale,
            mShift,
            tiled_copy,
            eps,
        ).launch(
            grid=[B * S, 1, 1],
            block=[self.num_threads, 1, 1],
            stream=stream,
        )

    @cute.kernel
    def kernel(
        self,
# ...
```
**EN:** This block declares the `ScaleResidualNormScaleShift` class and establishes the behavior or state it encapsulates. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `ScaleResidualNormScaleShift` 类，并建立其封装的行为或状态。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 228-236: Function `validate_x`
```python
def validate_x(t: torch.Tensor, B: int, S: int, D: int):
    if t.dtype not in (torch.float16, torch.bfloat16, torch.float32):
        raise ValueError(f"Validate failed: unsupported dtype: {t.dtype}")
    if t.shape != (B, S, D):
        raise ValueError(f"Validate failed: unsupported tensor shape: {t.shape}.")
    if t.stride()[-1] != 1:
        raise ValueError(f"Validate failed: not contiguous on dim D.")
```
**EN:** This block defines `validate_x`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `validate_x`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 237-247: Function `validate_weight_bias`
```python
def validate_weight_bias(t: Optional[torch.Tensor], B: int, S: int, D: int):
    if t is None:
        return
    if t.dtype not in (torch.float16, torch.bfloat16, torch.float32):
        raise ValueError(f"Validate failed: unsupported dtype: {t.dtype}")
    if t.shape != (D,):
        raise ValueError(f"Validate failed: unsupported tensor shape: {t.shape}.")
    if t.stride()[-1] != 1:
        raise ValueError(f"Validate failed: not contiguous on dim D.")
```
**EN:** This block defines `validate_weight_bias`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `validate_weight_bias`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 248-271: Function `validate_scale_shift`
```python
def validate_scale_shift(t: torch.Tensor, B: int, S: int, D: int):
    if t.dtype not in (torch.float16, torch.bfloat16, torch.float32):
        raise ValueError(f"Validate failed: unsupported dtype: {t.dtype}")
    failed = False
    if t.ndim == 1 and (t.shape[0] not in (1, D)):
        failed = True
    elif t.ndim == 2 and ((t.shape[0] not in (1, B)) or t.shape[1] != D):
        failed = True
    elif t.ndim == 3 and (
        (t.shape[0] not in (1, B)) or (t.shape[1] not in (1, S) or t.shape[2] != D)
    ):
        failed = True
    elif t.ndim == 4:
        F = t.shape[1]
        if t.shape[0] != B or t.shape[2] != 1 or t.shape[3] != D:
            failed = True
        elif S % F != 0:
            raise ValueError(f"Validate failed: S({S}) must be divisible by F({F}).")
    if failed:
        raise ValueError(f"Validate failed: unsupported tensor shape: {t.shape}.")
    if t.stride()[-1] != 1:
        raise ValueError(f"Validate failed: not contiguous on dim D.")
```
**EN:** This block defines `validate_scale_shift`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `validate_scale_shift`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 272-278: Function `validate_gate`
```python
def validate_gate(t: Union[torch.Tensor, int], B: int, S: int, D: int):
    if not isinstance(t, torch.Tensor):
        return
    validate_scale_shift(t, B, S, D)


@torch.library.custom_op("sglang::fused_norm_scale_shift", mutates_args=())
```
**EN:** This block defines `validate_gate`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `validate_gate`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 279-344: Function `fused_norm_scale_shift`
```python
def fused_norm_scale_shift(
    x: torch.Tensor,
    weight: Optional[torch.Tensor],
    bias: Optional[torch.Tensor],
    scale: torch.Tensor,
    shift: torch.Tensor,
    norm_type: str,
    eps: float = 1e-5,
) -> torch.Tensor:
    """
    Fuse: norm(x) * (1 + scale) + shift
      where norm is either layernorm or rmsnorm.

    Expects:
      - x: [B, S, D]
      - weight/bias: None, [D]
      - scale/shift: [1], [D], [1/B, D], [1/B, 1/S, D] or [B, F, 1, D]
      - norm_type: str, "layer" or "rms"
      - eps: Optional[float], default: 1e-5

    D must be a multiple of 256 and <= 8192 to enable LDG.128 vectorized loads per
    thread and avoid predicated loads (e.g., bounds checks such as `index < D`).
    """
    stream = cuda.CUstream(torch.cuda.current_stream().cuda_stream)
    # Tensor Validation
    BSD = x.shape
    validate_x(x, *BSD)
    validate_weight_bias(weight, *BSD)
    validate_weight_bias(bias, *BSD)
    validate_scale_shift(scale, *BSD)
    validate_scale_shift(shift, *BSD)

    if norm_type == "layer" or norm_type == "rms":
        D = x.shape[-1]
        if D % 256 != 0 or D > 8192:
            raise ValueError(
                f"D={D} not supported, must be multiple of 256 and <= 8192"
            )
        y = torch.empty_like(x)  # create output tensor
        scale = broadcast_tensor_for_bsfd(scale, *x.shape)  # handle various shapes
        shift = broadcast_tensor_for_bsfd(shift, *x.shape)  # handle various shapes
        # Use scalar placeholders for None tensors as a workaround, since the CuTe DSL
        # TVM-FFI backend does not support None parameters. scalar values do not result
        # in code generation and have no impact on runtime performance.
        weight = 1 if weight is None else weight
        bias = 0 if bias is None else bias
        ResOut, Residual, Gate = 0, 0, 1
        torch_tensors = [y, ResOut, Residual, x, Gate, weight, bias, scale, shift]
        # Compile cache
        hash_key = ScaleResidualNormScaleShift.make_hash_key(norm_type, *torch_tensors)
        compiled_fn = _COMPILE_CACHE.get(hash_key)
        if compiled_fn is None:
            kernel = ScaleResidualNormScaleShift(D, norm_type)
            fake_sig_args = [to_fake_cute_args(t) for t in torch_tensors]
            compiled_fn = cute.compile(
                kernel, *fake_sig_args, options="--enable-tvm-ffi"
            )
            _COMPILE_CACHE[hash_key] = compiled_fn
        # Execute
        compiled_fn(*torch_tensors, eps, stream)
        return y
    else:
        raise ValueError(f'norm_type must be one of "layer" and "rms"')


@fused_norm_scale_shift.register_fake
```
**EN:** This block defines `fused_norm_scale_shift`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_norm_scale_shift`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 345-352: Function `_fused_norm_scale_shift_fake`
```python
def _fused_norm_scale_shift_fake(x, weight, bias, scale, shift, norm_type, eps=1e-5):
    y = x.new_empty(x.shape)
    return y


@torch.library.custom_op(
    "sglang::fused_scale_residual_norm_scale_shift", mutates_args=()
)
```
**EN:** This block defines `_fused_norm_scale_shift_fake`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_fused_norm_scale_shift_fake`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 353-426: Function `fused_scale_residual_norm_scale_shift`
```python
def fused_scale_residual_norm_scale_shift(
    residual: torch.Tensor,
    x: torch.Tensor,
    gate: Optional[torch.Tensor],  # Union[Optional[torch.Tensor], int] indeed
    weight: Optional[torch.Tensor],
    bias: Optional[torch.Tensor],
    scale: torch.Tensor,
    shift: torch.Tensor,
    norm_type: str,
    eps: float = 1e-5,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """
    Fuse: norm(residual + gate * x) * (1 + scale) + shift
      where norm is either layernorm or rmsnorm.

    Expects:
      - residual, x: [B, S, D]
      - gate: None, [1], [D], [1/B, D], [1/B, 1/S, D] or [B, F, 1, D]
      - weight/bias: None, [D]
      - scale/shift: [1], [D], [1/B, D], [1/B, 1/S, D] or [B, F, 1, D]
      - norm_type: str, "layer" or "rms"
      - eps: Optional[float], default: 1e-5

    D must be a multiple of 256 and <= 8192 to enable LDG.128 vectorized loads per
    thread and avoid predicated loads (e.g., bounds checks such as `index < D`).
    """
    # Tensor Validation
    BSD = x.shape
    validate_x(x, *BSD)
    validate_x(residual, *BSD)
    validate_gate(gate, *BSD)
    validate_weight_bias(weight, *BSD)
    validate_weight_bias(bias, *BSD)
    validate_scale_shift(scale, *BSD)
    validate_scale_shift(shift, *BSD)
    if norm_type == "layer" or norm_type == "rms":
        stream = cuda.CUstream(torch.cuda.current_stream().cuda_stream)

        # if norm_type == "layer" or norm_type == "rms":
        D = x.shape[-1]
        if D % 256 != 0 or D > 8192:
            raise ValueError(
                f"D={D} not supported, must be multiple of 256 and <= 8192"
            )
        y = torch.empty_like(x)  # create output tensor
        resi_out = torch.empty_like(x)  # create output tensor
        gate = broadcast_tensor_for_bsfd(gate, *x.shape)  # handle various shapes
        scale = broadcast_tensor_for_bsfd(scale, *x.shape)  # handle various shapes
        shift = broadcast_tensor_for_bsfd(shift, *x.shape)  # handle various shapes
        # Use scalar placeholders for None tensors as a workaround, since the CuTe DSL
        # TVM-FFI backend does not support None parameters. scalar values do not result
        # in code generation and have no impact on runtime performance.
        gate = 1 if gate is None else gate
        weight = 1 if weight is None else weight
        bias = 0 if bias is None else bias
        torch_tensors = [y, resi_out, residual, x, gate, weight, bias, scale, shift]
        # Compile cache
        hash_key = ScaleResidualNormScaleShift.make_hash_key(norm_type, *torch_tensors)
        compiled_fn = _COMPILE_CACHE.get(hash_key)
        if compiled_fn is None:
            kernel = ScaleResidualNormScaleShift(D, norm_type)
            fake_sig_args = [to_fake_cute_args(t) for t in torch_tensors]
            compiled_fn = cute.compile(
                kernel, *fake_sig_args, options="--enable-tvm-ffi"
            )
            _COMPILE_CACHE[hash_key] = compiled_fn
        # Execute
        compiled_fn(*torch_tensors, eps, stream)
        return y, resi_out
    else:
        raise ValueError(f'norm_type must be one of "layer" and "rms"')


@fused_scale_residual_norm_scale_shift.register_fake
```
**EN:** This block defines `fused_scale_residual_norm_scale_shift`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_scale_residual_norm_scale_shift`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 427-432: Function `_fused_scale_residual_norm_scale_shift_fake`
```python
def _fused_scale_residual_norm_scale_shift_fake(
    residual, x, gate, weight, bias, scale, shift, norm_type, eps=1e-5
):
    y = x.new_empty(x.shape)
    residual_out = x.new_empty(x.shape)
    return y, residual_out
```
**EN:** This block defines `_fused_scale_residual_norm_scale_shift_fake`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_fused_scale_residual_norm_scale_shift_fake`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `typing -> Optional`
- `cuda.bindings.driver as cuda`
- `cutlass`
- `cutlass.cute as cute`
- `torch`
- `sglang.jit_kernel.diffusion.cutedsl.common.norm_fusion -> (`
- `sglang.jit_kernel.diffusion.cutedsl.utils -> TORCH_TO_CUTE_DTYPE`
