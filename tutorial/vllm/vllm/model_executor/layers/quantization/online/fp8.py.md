# fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/online/fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_Fp8OnlineLinearBase`, `Fp8PerTensorOnlineLinearMethod`, `Fp8PerBlockOnlineLinearMethod` for quantization backends, schemes, and utilities. / 实现 `_Fp8OnlineLinearBase`, `Fp8PerTensorOnlineLinearMethod`, `Fp8PerBlockOnlineLinearMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-51)
```python
from typing import TYPE_CHECKING

import torch
from torch.nn import Module

if TYPE_CHECKING:
    import vllm.model_executor.layers.fused_moe.modular_kernel as mk
    from vllm.model_executor.layers.fused_moe.config import (
        FusedMoEQuantConfig,
    )
    from vllm.model_executor.layers.fused_moe.oracle.fp8 import Fp8MoeBackend

import vllm.envs as envs
from vllm import _custom_ops as ops
from vllm.config import get_current_vllm_config
from vllm.model_executor.kernels.linear import init_fp8_linear_kernel
from vllm.model_executor.kernels.linear.scaled_mm import (
    CutlassFP8ScaledMMLinearKernel,
)
from vllm.model_executor.layers.fused_moe import RoutedExperts
from vllm.model_executor.layers.fused_moe.oracle.fp8 import (
    select_fp8_moe_backend,
)
from vllm.model_executor.layers.linear import (
    LinearMethodBase,
)
from vllm.model_executor.layers.quantization.online.moe_base import (
    OnlineMoEMethodBase,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    create_fp8_quant_key,
    kFp8Dynamic128Sym,
    kFp8DynamicTensorSym,
    kFp8DynamicTokenSym,
    kFp8Static128BlockSym,
    kFp8StaticTensorSym,
)
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    cutlass_fp8_supported,
)
from vllm.model_executor.model_loader.reload.layerwise import (
    initialize_online_processing,
)
from vllm.model_executor.parameter import ModelWeightParameter
from vllm.model_executor.utils import replace_parameter
from vllm.platforms import current_platform
from vllm.utils.deep_gemm import per_block_cast_to_fp8
```
**EN:** This opening block pulls in external dependencies such as `typing`, `torch` and internal modules such as `vllm.envs`, `vllm`, `vllm.config`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.kernels.linear.scaled_mm`, `vllm.model_executor.layers.fused_moe`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `torch`）以及内部模块（如 `vllm.envs`, `vllm`, `vllm.config`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.kernels.linear.scaled_mm`, `vllm.model_executor.layers.fused_moe`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `_Fp8OnlineLinearBase` overview (lines 58-99)
```python
class _Fp8OnlineLinearBase(LinearMethodBase):
    """Shared base for online FP8 linear methods. Loads fp16/bf16 checkpoint
    weights onto meta device and materializes them just-in-time."""

    uses_meta_device: bool = True

    def __init__(self):
        self.out_dtype = torch.get_default_dtype()
        self.input_dtype = get_current_vllm_config().model_config.dtype

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
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
```
**EN:** Defines class `_Fp8OnlineLinearBase` with base classes `LinearMethodBase` and decorators none. It acts as a quantization-oriented module building block and exposes 2 direct methods, with notable entries `__init__`, `create_weights`. Its docstring says: Shared base for online FP8 linear methods.
**CN:** 定义类 `_Fp8OnlineLinearBase`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 2 个方法，较重要的包括 `__init__`, `create_weights`。 文档字符串进一步说明了该类的定位。

### Method `_Fp8OnlineLinearBase.__init__` (lines 64-66)
```python
    def __init__(self):
        self.out_dtype = torch.get_default_dtype()
        self.input_dtype = get_current_vllm_config().model_config.dtype
```
**EN:** Defines function `_Fp8OnlineLinearBase.__init__` with signature `__init__(self)`. It mainly works with object context only; initializes the object state and cached resources. The body uses tensor/kernel operations. Key calls include `torch.get_default_dtype`, `get_current_vllm_config`.
**CN:** 定义函数 `_Fp8OnlineLinearBase.__init__`，其签名为 `__init__(self)`。它主要围绕 仅依赖对象上下文 展开；负责初始化对象状态和缓存资源。函数体包含张量或内核操作。关键调用包括 `torch.get_default_dtype`, `get_current_vllm_config`。

### Method `_Fp8OnlineLinearBase.create_weights` (lines 68-99)
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
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.orig_dtype = params_dtype
        layer.weight_block_size = None

        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition,
                device="meta",  # materialized and processed during loading
                dtype=params_dtype,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        initialize_online_processing(layer)
```
**EN:** Defines function `_Fp8OnlineLinearBase.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `initialize_online_processing`, `torch.empty`.
**CN:** 定义函数 `_Fp8OnlineLinearBase.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `initialize_online_processing`, `torch.empty`。

### Class `Fp8PerTensorOnlineLinearMethod` overview (lines 102-188)
```python
class Fp8PerTensorOnlineLinearMethod(_Fp8OnlineLinearBase):
    """Online tensorwise FP8 linear quantization.
    Loads fp16/bf16 weights and quantizes them per-tensor during loading."""

    def __init__(self):
        super().__init__()

        self.weight_quant_key = kFp8StaticTensorSym
        # Use per-token quantization for better perf if dynamic and cutlass
        if cutlass_fp8_supported():
            self.activation_quant_key = kFp8DynamicTokenSym
        else:
            self.activation_quant_key = kFp8DynamicTensorSym

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
        super().create_weights(
```
**EN:** Defines class `Fp8PerTensorOnlineLinearMethod` with base classes `_Fp8OnlineLinearBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Online tensorwise FP8 linear quantization.
**CN:** 定义类 `Fp8PerTensorOnlineLinearMethod`，其基类为 `_Fp8OnlineLinearBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `Fp8PerTensorOnlineLinearMethod.__init__` (lines 106-114)
```python
    def __init__(self):
        super().__init__()

        self.weight_quant_key = kFp8StaticTensorSym
        # Use per-token quantization for better perf if dynamic and cutlass
        if cutlass_fp8_supported():
            self.activation_quant_key = kFp8DynamicTokenSym
        else:
            self.activation_quant_key = kFp8DynamicTensorSym
```
**EN:** Defines function `Fp8PerTensorOnlineLinearMethod.__init__` with signature `__init__(self)`. It mainly works with object context only; initializes the object state and cached resources. The body uses branching. Key calls include `super.__init__`, `cutlass_fp8_supported`, `super`.
**CN:** 定义函数 `Fp8PerTensorOnlineLinearMethod.__init__`，其签名为 `__init__(self)`。它主要围绕 仅依赖对象上下文 展开；负责初始化对象状态和缓存资源。函数体包含分支判断。关键调用包括 `super.__init__`, `cutlass_fp8_supported`, `super`。

### Method `Fp8PerTensorOnlineLinearMethod.create_weights` (lines 116-143)
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
        super().create_weights(
            layer,
            input_size_per_partition,
            output_partition_sizes,
            input_size,
            output_size,
            params_dtype,
            **extra_weight_attrs,
        )

        self.fp8_linear = init_fp8_linear_kernel(
            activation_quant_key=self.activation_quant_key,
            weight_quant_key=self.weight_quant_key,
            weight_shape=layer.weight.shape,
            input_dtype=self.input_dtype,
            out_dtype=self.out_dtype,
            module_name=self.__class__.__name__,
        )
```
**EN:** Defines function `Fp8PerTensorOnlineLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses mostly straightforward data movement and object wiring. Key calls include `super.create_weights`, `init_fp8_linear_kernel`, `super`.
**CN:** 定义函数 `Fp8PerTensorOnlineLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.create_weights`, `init_fp8_linear_kernel`, `super`。

### Method `Fp8PerTensorOnlineLinearMethod.process_weights_after_loading` (lines 145-157)
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        layer.input_scale = None
        qweight, weight_scale = ops.scaled_fp8_quant(layer.weight, scale=None)

        # Update layer with new values.
        replace_parameter(layer, "weight", qweight.t().data)
        replace_parameter(layer, "weight_scale", weight_scale.data)

        # Prevent duplicate processing (e.g., during weight reload)
        layer._already_called_process_weights_after_loading = True
```
**EN:** Defines function `Fp8PerTensorOnlineLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, tensor/kernel operations. Key calls include `getattr`, `ops.scaled_fp8_quant`, `replace_parameter`, `qweight.t`.
**CN:** 定义函数 `Fp8PerTensorOnlineLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、张量或内核操作。关键调用包括 `getattr`, `ops.scaled_fp8_quant`, `replace_parameter`, `qweight.t`。

### Method `Fp8PerTensorOnlineLinearMethod.apply` (lines 159-188)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # if batch invariant mode is enabled, use BF16 dequant
        if envs.VLLM_BATCH_INVARIANT:
            if isinstance(self.fp8_linear, CutlassFP8ScaledMMLinearKernel):
                return self.fp8_linear.apply_weights(layer, x, bias)

            weight_fp8 = layer.weight.to(torch.bfloat16)
            weight_scale = layer.weight_scale.to(torch.bfloat16)
            if weight_scale.numel() == 1:
                # Per-tensor: simple scalar multiplication
                weight_bf16 = weight_fp8 * weight_scale
            else:
                # Multiple scales (fused modules like QKV)
                if (
                    weight_scale.dim() == 1
                    and weight_scale.shape[0] == weight_fp8.shape[0]
                ):
                    # Per-row scaling
                    weight_bf16 = weight_fp8 * weight_scale.unsqueeze(1)
                else:
                    # Fallback
                    weight_bf16 = weight_fp8 * weight_scale
            return torch.nn.functional.linear(x, weight_bf16.t(), bias)

        return self.fp8_linear.apply_weights(layer, x, bias)
```
**EN:** Defines function `Fp8PerTensorOnlineLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching, tensor/kernel operations. Key calls include `self.fp8_linear.apply_weights`, `isinstance`, `layer.weight.to`, `layer.weight_scale.to`, `torch.nn.functional.linear`, `weight_scale.numel`.
**CN:** 定义函数 `Fp8PerTensorOnlineLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `self.fp8_linear.apply_weights`, `isinstance`, `layer.weight.to`, `layer.weight_scale.to`, `torch.nn.functional.linear`, `weight_scale.numel`。

### Class `Fp8PerBlockOnlineLinearMethod` overview (lines 191-268)
```python
class Fp8PerBlockOnlineLinearMethod(_Fp8OnlineLinearBase):
    """Online blockwise FP8 linear quantization.
    Loads fp16/bf16 weights and quantizes them per-block during loading."""

    def __init__(self):
        super().__init__()
        self.weight_block_size = [128, 128]
        self.activation_quant_key = create_fp8_quant_key(
            static=False,
            group_shape=GroupShape(1, self.weight_block_size[0]),
        )
        self.weight_quant_key = create_fp8_quant_key(
            static=True, group_shape=GroupShape(*self.weight_block_size)
        )

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
```
**EN:** Defines class `Fp8PerBlockOnlineLinearMethod` with base classes `_Fp8OnlineLinearBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Online blockwise FP8 linear quantization.
**CN:** 定义类 `Fp8PerBlockOnlineLinearMethod`，其基类为 `_Fp8OnlineLinearBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `Fp8PerBlockOnlineLinearMethod.__init__` (lines 195-204)
```python
    def __init__(self):
        super().__init__()
        self.weight_block_size = [128, 128]
        self.activation_quant_key = create_fp8_quant_key(
            static=False,
            group_shape=GroupShape(1, self.weight_block_size[0]),
        )
        self.weight_quant_key = create_fp8_quant_key(
            static=True, group_shape=GroupShape(*self.weight_block_size)
        )
```
**EN:** Defines function `Fp8PerBlockOnlineLinearMethod.__init__` with signature `__init__(self)`. It mainly works with object context only; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `create_fp8_quant_key`, `super`, `GroupShape`.
**CN:** 定义函数 `Fp8PerBlockOnlineLinearMethod.__init__`，其签名为 `__init__(self)`。它主要围绕 仅依赖对象上下文 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `create_fp8_quant_key`, `super`, `GroupShape`。

### Method `Fp8PerBlockOnlineLinearMethod.create_weights` (lines 206-234)
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
        super().create_weights(
            layer,
            input_size_per_partition,
            output_partition_sizes,
            input_size,
            output_size,
            params_dtype,
            **extra_weight_attrs,
        )
        layer.weight_block_size = self.weight_block_size

        self.fp8_linear = init_fp8_linear_kernel(
            activation_quant_key=self.activation_quant_key,
            weight_quant_key=self.weight_quant_key,
            weight_shape=layer.weight.shape,
            input_dtype=self.input_dtype,
            out_dtype=self.out_dtype,
            module_name=self.__class__.__name__,
        )
```
**EN:** Defines function `Fp8PerBlockOnlineLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses mostly straightforward data movement and object wiring. Key calls include `super.create_weights`, `init_fp8_linear_kernel`, `super`.
**CN:** 定义函数 `Fp8PerBlockOnlineLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.create_weights`, `init_fp8_linear_kernel`, `super`。

### Method `Fp8PerBlockOnlineLinearMethod.process_weights_after_loading` (lines 236-253)
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        layer.input_scale = None
        block_size = self.weight_block_size

        qweight, weight_scale_inv = per_block_cast_to_fp8(
            layer.weight, block_size=block_size, use_ue8m0=False
        )

        replace_parameter(layer, "weight", qweight.data)
        replace_parameter(layer, "weight_scale_inv", weight_scale_inv.data)

        self.fp8_linear.process_weights_after_loading(layer)

        # Prevent duplicate processing (e.g., during weight reload)
        layer._already_called_process_weights_after_loading = True
```
**EN:** Defines function `Fp8PerBlockOnlineLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching. Key calls include `getattr`, `per_block_cast_to_fp8`, `replace_parameter`, `self.fp8_linear.process_weights_after_loading`.
**CN:** 定义函数 `Fp8PerBlockOnlineLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断。关键调用包括 `getattr`, `per_block_cast_to_fp8`, `replace_parameter`, `self.fp8_linear.process_weights_after_loading`。

### Method `Fp8PerBlockOnlineLinearMethod.apply` (lines 255-268)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        assert self.weight_block_size is not None

        # Note: batch invariance already handled in the function below
        return self.fp8_linear.apply_weights(
            layer,
            x,
            bias,
        )
```
**EN:** Defines function `Fp8PerBlockOnlineLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.fp8_linear.apply_weights`.
**CN:** 定义函数 `Fp8PerBlockOnlineLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.fp8_linear.apply_weights`。

### Class `_Fp8OnlineMoEBase` overview (lines 276-383)
```python
class _Fp8OnlineMoEBase(OnlineMoEMethodBase):
    """Shared base for online FP8 MoE methods. Loads fp16/bf16 checkpoint
    weights onto meta device and materializes them just-in-time."""

    # Declared here for mypy; actual values are set in __init__.
    fp8_backend: "Fp8MoeBackend"
    experts_cls: "type[mk.FusedMoEExperts] | None"
    weight_scale_name: str
    weight_block_size: list[int] | None

    def __init__(
        self,
        *,
        weight_block_size: list[int] | None,
        layer: torch.nn.Module,
    ):
        super().__init__(layer.moe_config)
        self.weight_block_size = weight_block_size
        self.block_quant: bool = self.weight_block_size is not None
        self.weight_scale_name = (
            "weight_scale_inv" if self.block_quant else "weight_scale"
        )

        # Set weight key and activation key for kernel compatibility
        if self.block_quant:
```
**EN:** Defines class `_Fp8OnlineMoEBase` with base classes `OnlineMoEMethodBase` and decorators none. It acts as a quantization-oriented module building block and exposes 3 direct methods, with notable entries `__init__`, `_setup_kernel`, `get_fused_moe_quant_config`. Its docstring says: Shared base for online FP8 MoE methods.
**CN:** 定义类 `_Fp8OnlineMoEBase`，其基类为 `OnlineMoEMethodBase`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 3 个方法，较重要的包括 `__init__`, `_setup_kernel`, `get_fused_moe_quant_config`。 文档字符串进一步说明了该类的定位。

### Method `_Fp8OnlineMoEBase.__init__` (lines 286-313)
```python
    def __init__(
        self,
        *,
        weight_block_size: list[int] | None,
        layer: torch.nn.Module,
    ):
        super().__init__(layer.moe_config)
        self.weight_block_size = weight_block_size
        self.block_quant: bool = self.weight_block_size is not None
        self.weight_scale_name = (
            "weight_scale_inv" if self.block_quant else "weight_scale"
        )

        # Set weight key and activation key for kernel compatibility
        if self.block_quant:
            weight_key = kFp8Static128BlockSym
            activation_key = kFp8Dynamic128Sym
        else:
            weight_key = kFp8StaticTensorSym
            activation_key = kFp8DynamicTensorSym

        # Select Fp8 MoE backend
        self.fp8_backend, self.experts_cls = select_fp8_moe_backend(
            config=self.moe,
            weight_key=weight_key,
            activation_key=activation_key,
            allow_vllm_cutlass=False,
        )
```
**EN:** Defines function `_Fp8OnlineMoEBase.__init__` with signature `__init__(self, *, weight_block_size: list[int] | None, layer: torch.nn.Module)`. It mainly works with `weight_block_size`, `layer`; initializes the object state and cached resources. The body uses branching. Key calls include `super.__init__`, `select_fp8_moe_backend`, `super`.
**CN:** 定义函数 `_Fp8OnlineMoEBase.__init__`，其签名为 `__init__(self, *, weight_block_size: list[int] | None, layer: torch.nn.Module)`。它主要围绕 `weight_block_size`, `layer` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断。关键调用包括 `super.__init__`, `select_fp8_moe_backend`, `super`。

### Method `_Fp8OnlineMoEBase._setup_kernel` (lines 315-358)
```python
    def _setup_kernel(
        self,
        layer: RoutedExperts,
        w13: torch.Tensor,
        w2: torch.Tensor,
        w13_scale: torch.Tensor,
        w2_scale: torch.Tensor,
        w13_input_scale: torch.Tensor | None,
        w2_input_scale: torch.Tensor | None,
    ) -> None:
        from vllm.model_executor.layers.fused_moe.oracle.fp8 import (
            convert_to_fp8_moe_kernel_format,
            make_fp8_moe_kernel,
        )

        # Shuffle weights to runtime format.
        w13, w2, w13_scale, w2_scale = convert_to_fp8_moe_kernel_format(
            fp8_backend=self.fp8_backend,
            layer=layer,
            w13=w13,
            w2=w2,
            w13_scale=w13_scale,
            w2_scale=w2_scale,
            w13_input_scale=w13_input_scale,
            w2_input_scale=w2_input_scale,
        )

        # Replace parameters with updated versions. Note that this helper
        # function ensures the replacement is compatible with RL weight reloads.
        replace_parameter(layer, "w13_weight", w13)
        replace_parameter(layer, "w2_weight", w2)
        replace_parameter(layer, f"w13_{self.weight_scale_name}", w13_scale)
        replace_parameter(layer, f"w2_{self.weight_scale_name}", w2_scale)

        self.moe_quant_config = self.get_fused_moe_quant_config(layer)
        if self.moe_quant_config:
            assert self.experts_cls is not None
            self.moe_kernel = make_fp8_moe_kernel(
                moe_quant_config=self.moe_quant_config,
                moe_config=self.moe,
                fp8_backend=self.fp8_backend,
                experts_cls=self.experts_cls,
                routing_tables=layer._expert_routing_tables(),
            )
```
**EN:** Defines function `_Fp8OnlineMoEBase._setup_kernel` with signature `_setup_kernel(self, layer: RoutedExperts, w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor, w13_input_scale: torch.Tensor | None, w2_input_scale: torch.Tensor | None) -> None`. It mainly works with `layer`, `w13`, `w2`, `w13_scale`, `w2_scale`, `w13_input_scale`, `w2_input_scale`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `convert_to_fp8_moe_kernel_format`, `replace_parameter`, `self.get_fused_moe_quant_config`, `make_fp8_moe_kernel`, `layer._expert_routing_tables`.
**CN:** 定义函数 `_Fp8OnlineMoEBase._setup_kernel`，其签名为 `_setup_kernel(self, layer: RoutedExperts, w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor, w13_input_scale: torch.Tensor | None, w2_input_scale: torch.Tensor | None) -> None`。它主要围绕 `layer`, `w13`, `w2`, `w13_scale`, `w2_scale`, `w13_input_scale`, `w2_input_scale` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `convert_to_fp8_moe_kernel_format`, `replace_parameter`, `self.get_fused_moe_quant_config`, `make_fp8_moe_kernel`, `layer._expert_routing_tables`。

### Method `_Fp8OnlineMoEBase.get_fused_moe_quant_config` (lines 360-383)
```python
    def get_fused_moe_quant_config(
        self, layer: torch.nn.Module
    ) -> "FusedMoEQuantConfig":
        from vllm.model_executor.layers.fused_moe.oracle.fp8 import (
            make_fp8_moe_quant_config,
        )

        w1_scale = getattr(layer, f"w13_{self.weight_scale_name}")
        w2_scale = getattr(layer, f"w2_{self.weight_scale_name}")
        a1_scale = layer.w13_input_scale
        a2_scale = layer.w2_input_scale

        quant_config = make_fp8_moe_quant_config(
            fp8_backend=self.fp8_backend,
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            a1_scale=a1_scale,
            a2_scale=a2_scale,
            block_shape=self.weight_block_size,
            swiglu_limit=getattr(layer, "swiglu_limit", None),
        )

        self._maybe_inject_biases(quant_config, layer)
        return quant_config
```
**EN:** Defines function `_Fp8OnlineMoEBase.get_fused_moe_quant_config` with signature `get_fused_moe_quant_config(self, layer: torch.nn.Module) -> 'FusedMoEQuantConfig'`. It mainly works with `layer`; handles quantization-related transformation logic. The body uses mostly straightforward data movement and object wiring. Key calls include `getattr`, `make_fp8_moe_quant_config`, `self._maybe_inject_biases`.
**CN:** 定义函数 `_Fp8OnlineMoEBase.get_fused_moe_quant_config`，其签名为 `get_fused_moe_quant_config(self, layer: torch.nn.Module) -> 'FusedMoEQuantConfig'`。它主要围绕 `layer` 展开；处理量化相关的变换逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `getattr`, `make_fp8_moe_quant_config`, `self._maybe_inject_biases`。

### Class `Fp8PerTensorOnlineMoEMethod` overview (lines 386-436)
```python
class Fp8PerTensorOnlineMoEMethod(_Fp8OnlineMoEBase):
    """Online tensorwise FP8 MoE quantization.
    Loads fp16/bf16 weights and quantizes them per-tensor during loading."""

    def __init__(
        self,
        *,
        layer: torch.nn.Module,
    ):
        super().__init__(
            weight_block_size=None,
            layer=layer,
        )

    def process_weights_after_loading(self, layer: Module) -> None:
        # TODO(@ksayers): inplace fp8 quant kernel, initialize scales with ones
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        # If checkpoint is fp16, quantize in place.
        fp8_dtype = current_platform.fp8_dtype()
        w13 = torch.empty_like(layer.w13_weight, dtype=fp8_dtype)
        w2 = torch.empty_like(layer.w2_weight, dtype=fp8_dtype)
        w13_scale = torch.ones(
            layer.num_experts, device=w13.device, dtype=torch.float32
```
**EN:** Defines class `Fp8PerTensorOnlineMoEMethod` with base classes `_Fp8OnlineMoEBase` and decorators none. It acts as a backend or execution-method adapter and exposes 2 direct methods, with notable entries `__init__`, `process_weights_after_loading`. Its docstring says: Online tensorwise FP8 MoE quantization.
**CN:** 定义类 `Fp8PerTensorOnlineMoEMethod`，其基类为 `_Fp8OnlineMoEBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 2 个方法，较重要的包括 `__init__`, `process_weights_after_loading`。 文档字符串进一步说明了该类的定位。

### Method `Fp8PerTensorOnlineMoEMethod.__init__` (lines 390-398)
```python
    def __init__(
        self,
        *,
        layer: torch.nn.Module,
    ):
        super().__init__(
            weight_block_size=None,
            layer=layer,
        )
```
**EN:** Defines function `Fp8PerTensorOnlineMoEMethod.__init__` with signature `__init__(self, *, layer: torch.nn.Module)`. It mainly works with `layer`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `Fp8PerTensorOnlineMoEMethod.__init__`，其签名为 `__init__(self, *, layer: torch.nn.Module)`。它主要围绕 `layer` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `Fp8PerTensorOnlineMoEMethod.process_weights_after_loading` (lines 400-436)
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        # TODO(@ksayers): inplace fp8 quant kernel, initialize scales with ones
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        # If checkpoint is fp16, quantize in place.
        fp8_dtype = current_platform.fp8_dtype()
        w13 = torch.empty_like(layer.w13_weight, dtype=fp8_dtype)
        w2 = torch.empty_like(layer.w2_weight, dtype=fp8_dtype)
        w13_scale = torch.ones(
            layer.num_experts, device=w13.device, dtype=torch.float32
        )
        w2_scale = torch.ones(layer.num_experts, device=w2.device, dtype=torch.float32)
        layer.w13_input_scale = None
        layer.w2_input_scale = None

        for expert in range(layer.local_num_experts):
            w13[expert, :, :], w13_scale[expert] = ops.scaled_fp8_quant(
                layer.w13_weight[expert, :, :]
            )
            w2[expert, :, :], w2_scale[expert] = ops.scaled_fp8_quant(
                layer.w2_weight[expert, :, :]
            )

        # Shuffle weights to runtime format and setup kernel.
        self._setup_kernel(
            layer,
            w13,
            w2,
            w13_scale,
            w2_scale,
            w13_input_scale=layer.w13_input_scale,
            w2_input_scale=layer.w2_input_scale,
        )

        # Prevent duplicate processing (e.g., during weight reload)
        layer._already_called_process_weights_after_loading = True
```
**EN:** Defines function `Fp8PerTensorOnlineMoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, iteration, tensor/kernel operations. Key calls include `getattr`, `current_platform.fp8_dtype`, `torch.empty_like`, `torch.ones`, `range`, `self._setup_kernel`.
**CN:** 定义函数 `Fp8PerTensorOnlineMoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `getattr`, `current_platform.fp8_dtype`, `torch.empty_like`, `torch.ones`, `range`, `self._setup_kernel`。

### Class `Fp8PerBlockOnlineMoEMethod` overview (lines 439-512)
```python
class Fp8PerBlockOnlineMoEMethod(_Fp8OnlineMoEBase):
    """Online blockwise FP8 MoE quantization.
    Loads fp16/bf16 weights and quantizes them per-block during loading."""

    def __init__(
        self,
        *,
        layer: torch.nn.Module,
    ):
        super().__init__(
            weight_block_size=[128, 128],
            layer=layer,
        )

    def process_weights_after_loading(self, layer: Module) -> None:
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        fp8_dtype = current_platform.fp8_dtype()
        w13 = torch.empty_like(layer.w13_weight, dtype=fp8_dtype)
        w2 = torch.empty_like(layer.w2_weight, dtype=fp8_dtype)

        block_size = self.weight_block_size
        assert block_size is not None
        block_n, block_k = block_size
```
**EN:** Defines class `Fp8PerBlockOnlineMoEMethod` with base classes `_Fp8OnlineMoEBase` and decorators none. It acts as a backend or execution-method adapter and exposes 2 direct methods, with notable entries `__init__`, `process_weights_after_loading`. Its docstring says: Online blockwise FP8 MoE quantization.
**CN:** 定义类 `Fp8PerBlockOnlineMoEMethod`，其基类为 `_Fp8OnlineMoEBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 2 个方法，较重要的包括 `__init__`, `process_weights_after_loading`。 文档字符串进一步说明了该类的定位。

### Method `Fp8PerBlockOnlineMoEMethod.__init__` (lines 443-451)
```python
    def __init__(
        self,
        *,
        layer: torch.nn.Module,
    ):
        super().__init__(
            weight_block_size=[128, 128],
            layer=layer,
        )
```
**EN:** Defines function `Fp8PerBlockOnlineMoEMethod.__init__` with signature `__init__(self, *, layer: torch.nn.Module)`. It mainly works with `layer`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `Fp8PerBlockOnlineMoEMethod.__init__`，其签名为 `__init__(self, *, layer: torch.nn.Module)`。它主要围绕 `layer` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `Fp8PerBlockOnlineMoEMethod.process_weights_after_loading` (lines 453-512)
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        fp8_dtype = current_platform.fp8_dtype()
        w13 = torch.empty_like(layer.w13_weight, dtype=fp8_dtype)
        w2 = torch.empty_like(layer.w2_weight, dtype=fp8_dtype)

        block_size = self.weight_block_size
        assert block_size is not None
        block_n, block_k = block_size

        # Create block-shaped scales (computed here rather than in
        # create_weights because online quant doesn't need them until now).
        num_experts = layer.local_num_experts
        _, w13_out, w13_in = layer.w13_weight.shape
        _, w2_out, w2_in = layer.w2_weight.shape

        w13_scale = torch.ones(
            num_experts,
            (w13_out + block_n - 1) // block_n,
            (w13_in + block_k - 1) // block_k,
            dtype=torch.float32,
            device=w13.device,
        )
        w2_scale = torch.ones(
            num_experts,
            (w2_out + block_n - 1) // block_n,
            (w2_in + block_k - 1) // block_k,
            dtype=torch.float32,
            device=w2.device,
        )

        for expert in range(num_experts):
            w13[expert], w13_scale[expert] = per_block_cast_to_fp8(
                layer.w13_weight[expert],
                block_size=block_size,
                use_ue8m0=False,
            )
            w2[expert], w2_scale[expert] = per_block_cast_to_fp8(
                layer.w2_weight[expert],
                block_size=block_size,
                use_ue8m0=False,
            )

        layer.weight_block_size = block_size

        # Shuffle weights to runtime format and setup kernel.
        self._setup_kernel(
            layer,
            w13,
            w2,
            w13_scale,
            w2_scale,
            layer.w13_input_scale,
            layer.w2_input_scale,
        )

        # Prevent duplicate processing (e.g., during weight reload)
        layer._already_called_process_weights_after_loading = True
```
**EN:** Defines function `Fp8PerBlockOnlineMoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, iteration, validation/error handling, tensor/kernel operations. Key calls include `getattr`, `current_platform.fp8_dtype`, `torch.empty_like`, `torch.ones`, `range`, `self._setup_kernel`.
**CN:** 定义函数 `Fp8PerBlockOnlineMoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、循环处理、校验或报错逻辑、张量或内核操作。关键调用包括 `getattr`, `current_platform.fp8_dtype`, `torch.empty_like`, `torch.ones`, `range`, `self._setup_kernel`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `_Fp8OnlineLinearBase`, `Fp8PerTensorOnlineLinearMethod`, `Fp8PerBlockOnlineLinearMethod`, `_Fp8OnlineMoEBase`, `Fp8PerTensorOnlineMoEMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `_Fp8OnlineLinearBase`, `Fp8PerTensorOnlineLinearMethod`, `Fp8PerBlockOnlineLinearMethod`, `_Fp8OnlineMoEBase`, `Fp8PerTensorOnlineMoEMethod`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `torch`
- **Internal / 内部**: `vllm.envs`, `vllm`, `vllm.config`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.kernels.linear.scaled_mm`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.oracle.fp8`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.online.moe_base`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.model_executor.model_loader.reload.layerwise`, `vllm.model_executor.parameter`, `vllm.model_executor.utils`, `vllm.platforms`, `vllm.utils.deep_gemm`
