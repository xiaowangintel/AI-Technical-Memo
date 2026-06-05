# quark_w4a4_mxfp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/quark/schemes/quark_w4a4_mxfp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for quark w4a4 mxfp4 quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 Quark w4a4 MXFP4 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0

from typing import Any, Callable, Optional

import torch

from sglang.srt.layers.parameter import GroupQuantScaleParameter, PackedvLLMParameter
from sglang.srt.layers.quantization.quark.schemes import QuarkLinearScheme
from sglang.srt.utils import is_hip
```
**EN:** This block imports aiter.ops.triton.gemm.fused.fused_gemm_afp4wfp4_split_cat, aiter.ops.triton.gemm_afp4wfp4, aiter.ops.triton.gemm_afp4wfp4_pre_quant_atomic, aiter.ops.triton.quant, sglang.srt.layers.parameter, sglang.srt.layers.quantization.quark.schemes, sglang.srt.utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 aiter.ops.triton.gemm.fused.fused_gemm_afp4wfp4_split_cat, aiter.ops.triton.gemm_afp4wfp4, aiter.ops.triton.gemm_afp4wfp4_pre_quant_atomic, aiter.ops.triton.quant, sglang.srt.layers.parameter, sglang.srt.layers.quantization.quark.schemes, sglang.srt.utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 11-11: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 12-18: conditional logic for _is_hip
```python
if _is_hip:
    from aiter.ops.triton.gemm.fused.fused_gemm_afp4wfp4_split_cat import (
        fused_gemm_afp4wfp4_split_cat,
    )
    from aiter.ops.triton.gemm_afp4wfp4 import gemm_afp4wfp4
    from aiter.ops.triton.gemm_afp4wfp4_pre_quant_atomic import gemm_afp4wfp4_pre_quant
    from aiter.ops.triton.quant import dynamic_mxfp4_quant
```
**EN:** This block applies conditional logic controlled by `_is_hip`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_hip` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 21-21: initialize __all__
```python
__all__ = ["QuarkW4A4MXFP4"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 23-23: initialize OCP_MX_BLOCK_SIZE
```python
OCP_MX_BLOCK_SIZE = 32
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as OCP_MX_BLOCK_SIZE.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 OCP_MX_BLOCK_SIZE。

### Lines 26-27: class QuarkW4A4MXFP4: definition
```python
class QuarkW4A4MXFP4(QuarkLinearScheme):
```
**EN:** This block declares `QuarkW4A4MXFP4`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, process_weights_after_loading, create_weights.
**CN:** 该代码块声明 `QuarkW4A4MXFP4`，它是量化栈中的执行方案类，组织了 __init__, get_min_capability, process_weights_after_loading, create_weights 等行为。

### Lines 28-34: QuarkW4A4MXFP4.__init__()
```python
    def __init__(
        self, weight_quant_spec: dict[str, Any], input_quant_spec: dict[str, Any]
    ):
        self.out_dtype = torch.get_default_dtype()
        self.qscheme = "per_group"
        self.weight_quant_spec = weight_quant_spec
        self.input_quant_spec = input_quant_spec
```
**EN:** This block defines `QuarkW4A4MXFP4.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuarkW4A4MXFP4.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 36-38: QuarkW4A4MXFP4.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 70
```
**EN:** This block defines `QuarkW4A4MXFP4.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkW4A4MXFP4.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 40-41: QuarkW4A4MXFP4.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        return
```
**EN:** This block defines `QuarkW4A4MXFP4.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `QuarkW4A4MXFP4.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 43-81: QuarkW4A4MXFP4.create_weights()
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

        # WEIGHT
        weight = PackedvLLMParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition // 2,
                dtype=torch.uint8,
            ),
            input_dim=1,
            output_dim=0,
            packed_dim=1,
            packed_factor=2,
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
**EN:** This block defines `QuarkW4A4MXFP4.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `QuarkW4A4MXFP4.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 83-127: QuarkW4A4MXFP4.apply_weights() (part 1/2)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        # This path does not have support for bias currently
        assert bias is None, "bias is not supported"

        three_d = False
        fused_gemm_split_cat = False
        x_s = None
        y = None

        if isinstance(x, tuple):
            assert len(x) in [
                2,
                3,
                5,
            ], "For tuple input, only (x, x_s), (x, x_s, y), or (x, y, S1, S2, out_dtype) formats are accepted"
            if len(x) == 2:
                x, x_s = x
            elif len(x) == 3:
                x, x_s, y = x
            elif len(x) == 5:
                x, y, S1, S2, out_dtype = x
                fused_gemm_split_cat = True

        use_fused_quant_gemm = (
            not fused_gemm_split_cat
            and x_s is None
            and y is not None
            and layer.weight.shape[0] == y.shape[1]
        )

        if x.dim() == 3:
            three_d = True
            x = x.view(-1, x.shape[-1])
            output_shape = [*x.shape[:-1], layer.weight.shape[0]]

        # use_fused_quant_gemm = true, x_q is a bf16/fp16 num
        # x_s is not None = true, x_q is uint8 num
        if use_fused_quant_gemm or x_s is not None:
            x_q = x
        else:
```
**EN:** This segment of `QuarkW4A4MXFP4.apply_weights()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `QuarkW4A4MXFP4.apply_weights()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 128-160: QuarkW4A4MXFP4.apply_weights() (part 2/2)
```python
            x_q, x_s = dynamic_mxfp4_quant(x)

        if y is None:
            y = torch.empty(
                x_q.shape[0],
                layer.weight.shape[0],
                device=x_q.device,
                dtype=self.out_dtype,
            )

        if use_fused_quant_gemm:
            gemm_afp4wfp4_pre_quant(x_q, layer.weight, layer.weight_scale, y.dtype, y)
            y = y.to(x.dtype)
        elif fused_gemm_split_cat:
            k, v = fused_gemm_afp4wfp4_split_cat(
                x=x_q,
                w=layer.weight,
                y=y,
                x_scale=x_s,
                w_scale=layer.weight_scale,
                S1=S1,
                S2=S2,
                dtype=out_dtype,
            )
        else:
            gemm_afp4wfp4(x_q, layer.weight, x_s, layer.weight_scale, self.out_dtype, y)

        if fused_gemm_split_cat:
            return k, v
        elif three_d:
            return y.view(*output_shape)
        else:
            return y
```
**EN:** This segment of `QuarkW4A4MXFP4.apply_weights()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `QuarkW4A4MXFP4.apply_weights()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

## Key Concepts / 关键概念
- `OCP_MX_BLOCK_SIZE`: A module-level constant or registry. / `OCP_MX_BLOCK_SIZE`：模块级常量或注册表。
- `QuarkW4A4MXFP4`: A scheme class that structures file-level quantization behavior. / `QuarkW4A4MXFP4` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `aiter.ops.triton.gemm.fused.fused_gemm_afp4wfp4_split_cat`, `aiter.ops.triton.gemm_afp4wfp4`, `aiter.ops.triton.gemm_afp4wfp4_pre_quant_atomic`, `aiter.ops.triton.quant`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.quark.schemes`, `sglang.srt.utils`
