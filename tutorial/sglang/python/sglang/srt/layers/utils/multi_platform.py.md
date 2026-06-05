# multi_platform.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/utils/multi_platform.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime layer support for the SGLang SRT runtime. It exposes symbols such as `MultiPlatformOp` and connects them to backend-specific paths such as `CUDA`, `NPU`, `XPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了运行时层支持。它提供了 `MultiPlatformOp` 等符号，并把这些符号连接到 `CUDA`、`NPU`、`XPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Imports, constants, and runtime setup
```python
from typing import Callable, ClassVar

from torch import nn

from sglang.kernel_api_logging import debug_kernel_api
from sglang.srt.platforms import current_platform
from sglang.srt.utils import (
    cpu_has_amx_support,
    is_cpu,
    is_cuda,
    is_hip,
    is_musa,
    is_npu,
    is_xpu,
)

_is_cuda = is_cuda()
_is_hip = is_hip()
_is_cpu = is_cpu()
_is_cpu_amx_available = cpu_has_amx_support()
_is_npu = is_npu()
_is_xpu = is_xpu()
_is_musa = is_musa()
```
**EN:** This section prepares the module namespace. It imports `typing.Callable`, `typing.ClassVar`, `torch.nn`, `sglang.kernel_api_logging.debug_kernel_api`, `sglang.srt.platforms.current_platform`, and `sglang.srt.utils.cpu_has_amx_support`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_cuda`, `_is_hip`, `_is_cpu`, `_is_cpu_amx_available`, and `_is_npu` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `typing.Callable`、`typing.ClassVar`、`torch.nn`、`sglang.kernel_api_logging.debug_kernel_api`、`sglang.srt.platforms.current_platform` 以及 `sglang.srt.utils.cpu_has_amx_support`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_cuda`、`_is_hip`、`_is_cpu`、`_is_cpu_amx_available` 以及 `_is_npu` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 26-30: Class `MultiPlatformOp` declaration and shared state
```python
class MultiPlatformOp(nn.Module):

    # OOT forward registry: maps dispatch_key -> {op_cls -> forward_fn}
    _oot_forward_registry: ClassVar[dict[str, dict[type, Callable]]] = {}
```
**EN:** This block introduces class `MultiPlatformOp` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `MultiPlatformOp`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。

### Lines 31-35: Function `MultiPlatformOp.register_oot_forward` and its core logic
```python
    @classmethod
    def register_oot_forward(cls, op_cls: type, fn: Callable, platform_key: str):
        """Register an OOT forward implementation for a specific op class and platform."""
        cls._oot_forward_registry.setdefault(platform_key, {})[op_cls] = fn
```
**EN:** This block defines `MultiPlatformOp.register_oot_forward` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._oot_forward_registry.setdefault`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `MultiPlatformOp.register_oot_forward`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._oot_forward_registry.setdefault`，说明该流程会编排底层辅助函数或计算内核。

### Lines 36-43: `MultiPlatformOp` initialization and state setup
```python
    def __init__(self):
        super().__init__()
        self._forward_method: Callable = self.dispatch_forward()

        # States for torch.compile
        self._original_forward_method = None
        self.is_torch_compile = False
```
**EN:** This block defines `MultiPlatformOp.__init__` and contains the main logic for this step. It mainly invokes `super.__init__` and `self.dispatch_forward`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._forward_method`, `self._original_forward_method`, and `self.is_torch_compile` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MultiPlatformOp.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__` 和 `self.dispatch_forward`，说明该流程会编排底层辅助函数或计算内核。 像 `self._forward_method`、`self._original_forward_method` 以及 `self.is_torch_compile` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 44-70: Function `MultiPlatformOp.enter_torch_compile` and its core logic
```python
    def enter_torch_compile(self, num_tokens: int):
        # Skip if Op is already entered compile mode.
        # NOTE(alcanderian): Some Ops(for example RotaryEmbedding) will be reused
        # among layers and `enter_torch_compile` will be called many times.
        # We should prevent `self._original_forward_method` from being overridden when
        # it is not the first time `enter_torch_compile` called.
        if self.is_torch_compile:
            return

        self._original_forward_method = self._forward_method
        # NOTE: Temporarily workaround MoE
        # The performance of torch.compile on this layer is not always good when bs > 1,
        # so we decide to only use torch.compile when bs=1
        if "FusedMoE" in self.__class__.__name__:
            if num_tokens == 1:
                from sglang.srt.layers.moe.fused_moe_native import (
                    fused_moe_forward_native,
                )

                self._forward_method = fused_moe_forward_native
        elif "TopK" in self.__class__.__name__:
            if num_tokens == 1:
                self._forward_method = self.forward_native
        else:
            self._forward_method = self.forward_native
        self.is_torch_compile = True
```
**EN:** This block defines `MultiPlatformOp.enter_torch_compile` and contains the main logic for this step. Intermediate names such as `self._original_forward_method`, `self.is_torch_compile`, and `self._forward_method` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MultiPlatformOp.enter_torch_compile`，并承载这一阶段的核心逻辑。 像 `self._original_forward_method`、`self.is_torch_compile` 以及 `self._forward_method` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 71-80: Function `MultiPlatformOp.leave_torch_compile` and its core logic
```python
    def leave_torch_compile(self):
        # Skip if Op is already exited compile mode.
        if not self.is_torch_compile:
            return

        self._forward_method = self._original_forward_method
        self._original_forward_method = None
        self.is_torch_compile = False

    # Please do not override this method, because `self._forward_method` can change when in torch compile mode
```
**EN:** This block defines `MultiPlatformOp.leave_torch_compile` and contains the main logic for this step. Intermediate names such as `self._forward_method`, `self._original_forward_method`, and `self.is_torch_compile` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MultiPlatformOp.leave_torch_compile`，并承载这一阶段的核心逻辑。 像 `self._forward_method`、`self._original_forward_method` 以及 `self.is_torch_compile` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 81-84: `MultiPlatformOp.forward` main forward path
```python
    @debug_kernel_api
    def forward(self, *args, **kwargs):
        return self._forward_method(*args, **kwargs)
```
**EN:** This block defines `MultiPlatformOp.forward` and contains the main logic for this step. Decorators like `debug_kernel_api` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `self._forward_method`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MultiPlatformOp.forward`，并承载这一阶段的核心逻辑。 像 `debug_kernel_api` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `self._forward_method`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 85-87: `MultiPlatformOp.forward_native` NATIVE execution path
```python
    def forward_native(self, *args, **kwargs):
        raise NotImplementedError
```
**EN:** This block defines `MultiPlatformOp.forward_native` and contains the main logic for this step.
**CN:** 该代码块定义了 `MultiPlatformOp.forward_native`，并承载这一阶段的核心逻辑。

### Lines 88-90: `MultiPlatformOp.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, *args, **kwargs):
        raise NotImplementedError
```
**EN:** This block defines `MultiPlatformOp.forward_cuda` and contains the main logic for this step.
**CN:** 该代码块定义了 `MultiPlatformOp.forward_cuda`，并承载这一阶段的核心逻辑。

### Lines 91-93: `MultiPlatformOp.forward_npu` NPU execution path
```python
    def forward_npu(self, *args, **kwargs):
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines `MultiPlatformOp.forward_npu` and contains the main logic for this step. It mainly invokes `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MultiPlatformOp.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 94-96: `MultiPlatformOp.forward_hip` HIP execution path
```python
    def forward_hip(self, *args, **kwargs):
        return self.forward_cuda(*args, **kwargs)
```
**EN:** This block defines `MultiPlatformOp.forward_hip` and contains the main logic for this step. It mainly invokes `self.forward_cuda`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MultiPlatformOp.forward_hip`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_cuda`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 97-99: `MultiPlatformOp.forward_xpu` XPU execution path
```python
    def forward_xpu(self, *args, **kwargs):
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines `MultiPlatformOp.forward_xpu` and contains the main logic for this step. It mainly invokes `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MultiPlatformOp.forward_xpu`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 100-102: `MultiPlatformOp.forward_musa` MUSA execution path
```python
    def forward_musa(self, *args, **kwargs):
        return self.forward_cuda(*args, **kwargs)
```
**EN:** This block defines `MultiPlatformOp.forward_musa` and contains the main logic for this step. It mainly invokes `self.forward_cuda`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MultiPlatformOp.forward_musa`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_cuda`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 103-105: `MultiPlatformOp.forward_hpu` HPU execution path
```python
    def forward_hpu(self, *args, **kwargs):
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines `MultiPlatformOp.forward_hpu` and contains the main logic for this step. It mainly invokes `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MultiPlatformOp.forward_hpu`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 106-108: `MultiPlatformOp.forward_cpu` CPU execution path
```python
    def forward_cpu(self, *args, **kwargs):
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines `MultiPlatformOp.forward_cpu` and contains the main logic for this step. It mainly invokes `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MultiPlatformOp.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 109-134: `MultiPlatformOp.dispatch_forward` step for forward
```python
    def dispatch_forward(self):
        # OOT platform dispatch: check registry then method lookup
        if current_platform.is_out_of_tree():
            key = current_platform.get_dispatch_key_name()
            oot = self._oot_forward_registry.get(key, {})
            if type(self) in oot:
                return oot[type(self)].__get__(self)
            method = getattr(self, f"forward_{key}", None)
            if method is not None:
                return method
            return self.forward_native

        if _is_cuda:
            return self.forward_cuda
        elif _is_hip:
            return self.forward_hip
        elif _is_cpu and _is_cpu_amx_available:
            return self.forward_cpu
        elif _is_npu:
            return self.forward_npu
        elif _is_xpu:
            return self.forward_xpu
        elif _is_musa:
            return self.forward_musa
        else:
            return self.forward_native
```
**EN:** This block defines `MultiPlatformOp.dispatch_forward` and contains the main logic for this step. It mainly invokes `current_platform.is_out_of_tree`, `current_platform.get_dispatch_key_name`, `self._oot_forward_registry.get`, `getattr`, and `type`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key`, `oot`, and `method` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MultiPlatformOp.dispatch_forward`，并承载这一阶段的核心逻辑。 它主要调用 `current_platform.is_out_of_tree`、`current_platform.get_dispatch_key_name`、`self._oot_forward_registry.get`、`getattr` 以及 `type`，说明该流程会编排底层辅助函数或计算内核。 像 `key`、`oot` 以及 `method` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `MultiPlatformOp`. / **主要符号**：核心入口包括 `MultiPlatformOp`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。

## Dependencies / 依赖关系
- **Standard library**: `typing.Callable` and `typing.ClassVar` / **标准库**：`typing.Callable` 和 `typing.ClassVar`
- **Third-party**: `torch.nn` / **第三方依赖**：`torch.nn`
- **Internal SGLang modules**: `sglang.kernel_api_logging.debug_kernel_api`, `sglang.srt.platforms.current_platform`, `sglang.srt.utils.cpu_has_amx_support`, `sglang.srt.utils.is_cpu`, `sglang.srt.utils.is_cuda`, `sglang.srt.utils.is_hip`, `sglang.srt.utils.is_musa`, `sglang.srt.utils.is_npu`, `sglang.srt.utils.is_xpu`, and `sglang.srt.layers.moe.fused_moe_native.fused_moe_forward_native` / **SGLang 内部模块**：`sglang.kernel_api_logging.debug_kernel_api`、`sglang.srt.platforms.current_platform`、`sglang.srt.utils.cpu_has_amx_support`、`sglang.srt.utils.is_cpu`、`sglang.srt.utils.is_cuda`、`sglang.srt.utils.is_hip`、`sglang.srt.utils.is_musa`、`sglang.srt.utils.is_npu`、`sglang.srt.utils.is_xpu` 以及 `sglang.srt.layers.moe.fused_moe_native.fused_moe_forward_native`
