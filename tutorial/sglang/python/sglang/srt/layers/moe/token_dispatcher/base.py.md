# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/token_dispatcher/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `_RemovableDispatcherHandle`, `DispatcherBaseHooks`, `_PreDispatchHooks`, and `_PostDispatchHooks` and connects them to backend-specific paths such as `Triton`, `FlashInfer`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合混合专家路由与专家执行。它提供了 `_RemovableDispatcherHandle`、`DispatcherBaseHooks`、`_PreDispatchHooks` 以及 `_PostDispatchHooks` 等符号，并把这些符号连接到 `Triton`、`FlashInfer` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-38: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import weakref
from abc import ABC, abstractmethod
from enum import Enum
from typing import (
    TYPE_CHECKING,
    Any,
    Callable,
    Optional,
    OrderedDict,
    Protocol,
    Tuple,
    TypeGuard,
    Union,
    runtime_checkable,
)

import torch

if TYPE_CHECKING:
    from sglang.srt.batch_overlap.single_batch_overlap import CombineOverlapArgs
    from sglang.srt.layers.moe.token_dispatcher import (
        DeepEPLLCombineInput,
        DeepEPLLDispatchOutput,
        DeepEPNormalCombineInput,
        DeepEPNormalDispatchOutput,
        FlashinferCombineInput,
        FlashinferDispatchOutput,
        StandardCombineInput,
        StandardDispatchOutput,
    )
    from sglang.srt.layers.moe.topk import TopKOutput


# ------------------------------ Dispatcher Hook -------------------------------------
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `weakref`, `abc.ABC`, `abc.abstractmethod`, `enum.Enum`, and `typing.TYPE_CHECKING`, so later blocks can reuse runtime, tensor, or backend helpers. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`weakref`、`abc.ABC`、`abc.abstractmethod`、`enum.Enum` 以及 `typing.TYPE_CHECKING`，让后续代码可以复用运行时、张量或后端辅助逻辑。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 39-42: Class `_RemovableDispatcherHandle` declaration and shared state
```python
class _RemovableDispatcherHandle:

    next_id = 0  # Global counter for unique IDs
```
**EN:** This block introduces class `_RemovableDispatcherHandle` and the state shared by its methods.
**CN:** 该代码块引入类 `_RemovableDispatcherHandle`，并定义其方法共享的状态。

### Lines 43-47: `_RemovableDispatcherHandle` initialization and state setup
```python
    def __init__(self, hooks_dict: OrderedDict):
        self.id = _RemovableDispatcherHandle.next_id
        _RemovableDispatcherHandle.next_id += 1
        self.weak_hooks_dict = weakref.ref(hooks_dict)
```
**EN:** This block defines `_RemovableDispatcherHandle.__init__` and contains the main logic for this step. It mainly invokes `weakref.ref`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.id` and `self.weak_hooks_dict` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_RemovableDispatcherHandle.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `weakref.ref`，说明该流程会编排底层辅助函数或计算内核。 像 `self.id` 和 `self.weak_hooks_dict` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 48-53: Function `_RemovableDispatcherHandle.remove` and its core logic
```python
    def remove(self):
        hooks_dict = self.weak_hooks_dict()
        if hooks_dict is not None and self.id in hooks_dict:
            del hooks_dict[self.id]
```
**EN:** This block defines `_RemovableDispatcherHandle.remove` and contains the main logic for this step. It mainly invokes `self.weak_hooks_dict`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hooks_dict` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_RemovableDispatcherHandle.remove`，并承载这一阶段的核心逻辑。 它主要调用 `self.weak_hooks_dict`，说明该流程会编排底层辅助函数或计算内核。 像 `hooks_dict` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 54-55: Class `DispatcherBaseHooks` declaration and shared state
```python
class DispatcherBaseHooks:
```
**EN:** This block introduces class `DispatcherBaseHooks` and the state shared by its methods.
**CN:** 该代码块引入类 `DispatcherBaseHooks`，并定义其方法共享的状态。

### Lines 56-58: `DispatcherBaseHooks` initialization and state setup
```python
    def __init__(self):
        self.hook_dict = OrderedDict[int, Callable]()
```
**EN:** This block defines `DispatcherBaseHooks.__init__` and contains the main logic for this step. It mainly invokes `OrderedDict`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.hook_dict` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DispatcherBaseHooks.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `OrderedDict`，说明该流程会编排底层辅助函数或计算内核。 像 `self.hook_dict` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 59-63: Function `DispatcherBaseHooks.register_hook` and its core logic
```python
    def register_hook(self, hook_fun: Callable) -> _RemovableDispatcherHandle:
        handle = _RemovableDispatcherHandle(self.hook_dict)
        self.hook_dict[handle.id] = hook_fun
        return handle
```
**EN:** This block defines `DispatcherBaseHooks.register_hook` and contains the main logic for this step. It mainly invokes `_RemovableDispatcherHandle`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `handle` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatcherBaseHooks.register_hook`，并承载这一阶段的核心逻辑。 它主要调用 `_RemovableDispatcherHandle`，说明该流程会编排底层辅助函数或计算内核。 像 `handle` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 64-67: `DispatcherBaseHooks.__call__` callable execution entry
```python
    def __call__(self, *args, **kwargs) -> Optional[Any]:
        raise NotImplementedError("This method should be overridden by subclasses")
```
**EN:** This block defines `DispatcherBaseHooks.__call__` and contains the main logic for this step. It mainly invokes `NotImplementedError`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `DispatcherBaseHooks.__call__`，并承载这一阶段的核心逻辑。 它主要调用 `NotImplementedError`，说明该流程会编排底层辅助函数或计算内核。

### Lines 68-69: Class `_PreDispatchHooks` declaration and shared state
```python
class _PreDispatchHooks(DispatcherBaseHooks):
```
**EN:** This block introduces class `_PreDispatchHooks` and the state shared by its methods. It inherits from `DispatcherBaseHooks`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_PreDispatchHooks`，并定义其方法共享的状态。 它继承自 `DispatcherBaseHooks`，说明了它在 SRT 层栈中的接入方式。

### Lines 70-82: `_PreDispatchHooks.__call__` callable execution entry
```python
    def __call__(
        self,
        dispatcher: BaseDispatcher,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ) -> Optional[Tuple[torch.Tensor, TopKOutput]]:
        for hook_fun in self.hook_dict.values():
            hook_output = hook_fun(dispatcher, hidden_states, topk_output)
            if hook_output is not None:
                hidden_states, topk_output = hook_output
        return hidden_states, topk_output
```
**EN:** This block defines `_PreDispatchHooks.__call__` and contains the main logic for this step. It mainly invokes `self.hook_dict.values` and `hook_fun`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hook_output`, `hidden_states`, and `topk_output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_PreDispatchHooks.__call__`，并承载这一阶段的核心逻辑。 它主要调用 `self.hook_dict.values` 和 `hook_fun`，说明该流程会编排底层辅助函数或计算内核。 像 `hook_output`、`hidden_states` 以及 `topk_output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 83-84: Class `_PostDispatchHooks` declaration and shared state
```python
class _PostDispatchHooks(DispatcherBaseHooks):
```
**EN:** This block introduces class `_PostDispatchHooks` and the state shared by its methods. It inherits from `DispatcherBaseHooks`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_PostDispatchHooks`，并定义其方法共享的状态。 它继承自 `DispatcherBaseHooks`，说明了它在 SRT 层栈中的接入方式。

### Lines 85-94: `_PostDispatchHooks.__call__` callable execution entry
```python
    def __call__(
        self, dispatcher: BaseDispatcher, dispatch_output: DispatchOutput
    ) -> Optional[DispatchOutput]:
        for hook_fun in self.hook_dict.values():
            hook_output = hook_fun(dispatcher, dispatch_output)
            if hook_output is not None:
                dispatch_output = hook_output
        return dispatch_output
```
**EN:** This block defines `_PostDispatchHooks.__call__` and contains the main logic for this step. It mainly invokes `self.hook_dict.values` and `hook_fun`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hook_output` and `dispatch_output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_PostDispatchHooks.__call__`，并承载这一阶段的核心逻辑。 它主要调用 `self.hook_dict.values` 和 `hook_fun`，说明该流程会编排底层辅助函数或计算内核。 像 `hook_output` 和 `dispatch_output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 95-96: Class `_PreCombineHooks` declaration and shared state
```python
class _PreCombineHooks(DispatcherBaseHooks):
```
**EN:** This block introduces class `_PreCombineHooks` and the state shared by its methods. It inherits from `DispatcherBaseHooks`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_PreCombineHooks`，并定义其方法共享的状态。 它继承自 `DispatcherBaseHooks`，说明了它在 SRT 层栈中的接入方式。

### Lines 97-106: `_PreCombineHooks.__call__` callable execution entry
```python
    def __call__(
        self, dispatcher: BaseDispatcher, combine_input: CombineInput
    ) -> Optional[CombineInput]:
        for hook_fun in self.hook_dict.values():
            hook_output = hook_fun(dispatcher, combine_input)
            if hook_output is not None:
                combine_input = hook_output
        return combine_input
```
**EN:** This block defines `_PreCombineHooks.__call__` and contains the main logic for this step. It mainly invokes `self.hook_dict.values` and `hook_fun`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hook_output` and `combine_input` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_PreCombineHooks.__call__`，并承载这一阶段的核心逻辑。 它主要调用 `self.hook_dict.values` 和 `hook_fun`，说明该流程会编排底层辅助函数或计算内核。 像 `hook_output` 和 `combine_input` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 107-108: Class `_PostCombineHooks` declaration and shared state
```python
class _PostCombineHooks(DispatcherBaseHooks):
```
**EN:** This block introduces class `_PostCombineHooks` and the state shared by its methods. It inherits from `DispatcherBaseHooks`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_PostCombineHooks`，并定义其方法共享的状态。 它继承自 `DispatcherBaseHooks`，说明了它在 SRT 层栈中的接入方式。

### Lines 109-121: `_PostCombineHooks.__call__` callable execution entry
```python
    def __call__(
        self, dispatcher: BaseDispatcher, hidden_states: torch.Tensor
    ) -> Optional[torch.Tensor]:
        for hook_fun in self.hook_dict.values():
            hook_output = hook_fun(dispatcher, hidden_states)
            if hook_output is not None:
                hidden_states = hook_output
        return hidden_states


# ------------------------------ Dispatch Output -------------------------------------
```
**EN:** This block defines `_PostCombineHooks.__call__` and contains the main logic for this step. It mainly invokes `self.hook_dict.values` and `hook_fun`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hook_output` and `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_PostCombineHooks.__call__`，并承载这一阶段的核心逻辑。 它主要调用 `self.hook_dict.values` 和 `hook_fun`，说明该流程会编排底层辅助函数或计算内核。 像 `hook_output` 和 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 122-123: Class `DispatchOutputChecker` declaration and shared state
```python
class DispatchOutputChecker:
```
**EN:** This block introduces class `DispatchOutputChecker` and the state shared by its methods.
**CN:** 该代码块引入类 `DispatchOutputChecker`，并定义其方法共享的状态。

### Lines 124-129: Function `DispatchOutputChecker.format_is_standard` and its core logic
```python
    @staticmethod
    def format_is_standard(
        dispatch_output: DispatchOutput,
    ) -> TypeGuard[StandardDispatchOutput]:
        return dispatch_output.format.is_standard()
```
**EN:** This block defines `DispatchOutputChecker.format_is_standard` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `dispatch_output.format.is_standard`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputChecker.format_is_standard`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `dispatch_output.format.is_standard`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 130-135: Function `DispatchOutputChecker.format_is_triton_kernels` and its core logic
```python
    @staticmethod
    def format_is_triton_kernels(
        dispatch_output: DispatchOutput,
    ) -> TypeGuard[StandardDispatchOutput]:
        return dispatch_output.format.is_standard()
```
**EN:** This block defines `DispatchOutputChecker.format_is_triton_kernels` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `dispatch_output.format.is_standard`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputChecker.format_is_triton_kernels`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `dispatch_output.format.is_standard`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 136-141: Function `DispatchOutputChecker.format_is_deepep_normal` and its core logic
```python
    @staticmethod
    def format_is_deepep_normal(
        dispatch_output: DispatchOutput,
    ) -> TypeGuard[DeepEPNormalDispatchOutput]:
        return dispatch_output.format.is_deepep_normal()
```
**EN:** This block defines `DispatchOutputChecker.format_is_deepep_normal` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `dispatch_output.format.is_deepep_normal`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputChecker.format_is_deepep_normal`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `dispatch_output.format.is_deepep_normal`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 142-147: Function `DispatchOutputChecker.format_is_deepep_ll` and its core logic
```python
    @staticmethod
    def format_is_deepep_ll(
        dispatch_output: DispatchOutput,
    ) -> TypeGuard[DeepEPLLDispatchOutput]:
        return dispatch_output.format.is_deepep_ll()
```
**EN:** This block defines `DispatchOutputChecker.format_is_deepep_ll` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `dispatch_output.format.is_deepep_ll`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputChecker.format_is_deepep_ll`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `dispatch_output.format.is_deepep_ll`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 148-153: Function `DispatchOutputChecker.format_is_deepep` and its core logic
```python
    @staticmethod
    def format_is_deepep(
        dispatch_output: DispatchOutput,
    ) -> TypeGuard[Union[DeepEPNormalDispatchOutput, DeepEPLLDispatchOutput]]:
        return dispatch_output.format.is_deepep()
```
**EN:** This block defines `DispatchOutputChecker.format_is_deepep` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `dispatch_output.format.is_deepep`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputChecker.format_is_deepep`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `dispatch_output.format.is_deepep`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 154-160: Function `DispatchOutputChecker.format_is_flashinfer` and its core logic
```python
    @staticmethod
    def format_is_flashinfer(
        dispatch_output: DispatchOutput,
    ) -> TypeGuard[FlashinferDispatchOutput]:
        return dispatch_output.format.is_flashinfer()
```
**EN:** This block defines `DispatchOutputChecker.format_is_flashinfer` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `dispatch_output.format.is_flashinfer`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputChecker.format_is_flashinfer`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `dispatch_output.format.is_flashinfer`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 161-167: Class `DispatchOutputFormat` declaration and shared state
```python
class DispatchOutputFormat(Enum):

    STANDARD = "standard"
    DEEPEP_NORMAL = "deepep_normal"
    DEEPEP_LL = "deepep_ll"
    FLASHINFER = "flashinfer"
```
**EN:** This block introduces class `DispatchOutputFormat` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DispatchOutputFormat`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 168-170: `DispatchOutputFormat.is_standard` predicate for is standard
```python
    def is_standard(self) -> bool:
        return self == DispatchOutputFormat.STANDARD
```
**EN:** This block defines `DispatchOutputFormat.is_standard` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputFormat.is_standard`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 171-173: `DispatchOutputFormat.is_deepep_normal` predicate for is deepep normal
```python
    def is_deepep_normal(self) -> bool:
        return self == DispatchOutputFormat.DEEPEP_NORMAL
```
**EN:** This block defines `DispatchOutputFormat.is_deepep_normal` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputFormat.is_deepep_normal`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 174-176: `DispatchOutputFormat.is_deepep_ll` predicate for is deepep ll
```python
    def is_deepep_ll(self) -> bool:
        return self == DispatchOutputFormat.DEEPEP_LL
```
**EN:** This block defines `DispatchOutputFormat.is_deepep_ll` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputFormat.is_deepep_ll`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 177-182: `DispatchOutputFormat.is_deepep` predicate for is deepep
```python
    def is_deepep(self) -> bool:
        return self in [
            DispatchOutputFormat.DEEPEP_NORMAL,
            DispatchOutputFormat.DEEPEP_LL,
        ]
```
**EN:** This block defines `DispatchOutputFormat.is_deepep` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputFormat.is_deepep`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 183-186: `DispatchOutputFormat.is_flashinfer` predicate for is FlashInfer
```python
    def is_flashinfer(self) -> bool:
        return self == DispatchOutputFormat.FLASHINFER
```
**EN:** This block defines `DispatchOutputFormat.is_flashinfer` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DispatchOutputFormat.is_flashinfer`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 187-192: Class `DispatchOutput` declaration and shared state
```python
@runtime_checkable
class DispatchOutput(Protocol):
    """Protocol for dispatch outputs in different formats."""

    hidden_states: torch.Tensor
```
**EN:** This block introduces class `DispatchOutput` and the state shared by its methods. It inherits from `Protocol`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Protocol for dispatch outputs in different formats.
**CN:** 该代码块引入类 `DispatchOutput`，并定义其方法共享的状态。 它继承自 `Protocol`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 193-199: Function `DispatchOutput.format` and its core logic
```python
    @property
    def format(self) -> DispatchOutputFormat: ...


# ------------------------------ Combine Input -------------------------------------
```
**EN:** This block defines `DispatchOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `DispatchOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 200-200: Class `CombineInputChecker` declaration and shared state
```python
class CombineInputChecker:
```
**EN:** This block introduces class `CombineInputChecker` and the state shared by its methods.
**CN:** 该代码块引入类 `CombineInputChecker`，并定义其方法共享的状态。

### Lines 201-206: Function `CombineInputChecker.format_is_standard` and its core logic
```python
    @staticmethod
    def format_is_standard(
        combine_input: CombineInput,
    ) -> TypeGuard[StandardCombineInput]:
        return combine_input.format == CombineInputFormat.STANDARD
```
**EN:** This block defines `CombineInputChecker.format_is_standard` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CombineInputChecker.format_is_standard`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 207-212: Function `CombineInputChecker.format_is_deepep_normal` and its core logic
```python
    @staticmethod
    def format_is_deepep_normal(
        combine_input: CombineInput,
    ) -> TypeGuard[DeepEPNormalCombineInput]:
        return combine_input.format == CombineInputFormat.DEEPEP_NORMAL
```
**EN:** This block defines `CombineInputChecker.format_is_deepep_normal` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CombineInputChecker.format_is_deepep_normal`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 213-218: Function `CombineInputChecker.format_is_deepep_ll` and its core logic
```python
    @staticmethod
    def format_is_deepep_ll(
        combine_input: CombineInput,
    ) -> TypeGuard[DeepEPLLCombineInput]:
        return combine_input.format == CombineInputFormat.DEEPEP_LL
```
**EN:** This block defines `CombineInputChecker.format_is_deepep_ll` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CombineInputChecker.format_is_deepep_ll`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 219-227: Function `CombineInputChecker.format_is_deepep` and its core logic
```python
    @staticmethod
    def format_is_deepep(
        combine_input: CombineInput,
    ) -> TypeGuard[Union[DeepEPNormalCombineInput, DeepEPLLCombineInput]]:
        return combine_input.format in [
            CombineInputFormat.DEEPEP_NORMAL,
            CombineInputFormat.DEEPEP_LL,
        ]
```
**EN:** This block defines `CombineInputChecker.format_is_deepep` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CombineInputChecker.format_is_deepep`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 228-234: Function `CombineInputChecker.format_is_flashinfer` and its core logic
```python
    @staticmethod
    def format_is_flashinfer(
        combine_input: CombineInput,
    ) -> TypeGuard[FlashinferCombineInput]:
        return combine_input.format == CombineInputFormat.FLASHINFER
```
**EN:** This block defines `CombineInputChecker.format_is_flashinfer` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CombineInputChecker.format_is_flashinfer`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 235-241: Class `CombineInputFormat` declaration and shared state
```python
class CombineInputFormat(Enum):
    STANDARD = "standard"
    DEEPEP_NORMAL = "deepep_normal"
    DEEPEP_LL = "deepep_ll"
    FLASHINFER = "flashinfer"
```
**EN:** This block introduces class `CombineInputFormat` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `CombineInputFormat`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 242-247: Class `CombineInput` declaration and shared state
```python
@runtime_checkable
class CombineInput(Protocol):
    """Protocol for combine inputs in different formats."""

    # TODO: add hidden_states to the protocol
```
**EN:** This block introduces class `CombineInput` and the state shared by its methods. It inherits from `Protocol`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Protocol for combine inputs in different formats.
**CN:** 该代码块引入类 `CombineInput`，并定义其方法共享的状态。 它继承自 `Protocol`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 248-254: Function `CombineInput.format` and its core logic
```python
    @property
    def format(self) -> CombineInputFormat: ...


# ------------------------------ Base Dispatcher -------------------------------------
```
**EN:** This block defines `CombineInput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `CombineInput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 255-260: Class `BaseDispatcherConfig` declaration and shared state
```python
class BaseDispatcherConfig(ABC):
    """Base class for dispatcher configs."""

    pass
```
**EN:** This block introduces class `BaseDispatcherConfig` and the state shared by its methods. It inherits from `ABC`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Base class for dispatcher configs.
**CN:** 该代码块引入类 `BaseDispatcherConfig`，并定义其方法共享的状态。 它继承自 `ABC`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 261-263: Class `BaseDispatcher` declaration and shared state
```python
class BaseDispatcher(ABC):
    """Base class for dispatchers."""
```
**EN:** This block introduces class `BaseDispatcher` and the state shared by its methods. It inherits from `ABC`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Base class for dispatchers.
**CN:** 该代码块引入类 `BaseDispatcher`，并定义其方法共享的状态。 它继承自 `ABC`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 264-278: `BaseDispatcher` initialization and state setup
```python
    def __init__(self):
        self.quant_config: dict = {}

        # Overlap args
        self.overlap_args: Optional[CombineOverlapArgs] = None
        self.meta_overlap_args: Optional[dict] = None

        # Hooks
        self._pre_dispatch_hooks: Optional[_PreDispatchHooks] = None
        self._post_dispatch_hooks: Optional[_PostDispatchHooks] = None
        self._pre_combine_hooks: Optional[_PreCombineHooks] = None
        self._post_combine_hooks: Optional[_PostCombineHooks] = None
        self._original_dispatch_func: Optional[Callable] = None
        self._original_combine_func: Optional[Callable] = None
```
**EN:** This block defines `BaseDispatcher.__init__` and contains the main logic for this step. Intermediate names such as `self.quant_config`, `self.overlap_args`, `self.meta_overlap_args`, `self._pre_dispatch_hooks`, and `self._post_dispatch_hooks` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `BaseDispatcher.__init__`，并承载这一阶段的核心逻辑。 像 `self.quant_config`、`self.overlap_args`、`self.meta_overlap_args`、`self._pre_dispatch_hooks` 以及 `self._post_dispatch_hooks` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 279-284: Function `BaseDispatcher.dispatch` and its core logic
```python
    @abstractmethod
    def dispatch(
        self, hidden_states: torch.Tensor, topk_output: TopKOutput
    ) -> DispatchOutput:
        pass
```
**EN:** This block defines `BaseDispatcher.dispatch` and contains the main logic for this step. Decorators like `abstractmethod` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `BaseDispatcher.dispatch`，并承载这一阶段的核心逻辑。 像 `abstractmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 285-298: Internal helper `BaseDispatcher._dispatch_with_hook`
```python
    def _dispatch_with_hook(
        self, hidden_states: torch.Tensor, topk_output: TopKOutput
    ) -> DispatchOutput:
        if self._pre_dispatch_hooks is not None:
            hidden_states, topk_output = self._pre_dispatch_hooks(
                self, hidden_states, topk_output
            )
        dispatch_output = self._original_dispatch_func(
            hidden_states=hidden_states, topk_output=topk_output
        )
        if self._post_dispatch_hooks is not None:
            dispatch_output = self._post_dispatch_hooks(self, dispatch_output)
        return dispatch_output
```
**EN:** This block defines `BaseDispatcher._dispatch_with_hook` and contains the main logic for this step. It mainly invokes `self._original_dispatch_func`, `self._pre_dispatch_hooks`, and `self._post_dispatch_hooks`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `dispatch_output`, `hidden_states`, and `topk_output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `BaseDispatcher._dispatch_with_hook`，并承载这一阶段的核心逻辑。 它主要调用 `self._original_dispatch_func`、`self._pre_dispatch_hooks` 以及 `self._post_dispatch_hooks`，说明该流程会编排底层辅助函数或计算内核。 像 `dispatch_output`、`hidden_states` 以及 `topk_output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 299-303: Internal helper `BaseDispatcher._override_dispatch_func`
```python
    def _override_dispatch_func(self) -> None:
        if self._original_dispatch_func is None:
            self._original_dispatch_func = self.dispatch
            self.dispatch = self._dispatch_with_hook
```
**EN:** This block defines `BaseDispatcher._override_dispatch_func` and contains the main logic for this step. Intermediate names such as `self._original_dispatch_func` and `self.dispatch` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `BaseDispatcher._override_dispatch_func`，并承载这一阶段的核心逻辑。 像 `self._original_dispatch_func` 和 `self.dispatch` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 304-307: Function `BaseDispatcher.combine` and its core logic
```python
    @abstractmethod
    def combine(self, combine_input: CombineInput) -> torch.Tensor:
        pass
```
**EN:** This block defines `BaseDispatcher.combine` and contains the main logic for this step. Decorators like `abstractmethod` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `BaseDispatcher.combine`，并承载这一阶段的核心逻辑。 像 `abstractmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 308-315: Internal helper `BaseDispatcher._combine_with_hook`
```python
    def _combine_with_hook(self, combine_input: CombineInput) -> torch.Tensor:
        if self._pre_combine_hooks is not None:
            combine_input = self._pre_combine_hooks(self, combine_input)
        hidden_states = self._original_combine_func(combine_input=combine_input)
        if self._post_combine_hooks is not None:
            hidden_states = self._post_combine_hooks(self, hidden_states)
        return hidden_states
```
**EN:** This block defines `BaseDispatcher._combine_with_hook` and contains the main logic for this step. It mainly invokes `self._original_combine_func`, `self._pre_combine_hooks`, and `self._post_combine_hooks`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states` and `combine_input` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `BaseDispatcher._combine_with_hook`，并承载这一阶段的核心逻辑。 它主要调用 `self._original_combine_func`、`self._pre_combine_hooks` 以及 `self._post_combine_hooks`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states` 和 `combine_input` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 316-320: Internal helper `BaseDispatcher._override_combine_func`
```python
    def _override_combine_func(self) -> None:
        if self._original_combine_func is None:
            self._original_combine_func = self.combine
            self.combine = self._combine_with_hook
```
**EN:** This block defines `BaseDispatcher._override_combine_func` and contains the main logic for this step. Intermediate names such as `self._original_combine_func` and `self.combine` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `BaseDispatcher._override_combine_func`，并承载这一阶段的核心逻辑。 像 `self._original_combine_func` 和 `self.combine` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 321-333: Function `BaseDispatcher.register_pre_dispatch_hook` and its core logic
```python
    def register_pre_dispatch_hook(
        self,
        hook: Callable[
            [BaseDispatcher, torch.Tensor, TopKOutput],
            Optional[Tuple[torch.Tensor, TopKOutput]],
        ],
    ) -> _RemovableDispatcherHandle:
        if self._pre_dispatch_hooks is None:
            self._pre_dispatch_hooks = _PreDispatchHooks()
            self._override_dispatch_func()
        handle = self._pre_dispatch_hooks.register_hook(hook)
        return handle
```
**EN:** This block defines `BaseDispatcher.register_pre_dispatch_hook` and contains the main logic for this step. It mainly invokes `self._pre_dispatch_hooks.register_hook`, `_PreDispatchHooks`, and `self._override_dispatch_func`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `handle` and `self._pre_dispatch_hooks` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `BaseDispatcher.register_pre_dispatch_hook`，并承载这一阶段的核心逻辑。 它主要调用 `self._pre_dispatch_hooks.register_hook`、`_PreDispatchHooks` 以及 `self._override_dispatch_func`，说明该流程会编排底层辅助函数或计算内核。 像 `handle` 和 `self._pre_dispatch_hooks` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 334-342: Function `BaseDispatcher.register_post_dispatch_hook` and its core logic
```python
    def register_post_dispatch_hook(
        self, hook: Callable[[BaseDispatcher, DispatchOutput], Optional[DispatchOutput]]
    ) -> _RemovableDispatcherHandle:
        if self._post_dispatch_hooks is None:
            self._post_dispatch_hooks = _PostDispatchHooks()
            self._override_dispatch_func()
        handle = self._post_dispatch_hooks.register_hook(hook)
        return handle
```
**EN:** This block defines `BaseDispatcher.register_post_dispatch_hook` and contains the main logic for this step. It mainly invokes `self._post_dispatch_hooks.register_hook`, `_PostDispatchHooks`, and `self._override_dispatch_func`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `handle` and `self._post_dispatch_hooks` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `BaseDispatcher.register_post_dispatch_hook`，并承载这一阶段的核心逻辑。 它主要调用 `self._post_dispatch_hooks.register_hook`、`_PostDispatchHooks` 以及 `self._override_dispatch_func`，说明该流程会编排底层辅助函数或计算内核。 像 `handle` 和 `self._post_dispatch_hooks` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 343-351: Function `BaseDispatcher.register_pre_combine_hook` and its core logic
```python
    def register_pre_combine_hook(
        self, hook: Callable[[BaseDispatcher, CombineInput], Optional[CombineInput]]
    ) -> _RemovableDispatcherHandle:
        if self._pre_combine_hooks is None:
            self._pre_combine_hooks = _PreCombineHooks()
            self._override_combine_func()
        handle = self._pre_combine_hooks.register_hook(hook)
        return handle
```
**EN:** This block defines `BaseDispatcher.register_pre_combine_hook` and contains the main logic for this step. It mainly invokes `self._pre_combine_hooks.register_hook`, `_PreCombineHooks`, and `self._override_combine_func`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `handle` and `self._pre_combine_hooks` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `BaseDispatcher.register_pre_combine_hook`，并承载这一阶段的核心逻辑。 它主要调用 `self._pre_combine_hooks.register_hook`、`_PreCombineHooks` 以及 `self._override_combine_func`，说明该流程会编排底层辅助函数或计算内核。 像 `handle` 和 `self._pre_combine_hooks` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 352-360: Function `BaseDispatcher.register_post_combine_hook` and its core logic
```python
    def register_post_combine_hook(
        self, hook: Callable[[BaseDispatcher, torch.Tensor], Optional[torch.Tensor]]
    ) -> _RemovableDispatcherHandle:
        if self._post_combine_hooks is None:
            self._post_combine_hooks = _PostCombineHooks()
            self._override_combine_func()
        handle = self._post_combine_hooks.register_hook(hook)
        return handle
```
**EN:** This block defines `BaseDispatcher.register_post_combine_hook` and contains the main logic for this step. It mainly invokes `self._post_combine_hooks.register_hook`, `_PostCombineHooks`, and `self._override_combine_func`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `handle` and `self._post_combine_hooks` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `BaseDispatcher.register_post_combine_hook`，并承载这一阶段的核心逻辑。 它主要调用 `self._post_combine_hooks.register_hook`、`_PostCombineHooks` 以及 `self._override_combine_func`，说明该流程会编排底层辅助函数或计算内核。 像 `handle` 和 `self._post_combine_hooks` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 361-363: `BaseDispatcher.set_quant_config` setter for quantization config
```python
    def set_quant_config(self, quant_config: dict) -> None:
        self.quant_config = quant_config
```
**EN:** This block defines `BaseDispatcher.set_quant_config` and contains the main logic for this step. Intermediate names such as `self.quant_config` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `BaseDispatcher.set_quant_config`，并承载这一阶段的核心逻辑。 像 `self.quant_config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 364-369: `BaseDispatcher.set_overlap_args` setter for overlap args
```python
    def set_overlap_args(
        self, combine_overlap_args: CombineOverlapArgs, meta_overlap_args: dict
    ) -> None:
        self.overlap_args = combine_overlap_args
        self.meta_overlap_args = meta_overlap_args
```
**EN:** This block defines `BaseDispatcher.set_overlap_args` and contains the main logic for this step. Intermediate names such as `self.overlap_args` and `self.meta_overlap_args` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `BaseDispatcher.set_overlap_args`，并承载这一阶段的核心逻辑。 像 `self.overlap_args` 和 `self.meta_overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 370-372: Function `BaseDispatcher.clear_overlap_args` and its core logic
```python
    def clear_overlap_args(self) -> None:
        self.overlap_args = None
        self.meta_overlap_args = None
```
**EN:** This block defines `BaseDispatcher.clear_overlap_args` and contains the main logic for this step. Intermediate names such as `self.overlap_args` and `self.meta_overlap_args` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `BaseDispatcher.clear_overlap_args`，并承载这一阶段的核心逻辑。 像 `self.overlap_args` 和 `self.meta_overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_RemovableDispatcherHandle`, `DispatcherBaseHooks`, `_PreDispatchHooks`, `_PostDispatchHooks`, and `_PreCombineHooks`. / **主要符号**：核心入口包括 `_RemovableDispatcherHandle`、`DispatcherBaseHooks`、`_PreDispatchHooks`、`_PostDispatchHooks` 以及 `_PreCombineHooks`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `weakref`, `abc.ABC`, `abc.abstractmethod`, `enum.Enum`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.Callable`, `typing.Optional`, and `typing.OrderedDict` / **标准库**：`__future__.annotations`、`weakref`、`abc.ABC`、`abc.abstractmethod`、`enum.Enum`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.Callable`、`typing.Optional` 以及 `typing.OrderedDict`
- **Third-party**: `torch` / **第三方依赖**：`torch`
- **Internal SGLang modules**: `sglang.srt.batch_overlap.single_batch_overlap.CombineOverlapArgs`, `sglang.srt.layers.moe.token_dispatcher.DeepEPLLCombineInput`, `sglang.srt.layers.moe.token_dispatcher.DeepEPLLDispatchOutput`, `sglang.srt.layers.moe.token_dispatcher.DeepEPNormalCombineInput`, `sglang.srt.layers.moe.token_dispatcher.DeepEPNormalDispatchOutput`, `sglang.srt.layers.moe.token_dispatcher.FlashinferCombineInput`, `sglang.srt.layers.moe.token_dispatcher.FlashinferDispatchOutput`, `sglang.srt.layers.moe.token_dispatcher.StandardCombineInput`, `sglang.srt.layers.moe.token_dispatcher.StandardDispatchOutput`, and `sglang.srt.layers.moe.topk.TopKOutput` / **SGLang 内部模块**：`sglang.srt.batch_overlap.single_batch_overlap.CombineOverlapArgs`、`sglang.srt.layers.moe.token_dispatcher.DeepEPLLCombineInput`、`sglang.srt.layers.moe.token_dispatcher.DeepEPLLDispatchOutput`、`sglang.srt.layers.moe.token_dispatcher.DeepEPNormalCombineInput`、`sglang.srt.layers.moe.token_dispatcher.DeepEPNormalDispatchOutput`、`sglang.srt.layers.moe.token_dispatcher.FlashinferCombineInput`、`sglang.srt.layers.moe.token_dispatcher.FlashinferDispatchOutput`、`sglang.srt.layers.moe.token_dispatcher.StandardCombineInput`、`sglang.srt.layers.moe.token_dispatcher.StandardDispatchOutput` 以及 `sglang.srt.layers.moe.topk.TopKOutput`
