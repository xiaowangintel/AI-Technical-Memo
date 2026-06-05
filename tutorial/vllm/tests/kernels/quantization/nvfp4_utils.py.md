# nvfp4_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/nvfp4_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for kernel tests in kernels / quantization / nvfp4_utils; it provides reference logic, fixtures, or utilities reused by multiple scenarios. / kernels / quantization / nvfp4_utils 对应的内核测试支撑模块；它提供会被多个场景复用的参考逻辑、fixture 或工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-6)
```python
import torch

from vllm._custom_ops import scaled_fp4_quant
from vllm.scalar_type import scalar_types
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as torch; and vLLM components like vllm._custom_ops, vllm.scalar_type.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 torch；vLLM 内部组件，例如 vllm._custom_ops、vllm.scalar_type。

### Constants and module state (lines 8-13)
```python
FLOAT4_E2M1_MAX = scalar_types.float4_e2m1f.max()
FLOAT8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max

kE2M1ToFloat = torch.tensor(
    [0.0, 0.5, 1.0, 1.5, 2.0, 3.0, 4.0, 6.0], dtype=torch.float32
)
```
**EN:** This block centralizes shared constants and parameter grids, including FLOAT4_E2M1_MAX, FLOAT8_E4M3_MAX. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FLOAT4_E2M1_MAX、FLOAT8_E4M3_MAX。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `convert_swizzled_to_linear` (lines 16-23)
```python
def convert_swizzled_to_linear(a_sf_swizzled: torch.Tensor, m, k, block_size):
    m_tiles = (m + 128 - 1) // 128
    f = block_size * 4
    k_tiles = (k + f - 1) // f
    tmp = torch.reshape(a_sf_swizzled, (1, m_tiles, k_tiles, 32, 4, 4))
    tmp = torch.permute(tmp, (0, 1, 4, 3, 2, 5))
    out = tmp.reshape(m_tiles * 128, k_tiles * f // block_size)
    return out[0:m, 0:k]
```
**EN:** This helper function implements the shared logic for convert swizzled to linear. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 convert swizzled to linear 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `convert_swizzled_8x4_layout_to_linear` (lines 26-35)
```python
def convert_swizzled_8x4_layout_to_linear(
    a_sf_swizzled: torch.Tensor, m, k, block_size
):
    m_tiles = (m + 8 - 1) // 8
    f = block_size * 4
    k_tiles = (k + f - 1) // f
    tmp = torch.reshape(a_sf_swizzled, (1, m_tiles, k_tiles, 8, 4))
    tmp = torch.permute(tmp, (0, 1, 3, 2, 4))
    out = tmp.reshape(m_tiles * 8, k_tiles * f // block_size)
    return out[0:m, 0:k]
```
**EN:** This helper function implements the shared logic for convert swizzled 8x4 layout to linear. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 convert swizzled 8x4 layout to linear 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `dequantize_nvfp4_to_dtype` (lines 38-64)
```python
def dequantize_nvfp4_to_dtype(
    tensor_fp4,
    tensor_sf,
    global_scale,
    dtype,
    device,
    block_size=16,
    is_sf_128x4_layout=True,
):
    """Dequantize the fp4 tensor back to high precision."""
    # Two fp4 values are packed into one uint8.
    assert tensor_fp4.dtype == torch.uint8
    m, packed_k = tensor_fp4.shape
    k = packed_k * 2
    tensor_f32 = break_fp4_bytes(tensor_fp4, dtype)
    tensor_f32 = tensor_f32.reshape(m, k // block_size, block_size)
    tensor_sf = tensor_sf.view(torch.float8_e4m3fn)
    if is_sf_128x4_layout:
        tensor_sf = convert_swizzled_to_linear(tensor_sf, m, k, block_size)
    else:
        tensor_sf = convert_swizzled_8x4_layout_to_linear(tensor_sf, m, k, block_size)

    tensor_sf_dtype = tensor_sf.to(torch.float32) / global_scale

    # scale the tensor
    out = (tensor_f32 * tensor_sf_dtype.unsqueeze(-1)).reshape(m, k)
    return out.to(dtype=dtype)
```
**EN:** This helper function implements the shared logic for dequantize nvfp4 to dtype. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 dequantize nvfp4 to dtype 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `break_fp4_bytes` (lines 67-88)
```python
def break_fp4_bytes(a, dtype):
    assert a.dtype == torch.uint8
    m, n = a.shape

    # Vectorized nibble processing
    a_flat = a.flatten()
    high = (a_flat & 0xF0) >> 4  # Upper nibbles
    low = a_flat & 0x0F  # Lower nibbles

    # Combine nibbles for batch processing
    combined = torch.stack((low, high), dim=1).flatten()

    # Vectorized sign and magnitude extraction
    signs = (combined & 0x08).to(torch.bool)  # Sign bits
    abs_vals = (combined & 0x07).to(torch.long)  # Magnitude indices

    # Device-aware lookup and sign application
    kE2M1 = kE2M1ToFloat.to(device=a.device)
    values = kE2M1[abs_vals] * torch.where(signs, -1.0, 1.0)

    # Reshape to final form
    return values.reshape(m, n * 2).to(dtype=dtype)
```
**EN:** This helper function implements the shared logic for break fp4 bytes. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 break fp4 bytes 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `dequant_nvfp4_kv_cache` (lines 91-142)
```python
def dequant_nvfp4_kv_cache(
    fp4_data: torch.Tensor,
    block_scale: torch.Tensor,
    global_scale: float,
    head_size: int,
    block_size: int,
) -> torch.Tensor:
    """Dequantize an NVFP4 KV cache with 4x4-swizzled block scales.

    The input must be in HND layout so that the last two dims are
    (block_size, last_dim).  For NHD caches, permute to HND first.

    Args:
        fp4_data: [..., num_heads, block_size, head_size//2] uint8 packed fp4.
        block_scale: [..., num_heads, block_size, head_size//16] fp8 block
            scales (as uint8 or float8_e4m3fn).
        global_scale: checkpoint dequant scale (k_scale or v_scale).
        head_size: head dimension.
        block_size: page size.

    Returns:
        [..., num_heads, block_size, head_size] float32.
    """
    data_dim = head_size // 2
    scale_dim = head_size // 16

    fp4_packed = fp4_data
    sf_swizzled = block_scale.view(torch.uint8)

    # Unswizzle 4x4 block scales on (block_size, scale_dim) plane.
    # [..., T, S] → [..., T//4, 4, sg, 4] → permute → [..., T, S]
    batch_shape = sf_swizzled.shape[:-2]
    T, S = block_size, scale_dim
    sg = S // 4
    sf_reshape = sf_swizzled.reshape(*batch_shape, T // 4, 4, sg, 4)
    ndim = sf_reshape.ndim
    # Swap the last four dims: (..., T//4, 4, sg, 4) → (..., T//4, 4, 4, sg)
    perm = list(range(ndim - 4)) + [ndim - 4, ndim - 1, ndim - 3, ndim - 2]
    sf_linear = sf_reshape.permute(*perm).reshape(*batch_shape, T, S)
    sf_f32 = sf_linear.view(torch.float8_e4m3fn).to(torch.float32)

    # Unpack fp4
    shape = fp4_packed.shape  # [..., T, data_dim]
    fp4_flat = fp4_packed.reshape(-1, data_dim)
    fp4_vals = break_fp4_bytes(fp4_flat, torch.float32)
    fp4_vals = fp4_vals.reshape(*shape[:-1], head_size)

    # Dequant: fp4_val * block_scale * global_scale per 16-element group
    return (
        fp4_vals.reshape(*shape[:-1], scale_dim, 16)
        * (sf_f32 * global_scale).unsqueeze(-1)
    ).reshape(*shape[:-1], head_size)
```
**EN:** This helper function implements the shared logic for dequant nvfp4 KV cache. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 dequant nvfp4 KV cache 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `get_nvfp4_global_scale` (lines 145-146)
```python
def get_nvfp4_global_scale(a: torch.Tensor):
    return (FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX) / torch.abs(a).max().to(torch.float32)
```
**EN:** This helper function implements the shared logic for nvfp4 global scale. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 nvfp4 global scale 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `quant_nvfp4_tensor` (lines 149-152)
```python
def quant_nvfp4_tensor(a: torch.Tensor):
    a_global_scale = get_nvfp4_global_scale(a)
    a_quant, a_block_scale = scaled_fp4_quant(a, a_global_scale)
    return a_quant, a_block_scale, a_global_scale
```
**EN:** This helper function implements the shared logic for quant nvfp4 tensor. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 quant nvfp4 tensor 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `torch`
- `vllm._custom_ops -> scaled_fp4_quant`
- `vllm.scalar_type -> scalar_types`
