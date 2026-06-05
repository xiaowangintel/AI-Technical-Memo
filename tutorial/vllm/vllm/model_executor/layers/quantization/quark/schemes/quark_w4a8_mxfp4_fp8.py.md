# quark_w4a8_mxfp4_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/quark/schemes/quark_w4a8_mxfp4_fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `QuarkW4A8_MXFP4_FP8` for quantization backends, schemes, and utilities. / 实现 `QuarkW4A8_MXFP4_FP8`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-23)
```python
from collections.abc import Callable
from fractions import Fraction
from typing import Any

import torch
import torch.nn.functional as F

from vllm._aiter_ops import is_aiter_found_and_supported, rocm_aiter_ops
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    get_fp8_min_max,
)
from vllm.model_executor.parameter import (
    GroupQuantScaleParameter,
    PackedvLLMParameter,
    PerTensorScaleParameter,
)
from vllm.platforms import current_platform

from .quark_scheme import QuarkScheme
```
**EN:** This opening block pulls in external dependencies such as `collections`, `fractions`, `typing`, `torch` and internal modules such as `vllm._aiter_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`, `vllm.platforms`, `.quark_scheme`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `collections`, `fractions`, `typing`, `torch`）以及内部模块（如 `vllm._aiter_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`, `vllm.platforms`, `.quark_scheme`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 25-30)
```python
logger = init_logger(__name__)


__all__ = ["QuarkW4A8_MXFP4_FP8"]

OCP_MX_BLOCK_SIZE = 32
```
**EN:** This block defines module-level metadata or constants such as `logger`, `__all__`, `OCP_MX_BLOCK_SIZE`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`, `__all__`, `OCP_MX_BLOCK_SIZE`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Class `QuarkW4A8_MXFP4_FP8` overview (lines 33-218)
```python
class QuarkW4A8_MXFP4_FP8(QuarkScheme):
    """
    - Weights: MXFP4 with E8M0 scales per block of 32
    - Activations: FP8 E4M3 (static per-tensor quantization)

    Uses the AITER Triton kernel and falls back to emulation if AITER not available.
    """

    def __init__(
        self,
        weight_quant_spec: dict[str, Any],
        input_quant_spec: dict[str, Any],
    ):
        self.out_dtype = None

        self.weight_dtype = "mxfp4"
        self.packed_factor: Fraction = Fraction(2, 1)  # 2 FP4 values per byte
        self.weight_block_size = OCP_MX_BLOCK_SIZE

        self.is_static_input_scheme = not input_quant_spec.get("is_dynamic")
        self.input_qscheme = input_quant_spec.get("qscheme")  # "per_tensor"

        self.fp8_min, self.fp8_max = get_fp8_min_max()
        self.fp8_dtype = current_platform.fp8_dtype()
```
**EN:** Defines class `QuarkW4A8_MXFP4_FP8` with base classes `QuarkScheme` and decorators none. It acts as a quantization-oriented module building block and exposes 8 direct methods, with notable entries `__init__`, `get_min_capability`, `get_packed_dim`, `create_weights`, `process_weights_after_loading`, `apply_weights`. Its docstring says: - Weights: MXFP4 with E8M0 scales per block of 32 - Activations: FP8 E4M3 (static per-tensor quantization) Uses the AITER Triton kernel and falls back to emulation if AITER not available.
**CN:** 定义类 `QuarkW4A8_MXFP4_FP8`，其基类为 `QuarkScheme`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 8 个方法，较重要的包括 `__init__`, `get_min_capability`, `get_packed_dim`, `create_weights`, `process_weights_after_loading`, `apply_weights`。 文档字符串进一步说明了该类的定位。

### Method `QuarkW4A8_MXFP4_FP8.__init__` (lines 41-80)
```python
    def __init__(
        self,
        weight_quant_spec: dict[str, Any],
        input_quant_spec: dict[str, Any],
    ):
        self.out_dtype = None

        self.weight_dtype = "mxfp4"
        self.packed_factor: Fraction = Fraction(2, 1)  # 2 FP4 values per byte
        self.weight_block_size = OCP_MX_BLOCK_SIZE

        self.is_static_input_scheme = not input_quant_spec.get("is_dynamic")
        self.input_qscheme = input_quant_spec.get("qscheme")  # "per_tensor"

        self.fp8_min, self.fp8_max = get_fp8_min_max()
        self.fp8_dtype = current_platform.fp8_dtype()

        if not self.is_static_input_scheme:
            raise NotImplementedError(
                "Dynamic FP8 activation quantization is not yet supported "
                "for W4A8. The current implementation expects static per-tensor "
                "FP8 scales stored in the checkpoint."
            )

        kernel_supported_gpu = False
        if current_platform.is_rocm():
            from vllm.platforms.rocm import on_gfx950

            kernel_supported_gpu = on_gfx950()

        self.use_aiter_kernel = (
            is_aiter_found_and_supported()
            and self.is_static_input_scheme
            and kernel_supported_gpu
        )

        if not self.use_aiter_kernel:
            logger.warning_once(
                "[W4A8 MXFP4+FP8] Aiter Triton kernel not found. Using emulation mode."
            )
```
**EN:** Defines function `QuarkW4A8_MXFP4_FP8.__init__` with signature `__init__(self, weight_quant_spec: dict[str, Any], input_quant_spec: dict[str, Any])`. It mainly works with `weight_quant_spec`, `input_quant_spec`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `Fraction`, `input_quant_spec.get`, `get_fp8_min_max`, `current_platform.fp8_dtype`, `current_platform.is_rocm`, `NotImplementedError`.
**CN:** 定义函数 `QuarkW4A8_MXFP4_FP8.__init__`，其签名为 `__init__(self, weight_quant_spec: dict[str, Any], input_quant_spec: dict[str, Any])`。它主要围绕 `weight_quant_spec`, `input_quant_spec` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `Fraction`, `input_quant_spec.get`, `get_fp8_min_max`, `current_platform.fp8_dtype`, `current_platform.is_rocm`, `NotImplementedError`。

### Method `QuarkW4A8_MXFP4_FP8.get_min_capability` (lines 83-84)
```python
    def get_min_capability(cls) -> int:
        return 70
```
**EN:** Defines function `QuarkW4A8_MXFP4_FP8.get_min_capability` with signature `get_min_capability(cls) -> int`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `QuarkW4A8_MXFP4_FP8.get_min_capability`，其签名为 `get_min_capability(cls) -> int`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `QuarkW4A8_MXFP4_FP8.get_packed_dim` (lines 86-88)
```python
    def get_packed_dim(self, dim: int) -> int:
        assert dim % 2 == 0, f"Dimension {dim} must be even for MXFP4 packing"
        return dim // 2
```
**EN:** Defines function `QuarkW4A8_MXFP4_FP8.get_packed_dim` with signature `get_packed_dim(self, dim: int) -> int`. It mainly works with `dim`; returns a derived property or capability check. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `QuarkW4A8_MXFP4_FP8.get_packed_dim`，其签名为 `get_packed_dim(self, dim: int) -> int`。它主要围绕 `dim` 展开；返回派生属性或能力判断结果。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Method `QuarkW4A8_MXFP4_FP8.create_weights` (lines 90-143)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        output_partition_sizes: list[int],
        input_size_per_partition: int,
        params_dtype: torch.dtype,
        weight_loader: Callable,
        **kwargs,
    ):
        output_size_per_partition = sum(output_partition_sizes)
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition

        # MXFP4 WEIGHT (packed, 2 values per byte)
        weight = PackedvLLMParameter(
            data=torch.empty(
                output_size_per_partition,
                self.get_packed_dim(input_size_per_partition),
                dtype=torch.uint8,
            ),
            input_dim=1,
            output_dim=0,
            packed_dim=1,
            packed_factor=self.packed_factor,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        # WEIGHT SCALE (E8M0 format, per block of 32)
        weight_scale = GroupQuantScaleParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition // self.weight_block_size,
                dtype=torch.uint8,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight_scale", weight_scale)

        # INPUT SCALE (FP8 per-tensor static scale)
        if self.is_static_input_scheme:
            input_scale = PerTensorScaleParameter(
                data=torch.empty(
                    len(output_partition_sizes),
                    dtype=torch.float32,
                ),
                weight_loader=weight_loader,
            )
            # Initialize to avoid NaN
            input_scale[:] = torch.finfo(torch.float32).min
            layer.register_parameter("input_scale", input_scale)
```
**EN:** Defines function `QuarkW4A8_MXFP4_FP8.create_weights` with signature `create_weights(self, layer: torch.nn.Module, output_partition_sizes: list[int], input_size_per_partition: int, params_dtype: torch.dtype, weight_loader: Callable, **kwargs)`. It mainly works with `layer`, `output_partition_sizes`, `input_size_per_partition`, `params_dtype`, `weight_loader`, `**kwargs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `sum`, `PackedvLLMParameter`, `layer.register_parameter`, `GroupQuantScaleParameter`, `PerTensorScaleParameter`, `torch.empty`.
**CN:** 定义函数 `QuarkW4A8_MXFP4_FP8.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, output_partition_sizes: list[int], input_size_per_partition: int, params_dtype: torch.dtype, weight_loader: Callable, **kwargs)`。它主要围绕 `layer`, `output_partition_sizes`, `input_size_per_partition`, `params_dtype`, `weight_loader`, `**kwargs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `sum`, `PackedvLLMParameter`, `layer.register_parameter`, `GroupQuantScaleParameter`, `PerTensorScaleParameter`, `torch.empty`。

### Method `QuarkW4A8_MXFP4_FP8.process_weights_after_loading` (lines 145-161)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Ensuring weights & scales are non-trainable
        layer.weight = torch.nn.Parameter(layer.weight.data, requires_grad=False)
        layer.weight_scale = torch.nn.Parameter(
            layer.weight_scale.data, requires_grad=False
        )

        if self.is_static_input_scheme:
            input_scale = layer.input_scale.data
            # For fused modules (QKV), take the max scale
            if input_scale.numel() != 1:
                input_scale = input_scale.max()

            layer.input_scale = torch.nn.Parameter(
                torch.tensor(input_scale, dtype=torch.float32),
                requires_grad=False,
            )
```
**EN:** Defines function `QuarkW4A8_MXFP4_FP8.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, tensor/kernel operations. Key calls include `torch.nn.Parameter`, `input_scale.numel`, `input_scale.max`, `torch.tensor`.
**CN:** 定义函数 `QuarkW4A8_MXFP4_FP8.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.nn.Parameter`, `input_scale.numel`, `input_scale.max`, `torch.tensor`。

### Method `QuarkW4A8_MXFP4_FP8.apply_weights` (lines 163-172)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if self.use_aiter_kernel:
            return self._apply_aiter_kernel(layer, x, bias)
        else:
            return self._apply_emulation(layer, x, bias)
```
**EN:** Defines function `QuarkW4A8_MXFP4_FP8.apply_weights` with signature `apply_weights(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `self._apply_aiter_kernel`, `self._apply_emulation`.
**CN:** 定义函数 `QuarkW4A8_MXFP4_FP8.apply_weights`，其签名为 `apply_weights(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `self._apply_aiter_kernel`, `self._apply_emulation`。

### Method `QuarkW4A8_MXFP4_FP8._apply_aiter_kernel` (lines 174-196)
```python
    def _apply_aiter_kernel(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        M = x.shape[0]
        out_dtype = x.dtype if self.out_dtype is None else self.out_dtype

        input_scale = layer.input_scale
        x_fp8 = (x / input_scale).clamp(self.fp8_min, self.fp8_max).to(self.fp8_dtype)

        # Broadcast per-tensor scale to per-row (M, 1) for Aiter kernel
        x_scales = input_scale.expand(M, 1).to(dtype=torch.float32, device=x.device)

        y = rocm_aiter_ops.gemm_a8wfp4(
            x_fp8, layer.weight, x_scales, layer.weight_scale, out_dtype
        )

        if bias is not None:
            y = y + bias

        return y
```
**EN:** Defines function `QuarkW4A8_MXFP4_FP8._apply_aiter_kernel` with signature `_apply_aiter_kernel(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `clamp.to`, `input_scale.expand.to`, `rocm_aiter_ops.gemm_a8wfp4`, `clamp`, `input_scale.expand`.
**CN:** 定义函数 `QuarkW4A8_MXFP4_FP8._apply_aiter_kernel`，其签名为 `_apply_aiter_kernel(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `clamp.to`, `input_scale.expand.to`, `rocm_aiter_ops.gemm_a8wfp4`, `clamp`, `input_scale.expand`。

### Method `QuarkW4A8_MXFP4_FP8._apply_emulation` (lines 198-218)
```python
    def _apply_emulation(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        from vllm.model_executor.layers.quantization.utils.mxfp4_utils import (
            dequant_mxfp4,
        )

        weight_dq = dequant_mxfp4(
            layer.weight,
            layer.weight_scale,
            x.dtype,
        )

        input_scale = layer.input_scale
        x_fp8 = (x / input_scale).clamp(self.fp8_min, self.fp8_max).to(self.fp8_dtype)
        x_dq = (x_fp8.to(x.dtype) * input_scale).to(x.dtype)

        return F.linear(x_dq, weight_dq, bias)
```
**EN:** Defines function `QuarkW4A8_MXFP4_FP8._apply_emulation` with signature `_apply_emulation(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `dequant_mxfp4`, `clamp.to`, `to`, `F.linear`, `clamp`, `x_fp8.to`.
**CN:** 定义函数 `QuarkW4A8_MXFP4_FP8._apply_emulation`，其签名为 `_apply_emulation(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `dequant_mxfp4`, `clamp.to`, `to`, `F.linear`, `clamp`, `x_fp8.to`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `QuarkW4A8_MXFP4_FP8`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `QuarkW4A8_MXFP4_FP8`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `collections`, `fractions`, `typing`, `torch`
- **Internal / 内部**: `vllm._aiter_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`, `vllm.platforms`, `.quark_scheme`
