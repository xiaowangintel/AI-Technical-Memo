# int8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/online/int8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `Int8OnlineMoEMethod` for quantization backends, schemes, and utilities. / 实现 `Int8OnlineMoEMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-27)
```python
from typing import TYPE_CHECKING

import torch
from torch.nn import Module

if TYPE_CHECKING:
    from vllm.model_executor.layers.fused_moe.config import (
        FusedMoEQuantConfig,
    )

from vllm.model_executor.layers.fused_moe import RoutedExperts
from vllm.model_executor.layers.fused_moe.oracle.int8 import (
    make_int8_moe_kernel,
    make_int8_moe_quant_config,
    select_int8_moe_backend,
)
from vllm.model_executor.layers.quantization.online.moe_base import (
    OnlineMoEMethodBase,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    kInt8DynamicTokenSym,
    kInt8StaticChannelSym,
)
from vllm.model_executor.utils import replace_parameter
```
**EN:** This opening block pulls in external dependencies such as `typing`, `torch` and internal modules such as `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.oracle.int8`, `vllm.model_executor.layers.quantization.online.moe_base`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `torch`）以及内部模块（如 `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.oracle.int8`, `vllm.model_executor.layers.quantization.online.moe_base`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `Int8OnlineMoEMethod` overview (lines 30-113)
```python
class Int8OnlineMoEMethod(OnlineMoEMethodBase):
    """Online per-channel INT8 MoE quantization.
    Loads fp16/bf16 weights and quantizes them per-row to int8 during loading.
    """

    def __init__(
        self,
        *,
        layer: torch.nn.Module,
    ):
        super().__init__(layer.moe_config)
        self.int8_backend, self.experts_cls = select_int8_moe_backend(
            config=self.moe,
            weight_key=kInt8StaticChannelSym,
            activation_key=kInt8DynamicTokenSym,
        )

    def process_weights_after_loading(self, layer: Module) -> None:
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        self._quantize_weights(layer)
        self._setup_kernel(layer)

        layer._already_called_process_weights_after_loading = True
```
**EN:** Defines class `Int8OnlineMoEMethod` with base classes `OnlineMoEMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 5 direct methods, with notable entries `__init__`, `process_weights_after_loading`, `_quantize_weights`, `_setup_kernel`, `get_fused_moe_quant_config`. Its docstring says: Online per-channel INT8 MoE quantization.
**CN:** 定义类 `Int8OnlineMoEMethod`，其基类为 `OnlineMoEMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 5 个方法，较重要的包括 `__init__`, `process_weights_after_loading`, `_quantize_weights`, `_setup_kernel`, `get_fused_moe_quant_config`。 文档字符串进一步说明了该类的定位。

### Method `Int8OnlineMoEMethod.__init__` (lines 35-45)
```python
    def __init__(
        self,
        *,
        layer: torch.nn.Module,
    ):
        super().__init__(layer.moe_config)
        self.int8_backend, self.experts_cls = select_int8_moe_backend(
            config=self.moe,
            weight_key=kInt8StaticChannelSym,
            activation_key=kInt8DynamicTokenSym,
        )
```
**EN:** Defines function `Int8OnlineMoEMethod.__init__` with signature `__init__(self, *, layer: torch.nn.Module)`. It mainly works with `layer`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `select_int8_moe_backend`, `super`.
**CN:** 定义函数 `Int8OnlineMoEMethod.__init__`，其签名为 `__init__(self, *, layer: torch.nn.Module)`。它主要围绕 `layer` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `select_int8_moe_backend`, `super`。

### Method `Int8OnlineMoEMethod.process_weights_after_loading` (lines 47-54)
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        self._quantize_weights(layer)
        self._setup_kernel(layer)

        layer._already_called_process_weights_after_loading = True
```
**EN:** Defines function `Int8OnlineMoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching. Key calls include `getattr`, `self._quantize_weights`, `self._setup_kernel`.
**CN:** 定义函数 `Int8OnlineMoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断。关键调用包括 `getattr`, `self._quantize_weights`, `self._setup_kernel`。

### Method `Int8OnlineMoEMethod._quantize_weights` (lines 56-92)
```python
    def _quantize_weights(self, layer: Module) -> None:
        vmax = torch.iinfo(torch.int8).max

        w13 = torch.empty_like(layer.w13_weight, dtype=torch.int8)
        w2 = torch.empty_like(layer.w2_weight, dtype=torch.int8)
        w13_scale = torch.zeros(
            layer.num_experts,
            layer.w13_weight.shape[1],
            device=w13.device,
            dtype=torch.float32,
        )
        w2_scale = torch.zeros(
            layer.num_experts,
            layer.w2_weight.shape[1],
            device=w2.device,
            dtype=torch.float32,
        )

        for expert in range(layer.local_num_experts):
            # w13: per-row quantization over hidden_size dim
            w = layer.w13_weight[expert, :, :]
            scales = w.abs().amax(dim=1) / vmax
            q = w.div(scales.unsqueeze(1)).round().clamp(-vmax, vmax)
            w13[expert, :, :] = q.to(torch.int8)
            w13_scale[expert, :] = scales

            # w2: per-row quantization over intermediate_size dim
            w = layer.w2_weight[expert, :, :]
            scales = w.abs().amax(dim=1) / vmax
            q = w.div(scales.unsqueeze(1)).round().clamp(-vmax, vmax)
            w2[expert, :, :] = q.to(torch.int8)
            w2_scale[expert, :] = scales

        replace_parameter(layer, "w13_weight", w13)
        replace_parameter(layer, "w2_weight", w2)
        replace_parameter(layer, "w13_scale", w13_scale)
        replace_parameter(layer, "w2_scale", w2_scale)
```
**EN:** Defines function `Int8OnlineMoEMethod._quantize_weights` with signature `_quantize_weights(self, layer: Module) -> None`. It mainly works with `layer`; handles quantization-related transformation logic. The body uses iteration, tensor/kernel operations. Key calls include `torch.empty_like`, `torch.zeros`, `range`, `replace_parameter`, `torch.iinfo`, `w.div.round.clamp`.
**CN:** 定义函数 `Int8OnlineMoEMethod._quantize_weights`，其签名为 `_quantize_weights(self, layer: Module) -> None`。它主要围绕 `layer` 展开；处理量化相关的变换逻辑。函数体包含循环处理、张量或内核操作。关键调用包括 `torch.empty_like`, `torch.zeros`, `range`, `replace_parameter`, `torch.iinfo`, `w.div.round.clamp`。

### Method `Int8OnlineMoEMethod._setup_kernel` (lines 94-103)
```python
    def _setup_kernel(self, layer: RoutedExperts) -> None:
        self.moe_quant_config = self.get_fused_moe_quant_config(layer)
        assert self.moe_quant_config is not None
        assert self.experts_cls is not None
        self.moe_kernel = make_int8_moe_kernel(
            moe_quant_config=self.moe_quant_config,
            moe_config=self.moe,
            experts_cls=self.experts_cls,
            routing_tables=layer._expert_routing_tables(),
        )
```
**EN:** Defines function `Int8OnlineMoEMethod._setup_kernel` with signature `_setup_kernel(self, layer: RoutedExperts) -> None`. It mainly works with `layer`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.get_fused_moe_quant_config`, `make_int8_moe_kernel`, `layer._expert_routing_tables`.
**CN:** 定义函数 `Int8OnlineMoEMethod._setup_kernel`，其签名为 `_setup_kernel(self, layer: RoutedExperts) -> None`。它主要围绕 `layer` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.get_fused_moe_quant_config`, `make_int8_moe_kernel`, `layer._expert_routing_tables`。

### Method `Int8OnlineMoEMethod.get_fused_moe_quant_config` (lines 105-113)
```python
    def get_fused_moe_quant_config(
        self, layer: torch.nn.Module
    ) -> "FusedMoEQuantConfig | None":
        quant_config = make_int8_moe_quant_config(
            w1_scale=layer.w13_scale,
            w2_scale=layer.w2_scale,
        )
        self._maybe_inject_biases(quant_config, layer)
        return quant_config
```
**EN:** Defines function `Int8OnlineMoEMethod.get_fused_moe_quant_config` with signature `get_fused_moe_quant_config(self, layer: torch.nn.Module) -> 'FusedMoEQuantConfig | None'`. It mainly works with `layer`; handles quantization-related transformation logic. The body uses mostly straightforward data movement and object wiring. Key calls include `make_int8_moe_quant_config`, `self._maybe_inject_biases`.
**CN:** 定义函数 `Int8OnlineMoEMethod.get_fused_moe_quant_config`，其签名为 `get_fused_moe_quant_config(self, layer: torch.nn.Module) -> 'FusedMoEQuantConfig | None'`。它主要围绕 `layer` 展开；处理量化相关的变换逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `make_int8_moe_quant_config`, `self._maybe_inject_biases`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `Int8OnlineMoEMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `Int8OnlineMoEMethod`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `torch`
- **Internal / 内部**: `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.oracle.int8`, `vllm.model_executor.layers.quantization.online.moe_base`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.utils`
