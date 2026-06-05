# nvfp4_emulation_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/nvfp4_emulation_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-9)
```python
from types import SimpleNamespace

import torch

from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
from vllm.triton_utils import tl, triton
```
**EN:** This opening block pulls in external dependencies such as `types`, `torch` and internal modules such as `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `types`, `torch`）以及内部模块（如 `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 11-22)
```python
__all__ = [
    "break_fp4_bytes",
    "dequantize_to_dtype",
    "ref_nvfp4_quant",
]

FLOAT4_E2M1_MAX = scalar_types.float4_e2m1f.max()
FLOAT4_E2M1_MAX_RECIPROCAL = 1 / FLOAT4_E2M1_MAX

kE2M1ToFloat_handle = SimpleNamespace(
    val=torch.tensor([0.0, 0.5, 1.0, 1.5, 2.0, 3.0, 4.0, 6.0], dtype=torch.float32)
)
```
**EN:** This block defines module-level metadata or constants such as `__all__`, `FLOAT4_E2M1_MAX`, `FLOAT4_E2M1_MAX_RECIPROCAL`, `kE2M1ToFloat_handle`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `__all__`, `FLOAT4_E2M1_MAX`, `FLOAT4_E2M1_MAX_RECIPROCAL`, `kE2M1ToFloat_handle`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `_dequantize_nvfp4_kernel` (lines 51-128)
```python
def _dequantize_nvfp4_kernel(
    fp4_ptr,
    scale_ptr,
    global_scale_ptr,
    output_ptr,
    rows_per_batch: tl.constexpr,
    num_blocks: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
    has_batch_global_scale: tl.constexpr,
    TILE_BLOCKS: tl.constexpr,
):
    """Triton kernel for NVFP4 dequantization (swizzle=False).

    Optimized with 2D tile processing + interleave for coalesced stores.
    """
    BLOCK_PACKED: tl.constexpr = BLOCK_SIZE // 2

    row_idx = tl.program_id(0)
    tile_idx = tl.program_id(1)

    if has_batch_global_scale:
        batch_idx = row_idx // rows_per_batch
        global_scale = tl.load(global_scale_ptr + batch_idx).to(tl.float32)
    else:
        global_scale = tl.load(global_scale_ptr).to(tl.float32)

    fp4_row_offset = row_idx * num_blocks * BLOCK_PACKED
    scale_row_offset = row_idx * num_blocks
    output_row_offset = row_idx * num_blocks * BLOCK_SIZE

    start_block = tile_idx * TILE_BLOCKS

    # Load scales for this tile: [TILE_BLOCKS]
    block_offsets = tl.arange(0, TILE_BLOCKS)
    block_mask = (start_block + block_offsets) < num_blocks

    raw_scales = tl.load(
        scale_ptr + scale_row_offset + start_block + block_offsets,
# ... truncated for analysis ...
    high_val = _e2m1_inline(high_mag)
    high_sign = (high_nibble >> 3) & 1
    high_result = tl.where(high_sign == 1, -high_val, high_val) * scale_values

    # Interleave for coalesced contiguous store
    result = tl.interleave(low_result, high_result)

    elem_offsets = tl.arange(0, BLOCK_SIZE)[None, :]
    out_indices = (
        output_row_offset
        + (start_block + block_offsets[:, None]) * BLOCK_SIZE
        + elem_offsets
    )
    tl.store(output_ptr + out_indices, result, mask=block_mask[:, None])
```
**EN:** Defines function `_dequantize_nvfp4_kernel` with signature `_dequantize_nvfp4_kernel(fp4_ptr, scale_ptr, global_scale_ptr, output_ptr, rows_per_batch: tl.constexpr, num_blocks: tl.constexpr, BLOCK_SIZE: tl.constexpr, has_batch_global_scale: tl.constexpr, TILE_BLOCKS: tl.constexpr)`. It mainly works with `fp4_ptr`, `scale_ptr`, `global_scale_ptr`, `output_ptr`, `rows_per_batch`, `num_blocks`, `BLOCK_SIZE`, `has_batch_global_scale`; handles quantization-related transformation logic. The body uses branching, tensor/kernel operations. Key calls include `tl.program_id`, `tl.arange`, `tl.load`, `tl.cast.to`, `_e2m1_inline`, `tl.interleave`.
**CN:** 定义函数 `_dequantize_nvfp4_kernel`，其签名为 `_dequantize_nvfp4_kernel(fp4_ptr, scale_ptr, global_scale_ptr, output_ptr, rows_per_batch: tl.constexpr, num_blocks: tl.constexpr, BLOCK_SIZE: tl.constexpr, has_batch_global_scale: tl.constexpr, TILE_BLOCKS: tl.constexpr)`。它主要围绕 `fp4_ptr`, `scale_ptr`, `global_scale_ptr`, `output_ptr`, `rows_per_batch`, `num_blocks`, `BLOCK_SIZE`, `has_batch_global_scale` 展开；处理量化相关的变换逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `tl.program_id`, `tl.arange`, `tl.load`, `tl.cast.to`, `_e2m1_inline`, `tl.interleave`。

### Function `_nvfp4_quant_dequant_kernel` (lines 163-214)
```python
def _nvfp4_quant_dequant_kernel(
    input_ptr,
    output_ptr,
    global_scale_ptr,
    k: tl.constexpr,
    num_blocks: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
    FP4_MAX_RECIPROCAL: tl.constexpr,
    TILE_BLOCKS: tl.constexpr,
):
    """Fused NVFP4 quantize-dequantize kernel.

    Uses a 2D grid (rows x tiles) to parallelize across both rows
    and quantization groups within a row. Each program handles
    TILE_BLOCKS groups at once using vectorized 2D operations.
    """
    row_idx = tl.program_id(0)
    tile_idx = tl.program_id(1)
    global_scale = tl.load(global_scale_ptr).to(tl.float32)
    row_offset = row_idx * k

    start_block = tile_idx * TILE_BLOCKS
    block_offsets = tl.arange(0, TILE_BLOCKS)
    block_mask = (start_block + block_offsets) < num_blocks

    # Load [TILE_BLOCKS, BLOCK_SIZE] elements
    indices = (
        row_offset
        + (start_block + block_offsets[:, None]) * BLOCK_SIZE
        + tl.arange(0, BLOCK_SIZE)[None, :]
    )
    mask_2d = block_mask[:, None]
    x = tl.load(input_ptr + indices, mask=mask_2d, other=0.0).to(tl.float32)

    # Per-group scale: [TILE_BLOCKS]
    vec_max = tl.max(tl.abs(x), axis=1)
    scale = global_scale * (vec_max * FP4_MAX_RECIPROCAL)
    scale = tl.clamp(scale, -448.0, 448.0)
    scale = scale.to(tl.float8e4nv).to(tl.float32)

    # Safe reciprocal, broadcast to [TILE_BLOCKS, 1]
    output_scale = tl.where(scale == 0.0, 0.0, global_scale / scale)[:, None]

    # Quantize: scale, clamp, round to FP4
    scaled_x = tl.clamp(x * output_scale, -6.0, 6.0)
    fp4_val = _round_to_fp4(scaled_x)

    # Dequantize: fp4_val * (scale / global_scale)
    dequant_scale = (scale / global_scale)[:, None]
    result = fp4_val * dequant_scale

    tl.store(output_ptr + indices, result, mask=mask_2d)
```
**EN:** Defines function `_nvfp4_quant_dequant_kernel` with signature `_nvfp4_quant_dequant_kernel(input_ptr, output_ptr, global_scale_ptr, k: tl.constexpr, num_blocks: tl.constexpr, BLOCK_SIZE: tl.constexpr, FP4_MAX_RECIPROCAL: tl.constexpr, TILE_BLOCKS: tl.constexpr)`. It mainly works with `input_ptr`, `output_ptr`, `global_scale_ptr`, `k`, `num_blocks`, `BLOCK_SIZE`, `FP4_MAX_RECIPROCAL`, `TILE_BLOCKS`; handles quantization-related transformation logic. The body uses tensor/kernel operations. Key calls include `tl.program_id`, `tl.load.to`, `tl.arange`, `tl.max`, `tl.clamp`, `scale.to.to`.
**CN:** 定义函数 `_nvfp4_quant_dequant_kernel`，其签名为 `_nvfp4_quant_dequant_kernel(input_ptr, output_ptr, global_scale_ptr, k: tl.constexpr, num_blocks: tl.constexpr, BLOCK_SIZE: tl.constexpr, FP4_MAX_RECIPROCAL: tl.constexpr, TILE_BLOCKS: tl.constexpr)`。它主要围绕 `input_ptr`, `output_ptr`, `global_scale_ptr`, `k`, `num_blocks`, `BLOCK_SIZE`, `FP4_MAX_RECIPROCAL`, `TILE_BLOCKS` 展开；处理量化相关的变换逻辑。函数体包含张量或内核操作。关键调用包括 `tl.program_id`, `tl.load.to`, `tl.arange`, `tl.max`, `tl.clamp`, `scale.to.to`。

### Function `_triton_nvfp4_quant_dequant` (lines 217-249)
```python
def _triton_nvfp4_quant_dequant(
    x: torch.Tensor,
    global_scale: torch.Tensor,
    block_size: int,
) -> torch.Tensor:
    """Triton-accelerated NVFP4 quantize-dequantize."""
    x_m, x_k = x.shape

    if not torch.compiler.is_compiling():
        assert x_k % block_size == 0, (
            f"Weight shape K={x_k} is not divisible by block_size={block_size}"
        )

    output_dtype = x.dtype
    num_blocks = x_k // block_size

    output = torch.empty(x_m, x_k, dtype=output_dtype, device=x.device)

    tile_blocks = min(64, triton.next_power_of_2(num_blocks))
    num_tiles = (num_blocks + tile_blocks - 1) // tile_blocks
    grid = (x_m, num_tiles)
    _nvfp4_quant_dequant_kernel[grid](
        x,
        output,
        global_scale,
        x_k,
        num_blocks,
        block_size,
        FLOAT4_E2M1_MAX_RECIPROCAL,
        tile_blocks,
    )

    return output
```
**EN:** Defines function `_triton_nvfp4_quant_dequant` with signature `_triton_nvfp4_quant_dequant(x: torch.Tensor, global_scale: torch.Tensor, block_size: int) -> torch.Tensor`. It mainly works with `x`, `global_scale`, `block_size`; handles quantization-related transformation logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `torch.empty`, `min`, `_nvfp4_quant_dequant_kernel`, `torch.compiler.is_compiling`, `triton.next_power_of_2`.
**CN:** 定义函数 `_triton_nvfp4_quant_dequant`，其签名为 `_triton_nvfp4_quant_dequant(x: torch.Tensor, global_scale: torch.Tensor, block_size: int) -> torch.Tensor`。它主要围绕 `x`, `global_scale`, `block_size` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `torch.empty`, `min`, `_nvfp4_quant_dequant_kernel`, `torch.compiler.is_compiling`, `triton.next_power_of_2`。

### Function `_triton_dequantize_nvfp4` (lines 252-325)
```python
def _triton_dequantize_nvfp4(
    tensor_fp4: torch.Tensor,
    tensor_sf: torch.Tensor,
    global_scale: torch.Tensor,
    dtype: torch.dtype,
    block_size: int = 16,
) -> torch.Tensor:
    """Dequantize NVFP4 using Triton (swizzle=False only).

    Supports both 2D and 3D inputs:
    - 2D: [m, packed_k] -> [m, k]
    - 3D: [dim0, m, packed_k] -> [dim0, m, k]
    """
    assert tensor_fp4.dtype == torch.uint8

    is_3d = tensor_fp4.ndim == 3
    if is_3d:
        dim0, m_per_batch, packed_k = tensor_fp4.shape
        tensor_fp4_2d = tensor_fp4.reshape(-1, packed_k)
        tensor_sf_2d = tensor_sf.reshape(-1, tensor_sf.shape[-1])
        total_rows_flat = dim0 * m_per_batch
    else:
        m_per_batch, packed_k = tensor_fp4.shape
        tensor_fp4_2d = tensor_fp4
        tensor_sf_2d = tensor_sf
        total_rows_flat = m_per_batch

    k = packed_k * 2
    num_blocks = k // block_size

    output = torch.empty(total_rows_flat, k, dtype=dtype, device=tensor_fp4.device)

    # View as uint8 so Triton can load raw bytes and bitcast to float8_e4m3fn
    scale_raw = tensor_sf_2d.contiguous().view(torch.uint8)

    # Shape-adaptive tile sizing: for large row counts (3D), process
    # entire row in one tile. For small row counts (2D), use smaller
    # tiles to increase parallelism across CUs.
# ... truncated for analysis ...
        output,
        m_per_batch,
        num_blocks,
        block_size,
        is_3d,
        tile_blocks,
        num_warps=nw,
        num_stages=ns,
    )

    if is_3d:
        output = output.reshape(dim0, m_per_batch, k)

    return output
```
**EN:** Defines function `_triton_dequantize_nvfp4` with signature `_triton_dequantize_nvfp4(tensor_fp4: torch.Tensor, tensor_sf: torch.Tensor, global_scale: torch.Tensor, dtype: torch.dtype, block_size: int=16) -> torch.Tensor`. It mainly works with `tensor_fp4`, `tensor_sf`, `global_scale`, `dtype`, `block_size`; handles quantization-related transformation logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `torch.empty`, `tensor_sf_2d.contiguous.view`, `triton.next_power_of_2`, `_dequantize_nvfp4_kernel`, `tensor_fp4.reshape`, `tensor_sf.reshape`.
**CN:** 定义函数 `_triton_dequantize_nvfp4`，其签名为 `_triton_dequantize_nvfp4(tensor_fp4: torch.Tensor, tensor_sf: torch.Tensor, global_scale: torch.Tensor, dtype: torch.dtype, block_size: int=16) -> torch.Tensor`。它主要围绕 `tensor_fp4`, `tensor_sf`, `global_scale`, `dtype`, `block_size` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `torch.empty`, `tensor_sf_2d.contiguous.view`, `triton.next_power_of_2`, `_dequantize_nvfp4_kernel`, `tensor_fp4.reshape`, `tensor_sf.reshape`。

### Function `dequantize_to_dtype` (lines 358-412)
```python
def dequantize_to_dtype(
    tensor_fp4: torch.Tensor,
    tensor_sf: torch.Tensor,
    global_scale: torch.Tensor,
    dtype: torch.dtype,
    block_size: int = 16,
    swizzle: bool | None = True,
):
    """Dequantize the fp4 tensor back to high precision.

    Supports both 2D and 3D inputs:
    - 2D: [m, packed_k] -> [m, k]
    - 3D: [dim0, m, packed_k] -> [dim0, m, k]
    """
    # Two fp4 values are packed into one uint8.
    assert tensor_fp4.dtype == torch.uint8

    if not swizzle and current_platform.is_cuda_alike():
        return _triton_dequantize_nvfp4(
            tensor_fp4, tensor_sf, global_scale, dtype, block_size
        )

    # We handle 3D tensors reshaping them to 2D.
    is_3d = tensor_fp4.ndim == 3

    if is_3d:
        dim0, m, packed_k = tensor_fp4.shape
        tensor_fp4 = tensor_fp4.reshape(-1, packed_k)
        tensor_sf = tensor_sf.reshape(-1, tensor_sf.shape[-1])
        global_scale = global_scale[:, None, None]
    else:
        m, packed_k = tensor_fp4.shape

    k = packed_k * 2
    tensor_f32 = break_fp4_bytes(tensor_fp4, torch.float32)
    tensor_f32 = tensor_f32.reshape(-1, k // block_size, block_size)
    tensor_sf = tensor_sf.view(torch.float8_e4m3fn)

    if swizzle:
        tensor_sf = convert_swizzled_to_linear(  # noqa: E501
            tensor_sf, tensor_f32.size(0), k, block_size
        )

    if is_3d:
        tensor_sf = tensor_sf.reshape(dim0, m, k // block_size)
    tensor_sf_dtype = tensor_sf.to(torch.float32) * global_scale

    if is_3d:
        tensor_f32 = tensor_f32.reshape(dim0, m, -1, block_size)

    # scale the tensor
    out = tensor_f32 * tensor_sf_dtype.unsqueeze(-1)
    out = out.reshape(*out.shape[:-2], -1)

    return out.to(dtype)
```
**EN:** Defines function `dequantize_to_dtype` with signature `dequantize_to_dtype(tensor_fp4: torch.Tensor, tensor_sf: torch.Tensor, global_scale: torch.Tensor, dtype: torch.dtype, block_size: int=16, swizzle: bool | None=True)`. It mainly works with `tensor_fp4`, `tensor_sf`, `global_scale`, `dtype`, `block_size`, `swizzle`; handles quantization-related transformation logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `break_fp4_bytes`, `tensor_f32.reshape`, `tensor_sf.view`, `out.reshape`, `out.to`, `current_platform.is_cuda_alike`.
**CN:** 定义函数 `dequantize_to_dtype`，其签名为 `dequantize_to_dtype(tensor_fp4: torch.Tensor, tensor_sf: torch.Tensor, global_scale: torch.Tensor, dtype: torch.dtype, block_size: int=16, swizzle: bool | None=True)`。它主要围绕 `tensor_fp4`, `tensor_sf`, `global_scale`, `dtype`, `block_size`, `swizzle` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `break_fp4_bytes`, `tensor_f32.reshape`, `tensor_sf.view`, `out.reshape`, `out.to`, `current_platform.is_cuda_alike`。

### Function `get_reciprocal` (lines 415-422)
```python
def get_reciprocal(x):
    if isinstance(x, torch.Tensor):
        # torch.where yields operation not permitted when stream is capturing.
        return 1.0 / (x + (x == 0) * 1e8)
    elif isinstance(x, (float, int)):
        return 0.0 if x == 0 else 1.0 / x
    else:
        raise TypeError("Input must be a float, int, or a torch.Tensor.")
```
**EN:** Defines function `get_reciprocal` with signature `get_reciprocal(x)`. It mainly works with `x`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `isinstance`, `TypeError`.
**CN:** 定义函数 `get_reciprocal`，其签名为 `get_reciprocal(x)`。它主要围绕 `x` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `isinstance`, `TypeError`。

### Function `run_nvfp4_emulations` (lines 481-507)
```python
def run_nvfp4_emulations(
    x: torch.Tensor,
    input_global_scale: torch.Tensor,
    weight: torch.Tensor,
    weight_scale_swizzled: torch.Tensor,
    weight_global_scale: torch.Tensor,
    swizzle: bool | None = True,
):
    output_dtype = x.dtype
    group_size = 16

    x_dq = ref_nvfp4_quant_dequant(x, input_global_scale, block_size=group_size)

    # dequantize weight
    w_fp4 = weight.data.view(torch.uint8)
    w_dq = dequantize_to_dtype(
        w_fp4,
        weight_scale_swizzled.data,
        weight_global_scale,
        output_dtype,
        group_size,
        swizzle=swizzle,
    )

    # matmul
    out = torch.matmul(x_dq, w_dq.t())
    return out
```
**EN:** Defines function `run_nvfp4_emulations` with signature `run_nvfp4_emulations(x: torch.Tensor, input_global_scale: torch.Tensor, weight: torch.Tensor, weight_scale_swizzled: torch.Tensor, weight_global_scale: torch.Tensor, swizzle: bool | None=True)`. It mainly works with `x`, `input_global_scale`, `weight`, `weight_scale_swizzled`, `weight_global_scale`, `swizzle`; implements one step in the quantized-weight execution flow. The body uses tensor/kernel operations. Key calls include `ref_nvfp4_quant_dequant`, `weight.data.view`, `dequantize_to_dtype`, `torch.matmul`, `w_dq.t`.
**CN:** 定义函数 `run_nvfp4_emulations`，其签名为 `run_nvfp4_emulations(x: torch.Tensor, input_global_scale: torch.Tensor, weight: torch.Tensor, weight_scale_swizzled: torch.Tensor, weight_global_scale: torch.Tensor, swizzle: bool | None=True)`。它主要围绕 `x`, `input_global_scale`, `weight`, `weight_scale_swizzled`, `weight_global_scale`, `swizzle` 展开；实现量化权重执行流程中的一个步骤。函数体包含张量或内核操作。关键调用包括 `ref_nvfp4_quant_dequant`, `weight.data.view`, `dequantize_to_dtype`, `torch.matmul`, `w_dq.t`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `_e2m1_inline`, `_dequantize_nvfp4_kernel`, `_e2m1_lookup`, `_round_to_fp4`, `_nvfp4_quant_dequant_kernel` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `_e2m1_inline`, `_dequantize_nvfp4_kernel`, `_e2m1_lookup`, `_round_to_fp4`, `_nvfp4_quant_dequant_kernel` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `types`, `torch`
- **Internal / 内部**: `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`
