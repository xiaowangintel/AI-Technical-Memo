# awq_cpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/awq/schemes/awq_cpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements scheme classes that describe how awq cpu weights are packed, loaded, and executed. / 该模块实现了执行方案类，用于描述 AWQ CPU 权重如何打包、加载与执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
from __future__ import annotations

from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.layers.amx_utils import (
    CPUQuantMethod,
    _amx_process_weight_after_loading,
)
from sglang.srt.layers.moe import MoeRunnerConfig

from .awq_linear import AWQLinearScheme
from .awq_moe import AWQMoEScheme
```
**EN:** This block imports __future__, torch, typing, .awq_linear, .awq_moe, sglang.srt.layers.amx_utils, sglang.srt.layers.moe and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, torch, typing, .awq_linear, .awq_moe, sglang.srt.layers.amx_utils, sglang.srt.layers.moe 等依赖，并为当前量化实现准备模块命名空间。

### Lines 17-19: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import StandardDispatchOutput
    from sglang.srt.layers.quantization.awq.awq import AWQConfig
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 21-21: initialize __all__
```python
__all__ = ["AWQIntelAMXLinearScheme", "AWQIntelAMXMoEScheme"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 24-24: class AWQIntelAMXLinearKernel: definition
```python
class AWQIntelAMXLinearKernel:
```
**EN:** This block declares `AWQIntelAMXLinearKernel`, a supporting class for the quantization stack. It organizes behaviors such as __init__, process_weights_after_loading, apply.
**CN:** 该代码块声明 `AWQIntelAMXLinearKernel`，它是量化栈中的支撑类，组织了 __init__, process_weights_after_loading, apply 等行为。

### Lines 25-26: AWQIntelAMXLinearKernel.__init__()
```python
    def __init__(self, quant_config: "AWQConfig"):
        self.quant_config = quant_config
```
**EN:** This block defines `AWQIntelAMXLinearKernel.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `AWQIntelAMXLinearKernel.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 28-34: AWQIntelAMXLinearKernel.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        _amx_process_weight_after_loading(
            layer, ["qweight", "qzeros", "scales"], None, "awq"
        )
        layer.qweight = torch.nn.Parameter(layer.qweight.data, requires_grad=False)
        layer.qzeros = torch.nn.Parameter(layer.qzeros.data, requires_grad=False)
        layer.scales = torch.nn.Parameter(layer.scales.data, requires_grad=False)
```
**EN:** This block defines `AWQIntelAMXLinearKernel.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `AWQIntelAMXLinearKernel.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 36-48: AWQIntelAMXLinearKernel.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return torch.ops.sgl_kernel.int4_scaled_mm_cpu(
            x,
            layer.qweight,
            layer.qzeros,
            layer.scales,
            bias,
        )
```
**EN:** This block defines `AWQIntelAMXLinearKernel.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `AWQIntelAMXLinearKernel.apply()`，用于将量化计算应用到运行时输入上。

### Lines 51-53: class AWQIntelAMXLinearScheme: definition
```python
class AWQIntelAMXLinearScheme(AWQLinearScheme):
    """Linear scheme for AWQ on Intel CPU with AMX."""
```
**EN:** This block declares `AWQIntelAMXLinearScheme`, a scheme class for the quantization stack. It organizes behaviors such as _init_kernel.
**CN:** 该代码块声明 `AWQIntelAMXLinearScheme`，它是量化栈中的执行方案类，组织了 _init_kernel 等行为。

### Lines 54-55: AWQIntelAMXLinearScheme._init_kernel()
```python
    def _init_kernel(self, quant_config: "AWQConfig"):
        return AWQIntelAMXLinearKernel(quant_config)
```
**EN:** This block defines `AWQIntelAMXLinearScheme._init_kernel()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `AWQIntelAMXLinearScheme._init_kernel()`，用于实现量化栈中的可复用模块逻辑。

### Lines 58-58: class AWQIntelAMXMoEKernel: definition
```python
class AWQIntelAMXMoEKernel:
```
**EN:** This block declares `AWQIntelAMXMoEKernel`, a supporting class for the quantization stack. It organizes behaviors such as __init__, process_weights_after_loading, create_moe_runner, apply.
**CN:** 该代码块声明 `AWQIntelAMXMoEKernel`，它是量化栈中的支撑类，组织了 __init__, process_weights_after_loading, create_moe_runner, apply 等行为。

### Lines 59-60: AWQIntelAMXMoEKernel.__init__()
```python
    def __init__(self, quant_config: "AWQConfig"):
        self.quant_config = quant_config
```
**EN:** This block defines `AWQIntelAMXMoEKernel.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `AWQIntelAMXMoEKernel.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 62-68: AWQIntelAMXMoEKernel.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        _amx_process_weight_after_loading(
            layer, ["w13_qweight", "w13_qzeros", "w13_scales"], None, "awq"
        )
        _amx_process_weight_after_loading(
            layer, ["w2_qweight", "w2_qzeros", "w2_scales"], None, "awq"
        )
```
**EN:** This block defines `AWQIntelAMXMoEKernel.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `AWQIntelAMXMoEKernel.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 70-73: AWQIntelAMXMoEKernel.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `AWQIntelAMXMoEKernel.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `AWQIntelAMXMoEKernel.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 75-104: AWQIntelAMXMoEKernel.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: "StandardDispatchOutput",
    ) -> torch.Tensor:
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        assert (
            self.moe_runner_config.activation == "silu"
        ), "Only SiLU activation is supported."

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output
        topk_weights, topk_ids, _ = topk_output
        output = torch.ops.sgl_kernel.fused_experts_cpu(
            x,
            layer.w13_qweight,
            layer.w2_qweight,
            topk_weights,
            topk_ids,
            False,  # inplace See [Note] inplace should be False in fused_experts.
            CPUQuantMethod.INT4_W4A8,
            layer.w13_scales,  # w1_scale
            layer.w2_scales,  # w2_scale
            layer.w13_qzeros,
            layer.w2_qzeros,
            None,  # block_size
            True,  # is_vnni
        )
        return StandardCombineInput(hidden_states=output)
```
**EN:** This block defines `AWQIntelAMXMoEKernel.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `AWQIntelAMXMoEKernel.apply()`，用于将量化计算应用到运行时输入上。

### Lines 107-109: class AWQIntelAMXMoEScheme: definition
```python
class AWQIntelAMXMoEScheme(AWQMoEScheme):
    """MoE scheme for AWQ on Intel CPU with AMX."""
```
**EN:** This block declares `AWQIntelAMXMoEScheme`, a scheme class for the quantization stack. It organizes behaviors such as _init_kernel, create_moe_runner.
**CN:** 该代码块声明 `AWQIntelAMXMoEScheme`，它是量化栈中的执行方案类，组织了 _init_kernel, create_moe_runner 等行为。

### Lines 110-111: AWQIntelAMXMoEScheme._init_kernel()
```python
    def _init_kernel(self, quant_config: "AWQConfig"):
        return AWQIntelAMXMoEKernel(quant_config)
```
**EN:** This block defines `AWQIntelAMXMoEScheme._init_kernel()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `AWQIntelAMXMoEScheme._init_kernel()`，用于实现量化栈中的可复用模块逻辑。

### Lines 113-117: AWQIntelAMXMoEScheme.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        self.kernel.create_moe_runner(layer, moe_runner_config)
```
**EN:** This block defines `AWQIntelAMXMoEScheme.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `AWQIntelAMXMoEScheme.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

## Key Concepts / 关键概念
- `AWQIntelAMXLinearKernel`: A supporting class that structures file-level quantization behavior. / `AWQIntelAMXLinearKernel` 是一个支撑类，用于组织该文件中的量化行为。
- `AWQIntelAMXLinearScheme`: A scheme class that structures file-level quantization behavior. / `AWQIntelAMXLinearScheme` 是一个执行方案类，用于组织该文件中的量化行为。
- `AWQIntelAMXMoEKernel`: A supporting class that structures file-level quantization behavior. / `AWQIntelAMXMoEKernel` 是一个支撑类，用于组织该文件中的量化行为。
- `AWQIntelAMXMoEScheme`: A scheme class that structures file-level quantization behavior. / `AWQIntelAMXMoEScheme` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `torch`, `typing`
- **Internal / 内部**: `.awq_linear`, `.awq_moe`, `sglang.srt.layers.amx_utils`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.awq.awq`
