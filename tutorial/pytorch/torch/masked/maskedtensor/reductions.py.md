# reductions.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/masked/maskedtensor/reductions.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements masked-tensor helpers and namespace wrappers for masked operations.
- **Purpose (CN)**: 实现 masked tensor 辅助逻辑以及 masked 运算的命名空间包装层。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行
````python
# mypy: allow-untyped-defs
# Copyright (c) Meta Platforms, Inc. and affiliates

import warnings

import torch

from .core import is_masked_tensor
from .creation import as_masked_tensor, masked_tensor


__all__ = []  # type: ignore[var-annotated]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, .core, .creation; standard-library helpers such as warnings. `__all__` defines the public symbols that this module chooses to export.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、.core、.creation；标准库辅助模块，如 warnings。 `__all__` 定义了本模块选择导出的公共符号。

### Lines 15-24 / 第 15-24 行
````python
def _masked_all_all(data, mask=None):
    if mask is None:
        return data.all()
    return data.masked_fill(~mask, True).all()


def _masked_all_dim(data, dim, keepdim=False, mask=None):
    if mask is None:
        return torch.all(data, dim=dim, keepdim=keepdim)
    return torch.all(data.masked_fill(~mask, True), dim=dim, keepdim=keepdim)
````
- **EN**: This chunk defines `_masked_all_dim`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_masked_all_dim`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 27-38 / 第 27-38 行
````python
def _masked_all(*args, **kwargs):
    if len(args) == 1 and len(kwargs) == 1:
        return _masked_all_all(args[0], mask=kwargs["mask"])
    return _masked_all_dim(*args, **kwargs)


def _multidim_any(mask, dim, keepdim):
    if isinstance(dim, int):
        return _multidim_any(mask, [dim], keepdim)
    for d in sorted(dim, reverse=True):
        mask = torch.any(mask, dim=d, keepdim=keepdim)
    return mask
````
- **EN**: This chunk defines `_multidim_any`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_multidim_any`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 41-54 / 第 41-54 行
````python
def _get_masked_fn(fn):
    if fn == "all":
        return _masked_all
    return getattr(torch.masked, fn)


def _torch_reduce_all(fn):
    def reduce_all(self):
        masked_fn = _get_masked_fn(fn)
        data = self.get_data()
        mask = self.get_mask().values() if self.is_sparse else self.get_mask()
        # When reduction is "all", then torch.argmin/torch.argmax needs to return the index of the
        # element corresponding to the min/max, but this operation isn't supported correctly for sparse layouts.
        # Therefore, this implementation calculates it using the strides.
````
- **EN**: This chunk defines `reduce_all`, which coordinates collective-style data movement or aggregation. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `reduce_all`，其作用是协调类似集合通信的数据移动或聚合。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 55-68 / 第 55-68 行
````python
        if fn == "all":
            result_data = masked_fn(data, mask=mask)

        elif fn in {"argmin", "argmax"} and self.is_sparse_coo():
            sparse_idx = masked_fn(data.values(), mask=mask).to(dtype=torch.int)
            indices = (
                data.to_sparse_coo().indices()
                if not self.is_sparse_coo()
                else data.indices()
            )
            idx = indices.unbind(1)[sparse_idx]
            stride = data.size().numel() / torch.tensor(
                data.size(), device=data.device
            ).cumprod(0)
````
- **EN**: This chunk continues `reduce_all` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `reduce_all`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 69-80 / 第 69-80 行
````python
            result_data = torch.sum(idx * stride)

        # we simply pass in the values for sparse COO/CSR tensors
        elif self.is_sparse:
            result_data = masked_fn(masked_tensor(data.values(), mask))

        else:
            result_data = masked_fn(self, mask=mask)

        return as_masked_tensor(result_data, torch.any(mask))

    return reduce_all
````
- **EN**: This chunk continues `reduce_all` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `reduce_all`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 83-96 / 第 83-96 行
````python
def _torch_reduce_dim(fn):
    def reduce_dim(self, dim, keepdim=False, dtype=None):
        if self.is_sparse:
            msg = (
                f"The sparse version of {fn} is not implemented in reductions.\n"
                "If you would like this operator to be supported, please file an issue for a feature request at "
                "https://github.com/pytorch/maskedtensor/issues with a minimal reproducible code snippet.\n"
                "In the case that the semantics for the operator are not trivial, it would be appreciated "
                "to also include a proposal for the semantics."
            )
            warnings.warn(msg, stacklevel=2)
            return NotImplemented
        if not is_masked_tensor(self):
            raise TypeError("Input to reduce_dim must be a MaskedTensor")
````
- **EN**: This chunk defines `reduce_dim`, which coordinates collective-style data movement or aggregation. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `reduce_dim`，其作用是协调类似集合通信的数据移动或聚合。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 98-109 / 第 98-109 行
````python
        masked_fn = _get_masked_fn(fn)
        data = self.get_data()
        mask = self.get_mask()
        if fn == "all":
            result_data = masked_fn(data, dim=dim, keepdim=keepdim, mask=mask)
        else:
            result_data = masked_fn(
                self, dim=dim, keepdim=keepdim, dtype=dtype, mask=self.get_mask()
            )
        return as_masked_tensor(result_data, _multidim_any(mask, dim, keepdim))

    return reduce_dim
````
- **EN**: This chunk continues `reduce_dim` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `reduce_dim`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 112-122 / 第 112-122 行
````python
def _torch_reduce(fn):
    def reduce_fn(*args, **kwargs):
        if len(args) == 1 and len(kwargs) == 0:
            return _torch_reduce_all(fn)(args[0])
        return _torch_reduce_dim(fn)(*args, **kwargs)

    return reduce_fn


def _reduce_dim_args(input, dim, keepdim=False, dtype=None):
    return input, dim, keepdim, dtype
````
- **EN**: This chunk defines `_reduce_dim_args`, which coordinates collective-style data movement or aggregation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_reduce_dim_args`，其作用是协调类似集合通信的数据移动或聚合。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 125-133 / 第 125-133 行
````python
def _torch_grad_reduce(fn):
    def grad_reduce(*args, **kwargs):
        if len(args) == 1 and len(kwargs) == 0:
            return _torch_reduce_all(fn)(args[0])
        # TODO: autograd.Function doesn't support kwarg
        input, dim, keepdim, dtype = _reduce_dim_args(*args, **kwargs)
        return _torch_reduce_dim(fn)(input, dim, keepdim, dtype)

    return grad_reduce
````
- **EN**: This chunk defines `grad_reduce`, which coordinates collective-style data movement or aggregation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `grad_reduce`，其作用是协调类似集合通信的数据移动或聚合。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 136-148 / 第 136-148 行
````python
REDUCE_NAMES = [
    "sum",
    "mean",
    "amin",
    "amax",
    "argmin",
    "argmax",
    "prod",
    "all",
    "norm",
    "var",
    "std",
]
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 150-162 / 第 150-162 行
````python
NATIVE_REDUCE_MAP = {
    getattr(torch.ops.aten, name): _torch_reduce(name) for name in REDUCE_NAMES
}
TORCH_REDUCE_MAP = {
    getattr(torch, name): _torch_grad_reduce(name) for name in REDUCE_NAMES
}
TENSOR_REDUCE_MAP = {
    getattr(torch.Tensor, name): _torch_grad_reduce(name) for name in REDUCE_NAMES
}

NATIVE_REDUCE_FNS = list(NATIVE_REDUCE_MAP.keys())
TORCH_REDUCE_FNS = list(TORCH_REDUCE_MAP.keys())
TENSOR_REDUCE_FNS = list(TENSOR_REDUCE_MAP.keys())
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 165-176 / 第 165-176 行
````python
def _is_reduction(fn):
    return fn in NATIVE_REDUCE_MAP or fn in TORCH_REDUCE_MAP or fn in TENSOR_REDUCE_MAP


def _apply_reduction(fn, *args, **kwargs):
    if fn in NATIVE_REDUCE_MAP:
        return NATIVE_REDUCE_MAP[fn](*args, **kwargs)
    if fn in TORCH_REDUCE_MAP:
        return TORCH_REDUCE_MAP[fn](*args, **kwargs)
    if fn in TENSOR_REDUCE_MAP:
        return TENSOR_REDUCE_MAP[fn](*args, **kwargs)
    return NotImplemented
````
- **EN**: This chunk defines `_apply_reduction`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_apply_reduction`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Masked operations**
  - EN: Represents operations that carry masks alongside data and preserve masked semantics.
  - CN: 表示同时携带 mask 与数据的运算，并保持 masked 语义。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **_masked_all_all**
  - EN: `_masked_all_all` is one of the main symbols declared or implemented in this file.
  - CN: `_masked_all_all` 是本文件声明或实现的主要符号之一。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Sparse semantics**
  - EN: The logic preserves sparse layouts and sparse-specific invariants across helpers and wrappers.
  - CN: 逻辑会在辅助函数和包装层中保持稀疏布局及其不变量。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `.core`, `.creation`
- **Standard library / 标准库**: `warnings`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `_masked_all_all`, `_masked_all_dim`, `_masked_all`, `_multidim_any`, `_get_masked_fn`, `_torch_reduce_all`, `_torch_reduce_dim`, `_torch_reduce`, `_reduce_dim_args`
