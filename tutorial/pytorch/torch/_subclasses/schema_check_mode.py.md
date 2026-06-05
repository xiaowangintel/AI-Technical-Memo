# schema_check_mode.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_subclasses/schema_check_mode.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements tensor-subclass helpers, fake tensor support, and subclass-aware dispatch utilities.
- **Purpose (CN)**: 实现张量子类辅助逻辑、fake tensor 支持以及面向子类的 dispatch 工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-17 / 第 1-17 行
````python
from __future__ import annotations

from copy import deepcopy
from itertools import combinations
from typing import Any, NamedTuple, TYPE_CHECKING

import torch
from torch.fx.operator_schemas import _normalize_function_or_error
from torch.utils import _pytree as pytree
from torch.utils._python_dispatch import TorchDispatchMode
from torch.utils._pytree import tree_map


if TYPE_CHECKING:
    from collections.abc import Iterable

    from torch._ops import OpOverload
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.fx.operator_schemas, torch.utils, ...; standard-library helpers such as __future__, copy, itertools, .... The future import postpones annotation evaluation, keeping type hints lightweight at import time. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.fx.operator_schemas、torch.utils、...；标准库辅助模块，如 __future__、copy、itertools、...。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 20-36 / 第 20-36 行
````python
class Mutation(NamedTuple):
    op_name: str
    arg_name: str


class Aliasing(NamedTuple):
    op_name: str
    arg_name: str
    output_number: str


# Simplified naming for C++ classes
# pyrefly: ignore [missing-attribute]
SchemaArgument = torch._C._SchemaArgument
# pyrefly: ignore [missing-attribute]
SchemaArgType = torch._C._SchemaArgType
SchemaInfo = torch._C._SchemaInfo
````
- **EN**: It introduces or extends `Mutation`, `Aliasing`, which hold the main object-oriented state for this portion of the file. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 它引入或扩展了 `Mutation`、`Aliasing`，这些类承载了本段涉及的主要面向对象状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 38-57 / 第 38-57 行
````python
# This TorchDispatchMode Subclass is used to verify op schemas
# This TorchDispatchMode Scubclass currently:
#  - Records the called ops
#  - Checks for mutations on all inputs
#  - Checks for aliasing on all inputs


# move these 2 functions here to avoid numpy dependency in testing/_internal/common_utils.py


def is_iterable_of_tensors(iterable: Iterable[Any]) -> bool:
    # Tensor itself is iterable so we check this first
    if isinstance(iterable, torch.Tensor):
        return False
    try:
        # pyrefly: ignore[bad-argument-type]
        if len(iterable) == 0:
            return False
        for t in iter(iterable):
            if not isinstance(t, torch.Tensor):
````
- **EN**: This chunk defines `is_iterable_of_tensors`, which checks a capability or invariant before later code relies on it. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_iterable_of_tensors`，其作用是检查某项能力或不变量，供后续逻辑依赖。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 58-75 / 第 58-75 行
````python
                return False
    except TypeError:
        return False
    return True


def clone_inputs(args: Iterable[Any]) -> list[Any]:
    inputs: list[Any] = []

    for arg in args:
        if isinstance(arg, torch.Tensor):
            inputs.append(arg.detach().clone())
        elif is_iterable_of_tensors(arg):
            inputs.append([t.detach().clone() for t in arg])
        else:
            inputs.append(arg)

    return inputs
````
- **EN**: This chunk defines `clone_inputs`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `clone_inputs`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 78-93 / 第 78-93 行
````python
class SchemaCheckMode(TorchDispatchMode):
    def __init__(self) -> None:
        # Information recorded for testing purposes. For example:
        #  - incorrect schemas
        #  - overly conservative schemas
        self.ops: list[str] = []
        self.mutated: list[Mutation] = []
        self.aliasing: list[Aliasing] = []

    def reset_cache(self) -> None:
        self.ops.clear()
        self.mutated.clear()
        self.aliasing.clear()

    def display_ops(self) -> None:
        print(*self.ops, sep=",")
````
- **EN**: It introduces or extends `SchemaCheckMode`, which hold the main object-oriented state for this portion of the file. This chunk defines `display_ops`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 它引入或扩展了 `SchemaCheckMode`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `display_ops`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 95-108 / 第 95-108 行
````python
    def __torch_dispatch__(
        self,
        func: OpOverload,
        types: tuple[type[Any], ...],
        args: tuple[Any, ...] = (),
        kwargs: dict[str, Any] | None = None,
    ) -> Any:
        def bitwise_equal(lhs: torch.Tensor, rhs: torch.Tensor) -> bool:
            if lhs.is_quantized:
                # TODO: This is only OK if can't have NaN quantized; idk if
                # this is actually true
                return torch.equal(lhs, rhs)
            else:
                return torch.allclose(lhs, rhs, equal_nan=True)
````
- **EN**: This chunk defines `bitwise_equal`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `bitwise_equal`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 110-125 / 第 110-125 行
````python
        def has_mutated(
            before: Any, after: Any, md: tuple[tuple[int, ...], int] | None
        ) -> bool:
            are_tensors = type(before) is torch.Tensor and type(after) is torch.Tensor
            if (
                are_tensors
                and before.layout != torch.sparse_csr
                and after.layout != torch.sparse_csr
            ):
                return md is not None and not (
                    before.size() == after.size()
                    and bitwise_equal(before, after)
                    and md[0] == after.stride()
                    and md[1] == after._typed_storage()._cdata
                )
            return False
````
- **EN**: This chunk defines `has_mutated`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `has_mutated`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 127-146 / 第 127-146 行
````python
        def has_aliased(lhs: Any, rhs: Any) -> bool:
            try:
                # pyrefly: ignore [missing-attribute]
                return torch._C._overlaps(lhs, rhs)
            except Exception as exception:
                if str(exception).startswith("Cannot inspect value of type "):
                    return False
                else:
                    raise exception

        def standardize_name(name: str) -> str:
            return name if name != "self" else "input"

        def unwrap(e: Any) -> Any:
            if isinstance(e, torch.Tensor) and type(e) is not torch.Tensor:
                try:
                    # pyrefly: ignore[missing-attribute]
                    return e.elem
                except AttributeError:
                    return e
````
- **EN**: This chunk defines `unwrap`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `unwrap`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 147-164 / 第 147-164 行
````python
            return e

        def parse_metadata(e: Any) -> tuple[tuple[int, ...], int] | None:
            if isinstance(e, torch.Tensor):
                if type(e) is not torch.Tensor:
                    try:
                        # pyrefly: ignore[missing-attribute]
                        current = e.elem
                        return (
                            deepcopy(current.stride()),
                            current._typed_storage()._cdata,
                        )
                    except AttributeError:
                        return None
                # Sparse CSR tensors do not have strides or storage
                elif e.layout != torch.sparse_csr:
                    return (deepcopy(e.stride()), e._typed_storage()._cdata)
            return None
````
- **EN**: This chunk defines `parse_metadata`, which parses structured input into internal objects or validated metadata. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `parse_metadata`，其作用是把结构化输入解析为内部对象或经过验证的元数据。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 166-182 / 第 166-182 行
````python
        self.ops.append(func._schema.name)

        # Clone and process arguments and outputs
        pre_arguments = _normalize_function_or_error(
            func, args, kwargs, normalize_to_only_use_kwargs=True
        ).kwargs

        c_p_args = dict(zip(pre_arguments.keys(), clone_inputs(pre_arguments.values())))
        cloned_arguments = {
            name: tree_map(unwrap, c_p_args.get(name)) for name in c_p_args
        }
        cloned_metadata = {
            name: [
                parse_metadata(a) for a in pytree.tree_leaves(pre_arguments.get(name))
            ]
            for name in pre_arguments
        }
````
- **EN**: This chunk continues `parse_metadata` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `parse_metadata`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 184-203 / 第 184-203 行
````python
        out = func(*args, **kwargs)
        arguments = {
            name: tree_map(unwrap, pre_arguments.get(name)) for name in pre_arguments
        }
        tuple_out = out if isinstance(out, tuple) else (out,)
        tuple_out = tree_map(unwrap, tuple_out)

        schema_info = SchemaInfo(func._schema)
        # pyrefly: ignore [missing-attribute]
        schema_info.add_argument_values(pre_arguments)

        # Process arguments with outputs
        for i in range(len(func._schema.arguments)):
            arg = func._schema.arguments[i]
            name = standardize_name(arg.name)
            if arguments.get(name) is not None:
                before = cloned_arguments.get(name)
                md = cloned_metadata.get(name)
                after = arguments.get(name)
                for j in range(len(tuple_out)):
````
- **EN**: This chunk continues `parse_metadata` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `parse_metadata`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 204-223 / 第 204-223 行
````python
                    # aten::_unsafe_view is intended to have incorrect aliasing notation (hence unsafe)
                    unsafe_ops = ("aten::_unsafe_view", "aten::unsafe_split")
                    if (
                        has_aliased(tuple_out[j], after)
                        and func._schema.name not in unsafe_ops
                    ):
                        # pyrefly: ignore [missing-attribute]
                        if not schema_info.may_contain_alias(
                            SchemaArgument(SchemaArgType.output, j),
                            SchemaArgument(SchemaArgType.input, i),
                        ):
                            raise RuntimeError(
                                f"Argument {name} is not defined to alias output but was aliasing"
                            )
                        else:
                            self.aliasing.append(
                                Aliasing(func._schema.name, name, f"output_{j}")
                            )
                    if after is tuple_out[j] and isinstance(after, torch.Tensor):
                        # Only mutable ops e.g. (add_, add.out) are allowed to directly return inputs.
````
- **EN**: This chunk continues `parse_metadata` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `parse_metadata`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 224-243 / 第 224-243 行
````python
                        if not schema_info.is_mutable(
                            SchemaArgument(SchemaArgType.input, i)
                        ) and func not in [
                            torch.ops.aten.lift.default,
                            torch.ops.aten.lift_fresh.default,
                        ]:
                            raise RuntimeError(
                                f"""\
Dispatcher operators below autograd are not allowed to directly return inputs.
However, we found that `outputs[{str(j)}] is {name}"""
                            )
                if md is not None and any(
                    has_mutated(a, b, c)
                    for a, b, c in zip(
                        pytree.tree_leaves(before), pytree.tree_leaves(after), md
                    )
                ):
                    if not schema_info.is_mutable(
                        SchemaArgument(SchemaArgType.input, i)
                    ):
````
- **EN**: This chunk continues `parse_metadata` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `parse_metadata`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 244-260 / 第 244-260 行
````python
                        raise RuntimeError(
                            f"Argument {name} is not defined as mutable but was mutated"
                        )
                    else:
                        self.mutated.append(Mutation(func._schema.name, name))

        # Aliasing between outputs
        for i, j in combinations(range(len(func._schema.returns)), 2):
            if has_aliased(tuple_out[i], tuple_out[j]):
                # pyrefly: ignore [missing-attribute]
                if not schema_info.may_contain_alias(
                    SchemaArgument(SchemaArgType.output, i),
                    SchemaArgument(SchemaArgType.output, j),
                ):
                    raise RuntimeError(f"Outputs {i} and {j} alias unexpectedly")

        return out
````
- **EN**: This chunk continues `parse_metadata` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `parse_metadata`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor subclassing**
  - EN: Handles tensor subclasses and fake/symbolic tensor behavior without breaking dispatch.
  - CN: 在不破坏 dispatch 的前提下处理张量子类与 fake/符号张量行为。
- **Mutation**
  - EN: `Mutation` is one of the main symbols declared or implemented in this file.
  - CN: `Mutation` 是本文件声明或实现的主要符号之一。
- **Aliasing**
  - EN: `Aliasing` is one of the main symbols declared or implemented in this file.
  - CN: `Aliasing` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **NumPy compatibility**
  - EN: The implementation mirrors NumPy-style semantics to keep array/tensor behavior familiar.
  - CN: 实现对齐 NumPy 风格语义，使数组/张量行为更易理解。
- **Sparse semantics**
  - EN: The logic preserves sparse layouts and sparse-specific invariants across helpers and wrappers.
  - CN: 逻辑会在辅助函数和包装层中保持稀疏布局及其不变量。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.fx.operator_schemas`, `torch.utils`, `torch.utils._python_dispatch`, `torch.utils._pytree`, `torch._ops`
- **Standard library / 标准库**: `__future__`, `copy`, `itertools`, `typing`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `Mutation`, `Aliasing`, `is_iterable_of_tensors`, `clone_inputs`, `SchemaCheckMode`
