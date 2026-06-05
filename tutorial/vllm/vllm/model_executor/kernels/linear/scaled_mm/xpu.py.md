# xpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/scaled_mm/xpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the XPU scaled matrix-multiplication kernel path. / 实现 XPU 的缩放矩阵乘内核路径。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-17)
```python
from collections.abc import Sequence

import torch

from vllm.model_executor.kernels.linear import (  # noqa: E501
    FP8ScaledMMLinearKernel,
    FP8ScaledMMLinearLayerConfig,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    kFp8StaticChannelSym,
    kFp8StaticTensorSym,
)
from vllm.model_executor.utils import replace_parameter
from vllm.platforms import current_platform
```
**EN:** This import block loads `collections.abc`, `torch`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.utils`, `vllm.platforms`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `xpu.py`.
**CN:** 该导入代码块加载了 `collections.abc`, `torch`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.utils`, `vllm.platforms`，为 `xpu.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Class `XPUFP8ScaledMMLinearKernel` (lines 20-86)
```python
class XPUFP8ScaledMMLinearKernel(FP8ScaledMMLinearKernel):
    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_xpu():
            return False, "XPUFP8ScaledMM only support on XPU"
        return True, None

    @classmethod
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        if c.weight_quant_key not in {kFp8StaticChannelSym, kFp8StaticTensorSym}:
            return (
                False,
                "XPUFP8ScaledMM only support per-channel and per-tensor quantization",
            )
        if c.weight_quant_key.dtype not in {torch.float8_e5m2, torch.float8_e4m3fn}:
            return False, "XPUFP8ScaledMM only support FP8 weight dtype"
        return True, None

    def __init__(
        self, c: FP8ScaledMMLinearLayerConfig, layer_param_names: Sequence[str]
    ) -> None:
        assert self.can_implement(c)[0]
        assert self.is_supported()[0]
        self.config = c
        self.layer_param_names = layer_param_names

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # fp8_gemm_w8a16 expects weight in [in, out] layout.
        # Transpose if weight is still in [out, in] layout.
        # For square matrices, use contiguity as tie-breaker:
        # checkpoint weights are contiguous, .t() views are not.
        weight = layer.weight
        out_features, in_features = self.config.weight_shape

        if weight.shape == (out_features, in_features) and (
            in_features != out_features or weight.is_contiguous()
        ):
            replace_parameter(layer, "weight", weight.data.t())
        # else: already in [in, out] layout — no-op

        weight_scale = layer.weight_scale.t().contiguous()
        replace_parameter(layer, "weight_scale", weight_scale.data)

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        weight = layer.weight
        weight_scale = layer.weight_scale
        return torch.ops._xpu_C.fp8_gemm_w8a16(x, weight, weight_scale, bias)

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
        pass
```
**EN:** This kernel class defines `XPUFP8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `FP8ScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `__init__`, `process_weights_after_loading`, `apply_weights`, ....
**CN:** 该内核类定义了 `XPUFP8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `FP8ScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `__init__`, `process_weights_after_loading`, `apply_weights`, ...。

### Method `XPUFP8ScaledMMLinearKernel.is_supported` (lines 22-27)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_xpu():
            return False, "XPUFP8ScaledMM only support on XPU"
        return True, None
```
**EN:** This method implements `XPUFP8ScaledMMLinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_xpu`.
**CN:** 该方法 `XPUFP8ScaledMMLinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_xpu` 等例程。

### Method `XPUFP8ScaledMMLinearKernel.can_implement` (lines 30-38)
```python
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        if c.weight_quant_key not in {kFp8StaticChannelSym, kFp8StaticTensorSym}:
            return (
                False,
                "XPUFP8ScaledMM only support per-channel and per-tensor quantization",
            )
        if c.weight_quant_key.dtype not in {torch.float8_e5m2, torch.float8_e4m3fn}:
            return False, "XPUFP8ScaledMM only support FP8 weight dtype"
        return True, None
```
**EN:** This method implements `XPUFP8ScaledMMLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `XPUFP8ScaledMMLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。

### Method `XPUFP8ScaledMMLinearKernel.__init__` (lines 40-46)
```python
    def __init__(
        self, c: FP8ScaledMMLinearLayerConfig, layer_param_names: Sequence[str]
    ) -> None:
        assert self.can_implement(c)[0]
        assert self.is_supported()[0]
        self.config = c
        self.layer_param_names = layer_param_names
```
**EN:** This method implements `XPUFP8ScaledMMLinearKernel.__init__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `can_implement`, `is_supported`.
**CN:** 该方法 `XPUFP8ScaledMMLinearKernel.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。 它内部会调用 `can_implement`, `is_supported` 等例程。

### Method `XPUFP8ScaledMMLinearKernel.process_weights_after_loading` (lines 48-63)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # fp8_gemm_w8a16 expects weight in [in, out] layout.
        # Transpose if weight is still in [out, in] layout.
        # For square matrices, use contiguity as tie-breaker:
        # checkpoint weights are contiguous, .t() views are not.
        weight = layer.weight
        out_features, in_features = self.config.weight_shape

        if weight.shape == (out_features, in_features) and (
            in_features != out_features or weight.is_contiguous()
        ):
            replace_parameter(layer, "weight", weight.data.t())
        # else: already in [in, out] layout — no-op

        weight_scale = layer.weight_scale.t().contiguous()
        replace_parameter(layer, "weight_scale", weight_scale.data)
```
**EN:** This method implements `XPUFP8ScaledMMLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `contiguous`, `replace_parameter`, `is_contiguous`, `t`.
**CN:** 该方法 `XPUFP8ScaledMMLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `contiguous`, `replace_parameter`, `is_contiguous`, `t` 等例程。

### Method `XPUFP8ScaledMMLinearKernel.apply_weights` (lines 65-73)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        weight = layer.weight
        weight_scale = layer.weight_scale
        return torch.ops._xpu_C.fp8_gemm_w8a16(x, weight, weight_scale, bias)
```
**EN:** This method implements `XPUFP8ScaledMMLinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `fp8_gemm_w8a16`.
**CN:** 该方法 `XPUFP8ScaledMMLinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `fp8_gemm_w8a16` 等例程。

### Method `XPUFP8ScaledMMLinearKernel.apply_scaled_mm` (lines 75-86)
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
        pass
```
**EN:** This method implements `XPUFP8ScaledMMLinearKernel.apply_scaled_mm`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `XPUFP8ScaledMMLinearKernel.apply_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply scaled mm 相关工作。

## Key Concepts / 关键概念
- **XPU support / XPU 支持**
  - **EN:** The file adds Intel XPU-specific dispatch, operators, or compatibility checks.
  - **CN:** 该文件加入 Intel XPU 专用的分派、算子或兼容性检查。
- **FP8 quantization / FP8 量化**
  - **EN:** The module handles FP8 data, scales, or kernels for low-precision inference.
  - **CN:** 该模块处理 FP8 数据、缩放因子或相关低精度推理内核。
- **Scaled matrix multiplication / 缩放矩阵乘**
  - **EN:** The code centers on scaled matrix multiplication kernels and their dispatch rules.
  - **CN:** 代码围绕缩放矩阵乘内核及其分派规则展开。
- **Linear kernel abstraction / 线性内核抽象**
  - **EN:** The module defines abstractions or selectors for quantized linear operators.
  - **CN:** 该模块定义量化线性算子的抽象或选择器。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.utils`, `vllm.platforms`
- **External / 外部依赖**: `collections.abc`, `torch`
