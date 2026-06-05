# nvfp4_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/nvfp4_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-10)
```python
import torch

from vllm._custom_ops import (
    cutlass_scaled_mm_supports_fp4,
)
from vllm.platforms import current_platform
from vllm.utils.math_utils import round_up
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm._custom_ops`, `vllm.platforms`, `vllm.utils.math_utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm._custom_ops`, `vllm.platforms`, `vllm.utils.math_utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `swizzle_blockscale` (lines 13-53)
```python
def swizzle_blockscale(scale: torch.Tensor) -> torch.Tensor:
    """
    Pad and block-interleave the FP4 block-scales so that they match the data
    layout expected by the CUTLASS / FlashInfer kernels.

    Parameters
    ----------
    scale: torch.Tensor

    Returns
    -------
    torch.Tensor
        The swizzled tensor with the same logical shape as *scale*.
    """
    assert scale.dtype == torch.float8_e4m3fn, (
        "swizzle_blockscale expects the input tensor to be in "
        "torch.float8_e4m3fn format."
    )

    scale_ndim = scale.ndim
    if scale_ndim == 2:
        scale = scale.unsqueeze(0)  # (1, M, K)
    assert scale.ndim == 3, "Expected a 2-D or 3-D tensor for block scales."

    B, M, K = scale.shape

    M_padded = round_up(M, 128)
    K_padded = round_up(K, 4)

    padded = torch.zeros(
        (B, M_padded, K_padded), dtype=scale.dtype, device=scale.device
    )
    padded[:B, :M, :K] = scale

    # Reshape / permute to the layout required by the kernel.
    padded = padded.reshape(B, M_padded // 128, 4, 32, K_padded // 4, 4)
    swizzled = padded.permute(0, 1, 4, 3, 2, 5).contiguous().cuda()

    if scale_ndim == 2:
        return swizzled.reshape(M_padded, K_padded)
    return swizzled.reshape(B, M_padded, K_padded)
```
**EN:** Defines function `swizzle_blockscale` with signature `swizzle_blockscale(scale: torch.Tensor) -> torch.Tensor`. It mainly works with `scale`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `round_up`, `torch.zeros`, `padded.reshape`, `padded.permute.contiguous.cuda`, `swizzled.reshape`, `scale.unsqueeze`.
**CN:** 定义函数 `swizzle_blockscale`，其签名为 `swizzle_blockscale(scale: torch.Tensor) -> torch.Tensor`。它主要围绕 `scale` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `round_up`, `torch.zeros`, `padded.reshape`, `padded.permute.contiguous.cuda`, `swizzled.reshape`, `scale.unsqueeze`。

### Function `cutlass_fp4_supported` (lines 56-61)
```python
def cutlass_fp4_supported() -> bool:
    if not current_platform.is_cuda():
        return False
    capability_tuple = current_platform.get_device_capability()
    capability = -1 if capability_tuple is None else capability_tuple.to_int()
    return cutlass_scaled_mm_supports_fp4(capability)
```
**EN:** Defines function `cutlass_fp4_supported` with signature `cutlass_fp4_supported() -> bool`. It mainly works with object context only; implements one step in the quantized-weight execution flow. The body uses branching, tensor/kernel operations. Key calls include `current_platform.get_device_capability`, `cutlass_scaled_mm_supports_fp4`, `current_platform.is_cuda`, `capability_tuple.to_int`.
**CN:** 定义函数 `cutlass_fp4_supported`，其签名为 `cutlass_fp4_supported() -> bool`。它主要围绕 仅依赖对象上下文 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `current_platform.get_device_capability`, `cutlass_scaled_mm_supports_fp4`, `current_platform.is_cuda`, `capability_tuple.to_int`。

### Function `pad_nvfp4_weight_for_cutlass` (lines 64-98)
```python
def pad_nvfp4_weight_for_cutlass(
    weight: torch.Tensor,
    alignment: int = 32,
) -> tuple[torch.Tensor, int]:
    """
    Pad packed NVFP4 weights so that both N (rows) and K (columns) satisfy
    the alignment constraints required by CUTLASS / FlashInfer FP4 kernels.

    CUTLASS FP4 kernel requires both K and N matrix dimensions to be divisible
    by 32 for aligned memory access and efficient tensor core operations.
    """
    weight_current_rows = weight.shape[0]

    # Pad N dimension (rows) if not aligned
    if weight_current_rows % alignment != 0:
        total_rows = round_up(weight_current_rows, alignment)
        pad_rows = total_rows - weight_current_rows
        weight = torch.nn.functional.pad(weight, (0, 0, 0, pad_rows)).contiguous()

    # Check K dimension alignment
    # 2 FP4 items are packed per byte in the input dimension
    weight_current_col_bytes = weight.shape[1]
    weight_current_col_elements = weight_current_col_bytes * 2

    weights_padding_bytes = 0
    if weight_current_col_elements % alignment != 0:
        total_cols = round_up(weight_current_col_elements, alignment)
        pad_cols = total_cols - weight_current_col_elements
        # Convert from FP4 element count to bytes (2 FP4 values per byte)
        # pad_cols is always even since alignment=32 and current elements are even
        pad_bytes = pad_cols // 2
        weight = torch.nn.functional.pad(weight, (0, pad_bytes, 0, 0)).contiguous()
        weights_padding_bytes = pad_bytes

    return weight, weights_padding_bytes
```
**EN:** Defines function `pad_nvfp4_weight_for_cutlass` with signature `pad_nvfp4_weight_for_cutlass(weight: torch.Tensor, alignment: int=32) -> tuple[torch.Tensor, int]`. It mainly works with `weight`, `alignment`; implements one step in the quantized-weight execution flow. The body uses branching, tensor/kernel operations. Key calls include `round_up`, `torch.nn.functional.pad.contiguous`, `torch.nn.functional.pad`.
**CN:** 定义函数 `pad_nvfp4_weight_for_cutlass`，其签名为 `pad_nvfp4_weight_for_cutlass(weight: torch.Tensor, alignment: int=32) -> tuple[torch.Tensor, int]`。它主要围绕 `weight`, `alignment` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `round_up`, `torch.nn.functional.pad.contiguous`, `torch.nn.functional.pad`。

### Function `pad_nvfp4_activation_for_cutlass` (lines 101-111)
```python
def pad_nvfp4_activation_for_cutlass(
    x_fp4: torch.Tensor,
    weights_padding_bytes: int,
) -> torch.Tensor:
    """
    Pad packed FP4 activations to match the K-dimension padding applied to weights.
    The padding is in bytes (tensor dimension), not FP4 elements.
    """
    if weights_padding_bytes > 0:
        return torch.nn.functional.pad(x_fp4, (0, weights_padding_bytes)).contiguous()
    return x_fp4
```
**EN:** Defines function `pad_nvfp4_activation_for_cutlass` with signature `pad_nvfp4_activation_for_cutlass(x_fp4: torch.Tensor, weights_padding_bytes: int) -> torch.Tensor`. It mainly works with `x_fp4`, `weights_padding_bytes`; implements one step in the quantized-weight execution flow. The body uses branching, tensor/kernel operations. Key calls include `torch.nn.functional.pad.contiguous`, `torch.nn.functional.pad`.
**CN:** 定义函数 `pad_nvfp4_activation_for_cutlass`，其签名为 `pad_nvfp4_activation_for_cutlass(x_fp4: torch.Tensor, weights_padding_bytes: int) -> torch.Tensor`。它主要围绕 `x_fp4`, `weights_padding_bytes` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.nn.functional.pad.contiguous`, `torch.nn.functional.pad`。

### Function `slice_nvfp4_output` (lines 114-123)
```python
def slice_nvfp4_output(
    out: torch.Tensor,
    output_size: int,
) -> torch.Tensor:
    """
    Slice the output tensor to remove padding in N dimension if weight was padded.
    """
    if out.shape[-1] != output_size:
        return out[..., :output_size].contiguous()
    return out
```
**EN:** Defines function `slice_nvfp4_output` with signature `slice_nvfp4_output(out: torch.Tensor, output_size: int) -> torch.Tensor`. It mainly works with `out`, `output_size`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `out.contiguous`.
**CN:** 定义函数 `slice_nvfp4_output`，其签名为 `slice_nvfp4_output(out: torch.Tensor, output_size: int) -> torch.Tensor`。它主要围绕 `out`, `output_size` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `out.contiguous`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `swizzle_blockscale`, `cutlass_fp4_supported`, `pad_nvfp4_weight_for_cutlass`, `pad_nvfp4_activation_for_cutlass`, `slice_nvfp4_output` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `swizzle_blockscale`, `cutlass_fp4_supported`, `pad_nvfp4_weight_for_cutlass`, `pad_nvfp4_activation_for_cutlass`, `slice_nvfp4_output` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm._custom_ops`, `vllm.platforms`, `vllm.utils.math_utils`
