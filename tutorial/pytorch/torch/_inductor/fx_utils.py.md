# fx_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `FakeTensorUpdater`. It exposes functions such as `matches_module_function_pattern`, `get_storage`, `get_node_storage`, `get_fake`, `get_fake_args_kwargs`, `is_node_realized`, and `...+2`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `FakeTensorUpdater` 等类。同时提供 `matches_module_function_pattern`、`get_storage`、`get_node_storage`、`get_fake`、`get_fake_args_kwargs`、`is_node_realized`、`另有2项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import contextlib
import operator
from collections import defaultdict
from collections.abc import Callable
from typing import Any

import sympy

import torch
import torch.fx
from torch._dispatch.python import enable_python_dispatcher
from torch._subclasses.fake_tensor import FakeTensorMode
from torch.fx.experimental.symbolic_shapes import (
    compute_unbacked_bindings,
    rebind_unbacked,
    statically_known_true,
    sym_eq,
)
from torch.utils import _pytree as pytree
````
- **EN**: Imports dependencies such as `contextlib`, `operator`, `collections`, `collections.abc`, `typing`, `sympy`, and `...+6` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `contextlib`、`operator`、`collections`、`collections.abc`、`typing`、`sympy`、`另有6项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。

### Lines 21-40 / 第 21-40 行
````python
from torch.utils._ordered_set import OrderedSet
from torch.utils._pytree import tree_map
from torch.utils.flop_counter import flop_registry

from .virtualized import V


# Check the pattern: (nn.module, F.function/torch.Tensor.method) matched.
# Works for length 2 patterns with 1 module and 1 function/method.
def matches_module_function_pattern(
    pattern: tuple[type[torch.nn.modules.Module], Callable[..., Any]],
    node: torch.fx.node.Node,
    modules: dict[str, torch.nn.modules.Module],
) -> bool:
    if len(node.args) == 0:
        return False
    if not isinstance(node.args[0], torch.fx.Node) or not isinstance(
        node, torch.fx.Node
    ):
        return False
````
- **EN**: Imports dependencies such as `torch.utils._ordered_set`, `torch.utils._pytree`, `torch.utils.flop_counter`, and `.virtualized` for the logic in this range. Introduces function `matches_module_function_pattern`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch.utils._ordered_set`、`torch.utils._pytree`、`torch.utils.flop_counter`、`.virtualized` 等依赖，为后续逻辑提供基础能力。这里定义了函数`matches_module_function_pattern`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 41-60 / 第 41-60 行
````python
    # the first node is call_module
    if node.args[0].op != "call_module":
        return False
    if not isinstance(node.args[0].target, str):
        return False
    if node.args[0].target not in modules:
        return False
    if type(modules[node.args[0].target]) is not pattern[0]:
        return False
    # the second node is call_function or call_method
    if node.op != "call_function" and node.op != "call_method":
        return False
    if node.target != pattern[1]:
        return False
    # make sure node.args[0] output is only used by current node.
    if len(node.args[0].users) > 1:
        return False
    return True


````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `matches_module_function_pattern`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`matches_module_function_pattern` 的具体实现。

### Lines 61-80 / 第 61-80 行
````python
class FakeTensorUpdater:
    """
    The main idea here is that it's difficult to maintain accurate fake
    tensors (our primary form of metadata) for each node in our graph as we
    transform it.

    The most reliable way to obtain this information is by rerunning
    faketensor propagation. However, in general, faketensor propagation is
    fairly expensive. So, instead we'd like to only rerun faketensor
    propagation on nodes that have changed.

    In order to detect which nodes have changed, we first hash its node,
    target, and argument lists (which are immutable in FX).

    Then, whenever we call incremental_update, we check which FX nodes have a
    new hash, and recompute the faketensor metadata for that node. Then, we
    continue to recursively compute the faketensors for all users until the
    fake tensors stop changing.
    """

````
- **EN**: Introduces class `FakeTensorUpdater`.
- **CN**: 这里定义了类`FakeTensorUpdater`。

### Lines 81-100 / 第 81-100 行
````python
    def __init__(self, graph: torch.fx.Graph) -> None:
        self.processed_hashes = OrderedSet[Any]()
        self.graph = graph

        for node in self.graph.nodes:
            self.processed_hashes.add(self.hash_node(node))

    def hash_node(self, node: torch.fx.Node):
        # todo(chilli): Not a great hash function
        return (node, node.target, id(node.args), id(node.kwargs))

    def incremental_update(self):
        """Update FakeTensors on self.graph. We will try to do the minimum amount of work."""
        existing_storages: defaultdict[int | None, int] = defaultdict(int)
        for node in self.graph.nodes:
            existing_storages[get_node_storage(node)] += 1

        def is_intlist_same(new, old):
            return statically_known_true(sym_eq(new, old))

````
- **EN**: Introduces function `__init__`, function `hash_node`, function `incremental_update`, function `is_intlist_same`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `existing_storages`.
- **CN**: 这里定义了函数`__init__`、函数`hash_node`、函数`incremental_update`、函数`is_intlist_same`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `existing_storages` 等值。

### Lines 101-120 / 第 101-120 行
````python
        def is_fake_tensor_same(new, old, *, node):
            if type(new) is not type(old):
                return False
            if isinstance(new, (list, tuple)):
                if len(new) != len(old):
                    return False
                return all(
                    is_fake_tensor_same(new_i, old_i, node=node)
                    for new_i, old_i in zip(new, old)
                )
            if new is None:
                return old is None
            if not isinstance(new, torch.Tensor):
                assert isinstance(new, (torch.SymInt, torch.SymBool, torch.SymFloat)), (
                    f"Unknown type {type(new)} in {self.graph}"
                )
                return (
                    new.node.shape_env._maybe_evaluate_static(
                        sympy.Eq(new.node.expr, old.node.expr)
                    )
````
- **EN**: Introduces function `is_fake_tensor_same`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`is_fake_tensor_same`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 121-140 / 第 121-140 行
````python
                    == sympy.true
                )
            if not is_intlist_same(new.shape, old.shape) or new.layout != old.layout:
                return False
            if new.layout == torch.strided and (
                not is_intlist_same(new.stride(), old.stride())
                or not statically_known_true(
                    new.storage_offset() == old.storage_offset()
                )
            ):
                return False

            if new.device != old.device:
                return False

            if get_storage(new) == get_storage(old):
                return True

            def any_user_may_alias(node):
                if not isinstance(node.meta["val"], torch.Tensor):
````
- **EN**: Introduces function `any_user_may_alias`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`any_user_may_alias`。借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 141-160 / 第 141-160 行
````python
                    # analysis too complicated on lists, can support in the future
                    return True
                for user in node.users:
                    if not (
                        isinstance(
                            user.target,
                            (torch._ops.OpOverload, torch._ops.HigherOrderOperator),
                        )
                        or user.target
                        is torch._inductor.fx_passes.reinplace._generalized_scatter
                    ):
                        return True
                    if isinstance(user.target, torch._ops.HigherOrderOperator):
                        # HOPs that survive until inductor are all non-aliasing HOPs.
                        # We will likely never support HOPs that are aliasing.
                        continue
                    # Strategy: do a FakeTensor prop, see if the storage aliases.
                    # If Inductor ever gets tighter invariants on OpOverloads
                    # (that is, we ban things like torch.ops.aten.reshape calls in the graph),
                    # Then this could just be a fast schema lookup.
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `FakeTensorUpdater.incremental_update.is_fake_tensor_same.any_user_may_alias`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`FakeTensorUpdater.incremental_update.is_fake_tensor_same.any_user_may_alias` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
                    is_valid, args, kwargs = get_fake_args_kwargs(user)
                    if not is_valid:
                        return True
                    with (
                        V.fake_mode,
                        enable_python_dispatcher(),
                        contextlib.ExitStack() as stack,
                    ):
                        # Ignore unbacked symbols (if they exist): we're making
                        # this FakeTensor and then throwing it away.
                        shape_env = V.fake_mode.shape_env
                        if shape_env is not None:
                            stack.enter_context(
                                shape_env.ignore_fresh_unbacked_symbols()
                            )
                        new_fake_tensor = user.target(*args, **kwargs)
                    if not isinstance(new_fake_tensor, torch.Tensor):
                        # analysis too complicated on lists, can support in the future
                        return True
                    if get_storage(new_fake_tensor) == get_storage(node.meta["val"]):
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `shape_env`, and `new_fake_tensor`. This range continues the implementation of function `FakeTensorUpdater.incremental_update.is_fake_tensor_same.any_user_may_alias`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `shape_env`、`new_fake_tensor` 等值。这一段延续了函数`FakeTensorUpdater.incremental_update.is_fake_tensor_same.any_user_may_alias` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
                        return True
                return False

            # This is the case where it returns a completely fresh storage that's used nowhere else.
            # If the FakeTensor's storage is fresh and none of the node's users can alias it, then
            # we don't need to update this node.
            if (
                existing_storages[get_storage(old)] == 1
                and get_storage(new) not in existing_storages
                and not any_user_may_alias(node)
            ):
                return True

            return False

        def should_process_node(node):
            # node.target for nodes returning true from this function
            # are called under fake mode and does not work for inductor
            # lowerings. We check if the node.target is an aten operator
            # or operator.getitem which is used when returning multiple
````
- **EN**: Introduces function `should_process_node`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`should_process_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 201-220 / 第 201-220 行
````python
            # tensors from an op.
            return node.op == "call_function" and (
                isinstance(node.target, torch._ops.OpOverload)
                or node.target is operator.getitem
                or node.target
                is torch._inductor.fx_passes.reinplace._generalized_scatter
            )

        to_process = OrderedSet[int]()
        for node in self.graph.nodes:
            # NB: Be very careful about skipping nodes (via continues) here
            # and ask for a careful review when changing this code. The
            # consequence for incorrect FakeTensor metadata is difficult-to-debug
            # silent incorrectness.
            if (
                self.hash_node(node) in self.processed_hashes
                and id(node) not in to_process
            ):
                continue

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `to_process`. This range continues the implementation of function `FakeTensorUpdater.incremental_update`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `to_process` 等值。这一段延续了函数`FakeTensorUpdater.incremental_update` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
            if not should_process_node(node):
                continue

            is_valid, args, kwargs = get_fake_args_kwargs(node)
            if not is_valid:
                continue
            with V.fake_mode, enable_python_dispatcher():
                new_fake_tensor = node.target(*args, **kwargs)

            if "val" in node.meta and is_fake_tensor_same(
                new_fake_tensor, node.meta["val"], node=node
            ):
                continue

            rebind_unbacked(V.fake_mode.shape_env, node, new_fake_tensor)

            node.meta["val"] = new_fake_tensor
            if (shape_env := V.fake_mode.shape_env) and (
                symbol_to_path := compute_unbacked_bindings(shape_env, new_fake_tensor)
            ):
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_fake_tensor`, and `symbol_to_path`. This range continues the implementation of function `FakeTensorUpdater.incremental_update`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `new_fake_tensor`、`symbol_to_path` 等值。这一段延续了函数`FakeTensorUpdater.incremental_update` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python
                # Refresh the bindings to the new symbols

                node.meta["unbacked_bindings"] = symbol_to_path

            existing_storages[get_node_storage(node)] += 1

            to_process.update([id(user) for user in node.users])

            self.processed_hashes.add(self.hash_node(node))


def get_storage(t: torch.Tensor) -> int:
    return t.untyped_storage()._cdata


def get_node_storage(node: torch.fx.Node) -> int | None:
    if "val" not in node.meta:
        return None
    if not isinstance(node.meta["val"], torch.Tensor):
        return None
````
- **EN**: Introduces function `get_storage`, function `get_node_storage`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_storage`、函数`get_node_storage`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 261-280 / 第 261-280 行
````python
    if not torch._C._has_storage(node.meta["val"]):
        return None
    return get_storage(node.meta["val"])


def get_fake(x):
    if isinstance(x, torch.fx.Node):
        if "val" not in x.meta:
            return x
        return x.meta["val"]
    return x


def get_fake_args_kwargs(x: torch.fx.Node) -> tuple[bool, tuple[Any], dict[str, Any]]:
    """
    First value returns a boolean if any of the input nodes don't have a faketensor.
    """
    args, kwargs = tree_map(get_fake, (x.args, x.kwargs))
    if any(
        isinstance(a, torch.fx.Node) for a in pytree.arg_tree_leaves(*args, **kwargs)
````
- **EN**: Introduces function `get_fake`, function `get_fake_args_kwargs`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_fake`、函数`get_fake_args_kwargs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 281-300 / 第 281-300 行
````python
    ):
        return False, args, kwargs
    return True, args, kwargs


def is_node_realized(node: torch.fx.Node) -> bool:
    """Returns true if a node is always realized when lowered to inductor IR.

    NOTE: This may return some false negatives. e.g. it doesn't
    handle buffers realized heuristically during lowering, or
    buffers realized indirectly through view ops.
    """
    from torch._inductor.lowering import fallbacks, needs_realized_inputs

    def is_buffer(node: torch.fx.Node) -> bool:
        if node.op == "call_function" and node.target is operator.getitem:
            # For nodes with multiple outputs, we get the fx graph:
            #     foo = torch.ops.aten.foo(...)
            #     getitem = foo[0]
            #     getitem_1 = foo[1]
````
- **EN**: Imports dependencies such as `torch._inductor.lowering` for the logic in this range. Introduces function `is_node_realized`, function `is_buffer`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.lowering` 等依赖，为后续逻辑提供基础能力。这里定义了函数`is_node_realized`、函数`is_buffer`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 301-320 / 第 301-320 行
````python
            # where we need to check if foo is a fallback kernel
            return is_buffer(node.args[0])  # type: ignore[arg-type]
        return node.op in ("placeholder", "output") or node.target in fallbacks

    if is_buffer(node):
        return True

    def realizes_inputs(node: torch.fx.Node) -> bool:
        return node.op == "output" or node.target in needs_realized_inputs

    if any(realizes_inputs(user) for user in node.users):
        return True

    # Otherwise, assume node isn't realized
    return False


def count_flops_fx(node: torch.fx.Node) -> int | None:
    if not countable_fx(node) or isinstance(node.target, str):
        return None
````
- **EN**: Introduces function `realizes_inputs`, function `count_flops_fx`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`realizes_inputs`、函数`count_flops_fx`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 321-340 / 第 321-340 行
````python
    with FakeTensorMode(allow_non_fake_inputs=True):
        success, args, kwargs = get_fake_args_kwargs(node)

        if success:
            with torch.utils.flop_counter.FlopCounterMode(
                display=False
            ) as flop_counter_mode:
                node.target(*args, **kwargs)

            counted_flops = flop_counter_mode.get_total_flops()
            return counted_flops
    return None


def countable_fx(node: torch.fx.Node) -> bool:
    """
    Whether or not we can count the flops of an FX node.
    """
    assert isinstance(node, torch.fx.Node)
    if not hasattr(node, "target"):
````
- **EN**: Introduces function `countable_fx`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `display`, and `counted_flops`.
- **CN**: 这里定义了函数`countable_fx`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `display`、`counted_flops` 等值。

### Lines 341-346 / 第 341-346 行
````python
        return False
    target = node.target
    if not hasattr(target, "overloadpacket"):
        return target in flop_registry
    packet = target.overloadpacket
    return packet in flop_registry
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `target`, and `packet`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `target`、`packet` 等值。

## Key Concepts / 关键概念
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Primary classes: `FakeTensorUpdater`  
  **CN**: 主要类：`FakeTensorUpdater`
- **EN**: Primary functions: `matches_module_function_pattern`, `get_storage`, `get_node_storage`, `get_fake`, `get_fake_args_kwargs`, `is_node_realized`, and `...+2`  
  **CN**: 主要函数：`matches_module_function_pattern`、`get_storage`、`get_node_storage`、`get_fake`、`get_fake_args_kwargs`、`is_node_realized`、`另有2项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `operator`, `collections`, `collections.abc`, `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `torch._dispatch.python`, `torch._subclasses.fake_tensor`, `torch.fx.experimental.symbolic_shapes`, `torch.utils`, `torch.utils._ordered_set`, `torch.utils._pytree`, `torch.utils.flop_counter`, `.virtualized`, `torch._inductor.lowering`
