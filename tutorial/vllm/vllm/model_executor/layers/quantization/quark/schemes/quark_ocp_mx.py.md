# quark_ocp_mx.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/quark/schemes/quark_ocp_mx.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `QuarkOCP_MX` for quantization backends, schemes, and utilities. / 实现 `QuarkOCP_MX`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-34)
```python
from collections.abc import Callable
from fractions import Fraction
from functools import partial
from typing import Any

import torch
import torch.nn.functional as F

from vllm._aiter_ops import rocm_aiter_ops
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.mxfp4_utils import (
    dequant_mxfp4,
    quant_dequant_mxfp4,
)
from vllm.model_executor.layers.quantization.utils.mxfp6_utils import (
    dequant_mxfp6,
    quant_dequant_mxfp6,
)
from vllm.model_executor.layers.quantization.utils.ocp_mx_utils import (
    OCP_MX_BLOCK_SIZE,
    OCP_MX_Scheme,
)
from vllm.model_executor.parameter import (
    GroupQuantScaleParameter,
    ModelWeightParameter,
    PackedvLLMParameter,
)
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform

from .quark_scheme import QuarkScheme
```
**EN:** This opening block pulls in external dependencies such as `collections`, `fractions`, `functools`, `typing`, `torch` and internal modules such as `vllm._aiter_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.mxfp4_utils`, `vllm.model_executor.layers.quantization.utils.mxfp6_utils`, `vllm.model_executor.layers.quantization.utils.ocp_mx_utils`, `vllm.model_executor.parameter`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `collections`, `fractions`, `functools`, `typing`, `torch`）以及内部模块（如 `vllm._aiter_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.mxfp4_utils`, `vllm.model_executor.layers.quantization.utils.mxfp6_utils`, `vllm.model_executor.layers.quantization.utils.ocp_mx_utils`, `vllm.model_executor.parameter`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 36-36)
```python
logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Class `QuarkOCP_MX` overview (lines 149-381)
```python
class QuarkOCP_MX(QuarkScheme):
    def __init__(
        self,
        weight_quant_spec: dict[str, Any],
        input_quant_spec: dict[str, Any] | None,
        dynamic_mxfp4_quant: bool = False,
    ):
        self.out_dtype = torch.get_default_dtype()
        self.qscheme = "per_group"
        self.weight_quant_spec = weight_quant_spec
        self.input_quant_spec = input_quant_spec
        self.dynamic_mxfp4_quant = dynamic_mxfp4_quant
        self.weight_dtype = weight_quant_spec["dtype"].replace("fp", "mxfp")
        self.input_dtype: str | None = None
        if input_quant_spec is not None:
            input_quant = input_quant_spec["dtype"]
            if input_quant == "fp8_e4m3":
                self.input_dtype = "fp8"
            else:
                self.input_dtype = input_quant.replace("fp", "mxfp")

        self.ocp_mx_scheme = OCP_MX_Scheme.from_quant_dtype(
            self.input_dtype, self.weight_dtype
        )
```
**EN:** Defines class `QuarkOCP_MX` with base classes `QuarkScheme` and decorators none. It acts as a quantization-oriented module building block and exposes 7 direct methods, with notable entries `__init__`, `get_packed_dim`, `get_min_capability`, `process_dynamic_mxfp4_weights_after_loading`, `process_weights_after_loading`, `create_weights`.
**CN:** 定义类 `QuarkOCP_MX`，其基类为 `QuarkScheme`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 7 个方法，较重要的包括 `__init__`, `get_packed_dim`, `get_min_capability`, `process_dynamic_mxfp4_weights_after_loading`, `process_weights_after_loading`, `create_weights`。

### Method `QuarkOCP_MX.__init__` (lines 150-240)
```python
    def __init__(
        self,
        weight_quant_spec: dict[str, Any],
        input_quant_spec: dict[str, Any] | None,
        dynamic_mxfp4_quant: bool = False,
    ):
        self.out_dtype = torch.get_default_dtype()
        self.qscheme = "per_group"
        self.weight_quant_spec = weight_quant_spec
        self.input_quant_spec = input_quant_spec
        self.dynamic_mxfp4_quant = dynamic_mxfp4_quant
        self.weight_dtype = weight_quant_spec["dtype"].replace("fp", "mxfp")
        self.input_dtype: str | None = None
        if input_quant_spec is not None:
            input_quant = input_quant_spec["dtype"]
            if input_quant == "fp8_e4m3":
                self.input_dtype = "fp8"
            else:
                self.input_dtype = input_quant.replace("fp", "mxfp")

        self.ocp_mx_scheme = OCP_MX_Scheme.from_quant_dtype(
            self.input_dtype, self.weight_dtype
        )

        if self.weight_dtype == "mxfp4":
            self.packed_factor: int | Fraction = 2
            self.dequant_func = dequant_mxfp4
        else:
            self.packed_factor = Fraction(numerator=8, denominator=6)
            self.dequant_func = partial(
                dequant_mxfp6, quant_dtype=self.weight_dtype.replace("mx", "")
            )

        if self.input_dtype is None:
            self.quant_dequant_func: Callable[[torch.Tensor], torch.Tensor] = (
                lambda x: x
            )  # no input Q/DQ for weight-only
        elif self.input_dtype == "mxfp4":
# ... truncated for analysis ...
                "layers computed in high precision."
            )

        if current_platform.supports_mx() and (
            self.input_dtype != "mxfp4" or self.weight_dtype != "mxfp4"
        ):
            logger.warning_once(
                "The current platform supports native MXFP4/MXFP6 "
                f"computation, but kernels for input_dtype={self.input_dtype} "
                f"and weight_dtype={self.weight_dtype} are not yet integrated "
                "in vLLM. Simulated weight dequantization and activation "
                "QDQ (quantize and dequantize) will be used, with the linear "
                "layers computed in high precision."
            )
```
**EN:** Defines function `QuarkOCP_MX.__init__` with signature `__init__(self, weight_quant_spec: dict[str, Any], input_quant_spec: dict[str, Any] | None, dynamic_mxfp4_quant: bool=False)`. It mainly works with `weight_quant_spec`, `input_quant_spec`, `dynamic_mxfp4_quant`; initializes the object state and cached resources. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `torch.get_default_dtype`, `weight_quant_spec.replace`, `OCP_MX_Scheme.from_quant_dtype`, `rocm_aiter_ops.is_asm_fp4_gemm_dynamic_quant_enabled`, `Fraction`, `partial`.
**CN:** 定义函数 `QuarkOCP_MX.__init__`，其签名为 `__init__(self, weight_quant_spec: dict[str, Any], input_quant_spec: dict[str, Any] | None, dynamic_mxfp4_quant: bool=False)`。它主要围绕 `weight_quant_spec`, `input_quant_spec`, `dynamic_mxfp4_quant` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `torch.get_default_dtype`, `weight_quant_spec.replace`, `OCP_MX_Scheme.from_quant_dtype`, `rocm_aiter_ops.is_asm_fp4_gemm_dynamic_quant_enabled`, `Fraction`, `partial`。

### Method `QuarkOCP_MX.get_packed_dim` (lines 242-255)
```python
    def get_packed_dim(self, dim: int, quant_dtype: str):
        if quant_dtype == "mxfp4":
            assert dim % 2 == 0
            return dim // 2
        elif quant_dtype in {"mxfp6_e3m2", "mxfp6_e2m3"}:
            # FP6 packs 4 * 6 = 24 bits on 3 bytes.
            assert (dim * 3) % 4 == 0
            return (dim * 3) // 4
        else:
            raise NotImplementedError(
                "Unsupported quant_dtype in QuarkOCP_MX.get_packed_dim, "
                f"got quant_dtype={quant_dtype}. Something is wrong, please "
                "open an issue."
            )
```
**EN:** Defines function `QuarkOCP_MX.get_packed_dim` with signature `get_packed_dim(self, dim: int, quant_dtype: str)`. It mainly works with `dim`, `quant_dtype`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `NotImplementedError`.
**CN:** 定义函数 `QuarkOCP_MX.get_packed_dim`，其签名为 `get_packed_dim(self, dim: int, quant_dtype: str)`。它主要围绕 `dim`, `quant_dtype` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `NotImplementedError`。

### Method `QuarkOCP_MX.get_min_capability` (lines 258-259)
```python
    def get_min_capability(cls) -> int:
        return 70
```
**EN:** Defines function `QuarkOCP_MX.get_min_capability` with signature `get_min_capability(cls) -> int`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `QuarkOCP_MX.get_min_capability`，其签名为 `get_min_capability(cls) -> int`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `QuarkOCP_MX.process_weights_after_loading` (lines 268-303)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.weight = torch.nn.Parameter(layer.weight.data, requires_grad=False)

        if self.emulate:
            if self.dynamic_mxfp4_quant:
                self.process_dynamic_mxfp4_weights_after_loading(layer)
            else:
                layer.weight_scale = torch.nn.Parameter(
                    layer.weight_scale.data, requires_grad=False
                )
        else:
            if self.dynamic_mxfp4_quant:
                self.process_dynamic_mxfp4_weights_after_loading(layer)
            elif self.rocm_use_aiter_fp4_asm_gemm:
                # shuffle weight scale
                weight_scale_shuffle = layer.weight_scale.data
                sm, sn = weight_scale_shuffle.shape
                weight_scale_shuffle = weight_scale_shuffle.view(
                    sm // 32, 2, 16, sn // 8, 2, 4, 1
                )
                weight_scale_shuffle = weight_scale_shuffle.permute(
                    0, 3, 5, 2, 4, 1, 6
                ).contiguous()
                weight_scale_shuffle = weight_scale_shuffle.view(sm, sn)
                layer.weight_scale = torch.nn.Parameter(
                    weight_scale_shuffle, requires_grad=False
                )

                # shuffle weight
                weight_shuffle = layer.weight.data
                weight_shuffle = shuffle_weight(weight_shuffle, layout=(16, 16))
                layer.weight = torch.nn.Parameter(weight_shuffle, requires_grad=False)
            else:
                layer.weight_scale = torch.nn.Parameter(
                    layer.weight_scale.data.T.contiguous(), requires_grad=False
                )
```
**EN:** Defines function `QuarkOCP_MX.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, tensor/kernel operations. Key calls include `torch.nn.Parameter`, `self.process_dynamic_mxfp4_weights_after_loading`, `weight_scale_shuffle.view`, `weight_scale_shuffle.permute.contiguous`, `shuffle_weight`, `layer.weight_scale.data.T.contiguous`.
**CN:** 定义函数 `QuarkOCP_MX.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.nn.Parameter`, `self.process_dynamic_mxfp4_weights_after_loading`, `weight_scale_shuffle.view`, `weight_scale_shuffle.permute.contiguous`, `shuffle_weight`, `layer.weight_scale.data.T.contiguous`。

### Method `QuarkOCP_MX.create_weights` (lines 305-358)
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
        if self.dynamic_mxfp4_quant:
            weight = ModelWeightParameter(
                data=torch.empty(
                    sum(output_partition_sizes),
                    input_size_per_partition,
                    dtype=params_dtype,
                ),
                input_dim=1,
                output_dim=0,
                weight_loader=weight_loader,
            )

            layer.register_parameter("weight", weight)
            set_weight_attrs(weight, kwargs)
        else:
            output_size_per_partition = sum(output_partition_sizes)
            layer.logical_widths = output_partition_sizes

            # WEIGHT
            weight = PackedvLLMParameter(
                data=torch.empty(
                    output_size_per_partition,
                    self.get_packed_dim(input_size_per_partition, self.weight_dtype),
                    dtype=torch.uint8,
                ),
                input_dim=1,
                output_dim=0,
                packed_dim=1,
                packed_factor=self.packed_factor,
                weight_loader=weight_loader,
            )
            layer.register_parameter("weight", weight)

            # WEIGHT SCALE
            weight_scale = GroupQuantScaleParameter(
                data=torch.empty(
                    output_size_per_partition,
                    input_size_per_partition // OCP_MX_BLOCK_SIZE,
                    dtype=torch.uint8,
                ),
                input_dim=1,
                output_dim=0,
                weight_loader=weight_loader,
            )
            layer.register_parameter("weight_scale", weight_scale)
```
**EN:** Defines function `QuarkOCP_MX.create_weights` with signature `create_weights(self, layer: torch.nn.Module, output_partition_sizes: list[int], input_size_per_partition: int, params_dtype: torch.dtype, weight_loader: Callable, **kwargs)`. It mainly works with `layer`, `output_partition_sizes`, `input_size_per_partition`, `params_dtype`, `weight_loader`, `**kwargs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `ModelWeightParameter`, `layer.register_parameter`, `set_weight_attrs`, `sum`, `PackedvLLMParameter`, `GroupQuantScaleParameter`.
**CN:** 定义函数 `QuarkOCP_MX.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, output_partition_sizes: list[int], input_size_per_partition: int, params_dtype: torch.dtype, weight_loader: Callable, **kwargs)`。它主要围绕 `layer`, `output_partition_sizes`, `input_size_per_partition`, `params_dtype`, `weight_loader`, `**kwargs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `ModelWeightParameter`, `layer.register_parameter`, `set_weight_attrs`, `sum`, `PackedvLLMParameter`, `GroupQuantScaleParameter`。

### Method `QuarkOCP_MX.apply_weights` (lines 360-381)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if self.emulate:
            dq_w = self.dequant_func(layer.weight, layer.weight_scale, x.dtype)
            qdq_x = self.quant_dequant_func(x)
            return F.linear(qdq_x, dq_w, bias)
        y = torch.ops.vllm.gemm_with_dynamic_quant(
            x,
            layer.weight,
            layer.weight_scale,
            self.rocm_use_aiter_fp4_asm_gemm,
            self.out_dtype,
        )
        # gemm_with_dynamic_quant has no bias argument; add it here so the
        # native path matches F.linear (e.g. qkv_proj with qkv_bias=True).
        if bias is not None:
            y = y + bias
        return y
```
**EN:** Defines function `QuarkOCP_MX.apply_weights` with signature `apply_weights(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching, tensor/kernel operations. Key calls include `torch.ops.vllm.gemm_with_dynamic_quant`, `self.dequant_func`, `self.quant_dequant_func`, `F.linear`.
**CN:** 定义函数 `QuarkOCP_MX.apply_weights`，其签名为 `apply_weights(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.ops.vllm.gemm_with_dynamic_quant`, `self.dequant_func`, `self.quant_dequant_func`, `F.linear`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `QuarkOCP_MX`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `QuarkOCP_MX`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `collections`, `fractions`, `functools`, `typing`, `torch`
- **Internal / 内部**: `vllm._aiter_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.mxfp4_utils`, `vllm.model_executor.layers.quantization.utils.mxfp6_utils`, `vllm.model_executor.layers.quantization.utils.ocp_mx_utils`, `vllm.model_executor.parameter`, `vllm.model_executor.utils`, `vllm.platforms`, `.quark_scheme`
