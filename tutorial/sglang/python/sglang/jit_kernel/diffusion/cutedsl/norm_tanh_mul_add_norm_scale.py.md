# norm_tanh_mul_add_norm_scale.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/cutedsl/norm_tanh_mul_add_norm_scale.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from typing import Optional, Tuple". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from typing import Optional, Tuple”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
from typing import Optional, Tuple

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

### Lines 57-214: Class `NormTanhMulAddNormScale`
```python
class NormTanhMulAddNormScale:
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
        NormTanhMulAddNormScale kernels.
        """

        def _sig(val):
            if isinstance(val, torch.Tensor):
                return (val.dtype, val.ndim, val.shape[-1])
            return val

        return tuple(_sig(val) for val in inputs)

    def __init__(self, D: int, norm_type: str, is_norm2: bool):
        self.D = D
        self.norm_type = norm_type  # "layer" or "rms"
        self.is_norm2 = is_norm2  # single norm or double norm
        self.num_warps = self.D // 256  # num of warps per cta
        self.num_threads = self.num_warps * WARP_SIZE  # num of threads per cta

    @cute.jit
    def __call__(
        self,
        mY,
        mY2,
        mX,
        mWeight,
        mBias,
        mScale,
        mShift,
        mWeight2,
        mBias2,
        mScale2,
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
            mY2,
            mX,
            mWeight,
            mBias,
            mScale,
            mShift,
            mWeight2,
            mBias2,
            mScale2,
            tiled_copy,
            eps,
        ).launch(
            grid=[B * S, 1, 1],
            block=[self.num_threads, 1, 1],
            stream=stream,
        )

# ...
```
**EN:** This block declares the `NormTanhMulAddNormScale` class and establishes the behavior or state it encapsulates. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `NormTanhMulAddNormScale` 类，并建立其封装的行为或状态。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 215-227: Function `validate_3d`
```python
def validate_3d(t: torch.Tensor, B: int, S: int, D: int):
    if t.dtype not in (torch.float16, torch.bfloat16, torch.float32):
        raise ValueError(f"Validate failed: unsupported dtype: {t.dtype}")
    if (
        t.ndim != 3
        or (t.shape[0] not in (1, B))
        or (t.shape[1] not in (1, S) or t.shape[2] != D)
    ):
        raise ValueError(f"Validate failed: unsupported 3d-tensor: {t.shape}.")
    if t.stride()[-1] != 1:
        raise ValueError(f"Validate failed: not contiguous on dim D.")
```
**EN:** This block defines `validate_3d`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `validate_3d`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 228-239: Function `validate_weight_bias`
```python
def validate_weight_bias(t: Optional[torch.Tensor], D: int):
    if t is None:
        return
    if t.dtype not in (torch.float16, torch.bfloat16, torch.float32):
        raise ValueError(f"Validate failed: unsupported dtype: {t.dtype}")
    if t.shape != (D,):
        raise ValueError(f"Validate failed: unsupported tensor shape: {t.shape}.")
    if t.stride()[-1] != 1:
        raise ValueError(f"Validate failed: not contiguous on dim D.")


@torch.library.custom_op("sglang::fused_norm_tanh_mul_add", mutates_args=())
```
**EN:** This block defines `validate_weight_bias`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `validate_weight_bias`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 240-300: Function `fused_norm_tanh_mul_add`
```python
def fused_norm_tanh_mul_add(
    x: torch.Tensor,
    weight: Optional[torch.Tensor],
    bias: Optional[torch.Tensor],
    scale: torch.Tensor,
    shift: torch.Tensor,
    norm_type: str,
    eps: float = 1e-5,
) -> torch.Tensor:
    """
    Fuse: norm(x) * tanh(scale) + shift
      where norm is either layernorm or rmsnorm.

    Expects:
      - x: [B, S, D]
      - weight/bias: None, [D]
      - scale/shift: [1/B, 1/S, D]
      - norm_type: str, "layer" or "rms"
      - eps: Optional[float], default: 1e-5

    D must be a multiple of 256 and <= 8192 to enable LDG.128 vectorized loads per
    thread and avoid predicated loads (e.g., bounds checks such as `index < D`).
    """
    stream = cuda.CUstream(torch.cuda.current_stream().cuda_stream)
    # Tensor Validation
    BSD = x.shape
    validate_3d(x, *BSD)
    validate_weight_bias(weight, BSD[2])
    validate_weight_bias(bias, BSD[2])
    validate_3d(scale, *BSD)
    validate_3d(shift, *BSD)
    if norm_type == "layer" or norm_type == "rms":
        D = x.shape[-1]
        if D % 256 != 0 or D > 8192:
            raise ValueError(
                f"D={D} not supported, must be multiple of 256 and <= 8192"
            )
        y = torch.empty_like(x)  # create output tensor
        scale = broadcast_tensor_for_bsfd(scale, *x.shape)  # handle various shapes
        shift = broadcast_tensor_for_bsfd(shift, *x.shape)  # handle various shapes
        # y2, weight2, bias2, scale2 is None
        torch_tensors = [y, None, x, weight, bias, scale, shift, None, None, None]
        cute_tensor_args = [to_cute_arg(t) for t in torch_tensors]
        # Compile cache
        hash_key = NormTanhMulAddNormScale.make_hash_key(norm_type, *torch_tensors)
        compiled_fn = _COMPILE_CACHE.get(hash_key)
        if compiled_fn is None:
            kernel = NormTanhMulAddNormScale(D, norm_type, is_norm2=False)
            fake_sig_args = [to_fake_cute_args(t) for t in torch_tensors]
            compiled_fn = cute.compile(
                kernel, *fake_sig_args, options="--enable-tvm-ffi"
            )
            _COMPILE_CACHE[hash_key] = compiled_fn
        # Execute
        compiled_fn(*cute_tensor_args, eps, stream)
        return y
    else:
        raise ValueError(f'norm_type must be one of "layer" and "rms"')


@fused_norm_tanh_mul_add.register_fake
```
**EN:** This block defines `fused_norm_tanh_mul_add`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_norm_tanh_mul_add`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 301-305: Function `_fused_norm_tanh_mul_add_fake`
```python
def _fused_norm_tanh_mul_add_fake(x, weight, bias, scale, shift, norm_type, eps=1e-5):
    return x.new_empty(x.shape)


@torch.library.custom_op("sglang::fused_norm_tanh_mul_add_norm_scale", mutates_args=())
```
**EN:** This block defines `_fused_norm_tanh_mul_add_fake`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_fused_norm_tanh_mul_add_fake`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 306-375: Function `fused_norm_tanh_mul_add_norm_scale`
```python
def fused_norm_tanh_mul_add_norm_scale(
    x: torch.Tensor,
    weight: Optional[torch.Tensor],
    bias: Optional[torch.Tensor],
    scale: torch.Tensor,
    shift: torch.Tensor,
    weight2: Optional[torch.Tensor],
    bias2: Optional[torch.Tensor],
    scale2: torch.Tensor,
    norm_type: str,
    eps: float = 1e-5,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """
    Fuse:
      y = norm(x) * tanh(scale) + shift
      y2 = norm(y) * (1 + scale2)
      where norm is either layernorm or rmsnorm.

    Expects:
      - x: [B, S, D]
      - weight/bia/weight2/bias2: None, [D]
      - scale/shift/scale2: [1/B, 1/S, D]
      - norm_type: str, "layer" or "rms"
      - eps: Optional[float], default: 1e-5

    D must be a multiple of 256 and <= 8192 to enable LDG.128 vectorized loads per
    thread and avoid predicated loads (e.g., bounds checks such as `index < D`).
    """
    stream = cuda.CUstream(torch.cuda.current_stream().cuda_stream)
    # Tensor Validation
    BSD = x.shape
    validate_3d(x, *BSD)
    validate_weight_bias(weight, BSD[2])
    validate_weight_bias(bias, BSD[2])
    validate_3d(scale, *BSD)
    validate_3d(shift, *BSD)
    validate_weight_bias(weight2, BSD[2])
    validate_weight_bias(bias2, BSD[2])
    validate_3d(scale2, *BSD)
    if norm_type == "layer" or norm_type == "rms":
        D = x.shape[-1]
        if D % 256 != 0 or D > 8192:
            raise ValueError(
                f"D={D} not supported, must be multiple of 256 and <= 8192"
            )
        y = torch.empty_like(x)  # create output tensor
        y2 = torch.empty_like(x)  # create output tensor
        scale = broadcast_tensor_for_bsfd(scale, *x.shape)  # handle various shapes
        shift = broadcast_tensor_for_bsfd(shift, *x.shape)  # handle various shapes
        scale2 = broadcast_tensor_for_bsfd(scale2, *x.shape)  # handle various shapes
        torch_tensors = [y, y2, x, weight, bias, scale, shift, weight2, bias2, scale2]
        cute_tensor_args = [to_cute_arg(t) for t in torch_tensors]
        # Compile cache
        hash_key = NormTanhMulAddNormScale.make_hash_key(norm_type, *torch_tensors)
        compiled_fn = _COMPILE_CACHE.get(hash_key)
        if compiled_fn is None:
            kernel = NormTanhMulAddNormScale(D, norm_type, is_norm2=True)
            fake_sig_args = [to_fake_cute_args(t) for t in torch_tensors]
            compiled_fn = cute.compile(
                kernel, *fake_sig_args, options="--enable-tvm-ffi"
            )
            _COMPILE_CACHE[hash_key] = compiled_fn
        # Execute
        compiled_fn(*cute_tensor_args, eps, stream)
        return y, y2
    else:
        raise ValueError(f'norm_type must be one of "layer" and "rms"')


@fused_norm_tanh_mul_add_norm_scale.register_fake
```
**EN:** This block defines `fused_norm_tanh_mul_add_norm_scale`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_norm_tanh_mul_add_norm_scale`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 376-379: Function `_fused_norm_tanh_mul_add_norm_scale_fake`
```python
def _fused_norm_tanh_mul_add_norm_scale_fake(
    x, weight, bias, scale, shift, weight2, bias2, scale2, norm_type, eps=1e-5
):
    return x.new_empty(x.shape), x.new_empty(x.shape)
```
**EN:** This block defines `_fused_norm_tanh_mul_add_norm_scale_fake`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_fused_norm_tanh_mul_add_norm_scale_fake`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

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
