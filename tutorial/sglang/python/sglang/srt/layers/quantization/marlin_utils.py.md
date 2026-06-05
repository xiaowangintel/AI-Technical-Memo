# marlin_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/marlin_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for marlin utils quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 Marlin 工具 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/layers/quantization/utils/marlin_utils.py

from __future__ import annotations

import logging
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, Optional

import numpy
import torch

from sglang.srt.layers.parameter import (
    BasevLLMParameter,
    ChannelQuantScaleParameter,
    GroupQuantScaleParameter,
    PackedvLLMParameter,
)
from sglang.srt.layers.quantization.base_config import (
    LinearMethodBase,
    QuantizationConfig,
)
from sglang.srt.layers.quantization.utils import (
    get_scalar_types,
    pack_cols,
    unpack_cols,
)
from sglang.srt.utils import get_device_capability, is_cuda
from sglang.srt.utils.custom_op import register_custom_op
```
**EN:** This block imports __future__, dataclasses, logging, numpy, sglang.jit_kernel.gptq_marlin, sglang.srt.compilation.piecewise_context_manager, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton.layer and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, dataclasses, logging, numpy, sglang.jit_kernel.gptq_marlin, sglang.srt.compilation.piecewise_context_manager, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton.layer 等依赖，并为当前量化实现准备模块命名空间。

### Lines 32-34: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.linear import LinearBase
    from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 36-36: module imports and setup
```python
from sglang.srt.compilation.piecewise_context_manager import get_forward_context
```
**EN:** This block imports __future__, dataclasses, logging, numpy, sglang.jit_kernel.gptq_marlin, sglang.srt.compilation.piecewise_context_manager, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton.layer and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, dataclasses, logging, numpy, sglang.jit_kernel.gptq_marlin, sglang.srt.compilation.piecewise_context_manager, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton.layer 等依赖，并为当前量化实现准备模块命名空间。

### Lines 38-41: guarded import or fallback path
```python
try:
    from vllm import _custom_ops as ops
except ImportError:
    ops = None
```
**EN:** This block uses exception handling to provide a fallback implementation when optional functionality is unavailable.
**CN:** 该代码块通过异常处理在可选功能不可用时提供后备实现。

### Lines 44-44: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 46-47: conditional logic for _is_cuda
```python
if _is_cuda:
    from sglang.jit_kernel.gptq_marlin import gptq_marlin_gemm
```
**EN:** This block applies conditional logic controlled by `_is_cuda`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_cuda` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 49-49: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 51-51: initialize (ScalarType, scalar_types)
```python
ScalarType, scalar_types = get_scalar_types()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as (ScalarType, scalar_types).
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 (ScalarType, scalar_types)。

### Lines 53-53: initialize GPTQ_MARLIN_TILE
```python
GPTQ_MARLIN_TILE = 16
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as GPTQ_MARLIN_TILE.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 GPTQ_MARLIN_TILE。

### Lines 54-54: initialize GPTQ_MARLIN_MIN_THREAD_N
```python
GPTQ_MARLIN_MIN_THREAD_N = 64
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as GPTQ_MARLIN_MIN_THREAD_N.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 GPTQ_MARLIN_MIN_THREAD_N。

### Lines 55-55: initialize GPTQ_MARLIN_MIN_THREAD_K
```python
GPTQ_MARLIN_MIN_THREAD_K = 128
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as GPTQ_MARLIN_MIN_THREAD_K.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 GPTQ_MARLIN_MIN_THREAD_K。

### Lines 56-56: initialize GPTQ_MARLIN_MAX_PARALLEL
```python
GPTQ_MARLIN_MAX_PARALLEL = 16
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as GPTQ_MARLIN_MAX_PARALLEL.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 GPTQ_MARLIN_MAX_PARALLEL。

### Lines 58-58: initialize MARLIN_SUPPORTED_GROUP_SIZES
```python
MARLIN_SUPPORTED_GROUP_SIZES = [-1, 32, 64, 128]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as MARLIN_SUPPORTED_GROUP_SIZES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 MARLIN_SUPPORTED_GROUP_SIZES。

### Lines 63-63: initialize USE_FP32_REDUCE_DEFAULT
```python
USE_FP32_REDUCE_DEFAULT = True
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as USE_FP32_REDUCE_DEFAULT.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 USE_FP32_REDUCE_DEFAULT。

### Lines 66-67: class MarlinLinearLayerConfig: definition
```python
@dataclass
class MarlinLinearLayerConfig:
```
**EN:** This block declares `MarlinLinearLayerConfig`, a configuration class for the quantization stack. It organizes behaviors such as class state.
**CN:** 该代码块声明 `MarlinLinearLayerConfig`，它是量化栈中的配置类，组织了 class state 等行为。

### Lines 68-68: MarlinLinearLayerConfig member: annotate full_weight_shape
```python
    full_weight_shape: tuple[int, int]  # [in, out]
```
**EN:** This block declares and initializes the annotated symbol `full_weight_shape`.
**CN:** 该代码块声明并初始化带类型注解的符号 `full_weight_shape`。

### Lines 69-69: MarlinLinearLayerConfig member: annotate partition_weight_shape
```python
    partition_weight_shape: tuple[int, int]
```
**EN:** This block declares and initializes the annotated symbol `partition_weight_shape`.
**CN:** 该代码块声明并初始化带类型注解的符号 `partition_weight_shape`。

### Lines 70-70: MarlinLinearLayerConfig member: annotate weight_type
```python
    weight_type: ScalarType
```
**EN:** This block declares and initializes the annotated symbol `weight_type`.
**CN:** 该代码块声明并初始化带类型注解的符号 `weight_type`。

### Lines 71-71: MarlinLinearLayerConfig member: annotate act_type
```python
    act_type: torch.dtype
```
**EN:** This block declares and initializes the annotated symbol `act_type`.
**CN:** 该代码块声明并初始化带类型注解的符号 `act_type`。

### Lines 72-72: MarlinLinearLayerConfig member: annotate group_size
```python
    group_size: int
```
**EN:** This block declares and initializes the annotated symbol `group_size`.
**CN:** 该代码块声明并初始化带类型注解的符号 `group_size`。

### Lines 73-73: MarlinLinearLayerConfig member: annotate zero_points
```python
    zero_points: bool
```
**EN:** This block declares and initializes the annotated symbol `zero_points`.
**CN:** 该代码块声明并初始化带类型注解的符号 `zero_points`。

### Lines 74-74: MarlinLinearLayerConfig member: annotate has_g_idx
```python
    has_g_idx: bool
```
**EN:** This block declares and initializes the annotated symbol `has_g_idx`.
**CN:** 该代码块声明并初始化带类型注解的符号 `has_g_idx`。

### Lines 80-113: query_marlin_supported_quant_types()
```python
def query_marlin_supported_quant_types(
    has_zp: Optional[bool] = None,
    include_fp_type: bool = True,
    device_capability: Optional[int] = None,
):
    if device_capability is None:
        major, minor = get_device_capability()
        capability = major * 10 + minor
        device_capability = -1 if capability is None else capability

    if device_capability < 80:
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
**EN:** This block defines `query_marlin_supported_quant_types()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `query_marlin_supported_quant_types()`，用于处理张量或权重量化逻辑。

### Lines 116-148: _check_marlin_supported()
```python
def _check_marlin_supported(
    quant_type: ScalarType,
    group_size: Optional[int],
    has_zp: bool,
    device_capability: Optional[int] = None,
) -> tuple[bool, Optional[str]]:

    if device_capability is None:
        major, minor = get_device_capability()
        capability = major * 10 + minor
        device_capability = -1 if capability is None else capability

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
**EN:** This block defines `_check_marlin_supported()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_check_marlin_supported()`，用于实现量化栈中的可复用模块逻辑。

### Lines 151-158: check_marlin_supported()
```python
def check_marlin_supported(
    quant_type: ScalarType,
    group_size: int,
    has_zp: bool = False,
    device_capability: Optional[int] = None,
) -> bool:
    cond, _ = _check_marlin_supported(quant_type, group_size, has_zp, device_capability)
    return cond
```
**EN:** This block defines `check_marlin_supported()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `check_marlin_supported()`，用于实现量化栈中的可复用模块逻辑。

### Lines 161-167: verify_marlin_supported()
```python
def verify_marlin_supported(
    quant_type: ScalarType, group_size: int, has_zp: bool = False
) -> None:
    cond, err_msg = _check_marlin_supported(quant_type, group_size, has_zp)
    if not cond:
        assert err_msg is not None
        raise ValueError(err_msg)
```
**EN:** This block defines `verify_marlin_supported()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `verify_marlin_supported()`，用于实现量化栈中的可复用模块逻辑。

### Lines 170-203: verify_marlin_supports_shape()
```python
def verify_marlin_supports_shape(
    output_size_per_partition: int,
    input_size_per_partition: int,
    input_size: int,
    group_size: int,
) -> None:

    # Validate output_size_per_partition
    if output_size_per_partition % GPTQ_MARLIN_MIN_THREAD_N != 0:
        raise ValueError(
            f"Weight output_size_per_partition = "
            f"{output_size_per_partition} is not divisible by "
            f" min_thread_n = {GPTQ_MARLIN_MIN_THREAD_N}. "
            "Consider reducing tensor_parallel_size or running "
            "with --quantization gptq."
        )

    # Validate input_size_per_partition
    if input_size_per_partition % GPTQ_MARLIN_MIN_THREAD_K != 0:
        raise ValueError(
            f"Weight input_size_per_partition = "
            f"{input_size_per_partition} is not divisible "
            f"by min_thread_k = {GPTQ_MARLIN_MIN_THREAD_K}. "
            "Consider reducing tensor_parallel_size or running "
            "with --quantization gptq."
        )

    if group_size < input_size and input_size_per_partition % group_size != 0:
        raise ValueError(
            f"Weight input_size_per_partition = {input_size_per_partition}"
            f" is not divisible by group_size = {group_size}. "
            "Consider reducing tensor_parallel_size or running "
            "with --quantization gptq."
        )
```
**EN:** This block defines `verify_marlin_supports_shape()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `verify_marlin_supports_shape()`，用于实现量化栈中的可复用模块逻辑。

### Lines 206-218: check_marlin_supports_shape()
```python
def check_marlin_supports_shape(
    output_size_per_partition: int,
    input_size_per_partition: int,
    input_size: int,
    group_size: int,
) -> tuple[bool, Optional[str]]:
    try:
        verify_marlin_supports_shape(
            output_size_per_partition, input_size_per_partition, input_size, group_size
        )
    except ValueError as e:
        return False, e.__str__()
    return True, None
```
**EN:** This block defines `check_marlin_supports_shape()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `check_marlin_supports_shape()`，用于实现量化栈中的可复用模块逻辑。

### Lines 221-234: check_marlin_supports_layer()
```python
def check_marlin_supports_layer(layer: LinearBase, group_size: int) -> bool:
    output_size_per_partition = (
        getattr(layer, "output_size_per_partition", None) or layer.output_size
    )
    input_size_per_partition = (
        getattr(layer, "input_size_per_partition", None) or layer.input_size
    )

    return check_marlin_supports_shape(
        output_size_per_partition=output_size_per_partition,
        input_size_per_partition=input_size_per_partition,
        input_size=layer.input_size,
        group_size=group_size,
    )[0]
```
**EN:** This block defines `check_marlin_supports_layer()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `check_marlin_supports_layer()`，用于实现量化栈中的可复用模块逻辑。

### Lines 237-258: check_moe_marlin_supports_layer()
```python
def check_moe_marlin_supports_layer(layer: FusedMoE, group_size: int) -> bool:
    hidden_size = layer.hidden_size
    intermediate_size_per_partition = layer.intermediate_size_per_partition
    # apply_router_weight_on_input is not supported for moe marlin
    supports_router_weight = not layer.moe_runner_config.apply_router_weight_on_input
    # moe marlin requires the activation to be silu
    supports_activation = layer.moe_runner_config.activation == "silu"

    # gate-up: (n, k) = (intermediate_size_per_partition * 2, hidden_size)
    # down: (n, k) = (hidden_size, intermediate_size_per_partition)
    # moe marlin requires n % 128 == 0 and k % 64 == 0
    supports_shape = (
        hidden_size % 128 == 0
        and intermediate_size_per_partition % max(64, group_size) == 0
    )
    supports_group_size = group_size in [-1, 32, 64, 128]
    return (
        supports_shape
        and supports_group_size
        and supports_router_weight
        and supports_activation
    )
```
**EN:** This block defines `check_moe_marlin_supports_layer()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `check_moe_marlin_supports_layer()`，用于实现量化栈中的可复用模块逻辑。

### Lines 261-269: marlin_make_workspace()
```python
def marlin_make_workspace(
    device: torch.device, max_blocks_per_sm: int = 1
) -> torch.Tensor:
    # In the new marlin kernel, we use the num of threadblocks as workspace
    # size. The num of threadblocks is sms_count * max_blocks_per_sm.
    sms = torch.cuda.get_device_properties(device).multi_processor_count
    return torch.zeros(
        sms * max_blocks_per_sm, dtype=torch.int, device=device, requires_grad=False
    )
```
**EN:** This block defines `marlin_make_workspace()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `marlin_make_workspace()`，用于实现量化栈中的可复用模块逻辑。

### Lines 272-273: marlin_is_k_full()
```python
def marlin_is_k_full(act_order: bool, is_row_parallel: bool) -> bool:
    return (not act_order) or (act_order and not is_row_parallel)
```
**EN:** This block defines `marlin_is_k_full()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `marlin_is_k_full()`，用于实现量化栈中的可复用模块逻辑。

### Lines 276-282: marlin_repeat_scales_on_all_ranks()
```python
def marlin_repeat_scales_on_all_ranks(
    act_order: bool, group_size: int, is_row_parallel: bool
) -> bool:
    # Need to repeat scales on every rank if act_ordering or
    # channelwise and RowParallelLinear
    is_channelwise = group_size == -1
    return act_order or (is_channelwise and is_row_parallel)
```
**EN:** This block defines `marlin_repeat_scales_on_all_ranks()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `marlin_repeat_scales_on_all_ranks()`，用于实现量化栈中的可复用模块逻辑。

### Lines 285-288: marlin_make_empty_g_idx()
```python
def marlin_make_empty_g_idx(device: torch.device) -> torch.Tensor:
    return torch.nn.Parameter(
        torch.empty(0, dtype=torch.int, device=device), requires_grad=False
    )
```
**EN:** This block defines `marlin_make_empty_g_idx()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `marlin_make_empty_g_idx()`，用于实现量化栈中的可复用模块逻辑。

### Lines 291-294: marlin_make_empty_zp()
```python
def marlin_make_empty_zp(device: torch.device) -> torch.Tensor:
    return torch.nn.Parameter(
        torch.empty(0, dtype=torch.int, device=device), requires_grad=False
    )
```
**EN:** This block defines `marlin_make_empty_zp()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `marlin_make_empty_zp()`，用于实现量化栈中的可复用模块逻辑。

### Lines 297-299: marlin_sort_g_idx()
```python
def marlin_sort_g_idx(g_idx: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]:
    g_idx_sort_indices = torch.argsort(g_idx).to(torch.int)
    return g_idx[g_idx_sort_indices], g_idx_sort_indices
```
**EN:** This block defines `marlin_sort_g_idx()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `marlin_sort_g_idx()`，用于实现量化栈中的可复用模块逻辑。

### Lines 302-309: get_scale_perms()
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
**EN:** This block defines `get_scale_perms()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `get_scale_perms()`，用于为调用方获取或计算派生值。

### Lines 312-323: marlin_permute_scales()
```python
def marlin_permute_scales(
    s: torch.Tensor, size_k: int, size_n: int, group_size: int
) -> torch.Tensor:

    scale_perm, scale_perm_single = get_scale_perms()
    if group_size < size_k and group_size != -1:
        s = s.reshape((-1, len(scale_perm)))[:, scale_perm]
    else:
        s = s.reshape((-1, len(scale_perm_single)))[:, scale_perm_single]
    s = s.reshape((-1, size_n)).contiguous()

    return s
```
**EN:** This block defines `marlin_permute_scales()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `marlin_permute_scales()`，用于实现量化栈中的可复用模块逻辑。

### Lines 326-330: marlin_permute_bias()
```python
def marlin_permute_bias(s: torch.Tensor) -> torch.Tensor:
    origin_shape = s.shape
    _, scale_perm_single = get_scale_perms()
    s = s.reshape((-1, len(scale_perm_single)))[:, scale_perm_single]
    return s.reshape(*origin_shape).contiguous()
```
**EN:** This block defines `marlin_permute_bias()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `marlin_permute_bias()`，用于实现量化栈中的可复用模块逻辑。

### Lines 333-348: marlin_moe_permute_scales()
```python
def marlin_moe_permute_scales(
    s: torch.Tensor,
    size_k: int,
    size_n: int,
    group_size: int,
):
    num_experts = s.shape[0]
    output = torch.empty(
        (num_experts, s.shape[1], s.shape[2]),
        device=s.device,
        dtype=s.dtype,
    )

    for e in range(num_experts):
        output[e] = marlin_permute_scales(s[e], size_k, size_n, group_size)
    return output
```
**EN:** This block defines `marlin_moe_permute_scales()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `marlin_moe_permute_scales()`，用于实现量化栈中的可复用模块逻辑。

### Lines 351-371: marlin_zero_points()
```python
def marlin_zero_points(
    zp: torch.Tensor, size_k: int, size_n: int, num_bits: int
) -> torch.Tensor:
    # Permute zero-points in a similar way to scales, but do not use the
    # "single" permutation, since zero-points are applied on every MMA
    scale_perm, _ = get_scale_perms()
    zp = zp.reshape((-1, len(scale_perm)))[:, scale_perm]

    # Interleave column dim (for the dequantize code) and pack it to int32
    if num_bits == 4:
        interleave = numpy.array([0, 2, 4, 6, 1, 3, 5, 7])
    elif num_bits == 8:
        interleave = numpy.array([0, 2, 1, 3])
    else:
        raise Exception("num_bits must be 4 or 8, got {}".format(num_bits))

    zp = zp.reshape((-1, len(interleave)))[:, interleave].ravel()
    zp = zp.reshape((-1, size_n)).contiguous()
    zp = pack_cols(zp, num_bits, size_k, size_n)

    return zp
```
**EN:** This block defines `marlin_zero_points()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `marlin_zero_points()`，用于实现量化栈中的可复用模块逻辑。

### Lines 374-395: awq_to_marlin_zero_points()
```python
def awq_to_marlin_zero_points(
    q_zp_packed: torch.Tensor, size_k: int, size_n: int, num_bits: int
) -> torch.Tensor:
    # AWQ zero-points are quantized and packed on the column dim.
    # In addition, the values are permuted based on dequantizer.
    # Here we undo both of these, and then apply marlin permutation
    # and pack it back.
    q_zp = unpack_cols(q_zp_packed, num_bits, size_k, size_n)

    # Undo interleaving (use argsort(..) to get inverse perm)
    if num_bits == 4:
        undo_interleave = numpy.argsort(numpy.array([0, 2, 4, 6, 1, 3, 5, 7]))
    elif num_bits == 8:
        undo_interleave = numpy.argsort(numpy.array([0, 2, 1, 3]))
    else:
        raise Exception("num_bits must be 4 or 8, got {}".format(num_bits))

    q_zp = q_zp.reshape((-1, len(undo_interleave)))[:, undo_interleave].ravel()
    q_zp = q_zp.reshape((-1, size_n)).contiguous()

    marlin_zp = marlin_zero_points(q_zp, size_k, size_n, num_bits)
    return marlin_zp
```
**EN:** This block defines `awq_to_marlin_zero_points()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `awq_to_marlin_zero_points()`，用于实现量化栈中的可复用模块逻辑。

### Lines 398-409: moe_awq_to_marlin_zero_points()
```python
def moe_awq_to_marlin_zero_points(
    q_zp_packed: torch.Tensor, size_k: int, size_n: int, num_bits: int
):
    num_experts = q_zp_packed.shape[0]
    output = torch.empty(
        (num_experts, q_zp_packed.shape[1], q_zp_packed.shape[2]),
        device=q_zp_packed.device,
        dtype=q_zp_packed.dtype,
    )
    for e in range(num_experts):
        output[e] = awq_to_marlin_zero_points(q_zp_packed[e], size_k, size_n, num_bits)
    return output
```
**EN:** This block defines `moe_awq_to_marlin_zero_points()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `moe_awq_to_marlin_zero_points()`，用于实现量化栈中的可复用模块逻辑。

### Lines 412-421: maybe_warn_marlin_atomic_add()
```python
def maybe_warn_marlin_atomic_add(device, dtype):
    if torch.compiler.is_dynamo_compiling():
        return
    device_capability = torch.cuda.get_device_capability(device)
    if device_capability[0] < 9 and dtype == torch.bfloat16:
        logger.info_once(
            "You are running Marlin kernel with bf16 on GPUs before SM90. "
            "You can consider change to fp16 to achieve better performance "
            "if possible."
        )
```
**EN:** This block defines `maybe_warn_marlin_atomic_add()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `maybe_warn_marlin_atomic_add()`，用于实现量化栈中的可复用模块逻辑。

### Lines 424-437: maybe_warn_marlin_atomic_add_env()
```python
def maybe_warn_marlin_atomic_add_env():
    if torch.compiler.is_dynamo_compiling():
        return
    # TODO(yiyun): Need to add sglang's MARLIN_USE_ATOMIC_ADD: bool = False
    if True:
        return
    # if envs.VLLM_MARLIN_USE_ATOMIC_ADD:
    #     return
    logger.info_once(
        "Marlin kernel can achieve better performance for small size_n "
        "with experimental use_atomic_add feature. "
        "You can consider set environment variable "
        "VLLM_MARLIN_USE_ATOMIC_ADD to 1 if possible."
    )
```
**EN:** This block defines `maybe_warn_marlin_atomic_add_env()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `maybe_warn_marlin_atomic_add_env()`，用于实现量化栈中的可复用模块逻辑。

### Lines 440-462: should_use_atomic_add_reduce()
```python
def should_use_atomic_add_reduce(
    m: int, n: int, k: int, device: torch.device, dtype: torch.dtype
) -> bool:

    # the performance of atomicAdd is better than global reduce
    # only when m*n is small and k is large
    if n >= 2048 or k < 2048 or device.type != "cuda":
        return False

    # disable atomicAdd reduce by default,
    # one can enable it with VLLM_MARLIN_USE_ATOMIC_ADD=1
    # TODO: Need to add sglang's MARLIN_USE_ATOMIC_ADD: bool = False
    if not True:
        maybe_warn_marlin_atomic_add_env()
        return False

    # sm8x doesn't support atomicAdd + bfloat16 natively
    device_capability = torch.cuda.get_device_capability(device)
    if device_capability[0] < 9 and dtype == torch.bfloat16:
        maybe_warn_marlin_atomic_add(device, dtype)
        return False

    return True
```
**EN:** This block defines `should_use_atomic_add_reduce()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `should_use_atomic_add_reduce()`，用于实现量化栈中的可复用模块逻辑。

### Lines 465-533: apply_gptq_marlin_linear()
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
    bias: Optional[torch.Tensor] = None,
    use_fp32_reduce: bool = USE_FP32_REDUCE_DEFAULT,
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

    forward_context = get_forward_context()
    if forward_context is None:
        output = gptq_marlin_gemm(
            reshaped_x,
            None,
            weight,
            weight_scale,
            None,
            weight_zp,
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
    else:
        output = unified_apply_gptq_marlin_gemm_with_wtype(
            input=reshaped_x,
            weight=weight,
            weight_scale=weight_scale,
            weight_zp=weight_zp,
            g_idx=g_idx,
            g_idx_sort_indices=g_idx_sort_indices,
            workspace=workspace,
            wtype_id=wtype.id,
            output_size_per_partition=output_size_per_partition,
            input_size_per_partition=input_size_per_partition,
            is_k_full=is_k_full,
            use_atomic_add=use_atomic_add,
            use_fp32_reduce=use_fp32_reduce,
            is_zp_float=False,
        )

    if bias is not None:
        output.add_(bias)  # In-place add

    return output.reshape(out_shape)
```
**EN:** This block defines `apply_gptq_marlin_linear()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `apply_gptq_marlin_linear()`，用于将量化计算应用到运行时输入上。

### Lines 536-600: apply_awq_marlin_linear()
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
    bias: Optional[torch.Tensor] = None,
    use_fp32_reduce: bool = USE_FP32_REDUCE_DEFAULT,
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

    forward_context = get_forward_context()
    if forward_context is None:
        output = gptq_marlin_gemm(
            reshaped_x,
            None,
            weight,
            weight_scale,
            None,
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
    else:
        output = unified_apply_gptq_marlin_gemm(
            input=reshaped_x,
            weight=weight,
            weight_scale=weight_scale,
            weight_zp=weight_zp,
            g_idx=g_idx,
            g_idx_sort_indices=g_idx_sort_indices,
            workspace=workspace,
            output_size_per_partition=output_size_per_partition,
            input_size_per_partition=input_size_per_partition,
            use_atomic_add=use_atomic_add,
            use_fp32_reduce=use_fp32_reduce,
            is_zp_float=False,
        )

    if bias is not None:
        output.add_(bias)  # In-place add

    return output.reshape(out_shape)
```
**EN:** This block defines `apply_awq_marlin_linear()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `apply_awq_marlin_linear()`，用于将量化计算应用到运行时输入上。

### Lines 603-608: class MarlinConfig: definition
```python
class MarlinConfig(QuantizationConfig):
    """Config class for Marlin.

    Reference: https://github.com/IST-DASLab/marlin/tree/master
    """
```
**EN:** This block declares `MarlinConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, __repr__, get_name, get_supported_act_dtypes.
**CN:** 该代码块声明 `MarlinConfig`，它是量化栈中的配置类，组织了 __init__, __repr__, get_name, get_supported_act_dtypes 等行为。

### Lines 609-643: MarlinConfig.__init__()
```python
    def __init__(
        self,
        group_size: int,
        lm_head_quantized: bool,
    ) -> None:
        super().__init__()

        # Group size for the quantization.
        self.group_size = group_size
        self.lm_head_quantized = lm_head_quantized
        if self.group_size != 128 and self.group_size != -1:
            raise ValueError(
                "Currently, only group size 128 and -1 (channelwise) "
                "is supported for Marlin, but got group_size of "
                f"{self.group_size}"
            )

        # 4 Bits packed into 32 bit datatype.
        self.pack_factor = 32 // 4

        # Tile size used by marlin kernels.
        self.tile_size = 16

        # Min out_features dim
        self.min_n_threads = 64

        # Min in_features dim
        self.min_k_threads = 128

        # Max parallel problems to solve at once (improves large
        # batch performance)
        self.max_parallel = 16

        # Permutation length used by the marlin kernels.
        self.perm_len = 1024
```
**EN:** This block defines `MarlinConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `MarlinConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 645-649: MarlinConfig.__repr__()
```python
    def __repr__(self) -> str:
        return (
            f"MarlinConfig(group_size={self.group_size}, "
            f"lm_head_quantized={self.lm_head_quantized})"
        )
```
**EN:** This block defines `MarlinConfig.__repr__()`, which builds a readable debug representation.
**CN:** 该代码块定义了 `MarlinConfig.__repr__()`，用于构造可读的调试表示。

### Lines 651-653: MarlinConfig.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "marlin"
```
**EN:** This block defines `MarlinConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MarlinConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 655-657: MarlinConfig.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.half]
```
**EN:** This block defines `MarlinConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MarlinConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 659-662: MarlinConfig.get_min_capability()
```python
    @classmethod
    # Need to figure it out
    def get_min_capability(cls) -> int:
        return 80
```
**EN:** This block defines `MarlinConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MarlinConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 664-666: MarlinConfig.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> list[str]:
        return ["quantize_config.json"]
```
**EN:** This block defines `MarlinConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MarlinConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 668-672: MarlinConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: dict[str, Any]) -> "MarlinConfig":
        group_size = cls.get_from_keys(config, ["group_size"])
        lm_head_quantized = cls.get_from_keys_or(config, ["lm_head"], default=False)
        return cls(group_size, lm_head_quantized)
```
**EN:** This block defines `MarlinConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `MarlinConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 674-693: MarlinConfig.override_quantization_method()
```python
    @classmethod
    def override_quantization_method(cls, hf_quant_cfg, user_quant) -> Optional[str]:
        # compat: autogptq >=0.8.0 use checkpoint_format: str
        # compat: autogptq <=0.7.1 is_marlin_format: bool
        is_marlin_format = hf_quant_cfg.get(
            "checkpoint_format"
        ) == "marlin" or hf_quant_cfg.get("is_marlin_format", False)

        is_valid_user_quant = (
            user_quant is None or user_quant == "gptq" or user_quant == "marlin"
        )

        if is_marlin_format and is_valid_user_quant:
            msg = "The model is serialized in {} format. Using {} kernel.".format(
                cls.get_name(), cls.get_name()
            )
            logger.info(msg)
            return cls.get_name()

        return None
```
**EN:** This block defines `MarlinConfig.override_quantization_method()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `MarlinConfig.override_quantization_method()`，用于处理张量或权重量化逻辑。

### Lines 695-705: MarlinConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[MarlinLinearMethod]:
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.vocab_parallel_embedding import ParallelLMHead

        if isinstance(layer, LinearBase) or (
            isinstance(layer, ParallelLMHead) and self.lm_head_quantized
        ):
            return MarlinLinearMethod(self)
        return None
```
**EN:** This block defines `MarlinConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MarlinConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 708-714: class MarlinLinearMethod: definition
```python
class MarlinLinearMethod(LinearMethodBase):
    """Linear method for Marlin.

    Args:
        quant_config: The Marlin quantization config.
    """
```
**EN:** This block declares `MarlinLinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `MarlinLinearMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, apply 等行为。

### Lines 715-716: MarlinLinearMethod.__init__()
```python
    def __init__(self, quant_config: MarlinConfig):
        self.quant_config = quant_config
```
**EN:** This block defines `MarlinLinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `MarlinLinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 718-762: MarlinLinearMethod.create_weights() (part 1/3)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        del output_size  # Unused.
        weight_loader = extra_weight_attrs["weight_loader"]

        if params_dtype != torch.float16:
            raise ValueError(
                f"The params dtype must be float16, but got {params_dtype}"
            )

        # Validate output_size_per_partition
        output_size_per_partition = sum(output_partition_sizes)
        if output_size_per_partition % self.quant_config.min_n_threads != 0:
            raise ValueError(
                f"Weight output_size_per_partition = "
                f"{output_size_per_partition} is not divisible by "
                f"min_n_threads = {self.quant_config.min_n_threads}."
            )
        if output_size_per_partition % self.quant_config.pack_factor != 0:
            raise ValueError(
                f"Weight output_size_per_partition = "
                f"{output_size_per_partition} is not divisible by "
                f"pack_factor = {self.quant_config.pack_factor}."
            )

        # Validate input_size_per_partition
        if input_size_per_partition % self.quant_config.min_k_threads != 0:
            raise ValueError(
                f"Weight input_size_per_partition = "
                f"{input_size_per_partition} is not divisible by "
                f"min_k_threads = {self.quant_config.min_k_threads}."
            )
        if (
            self.quant_config.group_size != -1
            and input_size_per_partition % self.quant_config.group_size != 0
        ):
            raise ValueError(
```
**EN:** This segment of `MarlinLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `MarlinLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 763-807: MarlinLinearMethod.create_weights() (part 2/3)
```python
                f"Weight input_size_per_partition = "
                f"{input_size_per_partition} is not divisible by "
                f"group_size = {self.quant_config.group_size}."
            )

        # Check that we have at least 4 tiles horizontally in the shard
        num_tiles_per_perm = self.quant_config.perm_len // (
            self.quant_config.tile_size**2
        )
        if output_size_per_partition % num_tiles_per_perm != 0:
            raise ValueError("Each permutation group must reside on the same gpu")

        # Quantized 4Bit weights packed into Int32.
        qweight = PackedvLLMParameter(
            data=torch.empty(
                input_size_per_partition // self.quant_config.tile_size,
                output_size_per_partition
                * self.quant_config.tile_size
                // self.quant_config.pack_factor,
                device="cuda",
                dtype=torch.int32,
            ),
            input_dim=0,
            output_dim=1,
            packed_dim=1,
            packed_factor=self.quant_config.pack_factor,
            marlin_tile_size=self.quant_config.tile_size,
            weight_loader=weight_loader,
        )

        # Determine if channelwise or not
        input_groups = (
            1
            if self.quant_config.group_size == -1
            else input_size_per_partition // self.quant_config.group_size
        )

        weight_scale_args = {
            "data": torch.empty(
                input_groups,
                output_size_per_partition,
                device="cuda",
                dtype=params_dtype,
            ),
            "weight_loader": weight_loader,
```
**EN:** This segment of `MarlinLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `MarlinLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 808-828: MarlinLinearMethod.create_weights() (part 3/3)
```python
        }
        if input_groups == 1:
            scales = ChannelQuantScaleParameter(output_dim=1, **weight_scale_args)
        else:
            scales = GroupQuantScaleParameter(
                output_dim=1, input_dim=0, **weight_scale_args
            )

        # Allocate workspace (Used for internal locking mechanism)
        max_workspace_size = (
            output_size_per_partition // self.quant_config.min_n_threads
        ) * self.quant_config.max_parallel

        workspace = BasevLLMParameter(
            data=torch.zeros(max_workspace_size, device="cuda", dtype=torch.int),
            weight_loader=weight_loader,
        )

        layer.register_parameter("B", qweight)
        layer.register_parameter("s", scales)
        layer.register_parameter("workspace", workspace)
```
**EN:** This segment of `MarlinLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `MarlinLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 830-834: MarlinLinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # required by torch.compile
        layer.B = torch.nn.Parameter(layer.B.data, requires_grad=False)
        layer.s = torch.nn.Parameter(layer.s.data, requires_grad=False)
        layer.workspace = torch.nn.Parameter(layer.workspace.data, requires_grad=False)
```
**EN:** This block defines `MarlinLinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `MarlinLinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 836-861: MarlinLinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        qweight = layer.B
        scales = layer.s
        workspace = layer.workspace

        x_2d = x.view(-1, x.shape[-1])

        size_m = x_2d.shape[0]
        size_k = x_2d.shape[1]
        size_n = scales.shape[1]

        output_2d = ops.marlin_gemm(
            x_2d, qweight, scales, workspace, size_m, size_n, size_k
        )

        output = output_2d.view(x.shape[:-1] + (output_2d.shape[1],))

        if bias is not None:
            output.add_(bias)  # In-place add

        return output
```
**EN:** This block defines `MarlinLinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `MarlinLinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 864-880: fake_unified_apply_gptq_marlin_gemm()
```python
def fake_unified_apply_gptq_marlin_gemm(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    weight_zp: torch.Tensor,
    g_idx: torch.Tensor,
    g_idx_sort_indices: torch.Tensor,
    workspace: torch.Tensor,
    output_size_per_partition: int,
    input_size_per_partition: int,
    use_atomic_add: bool,
    use_fp32_reduce: bool,
    is_zp_float: bool,
) -> torch.Tensor:
    return input.new_empty(
        (input.shape[0], output_size_per_partition), dtype=input.dtype
    )
```
**EN:** This block defines `fake_unified_apply_gptq_marlin_gemm()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `fake_unified_apply_gptq_marlin_gemm()`，用于实现量化栈中的可复用模块逻辑。

### Lines 883-917: unified_apply_gptq_marlin_gemm()
```python
@register_custom_op(fake_impl=fake_unified_apply_gptq_marlin_gemm)
def unified_apply_gptq_marlin_gemm(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    weight_zp: torch.Tensor,
    g_idx: torch.Tensor,
    g_idx_sort_indices: torch.Tensor,
    workspace: torch.Tensor,
    output_size_per_partition: int,
    input_size_per_partition: int,
    use_atomic_add: bool,
    use_fp32_reduce: bool,
    is_zp_float: bool,
) -> torch.Tensor:
    quant_config = get_forward_context().quant_config
    quant_type = quant_config.quant_type
    return gptq_marlin_gemm(
        input,
        None,
        weight,
        weight_scale,
        None,
        weight_zp,
        g_idx,
        g_idx_sort_indices,
        workspace,
        quant_type,
        size_m=input.shape[0],
        size_n=output_size_per_partition,
        size_k=input_size_per_partition,
        use_atomic_add=use_atomic_add,
        use_fp32_reduce=use_fp32_reduce,
        is_zp_float=is_zp_float,
    )
```
**EN:** This block defines `unified_apply_gptq_marlin_gemm()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `unified_apply_gptq_marlin_gemm()`，用于实现量化栈中的可复用模块逻辑。

### Lines 920-938: fake_unified_apply_gptq_marlin_gemm_with_wtype()
```python
def fake_unified_apply_gptq_marlin_gemm_with_wtype(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    weight_zp: torch.Tensor,
    g_idx: torch.Tensor,
    g_idx_sort_indices: torch.Tensor,
    workspace: torch.Tensor,
    wtype_id: int,
    output_size_per_partition: int,
    input_size_per_partition: int,
    is_k_full: bool,
    use_atomic_add: bool,
    use_fp32_reduce: bool,
    is_zp_float: bool,
) -> torch.Tensor:
    return input.new_empty(
        (input.shape[0], output_size_per_partition), dtype=input.dtype
    )
```
**EN:** This block defines `fake_unified_apply_gptq_marlin_gemm_with_wtype()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `fake_unified_apply_gptq_marlin_gemm_with_wtype()`，用于实现量化栈中的可复用模块逻辑。

### Lines 941-984: unified_apply_gptq_marlin_gemm_with_wtype()
```python
@register_custom_op(fake_impl=fake_unified_apply_gptq_marlin_gemm_with_wtype)
def unified_apply_gptq_marlin_gemm_with_wtype(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    weight_zp: torch.Tensor,
    g_idx: torch.Tensor,
    g_idx_sort_indices: torch.Tensor,
    workspace: torch.Tensor,
    wtype_id: int,
    output_size_per_partition: int,
    input_size_per_partition: int,
    is_k_full: bool,
    use_atomic_add: bool,
    use_fp32_reduce: bool,
    is_zp_float: bool,
) -> torch.Tensor:
    # Reconstruct ScalarType from id
    wtype = None
    for attr_name in dir(scalar_types):
        if not attr_name.startswith("_"):
            st = getattr(scalar_types, attr_name)
            if hasattr(st, "id") and st.id == wtype_id:
                wtype = st
                break
    return gptq_marlin_gemm(
        input,
        None,
        weight,
        weight_scale,
        None,
        weight_zp,
        g_idx,
        g_idx_sort_indices,
        workspace,
        wtype,
        size_m=input.shape[0],
        size_n=output_size_per_partition,
        size_k=input_size_per_partition,
        is_k_full=is_k_full,
        use_atomic_add=use_atomic_add,
        use_fp32_reduce=use_fp32_reduce,
        is_zp_float=is_zp_float,
    )
```
**EN:** This block defines `unified_apply_gptq_marlin_gemm_with_wtype()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `unified_apply_gptq_marlin_gemm_with_wtype()`，用于实现量化栈中的可复用模块逻辑。

## Key Concepts / 关键概念
- `GPTQ_MARLIN_TILE`: A module-level constant or registry. / `GPTQ_MARLIN_TILE`：模块级常量或注册表。
- `GPTQ_MARLIN_MIN_THREAD_N`: A module-level constant or registry. / `GPTQ_MARLIN_MIN_THREAD_N`：模块级常量或注册表。
- `GPTQ_MARLIN_MIN_THREAD_K`: A module-level constant or registry. / `GPTQ_MARLIN_MIN_THREAD_K`：模块级常量或注册表。
- `GPTQ_MARLIN_MAX_PARALLEL`: A module-level constant or registry. / `GPTQ_MARLIN_MAX_PARALLEL`：模块级常量或注册表。
- `MARLIN_SUPPORTED_GROUP_SIZES`: A module-level constant or registry. / `MARLIN_SUPPORTED_GROUP_SIZES`：模块级常量或注册表。
- `USE_FP32_REDUCE_DEFAULT`: A module-level constant or registry. / `USE_FP32_REDUCE_DEFAULT`：模块级常量或注册表。
- `MarlinLinearLayerConfig`: A configuration class that structures file-level quantization behavior. / `MarlinLinearLayerConfig` 是一个配置类，用于组织该文件中的量化行为。
- `query_marlin_supported_quant_types()` : A public function that handles tensor or weight quantization work. / `query_marlin_supported_quant_types()`：一个公开函数，用于处理张量或权重量化逻辑。
- `check_marlin_supported()` : A public function that implements reusable module logic for the quantization stack. / `check_marlin_supported()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `verify_marlin_supported()` : A public function that implements reusable module logic for the quantization stack. / `verify_marlin_supported()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `verify_marlin_supports_shape()` : A public function that implements reusable module logic for the quantization stack. / `verify_marlin_supports_shape()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `check_marlin_supports_shape()` : A public function that implements reusable module logic for the quantization stack. / `check_marlin_supports_shape()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `check_marlin_supports_layer()` : A public function that implements reusable module logic for the quantization stack. / `check_marlin_supports_layer()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `check_moe_marlin_supports_layer()` : A public function that implements reusable module logic for the quantization stack. / `check_moe_marlin_supports_layer()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `marlin_make_workspace()` : A public function that implements reusable module logic for the quantization stack. / `marlin_make_workspace()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `marlin_is_k_full()` : A public function that implements reusable module logic for the quantization stack. / `marlin_is_k_full()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `marlin_repeat_scales_on_all_ranks()` : A public function that implements reusable module logic for the quantization stack. / `marlin_repeat_scales_on_all_ranks()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `marlin_make_empty_g_idx()` : A public function that implements reusable module logic for the quantization stack. / `marlin_make_empty_g_idx()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `marlin_make_empty_zp()` : A public function that implements reusable module logic for the quantization stack. / `marlin_make_empty_zp()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `marlin_sort_g_idx()` : A public function that implements reusable module logic for the quantization stack. / `marlin_sort_g_idx()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `get_scale_perms()` : A public function that retrieves or computes a derived value for callers. / `get_scale_perms()`：一个公开函数，用于为调用方获取或计算派生值。
- `marlin_permute_scales()` : A public function that implements reusable module logic for the quantization stack. / `marlin_permute_scales()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `marlin_permute_bias()` : A public function that implements reusable module logic for the quantization stack. / `marlin_permute_bias()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `marlin_moe_permute_scales()` : A public function that implements reusable module logic for the quantization stack. / `marlin_moe_permute_scales()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `marlin_zero_points()` : A public function that implements reusable module logic for the quantization stack. / `marlin_zero_points()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `awq_to_marlin_zero_points()` : A public function that implements reusable module logic for the quantization stack. / `awq_to_marlin_zero_points()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `moe_awq_to_marlin_zero_points()` : A public function that implements reusable module logic for the quantization stack. / `moe_awq_to_marlin_zero_points()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `maybe_warn_marlin_atomic_add()` : A public function that implements reusable module logic for the quantization stack. / `maybe_warn_marlin_atomic_add()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `maybe_warn_marlin_atomic_add_env()` : A public function that implements reusable module logic for the quantization stack. / `maybe_warn_marlin_atomic_add_env()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `should_use_atomic_add_reduce()` : A public function that implements reusable module logic for the quantization stack. / `should_use_atomic_add_reduce()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `apply_gptq_marlin_linear()` : A public function that applies quantized computation to runtime inputs. / `apply_gptq_marlin_linear()`：一个公开函数，用于将量化计算应用到运行时输入上。
- `apply_awq_marlin_linear()` : A public function that applies quantized computation to runtime inputs. / `apply_awq_marlin_linear()`：一个公开函数，用于将量化计算应用到运行时输入上。
- `MarlinConfig`: A configuration class that structures file-level quantization behavior. / `MarlinConfig` 是一个配置类，用于组织该文件中的量化行为。
- `MarlinLinearMethod`: A runtime method class that structures file-level quantization behavior. / `MarlinLinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `fake_unified_apply_gptq_marlin_gemm()` : A public function that implements reusable module logic for the quantization stack. / `fake_unified_apply_gptq_marlin_gemm()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `unified_apply_gptq_marlin_gemm()` : A public function that implements reusable module logic for the quantization stack. / `unified_apply_gptq_marlin_gemm()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `fake_unified_apply_gptq_marlin_gemm_with_wtype()` : A public function that implements reusable module logic for the quantization stack. / `fake_unified_apply_gptq_marlin_gemm_with_wtype()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `unified_apply_gptq_marlin_gemm_with_wtype()` : A public function that implements reusable module logic for the quantization stack. / `unified_apply_gptq_marlin_gemm_with_wtype()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `dataclasses`, `logging`, `numpy`, `torch`, `typing`, `vllm`
- **Internal / 内部**: `sglang.jit_kernel.gptq_marlin`, `sglang.srt.compilation.piecewise_context_manager`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe.fused_moe_triton.layer`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.utils`, `sglang.srt.layers.vocab_parallel_embedding`, `sglang.srt.utils`, `sglang.srt.utils.custom_op`
