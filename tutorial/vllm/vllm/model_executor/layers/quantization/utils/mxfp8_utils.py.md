# mxfp8_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/mxfp8_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-6)
```python
import torch

from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm.utils.torch_utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm.utils.torch_utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 9-11)
```python
MXFP8_VALUE_DTYPE = torch.float8_e4m3fn
MXFP8_SCALE_DTYPE = torch.uint8
MXFP8_BLOCK_SIZE = 32
```
**EN:** This block defines module-level metadata or constants such as `MXFP8_VALUE_DTYPE`, `MXFP8_SCALE_DTYPE`, `MXFP8_BLOCK_SIZE`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `MXFP8_VALUE_DTYPE`, `MXFP8_SCALE_DTYPE`, `MXFP8_BLOCK_SIZE`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `swizzle_mxfp8_scale` (lines 14-35)
```python
def swizzle_mxfp8_scale(sf: torch.Tensor, M: int, K: int) -> torch.Tensor:
    """Swizzle MXFP8 scales from row-major 2D to F8_128x4 layout."""
    scaling_vector_size = MXFP8_BLOCK_SIZE  # 32 for MXFP8
    factor = scaling_vector_size * 4  # 128

    num_m_tiles = (M + 127) // 128
    num_k_tiles = (K + factor - 1) // factor

    m_padded = num_m_tiles * 128
    k_scale_padded = num_k_tiles * 4

    scale_cols = K // scaling_vector_size
    sf_padded = torch.zeros(
        (m_padded, k_scale_padded), dtype=sf.dtype, device=sf.device
    )
    sf_padded[:M, :scale_cols] = sf

    sf_reshaped = sf_padded.view(num_m_tiles, 4, 32, num_k_tiles, 4)

    sf_swizzled = sf_reshaped.transpose(1, 3)

    return sf_swizzled.contiguous().view(-1)
```
**EN:** Defines function `swizzle_mxfp8_scale` with signature `swizzle_mxfp8_scale(sf: torch.Tensor, M: int, K: int) -> torch.Tensor`. It mainly works with `sf`, `M`, `K`; implements one step in the quantized-weight execution flow. The body uses tensor/kernel operations. Key calls include `torch.zeros`, `sf_padded.view`, `sf_reshaped.transpose`, `sf_swizzled.contiguous.view`, `sf_swizzled.contiguous`.
**CN:** 定义函数 `swizzle_mxfp8_scale`，其签名为 `swizzle_mxfp8_scale(sf: torch.Tensor, M: int, K: int) -> torch.Tensor`。它主要围绕 `sf`, `M`, `K` 展开；实现量化权重执行流程中的一个步骤。函数体包含张量或内核操作。关键调用包括 `torch.zeros`, `sf_padded.view`, `sf_reshaped.transpose`, `sf_swizzled.contiguous.view`, `sf_swizzled.contiguous`。

### Function `_mxfp8_e4m3_quantize_torch` (lines 38-84)
```python
def _mxfp8_e4m3_quantize_torch(
    x: torch.Tensor,
    is_sf_swizzled_layout: bool = False,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Naive MXFP8 quantization.
    For each block of 32 elements along the last dimension, compute a
    shared e8m0 scale (the biased exponent of the block-wise amax)
    and quantize each element to float8_e4m3fn.

    Returns (quantized_values [same shape, fp8], scales uint8).
    Scale shape depends on is_sf_swizzled_layout:
      False -> [..., K//32]  (row-major 2D)
      True  -> [flat swizzled 1D]
    """
    assert x.shape[-1] % MXFP8_BLOCK_SIZE == 0
    orig_shape = x.shape
    num_blocks = x.shape[-1] // MXFP8_BLOCK_SIZE

    x_fp32 = x.to(torch.float32)
    x_blocked = x_fp32.view(*orig_shape[:-1], num_blocks, MXFP8_BLOCK_SIZE)

    amax = x_blocked.abs().amax(dim=-1)
    amax = amax.clamp(min=torch.finfo(torch.float32).tiny)
    scale_biased = torch.floor(torch.log2(amax)) + 127.0
    scale_biased = scale_biased.clamp(0, 254)
    scales_uint8 = scale_biased.to(torch.uint8)

    descale = torch.exp2(scale_biased - 127.0)
    x_scaled = x_blocked / descale.unsqueeze(-1)

    x_fp8 = x_scaled.view(orig_shape).to(MXFP8_VALUE_DTYPE)

    if x.ndim == 2:
        M, K = x.shape
        scales_uint8 = scales_uint8.view(M, -1)
        if is_sf_swizzled_layout:
            scales_uint8 = swizzle_mxfp8_scale(scales_uint8, M=M, K=K)
    elif x.ndim == 3:
        B, M, K = x.shape
        scales_uint8 = scales_uint8.view(B, M, -1)
        if is_sf_swizzled_layout:
            swizzled = []
            for i in range(B):
                swizzled.append(swizzle_mxfp8_scale(scales_uint8[i], M=M, K=K))
            scales_uint8 = torch.cat(swizzled)

    return x_fp8, scales_uint8
```
**EN:** Defines function `_mxfp8_e4m3_quantize_torch` with signature `_mxfp8_e4m3_quantize_torch(x: torch.Tensor, is_sf_swizzled_layout: bool=False) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `is_sf_swizzled_layout`; handles quantization-related transformation logic. The body uses branching, iteration, validation/error handling, tensor/kernel operations. Key calls include `x.to`, `x_fp32.view`, `x_blocked.abs.amax`, `amax.clamp`, `scale_biased.clamp`, `scale_biased.to`.
**CN:** 定义函数 `_mxfp8_e4m3_quantize_torch`，其签名为 `_mxfp8_e4m3_quantize_torch(x: torch.Tensor, is_sf_swizzled_layout: bool=False) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `is_sf_swizzled_layout` 展开；处理量化相关的变换逻辑。函数体包含分支判断、循环处理、校验或报错逻辑、张量或内核操作。关键调用包括 `x.to`, `x_fp32.view`, `x_blocked.abs.amax`, `amax.clamp`, `scale_biased.clamp`, `scale_biased.to`。

### Function `_mxfp8_e4m3_quantize_impl` (lines 87-106)
```python
def _mxfp8_e4m3_quantize_impl(
    x: torch.Tensor,
    is_sf_swizzled_layout: bool = False,
    alignment: int = 0,
) -> tuple[torch.Tensor, torch.Tensor]:
    from vllm.platforms import current_platform

    if current_platform.has_device_capability(100):
        from flashinfer import mxfp8_quantize as flashinfer_mxfp8_quantize

        x_q, x_scales = flashinfer_mxfp8_quantize(
            x,
            is_sf_swizzled_layout=is_sf_swizzled_layout,
            alignment=alignment if alignment > 0 else 32,
        )
        if x_scales.ndim == 1 and x.ndim == 2 and not is_sf_swizzled_layout:
            x_scales = x_scales.view(x.size(0), -1)
        return x_q, x_scales

    return _mxfp8_e4m3_quantize_torch(x, is_sf_swizzled_layout)
```
**EN:** Defines function `_mxfp8_e4m3_quantize_impl` with signature `_mxfp8_e4m3_quantize_impl(x: torch.Tensor, is_sf_swizzled_layout: bool=False, alignment: int=0) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `is_sf_swizzled_layout`, `alignment`; handles quantization-related transformation logic. The body uses branching. Key calls include `current_platform.has_device_capability`, `_mxfp8_e4m3_quantize_torch`, `flashinfer_mxfp8_quantize`, `x_scales.view`, `x.size`.
**CN:** 定义函数 `_mxfp8_e4m3_quantize_impl`，其签名为 `_mxfp8_e4m3_quantize_impl(x: torch.Tensor, is_sf_swizzled_layout: bool=False, alignment: int=0) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `is_sf_swizzled_layout`, `alignment` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `current_platform.has_device_capability`, `_mxfp8_e4m3_quantize_torch`, `flashinfer_mxfp8_quantize`, `x_scales.view`, `x.size`。

### Function `mxfp8_e4m3_quantize` (lines 109-114)
```python
def mxfp8_e4m3_quantize(
    x: torch.Tensor,
    is_sf_swizzled_layout: bool = False,
    alignment: int = 0,
) -> tuple[torch.Tensor, torch.Tensor]:
    return torch.ops.vllm.mxfp8_quantize(x, is_sf_swizzled_layout, alignment)
```
**EN:** Defines function `mxfp8_e4m3_quantize` with signature `mxfp8_e4m3_quantize(x: torch.Tensor, is_sf_swizzled_layout: bool=False, alignment: int=0) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `is_sf_swizzled_layout`, `alignment`; handles quantization-related transformation logic. The body uses tensor/kernel operations. Key calls include `torch.ops.vllm.mxfp8_quantize`.
**CN:** 定义函数 `mxfp8_e4m3_quantize`，其签名为 `mxfp8_e4m3_quantize(x: torch.Tensor, is_sf_swizzled_layout: bool=False, alignment: int=0) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `is_sf_swizzled_layout`, `alignment` 展开；处理量化相关的变换逻辑。函数体包含张量或内核操作。关键调用包括 `torch.ops.vllm.mxfp8_quantize`。

### Function `dequant_mxfp8_to_bf16` (lines 117-130)
```python
def dequant_mxfp8_to_bf16(x: torch.Tensor, scales: torch.Tensor) -> torch.Tensor:
    """Dequantize MXFP8 tensor to BF16."""
    x_float = x.to(torch.float32)

    num_blocks = x.shape[-1] // MXFP8_BLOCK_SIZE
    x_blocked = x_float.view(*x.shape[:-1], num_blocks, MXFP8_BLOCK_SIZE)

    descale = torch.exp2(scales.to(torch.float32) - 127.0)

    dequantized = x_blocked * descale.unsqueeze(-1)

    dequantized = dequantized.view(*x.shape)

    return dequantized.to(torch.bfloat16)
```
**EN:** Defines function `dequant_mxfp8_to_bf16` with signature `dequant_mxfp8_to_bf16(x: torch.Tensor, scales: torch.Tensor) -> torch.Tensor`. It mainly works with `x`, `scales`; handles quantization-related transformation logic. The body uses tensor/kernel operations. Key calls include `x.to`, `x_float.view`, `torch.exp2`, `dequantized.view`, `dequantized.to`, `descale.unsqueeze`.
**CN:** 定义函数 `dequant_mxfp8_to_bf16`，其签名为 `dequant_mxfp8_to_bf16(x: torch.Tensor, scales: torch.Tensor) -> torch.Tensor`。它主要围绕 `x`, `scales` 展开；处理量化相关的变换逻辑。函数体包含张量或内核操作。关键调用包括 `x.to`, `x_float.view`, `torch.exp2`, `dequantized.view`, `dequantized.to`, `descale.unsqueeze`。

### Function `mxfp8_e4m3_quantize_fake` (lines 133-170)
```python
def mxfp8_e4m3_quantize_fake(
    x: torch.Tensor,
    is_sf_swizzled_layout: bool = False,
    alignment: int = 0,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Fake implementation for torch.compile tracing."""
    fp_data = torch.empty_like(x, dtype=MXFP8_VALUE_DTYPE)

    block_size = MXFP8_BLOCK_SIZE

    if x.ndim == 2:
        M, N = x.shape
        K = (N + block_size - 1) // block_size
        if is_sf_swizzled_layout:
            M_padded = ((M + 127) // 128) * 128
            K_padded = ((K + 3) // 4) * 4
            scales = torch.empty(
                M_padded * K_padded, dtype=MXFP8_SCALE_DTYPE, device=x.device
            )
        else:
            scales = torch.empty((M, K), dtype=MXFP8_SCALE_DTYPE, device=x.device)
    elif x.ndim == 3:
        B, M, N = x.shape
        K = (N + block_size - 1) // block_size
        if is_sf_swizzled_layout:
            M_padded = ((M + 127) // 128) * 128
            K_padded = ((K + 3) // 4) * 4
            scales = torch.empty(
                B * M_padded * K_padded, dtype=MXFP8_SCALE_DTYPE, device=x.device
            )
        else:
            scales = torch.empty((B, M, K), dtype=MXFP8_SCALE_DTYPE, device=x.device)
    else:
        scale_shape = list(x.shape)
        scale_shape[-1] = (x.shape[-1] + block_size - 1) // block_size
        scales = torch.empty(scale_shape, dtype=MXFP8_SCALE_DTYPE, device=x.device)

    return fp_data, scales
```
**EN:** Defines function `mxfp8_e4m3_quantize_fake` with signature `mxfp8_e4m3_quantize_fake(x: torch.Tensor, is_sf_swizzled_layout: bool=False, alignment: int=0) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `is_sf_swizzled_layout`, `alignment`; handles quantization-related transformation logic. The body uses branching, tensor/kernel operations. Key calls include `torch.empty_like`, `torch.empty`, `list`.
**CN:** 定义函数 `mxfp8_e4m3_quantize_fake`，其签名为 `mxfp8_e4m3_quantize_fake(x: torch.Tensor, is_sf_swizzled_layout: bool=False, alignment: int=0) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `is_sf_swizzled_layout`, `alignment` 展开；处理量化相关的变换逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.empty_like`, `torch.empty`, `list`。

### Function `xpu_mxfp8_quantize` (lines 180-183)
```python
def xpu_mxfp8_quantize(
    x: torch.Tensor, dtype: torch.dtype | None = None
) -> tuple[torch.Tensor, torch.Tensor]:
    return torch.ops.vllm.xpu_mxfp8_quantize(x, dtype)
```
**EN:** Defines function `xpu_mxfp8_quantize` with signature `xpu_mxfp8_quantize(x: torch.Tensor, dtype: torch.dtype | None=None) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `dtype`; handles quantization-related transformation logic. The body uses tensor/kernel operations. Key calls include `torch.ops.vllm.xpu_mxfp8_quantize`.
**CN:** 定义函数 `xpu_mxfp8_quantize`，其签名为 `xpu_mxfp8_quantize(x: torch.Tensor, dtype: torch.dtype | None=None) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `dtype` 展开；处理量化相关的变换逻辑。函数体包含张量或内核操作。关键调用包括 `torch.ops.vllm.xpu_mxfp8_quantize`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `swizzle_mxfp8_scale`, `_mxfp8_e4m3_quantize_torch`, `_mxfp8_e4m3_quantize_impl`, `mxfp8_e4m3_quantize`, `dequant_mxfp8_to_bf16` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `swizzle_mxfp8_scale`, `_mxfp8_e4m3_quantize_torch`, `_mxfp8_e4m3_quantize_impl`, `mxfp8_e4m3_quantize`, `dequant_mxfp8_to_bf16` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm.utils.torch_utils`
