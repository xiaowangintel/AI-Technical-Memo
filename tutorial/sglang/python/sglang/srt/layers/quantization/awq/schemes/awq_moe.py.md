# awq_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/awq/schemes/awq_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements scheme classes that describe how awq moe weights are packed, loaded, and executed. / 该模块实现了执行方案类，用于描述 AWQ MoE 权重如何打包、加载与执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
from __future__ import annotations

from typing import TYPE_CHECKING

import torch

from sglang.srt.layers.linear import set_weight_attrs
from sglang.srt.layers.moe import (
    MoeRunner,
    MoeRunnerBackend,
    MoeRunnerConfig,
    get_moe_runner_backend,
)

from .awq_scheme import AWQMoESchemeBase
```
**EN:** This block imports __future__, torch, typing, .awq_scheme, sglang.srt.hardware_backend.gpu.quantization.awq_kernels, sglang.srt.hardware_backend.npu.quantization.awq_kernels, sglang.srt.layers.linear and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, torch, typing, .awq_scheme, sglang.srt.hardware_backend.gpu.quantization.awq_kernels, sglang.srt.hardware_backend.npu.quantization.awq_kernels, sglang.srt.layers.linear 等依赖，并为当前量化实现准备模块命名空间。

### Lines 18-20: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import StandardDispatchOutput
    from sglang.srt.layers.quantization.awq.awq import AWQConfig, AWQMarlinConfig
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 22-22: initialize __all__
```python
__all__ = ["AWQMoEScheme", "AWQAscendMoEScheme"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 25-25: class AWQMoEScheme: definition
```python
class AWQMoEScheme(AWQMoESchemeBase):
```
**EN:** This block declares `AWQMoEScheme`, a scheme class for the quantization stack. It organizes behaviors such as __init__, _init_kernel, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `AWQMoEScheme`，它是量化栈中的执行方案类，组织了 __init__, _init_kernel, create_weights, process_weights_after_loading 等行为。

### Lines 26-30: AWQMoEScheme.__init__()
```python
    def __init__(self, quant_config: "AWQMarlinConfig"):
        self.quant_config = quant_config
        if self.quant_config.weight_bits != 4:
            raise ValueError("AWQMoEScheme only supports 4bit now.")
        self.kernel = self._init_kernel(quant_config)
```
**EN:** This block defines `AWQMoEScheme.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `AWQMoEScheme.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 32-37: AWQMoEScheme._init_kernel()
```python
    def _init_kernel(self, quant_config: "AWQMarlinConfig"):
        from sglang.srt.hardware_backend.gpu.quantization.awq_kernels import (
            AWQMoEKernel,
        )

        return AWQMoEKernel(quant_config)
```
**EN:** This block defines `AWQMoEScheme._init_kernel()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `AWQMoEScheme._init_kernel()`，用于实现量化栈中的可复用模块逻辑。

### Lines 39-83: AWQMoEScheme.create_weights() (part 1/2)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        extra_weight_attrs.update(
            {
                "is_transposed": True,
                "quant_method": FusedMoeWeightScaleSupported.GROUP.value,
            }
        )

        w13_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                2 * intermediate_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_qweight", w13_qweight)
        set_weight_attrs(w13_qweight, extra_weight_attrs)

        w2_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition,
                hidden_size // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_qweight", w2_qweight)
        set_weight_attrs(w2_qweight, extra_weight_attrs)

        num_groups_w13 = hidden_size // self.quant_config.group_size
        num_groups_w2 = intermediate_size_per_partition // self.quant_config.group_size
```
**EN:** This segment of `AWQMoEScheme.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `AWQMoEScheme.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 84-125: AWQMoEScheme.create_weights() (part 2/2)
```python
        w13_scales = torch.nn.Parameter(
            torch.empty(
                num_experts,
                num_groups_w13,
                intermediate_size_per_partition * 2,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_scales", w13_scales)
        set_weight_attrs(w13_scales, extra_weight_attrs)

        w2_scales = torch.nn.Parameter(
            torch.empty(num_experts, num_groups_w2, hidden_size, dtype=params_dtype),
            requires_grad=False,
        )
        layer.register_parameter("w2_scales", w2_scales)
        set_weight_attrs(w2_scales, extra_weight_attrs)

        w13_qzeros = torch.nn.Parameter(
            torch.empty(
                num_experts,
                num_groups_w13,
                2 * intermediate_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_qzeros", w13_qzeros)
        set_weight_attrs(w13_qzeros, extra_weight_attrs)

        w2_qzeros = torch.nn.Parameter(
            torch.empty(
                num_experts,
                num_groups_w2,
                hidden_size // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_qzeros", w2_qzeros)
        set_weight_attrs(w2_qzeros, extra_weight_attrs)
```
**EN:** This segment of `AWQMoEScheme.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `AWQMoEScheme.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 127-128: AWQMoEScheme.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        self.kernel.process_weights_after_loading(layer)
```
**EN:** This block defines `AWQMoEScheme.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `AWQMoEScheme.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 130-135: AWQMoEScheme.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        assert get_moe_runner_backend().is_auto()
        self.moe_runner_config = moe_runner_config
        self.kernel.runner = MoeRunner(MoeRunnerBackend.MARLIN, moe_runner_config)
```
**EN:** This block defines `AWQMoEScheme.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `AWQMoEScheme.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 137-142: AWQMoEScheme.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        dispatch_output: "StandardDispatchOutput",
    ):
        return self.kernel.apply(layer, dispatch_output)
```
**EN:** This block defines `AWQMoEScheme.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `AWQMoEScheme.apply_weights()`，用于将量化计算应用到运行时输入上。

### Lines 145-145: class AWQAscendMoEScheme: definition
```python
class AWQAscendMoEScheme(AWQMoEScheme):
```
**EN:** This block declares `AWQAscendMoEScheme`, a scheme class for the quantization stack. It organizes behaviors such as _init_kernel, create_moe_runner.
**CN:** 该代码块声明 `AWQAscendMoEScheme`，它是量化栈中的执行方案类，组织了 _init_kernel, create_moe_runner 等行为。

### Lines 146-151: AWQAscendMoEScheme._init_kernel()
```python
    def _init_kernel(self, quant_config: "AWQConfig"):
        from sglang.srt.hardware_backend.npu.quantization.awq_kernels import (
            AWQAscendMoEKernel,
        )

        return AWQAscendMoEKernel(quant_config)
```
**EN:** This block defines `AWQAscendMoEScheme._init_kernel()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `AWQAscendMoEScheme._init_kernel()`，用于实现量化栈中的可复用模块逻辑。

### Lines 153-156: AWQAscendMoEScheme.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `AWQAscendMoEScheme.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `AWQAscendMoEScheme.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

## Key Concepts / 关键概念
- `AWQMoEScheme`: A scheme class that structures file-level quantization behavior. / `AWQMoEScheme` 是一个执行方案类，用于组织该文件中的量化行为。
- `AWQAscendMoEScheme`: A scheme class that structures file-level quantization behavior. / `AWQAscendMoEScheme` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `torch`, `typing`
- **Internal / 内部**: `.awq_scheme`, `sglang.srt.hardware_backend.gpu.quantization.awq_kernels`, `sglang.srt.hardware_backend.npu.quantization.awq_kernels`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.awq.awq`
