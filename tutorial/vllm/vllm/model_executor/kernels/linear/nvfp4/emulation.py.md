# emulation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/nvfp4/emulation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the emulation NVFP4 linear kernel path. / 实现 仿真 的 NVFP4 线性内核路径。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-11)
```python
import torch

from vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils import (
    kE2M1ToFloat_handle,
    run_nvfp4_emulations,
)

from .base import NvFp4LinearKernel, NvFp4LinearLayerConfig
```
**EN:** This import block loads `torch`, `vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils`, `.base`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `emulation.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils`, `.base`，为 `emulation.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Class `EmulationNvFp4LinearKernel` (lines 14-49)
```python
class EmulationNvFp4LinearKernel(NvFp4LinearKernel):
    """Software emulation fallback for NVFP4 (dequant → BF16 matmul)."""

    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        # Always available as a last-resort fallback.
        return True, None

    @classmethod
    def can_implement(cls, config: NvFp4LinearLayerConfig) -> tuple[bool, str | None]:
        return True, None

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Move the E2M1 lookup table to the device now, because
        # `.to(device)` is not allowed during CUDA graph capture.
        kE2M1ToFloat_handle.val = kE2M1ToFloat_handle.val.to(layer.weight.device)

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        out = run_nvfp4_emulations(
            x=x,
            input_global_scale=layer.input_global_scale_inv,
            weight=layer.weight,
            weight_scale_swizzled=layer.weight_scale,
            weight_global_scale=layer.weight_global_scale,
            swizzle=False,
        )
        if bias is not None:
            out = out + bias
        return out
```
**EN:** This kernel class defines `EmulationNvFp4LinearKernel`. Software emulation fallback for NVFP4 (dequant → BF16 matmul). It inherits from `NvFp4LinearKernel`. Key methods include `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_weights`.
**CN:** 该内核类定义了 `EmulationNvFp4LinearKernel`。 它主要负责与 `EmulationNvFp4LinearKernel` 对应的数据组织、接口约束或执行流程。 它继承自 `NvFp4LinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_weights`。

### Method `EmulationNvFp4LinearKernel.is_supported` (lines 18-22)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        # Always available as a last-resort fallback.
        return True, None
```
**EN:** This method implements `EmulationNvFp4LinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `EmulationNvFp4LinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。

### Method `EmulationNvFp4LinearKernel.can_implement` (lines 25-26)
```python
    def can_implement(cls, config: NvFp4LinearLayerConfig) -> tuple[bool, str | None]:
        return True, None
```
**EN:** This method implements `EmulationNvFp4LinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `EmulationNvFp4LinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。

### Method `EmulationNvFp4LinearKernel.process_weights_after_loading` (lines 28-31)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Move the E2M1 lookup table to the device now, because
        # `.to(device)` is not allowed during CUDA graph capture.
        kE2M1ToFloat_handle.val = kE2M1ToFloat_handle.val.to(layer.weight.device)
```
**EN:** This method implements `EmulationNvFp4LinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `to`.
**CN:** 该方法 `EmulationNvFp4LinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `to` 等例程。

### Method `EmulationNvFp4LinearKernel.apply_weights` (lines 33-49)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        out = run_nvfp4_emulations(
            x=x,
            input_global_scale=layer.input_global_scale_inv,
            weight=layer.weight,
            weight_scale_swizzled=layer.weight_scale,
            weight_global_scale=layer.weight_global_scale,
            swizzle=False,
        )
        if bias is not None:
            out = out + bias
        return out
```
**EN:** This method implements `EmulationNvFp4LinearKernel.apply_weights`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `run_nvfp4_emulations`.
**CN:** 该方法 `EmulationNvFp4LinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `run_nvfp4_emulations` 等例程。

## Key Concepts / 关键概念
- **NVFP4 kernels / NVFP4 内核**
  - **EN:** The file works with NVIDIA FP4-style quantized weights and runtime kernels.
  - **CN:** 该文件处理 NVIDIA FP4 风格量化权重及其运行时内核。
- **Linear kernel abstraction / 线性内核抽象**
  - **EN:** The module defines abstractions or selectors for quantized linear operators.
  - **CN:** 该模块定义量化线性算子的抽象或选择器。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils`, `.base`
- **External / 外部依赖**: `torch`
