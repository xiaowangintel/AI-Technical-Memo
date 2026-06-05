# distributed.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/variables/distributed.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Defines abstract variable trackers that model Python values during graph capture.
- **Purpose (CN)**: 定义抽象的变量跟踪器，用于在图捕获过程中建模 Python 值。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
"""
Distributed computing variable tracking classes for PyTorch Dynamo.

This module implements variable tracking for distributed computing components:
- Process Groups (for collective communication)
- Device Meshes (for distributed tensor sharding)
- Placement Types (for specifying distribution strategies)
- Distributed Tensors and their operations
- Backward hooks for distributed module operations

These classes are responsible for tracking distributed operations during graph
compilation while maintaining proper guards and handling distributed-specific
behaviors. They ensure correct handling of distributed components like process
groups, device meshes, and placement strategies while preserving proper semantics
for distributed tensor operations in the compiled code.
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 17-28
```python
The implementation provides special handling for distributed package availability
checks and proper tracking of distributed state and operations across processes.
"""

import functools
import inspect
from typing import Any, Literal, TYPE_CHECKING

import torch
from torch.fx.experimental._backward_state import BackwardState

from .. import compiled_autograd
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。

### Lines 29-40
```python
from .._trace_wrapped_higher_order_op import trace_wrapped
from ..exc import unimplemented
from ..external_utils import call_module_hooks_from_backward_state
from ..guards import GuardBuilder, install_guard
from ..source import AttrSource
from .base import VariableTracker


if TYPE_CHECKING:
    from torch._dynamo.symbolic_convert import InstructionTranslator
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 41-52
```python
class DistributedVariable(VariableTracker):
    """
    The base distributed variable that encapsulates common methods
    for the distributed objects (i.e. ProcessGroup, DeviceMesh, etc.).
    Concrete distributed objects could inherit this class and add object
    specific logic.

    i.e. It provides the check on the distributed package existence
    and hold the tracking value for the corresponding distributed object.
    """

    def __init__(self, value: Any, **kwargs: Any) -> None:
```
- **EN**: Declares `DistributedVariable(VariableTracker)`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `DistributedVariable(VariableTracker)`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 53-67
```python
        super().__init__(**kwargs)
        if not DistributedVariable.is_available():
            unimplemented(
                gb_type="torch.distributed package is not available!",
                context="",
                explanation="The PyTorch package doesn't include torch.distributed when building from source.",
                hints=[
                    "Set USE_DISTRIBUTED=1 to enable it when building PyTorch from source."
                ],
            )
        self.value = value

    def python_type(self) -> type:
        return type(self.value)
```
- **EN**: Declares `DistributedVariable(VariableTracker)`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `DistributedVariable(VariableTracker)`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 68-85
```python
    @staticmethod
    def is_available() -> bool:
        # check if the distributed package is available or not
        return torch.distributed.is_available()

    def is_python_hashable(self) -> Literal[True]:
        return True

    def get_python_hash(self) -> int:
        return hash(self.value)

    def is_python_equal(self, other: object) -> bool:
        return (
            isinstance(other, VariableTracker)
            and self.as_python_constant() == other.as_python_constant()
        )
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 86-97
```python
def is_from_local(value: object) -> bool:
    if not DistributedVariable.is_available():
        return False
    from torch.distributed.tensor import DTensor

    return inspect.isfunction(value) and value is DTensor.from_local


def is_constant_pg_functions(value: object) -> bool:
    if not DistributedVariable.is_available():
        return False
```
- **EN**: This module-level block helps capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 98-115
```python
    from torch.distributed.distributed_c10d import (
        _get_group_size_by_name,
        _get_group_tag,
        _rank_not_in_group,
        _resolve_group_name_by_ranks_and_tag,
        get_process_group_ranks,
    )

    constant_processgroup_functions = [
        _get_group_size_by_name,
        _get_group_tag,
        _rank_not_in_group,
        get_process_group_ranks,
        _resolve_group_name_by_ranks_and_tag,
    ]

    return inspect.isfunction(value) and value in constant_processgroup_functions
```
- **EN**: This module-level block helps capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果。

### Lines 116-131
```python

class WorldMetaClassVariable(DistributedVariable):
    """
    Tracks torch.distributed.GroupMember and torch.distributed.group, which are
    instances of the metaclass _WorldMeta.
    """

    @classmethod
    def is_group_member_type(cls, value: object) -> bool:
        if not cls.is_available():
            return False

        from torch.distributed.distributed_c10d import _WorldMeta

        return type(value) is _WorldMeta
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 132-149
```python
    def python_type(self) -> type:
        return type(self.value)

    def var_getattr(self, tx: "InstructionTranslator", name: str) -> VariableTracker:
        if name == "WORLD":
            from .builder import SourcelessBuilder

            assert self.source
            source = AttrSource(base=self.source, member="WORLD")
            install_guard(source.make_guard(GuardBuilder.ID_MATCH))
            return SourcelessBuilder.create(tx, self.value.WORLD)
        elif name == "NON_GROUP_MEMBER":
            assert self.source
            source = AttrSource(base=self.source, member="NON_GROUP_MEMBER")
            install_guard(source.make_guard(GuardBuilder.ID_MATCH))
            return VariableTracker.build(tx, self.value.NON_GROUP_MEMBER)
        return super().var_getattr(tx, name)
```
- **EN**: This module-level block helps enforce guards that validate whether cached compiled code can be reused. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于实施守卫检查以判断缓存的编译代码能否复用。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 150-161
```python

class BackwardHookVariable(VariableTracker):
    """
    Handles torch.utils.hooks.BackwardHook for module-level backward
    hooks.
    """

    @staticmethod
    def create(
        tx: "InstructionTranslator",
        module: VariableTracker,
        user_hooks: VariableTracker,
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 162-173
```python
        user_pre_hooks: VariableTracker,
    ) -> "BackwardHookVariable":
        if not compiled_autograd.compiled_autograd_enabled:
            unimplemented(
                gb_type="Module-level backwards hooks require compiled autograd.",
                context="",
                explanation="",
                hints=[
                    "Enable compiled autograd by setting torch._dynamo.config.compiled_autograd = True."
                ],
            )
```
- **EN**: This block continues `BackwardHookVariable.create` and works to hand work to a compiler/backend pipeline. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `BackwardHookVariable.create`，用于将工作移交给编译器或后端流水线。 同时它还会根据运行时条件分支处理。

### Lines 174-185
```python
        def _in_graph_bw_hooks(
            bw_state: BackwardState,
        ) -> torch.utils.hooks.BackwardHook:
            """
            Rather than installing the user hooks in the graph (which
            don't survive AotAutograd), we install hooks that will call
            trace_wrapped in the backward pass that CompiledAutograd
            can turn into actual hook calls.
            """
            return torch.utils.hooks.BackwardHook(
                None,
                (
```
- **EN**: Defines the `BackwardHookVariable.create._in_graph_bw_hooks` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`BackwardHookVariable.create._in_graph_bw_hooks` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 186-197
```python
                    functools.partial(
                        trace_wrapped,
                        fn=call_module_hooks_from_backward_state,
                        bw_state=bw_state,
                        hooks_name=user_hooks_name,
                        module_name=module_name,
                    ),
                ),
                (
                    functools.partial(
                        trace_wrapped,
                        fn=call_module_hooks_from_backward_state,
```
- **EN**: This block continues `BackwardHookVariable.create._in_graph_bw_hooks` and works to trace Python execution into an intermediate graph representation.
- **CN**: 该代码块继续实现 `BackwardHookVariable.create._in_graph_bw_hooks`，用于将 Python 执行过程跟踪为中间图表示。

### Lines 198-209
```python
                        bw_state=bw_state,
                        hooks_name=user_pre_hooks_name,
                        module_name=module_name,
                    ),
                ),
            )

        module_name, bw_state_proxy = tx.output.add_backward_state_hook(module, "mod")
        user_pre_hooks_name, _ = tx.output.add_backward_state_hook(user_pre_hooks)
        user_hooks_name, _ = tx.output.add_backward_state_hook(user_hooks)
        proxy = tx.output.create_proxy(
            "call_function",
```
- **EN**: This block continues `BackwardHookVariable.create` and works to proxy tensor-like values through symbolic execution helpers.
- **CN**: 该代码块继续实现 `BackwardHookVariable.create`，用于借助符号执行辅助逻辑代理类张量值。

### Lines 210-221
```python
            _in_graph_bw_hooks,
            (bw_state_proxy,),
            {},
        )
        proxy.node.meta["example_value"] = torch.utils.hooks.BackwardHook(None, (), ())
        return BackwardHookVariable(proxy, module, user_hooks, user_pre_hooks)

    def __init__(
        self,
        proxy: torch.fx.Proxy,
        module: VariableTracker,
        user_hooks: VariableTracker,
```
- **EN**: Declares `BackwardHookVariable(VariableTracker)`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `BackwardHookVariable(VariableTracker)`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 222-233
```python
        user_pre_hooks: VariableTracker,
        **options: Any,
    ) -> None:
        super().__init__(**options)
        self.proxy = proxy
        self.module = module
        self.user_hooks = user_hooks
        self.user_pre_hooks = user_pre_hooks

    def as_proxy(self) -> torch.fx.Proxy:
        return self.proxy
```
- **EN**: Declares `BackwardHookVariable(VariableTracker)`; this class packages state and methods that proxy tensor-like values through symbolic execution helpers.
- **CN**: 声明 `BackwardHookVariable(VariableTracker)`；该类封装了状态与方法，用于借助符号执行辅助逻辑代理类张量值。

### Lines 234-249
```python
    def call_method(
        self,
        tx: "InstructionTranslator",
        name: str,
        args: list[VariableTracker],
        kwargs: dict[str, VariableTracker],
    ) -> VariableTracker:
        if name in ("setup_input_hook", "setup_output_hook"):
            return self._setup_hook(tx, name, *args, **kwargs)
        return super().call_method(tx, name, args, kwargs)

    def _setup_hook(
        self, tx: "InstructionTranslator", hook_method_name: str, args: VariableTracker
    ) -> VariableTracker:
        from .builder import wrap_fx_proxy
```
- **EN**: Declares `BackwardHookVariable(VariableTracker)`; this class packages state and methods that proxy tensor-like values through symbolic execution helpers.
- **CN**: 声明 `BackwardHookVariable(VariableTracker)`；该类封装了状态与方法，用于借助符号执行辅助逻辑代理类张量值。

### Lines 250-258
```python
        return wrap_fx_proxy(
            tx,
            tx.output.create_proxy(
                "call_method",
                hook_method_name,
                (self.as_proxy(), args.as_proxy()),
                {},
            ),
        )
```
- **EN**: This block continues `BackwardHookVariable._setup_hook` and works to proxy tensor-like values through symbolic execution helpers. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `BackwardHookVariable._setup_hook`，用于借助符号执行辅助逻辑代理类张量值。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Guards / 守卫**
  - EN: Runtime predicates preserve correctness when cached compiled graphs are reused.
  - CN: 运行时谓词用于在复用缓存编译图时保持正确性。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx.experimental._backward_state`, `..`, `.._trace_wrapped_higher_order_op`, `..exc`, `..external_utils`, `..guards`, `..source`, `.base`, `torch._dynamo.symbolic_convert`, `torch.distributed.tensor`, `torch.distributed.distributed_c10d`, `.builder`
- **Standard library / 标准库**: `functools`, `inspect`, `typing`
- **Primary symbols / 核心符号**: `DistributedVariable`, `is_from_local`, `is_constant_pg_functions`, `WorldMetaClassVariable`, `BackwardHookVariable`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
