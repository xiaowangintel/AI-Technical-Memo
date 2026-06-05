# linear_method_npu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/quantization/linear_method_npu.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for quantization helpers inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的量化辅助逻辑支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Module setup and shared state / 模块设置与共享状态
```python
from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.hardware_backend.npu.utils import npu_format_cast
from sglang.srt.layers.quantization.base_config import LinearMethodBase

if TYPE_CHECKING:
    from sglang.srt.layers.quantization.base_config import QuantizationConfig
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `torch`, `sglang.srt.hardware_backend.npu.utils`, `sglang.srt.layers.quantization.base_config`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `torch`, `sglang.srt.hardware_backend.npu.utils`, `sglang.srt.layers.quantization.base_config`。

### Lines 12-12: Class `_NPULinearMethodBase` declaration / 类 `_NPULinearMethodBase` 声明
```python
class _NPULinearMethodBase(LinearMethodBase):
```
**EN:** This class establishes `_NPULinearMethodBase` as the main container/coordinator for the surrounding logic. It inherits from `LinearMethodBase`. Its core interface includes methods such as `__init__`.
**CN:** 该类将 `_NPULinearMethodBase` 定义为周边逻辑的主要封装体或协调者。 它继承自 `LinearMethodBase`。 其核心接口包括 `__init__` 等方法。

### Lines 14-18: Method `_NPULinearMethodBase.__init__` / 方法 `_NPULinearMethodBase.__init__`
```python
    def __init__(
        self,
        quant_config: Optional["QuantizationConfig"] = None,
    ):
        self.quant_config = quant_config
```
**EN:** This method implements `__init__` on `_NPULinearMethodBase`. State updates are written into `self.quant_config`.
**CN:** 该方法（属于 `_NPULinearMethodBase`）实现了 `__init__`。 状态更新主要写入 `self.quant_config`。

### Lines 21-21: Class `NPUW8A8Int8LinearMethod` declaration / 类 `NPUW8A8Int8LinearMethod` 声明
```python
class NPUW8A8Int8LinearMethod(_NPULinearMethodBase):
```
**EN:** This class establishes `NPUW8A8Int8LinearMethod` as the main container/coordinator for the surrounding logic. It inherits from `_NPULinearMethodBase`. Its core interface includes methods such as `process_weights_after_loading`, `apply`.
**CN:** 该类将 `NPUW8A8Int8LinearMethod` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_NPULinearMethodBase`。 其核心接口包括 `process_weights_after_loading`, `apply` 等方法。

### Lines 23-44: Method `NPUW8A8Int8LinearMethod.process_weights_after_loading` / 方法 `NPUW8A8Int8LinearMethod.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        layer.weight.data = layer.weight.data.transpose(0, 1).contiguous()
        layer.weight.data = npu_format_cast(layer.weight.data)

        layer.weight_scale.data = layer.weight_scale.data.flatten()
        # Compressed-tensors format doesn't have this field
        if hasattr(layer, "weight_offset"):
            layer.weight_offset.data = layer.weight_offset.data.flatten()

        expanding_factor = layer.weight.data.shape[0]
        layer.aclnn_input_scale = torch.nn.Parameter(
            layer.input_scale.data.repeat(expanding_factor).to(device="npu"),
            requires_grad=False,
        )
        layer.aclnn_input_scale_reciprocal = 1 / torch.nn.Parameter(
            layer.input_scale.data.repeat(expanding_factor).to(device="npu"),
            requires_grad=False,
        )
        layer.aclnn_input_offset = torch.nn.Parameter(
            layer.input_offset.data.repeat(expanding_factor).to(device="npu"),
            requires_grad=False,
        )
```
**EN:** This method implements `process_weights_after_loading` on `NPUW8A8Int8LinearMethod`. It primarily calls `layer.weight.data.transpose.contiguous`, `npu_format_cast`, `layer.weight_scale.data.flatten`, `hasattr`, `torch.nn.Parameter`, `layer.weight_offset.data.flatten` to complete its work. State updates are written into `layer.weight.data`, `layer.weight_scale.data`, `expanding_factor`, `layer.aclnn_input_scale`, `layer.aclnn_input_scale_reciprocal`, `layer.aclnn_input_offset`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW8A8Int8LinearMethod`）实现了 `process_weights_after_loading`。 它主要通过调用 `layer.weight.data.transpose.contiguous`, `npu_format_cast`, `layer.weight_scale.data.flatten`, `hasattr`, `torch.nn.Parameter`, `layer.weight_offset.data.flatten` 来完成任务。 状态更新主要写入 `layer.weight.data`, `layer.weight_scale.data`, `expanding_factor`, `layer.aclnn_input_scale`, `layer.aclnn_input_scale_reciprocal`, `layer.aclnn_input_offset`。 实现中使用了条件分支。

### Lines 46-76: Method `NPUW8A8Int8LinearMethod.apply` / 方法 `NPUW8A8Int8LinearMethod.apply`
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        from sglang.srt.layers.linear import RowParallelLinear

        original_dtype = x.dtype
        if original_dtype != torch.int8:
            x = torch.ops.npu.npu_quantize(
                x,
                layer.aclnn_input_scale_reciprocal,
                layer.aclnn_input_offset,
                torch.qint8,
                -1,
                False,
            )
        # Only fuse bias add into GEMM for rank 0 (this ensures that
        # bias will not get added more than once in Attention TP>1 case)
        if isinstance(layer, RowParallelLinear) and layer.tp_rank > 0:
            quant_bias = None
        else:
            quant_bias = layer.quant_bias
        return torch.ops.npu.npu_quant_matmul(
            x,
            layer.weight,
            layer.deq_scale,
            bias=quant_bias,
            output_dtype=original_dtype,
        )
```
**EN:** This method implements `apply` on `NPUW8A8Int8LinearMethod`. It primarily calls `torch.ops.npu.npu_quant_matmul`, `torch.ops.npu.npu_quantize`, `isinstance` to complete its work. State updates are written into `original_dtype`, `x`, `quant_bias`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW8A8Int8LinearMethod`）实现了 `apply`。 它主要通过调用 `torch.ops.npu.npu_quant_matmul`, `torch.ops.npu.npu_quantize`, `isinstance` 来完成任务。 状态更新主要写入 `original_dtype`, `x`, `quant_bias`。 实现中使用了条件分支。

### Lines 79-79: Class `NPUW8A8Int8DynamicLinearMethod` declaration / 类 `NPUW8A8Int8DynamicLinearMethod` 声明
```python
class NPUW8A8Int8DynamicLinearMethod(_NPULinearMethodBase):
```
**EN:** This class establishes `NPUW8A8Int8DynamicLinearMethod` as the main container/coordinator for the surrounding logic. It inherits from `_NPULinearMethodBase`. Its core interface includes methods such as `process_weights_after_loading`, `apply`.
**CN:** 该类将 `NPUW8A8Int8DynamicLinearMethod` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_NPULinearMethodBase`。 其核心接口包括 `process_weights_after_loading`, `apply` 等方法。

### Lines 81-88: Method `NPUW8A8Int8DynamicLinearMethod.process_weights_after_loading` / 方法 `NPUW8A8Int8DynamicLinearMethod.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        layer.weight.data = layer.weight.data.transpose(0, 1).contiguous()
        layer.weight.data = npu_format_cast(layer.weight.data)

        layer.weight_scale.data = layer.weight_scale.data.flatten()
        # Compressed-tensors format doesn't have this field
        if hasattr(layer, "weight_offset"):
            layer.weight_offset.data = layer.weight_offset.data.flatten()
```
**EN:** This method implements `process_weights_after_loading` on `NPUW8A8Int8DynamicLinearMethod`. It primarily calls `layer.weight.data.transpose.contiguous`, `npu_format_cast`, `layer.weight_scale.data.flatten`, `hasattr`, `layer.weight_offset.data.flatten`, `layer.weight.data.transpose` to complete its work. State updates are written into `layer.weight.data`, `layer.weight_scale.data`, `layer.weight_offset.data`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW8A8Int8DynamicLinearMethod`）实现了 `process_weights_after_loading`。 它主要通过调用 `layer.weight.data.transpose.contiguous`, `npu_format_cast`, `layer.weight_scale.data.flatten`, `hasattr`, `layer.weight_offset.data.flatten`, `layer.weight.data.transpose` 来完成任务。 状态更新主要写入 `layer.weight.data`, `layer.weight_scale.data`, `layer.weight_offset.data`。 实现中使用了条件分支。

### Lines 90-111: Method `NPUW8A8Int8DynamicLinearMethod.apply` / 方法 `NPUW8A8Int8DynamicLinearMethod.apply`
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        if isinstance(x, tuple):
            """dynamic_scale is calculated in malprolog kernel"""
            original_dtype = torch.bfloat16
            quant_out, dynamic_scale = x
        else:
            original_dtype = x.dtype
            quant_out, dynamic_scale = torch.ops.npu.npu_dynamic_quant(x)
        return torch.ops.npu.npu_quant_matmul(
            quant_out,
            layer.weight,
            layer.weight_scale,
            pertoken_scale=dynamic_scale.flatten(),
            bias=bias,
            output_dtype=original_dtype,
        )
```
**EN:** This method implements `apply` on `NPUW8A8Int8DynamicLinearMethod`. It primarily calls `isinstance`, `torch.ops.npu.npu_quant_matmul`, `torch.ops.npu.npu_dynamic_quant`, `dynamic_scale.flatten` to complete its work. State updates are written into `original_dtype`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW8A8Int8DynamicLinearMethod`）实现了 `apply`。 它主要通过调用 `isinstance`, `torch.ops.npu.npu_quant_matmul`, `torch.ops.npu.npu_dynamic_quant`, `dynamic_scale.flatten` 来完成任务。 状态更新主要写入 `original_dtype`。 实现中使用了条件分支。

### Lines 114-114: Class `NPU_W4A4DynamicLinearMethod` declaration / 类 `NPU_W4A4DynamicLinearMethod` 声明
```python
class NPU_W4A4DynamicLinearMethod(_NPULinearMethodBase):
```
**EN:** This class establishes `NPU_W4A4DynamicLinearMethod` as the main container/coordinator for the surrounding logic. It inherits from `_NPULinearMethodBase`. Its core interface includes methods such as `process_weights_after_loading`, `apply`.
**CN:** 该类将 `NPU_W4A4DynamicLinearMethod` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_NPULinearMethodBase`。 其核心接口包括 `process_weights_after_loading`, `apply` 等方法。

### Lines 116-123: Method `NPU_W4A4DynamicLinearMethod.process_weights_after_loading` / 方法 `NPU_W4A4DynamicLinearMethod.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer):
        layer.weight.data = layer.weight.data.transpose(0, 1).contiguous()
        layer.weight_scale.data = layer.weight_scale.data.flatten()
        layer.weight_scale_fp32 = layer.weight_scale.data.to(torch.float32)
        layer.weight_offset.data = layer.weight_offset.data.flatten()
        layer.weight.data = torch.ops.npu.npu_convert_weight_to_int4pack(
            layer.weight.data.to(torch.int32)
        )
```
**EN:** This method implements `process_weights_after_loading` on `NPU_W4A4DynamicLinearMethod`. It primarily calls `layer.weight.data.transpose.contiguous`, `layer.weight_scale.data.flatten`, `layer.weight_scale.data.to`, `layer.weight_offset.data.flatten`, `torch.ops.npu.npu_convert_weight_to_int4pack`, `layer.weight.data.to` to complete its work. State updates are written into `layer.weight.data`, `layer.weight_scale.data`, `layer.weight_scale_fp32`, `layer.weight_offset.data`.
**CN:** 该方法（属于 `NPU_W4A4DynamicLinearMethod`）实现了 `process_weights_after_loading`。 它主要通过调用 `layer.weight.data.transpose.contiguous`, `layer.weight_scale.data.flatten`, `layer.weight_scale.data.to`, `layer.weight_offset.data.flatten`, `torch.ops.npu.npu_convert_weight_to_int4pack`, `layer.weight.data.to` 来完成任务。 状态更新主要写入 `layer.weight.data`, `layer.weight_scale.data`, `layer.weight_scale_fp32`, `layer.weight_offset.data`。

### Lines 125-143: Method `NPU_W4A4DynamicLinearMethod.apply` / 方法 `NPU_W4A4DynamicLinearMethod.apply`
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
        tp_rank: Optional[int] = 0,
    ) -> torch.Tensor:
        original_dtype = x.dtype
        quant_out, dynamic_scale = torch.ops.npu.npu_dynamic_quant(
            x, dst_type=torch.quint4x2
        )
        return torch.ops.npu.npu_quant_matmul(
            quant_out,
            layer.weight,
            layer.weight_scale,
            pertoken_scale=dynamic_scale.flatten(),
            bias=bias,
            output_dtype=original_dtype,
        )
```
**EN:** This method implements `apply` on `NPU_W4A4DynamicLinearMethod`. It primarily calls `torch.ops.npu.npu_dynamic_quant`, `torch.ops.npu.npu_quant_matmul`, `dynamic_scale.flatten` to complete its work. State updates are written into `original_dtype`.
**CN:** 该方法（属于 `NPU_W4A4DynamicLinearMethod`）实现了 `apply`。 它主要通过调用 `torch.ops.npu.npu_dynamic_quant`, `torch.ops.npu.npu_quant_matmul`, `dynamic_scale.flatten` 来完成任务。 状态更新主要写入 `original_dtype`。

## Key Concepts / 关键概念
- **Classes / 类**: `_NPULinearMethodBase`, `NPUW8A8Int8LinearMethod`, `NPUW8A8Int8DynamicLinearMethod`, `NPU_W4A4DynamicLinearMethod`
- **Functions / 函数**: `__init__`, `process_weights_after_loading`, `apply`, `process_weights_after_loading`, `apply`, `process_weights_after_loading`, `apply`
- **Themes / 主题**: `quant`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.npu.utils`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.linear`
- **External / 外部依赖**: `torch`
- **Standard library / 标准库**: `typing`
