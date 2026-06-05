# awq_kernels.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/quantization/awq_kernels.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for quantization helpers inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的量化辅助逻辑支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu import (
    NPUW4A16Int4DynamicMoEMethod,
)
from sglang.srt.layers.quantization.utils import replace_parameter

if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import StandardDispatchOutput
    from sglang.srt.layers.quantization.base_config import QuantizationConfig

import torch_npu
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `typing`, `torch`, `sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu`, `sglang.srt.layers.quantization.utils`, `torch_npu`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `typing`, `torch`, `sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu`, `sglang.srt.layers.quantization.utils`, `torch_npu`。

### Lines 19-19: Class `AWQAscendLinearKernel` declaration / 类 `AWQAscendLinearKernel` 声明
```python
class AWQAscendLinearKernel:
```
**EN:** This class establishes `AWQAscendLinearKernel` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `process_weights_after_loading`, `apply`.
**CN:** 该类将 `AWQAscendLinearKernel` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `process_weights_after_loading`, `apply` 等方法。

### Lines 20-21: Method `AWQAscendLinearKernel.__init__` / 方法 `AWQAscendLinearKernel.__init__`
```python
    def __init__(self, quant_config: Optional["QuantizationConfig"] = None):
        self.quant_config = quant_config
```
**EN:** This method implements `__init__` on `AWQAscendLinearKernel`. State updates are written into `self.quant_config`.
**CN:** 该方法（属于 `AWQAscendLinearKernel`）实现了 `__init__`。 状态更新主要写入 `self.quant_config`。

### Lines 23-44: Method `AWQAscendLinearKernel.process_weights_after_loading` / 方法 `AWQAscendLinearKernel.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.scales = torch.nn.Parameter(layer.scales.data, requires_grad=False)
        qweight_tmp = torch.zeros_like(layer.qweight.data)
        qzeros_tmp = layer.qzeros.data
        qzeros_list = []
        shifts = [0, 4, 1, 5, 2, 6, 3, 7]

        for i in range(0, self.quant_config.pack_factor):
            shift_num = shifts[i] * 4
            qzeros_list.append((qzeros_tmp.reshape(-1, 1) >> shift_num) & 0xF)
            qweight_tmp.bitwise_or_(
                ((layer.qweight.data >> shift_num) & 0xF) << (4 * i)
            )

        qweight_tmp.bitwise_xor_(0x88888888)

        qzeros_tmp = torch.cat(qzeros_list, dim=-1).reshape(qzeros_tmp.shape[0], -1)
        qzeros_tmp = -(qzeros_tmp - 8)
        qzeros_tmp = qzeros_tmp.to(layer.scales.data.dtype)

        layer.zeros = torch.nn.Parameter(qzeros_tmp, requires_grad=False)
        layer.weight = torch.nn.Parameter(qweight_tmp, requires_grad=False)
```
**EN:** This method implements `process_weights_after_loading` on `AWQAscendLinearKernel`. It primarily calls `torch.nn.Parameter`, `torch.zeros_like`, `range`, `qweight_tmp.bitwise_xor_`, `torch.cat.reshape`, `qzeros_tmp.to` to complete its work. State updates are written into `layer.scales`, `qweight_tmp`, `qzeros_tmp`, `qzeros_list`, `shifts`, `layer.zeros`. The implementation relies on iteration.
**CN:** 该方法（属于 `AWQAscendLinearKernel`）实现了 `process_weights_after_loading`。 它主要通过调用 `torch.nn.Parameter`, `torch.zeros_like`, `range`, `qweight_tmp.bitwise_xor_`, `torch.cat.reshape`, `qzeros_tmp.to` 来完成任务。 状态更新主要写入 `layer.scales`, `qweight_tmp`, `qzeros_tmp`, `qzeros_list`, `shifts`, `layer.zeros`。 实现中使用了迭代逻辑。

### Lines 46-71: Method `AWQAscendLinearKernel.apply` / 方法 `AWQAscendLinearKernel.apply`
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        qweight = layer.weight
        scales = layer.scales
        qzeros = layer.zeros
        pack_factor = self.quant_config.pack_factor
        out_shape = x.shape[:-1] + (qweight.shape[-1] * pack_factor,)
        reshaped_x = x.reshape(-1, x.shape[-1])

        if bias is not None and bias.dtype == torch.bfloat16:
            bias = bias.float()

        out = torch_npu.npu_weight_quant_batchmatmul(
            reshaped_x,
            qweight,
            antiquant_scale=scales,
            antiquant_offset=qzeros,
            antiquant_group_size=self.quant_config.group_size,
            bias=bias,
        )

        return out.reshape(out_shape)
```
**EN:** This method implements `apply` on `AWQAscendLinearKernel`. It primarily calls `x.reshape`, `torch_npu.npu_weight_quant_batchmatmul`, `out.reshape`, `bias.float` to complete its work. State updates are written into `qweight`, `scales`, `qzeros`, `pack_factor`, `out_shape`, `reshaped_x`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AWQAscendLinearKernel`）实现了 `apply`。 它主要通过调用 `x.reshape`, `torch_npu.npu_weight_quant_batchmatmul`, `out.reshape`, `bias.float` 来完成任务。 状态更新主要写入 `qweight`, `scales`, `qzeros`, `pack_factor`, `out_shape`, `reshaped_x`。 实现中使用了条件分支。

### Lines 74-74: Class `AWQAscendMoEKernel` declaration / 类 `AWQAscendMoEKernel` 声明
```python
class AWQAscendMoEKernel:
```
**EN:** This class establishes `AWQAscendMoEKernel` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `_register_or_replace_parameter`, `_convert_awq_weight_to_npu_layout`, `_convert_awq_qzeros_to_npu_offset`, `process_weights_after_loading`, `apply`.
**CN:** 该类将 `AWQAscendMoEKernel` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `_register_or_replace_parameter`, `_convert_awq_weight_to_npu_layout`, `_convert_awq_qzeros_to_npu_offset`, `process_weights_after_loading`, `apply` 等方法。

### Lines 75-77: Method `AWQAscendMoEKernel.__init__` / 方法 `AWQAscendMoEKernel.__init__`
```python
    def __init__(self, quant_config: Optional["QuantizationConfig"] = None):
        self.quant_config = quant_config
        self.kernel = NPUW4A16Int4DynamicMoEMethod()
```
**EN:** This method implements `__init__` on `AWQAscendMoEKernel`. It primarily calls `NPUW4A16Int4DynamicMoEMethod` to complete its work. State updates are written into `self.quant_config`, `self.kernel`.
**CN:** 该方法（属于 `AWQAscendMoEKernel`）实现了 `__init__`。 它主要通过调用 `NPUW4A16Int4DynamicMoEMethod` 来完成任务。 状态更新主要写入 `self.quant_config`, `self.kernel`。

### Lines 79-88: Method `AWQAscendMoEKernel._register_or_replace_parameter` / 方法 `AWQAscendMoEKernel._register_or_replace_parameter`
```python
    @staticmethod
    def _register_or_replace_parameter(
        layer: torch.nn.Module, name: str, tensor: torch.Tensor
    ) -> None:
        if hasattr(layer, name):
            replace_parameter(layer, name, tensor)
        else:
            layer.register_parameter(
                name, torch.nn.Parameter(tensor, requires_grad=False)
            )
```
**EN:** This method implements `_register_or_replace_parameter` on `AWQAscendMoEKernel`. It primarily calls `hasattr`, `replace_parameter`, `layer.register_parameter`, `torch.nn.Parameter` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AWQAscendMoEKernel`）实现了 `_register_or_replace_parameter`。 它主要通过调用 `hasattr`, `replace_parameter`, `layer.register_parameter`, `torch.nn.Parameter` 来完成任务。 实现中使用了条件分支。

### Lines 90-99: Method `AWQAscendMoEKernel._convert_awq_weight_to_npu_layout` / 方法 `AWQAscendMoEKernel._convert_awq_weight_to_npu_layout`
```python
    def _convert_awq_weight_to_npu_layout(self, qweight: torch.Tensor) -> torch.Tensor:
        num_experts, input_size, _ = qweight.shape
        unpacked_weight = (
            self.kernel._unpack_from_int32(qweight.flatten(0, 1), 4)
            .view(num_experts, input_size, -1)
            .transpose(1, 2)
            .contiguous()
            .int()
        )
        return self.kernel._pack_to_int32(unpacked_weight)
```
**EN:** This method implements `_convert_awq_weight_to_npu_layout` on `AWQAscendMoEKernel`. It primarily calls `self.kernel._unpack_from_int32.view.transpose.contiguous.int`, `self.kernel._pack_to_int32`, `self.kernel._unpack_from_int32.view.transpose.contiguous`, `self.kernel._unpack_from_int32.view.transpose`, `self.kernel._unpack_from_int32.view`, `self.kernel._unpack_from_int32` to complete its work. State updates are written into `unpacked_weight`.
**CN:** 该方法（属于 `AWQAscendMoEKernel`）实现了 `_convert_awq_weight_to_npu_layout`。 它主要通过调用 `self.kernel._unpack_from_int32.view.transpose.contiguous.int`, `self.kernel._pack_to_int32`, `self.kernel._unpack_from_int32.view.transpose.contiguous`, `self.kernel._unpack_from_int32.view.transpose`, `self.kernel._unpack_from_int32.view`, `self.kernel._unpack_from_int32` 来完成任务。 状态更新主要写入 `unpacked_weight`。

### Lines 101-111: Method `AWQAscendMoEKernel._convert_awq_qzeros_to_npu_offset` / 方法 `AWQAscendMoEKernel._convert_awq_qzeros_to_npu_offset`
```python
    def _convert_awq_qzeros_to_npu_offset(
        self, qzeros: torch.Tensor, dtype: torch.dtype
    ) -> torch.Tensor:
        num_experts, num_groups, _ = qzeros.shape
        offset = (
            -self.kernel._unpack_from_int32(qzeros.flatten(0, 1), 4)
            .view(num_experts, num_groups, -1)
            .transpose(1, 2)
            .contiguous()
        )
        return offset.to(dtype)
```
**EN:** This method implements `_convert_awq_qzeros_to_npu_offset` on `AWQAscendMoEKernel`. It primarily calls `offset.to`, `self.kernel._unpack_from_int32.view.transpose.contiguous`, `self.kernel._unpack_from_int32.view.transpose`, `self.kernel._unpack_from_int32.view`, `self.kernel._unpack_from_int32`, `qzeros.flatten` to complete its work. State updates are written into `offset`.
**CN:** 该方法（属于 `AWQAscendMoEKernel`）实现了 `_convert_awq_qzeros_to_npu_offset`。 它主要通过调用 `offset.to`, `self.kernel._unpack_from_int32.view.transpose.contiguous`, `self.kernel._unpack_from_int32.view.transpose`, `self.kernel._unpack_from_int32.view`, `self.kernel._unpack_from_int32`, `qzeros.flatten` 来完成任务。 状态更新主要写入 `offset`。

### Lines 113-149: Method `AWQAscendMoEKernel.process_weights_after_loading` / 方法 `AWQAscendMoEKernel.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        self._register_or_replace_parameter(
            layer,
            "w13_weight",
            self._convert_awq_weight_to_npu_layout(layer.w13_qweight.data),
        )
        self._register_or_replace_parameter(
            layer,
            "w2_weight",
            self._convert_awq_weight_to_npu_layout(layer.w2_qweight.data),
        )
        self._register_or_replace_parameter(
            layer,
            "w13_weight_scale",
            layer.w13_scales.data.transpose(1, 2).contiguous(),
        )
        self._register_or_replace_parameter(
            layer,
            "w2_weight_scale",
            layer.w2_scales.data.transpose(1, 2).contiguous(),
        )
        self._register_or_replace_parameter(
            layer,
            "w13_weight_offset",
            self._convert_awq_qzeros_to_npu_offset(
                layer.w13_qzeros.data, layer.w13_scales.data.dtype
            ),
        )
# ... omitted for brevity ...
            self._convert_awq_qzeros_to_npu_offset(
                layer.w2_qzeros.data, layer.w2_scales.data.dtype
            ),
        )

        self.kernel.process_weights_after_loading(layer)
```
**EN:** This method implements `process_weights_after_loading` on `AWQAscendMoEKernel`. It primarily calls `self._register_or_replace_parameter`, `self.kernel.process_weights_after_loading`, `self._convert_awq_weight_to_npu_layout`, `layer.w13_scales.data.transpose.contiguous`, `layer.w2_scales.data.transpose.contiguous`, `self._convert_awq_qzeros_to_npu_offset` to complete its work.
**CN:** 该方法（属于 `AWQAscendMoEKernel`）实现了 `process_weights_after_loading`。 它主要通过调用 `self._register_or_replace_parameter`, `self.kernel.process_weights_after_loading`, `self._convert_awq_weight_to_npu_layout`, `layer.w13_scales.data.transpose.contiguous`, `layer.w2_scales.data.transpose.contiguous`, `self._convert_awq_qzeros_to_npu_offset` 来完成任务。

### Lines 151-156: Method `AWQAscendMoEKernel.apply` / 方法 `AWQAscendMoEKernel.apply`
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: "StandardDispatchOutput",
    ) -> torch.Tensor:
        return self.kernel.apply(layer, dispatch_output)
```
**EN:** This method implements `apply` on `AWQAscendMoEKernel`. It primarily calls `self.kernel.apply` to complete its work.
**CN:** 该方法（属于 `AWQAscendMoEKernel`）实现了 `apply`。 它主要通过调用 `self.kernel.apply` 来完成任务。

### Lines 158-174: Method `AWQAscendMoEKernel.apply_without_routing_weights` / 方法 `AWQAscendMoEKernel.apply_without_routing_weights`
```python
    def apply_without_routing_weights(
        self,
        layer,
        hidden_states,
        hidden_states_scale,
        group_list_type,
        group_list,
        output_dtype,
    ):
        return self.kernel.apply_without_routing_weights(
            layer,
            hidden_states,
            hidden_states_scale,
            group_list_type,
            group_list,
            output_dtype,
        )
```
**EN:** This method implements `apply_without_routing_weights` on `AWQAscendMoEKernel`. It primarily calls `self.kernel.apply_without_routing_weights` to complete its work.
**CN:** 该方法（属于 `AWQAscendMoEKernel`）实现了 `apply_without_routing_weights`。 它主要通过调用 `self.kernel.apply_without_routing_weights` 来完成任务。

## Key Concepts / 关键概念
- **Classes / 类**: `AWQAscendLinearKernel`, `AWQAscendMoEKernel`
- **Functions / 函数**: `__init__`, `process_weights_after_loading`, `apply`, `__init__`, `_register_or_replace_parameter`, `_convert_awq_weight_to_npu_layout`, `_convert_awq_qzeros_to_npu_offset`, `process_weights_after_loading`
- **Themes / 主题**: `quant`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu`, `sglang.srt.layers.quantization.utils`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.base_config`
- **External / 外部依赖**: `torch`, `torch_npu`
- **Standard library / 标准库**: `__future__`, `typing`
