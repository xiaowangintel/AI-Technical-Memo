# modelslim_w4a4_int4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/modelslim/schemes/modelslim_w4a4_int4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for modelslim w4a4 int4 quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 ModelSlim w4a4 INT4 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: module imports and setup
```python
# Adapted from https://github.com/vllm-project/vllm/tree/main/vllm/model_executor/layers/quantization/compressed_tensors
# SPDX-License-Identifier: Apache-2.0

# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from typing import Any, Dict, List, Optional

import torch

from sglang.srt.hardware_backend.npu.quantization.linear_method_npu import (
    NPU_W4A4DynamicLinearMethod,
)
from sglang.srt.layers.parameter import PerTensorScaleParameter
from sglang.srt.layers.quantization.modelslim.schemes import ModelSlimLinearScheme
from sglang.srt.utils import set_weight_attrs
```
**EN:** This block imports torch, typing, sglang.srt.hardware_backend.npu.quantization.linear_method_npu, sglang.srt.layers.parameter, sglang.srt.layers.quantization.modelslim.schemes, sglang.srt.utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 torch, typing, sglang.srt.hardware_backend.npu.quantization.linear_method_npu, sglang.srt.layers.parameter, sglang.srt.layers.quantization.modelslim.schemes, sglang.srt.utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 17-18: class ModelSlimW4A4Int4: definition
```python
class ModelSlimW4A4Int4(ModelSlimLinearScheme):
```
**EN:** This block declares `ModelSlimW4A4Int4`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_weight, get_perchannel_param, create_weights.
**CN:** 该代码块声明 `ModelSlimW4A4Int4`，它是量化栈中的执行方案类，组织了 __init__, get_weight, get_perchannel_param, create_weights 等行为。

### Lines 19-26: ModelSlimW4A4Int4.__init__()
```python
    def __init__(
        self,
        quant_config: Dict[str, any],
        prefix: str,
    ):
        self.quant_config = quant_config
        self.is_dynamic = self.quant_config[prefix + ".weight"] == "W4A4_DYNAMIC"
        self.kernel = NPU_W4A4DynamicLinearMethod()
```
**EN:** This block defines `ModelSlimW4A4Int4.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 28-33: ModelSlimW4A4Int4.get_weight()
```python
    @staticmethod
    def get_weight(
        input_size: int, output_size: int, params_dtype: torch.dtype
    ) -> Dict[str, Any]:
        params_dict = {"weight": torch.empty(output_size, input_size, dtype=torch.int8)}
        return params_dict
```
**EN:** This block defines `ModelSlimW4A4Int4.get_weight()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4.get_weight()`，用于为调用方获取或计算派生值。

### Lines 35-43: ModelSlimW4A4Int4.get_perchannel_param()
```python
    @staticmethod
    def get_perchannel_param(
        output_size: int,
        params_dtype: torch.dtype,
    ) -> Dict[str, Any]:
        params_dict = {}
        params_dict["weight_scale"] = torch.empty(output_size, 1, dtype=params_dtype)
        params_dict["weight_offset"] = torch.empty(output_size, 1, dtype=params_dtype)
        return params_dict
```
**EN:** This block defines `ModelSlimW4A4Int4.get_perchannel_param()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4.get_perchannel_param()`，用于为调用方获取或计算派生值。

### Lines 45-89: ModelSlimW4A4Int4.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: List[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ) -> None:
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")

        weight_dict = {
            "weight": torch.empty(
                output_size_per_partition, input_size_per_partition, dtype=torch.int8
            )
        }
        for weight_name, weight_param in weight_dict.items():
            param = torch.nn.Parameter(weight_param, requires_grad=False)
            set_weight_attrs(param, {"input_dim": 1, "output_dim": 0})
            layer.register_parameter(weight_name, param)
            set_weight_attrs(param, extra_weight_attrs)

        pertensor_dict = {}
        for pertensor_name, pertensor_param in pertensor_dict.items():
            param = PerTensorScaleParameter(
                data=pertensor_param, weight_loader=weight_loader
            )
            # disable warning
            param.ignore_warning = True
            layer.register_parameter(pertensor_name, param)

        perchannel_dict = {}
        perchannel_dict["weight_scale"] = torch.empty(
            output_size_per_partition, 1, dtype=params_dtype
        )
        perchannel_dict["weight_offset"] = torch.empty(
            output_size_per_partition, 1, dtype=params_dtype
        )
        for perchannel_name, perchannel_param in perchannel_dict.items():
            param = torch.nn.Parameter(perchannel_param, requires_grad=False)
            set_weight_attrs(param, {"output_dim": 0})
            layer.register_parameter(perchannel_name, param)
            set_weight_attrs(param, extra_weight_attrs)
```
**EN:** This block defines `ModelSlimW4A4Int4.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 91-92: ModelSlimW4A4Int4.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer):
        self.kernel.process_weights_after_loading(layer)
```
**EN:** This block defines `ModelSlimW4A4Int4.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 94-100: ModelSlimW4A4Int4.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return self.kernel.apply(layer, x, bias)
```
**EN:** This block defines `ModelSlimW4A4Int4.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4.apply_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `ModelSlimW4A4Int4`: A scheme class that structures file-level quantization behavior. / `ModelSlimW4A4Int4` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `torch`, `typing`
- **Internal / 内部**: `sglang.srt.hardware_backend.npu.quantization.linear_method_npu`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.modelslim.schemes`, `sglang.srt.utils`
