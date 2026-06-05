# int8_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/int8_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-14)
```python
import functools
import json
import logging
import os
from typing import Any

import torch

from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
```
**EN:** This opening block pulls in external dependencies such as `functools`, `json`, `logging`, `os`, `typing`, `torch` and internal modules such as `vllm.platforms`, `vllm.triton_utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `functools`, `json`, `logging`, `os`, `typing`, `torch`）以及内部模块（如 `vllm.platforms`, `vllm.triton_utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 16-16)
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `apply_w8a8_block_int8_linear` (lines 19-39)
```python
def apply_w8a8_block_int8_linear(
    input: torch.Tensor,
    weight: torch.Tensor,
    block_size: list[int],
    weight_scale: torch.Tensor,
    input_scale: torch.Tensor | None = None,
    bias: torch.Tensor | None = None,
) -> torch.Tensor:
    assert input_scale is None
    # View input as 2D matrix for fp8 methods
    input_2d = input.view(-1, input.shape[-1])
    output_shape = [*input.shape[:-1], weight.shape[0]]

    q_input, x_scale = per_token_group_quant_int8(input_2d, block_size[1])
    output = w8a8_block_int8_matmul(
        q_input, weight, x_scale, weight_scale, block_size, output_dtype=input.dtype
    )

    if bias is not None:
        output = output + bias
    return output.to(dtype=input.dtype).view(*output_shape)
```
**EN:** Defines function `apply_w8a8_block_int8_linear` with signature `apply_w8a8_block_int8_linear(input: torch.Tensor, weight: torch.Tensor, block_size: list[int], weight_scale: torch.Tensor, input_scale: torch.Tensor | None=None, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `input`, `weight`, `block_size`, `weight_scale`, `input_scale`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `input.view`, `per_token_group_quant_int8`, `w8a8_block_int8_matmul`, `output.to.view`, `output.to`.
**CN:** 定义函数 `apply_w8a8_block_int8_linear`，其签名为 `apply_w8a8_block_int8_linear(input: torch.Tensor, weight: torch.Tensor, block_size: list[int], weight_scale: torch.Tensor, input_scale: torch.Tensor | None=None, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `input`, `weight`, `block_size`, `weight_scale`, `input_scale`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `input.view`, `per_token_group_quant_int8`, `w8a8_block_int8_matmul`, `output.to.view`, `output.to`。

### Function `block_dequant` (lines 56-82)
```python
def block_dequant(
    x_q_block: torch.Tensor,
    x_s: torch.Tensor,
    block_size: list[int],
) -> torch.Tensor:
    """This function conducts block-wise dequantization.
    The inputs are block-wise quantization tensor `x_q_block`,
    block-wise quantization scale and the block size.
    The outputs are dequantized tensor.
    """
    block_n, block_k = block_size[0], block_size[1]
    n, k = x_q_block.shape
    n_tiles = (n + block_n - 1) // block_n
    k_tiles = (k + block_k - 1) // block_k
    assert n_tiles == x_s.shape[0]
    assert k_tiles == x_s.shape[1]

    x_dq_block = x_q_block.to(torch.float32)

    for i in range(k_tiles):
        for j in range(n_tiles):
            x_dq_block[
                j * block_n : min((j + 1) * block_n, n),
                i * block_k : min((i + 1) * block_k, k),
            ] *= x_s[j][i]

    return x_dq_block
```
**EN:** Defines function `block_dequant` with signature `block_dequant(x_q_block: torch.Tensor, x_s: torch.Tensor, block_size: list[int]) -> torch.Tensor`. It mainly works with `x_q_block`, `x_s`, `block_size`; handles quantization-related transformation logic. The body uses iteration, validation/error handling. Key calls include `x_q_block.to`, `range`, `min`.
**CN:** 定义函数 `block_dequant`，其签名为 `block_dequant(x_q_block: torch.Tensor, x_s: torch.Tensor, block_size: list[int]) -> torch.Tensor`。它主要围绕 `x_q_block`, `x_s`, `block_size` 展开；处理量化相关的变换逻辑。函数体包含循环处理、校验或报错逻辑。关键调用包括 `x_q_block.to`, `range`, `min`。

### Function `_per_token_group_quant_int8` (lines 160-198)
```python
def _per_token_group_quant_int8(
    # Pointers to inputs and output
    y_ptr,
    y_q_ptr,
    y_s_ptr,
    # Stride of input
    y_stride,
    # Columns of input
    N,
    # Avoid to divide zero
    eps,
    # Information for int8
    int8_min,
    int8_max,
    # Meta-parameters
    BLOCK: tl.constexpr,
):
    """A Triton-accelerated function to perform per-token-group
    quantization on a tensor.

    This function converts the tensor values into int8 values.
    """
    # Map the program id to the row of X and Y it should compute.
    g_id = tl.program_id(0)
    y_ptr += g_id * y_stride
    y_q_ptr += g_id * y_stride
    y_s_ptr += g_id

    cols = tl.arange(0, BLOCK)  # N <= BLOCK
    mask = cols < N

    y = tl.load(y_ptr + cols, mask=mask, other=0.0).to(tl.float32)
    # Quant
    _absmax = tl.maximum(tl.max(tl.abs(y)), eps)
    y_s = _absmax / int8_max
    y_q = tl.clamp(y / y_s, int8_min, int8_max).to(y_q_ptr.dtype.element_ty)

    tl.store(y_q_ptr + cols, y_q, mask=mask)
    tl.store(y_s_ptr, y_s)
```
**EN:** Defines function `_per_token_group_quant_int8` with signature `_per_token_group_quant_int8(y_ptr, y_q_ptr, y_s_ptr, y_stride, N, eps, int8_min, int8_max, BLOCK: tl.constexpr)`. It mainly works with `y_ptr`, `y_q_ptr`, `y_s_ptr`, `y_stride`, `N`, `eps`, `int8_min`, `int8_max`; handles quantization-related transformation logic. The body uses tensor/kernel operations. Key calls include `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.maximum`, `tl.clamp.to`, `tl.store`.
**CN:** 定义函数 `_per_token_group_quant_int8`，其签名为 `_per_token_group_quant_int8(y_ptr, y_q_ptr, y_s_ptr, y_stride, N, eps, int8_min, int8_max, BLOCK: tl.constexpr)`。它主要围绕 `y_ptr`, `y_q_ptr`, `y_s_ptr`, `y_stride`, `N`, `eps`, `int8_min`, `int8_max` 展开；处理量化相关的变换逻辑。函数体包含张量或内核操作。关键调用包括 `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.maximum`, `tl.clamp.to`, `tl.store`。

### Function `per_token_group_quant_int8` (lines 201-266)
```python
def per_token_group_quant_int8(
    x: torch.Tensor,
    group_size: int,
    eps: float = 1e-10,
    dtype: torch.dtype = torch.int8,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Function to perform per-token-group quantization on an input tensor `x`.

    It converts the tensor values into signed int8 values and returns the
    quantized tensor along with the scaling factor used for quantization.

    Args:
        x: The input tensor with ndim >= 2.
        group_size: The group size used for quantization.
        eps: The minimum to avoid dividing zero.
        dtype: The dype of output tensor. Note that only `torch.int8`
            is supported for now.

    Returns:
        tuple[torch.Tensor, torch.Tensor]: The quantized tensor and the
            scaling factor for quantization.
    """
    assert x.shape[-1] % group_size == 0, (
        "the last dimension of `x` cannot be divisible by `group_size`"
    )
    assert x.is_contiguous(), "`x` is not contiguous"

    iinfo = torch.iinfo(dtype)
    int8_max = iinfo.max
    int8_min = iinfo.min

    x_q = torch.empty_like(x, device=x.device, dtype=dtype)
    x_s = torch.empty(
        x.shape[:-1] + (x.shape[-1] // group_size,),
        device=x.device,
        dtype=torch.float32,
    )
    # prefer CUDA kernel if available
# ... truncated for analysis ...
        x,
        x_q,
        x_s,
        group_size,
        N,
        eps,
        int8_min=int8_min,
        int8_max=int8_max,
        BLOCK=BLOCK,
        num_warps=num_warps,
        num_stages=num_stages,
    )

    return x_q, x_s
```
**EN:** Defines function `per_token_group_quant_int8` with signature `per_token_group_quant_int8(x: torch.Tensor, group_size: int, eps: float=1e-10, dtype: torch.dtype=torch.int8) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `group_size`, `eps`, `dtype`; handles quantization-related transformation logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `x.is_contiguous`, `torch.iinfo`, `torch.empty_like`, `torch.empty`, `current_platform.is_cuda`, `triton.next_power_of_2`.
**CN:** 定义函数 `per_token_group_quant_int8`，其签名为 `per_token_group_quant_int8(x: torch.Tensor, group_size: int, eps: float=1e-10, dtype: torch.dtype=torch.int8) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `group_size`, `eps`, `dtype` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `x.is_contiguous`, `torch.iinfo`, `torch.empty_like`, `torch.empty`, `current_platform.is_cuda`, `triton.next_power_of_2`。

### Function `_w8a8_block_int8_matmul` (lines 270-351)
```python
def _w8a8_block_int8_matmul(
    # Pointers to inputs and output
    A,
    B,
    C,
    As,
    Bs,
    # Shape for matmul
    M,
    N,
    K,
    # Block size for block-wise quantization
    group_n,
    group_k,
    # Stride for inputs and output
    stride_am,
    stride_ak,
    stride_bk,
    stride_bn,
    stride_cm,
    stride_cn,
    stride_As_m,
    stride_As_k,
    stride_Bs_k,
    stride_Bs_n,
    # Meta-parameters
    BLOCK_SIZE_M: tl.constexpr,
    BLOCK_SIZE_N: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
    GROUP_SIZE_M: tl.constexpr,
):
    """Triton-accelerated function used to perform linear operations (dot
    product) on input tensors `A` and `B` with block-wise quantization, and
    store the result in output tensor `C`.
    """

    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
# ... truncated for analysis ...
        b_ptrs += BLOCK_SIZE_K * stride_bk

    if C.dtype.element_ty == tl.bfloat16:
        c = accumulator.to(tl.bfloat16)
    elif C.dtype.element_ty == tl.float16:
        c = accumulator.to(tl.float16)
    else:
        c = accumulator.to(tl.float32)

    offs_cm = pid_m * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)
    offs_cn = pid_n * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)
    c_ptrs = C + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
    c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)
    tl.store(c_ptrs, c, mask=c_mask)
```
**EN:** Defines function `_w8a8_block_int8_matmul` with signature `_w8a8_block_int8_matmul(A, B, C, As, Bs, M, N, K, group_n, group_k, stride_am, stride_ak, stride_bk, stride_bn, stride_cm, stride_cn, stride_As_m, stride_As_k, stride_Bs_k, stride_Bs_n, BLOCK_SIZE_M: tl.constexpr, BLOCK_SIZE_N: tl.constexpr, BLOCK_SIZE_K: tl.constexpr, GROUP_SIZE_M: tl.constexpr)`. It mainly works with `A`, `B`, `C`, `As`, `Bs`, `M`, `N`, `K`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, tensor/kernel operations. Key calls include `tl.program_id`, `tl.cdiv`, `min`, `tl.arange`, `tl.zeros`, `range`.
**CN:** 定义函数 `_w8a8_block_int8_matmul`，其签名为 `_w8a8_block_int8_matmul(A, B, C, As, Bs, M, N, K, group_n, group_k, stride_am, stride_ak, stride_bk, stride_bn, stride_cm, stride_cn, stride_As_m, stride_As_k, stride_Bs_k, stride_Bs_n, BLOCK_SIZE_M: tl.constexpr, BLOCK_SIZE_N: tl.constexpr, BLOCK_SIZE_K: tl.constexpr, GROUP_SIZE_M: tl.constexpr)`。它主要围绕 `A`, `B`, `C`, `As`, `Bs`, `M`, `N`, `K` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `tl.program_id`, `tl.cdiv`, `min`, `tl.arange`, `tl.zeros`, `range`。

### Function `get_w8a8_block_int8_configs` (lines 355-393)
```python
def get_w8a8_block_int8_configs(
    N: int, K: int, block_n: int, block_k: int
) -> dict[int, Any] | None:
    """
    Return optimized configurations for the w8a8 block fp8 kernel.

    The return value will be a dictionary that maps an irregular grid of
    batch sizes to configurations of the w8a8 block fp8 kernel. To evaluate the
    kernel on a given batch size bs, the closest batch size in the grid should
    be picked and the associated configuration chosen to invoke the kernel.
    """

    # First look up if an optimized configuration is available in the configs
    # directory
    device_name = current_platform.get_device_name().replace(" ", "_")
    json_file_name = f"N={N},K={K},device_name={device_name},dtype=int8_w8a8,block_shape=[{block_n}, {block_k}].json"  # noqa: E501

    config_file_path = os.path.join(
        os.path.dirname(os.path.realpath(__file__)), "configs", json_file_name
    )
    if os.path.exists(config_file_path):
        with open(config_file_path) as f:
            logger.info(
                "Using configuration from %s for W8A8 Block INT8 kernel.",
                config_file_path,
            )
            # If a configuration has been found, return it
            return {int(key): val for key, val in json.load(f).items()}

    # If no optimized configuration is available, we will use the default
    # configuration
    logger.warning(
        (
            "Using default W8A8 Block INT8 kernel config. Performance might "
            "be sub-optimal! Config file not found at %s"
        ),
        config_file_path,
    )
    return None
```
**EN:** Defines function `get_w8a8_block_int8_configs` with signature `get_w8a8_block_int8_configs(N: int, K: int, block_n: int, block_k: int) -> dict[int, Any] | None`. It mainly works with `N`, `K`, `block_n`, `block_k`; returns a derived property or capability check. The body uses branching, comprehensions, context-managed execution. Key calls include `current_platform.get_device_name.replace`, `os.path.join`, `os.path.exists`, `logger.warning`, `os.path.dirname`, `current_platform.get_device_name`.
**CN:** 定义函数 `get_w8a8_block_int8_configs`，其签名为 `get_w8a8_block_int8_configs(N: int, K: int, block_n: int, block_k: int) -> dict[int, Any] | None`。它主要围绕 `N`, `K`, `block_n`, `block_k` 展开；返回派生属性或能力判断结果。函数体包含分支判断、推导式、上下文管理。关键调用包括 `current_platform.get_device_name.replace`, `os.path.join`, `os.path.exists`, `logger.warning`, `os.path.dirname`, `current_platform.get_device_name`。

### Function `w8a8_block_int8_matmul` (lines 396-484)
```python
def w8a8_block_int8_matmul(
    A: torch.Tensor,
    B: torch.Tensor,
    As: torch.Tensor,
    Bs: torch.Tensor,
    block_size: list[int],
    output_dtype: torch.dtype = torch.float16,
) -> torch.Tensor:
    """This function performs matrix multiplication with block-wise
    quantization.

    It takes two input tensors `A` and `B` with scales `As` and `Bs`.
    The output is returned in the specified `output_dtype`.

    Args:
        A: The input tensor, e.g., activation.
        B: The input tensor, e.g., weight.
        As: The per-token-group quantization scale for `A`.
        Bs: The per-block quantization scale for `B`.
        block_size: The block size for per-block quantization. It should be
            2-dim, e.g., [128, 128].
        output_dtype: The dtype of the returned tensor.

    Returns:
        torch.Tensor: The result of matmul.
    """
    assert len(block_size) == 2
    block_n, block_k = block_size[0], block_size[1]

    assert A.shape[-1] == B.shape[-1]
    assert A.shape[:-1] == As.shape[:-1] and A.is_contiguous()
    assert triton.cdiv(A.shape[-1], block_k) == As.shape[-1]
    M = A.numel() // A.shape[-1]

    assert B.ndim == 2 and B.is_contiguous() and Bs.ndim == 2
    N, K = B.shape
    assert triton.cdiv(N, block_n) == Bs.shape[0]
    assert triton.cdiv(K, block_k) == Bs.shape[1]
# ... truncated for analysis ...
        A.stride(-2),
        A.stride(-1),
        B.stride(1),
        B.stride(0),
        C.stride(-2),
        C.stride(-1),
        As.stride(-2),
        As.stride(-1),
        Bs.stride(1),
        Bs.stride(0),
        **config,
    )

    return C
```
**EN:** Defines function `w8a8_block_int8_matmul` with signature `w8a8_block_int8_matmul(A: torch.Tensor, B: torch.Tensor, As: torch.Tensor, Bs: torch.Tensor, block_size: list[int], output_dtype: torch.dtype=torch.float16) -> torch.Tensor`. It mainly works with `A`, `B`, `As`, `Bs`, `block_size`, `output_dtype`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `A.new_empty`, `get_w8a8_block_int8_configs`, `_w8a8_block_int8_matmul`, `len`, `A.is_contiguous`, `triton.cdiv`.
**CN:** 定义函数 `w8a8_block_int8_matmul`，其签名为 `w8a8_block_int8_matmul(A: torch.Tensor, B: torch.Tensor, As: torch.Tensor, Bs: torch.Tensor, block_size: list[int], output_dtype: torch.dtype=torch.float16) -> torch.Tensor`。它主要围绕 `A`, `B`, `As`, `Bs`, `block_size`, `output_dtype` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `A.new_empty`, `get_w8a8_block_int8_configs`, `_w8a8_block_int8_matmul`, `len`, `A.is_contiguous`, `triton.cdiv`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `apply_w8a8_block_int8_linear`, `input_to_int8`, `block_dequant`, `_per_token_quant_int8`, `per_token_quant_int8` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `apply_w8a8_block_int8_linear`, `input_to_int8`, `block_dequant`, `_per_token_quant_int8`, `per_token_quant_int8` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `functools`, `json`, `logging`, `os`, `typing`, `torch`
- **Internal / 内部**: `vllm.platforms`, `vllm.triton_utils`
