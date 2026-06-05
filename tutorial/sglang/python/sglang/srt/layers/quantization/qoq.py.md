# qoq.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/qoq.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for qoq quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 QoQ 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: module imports and setup
```python
from __future__ import annotations

from typing import Any, Dict, List, Optional

import torch
from torch.nn.parameter import Parameter

from sglang.srt.layers.parameter import (
    ChannelQuantScaleParameter,
    GroupQuantScaleParameter,
    ModelWeightParameter,
)
from sglang.srt.layers.quantization.base_config import (
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.int8_kernel import per_token_quant_int8
from sglang.srt.utils import is_cuda
```
**EN:** This block imports __future__, sgl_kernel, torch, torch.nn.parameter, sglang.srt.layers.linear, sglang.srt.layers.parameter, sglang.srt.layers.quantization.base_config, sglang.srt.layers.quantization.int8_kernel and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, sgl_kernel, torch, torch.nn.parameter, sglang.srt.layers.linear, sglang.srt.layers.parameter, sglang.srt.layers.quantization.base_config, sglang.srt.layers.quantization.int8_kernel 等依赖，并为当前量化实现准备模块命名空间。

### Lines 21-21: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 22-23: conditional logic for _is_cuda
```python
if _is_cuda:
    from sgl_kernel import qserve_w4a8_per_chn_gemm, qserve_w4a8_per_group_gemm
```
**EN:** This block applies conditional logic controlled by `_is_cuda`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_cuda` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 26-26: initialize QoQ_SUPPORTED_WEIGHT_BITS
```python
QoQ_SUPPORTED_WEIGHT_BITS = [4]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as QoQ_SUPPORTED_WEIGHT_BITS.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 QoQ_SUPPORTED_WEIGHT_BITS。

### Lines 27-27: initialize QoQ_SUPPORTED_GROUP_SIZES
```python
QoQ_SUPPORTED_GROUP_SIZES = [-1, 128]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as QoQ_SUPPORTED_GROUP_SIZES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 QoQ_SUPPORTED_GROUP_SIZES。

### Lines 30-39: class QoQConfig: definition
```python
class QoQConfig(QuantizationConfig):
    """Config class for QoQ Quantization.

    - Weight: static, per-channel/group, asymmetric
    - Activation: dynamic, per-token, symmetric

    Reference: https://arxiv.org/abs/2405.04532
    https://github.com/mit-han-lab/omniserve
    """
```
**EN:** This block declares `QoQConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, __repr__, get_supported_act_dtypes, get_min_capability.
**CN:** 该代码块声明 `QoQConfig`，它是量化栈中的配置类，组织了 __init__, __repr__, get_supported_act_dtypes, get_min_capability 等行为。

### Lines 40-59: QoQConfig.__init__()
```python
    def __init__(self, weight_bits: int, group_size: int) -> None:
        self.weight_bits = weight_bits
        self.group_size = group_size

        # Verify
        if self.weight_bits not in QoQ_SUPPORTED_WEIGHT_BITS:
            raise ValueError(
                f"QoQ does not support weight_bits = {self.weight_bits}. "
                f"Only weight_bits = {QoQ_SUPPORTED_WEIGHT_BITS} "
                "are supported."
            )
        if self.group_size not in QoQ_SUPPORTED_GROUP_SIZES:
            raise ValueError(
                f"QoQ does not support group_size = {self.group_size}. "
                f"Only group_sizes = {QoQ_SUPPORTED_GROUP_SIZES} "
                "are supported."
            )

        # 4 bits packed into 8 bit datatype.
        self.pack_factor = 8 // self.weight_bits
```
**EN:** This block defines `QoQConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QoQConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 61-64: QoQConfig.__repr__()
```python
    def __repr__(self) -> str:
        return "QoQConfig(weight_bits={}, group_size={})".format(
            self.weight_bits, self.group_size
        )
```
**EN:** This block defines `QoQConfig.__repr__()`, which builds a readable debug representation.
**CN:** 该代码块定义了 `QoQConfig.__repr__()`，用于构造可读的调试表示。

### Lines 66-68: QoQConfig.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.float16]
```
**EN:** This block defines `QoQConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QoQConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 70-72: QoQConfig.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 80
```
**EN:** This block defines `QoQConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QoQConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 74-76: QoQConfig.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "qoq"
```
**EN:** This block defines `QoQConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QoQConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 78-84: QoQConfig.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        """List of filenames to search for in the model directory."""
        return [
            "quant_config.json",
            "quantize_config.json",
        ]
```
**EN:** This block defines `QoQConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QoQConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 86-90: QoQConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> QoQConfig:
        weight_bits = cls.get_from_keys(config, ["wbits"])
        group_size = cls.get_from_keys(config, ["group_size"])
        return cls(weight_bits, group_size)
```
**EN:** This block defines `QoQConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `QoQConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 92-101: QoQConfig.get_quant_method()
```python
    def get_quant_method(
        self,
        layer: torch.nn.Module,
        prefix: str,
    ) -> Optional[QuantizeMethodBase]:
        from sglang.srt.layers.linear import LinearBase

        if isinstance(layer, LinearBase):
            return QoQLinearMethod(self)
        return None
```
**EN:** This block defines `QoQConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QoQConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 103-104: QoQConfig.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `QoQConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QoQConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 107-113: class QoQLinearMethod: definition
```python
class QoQLinearMethod(LinearMethodBase):
    """Linear method for QoQ.

    Args:
        quant_config: The QoQ quantization config.
    """
```
**EN:** This block declares `QoQLinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `QoQLinearMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, apply 等行为。

### Lines 114-115: QoQLinearMethod.__init__()
```python
    def __init__(self, quant_config: QoQConfig):
        self.quant_config = quant_config
```
**EN:** This block defines `QoQLinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QoQLinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 117-161: QoQLinearMethod.create_weights() (part 1/3)
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
    ):

        weight_loader = extra_weight_attrs.get("weight_loader")

        # Validate output_size_per_partition
        output_size_per_partition = sum(output_partition_sizes)
        if output_size_per_partition % 32 != 0:
            raise ValueError(
                f"Weight output_size_per_partition = "
                f"{output_size_per_partition} is not divisible by 32."
            )

        # Validate input_size_per_partition
        if input_size_per_partition % self.quant_config.pack_factor != 0:
            raise ValueError(
                f"Weight input_size_per_partition = "
                f"{input_size_per_partition} is not divisible by "
                f"pack_factor = {self.quant_config.pack_factor}."
            )
        if (
            self.quant_config.group_size != -1
            and input_size_per_partition % self.quant_config.group_size != 0
        ):
            raise ValueError(
                f"Weight input_size_per_partition = "
                f"{input_size_per_partition} is not divisible by "
                f"group_size = {self.quant_config.group_size}."
            )

        qweight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int8,
            ),
            input_dim=1,
```
**EN:** This segment of `QoQLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QoQLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 162-206: QoQLinearMethod.create_weights() (part 2/3)
```python
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("qweight", qweight)

        s1_scales = ChannelQuantScaleParameter(
            data=torch.empty(output_size_per_partition, dtype=torch.float16),
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("s1_scales", s1_scales)

        if self.quant_config.group_size == -1:
            s1_szeros = ChannelQuantScaleParameter(
                data=torch.empty(output_size_per_partition, dtype=torch.float16),
                output_dim=0,
                weight_loader=weight_loader,
            )
            layer.register_parameter("s1_szeros", s1_szeros)
        else:
            s2_scales = GroupQuantScaleParameter(
                data=torch.empty(
                    (
                        input_size_per_partition // self.quant_config.group_size,
                        output_size_per_partition,
                    ),
                    dtype=torch.int8,
                ),
                input_dim=0,
                output_dim=1,
                weight_loader=weight_loader,
            )
            layer.register_parameter("s2_scales", s2_scales)

            s2_zeros = GroupQuantScaleParameter(
                data=torch.empty(
                    (
                        input_size_per_partition // self.quant_config.group_size,
                        output_size_per_partition,
                    ),
                    dtype=torch.int8,
                ),
                input_dim=0,
                output_dim=1,
                weight_loader=weight_loader,
```
**EN:** This segment of `QoQLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QoQLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 207-208: QoQLinearMethod.create_weights() (part 3/3)
```python
            )
            layer.register_parameter("s2_zeros", s2_zeros)
```
**EN:** This segment of `QoQLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QoQLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 210-217: QoQLinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.qweight = Parameter(layer.qweight.data, requires_grad=False)
        layer.s1_scales = Parameter(layer.s1_scales.data, requires_grad=False)
        if self.quant_config.group_size == -1:
            layer.s1_szeros = Parameter(layer.s1_szeros.data, requires_grad=False)
        else:
            layer.s2_scales = Parameter(layer.s2_scales.data, requires_grad=False)
            layer.s2_zeros = Parameter(layer.s2_zeros.data, requires_grad=False)
```
**EN:** This block defines `QoQLinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `QoQLinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 219-245: QoQLinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ):
        assert x.dtype == torch.float16, "QoQ only supports float16 input now"
        if self.quant_config.group_size == -1:
            x_q, x_scale, x_sum = per_token_quant_int8(
                x, scale_dtype=x.dtype, cal_sum=True
            )
            out = qserve_w4a8_per_chn_gemm(
                x_q, layer.qweight, layer.s1_scales, x_scale, layer.s1_szeros, x_sum
            )
        else:
            x_q, x_scale = per_token_quant_int8(x, scale_dtype=x.dtype)
            out = qserve_w4a8_per_group_gemm(
                x_q,
                layer.qweight,
                layer.s2_zeros,
                layer.s2_scales,
                layer.s1_scales,
                x_scale,
            )
        if bias is not None:
            out = out + bias
        return out
```
**EN:** This block defines `QoQLinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `QoQLinearMethod.apply()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `QoQConfig`: A configuration class that structures file-level quantization behavior. / `QoQConfig` 是一个配置类，用于组织该文件中的量化行为。
- `QoQLinearMethod`: A runtime method class that structures file-level quantization behavior. / `QoQLinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `sgl_kernel`, `torch`, `torch.nn.parameter`, `typing`
- **Internal / 内部**: `sglang.srt.layers.linear`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.int8_kernel`, `sglang.srt.utils`
