# fpgemm_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/fpgemm_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for fpgemm fp8 quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 fpgemm FP8 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
from __future__ import annotations

import logging
from typing import Any, List, Optional

import torch
from torch.nn import Module
from torch.nn.parameter import Parameter

from sglang.srt.layers.linear import LinearBase
from sglang.srt.layers.parameter import ChannelQuantScaleParameter, ModelWeightParameter
from sglang.srt.layers.quantization.base_config import (
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz
from sglang.srt.layers.quantization.fp8_utils import (
    apply_fp8_linear,
    can_auto_enable_marlin_fp8,
    cutlass_fp8_supported,
    normalize_e4m3fn_to_e4m3fnuz,
)
from sglang.srt.layers.quantization.marlin_utils_fp8 import (
    apply_fp8_marlin_linear,
    prepare_fp8_layer_for_marlin,
)
from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
from sglang.srt.layers.quantization.utils import is_layer_skipped
from sglang.srt.utils import get_bool_env_var, is_cuda
```
**EN:** This block imports __future__, logging, torch, torch.nn, sglang.srt.layers.linear, sglang.srt.layers.parameter, sglang.srt.layers.quantization.base_config, sglang.srt.layers.quantization.fp8_kernel and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, logging, torch, torch.nn, sglang.srt.layers.linear, sglang.srt.layers.parameter, sglang.srt.layers.quantization.base_config, sglang.srt.layers.quantization.fp8_kernel 等依赖，并为当前量化实现准备模块命名空间。

### Lines 33-33: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 34-34: initialize _is_fp8_fnuz
```python
_is_fp8_fnuz = is_fp8_fnuz()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_fp8_fnuz.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_fp8_fnuz。

### Lines 36-36: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 39-41: class FBGEMMFp8Config: definition
```python
class FBGEMMFp8Config(QuantizationConfig):
    """Config class for FBGEMM Fp8."""
```
**EN:** This block declares `FBGEMMFp8Config`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_name, get_supported_act_dtypes, get_min_capability.
**CN:** 该代码块声明 `FBGEMMFp8Config`，它是量化栈中的配置类，组织了 __init__, get_name, get_supported_act_dtypes, get_min_capability 等行为。

### Lines 42-54: FBGEMMFp8Config.__init__()
```python
    def __init__(self, ignore_list: list[str], input_scale_ub: float):
        super().__init__()
        self.ignore_list = ignore_list if ignore_list else []
        self.input_scale_ub = input_scale_ub

        # For GPUs that lack FP8 hardware suspport, we can leverage the Marlin
        # kernel for fast weight-only FP8 quantization
        # self.use_marlin = not marlin_fp8_supported()
        self.use_marlin = False
        if _is_cuda:
            force_marlin = get_bool_env_var("SGLANG_FORCE_FP8_MARLIN")
            auto_enable = can_auto_enable_marlin_fp8()
            self.use_marlin = force_marlin or auto_enable
```
**EN:** This block defines `FBGEMMFp8Config.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `FBGEMMFp8Config.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 56-58: FBGEMMFp8Config.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "fbgemm_fp8"
```
**EN:** This block defines `FBGEMMFp8Config.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `FBGEMMFp8Config.get_name()`，用于为调用方获取或计算派生值。

### Lines 60-62: FBGEMMFp8Config.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.bfloat16, torch.float16]
```
**EN:** This block defines `FBGEMMFp8Config.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `FBGEMMFp8Config.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 64-66: FBGEMMFp8Config.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 80
```
**EN:** This block defines `FBGEMMFp8Config.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `FBGEMMFp8Config.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 68-70: FBGEMMFp8Config.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> list[str]:
        return []
```
**EN:** This block defines `FBGEMMFp8Config.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `FBGEMMFp8Config.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 72-76: FBGEMMFp8Config.from_config()
```python
    @classmethod
    def from_config(cls, config: dict[str, Any]) -> FBGEMMFp8Config:
        ignore_list = cls.get_from_keys(config, ["modules_to_not_convert"])
        input_scale_ub = cls.get_from_keys(config, ["activation_scale_ub"])
        return cls(ignore_list=ignore_list, input_scale_ub=input_scale_ub)
```
**EN:** This block defines `FBGEMMFp8Config.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `FBGEMMFp8Config.from_config()`，用于从序列化配置数据构造对象。

### Lines 78-89: FBGEMMFp8Config.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        if isinstance(layer, LinearBase):
            if is_layer_skipped(
                prefix=prefix,
                ignored_layers=self.ignore_list,
                fused_mapping=self.packed_modules_mapping,
            ):
                return UnquantizedLinearMethod()
            return FBGEMMFp8LinearMethod(self)
        return None
```
**EN:** This block defines `FBGEMMFp8Config.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `FBGEMMFp8Config.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 91-92: FBGEMMFp8Config.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `FBGEMMFp8Config.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `FBGEMMFp8Config.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 95-96: class FBGEMMFp8LinearMethod: definition
```python
class FBGEMMFp8LinearMethod(LinearMethodBase):
```
**EN:** This block declares `FBGEMMFp8LinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `FBGEMMFp8LinearMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, apply 等行为。

### Lines 97-102: FBGEMMFp8LinearMethod.__init__()
```python
    def __init__(self, quant_config: FBGEMMFp8Config):
        self.quant_config = quant_config
        # self.fp8_linear = Fp8LinearOp(
        #     act_quant_static=False, act_quant_group_shape=GroupShape.PER_TOKEN)
        self.out_dtype = torch.get_default_dtype()
        self.cutlass_fp8_supported = cutlass_fp8_supported()
```
**EN:** This block defines `FBGEMMFp8LinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `FBGEMMFp8LinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 104-152: FBGEMMFp8LinearMethod.create_weights()
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
        # maybe_create_device_identity()
        weight_loader = extra_weight_attrs.get("weight_loader")
        del input_size, output_size
        output_size_per_partition = sum(output_partition_sizes)

        layer.logical_widths = output_partition_sizes

        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.orig_dtype = params_dtype

        # WEIGHT
        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition,
                dtype=torch.float8_e4m3fn,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        # WEIGHT SCALE
        weight_scale = ChannelQuantScaleParameter(
            data=torch.empty((sum(output_partition_sizes), 1), dtype=torch.float32),
            output_dim=0,
            weight_loader=weight_loader,
        )
        weight_scale[:] = torch.finfo(torch.float32).min
        layer.register_parameter("weight_scale", weight_scale)

        # INPUT SCALE UPPER BOUND
        input_scale_ub = torch.nn.Parameter(
            torch.tensor((self.quant_config.input_scale_ub), dtype=torch.float32),
            requires_grad=False,
        )
        layer.input_scale_ub = input_scale_ub
```
**EN:** This block defines `FBGEMMFp8LinearMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `FBGEMMFp8LinearMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 154-173: FBGEMMFp8LinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        # required by torch.compile
        layer.weight_scale = Parameter(layer.weight_scale.data, requires_grad=False)
        layer.weight = Parameter(layer.weight.data, requires_grad=False)

        weight = layer.weight

        if _is_fp8_fnuz:
            weight, weight_scale, input_scale = normalize_e4m3fn_to_e4m3fnuz(
                weight=weight, weight_scale=layer.weight_scale, input_scale=None
            )
            if input_scale is not None:
                layer.input_scale = Parameter(input_scale, requires_grad=False)
            layer.weight_scale = Parameter(weight_scale, requires_grad=False)

        layer.weight = Parameter(weight.t(), requires_grad=False)
        if self.quant_config.use_marlin:
            prepare_fp8_layer_for_marlin(layer)
            # Activations not quantized for marlin.
            del layer.input_scale_ub
```
**EN:** This block defines `FBGEMMFp8LinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `FBGEMMFp8LinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 175-202: FBGEMMFp8LinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        if self.quant_config.use_marlin:
            return apply_fp8_marlin_linear(
                input=x,
                weight=layer.weight,
                weight_scale=layer.weight_scale,
                workspace=layer.workspace,
                size_n=layer.output_size_per_partition,
                size_k=layer.input_size_per_partition,
                bias=bias,
            )

        return apply_fp8_linear(
            input=x,
            weight=layer.weight,
            weight_scale=layer.weight_scale,
            input_scale=None,
            input_scale_ub=layer.input_scale_ub,
            bias=bias,
            cutlass_fp8_supported=self.cutlass_fp8_supported,
            use_per_token_if_dynamic=False,
        )
```
**EN:** This block defines `FBGEMMFp8LinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `FBGEMMFp8LinearMethod.apply()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `FBGEMMFp8Config`: A configuration class that structures file-level quantization behavior. / `FBGEMMFp8Config` 是一个配置类，用于组织该文件中的量化行为。
- `FBGEMMFp8LinearMethod`: A runtime method class that structures file-level quantization behavior. / `FBGEMMFp8LinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `torch`, `torch.nn`, `torch.nn.parameter`, `typing`
- **Internal / 内部**: `sglang.srt.layers.linear`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.quantization.marlin_utils_fp8`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
