# mxfp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/online/mxfp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `Mxfp8OnlineLinearMethod`, `Mxfp8OnlineMoEMethod` for quantization backends, schemes, and utilities. / 实现 `Mxfp8OnlineLinearMethod`, `Mxfp8OnlineMoEMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 4-4)
```python
"""Online MXFP8 (microscaling FP8, block-32) quantization methods."""
```
**EN:** This docstring gives the module author's high-level intent: Online MXFP8 (microscaling FP8, block-32) quantization methods. It is useful for reading the rest of the file because later classes and helpers refine this stated purpose.
**CN:** 这个文档字符串给出了模块作者的高层意图：Online MXFP8 (microscaling FP8, block-32) quantization methods. 在阅读后续类和辅助函数时，可以把它视为整个文件的总纲。

### Imports and module setup (lines 6-34)
```python
from typing import TYPE_CHECKING

import torch
from torch.nn import Module

if TYPE_CHECKING:
    import vllm.model_executor.layers.fused_moe.modular_kernel as mk
    from vllm.model_executor.layers.fused_moe import (
        FusedMoEQuantConfig,
        RoutedExperts,
    )
    from vllm.model_executor.layers.fused_moe.oracle.fp8 import Fp8MoeBackend

from vllm.model_executor.kernels.linear import init_mxfp8_linear_kernel
from vllm.model_executor.layers.fused_moe.oracle.mxfp8 import (
    select_mxfp8_moe_backend,
)
from vllm.model_executor.layers.quantization.online.fp8 import (
    _Fp8OnlineLinearBase,
)
from vllm.model_executor.layers.quantization.online.moe_base import (
    OnlineMoEMethodBase,
)
from vllm.model_executor.layers.quantization.utils.mxfp8_utils import (
    MXFP8_BLOCK_SIZE,
    mxfp8_e4m3_quantize,
)
from vllm.model_executor.utils import replace_parameter
from vllm.platforms import current_platform
```
**EN:** This opening block pulls in external dependencies such as `typing`, `torch` and internal modules such as `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.fused_moe.oracle.mxfp8`, `vllm.model_executor.layers.quantization.online.fp8`, `vllm.model_executor.layers.quantization.online.moe_base`, `vllm.model_executor.layers.quantization.utils.mxfp8_utils`, `vllm.model_executor.utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `torch`）以及内部模块（如 `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.fused_moe.oracle.mxfp8`, `vllm.model_executor.layers.quantization.online.fp8`, `vllm.model_executor.layers.quantization.online.moe_base`, `vllm.model_executor.layers.quantization.utils.mxfp8_utils`, `vllm.model_executor.utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `Mxfp8OnlineLinearMethod` overview (lines 37-94)
```python
class Mxfp8OnlineLinearMethod(_Fp8OnlineLinearBase):
    """Online MXFP8 linear method.
    Loads bf16/fp16 checkpoints and quantizes weights to MXFP8 (microscaling
    FP8 with block-32 scales) during weight loading.
    """

    def __init__(self):
        super().__init__()
        self.kernel = init_mxfp8_linear_kernel()

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
        if input_size_per_partition % MXFP8_BLOCK_SIZE != 0:
            raise ValueError(
                f"MXFP8 requires input_size_per_partition "
                f"({input_size_per_partition}) to be divisible by "
                f"{MXFP8_BLOCK_SIZE}."
```
**EN:** Defines class `Mxfp8OnlineLinearMethod` with base classes `_Fp8OnlineLinearBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Online MXFP8 linear method.
**CN:** 定义类 `Mxfp8OnlineLinearMethod`，其基类为 `_Fp8OnlineLinearBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `Mxfp8OnlineLinearMethod.__init__` (lines 43-45)
```python
    def __init__(self):
        super().__init__()
        self.kernel = init_mxfp8_linear_kernel()
```
**EN:** Defines function `Mxfp8OnlineLinearMethod.__init__` with signature `__init__(self)`. It mainly works with object context only; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `init_mxfp8_linear_kernel`, `super`.
**CN:** 定义函数 `Mxfp8OnlineLinearMethod.__init__`，其签名为 `__init__(self)`。它主要围绕 仅依赖对象上下文 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `init_mxfp8_linear_kernel`, `super`。

### Method `Mxfp8OnlineLinearMethod.create_weights` (lines 47-72)
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
        if input_size_per_partition % MXFP8_BLOCK_SIZE != 0:
            raise ValueError(
                f"MXFP8 requires input_size_per_partition "
                f"({input_size_per_partition}) to be divisible by "
                f"{MXFP8_BLOCK_SIZE}."
            )

        super().create_weights(
            layer,
            input_size_per_partition,
            output_partition_sizes,
            input_size,
            output_size,
            params_dtype,
            **extra_weight_attrs,
        )
```
**EN:** Defines function `Mxfp8OnlineLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling. Key calls include `super.create_weights`, `ValueError`, `super`.
**CN:** 定义函数 `Mxfp8OnlineLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.create_weights`, `ValueError`, `super`。

### Method `Mxfp8OnlineLinearMethod.process_weights_after_loading` (lines 74-86)
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        weight_fp8, weight_scale = mxfp8_e4m3_quantize(layer.weight.contiguous())

        layer.input_scale = None
        replace_parameter(layer, "weight", weight_fp8.data)
        replace_parameter(layer, "weight_scale", weight_scale.data)

        self.kernel.process_weights_after_loading(layer)

        layer._already_called_process_weights_after_loading = True
```
**EN:** Defines function `Mxfp8OnlineLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching. Key calls include `getattr`, `mxfp8_e4m3_quantize`, `replace_parameter`, `self.kernel.process_weights_after_loading`, `layer.weight.contiguous`.
**CN:** 定义函数 `Mxfp8OnlineLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断。关键调用包括 `getattr`, `mxfp8_e4m3_quantize`, `replace_parameter`, `self.kernel.process_weights_after_loading`, `layer.weight.contiguous`。

### Method `Mxfp8OnlineLinearMethod.apply` (lines 88-94)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.kernel.apply_weights(layer, x, bias)
```
**EN:** Defines function `Mxfp8OnlineLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.kernel.apply_weights`.
**CN:** 定义函数 `Mxfp8OnlineLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.kernel.apply_weights`。

### Class `Mxfp8OnlineMoEMethod` overview (lines 97-253)
```python
class Mxfp8OnlineMoEMethod(OnlineMoEMethodBase):
    """MoE method for online MXFP8 (block) quantization."""

    fp8_backend: "Fp8MoeBackend"
    experts_cls: "type[mk.FusedMoEExperts] | None"

    def __init__(self, *, layer: torch.nn.Module):
        super().__init__(layer.moe_config)
        self.weight_block_size: list[int] = [1, MXFP8_BLOCK_SIZE]
        self.weight_scale_name = "weight_scale"

        self.fp8_backend, self.experts_cls = select_mxfp8_moe_backend(config=self.moe)

    def create_weights(
        self,
        layer: Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        if (
            hidden_size % MXFP8_BLOCK_SIZE != 0
            or intermediate_size_per_partition % MXFP8_BLOCK_SIZE != 0
```
**EN:** Defines class `Mxfp8OnlineMoEMethod` with base classes `OnlineMoEMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 6 direct methods, with notable entries `__init__`, `create_weights`, `_quantize_mxfp8_moe_weight`, `_setup_kernel`, `get_fused_moe_quant_config`, `process_weights_after_loading`. Its docstring says: MoE method for online MXFP8 (block) quantization.
**CN:** 定义类 `Mxfp8OnlineMoEMethod`，其基类为 `OnlineMoEMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 6 个方法，较重要的包括 `__init__`, `create_weights`, `_quantize_mxfp8_moe_weight`, `_setup_kernel`, `get_fused_moe_quant_config`, `process_weights_after_loading`。 文档字符串进一步说明了该类的定位。

### Method `Mxfp8OnlineMoEMethod.__init__` (lines 103-108)
```python
    def __init__(self, *, layer: torch.nn.Module):
        super().__init__(layer.moe_config)
        self.weight_block_size: list[int] = [1, MXFP8_BLOCK_SIZE]
        self.weight_scale_name = "weight_scale"

        self.fp8_backend, self.experts_cls = select_mxfp8_moe_backend(config=self.moe)
```
**EN:** Defines function `Mxfp8OnlineMoEMethod.__init__` with signature `__init__(self, *, layer: torch.nn.Module)`. It mainly works with `layer`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `select_mxfp8_moe_backend`, `super`.
**CN:** 定义函数 `Mxfp8OnlineMoEMethod.__init__`，其签名为 `__init__(self, *, layer: torch.nn.Module)`。它主要围绕 `layer` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `select_mxfp8_moe_backend`, `super`。

### Method `Mxfp8OnlineMoEMethod.create_weights` (lines 110-137)
```python
    def create_weights(
        self,
        layer: Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        if (
            hidden_size % MXFP8_BLOCK_SIZE != 0
            or intermediate_size_per_partition % MXFP8_BLOCK_SIZE != 0
        ):
            raise ValueError(
                "Online MXFP8 MoE requires hidden/intermediate sizes divisible "
                f"by {MXFP8_BLOCK_SIZE}."
            )

        super().create_weights(
            layer=layer,
            num_experts=num_experts,
            hidden_size=hidden_size,
            intermediate_size_per_partition=intermediate_size_per_partition,
            params_dtype=params_dtype,
            **extra_weight_attrs,
        )

        layer.weight_block_size = [1, MXFP8_BLOCK_SIZE]
```
**EN:** Defines function `Mxfp8OnlineMoEMethod.create_weights` with signature `create_weights(self, layer: Module, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling. Key calls include `super.create_weights`, `ValueError`, `super`.
**CN:** 定义函数 `Mxfp8OnlineMoEMethod.create_weights`，其签名为 `create_weights(self, layer: Module, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.create_weights`, `ValueError`, `super`。

### Method `Mxfp8OnlineMoEMethod._quantize_mxfp8_moe_weight` (lines 139-160)
```python
    def _quantize_mxfp8_moe_weight(
        self, weight: torch.Tensor
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """Batch quantization: bf16/fp16 weights -> MXFP8 (fp8 + uint8 scales)."""
        E = weight.size(0)
        first_q, first_s = mxfp8_e4m3_quantize(weight[0], is_sf_swizzled_layout=False)
        # Pre-allocate the output tensors rather than stacking.
        # This is important for consistent memory layout.
        w_quant = torch.empty(
            (E, *first_q.shape), dtype=first_q.dtype, device=weight.device
        )
        w_scales = torch.empty(
            (E, *first_s.shape), dtype=first_s.dtype, device=weight.device
        )
        w_quant[0] = first_q
        w_scales[0] = first_s
        for i in range(1, E):
            w_quant[i], w_scales[i] = mxfp8_e4m3_quantize(
                weight[i], is_sf_swizzled_layout=False
            )

        return w_quant, w_scales
```
**EN:** Defines function `Mxfp8OnlineMoEMethod._quantize_mxfp8_moe_weight` with signature `_quantize_mxfp8_moe_weight(self, weight: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `weight`; handles quantization-related transformation logic. The body uses iteration, tensor/kernel operations. Key calls include `weight.size`, `mxfp8_e4m3_quantize`, `torch.empty`, `range`.
**CN:** 定义函数 `Mxfp8OnlineMoEMethod._quantize_mxfp8_moe_weight`，其签名为 `_quantize_mxfp8_moe_weight(self, weight: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `weight` 展开；处理量化相关的变换逻辑。函数体包含循环处理、张量或内核操作。关键调用包括 `weight.size`, `mxfp8_e4m3_quantize`, `torch.empty`, `range`。

### Method `Mxfp8OnlineMoEMethod._setup_kernel` (lines 162-203)
```python
    def _setup_kernel(
        self,
        layer: "RoutedExperts",
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
**EN:** Defines function `Mxfp8OnlineMoEMethod._setup_kernel` with signature `_setup_kernel(self, layer: 'RoutedExperts', w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor, w13_input_scale: torch.Tensor | None, w2_input_scale: torch.Tensor | None) -> None`. It mainly works with `layer`, `w13`, `w2`, `w13_scale`, `w2_scale`, `w13_input_scale`, `w2_input_scale`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `convert_to_fp8_moe_kernel_format`, `replace_parameter`, `self.get_fused_moe_quant_config`, `make_fp8_moe_kernel`, `layer._expert_routing_tables`.
**CN:** 定义函数 `Mxfp8OnlineMoEMethod._setup_kernel`，其签名为 `_setup_kernel(self, layer: 'RoutedExperts', w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor, w13_input_scale: torch.Tensor | None, w2_input_scale: torch.Tensor | None) -> None`。它主要围绕 `layer`, `w13`, `w2`, `w13_scale`, `w2_scale`, `w13_input_scale`, `w2_input_scale` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `convert_to_fp8_moe_kernel_format`, `replace_parameter`, `self.get_fused_moe_quant_config`, `make_fp8_moe_kernel`, `layer._expert_routing_tables`。

### Method `Mxfp8OnlineMoEMethod.get_fused_moe_quant_config` (lines 205-228)
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
**EN:** Defines function `Mxfp8OnlineMoEMethod.get_fused_moe_quant_config` with signature `get_fused_moe_quant_config(self, layer: torch.nn.Module) -> 'FusedMoEQuantConfig'`. It mainly works with `layer`; handles quantization-related transformation logic. The body uses mostly straightforward data movement and object wiring. Key calls include `getattr`, `make_fp8_moe_quant_config`, `self._maybe_inject_biases`.
**CN:** 定义函数 `Mxfp8OnlineMoEMethod.get_fused_moe_quant_config`，其签名为 `get_fused_moe_quant_config(self, layer: torch.nn.Module) -> 'FusedMoEQuantConfig'`。它主要围绕 `layer` 展开；处理量化相关的变换逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `getattr`, `make_fp8_moe_quant_config`, `self._maybe_inject_biases`。

### Method `Mxfp8OnlineMoEMethod.process_weights_after_loading` (lines 230-253)
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        fp8_dtype = current_platform.fp8_dtype()
        w13 = torch.empty_like(layer.w13_weight, dtype=fp8_dtype)
        w2 = torch.empty_like(layer.w2_weight, dtype=fp8_dtype)
        layer.w13_input_scale = None
        layer.w2_input_scale = None

        w13, w13_scale = self._quantize_mxfp8_moe_weight(layer.w13_weight)
        w2, w2_scale = self._quantize_mxfp8_moe_weight(layer.w2_weight)

        self._setup_kernel(
            layer,
            w13,
            w2,
            w13_scale,
            w2_scale,
            layer.w13_input_scale,
            layer.w2_input_scale,
        )

        layer._already_called_process_weights_after_loading = True
```
**EN:** Defines function `Mxfp8OnlineMoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, tensor/kernel operations. Key calls include `getattr`, `current_platform.fp8_dtype`, `torch.empty_like`, `self._quantize_mxfp8_moe_weight`, `self._setup_kernel`.
**CN:** 定义函数 `Mxfp8OnlineMoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、张量或内核操作。关键调用包括 `getattr`, `current_platform.fp8_dtype`, `torch.empty_like`, `self._quantize_mxfp8_moe_weight`, `self._setup_kernel`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `Mxfp8OnlineLinearMethod`, `Mxfp8OnlineMoEMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `Mxfp8OnlineLinearMethod`, `Mxfp8OnlineMoEMethod`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `torch`
- **Internal / 内部**: `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.fused_moe.oracle.mxfp8`, `vllm.model_executor.layers.quantization.online.fp8`, `vllm.model_executor.layers.quantization.online.moe_base`, `vllm.model_executor.layers.quantization.utils.mxfp8_utils`, `vllm.model_executor.utils`, `vllm.platforms`
