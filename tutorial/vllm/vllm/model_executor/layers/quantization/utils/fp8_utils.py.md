# fp8_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/fp8_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-36)
```python
import functools
import json
import os
from collections.abc import Callable, Sequence
from typing import Any

import torch

import vllm.envs as envs
from vllm import _custom_ops as ops
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    get_fp8_min_max,
)
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    all_close_1d,
    per_tensor_dequantize,
)
from vllm.model_executor.parameter import (
    BlockQuantScaleParameter,
    ChannelQuantScaleParameter,
    PerTensorScaleParameter,
)
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.utils.deep_gemm import (
    get_tma_aligned_size,
    is_deep_gemm_e8m0_used,
    transform_sf_into_required_layout,
)
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This opening block pulls in external dependencies such as `functools`, `json`, `os`, `collections`, `typing`, `torch` and internal modules such as `vllm.envs`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.model_executor.parameter`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `functools`, `json`, `os`, `collections`, `typing`, `torch`）以及内部模块（如 `vllm.envs`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.model_executor.parameter`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 38-38)
```python
logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `_silu_mul_quant_fp8_packed_kernel` (lines 153-230)
```python
def _silu_mul_quant_fp8_packed_kernel(
    input_ptr,
    output_q_ptr,
    output_scale_ptr,
    M,
    input_stride_m,
    output_q_stride_m,
    output_scale_stride_k,
    clamp_limit,
    N: tl.constexpr,
    NUM_GROUPS: tl.constexpr,
    fp8_min: tl.constexpr,
    fp8_max: tl.constexpr,
    GROUP_SIZE: tl.constexpr,
    BLOCK_M: tl.constexpr,
    HAS_CLAMP: tl.constexpr,
):
    N_2: tl.constexpr = N // 2

    pid_pack = tl.program_id(0)
    pid_m = tl.program_id(1)
    m_offset = pid_m.to(tl.int64) * BLOCK_M

    if m_offset >= M:
        return

    offs_m = tl.arange(0, BLOCK_M)
    offs_n = tl.arange(0, GROUP_SIZE)
    row_mask = (m_offset + offs_m) < M

    base_row_offset = (m_offset + offs_m[:, None]) * input_stride_m
    base_out_offset = (m_offset + offs_m[:, None]) * output_q_stride_m

    packed_scale = tl.zeros((BLOCK_M,), dtype=tl.int32)

    for pack_idx in tl.static_range(4):
        group_id = pid_pack * 4 + pack_idx

# ... truncated for analysis ...
            y_q = tl.clamp(y / scale[:, None], fp8_min, fp8_max)

            out_q_ptrs = output_q_ptr + base_out_offset + n_offset + offs_n[None, :]
            tl.store(
                out_q_ptrs,
                y_q.to(output_q_ptr.dtype.element_ty),
                mask=row_mask[:, None],
            )

            exponent_biased = tl.clamp(exponent + 127.0, 0.0, 255.0).to(tl.int32)
            packed_scale = packed_scale | (exponent_biased << (pack_idx * 8))

    scale_ptrs = output_scale_ptr + pid_pack * output_scale_stride_k + m_offset + offs_m
    tl.store(scale_ptrs, packed_scale, mask=row_mask)
```
**EN:** Defines function `_silu_mul_quant_fp8_packed_kernel` with signature `_silu_mul_quant_fp8_packed_kernel(input_ptr, output_q_ptr, output_scale_ptr, M, input_stride_m, output_q_stride_m, output_scale_stride_k, clamp_limit, N: tl.constexpr, NUM_GROUPS: tl.constexpr, fp8_min: tl.constexpr, fp8_max: tl.constexpr, GROUP_SIZE: tl.constexpr, BLOCK_M: tl.constexpr, HAS_CLAMP: tl.constexpr)`. It mainly works with `input_ptr`, `output_q_ptr`, `output_scale_ptr`, `M`, `input_stride_m`, `output_q_stride_m`, `output_scale_stride_k`, `clamp_limit`; handles quantization-related transformation logic. The body uses branching, iteration, tensor/kernel operations. Key calls include `tl.program_id`, `tl.arange`, `tl.zeros`, `tl.static_range`, `tl.store`, `pid_m.to`.
**CN:** 定义函数 `_silu_mul_quant_fp8_packed_kernel`，其签名为 `_silu_mul_quant_fp8_packed_kernel(input_ptr, output_q_ptr, output_scale_ptr, M, input_stride_m, output_q_stride_m, output_scale_stride_k, clamp_limit, N: tl.constexpr, NUM_GROUPS: tl.constexpr, fp8_min: tl.constexpr, fp8_max: tl.constexpr, GROUP_SIZE: tl.constexpr, BLOCK_M: tl.constexpr, HAS_CLAMP: tl.constexpr)`。它主要围绕 `input_ptr`, `output_q_ptr`, `output_scale_ptr`, `M`, `input_stride_m`, `output_q_stride_m`, `output_scale_stride_k`, `clamp_limit` 展开；处理量化相关的变换逻辑。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `tl.program_id`, `tl.arange`, `tl.zeros`, `tl.static_range`, `tl.store`, `pid_m.to`。

### Function `_silu_mul_per_token_group_quant_fp8_colmajor` (lines 295-373)
```python
def _silu_mul_per_token_group_quant_fp8_colmajor(
    y_ptr,  # [M, N]
    y_q_ptr,  # [M, N // 2]
    y_s_ptr,  # [M, (N // 2) // GROUP_SIZE]
    M,  # num tokens
    N,  # intermediate size
    # Stride
    y_s_col_stride: tl.int64,
    # Information for float8
    eps,
    clamp_limit,
    fp8_min: tl.constexpr,
    fp8_max: tl.constexpr,
    use_ue8m0: tl.constexpr,
    HAS_CLAMP: tl.constexpr,
    # Meta-parameters
    GROUP_SIZE: tl.constexpr,
    BLOCK_M: tl.constexpr,
    BLOCK_N: tl.constexpr,
):
    # TODO(varun) : Add expert_ids so we may early-exit no-op thread blocks.
    """
    Each thread block (BLOCK_N) computes [BLOCK_M, GROUP_SIZE] act-mul outputs. Then
    the thread block quantizes the [BLOCK_M, GROUP_SIZE] block of values and fills
    the outputs tensors at the right positions.
    """

    pid_m = tl.program_id(0)
    pid_n = tl.program_id(1)
    N_2 = N // 2

    m_offset = pid_m.to(tl.int64) * BLOCK_M
    n_offset = pid_n.to(tl.int64) * BLOCK_N
    if m_offset >= M:
        return

    offs_n = tl.arange(0, BLOCK_N).to(tl.int64)
    offs_m = tl.arange(0, BLOCK_M).to(tl.int64)
# ... truncated for analysis ...
    y_s = tl.reshape(y_s, (BLOCK_M, 1))
    y_q = tl.clamp(y / y_s, fp8_min, fp8_max).to(y_q_ptr.dtype.element_ty)

    # store y_q
    base_y_q_ptr = y_q_ptr + m_offset * N_2 + n_offset
    y_q_ptrs = base_y_q_ptr + offs_m[:, None] * N_2 + offs_n[None, :]
    tl.store(y_q_ptrs, y_q)

    # store y_s
    group_id = n_offset // GROUP_SIZE
    base_y_s_ptr = y_s_ptr + group_id * y_s_col_stride + m_offset
    y_s_ptrs = base_y_s_ptr + offs_m
    y_s = tl.reshape(y_s, (BLOCK_M,))
    tl.store(y_s_ptrs, y_s)
```
**EN:** Defines function `_silu_mul_per_token_group_quant_fp8_colmajor` with signature `_silu_mul_per_token_group_quant_fp8_colmajor(y_ptr, y_q_ptr, y_s_ptr, M, N, y_s_col_stride: tl.int64, eps, clamp_limit, fp8_min: tl.constexpr, fp8_max: tl.constexpr, use_ue8m0: tl.constexpr, HAS_CLAMP: tl.constexpr, GROUP_SIZE: tl.constexpr, BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr)`. It mainly works with `y_ptr`, `y_q_ptr`, `y_s_ptr`, `M`, `N`, `y_s_col_stride`, `eps`, `clamp_limit`; handles quantization-related transformation logic. The body uses branching, tensor/kernel operations. Key calls include `tl.program_id`, `tl.arange.to`, `tl.load`, `act_in.to`, `tl.cast`, `to`.
**CN:** 定义函数 `_silu_mul_per_token_group_quant_fp8_colmajor`，其签名为 `_silu_mul_per_token_group_quant_fp8_colmajor(y_ptr, y_q_ptr, y_s_ptr, M, N, y_s_col_stride: tl.int64, eps, clamp_limit, fp8_min: tl.constexpr, fp8_max: tl.constexpr, use_ue8m0: tl.constexpr, HAS_CLAMP: tl.constexpr, GROUP_SIZE: tl.constexpr, BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr)`。它主要围绕 `y_ptr`, `y_q_ptr`, `y_s_ptr`, `M`, `N`, `y_s_col_stride`, `eps`, `clamp_limit` 展开；处理量化相关的变换逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `tl.program_id`, `tl.arange.to`, `tl.load`, `act_in.to`, `tl.cast`, `to`。

### Function `per_token_group_quant_fp8` (lines 510-634)
```python
def per_token_group_quant_fp8(
    x: torch.Tensor,
    group_size: int,
    eps: float = 1e-10,
    dtype: torch.dtype | None = None,
    column_major_scales: bool = False,
    tma_aligned_scales: bool = False,
    out_q: torch.Tensor | None = None,
    use_ue8m0: bool | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Function to perform per-token-group quantization on an input tensor `x`.
    It converts the tensor values into signed float8 values and returns the
    quantized tensor along with the scaling factor used for quantization.
    Args:
        x: The input tensor with ndim >= 2.
        group_size: The group size used for quantization.
        eps: The minimum to avoid dividing zero.
        dtype: The dtype of output tensor. Note that only `torch.float8_e4m3fn`
        is supported for now.
        column_major_scales: Outputs scales in column major.
        tma_aligned_scales: Outputs scales in TMA-aligned layout.
        out_q: Optional output tensor. If not provided, function will create.
    Returns:
        tuple[torch.Tensor, torch.Tensor]: The quantized tensor and the
        scaling factor.
    """
    if use_ue8m0 is None:
        use_ue8m0 = is_deep_gemm_e8m0_used()
    dtype = current_platform.fp8_dtype() if dtype is None else dtype
    assert x.shape[-1] % group_size == 0, (
        f"the last dimension of `x` {x.shape[-1]} must be divisible "
        f"by `group_size` {group_size}"
    )
    assert x.stride(-1) == 1, "`x` groups must be contiguous"

    fp8_min, fp8_max = get_fp8_min_max()

    assert out_q is None or out_q.shape == x.shape
# ... truncated for analysis ...
            x_s,
            group_size,
            x.shape[1],
            x.stride(0),
            eps,
            fp8_min=fp8_min,
            fp8_max=fp8_max,
            use_ue8m0=use_ue8m0,
            BLOCK=BLOCK,
            num_warps=num_warps,
            num_stages=num_stages,
        )

    return x_q, x_s
```
**EN:** Defines function `per_token_group_quant_fp8` with signature `per_token_group_quant_fp8(x: torch.Tensor, group_size: int, eps: float=1e-10, dtype: torch.dtype | None=None, column_major_scales: bool=False, tma_aligned_scales: bool=False, out_q: torch.Tensor | None=None, use_ue8m0: bool | None=None) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `group_size`, `eps`, `dtype`, `column_major_scales`, `tma_aligned_scales`, `out_q`, `use_ue8m0`; handles quantization-related transformation logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `get_fp8_min_max`, `triton.next_power_of_2`, `min`, `is_deep_gemm_e8m0_used`, `current_platform.fp8_dtype`, `x.stride`.
**CN:** 定义函数 `per_token_group_quant_fp8`，其签名为 `per_token_group_quant_fp8(x: torch.Tensor, group_size: int, eps: float=1e-10, dtype: torch.dtype | None=None, column_major_scales: bool=False, tma_aligned_scales: bool=False, out_q: torch.Tensor | None=None, use_ue8m0: bool | None=None) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `group_size`, `eps`, `dtype`, `column_major_scales`, `tma_aligned_scales`, `out_q`, `use_ue8m0` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `get_fp8_min_max`, `triton.next_power_of_2`, `min`, `is_deep_gemm_e8m0_used`, `current_platform.fp8_dtype`, `x.stride`。

### Function `_w8a8_triton_block_scaled_mm` (lines 712-793)
```python
def _w8a8_triton_block_scaled_mm(
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
**EN:** Defines function `_w8a8_triton_block_scaled_mm` with signature `_w8a8_triton_block_scaled_mm(A, B, C, As, Bs, M, N, K, group_n, group_k, stride_am, stride_ak, stride_bk, stride_bn, stride_cm, stride_cn, stride_As_m, stride_As_k, stride_Bs_k, stride_Bs_n, BLOCK_SIZE_M: tl.constexpr, BLOCK_SIZE_N: tl.constexpr, BLOCK_SIZE_K: tl.constexpr, GROUP_SIZE_M: tl.constexpr)`. It mainly works with `A`, `B`, `C`, `As`, `Bs`, `M`, `N`, `K`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, tensor/kernel operations. Key calls include `tl.program_id`, `tl.cdiv`, `min`, `tl.arange`, `tl.zeros`, `range`.
**CN:** 定义函数 `_w8a8_triton_block_scaled_mm`，其签名为 `_w8a8_triton_block_scaled_mm(A, B, C, As, Bs, M, N, K, group_n, group_k, stride_am, stride_ak, stride_bk, stride_bn, stride_cm, stride_cn, stride_As_m, stride_As_k, stride_Bs_k, stride_Bs_n, BLOCK_SIZE_M: tl.constexpr, BLOCK_SIZE_N: tl.constexpr, BLOCK_SIZE_K: tl.constexpr, GROUP_SIZE_M: tl.constexpr)`。它主要围绕 `A`, `B`, `C`, `As`, `Bs`, `M`, `N`, `K` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `tl.program_id`, `tl.cdiv`, `min`, `tl.arange`, `tl.zeros`, `range`。

### Function `get_w8a8_block_fp8_configs` (lines 797-832)
```python
def get_w8a8_block_fp8_configs(
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
    json_file_name = f"N={N},K={K},device_name={device_name},dtype=fp8_w8a8,block_shape=[{block_n},{block_k}].json"  # noqa: E501

    config_file_path = os.path.join(
        os.path.dirname(os.path.realpath(__file__)), "configs", json_file_name
    )
    if os.path.exists(config_file_path):
        with open(config_file_path) as f:
            logger.info(
                "Using configuration from %s for W8A8 Block FP8 kernel.",
                config_file_path,
            )
            # If a configuration has been found, return it
            return {int(key): val for key, val in json.load(f).items()}

    # If no optimized configuration is available, we will use the default
    # configuration
    logger.warning(
        "Using default W8A8 Block FP8 kernel config. Performance might "
        "be sub-optimal! Config file not found at %s",
        config_file_path,
    )
    return None
```
**EN:** Defines function `get_w8a8_block_fp8_configs` with signature `get_w8a8_block_fp8_configs(N: int, K: int, block_n: int, block_k: int) -> dict[int, Any] | None`. It mainly works with `N`, `K`, `block_n`, `block_k`; returns a derived property or capability check. The body uses branching, comprehensions, context-managed execution. Key calls include `current_platform.get_device_name.replace`, `os.path.join`, `os.path.exists`, `logger.warning`, `os.path.dirname`, `current_platform.get_device_name`.
**CN:** 定义函数 `get_w8a8_block_fp8_configs`，其签名为 `get_w8a8_block_fp8_configs(N: int, K: int, block_n: int, block_k: int) -> dict[int, Any] | None`。它主要围绕 `N`, `K`, `block_n`, `block_k` 展开；返回派生属性或能力判断结果。函数体包含分支判断、推导式、上下文管理。关键调用包括 `current_platform.get_device_name.replace`, `os.path.join`, `os.path.exists`, `logger.warning`, `os.path.dirname`, `current_platform.get_device_name`。

### Function `w8a8_triton_block_scaled_mm` (lines 835-929)
```python
def w8a8_triton_block_scaled_mm(
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
        block_size: The block size for per-block quantization. It should
        be 2-dim, e.g., [128, 128].
        output_dytpe: The dtype of the returned tensor.
    Returns:
        torch.Tensor: The result of matmul.
    """
    assert len(block_size) == 2
    block_n, block_k = block_size[0], block_size[1]

    # Triton cannot currently bind E8M0 scale tensors directly. On ROCm,
    # DeepSeek-V4 checkpoints store block scales in exponent-only E8M0 format,
    # so decode them to fp32 before launching the kernel.
    if current_platform.is_rocm():
        if As.dtype == torch.float8_e8m0fnu:
            As = _upcast_e8m0_to_fp32(As).contiguous()
        if Bs.dtype == torch.float8_e8m0fnu:
            Bs = _upcast_e8m0_to_fp32(Bs).contiguous()

    assert A.shape[-1] == B.shape[-1]
    assert A.shape[:-1] == As.shape[:-1] and A.is_contiguous()
    assert triton.cdiv(A.shape[-1], block_k) == As.shape[-1]
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
**EN:** Defines function `w8a8_triton_block_scaled_mm` with signature `w8a8_triton_block_scaled_mm(A: torch.Tensor, B: torch.Tensor, As: torch.Tensor, Bs: torch.Tensor, block_size: list[int], output_dtype: torch.dtype=torch.float16) -> torch.Tensor`. It mainly works with `A`, `B`, `As`, `Bs`, `block_size`, `output_dtype`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `current_platform.is_rocm`, `A.new_empty`, `get_w8a8_block_fp8_configs`, `_w8a8_triton_block_scaled_mm`, `len`, `A.is_contiguous`.
**CN:** 定义函数 `w8a8_triton_block_scaled_mm`，其签名为 `w8a8_triton_block_scaled_mm(A: torch.Tensor, B: torch.Tensor, As: torch.Tensor, Bs: torch.Tensor, block_size: list[int], output_dtype: torch.dtype=torch.float16) -> torch.Tensor`。它主要围绕 `A`, `B`, `As`, `Bs`, `block_size`, `output_dtype` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `current_platform.is_rocm`, `A.new_empty`, `get_w8a8_block_fp8_configs`, `_w8a8_triton_block_scaled_mm`, `len`, `A.is_contiguous`。

### Function `prepare_fp8_moe_layer_for_deepgemm` (lines 1082-1102)
```python
def prepare_fp8_moe_layer_for_deepgemm(
    w13: torch.Tensor,
    w2: torch.Tensor,
    w13_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    block_shape: tuple[int],
):
    w13, w13_scale = deepgemm_post_process_fp8_weight_block(
        wq=w13,
        ws=w13_scale,
        quant_block_shape=block_shape,
        use_e8m0=is_deep_gemm_e8m0_used(),
    )
    w2, w2_scale = deepgemm_post_process_fp8_weight_block(
        wq=w2,
        ws=w2_scale,
        quant_block_shape=block_shape,
        use_e8m0=is_deep_gemm_e8m0_used(),
    )

    return w13, w2, w13_scale, w2_scale
```
**EN:** Defines function `prepare_fp8_moe_layer_for_deepgemm` with signature `prepare_fp8_moe_layer_for_deepgemm(w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor, block_shape: tuple[int])`. It mainly works with `w13`, `w2`, `w13_scale`, `w2_scale`, `block_shape`; constructs runtime helpers from the current configuration. The body uses mostly straightforward data movement and object wiring. Key calls include `deepgemm_post_process_fp8_weight_block`, `is_deep_gemm_e8m0_used`.
**CN:** 定义函数 `prepare_fp8_moe_layer_for_deepgemm`，其签名为 `prepare_fp8_moe_layer_for_deepgemm(w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor, block_shape: tuple[int])`。它主要围绕 `w13`, `w2`, `w13_scale`, `w2_scale`, `block_shape` 展开；根据当前配置构建运行时辅助对象。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `deepgemm_post_process_fp8_weight_block`, `is_deep_gemm_e8m0_used`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `is_fp8`, `_triton_per_token_group_quant_fp8_impl`, `_triton_per_token_group_quant_fp8_fake`, `input_to_float8`, `_per_token_group_quant_fp8` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `is_fp8`, `_triton_per_token_group_quant_fp8_impl`, `_triton_per_token_group_quant_fp8_fake`, `input_to_float8`, `_per_token_group_quant_fp8` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `functools`, `json`, `os`, `collections`, `typing`, `torch`
- **Internal / 内部**: `vllm.envs`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.model_executor.parameter`, `vllm.model_executor.utils`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`
