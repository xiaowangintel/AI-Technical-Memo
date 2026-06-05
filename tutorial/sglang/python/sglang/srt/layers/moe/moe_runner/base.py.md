# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `moe_output_buffer_ctx`, `MoeRunnerConfig`, `RunnerInput`, and `RunnerOutput` and connects them to backend-specific paths such as `Triton` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `moe_output_buffer_ctx`、`MoeRunnerConfig`、`RunnerInput` 以及 `RunnerOutput` 等符号，并把这些符号连接到 `Triton` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import contextvars
from abc import ABC, abstractmethod
from contextlib import contextmanager
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, Callable, Generator, Optional, Tuple, TypeGuard

import torch

from sglang.srt.layers.moe.utils import (
    MoeA2ABackend,
    MoeRunnerBackend,
    RoutingMethodType,
)

if TYPE_CHECKING:
    from sglang.srt.layers.moe.moe_runner.triton import (
        TritonRunnerCore,
        TritonRunnerInput,
        TritonRunnerOutput,
    )
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        CombineInputFormat,
        DispatchOutput,
        DispatchOutputFormat,
    )


_moe_output_buf: contextvars.ContextVar[Optional[torch.Tensor]] = (
    contextvars.ContextVar("moe_output_buf", default=None)
)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `contextvars`, `abc.ABC`, `abc.abstractmethod`, `contextlib.contextmanager`, and `dataclasses.dataclass`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_moe_output_buf` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`contextvars`、`abc.ABC`、`abc.abstractmethod`、`contextlib.contextmanager` 以及 `dataclasses.dataclass`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_moe_output_buf` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 36-44: Function `moe_output_buffer_ctx` and its core logic
```python
@contextmanager
def moe_output_buffer_ctx(buf: torch.Tensor) -> Generator[None, None, None]:
    token = _moe_output_buf.set(buf)
    try:
        yield
    finally:
        _moe_output_buf.reset(token)
```
**EN:** This block defines `moe_output_buffer_ctx` and contains the main logic for this step. Decorators like `contextmanager` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `_moe_output_buf.set` and `_moe_output_buf.reset`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `token` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `moe_output_buffer_ctx`，并承载这一阶段的核心逻辑。 像 `contextmanager` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `_moe_output_buf.set` 和 `_moe_output_buf.reset`，说明该流程会编排底层辅助函数或计算内核。 像 `token` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 45-69: Class `MoeRunnerConfig` declaration and shared state
```python
@dataclass
class MoeRunnerConfig:
    # MoE parameters
    num_experts: Optional[int] = None
    num_local_experts: Optional[int] = None
    hidden_size: Optional[int] = None
    intermediate_size_per_partition: Optional[int] = None
    layer_id: Optional[int] = None
    top_k: Optional[int] = None
    num_fused_shared_experts: Optional[int] = None
    params_dtype: Optional[torch.dtype] = None
    routing_method_type: Optional[RoutingMethodType] = None

    # Runner configuration
    activation: str = "silu"
    is_gated: bool = True
    apply_router_weight_on_input: bool = False
    inplace: bool = True
    no_combine: bool = False
    routed_scaling_factor: Optional[float] = None
    gemm1_alpha: Optional[float] = None
    gemm1_clamp_limit: Optional[float] = None
    swiglu_limit: Optional[float] = None
```
**EN:** This block introduces class `MoeRunnerConfig` and the state shared by its methods.
**CN:** 该代码块引入类 `MoeRunnerConfig`，并定义其方法共享的状态。

### Lines 70-71: Class `RunnerInput` declaration and shared state
```python
@dataclass
class RunnerInput(ABC):
```
**EN:** This block introduces class `RunnerInput` and the state shared by its methods. It inherits from `ABC`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `RunnerInput`，并定义其方法共享的状态。 它继承自 `ABC`，说明了它在 SRT 层栈中的接入方式。

### Lines 72-75: Function `RunnerInput.runner_backend` and its core logic
```python
    @property
    @abstractmethod
    def runner_backend(self) -> MoeRunnerBackend: ...
```
**EN:** This block defines `RunnerInput.runner_backend` and contains the main logic for this step. Decorators like `property` and `abstractmethod` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `RunnerInput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 和 `abstractmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 76-79: Function `RunnerInput.runner_backend_is_triton` and its core logic
```python
    def runner_backend_is_triton(self) -> TypeGuard[TritonRunnerInput]:
        return self.runner_backend == MoeRunnerBackend.TRITON
```
**EN:** This block defines `RunnerInput.runner_backend_is_triton` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RunnerInput.runner_backend_is_triton`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 80-80: Class `RunnerOutput` declaration and shared state
```python
class RunnerOutput(ABC):
```
**EN:** This block introduces class `RunnerOutput` and the state shared by its methods. It inherits from `ABC`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `RunnerOutput`，并定义其方法共享的状态。 它继承自 `ABC`，说明了它在 SRT 层栈中的接入方式。

### Lines 81-84: Function `RunnerOutput.runner_backend` and its core logic
```python
    @property
    @abstractmethod
    def runner_backend(self) -> MoeRunnerBackend: ...
```
**EN:** This block defines `RunnerOutput.runner_backend` and contains the main logic for this step. Decorators like `property` and `abstractmethod` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `RunnerOutput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 和 `abstractmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 85-88: Function `RunnerOutput.runner_backend_is_triton` and its core logic
```python
    def runner_backend_is_triton(self) -> TypeGuard[TritonRunnerOutput]:
        return self.runner_backend == MoeRunnerBackend.TRITON
```
**EN:** This block defines `RunnerOutput.runner_backend_is_triton` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RunnerOutput.runner_backend_is_triton`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 89-95: Class `MoeQuantInfo` declaration and shared state
```python
@dataclass
class MoeQuantInfo(ABC):
    """Moe quantization data."""

    pass
```
**EN:** This block introduces class `MoeQuantInfo` and the state shared by its methods. It inherits from `ABC`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Moe quantization data.
**CN:** 该代码块引入类 `MoeQuantInfo`，并定义其方法共享的状态。 它继承自 `ABC`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 96-96: Class `MoeRunnerCore` declaration and shared state
```python
class MoeRunnerCore(ABC):
```
**EN:** This block introduces class `MoeRunnerCore` and the state shared by its methods. It inherits from `ABC`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `MoeRunnerCore`，并定义其方法共享的状态。 它继承自 `ABC`，说明了它在 SRT 层栈中的接入方式。

### Lines 97-99: `MoeRunnerCore` initialization and state setup
```python
    def __init__(self, config: MoeRunnerConfig):
        self.config = config
```
**EN:** This block defines `MoeRunnerCore.__init__` and contains the main logic for this step. Intermediate names such as `self.config` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MoeRunnerCore.__init__`，并承载这一阶段的核心逻辑。 像 `self.config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 100-109: Function `MoeRunnerCore.run` and its core logic
```python
    @abstractmethod
    def run(
        self,
        runner_input: RunnerInput,
        quant_info: MoeQuantInfo,
        running_state: dict,
        hooks: Optional[Any] = None,
    ) -> RunnerOutput:
        pass
```
**EN:** This block defines `MoeRunnerCore.run` and contains the main logic for this step. Decorators like `abstractmethod` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `MoeRunnerCore.run`，并承载这一阶段的核心逻辑。 像 `abstractmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 110-113: Function `MoeRunnerCore.runner_backend` and its core logic
```python
    @property
    @abstractmethod
    def runner_backend(self) -> MoeRunnerBackend: ...
```
**EN:** This block defines `MoeRunnerCore.runner_backend` and contains the main logic for this step. Decorators like `property` and `abstractmethod` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `MoeRunnerCore.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 和 `abstractmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 114-117: Function `MoeRunnerCore.runner_backend_is_triton` and its core logic
```python
    def runner_backend_is_triton(self) -> TypeGuard[TritonRunnerCore]:
        return self.runner_backend == MoeRunnerBackend.TRITON
```
**EN:** This block defines `MoeRunnerCore.runner_backend_is_triton` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerCore.runner_backend_is_triton`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 118-120: Class `FusedOpPool` declaration and shared state
```python
class FusedOpPool:
    _fused_funcs: dict[str, Callable] = {}
```
**EN:** This block introduces class `FusedOpPool` and the state shared by its methods.
**CN:** 该代码块引入类 `FusedOpPool`，并定义其方法共享的状态。

### Lines 121-137: Function `FusedOpPool.register_fused_func` and its core logic
```python
    @classmethod
    def register_fused_func(
        cls, a2a_backend_name: str, runner_backend_name: str, fused_func: Callable
    ):
        key = (a2a_backend_name, runner_backend_name)
        if key in cls._fused_funcs:
            raise ValueError(
                f"Fused function for {a2a_backend_name} to {runner_backend_name} is already registered."
            )
        assert MoeA2ABackend(
            a2a_backend_name
        ), f"Invalid dispatch name: {a2a_backend_name}"
        assert MoeRunnerBackend(
            runner_backend_name
        ), f"Invalid runner name: {runner_backend_name}"
        cls._fused_funcs[key] = fused_func
```
**EN:** This block defines `FusedOpPool.register_fused_func` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `MoeA2ABackend`, `MoeRunnerBackend`, and `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `FusedOpPool.register_fused_func`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `MoeA2ABackend`、`MoeRunnerBackend` 以及 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `key` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 138-144: `FusedOpPool.get_fused_func` getter for fused operations func
```python
    @classmethod
    def get_fused_func(cls, dispatch_name: str, runner_name: str) -> Optional[Callable]:
        key = (dispatch_name, runner_name)
        fused_func = cls._fused_funcs.get(key)
        return fused_func
```
**EN:** This block defines `FusedOpPool.get_fused_func` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._fused_funcs.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key` and `fused_func` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FusedOpPool.get_fused_func`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._fused_funcs.get`，说明该流程会编排底层辅助函数或计算内核。 像 `key` 和 `fused_func` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 145-152: Class `PermuteMethodPool` declaration and shared state
```python
class PermuteMethodPool:
    _pre_permute_methods: dict[
        Tuple[DispatchOutputFormat, MoeRunnerBackend], Callable
    ] = {}
    _post_permute_methods: dict[
        Tuple[MoeRunnerBackend, CombineInputFormat], Callable
    ] = {}
```
**EN:** This block introduces class `PermuteMethodPool` and the state shared by its methods.
**CN:** 该代码块引入类 `PermuteMethodPool`，并定义其方法共享的状态。

### Lines 153-174: Function `PermuteMethodPool.register_pre_permute` and its core logic
```python
    @classmethod
    def register_pre_permute(
        cls,
        dispatch_output_name: str,
        runner_backend_name: str,
        permute_func: Callable,
    ):
        """
        Register a customized pre-permute function for the given DispatchOutputFormat and MoeRunnerBackend.

        :param dispatch_output_name: The DispatchOutputFormat name.
        :param runner_backend_name: The MoeRunnerBackend name.
        :param permute_func: The permute function to register.
        """
        # TODO: check if registration is valid
        key = (dispatch_output_name, runner_backend_name)
        if key in cls._pre_permute_methods:
            raise ValueError(
                f"Pre-permute method for {dispatch_output_name} to {runner_backend_name} is already registered."
            )
        cls._pre_permute_methods[key] = permute_func
```
**EN:** This block defines `PermuteMethodPool.register_pre_permute` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `PermuteMethodPool.register_pre_permute`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `key` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 175-196: Function `PermuteMethodPool.register_post_permute` and its core logic
```python
    @classmethod
    def register_post_permute(
        cls,
        runner_backend_name: str,
        combine_input_name: str,
        permute_func: Callable,
    ):
        """
        Register a customized post-permute function for the given MoeRunnerBackend and CombineInputFormat.

        :param runner_backend_name: The MoeRunnerBackend name.
        :param combine_input_name: The CombineInputFormat name.
        :param permute_func: The permute function to register.
        """
        # TODO: check if registration is valid
        key = (runner_backend_name, combine_input_name)
        if key in cls._post_permute_methods:
            raise ValueError(
                f"Post-permute method for {runner_backend_name} to {combine_input_name} is already registered."
            )
        cls._post_permute_methods[key] = permute_func
```
**EN:** This block defines `PermuteMethodPool.register_post_permute` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `PermuteMethodPool.register_post_permute`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `key` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 197-216: `PermuteMethodPool.get_pre_permute` getter for pre permute
```python
    @classmethod
    def get_pre_permute(
        cls,
        dispatch_output_format: DispatchOutputFormat,
        runner_input_format: MoeRunnerBackend,
    ) -> Callable:
        """
        Retrieve the pre-permute function for the given DispatchOutputFormat and MoeRunnerBackend.

        :param dispatch_output_format: The DispatchOutputFormat type.
        :param runner_input_format: The MoeRunnerBackend type.
        :return: The registered permute function or None if not found.
        """
        key = (dispatch_output_format, runner_input_format)
        pre_permute_func = cls._pre_permute_methods.get(key)
        assert (
            pre_permute_func is not None
        ), f"Pre-permute function for {dispatch_output_format} to {runner_input_format} is not registered"
        return pre_permute_func
```
**EN:** This block defines `PermuteMethodPool.get_pre_permute` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._pre_permute_methods.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key` and `pre_permute_func` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PermuteMethodPool.get_pre_permute`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._pre_permute_methods.get`，说明该流程会编排底层辅助函数或计算内核。 像 `key` 和 `pre_permute_func` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 217-237: `PermuteMethodPool.get_post_permute` getter for post permute
```python
    @classmethod
    def get_post_permute(
        cls,
        runner_output_format: MoeRunnerBackend,
        combine_input_format: CombineInputFormat,
    ) -> Callable:
        """
        Retrieve the post-permute function for the given MoeRunnerBackend and CombineInputFormat.

        :param runner_output_format: The MoeRunnerBackend type.
        :param combine_input_format: The CombineInputFormat type.
        :return: The registered permute function or None if not found.
        """
        key = (runner_output_format, combine_input_format)
        post_permute_func = cls._post_permute_methods.get(key)
        assert (
            post_permute_func is not None
        ), f"Post-permute function for {runner_output_format} to {combine_input_format} is not registered"
        return post_permute_func
```
**EN:** This block defines `PermuteMethodPool.get_post_permute` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._post_permute_methods.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key` and `post_permute_func` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PermuteMethodPool.get_post_permute`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._post_permute_methods.get`，说明该流程会编排底层辅助函数或计算内核。 像 `key` 和 `post_permute_func` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 238-258: Function `register_fused_func` and its core logic
```python
def register_fused_func(
    a2a_backend_name: str,
    runner_backend_name: str,
) -> Callable:
    """
    Decorator to register a fused function for the given DispatchOutputFormat and MoeRunnerBackend.

    :param a2a_backend_name: The A2A backend name.
    :param runner_backend_name: The MoeRunnerBackend name.
    :return: The decorator function.
    """

    def decorator(fused_func: Callable):
        FusedOpPool.register_fused_func(
            a2a_backend_name, runner_backend_name, fused_func
        )
        return fused_func

    return decorator
```
**EN:** This block defines `register_fused_func` and contains the main logic for this step. It mainly invokes `FusedOpPool.register_fused_func`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `register_fused_func`，并承载这一阶段的核心逻辑。 它主要调用 `FusedOpPool.register_fused_func`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 259-283: Function `register_pre_permute` and its core logic
```python
def register_pre_permute(
    dispatch_output_name: str,
    runner_backend_name: str,
) -> Callable:
    """
    Decorator to register a pre-permute function for the given DispatchOutputFormat and MoeRunnerBackend.

    :param dispatch_output_name: The DispatchOutputFormat name.
    :param runner_backend_name: The MoeRunnerBackend name.
    :return: The decorator function.
    """

    def decorator(
        permute_func: Callable[
            [DispatchOutput, MoeQuantInfo, MoeRunnerConfig, dict], RunnerInput
        ],
    ) -> Callable:
        PermuteMethodPool.register_pre_permute(
            dispatch_output_name, runner_backend_name, permute_func
        )
        return permute_func

    return decorator
```
**EN:** This block defines `register_pre_permute` and contains the main logic for this step. It mainly invokes `PermuteMethodPool.register_pre_permute`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `register_pre_permute`，并承载这一阶段的核心逻辑。 它主要调用 `PermuteMethodPool.register_pre_permute`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 284-306: Function `register_post_permute` and its core logic
```python
def register_post_permute(
    runner_backend_name: str,
    combine_input_name: str,
) -> Callable:
    """
    Decorator to register a post-permute function for the given MoeRunnerBackend and CombineInputFormat.

    :param runner_backend_name: The MoeRunnerBackend name.
    :param combine_input_name: The CombineInputFormat name.
    :return: The decorator function.
    """

    def decorator(
        permute_func: Callable[
            [RunnerOutput, MoeQuantInfo, MoeRunnerConfig, dict], CombineInput
        ],
    ) -> Callable:
        PermuteMethodPool.register_post_permute(
            runner_backend_name, combine_input_name, permute_func
        )
        return permute_func

    return decorator
```
**EN:** This block defines `register_post_permute` and contains the main logic for this step. It mainly invokes `PermuteMethodPool.register_post_permute`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `register_post_permute`，并承载这一阶段的核心逻辑。 它主要调用 `PermuteMethodPool.register_post_permute`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `moe_output_buffer_ctx`, `MoeRunnerConfig`, `RunnerInput`, `RunnerOutput`, and `MoeQuantInfo`. / **主要符号**：核心入口包括 `moe_output_buffer_ctx`、`MoeRunnerConfig`、`RunnerInput`、`RunnerOutput` 以及 `MoeQuantInfo`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `contextvars`, `abc.ABC`, `abc.abstractmethod`, `contextlib.contextmanager`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.Callable`, and `typing.Generator` / **标准库**：`__future__.annotations`、`contextvars`、`abc.ABC`、`abc.abstractmethod`、`contextlib.contextmanager`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.Callable` 以及 `typing.Generator`
- **Third-party**: `torch` / **第三方依赖**：`torch`
- **Internal SGLang modules**: `sglang.srt.layers.moe.utils.MoeA2ABackend`, `sglang.srt.layers.moe.utils.MoeRunnerBackend`, `sglang.srt.layers.moe.utils.RoutingMethodType`, `sglang.srt.layers.moe.moe_runner.triton.TritonRunnerCore`, `sglang.srt.layers.moe.moe_runner.triton.TritonRunnerInput`, `sglang.srt.layers.moe.moe_runner.triton.TritonRunnerOutput`, `sglang.srt.layers.moe.token_dispatcher.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.CombineInputFormat`, `sglang.srt.layers.moe.token_dispatcher.DispatchOutput`, and `sglang.srt.layers.moe.token_dispatcher.DispatchOutputFormat` / **SGLang 内部模块**：`sglang.srt.layers.moe.utils.MoeA2ABackend`、`sglang.srt.layers.moe.utils.MoeRunnerBackend`、`sglang.srt.layers.moe.utils.RoutingMethodType`、`sglang.srt.layers.moe.moe_runner.triton.TritonRunnerCore`、`sglang.srt.layers.moe.moe_runner.triton.TritonRunnerInput`、`sglang.srt.layers.moe.moe_runner.triton.TritonRunnerOutput`、`sglang.srt.layers.moe.token_dispatcher.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.CombineInputFormat`、`sglang.srt.layers.moe.token_dispatcher.DispatchOutput` 以及 `sglang.srt.layers.moe.token_dispatcher.DispatchOutputFormat`
