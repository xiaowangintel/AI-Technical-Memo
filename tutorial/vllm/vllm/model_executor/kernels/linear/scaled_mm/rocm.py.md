# rocm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/scaled_mm/rocm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the ROCm scaled matrix-multiplication kernel path. / 实现 ROCm 的缩放矩阵乘内核路径。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-16)
```python
import torch

import vllm.envs as envs
from vllm import _custom_ops as ops
from vllm.platforms import current_platform
from vllm.utils.platform_utils import num_compute_units
from vllm.utils.torch_utils import direct_register_custom_op

from .ScaledMMLinearKernel import (
    FP8ScaledMMLinearKernel,
    FP8ScaledMMLinearLayerConfig,
)
```
**EN:** This import block loads `torch`, `vllm.envs`, `vllm`, `vllm.platforms`, `vllm.utils.platform_utils`, `vllm.utils.torch_utils`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `rocm.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm.envs`, `vllm`, `vllm.platforms`, `vllm.utils.platform_utils`, `vllm.utils.torch_utils`, ...，为 `rocm.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Function `rocm_per_tensor_float_w8a8_scaled_mm_impl` (lines 19-52)
```python
def rocm_per_tensor_float_w8a8_scaled_mm_impl(
    A: torch.Tensor,
    B: torch.Tensor,
    out_dtype: torch.dtype,
    As: torch.Tensor,
    Bs: torch.Tensor,
    bias: torch.Tensor,
) -> torch.Tensor:
    if (
        A.shape[0] <= 4
        and B.shape[0] % 16 == 0  # M TODO: needed?
        and B.shape[1] % 16 == 0  # K
        and ((bias is None) or (bias.dtype == out_dtype))
    ):
        output = ops.wvSplitKQ(
            B.t(),
            A,
            out_dtype,
            As,
            Bs,
            num_compute_units(),
            bias,
        )
    # Fallback
    else:
        output = torch._scaled_mm(
            A,
            B,
            out_dtype=out_dtype,
            scale_a=As,
            scale_b=Bs,
            bias=bias,
        )
    return output
```
**EN:** This helper implements `rocm_per_tensor_float_w8a8_scaled_mm_impl`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `wvSplitKQ`, `_scaled_mm`, `t`, `num_compute_units`.
**CN:** 该函数 `rocm_per_tensor_float_w8a8_scaled_mm_impl` 封装了此模块中的一段关键运行时逻辑，重点处理 rocm per tensor float w 8 a 8 scaled mm impl 相关工作。 它内部会调用 `wvSplitKQ`, `_scaled_mm`, `t`, `num_compute_units` 等例程。

### Function `rocm_per_tensor_float_w8a8_scaled_mm_fake` (lines 55-63)
```python
def rocm_per_tensor_float_w8a8_scaled_mm_fake(
    A: torch.Tensor,
    B: torch.Tensor,
    out_dtype: torch.dtype,
    As: torch.Tensor,
    Bs: torch.Tensor,
    bias: torch.Tensor,
) -> torch.Tensor:
    return A.new_empty((*A.shape[:-1], B.shape[1]), dtype=out_dtype)
```
**EN:** This helper implements `rocm_per_tensor_float_w8a8_scaled_mm_fake`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `new_empty`.
**CN:** 该函数 `rocm_per_tensor_float_w8a8_scaled_mm_fake` 封装了此模块中的一段关键运行时逻辑，重点处理 rocm per tensor float w 8 a 8 scaled mm fake 相关工作。 它内部会调用 `new_empty` 等例程。

### Conditional setup (lines 66-71)
```python
if current_platform.is_rocm():
    direct_register_custom_op(
        op_name="rocm_per_tensor_float_w8a8_scaled_mm_impl",
        op_func=rocm_per_tensor_float_w8a8_scaled_mm_impl,
        fake_impl=rocm_per_tensor_float_w8a8_scaled_mm_fake,
    )
```
**EN:** This conditional branch handles the top-level case `current_platform.is_rocm()`, selecting the appropriate setup path for the module.
**CN:** 该条件分支处理顶层情况 `current_platform.is_rocm()`，为模块选择合适的初始化路径。

### Class `ROCmFP8ScaledMMLinearKernel` (lines 74-118)
```python
class ROCmFP8ScaledMMLinearKernel(FP8ScaledMMLinearKernel):
    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_rocm():
            return False, "requires ROCm."

        from vllm.platforms.rocm import on_gfx12x, on_mi3xx

        if not (on_mi3xx() or on_gfx12x()):
            return False, "requires MI3xx or gfx12x"

        if not envs.VLLM_ROCM_USE_SKINNY_GEMM:
            return False, "requires VLLM_ROCM_USE_SKINNY_GEMM to be enabled."

        return True, None

    @classmethod
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        per_tensor_activation_scales = (
            c.activation_quant_key.scale.group_shape.is_per_tensor()
        )
        per_tensor_weight_scales = c.weight_quant_key.scale.group_shape.is_per_tensor()

        if not (per_tensor_activation_scales and per_tensor_weight_scales):
            return False, "requires per tensor activation and weight scales."

        return True, None

    def apply_scaled_mm(
        self,
        *,
        A: torch.Tensor,
        B: torch.Tensor,
        out_dtype: torch.dtype,
        As: torch.Tensor,
        Bs: torch.Tensor,
        bias: torch.Tensor | None,
        output_shape: list,
    ) -> torch.Tensor:
        output = torch.ops.vllm.rocm_per_tensor_float_w8a8_scaled_mm_impl(
            A, B, out_dtype, As, Bs, bias
        )
        return torch.narrow(output, 0, 0, A.shape[0]).view(*output_shape)
```
**EN:** This kernel class defines `ROCmFP8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `FP8ScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `apply_scaled_mm`.
**CN:** 该内核类定义了 `ROCmFP8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `FP8ScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `apply_scaled_mm`。

### Method `ROCmFP8ScaledMMLinearKernel.is_supported` (lines 76-90)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_rocm():
            return False, "requires ROCm."

        from vllm.platforms.rocm import on_gfx12x, on_mi3xx

        if not (on_mi3xx() or on_gfx12x()):
            return False, "requires MI3xx or gfx12x"

        if not envs.VLLM_ROCM_USE_SKINNY_GEMM:
            return False, "requires VLLM_ROCM_USE_SKINNY_GEMM to be enabled."

        return True, None
```
**EN:** This method implements `ROCmFP8ScaledMMLinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_rocm`, `on_mi3xx`, `on_gfx12x`.
**CN:** 该方法 `ROCmFP8ScaledMMLinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_rocm`, `on_mi3xx`, `on_gfx12x` 等例程。

### Method `ROCmFP8ScaledMMLinearKernel.can_implement` (lines 93-102)
```python
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        per_tensor_activation_scales = (
            c.activation_quant_key.scale.group_shape.is_per_tensor()
        )
        per_tensor_weight_scales = c.weight_quant_key.scale.group_shape.is_per_tensor()

        if not (per_tensor_activation_scales and per_tensor_weight_scales):
            return False, "requires per tensor activation and weight scales."

        return True, None
```
**EN:** This method implements `ROCmFP8ScaledMMLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_per_tensor`.
**CN:** 该方法 `ROCmFP8ScaledMMLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `is_per_tensor` 等例程。

### Method `ROCmFP8ScaledMMLinearKernel.apply_scaled_mm` (lines 104-118)
```python
    def apply_scaled_mm(
        self,
        *,
        A: torch.Tensor,
        B: torch.Tensor,
        out_dtype: torch.dtype,
        As: torch.Tensor,
        Bs: torch.Tensor,
        bias: torch.Tensor | None,
        output_shape: list,
    ) -> torch.Tensor:
        output = torch.ops.vllm.rocm_per_tensor_float_w8a8_scaled_mm_impl(
            A, B, out_dtype, As, Bs, bias
        )
        return torch.narrow(output, 0, 0, A.shape[0]).view(*output_shape)
```
**EN:** This method implements `ROCmFP8ScaledMMLinearKernel.apply_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `rocm_per_tensor_float_w8a8_scaled_mm_impl`, `view`, `narrow`.
**CN:** 该方法 `ROCmFP8ScaledMMLinearKernel.apply_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply scaled mm 相关工作。 它内部会调用 `rocm_per_tensor_float_w8a8_scaled_mm_impl`, `view`, `narrow` 等例程。

## Key Concepts / 关键概念
- **ROCm support / ROCm 支持**
  - **EN:** The implementation targets ROCm-capable devices and their kernel constraints.
  - **CN:** 该实现面向 ROCm 设备及其内核约束。
- **FP8 quantization / FP8 量化**
  - **EN:** The module handles FP8 data, scales, or kernels for low-precision inference.
  - **CN:** 该模块处理 FP8 数据、缩放因子或相关低精度推理内核。
- **Scaled matrix multiplication / 缩放矩阵乘**
  - **EN:** The code centers on scaled matrix multiplication kernels and their dispatch rules.
  - **CN:** 代码围绕缩放矩阵乘内核及其分派规则展开。
- **Linear kernel abstraction / 线性内核抽象**
  - **EN:** The module defines abstractions or selectors for quantized linear operators.
  - **CN:** 该模块定义量化线性算子的抽象或选择器。
- **Kernel registration / 内核注册**
  - **EN:** The file registers operators, backends, or preset configurations with PyTorch/vLLM.
  - **CN:** 该文件向 PyTorch/vLLM 注册算子、后端或预设配置。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.envs`, `vllm`, `vllm.platforms`, `vllm.utils.platform_utils`, `vllm.utils.torch_utils`, `.ScaledMMLinearKernel`
- **External / 外部依赖**: `torch`
