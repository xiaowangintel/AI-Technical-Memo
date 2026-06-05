# fp_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/fp_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `FPQuantConfig`, `FPQuantLinearMethod` for quantization backends, schemes, and utilities. / 实现 `FPQuantConfig`, `FPQuantLinearMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 6-27)
```python
from typing import Any, Literal, cast

import torch
from torch.nn.parameter import Parameter

from vllm._custom_ops import (
    cutlass_scaled_fp4_mm,
    fusedQuantizeMx,
    fusedQuantizeNv,
    matmul_mxf4_bf16_tn,
)
from vllm.model_executor.layers.linear import (
    LinearBase,
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from vllm.model_executor.layers.quantization import QuantizationMethods
from vllm.model_executor.layers.quantization.base_config import QuantizationConfig
from vllm.model_executor.layers.quantization.qutlass_utils import to_blocked
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This opening block pulls in external dependencies such as `typing`, `torch` and internal modules such as `vllm._custom_ops`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.qutlass_utils`, `vllm.model_executor.utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `torch`）以及内部模块（如 `vllm._custom_ops`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.qutlass_utils`, `vllm.model_executor.utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `fused_quantize_mx` (lines 251-258)
```python
def fused_quantize_mx(
    x_flat: torch.Tensor, hadamard_matrix: torch.Tensor, forward_method: str
) -> tuple[torch.Tensor, torch.Tensor]:
    return fusedQuantizeMx(
        x_flat,
        hadamard_matrix,
        method=cast(Literal["quest", "abs_max"], forward_method),
    )
```
**EN:** Defines function `fused_quantize_mx` with signature `fused_quantize_mx(x_flat: torch.Tensor, hadamard_matrix: torch.Tensor, forward_method: str) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x_flat`, `hadamard_matrix`, `forward_method`; handles quantization-related transformation logic. The body uses mostly straightforward data movement and object wiring. Key calls include `fusedQuantizeMx`, `cast`.
**CN:** 定义函数 `fused_quantize_mx`，其签名为 `fused_quantize_mx(x_flat: torch.Tensor, hadamard_matrix: torch.Tensor, forward_method: str) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x_flat`, `hadamard_matrix`, `forward_method` 展开；处理量化相关的变换逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `fusedQuantizeMx`, `cast`。

### Function `fused_quantize_mx_fake` (lines 261-273)
```python
def fused_quantize_mx_fake(x_flat, hadamard_matrix, forward_method):
    rows, cols = x_flat.size(0), x_flat.size(1) // 32
    padded_rows = ((rows + 128 - 1) // 128) * 128
    padded_cols = ((cols + 4 - 1) // 4) * 4

    xh_e2m1 = torch.empty(
        x_flat.size(0), x_flat.size(1) // 2, dtype=torch.uint8, device=x_flat.device
    )
    xh_e8m0 = torch.empty(
        padded_rows, padded_cols, dtype=torch.float8_e8m0fnu, device=x_flat.device
    )

    return xh_e2m1, xh_e8m0
```
**EN:** Defines function `fused_quantize_mx_fake` with signature `fused_quantize_mx_fake(x_flat, hadamard_matrix, forward_method)`. It mainly works with `x_flat`, `hadamard_matrix`, `forward_method`; handles quantization-related transformation logic. The body uses tensor/kernel operations. Key calls include `torch.empty`, `x_flat.size`.
**CN:** 定义函数 `fused_quantize_mx_fake`，其签名为 `fused_quantize_mx_fake(x_flat, hadamard_matrix, forward_method)`。它主要围绕 `x_flat`, `hadamard_matrix`, `forward_method` 展开；处理量化相关的变换逻辑。函数体包含张量或内核操作。关键调用包括 `torch.empty`, `x_flat.size`。

### Function `matmul_mxf4_bf16` (lines 285-298)
```python
def matmul_mxf4_bf16(
    x: torch.Tensor,
    w: torch.Tensor,
    xs: torch.Tensor,
    ws: torch.Tensor,
    alpha: torch.Tensor,
) -> torch.Tensor:
    return matmul_mxf4_bf16_tn(
        x,
        w,
        to_blocked(xs, backend="triton").view(torch.float8_e8m0fnu),
        to_blocked(ws, backend="triton").view(torch.float8_e8m0fnu),
        alpha,
    )
```
**EN:** Defines function `matmul_mxf4_bf16` with signature `matmul_mxf4_bf16(x: torch.Tensor, w: torch.Tensor, xs: torch.Tensor, ws: torch.Tensor, alpha: torch.Tensor) -> torch.Tensor`. It mainly works with `x`, `w`, `xs`, `ws`, `alpha`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `matmul_mxf4_bf16_tn`, `to_blocked.view`, `to_blocked`.
**CN:** 定义函数 `matmul_mxf4_bf16`，其签名为 `matmul_mxf4_bf16(x: torch.Tensor, w: torch.Tensor, xs: torch.Tensor, ws: torch.Tensor, alpha: torch.Tensor) -> torch.Tensor`。它主要围绕 `x`, `w`, `xs`, `ws`, `alpha` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `matmul_mxf4_bf16_tn`, `to_blocked.view`, `to_blocked`。

### Function `fused_quantize_nv` (lines 314-317)
```python
def fused_quantize_nv(
    x_flat: torch.Tensor, hadamard_matrix: torch.Tensor, global_scale: torch.Tensor
) -> tuple[torch.Tensor, torch.Tensor]:
    return fusedQuantizeNv(x_flat, hadamard_matrix, global_scale)
```
**EN:** Defines function `fused_quantize_nv` with signature `fused_quantize_nv(x_flat: torch.Tensor, hadamard_matrix: torch.Tensor, global_scale: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x_flat`, `hadamard_matrix`, `global_scale`; handles quantization-related transformation logic. The body uses mostly straightforward data movement and object wiring. Key calls include `fusedQuantizeNv`.
**CN:** 定义函数 `fused_quantize_nv`，其签名为 `fused_quantize_nv(x_flat: torch.Tensor, hadamard_matrix: torch.Tensor, global_scale: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x_flat`, `hadamard_matrix`, `global_scale` 展开；处理量化相关的变换逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `fusedQuantizeNv`。

### Function `fused_quantize_nv_fake` (lines 320-332)
```python
def fused_quantize_nv_fake(x_flat, hadamard_matrix, global_scale):
    rows, cols = x_flat.size(0), x_flat.size(1) // 16
    padded_rows = ((rows + 128 - 1) // 128) * 128
    padded_cols = ((cols + 4 - 1) // 4) * 4

    xh_e2m1 = torch.empty(
        x_flat.size(0), x_flat.size(1) // 2, dtype=torch.uint8, device=x_flat.device
    )
    xh_e8m0 = torch.empty(
        padded_rows, padded_cols, dtype=torch.float8_e4m3fn, device=x_flat.device
    )

    return xh_e2m1, xh_e8m0
```
**EN:** Defines function `fused_quantize_nv_fake` with signature `fused_quantize_nv_fake(x_flat, hadamard_matrix, global_scale)`. It mainly works with `x_flat`, `hadamard_matrix`, `global_scale`; handles quantization-related transformation logic. The body uses tensor/kernel operations. Key calls include `torch.empty`, `x_flat.size`.
**CN:** 定义函数 `fused_quantize_nv_fake`，其签名为 `fused_quantize_nv_fake(x_flat, hadamard_matrix, global_scale)`。它主要围绕 `x_flat`, `hadamard_matrix`, `global_scale` 展开；处理量化相关的变换逻辑。函数体包含张量或内核操作。关键调用包括 `torch.empty`, `x_flat.size`。

### Function `matmul_nvf4_bf16` (lines 344-362)
```python
def matmul_nvf4_bf16(
    x: torch.Tensor,
    w: torch.Tensor,
    xs: torch.Tensor,
    ws: torch.Tensor,
    alpha: torch.Tensor,
) -> torch.Tensor:
    return cutlass_scaled_fp4_mm(
        x,
        w,
        to_blocked(xs, backend="triton")
        .view(torch.float8_e4m3fn)
        .view(-1, x.shape[1] // 8),  # *2//16
        to_blocked(ws, backend="triton")
        .view(torch.float8_e4m3fn)
        .view(-1, x.shape[1] // 8),
        alpha,
        torch.bfloat16,
    )
```
**EN:** Defines function `matmul_nvf4_bf16` with signature `matmul_nvf4_bf16(x: torch.Tensor, w: torch.Tensor, xs: torch.Tensor, ws: torch.Tensor, alpha: torch.Tensor) -> torch.Tensor`. It mainly works with `x`, `w`, `xs`, `ws`, `alpha`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `cutlass_scaled_fp4_mm`, `to_blocked.view.view`, `to_blocked.view`, `to_blocked`.
**CN:** 定义函数 `matmul_nvf4_bf16`，其签名为 `matmul_nvf4_bf16(x: torch.Tensor, w: torch.Tensor, xs: torch.Tensor, ws: torch.Tensor, alpha: torch.Tensor) -> torch.Tensor`。它主要围绕 `x`, `w`, `xs`, `ws`, `alpha` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cutlass_scaled_fp4_mm`, `to_blocked.view.view`, `to_blocked.view`, `to_blocked`。

### Function `quantized_forward` (lines 378-420)
```python
def quantized_forward(
    x: torch.Tensor,
    qweight: torch.Tensor,
    weight_scales: torch.Tensor,
    weight_global_scale: torch.Tensor,
    act_global_scale: torch.Tensor,
    bias: torch.Tensor | None,
    forward_hadamard_matrix: torch.Tensor,
    forward_method: str,
    forward_dtype: str,
) -> torch.Tensor:
    x_flat = x.contiguous().flatten(end_dim=-2)

    if forward_dtype == "mxfp4":
        x_flat_q, x_flat_scales = torch.ops.vllm.fused_quantize_mx(
            x_flat, forward_hadamard_matrix, forward_method
        )
        y = torch.ops.vllm.matmul_mxf4_bf16(
            x_flat_q,
            qweight,
            x_flat_scales,
            weight_scales,
            1 / (weight_global_scale * act_global_scale),
        )
    elif forward_dtype == "nvfp4":
        x_flat_q, x_flat_scales = torch.ops.vllm.fused_quantize_nv(
            x_flat, forward_hadamard_matrix, act_global_scale
        )
        y = torch.ops.vllm.matmul_nvf4_bf16(
            x_flat_q,
            qweight,
            x_flat_scales,
            weight_scales,
            1 / (weight_global_scale * act_global_scale),
        )
    else:
        raise ValueError(f"Unsupported forward_dtype: {forward_dtype}")

    y = y.view(*x.shape[:-1], y.shape[-1])
    if bias is not None:
        y += bias

    return y
```
**EN:** Defines function `quantized_forward` with signature `quantized_forward(x: torch.Tensor, qweight: torch.Tensor, weight_scales: torch.Tensor, weight_global_scale: torch.Tensor, act_global_scale: torch.Tensor, bias: torch.Tensor | None, forward_hadamard_matrix: torch.Tensor, forward_method: str, forward_dtype: str) -> torch.Tensor`. It mainly works with `x`, `qweight`, `weight_scales`, `weight_global_scale`, `act_global_scale`, `bias`, `forward_hadamard_matrix`, `forward_method`; handles quantization-related transformation logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `x.contiguous.flatten`, `y.view`, `torch.ops.vllm.fused_quantize_mx`, `torch.ops.vllm.matmul_mxf4_bf16`, `x.contiguous`, `torch.ops.vllm.fused_quantize_nv`.
**CN:** 定义函数 `quantized_forward`，其签名为 `quantized_forward(x: torch.Tensor, qweight: torch.Tensor, weight_scales: torch.Tensor, weight_global_scale: torch.Tensor, act_global_scale: torch.Tensor, bias: torch.Tensor | None, forward_hadamard_matrix: torch.Tensor, forward_method: str, forward_dtype: str) -> torch.Tensor`。它主要围绕 `x`, `qweight`, `weight_scales`, `weight_global_scale`, `act_global_scale`, `bias`, `forward_hadamard_matrix`, `forward_method` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `x.contiguous.flatten`, `y.view`, `torch.ops.vllm.fused_quantize_mx`, `torch.ops.vllm.matmul_mxf4_bf16`, `x.contiguous`, `torch.ops.vllm.fused_quantize_nv`。

### Class `FPQuantConfig` overview (lines 30-101)
```python
class FPQuantConfig(QuantizationConfig):
    """Config class for FPQuant."""

    def __init__(
        self,
        hadamard_group_size: int = 32,
        forward_dtype: str = "mxfp4",
        forward_method: str = "abs_max",
        pseudoquantization: bool = False,
        modules_to_not_convert: list[str] | None = None,
    ) -> None:
        super().__init__()
        self.hadamard_group_size = hadamard_group_size
        self.forward_dtype = forward_dtype
        self.forward_method = forward_method
        self.pseudoquantization = pseudoquantization
        self.modules_to_not_convert = modules_to_not_convert

        if pseudoquantization:
            raise ValueError("Pseudoquantization is not supported for vLLM")

    def __repr__(self) -> str:
        return (
            f"FPQuantConfig(hadamard_group_size={self.hadamard_group_size}, "
            f"forward_dtype={self.forward_dtype}, "
```
**EN:** Defines class `FPQuantConfig` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 8 direct methods, with notable entries `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`. Its docstring says: Config class for FPQuant.
**CN:** 定义类 `FPQuantConfig`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 8 个方法，较重要的包括 `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`。 文档字符串进一步说明了该类的定位。

### Method `FPQuantConfig.__init__` (lines 33-49)
```python
    def __init__(
        self,
        hadamard_group_size: int = 32,
        forward_dtype: str = "mxfp4",
        forward_method: str = "abs_max",
        pseudoquantization: bool = False,
        modules_to_not_convert: list[str] | None = None,
    ) -> None:
        super().__init__()
        self.hadamard_group_size = hadamard_group_size
        self.forward_dtype = forward_dtype
        self.forward_method = forward_method
        self.pseudoquantization = pseudoquantization
        self.modules_to_not_convert = modules_to_not_convert

        if pseudoquantization:
            raise ValueError("Pseudoquantization is not supported for vLLM")
```
**EN:** Defines function `FPQuantConfig.__init__` with signature `__init__(self, hadamard_group_size: int=32, forward_dtype: str='mxfp4', forward_method: str='abs_max', pseudoquantization: bool=False, modules_to_not_convert: list[str] | None=None) -> None`. It mainly works with `hadamard_group_size`, `forward_dtype`, `forward_method`, `pseudoquantization`, `modules_to_not_convert`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `ValueError`, `super`.
**CN:** 定义函数 `FPQuantConfig.__init__`，其签名为 `__init__(self, hadamard_group_size: int=32, forward_dtype: str='mxfp4', forward_method: str='abs_max', pseudoquantization: bool=False, modules_to_not_convert: list[str] | None=None) -> None`。它主要围绕 `hadamard_group_size`, `forward_dtype`, `forward_method`, `pseudoquantization`, `modules_to_not_convert` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `ValueError`, `super`。

### Method `FPQuantConfig.get_name` (lines 61-62)
```python
    def get_name(cls) -> QuantizationMethods:
        return "fp_quant"
```
**EN:** Defines function `FPQuantConfig.get_name` with signature `get_name(cls) -> QuantizationMethods`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `FPQuantConfig.get_name`，其签名为 `get_name(cls) -> QuantizationMethods`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `FPQuantConfig.get_supported_act_dtypes` (lines 65-66)
```python
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.bfloat16]
```
**EN:** Defines function `FPQuantConfig.get_supported_act_dtypes` with signature `get_supported_act_dtypes(cls) -> list[torch.dtype]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `FPQuantConfig.get_supported_act_dtypes`，其签名为 `get_supported_act_dtypes(cls) -> list[torch.dtype]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `FPQuantConfig.get_min_capability` (lines 69-70)
```python
    def get_min_capability(cls) -> int:
        return 100
```
**EN:** Defines function `FPQuantConfig.get_min_capability` with signature `get_min_capability(cls) -> int`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `FPQuantConfig.get_min_capability`，其签名为 `get_min_capability(cls) -> int`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `FPQuantConfig.from_config` (lines 77-89)
```python
    def from_config(cls, config: dict[str, Any]) -> "FPQuantConfig":
        hadamard_group_size = cls.get_from_keys(config, ["hadamard_group_size"])
        forward_dtype = cls.get_from_keys(config, ["forward_dtype"])
        forward_method = cls.get_from_keys(config, ["forward_method"])
        pseudoquantization = cls.get_from_keys(config, ["pseudoquantization"])
        modules_to_not_convert = cls.get_from_keys(config, ["modules_to_not_convert"])
        return cls(
            hadamard_group_size,
            forward_dtype,
            forward_method,
            pseudoquantization,
            modules_to_not_convert,
        )
```
**EN:** Defines function `FPQuantConfig.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'FPQuantConfig'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `cls.get_from_keys`, `cls`.
**CN:** 定义函数 `FPQuantConfig.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'FPQuantConfig'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls.get_from_keys`, `cls`。

### Method `FPQuantConfig.get_quant_method` (lines 91-101)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> LinearMethodBase | None:
        if self.modules_to_not_convert is not None and any(
            prefix.endswith(module) for module in self.modules_to_not_convert
        ):
            return UnquantizedLinearMethod()

        if isinstance(layer, LinearBase):
            return FPQuantLinearMethod(self)
        return None
```
**EN:** Defines function `FPQuantConfig.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> LinearMethodBase | None`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching, comprehensions. Key calls include `isinstance`, `any`, `UnquantizedLinearMethod`, `FPQuantLinearMethod`, `prefix.endswith`.
**CN:** 定义函数 `FPQuantConfig.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> LinearMethodBase | None`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断、推导式。关键调用包括 `isinstance`, `any`, `UnquantizedLinearMethod`, `FPQuantLinearMethod`, `prefix.endswith`。

### Class `FPQuantLinearMethod` overview (lines 104-248)
```python
class FPQuantLinearMethod(LinearMethodBase):
    """Linear method for FPQuant.

    Args:
        quant_config: The FPQuant quantization config.
    """

    def __init__(self, quant_config: FPQuantConfig):
        self.quant_config = quant_config

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
        del input_size  # Unused.

        if params_dtype != torch.bfloat16:
            raise ValueError("Only bfloat16 is currently supported by FPQuant")
```
**EN:** Defines class `FPQuantLinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 3 direct methods, with notable entries `__init__`, `create_weights`, `apply`. Its docstring says: Linear method for FPQuant.
**CN:** 定义类 `FPQuantLinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 3 个方法，较重要的包括 `__init__`, `create_weights`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `FPQuantLinearMethod.__init__` (lines 111-112)
```python
    def __init__(self, quant_config: FPQuantConfig):
        self.quant_config = quant_config
```
**EN:** Defines function `FPQuantLinearMethod.__init__` with signature `__init__(self, quant_config: FPQuantConfig)`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `FPQuantLinearMethod.__init__`，其签名为 `__init__(self, quant_config: FPQuantConfig)`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `FPQuantLinearMethod.create_weights` (lines 114-230)
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
        del input_size  # Unused.

        if params_dtype != torch.bfloat16:
            raise ValueError("Only bfloat16 is currently supported by FPQuant")
        if input_size_per_partition % self.quant_config.hadamard_group_size != 0:  # noqa: E501
            raise ValueError(
                "The input size is not aligned with the quantized "
                "weight shape. This can be caused by too large "
                "tensor parallel size. Or other skill issues."
            )

        assert self.quant_config.forward_dtype in ["mxfp4", "nvfp4"], (
            "Only mxfp4 and nvfp4 are supported for now"
        )
        if self.quant_config.forward_dtype == "mxfp4":
            group_size = 32
        elif self.quant_config.forward_dtype == "nvfp4":
            group_size = 16
        else:
            raise ValueError(
                f"Unsupported forward_dtype: {self.quant_config.forward_dtype}"
            )

        qweight = Parameter(
            torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition // 2,
# ... truncated for analysis ...
        layer.register_parameter("forward_hadamard_matrix", forward_hadamard_matrix)

        backward_hadamard_matrix = Parameter(
            torch.empty(
                self.quant_config.hadamard_group_size,
                self.quant_config.hadamard_group_size,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        set_weight_attrs(
            backward_hadamard_matrix, {"ignore_warning": True} | extra_weight_attrs
        )
        layer.register_parameter("backward_hadamard_matrix", backward_hadamard_matrix)
```
**EN:** Defines function `FPQuantLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `Parameter`, `set_weight_attrs`, `layer.register_parameter`, `ValueError`, `torch.empty`, `sum`.
**CN:** 定义函数 `FPQuantLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `Parameter`, `set_weight_attrs`, `layer.register_parameter`, `ValueError`, `torch.empty`, `sum`。

### Method `FPQuantLinearMethod.apply` (lines 232-248)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return quantized_forward(
            x,
            layer.qweight,
            layer.scales,
            layer.weight_global_scale,
            layer.act_global_scale,
            bias,
            layer.forward_hadamard_matrix,
            self.quant_config.forward_method,
            self.quant_config.forward_dtype,
        )
```
**EN:** Defines function `FPQuantLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `quantized_forward`.
**CN:** 定义函数 `FPQuantLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `quantized_forward`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `FPQuantConfig`, `FPQuantLinearMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `FPQuantConfig`, `FPQuantLinearMethod`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `fused_quantize_mx`, `fused_quantize_mx_fake`, `matmul_mxf4_bf16`, `matmul_mxf4_bf16_fake`, `fused_quantize_nv` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `fused_quantize_mx`, `fused_quantize_mx_fake`, `matmul_mxf4_bf16`, `matmul_mxf4_bf16_fake`, `fused_quantize_nv` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `torch`
- **Internal / 内部**: `vllm._custom_ops`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.qutlass_utils`, `vllm.model_executor.utils`, `vllm.platforms`, `vllm.utils.torch_utils`
