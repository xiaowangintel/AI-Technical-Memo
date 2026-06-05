# fake_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_subclasses/fake_utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements tensor-subclass helpers, fake tensor support, and subclass-aware dispatch utilities.
- **Purpose (CN)**: 实现张量子类辅助逻辑、fake tensor 支持以及面向子类的 dispatch 工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
````python
from __future__ import annotations

import functools
import warnings
from typing import Any, TYPE_CHECKING

import torch
import torch.utils._pytree as pytree
from torch._subclasses.fake_tensor import (
    FakeTensor,
    FakeTensorMode,
    MetadataMismatchError,
    tree_flatten_only,
    UnsupportedFakeTensorException,
)
from torch.utils._python_dispatch import TorchDispatchMode
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.utils._pytree, torch._subclasses.fake_tensor, ...; standard-library helpers such as __future__, functools, warnings, .... The future import postpones annotation evaluation, keeping type hints lightweight at import time.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.utils._pytree、torch._subclasses.fake_tensor、...；标准库辅助模块，如 __future__、functools、warnings、...。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。

### Lines 19-38 / 第 19-38 行
````python
if TYPE_CHECKING:
    from collections.abc import Callable, Mapping, Sequence

    from torch._ops import OpOverload
    from torch.utils._pytree import PyTree


aten = torch._ops.ops.aten


def outputs_alias_inputs(outputs: PyTree, inputs: PyTree) -> bool:
    input_storages = {
        inp._typed_storage()._cdata
        for inp in tree_flatten_only(torch.Tensor, inputs)
        if torch._C._has_storage(inp)
    }
    return any(
        torch._C._has_storage(out) and out._typed_storage()._cdata in input_storages
        for out in tree_flatten_only(torch.Tensor, outputs)
    )
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._ops, torch.utils._pytree; standard-library helpers such as collections.abc. This chunk defines `outputs_alias_inputs`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._ops、torch.utils._pytree；标准库辅助模块，如 collections.abc。 这一段定义了 `outputs_alias_inputs`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 41-55 / 第 41-55 行
````python
def outputs_are_inputs(outputs: PyTree, inputs: PyTree) -> bool:
    input_ids = {id(inp) for inp in tree_flatten_only(torch.Tensor, inputs)}
    return any(id(out) in input_ids for out in tree_flatten_only(torch.Tensor, outputs))


def output_alias_each_other(outputs: PyTree) -> bool:
    storages = set()
    for out in tree_flatten_only(torch.Tensor, outputs):
        if not torch._C._has_storage(out):
            continue
        stor = out._typed_storage()._cdata
        if stor in storages:
            return True
        storages.add(stor)
    return False
````
- **EN**: This chunk defines `output_alias_each_other`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `output_alias_each_other`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 58-70 / 第 58-70 行
````python
def _check_alias_info(
    context: str,
    real_out: PyTree,
    real_in: PyTree,
    fake_out: PyTree,
    fake_in: PyTree,
) -> None:
    r_aliasing = outputs_alias_inputs(real_out, real_in)
    f_aliasing = outputs_alias_inputs(fake_out, fake_in)
    if r_aliasing != f_aliasing:
        raise MetadataMismatchError(
            f"{context} mismatch in outputs_alias_inputs check {f_aliasing} != {r_aliasing}"
        )
````
- **EN**: This chunk defines `_check_alias_info`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `_check_alias_info`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 72-85 / 第 72-85 行
````python
    r_identity_eq = outputs_are_inputs(real_out, real_in)
    f_identity_eq = outputs_are_inputs(fake_out, fake_in)
    if r_identity_eq != f_identity_eq:
        raise MetadataMismatchError(
            f"{context} mismatch in outputs_are_inputs check {f_identity_eq} != {r_identity_eq}"
        )

    r_output_alias_each_other = output_alias_each_other(real_out)
    f_output_alias_each_other = output_alias_each_other(fake_out)
    if r_output_alias_each_other != f_output_alias_each_other:
        raise MetadataMismatchError(
            f"{context} mismatch in outputs_alias_each_other check "
            f"{f_output_alias_each_other} != {r_output_alias_each_other}"
        )
````
- **EN**: This chunk continues `_check_alias_info` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_check_alias_info`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 88-107 / 第 88-107 行
````python
def is_sdpa_error(func: OpOverload, idx: int, e: Exception) -> bool:
    if (
        (
            func is aten._scaled_dot_product_flash_attention.default
            or func is aten._flash_attention_forward.default
        )
        and idx in (6, 7)
        and "Devices" in repr(e)
    ):
        return True
    if (
        (
            func is aten._scaled_dot_product_efficient_attention.default
            or func is aten._efficient_attention_forward.default
        )
        and idx in (2, 3)
        and "Devices" in repr(e)
    ):
        return True
    if (
````
- **EN**: This chunk defines `is_sdpa_error`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_sdpa_error`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 108-125 / 第 108-125 行
````python
        func is aten._scaled_dot_product_cudnn_attention.default
        and idx in (6, 7)
        and "Devices" in repr(e)
    ):
        return True
    return False


def try_convert_fake_to_real(
    ten_list: list[FakeTensor | Any],
) -> list[FakeTensor | torch.Tensor | Any]:
    """
    Attempt to convert fake tensors to a corresponding real tensor with the correct underlying storage by looking up
    the FakeTensorMode meta to real storage mapping. On failure to find the storage mapping, the FakeTensor will
    remain in the list.

    Note: this is not currently optimized (makes copies of the meta converter internal dictionaries)
    """
````
- **EN**: This chunk defines `try_convert_fake_to_real`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `try_convert_fake_to_real`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 127-143 / 第 127-143 行
````python
    fake_tensor = next(
        (item for item in ten_list if isinstance(item, FakeTensor)), None
    )
    if fake_tensor is None:
        return ten_list

    fake_mode = fake_tensor.fake_mode
    meta_converter = fake_mode.fake_tensor_converter.meta_converter
    desc = meta_converter.describer

    storage_to_key = {v: k for k, v in meta_converter.storage_memo.items()}
    key_to_real_storage = {v: k for k, v in desc.lookup_storage.items()}
    out = []
    for t in ten_list:
        if not isinstance(t, FakeTensor) or t.layout != torch.strided:
            out.append(t)
            continue
````
- **EN**: This chunk continues `try_convert_fake_to_real` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `try_convert_fake_to_real`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 145-162 / 第 145-162 行
````python
        key = storage_to_key.get(t.untyped_storage())
        real_storage = None if key is None else key_to_real_storage.get(key)
        if real_storage is None:
            out.append(t)
            continue

        unhinted = False

        def map_symint(s: torch.SymInt | int) -> int:
            nonlocal unhinted
            if not isinstance(s, torch.SymInt):
                return s
            unhinted = unhinted if not unhinted else s.node.has_hint()
            return s.node.hint

        stor_offset = map_symint(t.storage_offset())
        size = [map_symint(s) for s in t.shape]
        stride = [map_symint(s) for s in t.stride()]
````
- **EN**: This chunk defines `map_symint`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `map_symint`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 164-181 / 第 164-181 行
````python
        if unhinted:
            out.append(t)
            continue

        new_tensor = torch.empty(
            [],
            dtype=t.dtype,
            device=t.device,
        )
        new_tensor.set_(
            real_storage,
            storage_offset=stor_offset,
            size=size,
            stride=stride,
        )
        out.append(new_tensor.clone())

    return out
````
- **EN**: This chunk continues `map_symint` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `map_symint`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 184-200 / 第 184-200 行
````python
def _check_fake_real_tensors(
    real_out: torch.Tensor,
    fake_out: FakeTensor,
    context: str = "",
    sizes: bool = True,
    strides: bool = False,
    storage_offset: bool = True,
    requires_grad: bool = True,
) -> None:
    if requires_grad:
        if real_out.requires_grad != fake_out.requires_grad:
            raise MetadataMismatchError(
                f"{context} mismatched requires_grad-ness of outputs. "
                f"This usually means that you have added autograd support "
                f"for your operator at a dispatch key other than Autograd, "
                f"which will lead to problems"
            )
````
- **EN**: This chunk defines `_check_fake_real_tensors`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `_check_fake_real_tensors`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 202-214 / 第 202-214 行
````python
    if torch._C._has_storage(real_out):
        r_offset = real_out.storage_offset()
        f_offset = fake_out.storage_offset()
        if r_offset != f_offset:
            raise MetadataMismatchError(f"{context} mismatched storage offset")

    torch._prims.utils.compare_tensor_meta(
        real_out,
        fake_out,
        check_sizes=sizes,
        check_strides=strides,
        allow_rhs_unbacked=True,
    )
````
- **EN**: This chunk continues `_check_fake_real_tensors` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_check_fake_real_tensors`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 217-232 / 第 217-232 行
````python
class CrossRefFakeMode(TorchDispatchMode):
    def __init__(
        self,
        ignore_op_fn: Callable[[OpOverload], bool] | None = None,
        *,
        check_strides: bool = True,
        check_aliasing: bool = True,
        only_check_ops_with_meta: bool = True,
    ) -> None:
        super().__init__()
        self.ignore_op_fn = (
            ignore_op_fn if ignore_op_fn is not None else lambda fn: False
        )
        self.check_strides = check_strides
        self.check_aliasing = check_aliasing
        self.only_check_ops_with_meta = only_check_ops_with_meta
````
- **EN**: It introduces or extends `CrossRefFakeMode`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 它引入或扩展了 `CrossRefFakeMode`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 234-253 / 第 234-253 行
````python
    def __torch_dispatch__(
        self,
        func: OpOverload,
        types: Sequence[type],
        args: Sequence[object] = (),
        kwargs: Mapping[str, object] | None = None,
    ) -> object:
        kwargs = kwargs or {}

        fake_r = None
        fake_args: Sequence[object] = ()
        fake_kwargs: Mapping[str, object] = {}

        # empty_like excluded for now due to sparse complex
        # aten._to_dense.default this one is getting called with csc
        if (
            func
            not in (
                aten.lift_fresh.default,
                aten.lift_fresh_copy.default,
````
- **EN**: This chunk defines `__torch_dispatch__`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `__torch_dispatch__`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 254-266 / 第 254-266 行
````python
                aten.set_.source_Storage_storage_offset,
            )
            and not self.ignore_op_fn(func)
            and (
                not self.only_check_ops_with_meta
                or torch._subclasses.fake_impls.has_meta(func)
            )
            and torch.Tag.dynamic_output_shape not in func.tags
            and torch.Tag.inplace_view not in func.tags
            and torch.Tag.data_dependent_output not in func.tags
        ):
            # Do not import symbolic_shapes at the top of the module as it imports sympy and that's slow
            from torch.fx.experimental.symbolic_shapes import ShapeEnv
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk continues `__torch_dispatch__` and expands its internal control flow or state updates.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段延续了 `__torch_dispatch__`，进一步展开其内部控制流或状态更新。

### Lines 268-287 / 第 268-287 行
````python
            try:
                # TODO: enable_python_dispatcher() here
                with FakeTensorMode(shape_env=ShapeEnv()) as fake_mode:
                    fake_args, fake_kwargs = pytree.tree_map_only(
                        torch.Tensor,
                        functools.partial(fake_mode.from_tensor, static_shapes=True),
                        (args, kwargs),
                    )
                    with warnings.catch_warnings():
                        fake_r = func(*fake_args, **fake_kwargs)
            except UnsupportedFakeTensorException:
                pass

        context = (
            f"When comparing the output of {func} on FakeTensor and concrete Tensors, "
            f"found"
        )
        r = func(*args, **kwargs)
        if fake_r is not None:
            r_flat = pytree.tree_leaves(r)
````
- **EN**: This chunk continues `__torch_dispatch__` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `__torch_dispatch__`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 288-307 / 第 288-307 行
````python
            f_flat = pytree.tree_leaves(fake_r)
            if len(f_flat) != len(r_flat):
                raise AssertionError(
                    f"{context} mismatch in number of returns {len(f_flat)} != {len(r_flat)}"
                )

            if self.check_aliasing:
                _check_alias_info(
                    context, r, (args, kwargs), fake_r, (fake_args, fake_kwargs)
                )

            for idx, (r_out, f_out) in enumerate(
                zip(pytree.tree_leaves(r), pytree.tree_leaves(fake_r))
            ):
                r_is_ten = isinstance(r_out, torch.Tensor)
                if r_is_ten != isinstance(f_out, torch.Tensor):
                    raise AssertionError(
                        f"{context} mismatched number of tensor outputs"
                    )
                if r_is_ten:
````
- **EN**: This chunk continues `__torch_dispatch__` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `__torch_dispatch__`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 308-326 / 第 308-326 行
````python
                    try:
                        _check_fake_real_tensors(
                            r_out,
                            f_out,
                            sizes=True,
                            strides=self.check_strides,
                            storage_offset=True,
                            requires_grad=True,
                        )
                    except Exception as e:
                        if is_sdpa_error(func, idx, e):
                            continue
                        error_message = (
                            f"{context} mismatched tensor metadata: {e}"
                            if len(r_flat) == 1
                            else f"{context} mismatched tensor metadata for output[{idx}]: {e}"
                        )
                        raise MetadataMismatchError(error_message) from e
        return r
````
- **EN**: This chunk continues `__torch_dispatch__` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `__torch_dispatch__`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor subclassing**
  - EN: Handles tensor subclasses and fake/symbolic tensor behavior without breaking dispatch.
  - CN: 在不破坏 dispatch 的前提下处理张量子类与 fake/符号张量行为。
- **outputs_alias_inputs**
  - EN: `outputs_alias_inputs` is one of the main symbols declared or implemented in this file.
  - CN: `outputs_alias_inputs` 是本文件声明或实现的主要符号之一。
- **outputs_are_inputs**
  - EN: `outputs_are_inputs` is one of the main symbols declared or implemented in this file.
  - CN: `outputs_are_inputs` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.utils._pytree`, `torch._subclasses.fake_tensor`, `torch.utils._python_dispatch`, `torch._ops`, `torch.fx.experimental.symbolic_shapes`
- **Standard library / 标准库**: `__future__`, `functools`, `warnings`, `typing`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `outputs_alias_inputs`, `outputs_are_inputs`, `output_alias_each_other`, `_check_alias_info`, `is_sdpa_error`, `try_convert_fake_to_real`, `_check_fake_real_tensors`, `CrossRefFakeMode`
