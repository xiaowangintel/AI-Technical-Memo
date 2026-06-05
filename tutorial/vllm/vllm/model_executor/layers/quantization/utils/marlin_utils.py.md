# marlin_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/marlin_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-22)
```python
import numpy
import torch

import vllm.envs as envs
from vllm import _custom_ops as ops
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe import RoutedExperts
from vllm.model_executor.layers.linear import LinearBase
from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.utils.int8_utils import (
    per_token_quant_int8,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import GroupShape
from vllm.platforms import current_platform
from vllm.scalar_type import ScalarType, scalar_types
from vllm.utils.platform_utils import num_compute_units

from .quant_utils import pack_cols, unpack_cols
```
**EN:** This opening block pulls in external dependencies such as `numpy`, `torch` and internal modules such as `vllm.envs`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.input_quant_fp8`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `numpy`, `torch`）以及内部模块（如 `vllm.envs`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.input_quant_fp8`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 24-31)
```python
logger = init_logger(__name__)

GPTQ_MARLIN_TILE = 16
GPTQ_MARLIN_MIN_THREAD_N = 64
GPTQ_MARLIN_MIN_THREAD_K = 128
GPTQ_MARLIN_MAX_PARALLEL = 16

MARLIN_SUPPORTED_GROUP_SIZES = [-1, 32, 64, 128]
```
**EN:** This block defines module-level metadata or constants such as `logger`, `GPTQ_MARLIN_TILE`, `GPTQ_MARLIN_MIN_THREAD_N`, `GPTQ_MARLIN_MIN_THREAD_K`, `GPTQ_MARLIN_MAX_PARALLEL`, `MARLIN_SUPPORTED_GROUP_SIZES`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`, `GPTQ_MARLIN_TILE`, `GPTQ_MARLIN_MIN_THREAD_N`, `GPTQ_MARLIN_MIN_THREAD_K`, `GPTQ_MARLIN_MAX_PARALLEL`, `MARLIN_SUPPORTED_GROUP_SIZES`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `query_marlin_supported_quant_types` (lines 42-80)
```python
def query_marlin_supported_quant_types(
    has_zp: bool | None = None,
    include_fp_type: bool = True,
    device_capability: int | None = None,
):
    if current_platform.is_cpu():
        return _query_cpu_marlin_supported_quant_types(has_zp, include_fp_type)

    if not current_platform.is_rocm():
        if device_capability is None:
            capability_tuple = current_platform.get_device_capability()
            device_capability = (
                -1 if capability_tuple is None else capability_tuple.to_int()
            )

        if device_capability < 75:
            return []

    # - has_zp is True: return quant_types that has zero points
    # - has_zp is False: return quant_types that has not zero points
    # - has_zp is None: both
    if has_zp is None:
        types0 = query_marlin_supported_quant_types(
            False, include_fp_type, device_capability
        )
        types1 = query_marlin_supported_quant_types(
            True, include_fp_type, device_capability
        )
        return types0 + types1

    if has_zp:
        # AWQ style, unsigned + runtime zero-point
        return [scalar_types.uint4]
    else:
        # GPTQ style, unsigned + symmetric bias
        res = [scalar_types.uint4b8, scalar_types.uint8b128]
        if include_fp_type:
            res += [scalar_types.float8_e4m3fn, scalar_types.float4_e2m1f]
        return res
```
**EN:** Defines function `query_marlin_supported_quant_types` with signature `query_marlin_supported_quant_types(has_zp: bool | None=None, include_fp_type: bool=True, device_capability: int | None=None)`. It mainly works with `has_zp`, `include_fp_type`, `device_capability`; handles quantization-related transformation logic. The body uses branching. Key calls include `current_platform.is_cpu`, `_query_cpu_marlin_supported_quant_types`, `current_platform.is_rocm`, `query_marlin_supported_quant_types`, `current_platform.get_device_capability`, `capability_tuple.to_int`.
**CN:** 定义函数 `query_marlin_supported_quant_types`，其签名为 `query_marlin_supported_quant_types(has_zp: bool | None=None, include_fp_type: bool=True, device_capability: int | None=None)`。它主要围绕 `has_zp`, `include_fp_type`, `device_capability` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `current_platform.is_cpu`, `_query_cpu_marlin_supported_quant_types`, `current_platform.is_rocm`, `query_marlin_supported_quant_types`, `current_platform.get_device_capability`, `capability_tuple.to_int`。

### Function `_check_marlin_supported` (lines 110-142)
```python
def _check_marlin_supported(
    quant_type: ScalarType,
    group_size: int | None,
    has_zp: bool,
    device_capability: int | None = None,
) -> tuple[bool, str | None]:
    if device_capability is None:
        capability_tuple = current_platform.get_device_capability()
        device_capability = (
            -1 if capability_tuple is None else capability_tuple.to_int()
        )

    supported_types = query_marlin_supported_quant_types(
        has_zp, True, device_capability
    )

    if quant_type not in supported_types:
        return (
            False,
            f"Marlin does not support weight_bits = {quant_type}. "
            f"Only types = {supported_types} "
            f"are supported (for group_size = {group_size}, "
            f"device_capability = {device_capability}, zp = {has_zp}).",
        )
    if group_size is None or group_size not in MARLIN_SUPPORTED_GROUP_SIZES:
        return (
            False,
            f"Marlin does not support group_size = {group_size}. "
            f"Only group_sizes = {MARLIN_SUPPORTED_GROUP_SIZES} "
            "are supported.",
        )

    return True, None
```
**EN:** Defines function `_check_marlin_supported` with signature `_check_marlin_supported(quant_type: ScalarType, group_size: int | None, has_zp: bool, device_capability: int | None=None) -> tuple[bool, str | None]`. It mainly works with `quant_type`, `group_size`, `has_zp`, `device_capability`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `query_marlin_supported_quant_types`, `current_platform.get_device_capability`, `capability_tuple.to_int`.
**CN:** 定义函数 `_check_marlin_supported`，其签名为 `_check_marlin_supported(quant_type: ScalarType, group_size: int | None, has_zp: bool, device_capability: int | None=None) -> tuple[bool, str | None]`。它主要围绕 `quant_type`, `group_size`, `has_zp`, `device_capability` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `query_marlin_supported_quant_types`, `current_platform.get_device_capability`, `capability_tuple.to_int`。

### Function `get_scale_perms` (lines 293-300)
```python
def get_scale_perms():
    scale_perm: list[int] = []
    for i in range(8):
        scale_perm.extend([i + 8 * j for j in range(8)])
    scale_perm_single: list[int] = []
    for i in range(4):
        scale_perm_single.extend([2 * i + j for j in [0, 1, 8, 9, 16, 17, 24, 25]])
    return scale_perm, scale_perm_single
```
**EN:** Defines function `get_scale_perms` with signature `get_scale_perms()`. It mainly works with object context only; returns a derived property or capability check. The body uses iteration, comprehensions. Key calls include `range`, `scale_perm.extend`, `scale_perm_single.extend`.
**CN:** 定义函数 `get_scale_perms`，其签名为 `get_scale_perms()`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含循环处理、推导式。关键调用包括 `range`, `scale_perm.extend`, `scale_perm_single.extend`。

### Function `get__quant_fp8_method` (lines 468-472)
```python
def get__quant_fp8_method() -> QuantFP8:
    global _quant_fp8_method
    if _quant_fp8_method is None:
        _quant_fp8_method = QuantFP8(False, GroupShape.PER_TOKEN)
    return _quant_fp8_method
```
**EN:** Defines function `get__quant_fp8_method` with signature `get__quant_fp8_method() -> QuantFP8`. It mainly works with object context only; handles quantization-related transformation logic. The body uses branching. Key calls include `QuantFP8`.
**CN:** 定义函数 `get__quant_fp8_method`，其签名为 `get__quant_fp8_method() -> QuantFP8`。它主要围绕 仅依赖对象上下文 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `QuantFP8`。

### Function `get_marlin_input_dtype` (lines 475-494)
```python
def get_marlin_input_dtype(prefix: str | None = None):
    if envs.VLLM_MARLIN_INPUT_DTYPE is None:
        return
    elif envs.VLLM_MARLIN_INPUT_DTYPE.lower() == "int8":
        return torch.int8
    elif envs.VLLM_MARLIN_INPUT_DTYPE.lower() == "fp8":
        if not current_platform.is_device_capability(
            89
        ) and not current_platform.is_device_capability_family(120):
            raise ValueError(
                "Marlin W4A8-FP8 only support SM89 or SM12x device "
                "(It is slower than Marlin W4A16 on other devices). "
                "You can consider using W4A8-INT8 instead"
                "(set VLLM_MARLIN_INPUT_DTYPE=int8)."
            )

        _ = get__quant_fp8_method()
        return torch.float8_e4m3fn
    else:
        return
```
**EN:** Defines function `get_marlin_input_dtype` with signature `get_marlin_input_dtype(prefix: str | None=None)`. It mainly works with `prefix`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `envs.VLLM_MARLIN_INPUT_DTYPE.lower`, `get__quant_fp8_method`, `ValueError`, `current_platform.is_device_capability`, `current_platform.is_device_capability_family`.
**CN:** 定义函数 `get_marlin_input_dtype`，其签名为 `get_marlin_input_dtype(prefix: str | None=None)`。它主要围绕 `prefix` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `envs.VLLM_MARLIN_INPUT_DTYPE.lower`, `get__quant_fp8_method`, `ValueError`, `current_platform.is_device_capability`, `current_platform.is_device_capability_family`。

### Function `apply_gptq_marlin_linear` (lines 507-571)
```python
def apply_gptq_marlin_linear(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    weight_zp: torch.Tensor,
    g_idx: torch.Tensor,
    g_idx_sort_indices: torch.Tensor,
    workspace: torch.Tensor,
    wtype: ScalarType,
    output_size_per_partition: int,
    input_size_per_partition: int,
    is_k_full: bool,
    input_global_scale: torch.Tensor | None = None,
    bias: torch.Tensor | None = None,
    use_fp32_reduce: bool = USE_FP32_REDUCE_DEFAULT,
    input_dtype: torch.dtype | None = None,
) -> torch.Tensor:
    reshaped_x = input.reshape(-1, input.shape[-1])
    out_shape = input.shape[:-1] + (output_size_per_partition,)

    use_atomic_add = should_use_atomic_add_reduce(
        m=reshaped_x.size(0),
        n=output_size_per_partition,
        k=reshaped_x.size(1),
        device=input.device,
        dtype=input.dtype,
    )

    a_scales = None
    if input_dtype == torch.int8:
        assert wtype == scalar_types.uint4b8, (
            "W8A8-INT8 is not supported by marlin kernel."
        )
        reshaped_x, a_scales = marlin_quant_input(reshaped_x, input_dtype)
        a_scales = a_scales * input_global_scale
    elif input_dtype == torch.float8_e4m3fn:
        assert wtype == scalar_types.uint4b8, (
            "INT8 weight + FP8 activation is not supported."
# ... truncated for analysis ...
        g_idx,
        g_idx_sort_indices,
        workspace,
        wtype,
        size_m=reshaped_x.shape[0],
        size_n=output_size_per_partition,
        size_k=input_size_per_partition,
        is_k_full=is_k_full,
        use_atomic_add=use_atomic_add,
        use_fp32_reduce=use_fp32_reduce,
        is_zp_float=False,
    )

    return output.reshape(out_shape)
```
**EN:** Defines function `apply_gptq_marlin_linear` with signature `apply_gptq_marlin_linear(input: torch.Tensor, weight: torch.Tensor, weight_scale: torch.Tensor, weight_zp: torch.Tensor, g_idx: torch.Tensor, g_idx_sort_indices: torch.Tensor, workspace: torch.Tensor, wtype: ScalarType, output_size_per_partition: int, input_size_per_partition: int, is_k_full: bool, input_global_scale: torch.Tensor | None=None, bias: torch.Tensor | None=None, use_fp32_reduce: bool=USE_FP32_REDUCE_DEFAULT, input_dtype: torch.dtype | None=None) -> torch.Tensor`. It mainly works with `input`, `weight`, `weight_scale`, `weight_zp`, `g_idx`, `g_idx_sort_indices`, `workspace`, `wtype`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `input.reshape`, `should_use_atomic_add_reduce`, `ops.marlin_gemm`, `output.reshape`, `marlin_quant_input`, `reshaped_x.size`.
**CN:** 定义函数 `apply_gptq_marlin_linear`，其签名为 `apply_gptq_marlin_linear(input: torch.Tensor, weight: torch.Tensor, weight_scale: torch.Tensor, weight_zp: torch.Tensor, g_idx: torch.Tensor, g_idx_sort_indices: torch.Tensor, workspace: torch.Tensor, wtype: ScalarType, output_size_per_partition: int, input_size_per_partition: int, is_k_full: bool, input_global_scale: torch.Tensor | None=None, bias: torch.Tensor | None=None, use_fp32_reduce: bool=USE_FP32_REDUCE_DEFAULT, input_dtype: torch.dtype | None=None) -> torch.Tensor`。它主要围绕 `input`, `weight`, `weight_scale`, `weight_zp`, `g_idx`, `g_idx_sort_indices`, `workspace`, `wtype` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `input.reshape`, `should_use_atomic_add_reduce`, `ops.marlin_gemm`, `output.reshape`, `marlin_quant_input`, `reshaped_x.size`。

### Function `apply_awq_marlin_linear` (lines 574-635)
```python
def apply_awq_marlin_linear(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    weight_zp: torch.Tensor,
    g_idx: torch.Tensor,
    g_idx_sort_indices: torch.Tensor,
    workspace: torch.Tensor,
    quant_type: ScalarType,
    output_size_per_partition: int,
    input_size_per_partition: int,
    input_global_scale: torch.Tensor | None = None,
    bias: torch.Tensor | None = None,
    use_fp32_reduce: bool = USE_FP32_REDUCE_DEFAULT,
    input_dtype: torch.dtype | None = None,
) -> torch.Tensor:
    reshaped_x = input.reshape(-1, input.shape[-1])
    out_shape = input.shape[:-1] + (output_size_per_partition,)

    use_atomic_add = should_use_atomic_add_reduce(
        m=reshaped_x.size(0),
        n=output_size_per_partition,
        k=reshaped_x.size(1),
        device=input.device,
        dtype=input.dtype,
    )

    a_scales = None
    if input_dtype == torch.int8:
        assert quant_type == scalar_types.uint4, (
            "W8A8-INT8 is not supported by marlin kernel."
        )
        reshaped_x, a_scales = marlin_quant_input(reshaped_x, input_dtype)
        a_scales = a_scales * input_global_scale
    elif input_dtype == torch.float8_e4m3fn:
        assert quant_type == scalar_types.uint4, (
            "INT8 weight + FP8 activation is not supported."
        )
# ... truncated for analysis ...
        weight_zp,
        g_idx,
        g_idx_sort_indices,
        workspace,
        quant_type,
        size_m=reshaped_x.shape[0],
        size_n=output_size_per_partition,
        size_k=input_size_per_partition,
        use_atomic_add=use_atomic_add,
        use_fp32_reduce=use_fp32_reduce,
        is_zp_float=False,
    )

    return output.reshape(out_shape)
```
**EN:** Defines function `apply_awq_marlin_linear` with signature `apply_awq_marlin_linear(input: torch.Tensor, weight: torch.Tensor, weight_scale: torch.Tensor, weight_zp: torch.Tensor, g_idx: torch.Tensor, g_idx_sort_indices: torch.Tensor, workspace: torch.Tensor, quant_type: ScalarType, output_size_per_partition: int, input_size_per_partition: int, input_global_scale: torch.Tensor | None=None, bias: torch.Tensor | None=None, use_fp32_reduce: bool=USE_FP32_REDUCE_DEFAULT, input_dtype: torch.dtype | None=None) -> torch.Tensor`. It mainly works with `input`, `weight`, `weight_scale`, `weight_zp`, `g_idx`, `g_idx_sort_indices`, `workspace`, `quant_type`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `input.reshape`, `should_use_atomic_add_reduce`, `ops.marlin_gemm`, `output.reshape`, `marlin_quant_input`, `reshaped_x.size`.
**CN:** 定义函数 `apply_awq_marlin_linear`，其签名为 `apply_awq_marlin_linear(input: torch.Tensor, weight: torch.Tensor, weight_scale: torch.Tensor, weight_zp: torch.Tensor, g_idx: torch.Tensor, g_idx_sort_indices: torch.Tensor, workspace: torch.Tensor, quant_type: ScalarType, output_size_per_partition: int, input_size_per_partition: int, input_global_scale: torch.Tensor | None=None, bias: torch.Tensor | None=None, use_fp32_reduce: bool=USE_FP32_REDUCE_DEFAULT, input_dtype: torch.dtype | None=None) -> torch.Tensor`。它主要围绕 `input`, `weight`, `weight_scale`, `weight_zp`, `g_idx`, `g_idx_sort_indices`, `workspace`, `quant_type` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `input.reshape`, `should_use_atomic_add_reduce`, `ops.marlin_gemm`, `output.reshape`, `marlin_quant_input`, `reshaped_x.size`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `query_marlin_supported_quant_types`, `_query_cpu_marlin_supported_quant_types`, `_check_marlin_supported`, `check_marlin_supported`, `verify_marlin_supported` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `query_marlin_supported_quant_types`, `_query_cpu_marlin_supported_quant_types`, `_check_marlin_supported`, `check_marlin_supported`, `verify_marlin_supported` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `numpy`, `torch`
- **Internal / 内部**: `vllm.envs`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.int8_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.scalar_type`, `vllm.utils.platform_utils`, `.quant_utils`
