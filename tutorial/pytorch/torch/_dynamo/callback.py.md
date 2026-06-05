# callback.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/callback.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.compile front-end capture, guard management, and graph/lowering helpers in TorchDynamo. The module docstring emphasizes: This module provides callback management functionality for TorchDynamo's compilation process.
- **Purpose (CN)**: 实现 TorchDynamo 中的 torch.compile 前端捕获、守卫管理与图/降级辅助逻辑。 模块文档字符串重点说明了该实现的职责。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```python
"""
This module provides callback management functionality for TorchDynamo's compilation process.

It implements a thread-safe system for registering, managing and executing callbacks that run
at the start and end of TorchDynamo compilations. Key features include:

- Registration and deregistration of compilation callbacks
- Thread-safe callback handling with proper locking mechanisms
- Prevention of duplicate callback execution when configured
- Decorator utilities for easy callback registration
- Context manager for controlled callback lifecycle

The module centers around the CompilationCallbackHandler class which maintains separate
lists for start and end callbacks, manages their execution order, and ensures thread-safety.
Utility decorators @on_compile_start and @on_compile_end provide a convenient way to
register compilation hooks.
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 18-35
```python
Example usage:
    @on_compile_start
    def my_start_callback():
        print("Starting compilation")

    @on_compile_end
    def my_end_callback():
        print("Compilation complete")
"""

import enum
import threading
from collections.abc import Callable, Generator
from contextlib import contextmanager
from dataclasses import dataclass, field
from typing import Any
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 36-52
```python
class CallbackTrigger(enum.Enum):
    # most common case, dynamo attempts to trace a new frame
    DYNAMO = 1
    # backward compilation can be deferred to runtime
    LAZY_BACKWARD = 2
    # some backends autotune at runtime
    TRITON_AUTOTUNING = 3  # Temporarily disabled due to spam
    # cudagraphs record at runtime
    CUDAGRAPH_RECORDING = 4


@dataclass
class CallbackArgs:
    callback_trigger: CallbackTrigger
    compile_id: str
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 53-68
```python
@dataclass
class CompilationCallbackHandler:
    start_callbacks: list[Callable[[CallbackArgs], None]] = field(default_factory=list)
    end_callbacks: list[Callable[[CallbackArgs], None]] = field(default_factory=list)

    __pending_callbacks_counter: int = field(default=0, init=False, repr=False)
    __pending_callbacks_counter_lock: threading.Lock = field(
        default_factory=threading.Lock, init=False, repr=False
    )

    def register_start_callback(
        self, callback: Callable[[CallbackArgs], None]
    ) -> Callable[[CallbackArgs], None]:
        """
        Register a callback function to be called when the compilation starts.
```
- **EN**: These decorators register or transform the following definition so it can initialize learnable tensors and related state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够初始化可学习张量及相关状态。

### Lines 69-86
```python
        Args:
        - callback (Callable): The callback function to register.
        """
        self.start_callbacks.append(callback)
        return callback

    def register_end_callback(
        self, callback: Callable[[CallbackArgs], None]
    ) -> Callable[[CallbackArgs], None]:
        """
        Register a callback function to be called when the compilation ends.

        Args:
        - callback (Callable): The callback function to register.
        """
        self.end_callbacks.append(callback)
        return callback
```
- **EN**: Declares `CompilationCallbackHandler`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `CompilationCallbackHandler`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 87-104
```python
    def remove_start_callback(self, callback: Callable[[CallbackArgs], None]) -> None:
        """
        Remove a registered start callback function.

        Args:
        - callback (Callable): The callback function to remove.
        """
        self.start_callbacks.remove(callback)

    def remove_end_callback(self, callback: Callable[[CallbackArgs], None]) -> None:
        """
        Remove a registered end callback function.

        Args:
        - callback (Callable): The callback function to remove.
        """
        self.end_callbacks.remove(callback)
```
- **EN**: Declares `CompilationCallbackHandler`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `CompilationCallbackHandler`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 105-118
```python
    def run_start_callbacks(self, args: CallbackArgs) -> None:
        """
        Execute all registered start callbacks.
        """
        for callback in self.start_callbacks:
            callback(args)

    def run_end_callbacks(self, args: CallbackArgs) -> None:
        """
        Execute all registered end callbacks.
        """
        for callback in self.end_callbacks:
            callback(args)
```
- **EN**: Declares `CompilationCallbackHandler`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `CompilationCallbackHandler`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 119-130
```python
    @contextmanager
    def install_callbacks(
        self, trigger: CallbackTrigger, compile_id: str
    ) -> Generator[None, Any, Any]:
        """
        Context manager to install the callbacks and run them when the context is exited.
        """
        args = CallbackArgs(trigger, compile_id)
        try:
            with self.__pending_callbacks_counter_lock:
                self.__pending_callbacks_counter += 1
                if self.__pending_callbacks_counter == 1:
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 131-142
```python
                    self.run_start_callbacks(args)
            yield
        finally:
            with self.__pending_callbacks_counter_lock:
                assert self.__pending_callbacks_counter > 0, (
                    "Pending callbacks counter cannot become negative."
                )
                if self.__pending_callbacks_counter == 1:
                    self.run_end_callbacks(args)
                self.__pending_callbacks_counter -= 1

    def clear(self) -> None:
```
- **EN**: Declares `CompilationCallbackHandler`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `CompilationCallbackHandler`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 143-154
```python
        """
        Clear all registered callbacks.
        """
        self.start_callbacks.clear()
        self.end_callbacks.clear()
        assert self.__pending_callbacks_counter == 0


callback_handler = CompilationCallbackHandler()


def on_compile_start(
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 155-166
```python
    callback: Callable[[CallbackArgs], None],
) -> Callable[[CallbackArgs], None]:
    """
    Decorator to register a callback function for the start of the compilation.
    """
    callback_handler.register_start_callback(callback)
    return callback


def on_compile_end(
    callback: Callable[[CallbackArgs], None],
) -> Callable[[CallbackArgs], None]:
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果。

### Lines 167-171
```python
    """
    Decorator to register a callback function for the end of the compilation.
    """
    callback_handler.register_end_callback(callback)
    return callback
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Python execution capture / Python 执行捕获**
  - EN: The file hooks or rewrites Python execution machinery to observe user programs.
  - CN: 该文件会挂接或改写 Python 执行机制，以观察用户程序。
## Dependencies / 依赖关系

- **Standard library / 标准库**: `enum`, `threading`, `collections.abc`, `contextlib`, `dataclasses`, `typing`
- **Primary symbols / 核心符号**: `CallbackTrigger`, `CallbackArgs`, `CompilationCallbackHandler`, `on_compile_start`, `on_compile_end`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
