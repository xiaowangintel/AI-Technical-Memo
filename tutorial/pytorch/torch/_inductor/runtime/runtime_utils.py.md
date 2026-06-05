# runtime_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/runtime/runtime_utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module provides runtime helpers used after compilation. It exposes functions such as `conditional_product`, `ceildiv`, `is_power_of_2`, `next_power_of_2`, `last_power_of_2`, `get_num_bytes`, and `...+24`.
- **用途（中文）**: 该模块提供编译完成后使用的运行期辅助逻辑。同时提供 `conditional_product`、`ceildiv`、`is_power_of_2`、`next_power_of_2`、`last_power_of_2`、`get_num_bytes`、`另有24项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
from __future__ import annotations

import functools
import math
import operator
from typing import Any, TYPE_CHECKING

import sympy

import torch

# NOTE: other files rely on the imports below
from torch._dynamo import callback as compilation_callback  # noqa: F401
from torch._inductor.runtime.cache_dir_utils import (  # noqa: F401
    cache_dir,
    default_cache_dir,
    triton_cache_dir,
)


if TYPE_CHECKING:
    from collections.abc import Hashable

    from .triton_compat import Config


def conditional_product(*args: int) -> int:
    return functools.reduce(operator.mul, [x for x in args if x])
````
- **EN**: Imports dependencies such as `__future__`, `functools`, `math`, `operator`, `typing`, `sympy`, and `...+5` for the logic in this range. Introduces function `conditional_product`. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `__future__`、`functools`、`math`、`operator`、`typing`、`sympy`、`另有5项` 等依赖，为后续逻辑提供基础能力。这里定义了函数`conditional_product`。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 29-56 / 第 29-56 行
````python


def ceildiv(number: int, denom: int) -> int:
    return -(number // -denom)


def is_power_of_2(n: int) -> bool:
    """Returns whether n = 2 ** m for some integer m."""
    return n > 0 and n & n - 1 == 0


def next_power_of_2(n: int) -> int:
    """Return the smallest power of 2 greater than or equal to n"""
    if isinstance(n, sympy.Integer):
        n = int(n)
    if n <= 0:
        return 1
    return 1 << (n - 1).bit_length()


def last_power_of_2(n: int) -> int:
    """Return the largest power of 2 less than or equal to n"""
    next_pow2 = next_power_of_2(n)
    return next_pow2 // 2 if next_pow2 > n else next_pow2


def get_num_bytes(*args: torch.Tensor, num_in_out_args: int = 0) -> int:
    """
````
- **EN**: Introduces function `ceildiv`, function `is_power_of_2`, function `next_power_of_2`, function `last_power_of_2`, function `get_num_bytes`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`ceildiv`、函数`is_power_of_2`、函数`next_power_of_2`、函数`last_power_of_2`、函数`get_num_bytes`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 57-84 / 第 57-84 行
````python
    Return the total number of bytes the arguments of tensor type takes.

    For in/out args, tensor sizes are counted twice: once for reading and
    once for writing.

    The first num_in_out_args arguments are in out tensors.
    """
    return sum(
        arg.numel() * arg.element_size() * (1 + int(i < num_in_out_args))
        for i, arg in enumerate(args)
        if isinstance(arg, torch.Tensor)
    )


def triton_config_to_hashable(cfg: Config) -> Hashable:
    """
    Convert triton config to a tuple that can uniquely identify it. We can use
    the return value as a dictionary key.
    """
    # pyrefly: ignore [missing-attribute]
    items = sorted(cfg.kwargs.items())
    # pyrefly: ignore [missing-attribute]
    items.append(("num_warps", cfg.num_warps))
    # pyrefly: ignore [missing-attribute]
    items.append(("num_stages", cfg.num_stages))
    return tuple(items)


````
- **EN**: Introduces function `triton_config_to_hashable`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`triton_config_to_hashable`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 85-112 / 第 85-112 行
````python
def validate_triton_config(cfg: Config) -> None:
    # [Note: Triton pre_hook in inductor]
    # pre-hook is a lambda function, which we don't attempt to serialize.
    # right now, if a pre-hook is attached to the config, it will not be saved;
    # and then it won't be used when the config is loaded from cache.
    # So we assert - if we do get a pre_hook, it might get ignored after caching.
    assert getattr(cfg, "pre_hook", None) is None, (
        "triton configs with pre_hooks not supported"
    )


def create_bandwidth_info_str(
    ms: float,
    num_gb: float,
    gb_per_s: float,
    prefix: str = "",
    suffix: str = "",
    color: bool = True,
) -> str:
    info_str = f"{prefix}{ms:.3f}ms    \t{num_gb:.3f} GB \t {gb_per_s:7.2f}GB/s{suffix}"
    slow = ms > 0.012 and gb_per_s < 650
    return red_text(info_str) if color and slow else info_str


def get_max_y_grid() -> int:
    return 65535


````
- **EN**: Introduces function `validate_triton_config`, function `create_bandwidth_info_str`, function `get_max_y_grid`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`validate_triton_config`、函数`create_bandwidth_info_str`、函数`get_max_y_grid`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 113-140 / 第 113-140 行
````python
try:
    import colorama

    HAS_COLORAMA = True
except ModuleNotFoundError:
    HAS_COLORAMA = False
    colorama = None  # type: ignore[assignment]


if HAS_COLORAMA:

    def _color_text(msg: str, color: str) -> str:
        # pyrefly: ignore [missing-attribute]
        return getattr(colorama.Fore, color.upper()) + msg + colorama.Fore.RESET

else:

    def _color_text(msg: str, color: str) -> str:
        return msg


def green_text(msg: str) -> str:
    return _color_text(msg, "green")


def yellow_text(msg: str) -> str:
    return _color_text(msg, "yellow")

````
- **EN**: Imports dependencies such as `colorama` for the logic in this range. Introduces function `_color_text`, function `_color_text`, function `green_text`, function `yellow_text`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `colorama` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_color_text`、函数`_color_text`、函数`green_text`、函数`yellow_text`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-168 / 第 141-168 行
````python

def red_text(msg: str) -> str:
    return _color_text(msg, "red")


def blue_text(msg: str) -> str:
    return _color_text(msg, "blue")


def get_first_attr(obj: Any, *attrs: str) -> Any:
    """
    Return the first available attribute or throw an exception if none is present.
    """
    for attr in attrs:
        if hasattr(obj, attr):
            return getattr(obj, attr)

    raise AssertionError(f"{obj} does not has any of the attributes: {attrs}")


dynamo_timed = torch._dynamo.utils.dynamo_timed  # type: ignore[has-type]


def triton_hash_to_path_key(key: str) -> str:
    # In early versions of Triton, the hash is directly used in the path name.
    # Later, the hash is converted to base64 before being used in the path name.
    # Later, the base64 conversion was replaced to the base32
    #
````
- **EN**: Introduces function `red_text`, function `blue_text`, function `get_first_attr`, function `triton_hash_to_path_key`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`red_text`、函数`blue_text`、函数`get_first_attr`、函数`triton_hash_to_path_key`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 169-196 / 第 169-196 行
````python
    # This code tries to import _base64 and falls back to _base32 if _base64 is unavailable.
    #
    # To handle this, try to import the to-base64-conversion function.
    # If it exists, use it; otherwise, try using _base32; if both are unavailable, use the hash directly.
    try:
        from triton.runtime.cache import _base64

        return _base64(key)
    except Exception:
        try:
            from triton.runtime.cache import _base32

            return _base32(key)
        except Exception:
            return key


def compile_mps_shader(source: str) -> Any:
    """
    Compiles shader source but raise more actionable error message when needed
    """
    try:
        return torch.mps.compile_shader(source)
    except SyntaxError as err:
        raise SyntaxError(f"failed to compile {source} with {err.msg}") from err


def compile_mps_shaders(
````
- **EN**: Imports dependencies such as `triton.runtime.cache` for the logic in this range. Introduces function `compile_mps_shader`, function `compile_mps_shaders`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `triton.runtime.cache` 等依赖，为后续逻辑提供基础能力。这里定义了函数`compile_mps_shader`、函数`compile_mps_shaders`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 197-224 / 第 197-224 行
````python
    kernels: list[tuple[str, str, list[str]]],
) -> dict[str, Any]:
    """Compile a batch of Metal kernels into one library.

    Args:
        kernels: list of (kernel_name, metal_source, headers) tuples.
            headers are bare names resolved as <c10/metal/{name}.h>.

    Returns:
        dict mapping each kernel_name to its compiled function handle.
    """
    from torch.utils._ordered_set import OrderedSet

    all_headers = sorted(OrderedSet(h for _, _, hs in kernels for h in hs))
    header_src = "\n".join(f"#include <c10/metal/{h}.h>" for h in all_headers)
    body_src = "\n".join(src for _, src, _ in kernels)
    lib = compile_mps_shader(header_src + "\n" + body_src)
    return {name: getattr(lib, name) for name, _, _ in kernels}


def torch_dtype_to_jax_runtime(dtype: torch.dtype) -> Any:
    """
    Map PyTorch dtype to actual JAX dtype object at runtime.

    This helper is used in generated Pallas kernels at runtime to convert
    PyTorch dtypes to JAX dtype objects (not string representations).

    Args:
````
- **EN**: Imports dependencies such as `torch.utils._ordered_set` for the logic in this range. Introduces function `torch_dtype_to_jax_runtime`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch.utils._ordered_set` 等依赖，为后续逻辑提供基础能力。这里定义了函数`torch_dtype_to_jax_runtime`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 225-252 / 第 225-252 行
````python
        dtype: PyTorch dtype to convert

    Returns:
        JAX dtype object (e.g., jnp.float32 object itself)
    """
    import jax.numpy as jnp  # pyrefly: ignore [import-error, missing-import]

    dtype_map = {
        torch.float32: jnp.float32,
        torch.float64: jnp.float64,
        torch.float16: jnp.float16,
        torch.bfloat16: jnp.bfloat16,
        torch.int32: jnp.int32,
        torch.int64: jnp.int64,
        torch.int16: jnp.int16,
        torch.int8: jnp.int8,
        torch.uint8: jnp.uint8,
        torch.bool: jnp.bool_,
        torch.complex64: jnp.complex64,
        torch.complex128: jnp.complex128,
    }
    if dtype not in dtype_map:
        raise ValueError(f"Unsupported dtype for JAX conversion: {dtype}")
    return dtype_map[dtype]


def torch_dtype_to_jax(dtype: torch.dtype) -> str:
    """
````
- **EN**: Imports dependencies such as `jax.numpy`, and `missing-import]` for the logic in this range. Introduces function `torch_dtype_to_jax`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `jax.numpy`、`missing-import]` 等依赖，为后续逻辑提供基础能力。这里定义了函数`torch_dtype_to_jax`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 253-280 / 第 253-280 行
````python
    Map PyTorch dtype to JAX dtype expression string.

    This helper is used at compile time in codegen to generate
    JAX dtype expressions for Pallas kernels.

    Args:
        dtype: PyTorch dtype to convert

    Returns:
        JAX dtype expression as string (e.g., "jnp.float32")
    """
    jax_dtype = torch_dtype_to_jax_runtime(dtype)
    dtype_name = jax_dtype.__name__
    if dtype_name == "bool":
        dtype_name = "bool_"
    return f"jnp.{dtype_name}"


def pallas_gpu_pad_inputs(inputs: list[Any], alignment: int = 128) -> list[Any]:
    """Flatten and pad each input JAX array to a multiple of alignment."""
    import jax.numpy as jnp  # pyrefly: ignore [import-error, missing-import]

    padded = []
    for inp in inputs:
        flat = inp.flatten()
        orig_size = flat.size
        aligned_size = ((orig_size + alignment - 1) // alignment) * alignment
        if orig_size != aligned_size:
````
- **EN**: Imports dependencies such as `jax.numpy`, and `missing-import]` for the logic in this range. Introduces function `pallas_gpu_pad_inputs`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `jax.numpy`、`missing-import]` 等依赖，为后续逻辑提供基础能力。这里定义了函数`pallas_gpu_pad_inputs`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 281-308 / 第 281-308 行
````python
            padded.append(jnp.pad(flat, (0, aligned_size - orig_size)))
        else:
            padded.append(flat)
    return padded


def pallas_gpu_align_output_specs(
    out_shapes: tuple[Any, ...],
    out_dtypes: tuple[Any, ...],
    alignment: int = 128,
) -> tuple[tuple[Any, ...], list[bool]]:
    """Build aligned output ShapeDtypeStruct specs for GPU kernels.

    Returns (aligned_specs, is_scalar_output) where is_scalar_output[i] is True
    when the i-th output is scalar and should not be padded/unpadded.
    """
    import jax  # pyrefly: ignore [import-error, missing-import]

    aligned_specs = []
    is_scalar = []
    for shape, dtype in zip(out_shapes, out_dtypes):
        numel = math.prod(shape)
        if numel <= 1:
            aligned_specs.append(jax.ShapeDtypeStruct(shape, dtype))
            is_scalar.append(True)
        else:
            aligned_numel = ((numel + alignment - 1) // alignment) * alignment
            aligned_specs.append(jax.ShapeDtypeStruct((aligned_numel,), dtype))
````
- **EN**: Imports dependencies such as `jax  # pyrefly: ignore [import-error`, and `missing-import]` for the logic in this range. Introduces function `pallas_gpu_align_output_specs`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里导入了 `jax  # pyrefly: ignore [import-error`、`missing-import]` 等依赖，为后续逻辑提供基础能力。这里定义了函数`pallas_gpu_align_output_specs`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 309-336 / 第 309-336 行
````python
            is_scalar.append(False)
    return tuple(aligned_specs), is_scalar


def pallas_gpu_unpad_results(
    results: Any,
    orig_shapes: tuple[Any, ...],
    is_scalar_output: list[bool] | None = None,
) -> Any:
    """Remove padding from GPU kernel results and reshape to original shapes.

    If is_scalar_output is None, all outputs are treated as non-scalar.
    """
    if not isinstance(results, tuple):
        results = (results,)
    unpadded = []
    for i, (res, shape) in enumerate(zip(results, orig_shapes)):
        if is_scalar_output is not None and is_scalar_output[i]:
            unpadded.append(res)
        else:
            orig_numel = math.prod(shape)
            unpadded.append(res[:orig_numel].reshape(shape))
    return unpadded[0] if len(unpadded) == 1 else tuple(unpadded)


# ---------------------------------------------------------------------------
# Pallas CPU / TPU tiling helpers
# ---------------------------------------------------------------------------
````
- **EN**: Introduces function `pallas_gpu_unpad_results`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`pallas_gpu_unpad_results`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 337-364 / 第 337-364 行
````python
# TPU alignment: last dim must be full or a multiple of 128,
#                second-to-last dim must be full or a multiple of 8.
_TPU_ALIGN_LAST = 128
_TPU_ALIGN_SECOND_LAST = 8


def _pallas_tile_size(
    dim: int, alignment: int, max_tile: int = 1024, is_tpu: bool = False
) -> int:
    """Pick the largest aligned tile size <= max_tile for *dim*.

    If *dim* is already <= alignment the full dimension is used (no tiling
    on this axis).
    """
    if dim == 0:
        # Tile size >= 1 avoids division by zero in JAX's _pad_to_block_dimension
        return alignment if is_tpu else 1

    if is_tpu:
        # On TPU, Mosaic requires block dimensions to perfectly align to hardware
        # registers (128 for inner, 8 for outer). We MUST pad the block spec up to
        # the next alignment boundary (Mosaic handles the OOB masking).
        if dim <= alignment:
            return alignment
        t = min(max_tile, dim)
        # Use ceiling division to ensure the tile covers the dimension or aligns upward
        t = ((t + alignment - 1) // alignment) * alignment
        return t
````
- **EN**: Introduces function `_pallas_tile_size`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_pallas_tile_size`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 365-392 / 第 365-392 行
````python

    if dim <= alignment:
        return dim
    t = min(max_tile, dim)
    t = (t // alignment) * alignment
    return max(alignment, t)


def pallas_permute(x, perm):
    """Permute array x according to perm, working around Mosaic TPU bugs.

    Mosaic's jnp.permute_dims produces corrupted output for certain 3D+
    permutations at large sizes.  Decomposes into: loop over smallest output
    dim via unrolled Python loop + stack, with recursive sub-permutation.
    At the 2D base case, uses jnp.permute_dims (which works on Mosaic).

    Callers should ensure tiles are small enough via tiling so that the
    Mosaic bug doesn't trigger (threshold ~ 256K elements).
    """
    import jax.numpy as jnp  # pyrefly: ignore [import-error, missing-import]

    ndim = len(perm)
    if ndim <= 2:
        return jnp.permute_dims(x, perm)
    if perm == tuple(range(ndim)):
        return x  # identity

    shape = x.shape
````
- **EN**: Imports dependencies such as `jax.numpy`, and `missing-import]` for the logic in this range. Introduces function `pallas_permute`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `jax.numpy`、`missing-import]` 等依赖，为后续逻辑提供基础能力。这里定义了函数`pallas_permute`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 393-420 / 第 393-420 行
````python
    out_shape = tuple(shape[p] for p in perm)

    # Choose the output dimension with the smallest size to loop over.
    loop_out_dim = min(range(ndim), key=lambda d: out_shape[d])
    loop_size = out_shape[loop_out_dim]
    loop_in_dim = perm[loop_out_dim]

    # Build the sub-permutation for the remaining (ndim-1) dimensions.
    remaining_in_dims = [d for d in range(ndim) if d != loop_in_dim]
    remaining_out_perm_raw = [perm[d] for d in range(ndim) if d != loop_out_dim]
    dim_map = {old: new for new, old in enumerate(remaining_in_dims)}
    sub_perm = tuple(dim_map[d] for d in remaining_out_perm_raw)

    # Unrolled loop: extract slices with static indices, apply sub-perm,
    # then stack along the loop output dimension.
    slices = []
    # pyrefly: ignore [bad-argument-type]
    for i in range(loop_size):
        idx: list[Any] = [slice(None)] * ndim
        idx[loop_in_dim] = i
        slc = x[tuple(idx)]  # static index, removes loop_in_dim
        slc_p = pallas_permute(slc, sub_perm)
        slices.append(slc_p)

    return jnp.stack(slices, axis=loop_out_dim)


# Maximum tile elements for permutation kernels.  Mosaic's
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `out_shape`, `loop_out_dim`, `loop_size`, `loop_in_dim`, `remaining_in_dims`, `remaining_out_perm_raw`, and `...+6`. This range continues the implementation of function `pallas_permute`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `out_shape`、`loop_out_dim`、`loop_size`、`loop_in_dim`、`remaining_in_dims`、`remaining_out_perm_raw`、`另有6项` 等值。这一段延续了函数`pallas_permute` 的具体实现。

### Lines 421-448 / 第 421-448 行
````python
# jnp.permute_dims produces corrupted output above ~1.5M elements for
# certain 3D+ permutations.  Keep tiles well under that threshold.
_PERMUTE_MAX_TILE_ELEMS = 1 << 18  # ~256K elements


def pallas_compute_tiling(
    ref_shape: tuple[int, ...],
    transpose: bool = False,  # constrain tile size for pallas_permute VMEM
    skip_last_n: int = 0,
    exact_only: bool = False,
    is_tpu: bool = False,
    permutations: list[tuple[int, ...]] | None = None,
    max_grid_product: int | None = None,
) -> tuple[tuple[int, ...], tuple[int, ...], dict[int, int]]:
    """Compute tile shape, grid and axis→grid-dim mapping for CPU/TPU.

    Always uses TPU-compatible alignment (last dim multiple of 128,
    second-to-last multiple of 8) so that the same generated kernel works
    on both CPU-interpret and real TPU.

    *transpose* constrains tile sizes to keep VMEM usage within safe
    limits for pallas_permute (gather-based permutation).

    *skip_last_n* prevents tiling the last N dimensions (used when those
    dims correspond to internal reduction ranges that must remain full).

    *exact_only* restricts tiling to dimensions that divide evenly by the
    tile size (no remainder blocks).  Required on TPU where Mosaic needs
````
- **EN**: Introduces function `pallas_compute_tiling`. Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `_PERMUTE_MAX_TILE_ELEMS`, `ref_shape`, `transpose`, `skip_last_n`, `exact_only`, `is_tpu`, and `...+2`.
- **CN**: 这里定义了函数`pallas_compute_tiling`。处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `_PERMUTE_MAX_TILE_ELEMS`、`ref_shape`、`transpose`、`skip_last_n`、`exact_only`、`is_tpu`、`另有2项` 等值。

### Lines 449-476 / 第 449-476 行
````python
    block shapes to match the XLA memory layout.

    Returns ``(tile_shape, grid, axis_to_grid)`` where *axis_to_grid*
    maps each tiled reference-shape axis index to its position in the
    grid tuple.

    When no dimension benefits from tiling the grid is ``(1,)`` and the
    tile covers the full tensor.
    """
    nd = len(ref_shape)
    if nd == 0:
        return (), (1,), {}

    # Effective number of dims eligible for tiling
    tileable_nd = nd - skip_last_n

    tile = list(ref_shape)
    grid_parts: list[int] = []
    axis_to_grid: dict[int, int] = {}  # ref axis → grid dim

    # Pick alignment based on the physical position of the axis in the
    # tensor, not its position in the tileable subset.  The TPU requires
    # the physical last dim to be a multiple of 128 and the physical
    # second-to-last dim to be a multiple of 8.
    #
    # When permutations are present, an output dim `ax` maps to input
    # dim `perm[ax]`.  If `perm[ax]` is the input's last dim, that
    # output dim must also satisfy _TPU_ALIGN_LAST.  Compute the
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `nd`, `tileable_nd`, `tile`, `grid_parts`, and `axis_to_grid`. This range continues the implementation of function `pallas_compute_tiling`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `nd`、`tileable_nd`、`tile`、`grid_parts`、`axis_to_grid` 等值。这一段延续了函数`pallas_compute_tiling` 的具体实现。

### Lines 477-504 / 第 477-504 行
````python
    # strictest alignment across all buffers.
    def _input_align(input_ax: int, input_nd: int) -> int:
        return _TPU_ALIGN_LAST if input_ax == input_nd - 1 else _TPU_ALIGN_SECOND_LAST

    def _align(ax: int) -> int:
        out_align = _TPU_ALIGN_LAST if ax == nd - 1 else _TPU_ALIGN_SECOND_LAST
        if permutations:
            for perm in permutations:
                if perm is not None and len(perm) == nd:
                    in_ax = perm[ax]
                    in_align = _input_align(in_ax, len(perm))
                    out_align = max(out_align, in_align)
        return out_align

    def _can_tile_ax(ax: int, dim: int, t: int) -> bool:
        """Check if tiling dim to t is valid."""
        if t >= dim:
            # For TPU padding, we allow tiles >= dimension for the aligned axes
            if _align(ax) == _TPU_ALIGN_LAST or _align(ax) == _TPU_ALIGN_SECOND_LAST:
                return True
            return False
        if exact_only and dim % t != 0:
            if _align(ax) == _TPU_ALIGN_LAST or _align(ax) == _TPU_ALIGN_SECOND_LAST:
                # TPU DMA `#tpu.element_window` natively masks out-of-bounds remainder tiles
                return True
            return False
        return True

````
- **EN**: Introduces function `_input_align`, function `_align`, function `_can_tile_ax`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `out_align`, `in_ax`, and `in_align`.
- **CN**: 这里定义了函数`_input_align`、函数`_align`、函数`_can_tile_ax`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `out_align`、`in_ax`、`in_align` 等值。

### Lines 505-532 / 第 505-532 行
````python
    # Second-to-last tileable dim (added first so it becomes grid dim 0)
    if tileable_nd >= 2:
        ax = tileable_nd - 2
        t = _pallas_tile_size(ref_shape[ax], _align(ax), is_tpu=is_tpu)
        if _can_tile_ax(ax, ref_shape[ax], t):
            tile[ax] = t
            axis_to_grid[ax] = len(grid_parts)
            grid_parts.append((ref_shape[ax] + t - 1) // t)

    # Last tileable dim
    if tileable_nd >= 1:
        ax = tileable_nd - 1
        t = _pallas_tile_size(ref_shape[ax], _align(ax), is_tpu=is_tpu)
        if _can_tile_ax(ax, ref_shape[ax], t):
            tile[ax] = t
            axis_to_grid[ax] = len(grid_parts)
            grid_parts.append((ref_shape[ax] + t - 1) // t)

    # When transpose is active, tile dimensions to keep the total tile
    # elements below the safe threshold for Mosaic's permute_dims.
    # Mosaic pads the last dimension to _TPU_ALIGN_LAST (128 for f32),
    # so when the last dim is small (e.g. 2), VMEM usage is much higher
    # than the logical tile size.  We compute an effective element count
    # that accounts for this padding.
    if transpose:
        tile_elems = 1
        for t in tile:
            tile_elems *= t
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ax`, `t`, and `tile_elems`. This range continues the implementation of function `pallas_compute_tiling`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `ax`、`t`、`tile_elems` 等值。这一段延续了函数`pallas_compute_tiling` 的具体实现。

### Lines 533-560 / 第 533-560 行
````python
        last_dim = tile[nd - 1] if nd > 0 else 1
        padded_last = max(last_dim, _TPU_ALIGN_LAST)
        # pallas_permute decomposes ndim>2 permutations by looping over
        # the smallest dim and recursing, creating (ndim-1)-D intermediates
        # at each level.  Mosaic pads the last physical dim of each
        # intermediate to _TPU_ALIGN_LAST (128), which can cause VMEM OOM
        # when a small dim (e.g. 4) ends up as the last dim.
        #
        # For ndim >= 3, compute effective_max from the worst-case VMEM
        # of the (ndim-1)-D intermediate created by pallas_permute.
        # The intermediate removes the loop dim (smallest output dim) and
        # may have a small dim as its last physical dim -> padded to 128.
        if nd >= 4:
            # For 4D+, pallas_permute creates (nd-1)-D intermediates
            # that can have small dims as their last physical dim,
            # which Mosaic pads to _TPU_ALIGN_LAST (128).  This can
            # cause VMEM OOM.  (3D pallas_permute only creates 2D
            # intermediates which Mosaic handles efficiently.)
            sorted_dims = sorted(tile)
            # The loop dim is the smallest; intermediate has the rest.
            inter_dims = sorted_dims[1:]  # (nd-1) dims, sorted
            smallest_inter = inter_dims[0]
            # Padded intermediate: smallest remaining dim padded to 128.
            padded_inter_elems = 1
            for d in inter_dims:
                padded_inter_elems *= d
            pad_factor = max(smallest_inter, _TPU_ALIGN_LAST) // max(smallest_inter, 1)
            padded_inter_elems *= pad_factor
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `last_dim`, `padded_last`, `sorted_dims`, `inter_dims`, `smallest_inter`, `padded_inter_elems`, and `...+1`. This range continues the implementation of function `pallas_compute_tiling`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `last_dim`、`padded_last`、`sorted_dims`、`inter_dims`、`smallest_inter`、`padded_inter_elems`、`另有1项` 等值。这一段延续了函数`pallas_compute_tiling` 的具体实现。

### Lines 561-588 / 第 561-588 行
````python
            padded_inter_bytes = padded_inter_elems * 4
            # Conservative VMEM budget: 2MB per intermediate buffer.
            # pallas_permute creates multiple live intermediates (input
            # slices, sub-permutation results, stack temps).  Empirically,
            # 2MB per worst-case intermediate avoids OOM on TPU v4 (64MB VMEM).
            vmem_per_buf = 2 * 1024 * 1024  # 2MB
            if padded_inter_bytes > vmem_per_buf:
                scale = vmem_per_buf / padded_inter_bytes
                effective_max = max(int(tile_elems * scale), 1024)
            else:
                effective_max = _PERMUTE_MAX_TILE_ELEMS
        else:
            # For ndim <= 3: pallas_permute creates 2D intermediates
            # (fine) but jnp.stack creates a 3D result with the same
            # shape as the output.  If the last dim is small, Mosaic
            # pads it to _TPU_ALIGN_LAST (128), which can exhaust VMEM.
            # Empirically: 8MB per padded buffer keeps us safe on TPU v4
            # (64MB VMEM, multiple buffers live simultaneously).
            vmem_per_buf = 8 * 1024 * 1024  # 8MB
            max_nonlast_product = vmem_per_buf // (padded_last * 4)
            effective_max = min(
                max_nonlast_product * last_dim,
                _PERMUTE_MAX_TILE_ELEMS,
            )
        effective_max = min(effective_max, _PERMUTE_MAX_TILE_ELEMS)
        effective_max = max(effective_max, 1024)

        if tile_elems > effective_max:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `padded_inter_bytes`, `vmem_per_buf`, `scale`, `effective_max`, `else`, and `max_nonlast_product`. This range continues the implementation of function `pallas_compute_tiling`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `padded_inter_bytes`、`vmem_per_buf`、`scale`、`effective_max`、`else`、`max_nonlast_product` 等值。这一段延续了函数`pallas_compute_tiling` 的具体实现。

### Lines 589-616 / 第 589-616 行
````python
            # Collect tileable dims.  For ndim >= 4 with VMEM-constrained
            # intermediates, include ALL tileable dims (even the last)
            # since we may need to shrink every dim to fit in VMEM.
            # For ndim <= 3, exclude the last dim (it's often small
            # and already at minimum).
            if nd >= 4:
                tileable_axes = [
                    ax for ax in range(tileable_nd - 1, -1, -1) if ref_shape[ax] > 1
                ]
            else:
                tileable_axes = [
                    ax for ax in range(tileable_nd - 2, -1, -1) if ref_shape[ax] > 1
                ]
            # Distribute reduction proportionally across dims.
            reduction = tile_elems / effective_max
            n = len(tileable_axes)
            per_dim = reduction ** (1.0 / max(n, 1))

            for ax in tileable_axes:
                if tile_elems <= effective_max:
                    break
                dim = ref_shape[ax]
                cur_t = tile[ax]
                align = _align(ax)
                # Target tile: proportional reduction, but also check
                # against the remaining budget for subsequent dims.
                target = int(cur_t / per_dim)
                remaining_budget = effective_max * cur_t // tile_elems
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tileable_axes`, `else`, `reduction`, `n`, `per_dim`, `dim`, and `...+4`. This range continues the implementation of function `pallas_compute_tiling`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `tileable_axes`、`else`、`reduction`、`n`、`per_dim`、`dim`、`另有4项` 等值。这一段延续了函数`pallas_compute_tiling` 的具体实现。

### Lines 617-644 / 第 617-644 行
````python
                target = min(target, remaining_budget)
                target = (target // align) * align
                target = max(target, align)
                # Find largest aligned divisor of dim <= target.
                best_t = cur_t
                for candidate in range(target, 0, -align):
                    if candidate >= cur_t:
                        continue
                    if dim % candidate != 0:
                        continue
                    best_t = candidate
                    break
                if best_t < cur_t:
                    if ax in axis_to_grid:
                        grid_parts[axis_to_grid[ax]] = dim // best_t
                    else:
                        axis_to_grid[ax] = len(grid_parts)
                        grid_parts.append(dim // best_t)
                    tile_elems = tile_elems * best_t // cur_t
                    tile[ax] = best_t

    grid = tuple(grid_parts) if grid_parts else (1,)

    # Enforce max_grid_product by scaling up tiles on tiled axes.
    if max_grid_product is not None and grid_parts:
        grid_product = 1
        for g in grid_parts:
            grid_product *= g
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `target`, `best_t`, `else`, `tile_elems`, `grid`, and `grid_product`. This range continues the implementation of function `pallas_compute_tiling`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `target`、`best_t`、`else`、`tile_elems`、`grid`、`grid_product` 等值。这一段延续了函数`pallas_compute_tiling` 的具体实现。

### Lines 645-672 / 第 645-672 行
````python
        if grid_product > max_grid_product:
            # Sort tiled axes by their grid contribution (descending)
            # and increase tiles to reduce the grid.
            tiled_axes = sorted(
                axis_to_grid.keys(),
                key=lambda ax: grid_parts[axis_to_grid[ax]],
                reverse=True,
            )
            for ax in tiled_axes:
                if grid_product <= max_grid_product:
                    break
                gi = axis_to_grid[ax]
                cur_grid = grid_parts[gi]
                cur_tile = tile[ax]
                dim = ref_shape[ax]
                align = _align(ax)
                # Target grid for this axis: proportional share of reduction
                target_grid = max(1, int(cur_grid * max_grid_product / grid_product))
                target_tile = (dim + target_grid - 1) // target_grid
                # Round up to alignment
                target_tile = ((target_tile + align - 1) // align) * align
                target_tile = min(target_tile, dim)
                # Find smallest aligned value >= target_tile that divides dim
                # (or just use the rounded-up value if exact_only is False)
                best_tile = dim  # fallback: full dim
                if exact_only:
                    for candidate in range(target_tile, dim + 1, align):
                        if dim % candidate == 0:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tiled_axes`, `key`, `reverse`, `gi`, `cur_grid`, `cur_tile`, and `...+5`. This range continues the implementation of function `pallas_compute_tiling`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `tiled_axes`、`key`、`reverse`、`gi`、`cur_grid`、`cur_tile`、`另有5项` 等值。这一段延续了函数`pallas_compute_tiling` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python
                            best_tile = candidate
                            break
                else:
                    best_tile = target_tile
                if best_tile > cur_tile:
                    new_grid = (dim + best_tile - 1) // best_tile
                    grid_product = grid_product * new_grid // cur_grid
                    grid_parts[gi] = new_grid
                    tile[ax] = best_tile
            grid = tuple(grid_parts)

    return tuple(tile), grid, axis_to_grid


def pallas_make_block_spec(
    buf_shape: tuple[int, ...],
    ref_shape: tuple[int, ...],
    tile_shape: tuple[int, ...],
    axis_to_grid: dict[int, int],
    n_grid: int,
    permutation: tuple[int, ...] | None = None,
    is_output: bool = False,
) -> Any:
    """Build a ``pl.BlockSpec`` for *buf_shape* given tiling of *ref_shape*.

    Lower-ndim buffers are right-aligned with the reference shape (numpy
    broadcast rules).  Dimensions that match a tiled reference dimension
    are tiled; broadcast dimensions (size 1 or absent) are kept full.
````
- **EN**: Introduces function `pallas_make_block_spec`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `best_tile`, `else`, `new_grid`, `grid_product`, `grid`, `buf_shape`, and `...+6`.
- **CN**: 这里定义了函数`pallas_make_block_spec`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `best_tile`、`else`、`new_grid`、`grid_product`、`grid`、`buf_shape`、`另有6项` 等值。

### Lines 701-728 / 第 701-728 行
````python

    When *buf_nd > ref_nd* (reduction inputs), we find an alignment offset
    so the ref dims map into the buffer.  Extra dims are kept at full size
    with index 0 in the index_map.

    When *permutation* is given (a tuple mapping ref axis -> buf axis),
    the buffer dimensions are accessed in permuted order relative to the
    reference shape.  For example, permutation=(1, 0) swaps the last two.

    When *is_output* is True and *buf_nd < ref_nd*, left-alignment is used
    as a fallback (for reduction outputs whose trailing dims were reduced).
    """
    from jax.experimental import (  # pyrefly: ignore [import-error, missing-import]
        pallas as pl,
    )

    buf_nd = len(buf_shape)
    ref_nd = len(ref_shape)

    if buf_nd == 0:
        # Scalar — untouched regardless of grid shape.
        return pl.BlockSpec((1,), _make_index_map([], 1, n_grid))

    bs = list(buf_shape)
    tiled_pairs: list[tuple[int, int]] = []

    if buf_nd > ref_nd:
        # Reduction input: find alignment offset k where ref dims map into buf.
````
- **EN**: Imports dependencies such as `jax.experimental` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `buf_nd`, `ref_nd`, `bs`, and `tiled_pairs`.
- **CN**: 这里导入了 `jax.experimental` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `buf_nd`、`ref_nd`、`bs`、`tiled_pairs` 等值。

### Lines 729-756 / 第 729-756 行
````python
        align_k = 0
        for k in range(buf_nd - ref_nd + 1):
            ok = True
            for i in range(ref_nd):
                if ref_shape[i] == 1:
                    continue
                if buf_shape[k + i] != ref_shape[i]:
                    ok = False
                    break
            if ok:
                align_k = k
                break

        for ref_ax, grid_dim in axis_to_grid.items():
            buf_ax = align_k + ref_ax
            if 0 <= buf_ax < buf_nd and buf_shape[buf_ax] == ref_shape[ref_ax]:
                bs[buf_ax] = tile_shape[ref_ax]
                tiled_pairs.append((buf_ax, grid_dim))

    elif permutation is not None and buf_nd == ref_nd:
        # Permuted buffer: map ref axes through the permutation.
        for ref_ax, grid_dim in axis_to_grid.items():
            buf_ax = permutation[ref_ax]
            if 0 <= buf_ax < buf_nd and buf_shape[buf_ax] == ref_shape[ref_ax]:
                bs[buf_ax] = tile_shape[ref_ax]
                tiled_pairs.append((buf_ax, grid_dim))

    else:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `align_k`, `ok`, `buf_ax`, and `else`. This range continues the implementation of function `pallas_make_block_spec`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `align_k`、`ok`、`buf_ax`、`else` 等值。这一段延续了函数`pallas_make_block_spec` 的具体实现。

### Lines 757-784 / 第 757-784 行
````python
        # Standard right-alignment, with left-alignment fallback for
        # reduction outputs (e.g. sum(dim=-1) on (10,10) → (10,)).
        for ref_ax, grid_dim in axis_to_grid.items():
            buf_ax = ref_ax - (ref_nd - buf_nd)
            if 0 <= buf_ax < buf_nd and buf_shape[buf_ax] == ref_shape[ref_ax]:
                bs[buf_ax] = tile_shape[ref_ax]
                tiled_pairs.append((buf_ax, grid_dim))
            elif (
                is_output
                and buf_nd < ref_nd
                and 0 <= ref_ax < buf_nd
                and buf_shape[ref_ax] == ref_shape[ref_ax]
            ):
                # Left-aligned match for output: buf dim i matches ref dim i
                # (reduction output whose trailing dims were reduced away)
                bs[ref_ax] = tile_shape[ref_ax]
                tiled_pairs.append((ref_ax, grid_dim))

    return pl.BlockSpec(
        tuple(bs),
        _make_index_map(tiled_pairs, buf_nd, n_grid),
    )


def _make_index_map(
    tiled_pairs: list[tuple[int, int]],
    buf_nd: int,
    n_grid: int,
````
- **EN**: Introduces function `_make_index_map`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `buf_ax`, `tiled_pairs`, `buf_nd`, and `n_grid`.
- **CN**: 这里定义了函数`_make_index_map`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `buf_ax`、`tiled_pairs`、`buf_nd`、`n_grid` 等值。

### Lines 785-812 / 第 785-812 行
````python
) -> Any:
    """Return an index_map callable for ``pl.BlockSpec``.

    *tiled_pairs* is a list of ``(buf_axis, grid_dim)`` indicating which
    buffer axes receive a grid index.  All other axes return 0 (full block).

    All returned values are explicitly ``jnp.int32`` so that TPU Mosaic
    lowering (which rejects 64-bit types) works when ``jax_enable_x64`` is
    active.  The casts are created inside the function body (not captured)
    to satisfy JAX's "index_map must not capture constants" rule.
    """
    import jax.numpy as jnp  # pyrefly: ignore [import-error, missing-import]

    # Pre-build the mapping so the returned lambda is a plain lookup.
    mapping = dict(tiled_pairs)

    if n_grid == 0 or (n_grid == 1 and not mapping):
        return lambda _i: tuple(jnp.int32(0) for _ in range(buf_nd))

    def index_map(*grid_args):
        return tuple(
            jnp.int32(grid_args[mapping[d]]) if d in mapping else jnp.int32(0)
            for d in range(buf_nd)
        )

    return index_map


````
- **EN**: Imports dependencies such as `jax.numpy`, and `missing-import]` for the logic in this range. Introduces function `index_map`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `jax.numpy`、`missing-import]` 等依赖，为后续逻辑提供基础能力。这里定义了函数`index_map`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 813-829 / 第 813-829 行
````python
def pallas_ensure_nonzero_rank(x: torch.Tensor) -> torch.Tensor:
    if len(x.shape) == 0:
        return x.reshape((1,))
    return x


def pallas_make_block_spec_non_tiled(shape: tuple[int, ...]) -> Any:
    import jax.numpy as jnp  # pyrefly: ignore [import-error, missing-import]
    from jax.experimental import (  # pyrefly: ignore [import-error, missing-import]
        pallas as pl,
    )

    nonzero_rank_shape = shape if len(shape) > 0 else (1,)
    return pl.BlockSpec(
        nonzero_rank_shape,
        lambda i: [jnp.int32(i)] * len(nonzero_rank_shape),
    )
````
- **EN**: Imports dependencies such as `jax.numpy`, `missing-import]`, and `jax.experimental` for the logic in this range. Introduces function `pallas_ensure_nonzero_rank`, function `pallas_make_block_spec_non_tiled`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `jax.numpy`、`missing-import]`、`jax.experimental` 等依赖，为后续逻辑提供基础能力。这里定义了函数`pallas_ensure_nonzero_rank`、函数`pallas_make_block_spec_non_tiled`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

## Key Concepts / 关键概念
- **EN**: Provides runtime helpers used after compilation  
  **CN**: 提供编译完成后使用的运行期辅助逻辑
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `conditional_product`, `ceildiv`, `is_power_of_2`, `next_power_of_2`, `last_power_of_2`, `get_num_bytes`, and `...+24`  
  **CN**: 主要函数：`conditional_product`、`ceildiv`、`is_power_of_2`、`next_power_of_2`、`last_power_of_2`、`get_num_bytes`、`另有24项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `functools`, `math`, `operator`, `typing`, `collections.abc`
- **Third-party / 第三方**: `sympy`, `colorama`, `jax.numpy`, `jax`, `jax.experimental`, `triton.runtime.cache`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._dynamo`, `torch._inductor.runtime.cache_dir_utils`, `.triton_compat`, `torch.utils._ordered_set`
