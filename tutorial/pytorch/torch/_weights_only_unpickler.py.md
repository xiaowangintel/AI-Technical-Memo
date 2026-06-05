# _weights_only_unpickler.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_weights_only_unpickler.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements a restricted unpickler used by safer weights-only loading paths.
- **Purpose (CN)**: 实现受限的 unpickler，供更安全的 weights-only 加载路径使用。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
````python
# mypy: allow-untyped-defs
# Unpickler restricted to loading only state dicts
# Restrict constructing types to a list defined in _get_allowed_globals()
# Restrict BUILD operation to `Tensor`, `Parameter` and `OrderedDict` types only
# Restrict APPEND/APPENDS to `list`
# In `GLOBALS` operation do not do class lookup by name, but rather rely on dictionary
# defined by `_get_allowed_globals()` method, that contains:
# - torch types (Storage, dtypes, Tensor, `torch.Size`),
# - `torch._utils._rebuild` functions.
# - `torch.nn.Parameter`
# - `collections.Counter`
# - `collections.OrderedDict`
# Additionally, users can use an allowlist for adding classes they have deemed as safe using
# `_add_safe_globals()` (`torch.serialization.add_safe_globals`)
# `_clear_safe_globals()` (`torch.serialization.clear_safe_globals`)
# `_get_safe_globals()` (`torch.serialization.get_safe_globals`)
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 18-37 / 第 18-37 行
````python
# Based of https://github.com/python/cpython/blob/main/Lib/pickle.py
# Expected to be useful for loading PyTorch model weights
# For example:
# data = urllib.request.urlopen('https://download.pytorch.org/models/resnet50-0676ba61.pth').read()
# buf = io.BytesIO(data)
# weights = torch.load(buf, weights_only = True)

import functools as _functools
import warnings

from _codecs import encode
from collections import Counter, OrderedDict
from collections.abc import Callable
from pickle import (
    APPEND,
    APPENDS,
    BINFLOAT,
    BINGET,
    BININT,
    BININT1,
````
- **EN**: This block assembles module dependencies, pulling in standard-library helpers such as functools, warnings, _codecs, .... Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段组织模块依赖，引入了标准库辅助模块，如 functools、warnings、_codecs、...。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 38-57 / 第 38-57 行
````python
    BININT2,
    BINPERSID,
    BINPUT,
    BINUNICODE,
    BUILD,
    bytes_types,
    decode_long,
    EMPTY_DICT,
    EMPTY_LIST,
    EMPTY_SET,
    EMPTY_TUPLE,
    GLOBAL,
    LONG1,
    LONG_BINGET,
    LONG_BINPUT,
    MARK,
    NEWFALSE,
    NEWOBJ,
    NEWTRUE,
    NONE,
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 58-75 / 第 58-75 行
````python
    PROTO,
    REDUCE,
    SETITEM,
    SETITEMS,
    SHORT_BINSTRING,
    STOP,
    TUPLE,
    TUPLE1,
    TUPLE2,
    TUPLE3,
    UnpicklingError,
)
from struct import unpack
from sys import maxsize
from typing import Any

import torch
from torch._utils import _sparse_tensors_to_validate, IMPORT_MAPPING, NAME_MAPPING
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._utils; standard-library helpers such as struct, sys, typing.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._utils；标准库辅助模块，如 struct、sys、typing。

### Lines 78-92 / 第 78-92 行
````python
# modules in this list are never allowed, even if the user attempts to allowlist
# functions/classes from them
_blocklisted_modules = [
    "sys",
    "os",
    "posix",
    "nt",
]

_marked_safe_globals_set: set[Callable | tuple[Callable, str]] = set()


def _add_safe_globals(safe_globals: list[Callable | tuple[Callable, str]]):
    global _marked_safe_globals_set
    _marked_safe_globals_set = _marked_safe_globals_set.union(set(safe_globals))
````
- **EN**: This chunk defines `_add_safe_globals`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `_add_safe_globals`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 95-109 / 第 95-109 行
````python
def _get_safe_globals() -> list[Callable | tuple[Callable, str]]:
    global _marked_safe_globals_set
    return list(_marked_safe_globals_set)


def _clear_safe_globals():
    global _marked_safe_globals_set
    _marked_safe_globals_set = set()


def _remove_safe_globals(
    globals_to_remove: list[Callable | tuple[Callable, str]],
):
    global _marked_safe_globals_set
    _marked_safe_globals_set = _marked_safe_globals_set - set(globals_to_remove)
````
- **EN**: This chunk defines `_remove_safe_globals`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_remove_safe_globals`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 112-131 / 第 112-131 行
````python
class _safe_globals:
    def __init__(self, safe_globals: list[Callable | tuple[Callable, str]]):
        self.safe_globals = safe_globals

    def __enter__(self):
        _add_safe_globals(self.safe_globals)

    def __exit__(self, type, value, tb):
        _remove_safe_globals(self.safe_globals)


# Separate from _get_allowed_globals because of the lru_cache on _get_allowed_globals
# For example if user had a script like
#   torch.load(file_a)
#   torch.serialization._add_safe_globals([torch.foo])
#   torch.load(file_b)
# the dynamic additions to safe_globals would not be picked up by
# _get_allowed_globals due to the lru_cache
def _get_user_allowed_globals():
    rc: dict[str, Any] = {}
````
- **EN**: It introduces or extends `_safe_globals`, which hold the main object-oriented state for this portion of the file. This chunk defines `_get_user_allowed_globals`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 它引入或扩展了 `_safe_globals`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `_get_user_allowed_globals`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 132-147 / 第 132-147 行
````python
    for f in _marked_safe_globals_set:
        if isinstance(f, tuple):
            if len(f) != 2:
                raise ValueError(
                    f"Expected tuple of length 2 (global, str of callable full path), but got tuple of length: {len(f)}"
                )
            if type(f[1]) is not str:
                raise TypeError(
                    f"Expected second item in tuple to be str of callable full path, but got: {type(f[1])}"
                )
            f, name = f
            rc[name] = f
        else:
            module, name = f.__module__, f.__qualname__
            rc[f"{module}.{name}"] = f
    return rc
````
- **EN**: This chunk continues `_safe_globals` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_safe_globals`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 150-169 / 第 150-169 行
````python
def _tensor_rebuild_functions():
    return {
        torch._utils._rebuild_parameter,
        torch._utils._rebuild_parameter_with_state,
        torch._utils._rebuild_qtensor,
        torch._utils._rebuild_tensor,
        torch._utils._rebuild_tensor_v2,
        torch._utils._rebuild_tensor_v3,
        torch._utils._rebuild_sparse_tensor,
        torch._utils._rebuild_meta_tensor_no_storage,
        torch._utils._rebuild_nested_tensor,
        torch._utils._rebuild_wrapper_subclass,
        # Allowlisting this, but not allowlisting the numpy functions by default
        # Reasoning is that we don't have control over the numpy functions, but
        # this utility is provided by pytorch
        torch._utils._rebuild_device_tensor_from_numpy,
        # In 2.6, we should no longer have a dependency on numpy and the above
        # _rebuild_device_tensor_from_numpy function.
        torch._utils._rebuild_device_tensor_from_cpu_tensor,
    }
````
- **EN**: This chunk defines `_tensor_rebuild_functions`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_tensor_rebuild_functions`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 172-187 / 第 172-187 行
````python
# Unpickling machinery
@_functools.lru_cache(maxsize=1)
def _get_allowed_globals():
    rc: dict[str, Any] = {
        "collections.OrderedDict": OrderedDict,
        "collections.Counter": Counter,
        "torch.nn.parameter.Parameter": torch.nn.Parameter,
        "torch.serialization._get_layout": torch.serialization._get_layout,
        "torch.Size": torch.Size,
        "torch.Tensor": torch.Tensor,
        "torch.device": torch.device,
        "_codecs.encode": encode,  # for bytes
        "builtins.bytearray": bytearray,  # for bytearray
        "builtins.set": set,  # for set
        "builtins.complex": complex,  # for complex
    }
````
- **EN**: This chunk defines `_get_allowed_globals`, which implements a focused helper used by the surrounding module. Decorators such as `_functools.lru_cache` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `_get_allowed_globals`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_functools.lru_cache` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 189-208 / 第 189-208 行
````python
    # dtype
    for t in torch.storage._dtype_to_storage_type_map():
        rc[str(t)] = t
    for t in torch.storage._new_dtypes():
        rc[str(t)] = t
    for t in [getattr(torch, f"uint{x}") for x in range(1, 8)]:
        rc[str(t)] = t
    for t in [getattr(torch, f"int{x}") for x in range(1, 8)]:
        rc[str(t)] = t

    # Tensor classes
    for tt in torch._tensor_classes:
        rc[f"{tt.__module__}.{tt.__name__}"] = tt
    # Storage classes
    for ts in torch._storage_classes:
        if ts not in (torch.storage.TypedStorage, torch.storage.UntypedStorage):
            # Wrap legacy storage types in a dummy class
            rc[f"{ts.__module__}.{ts.__name__}"] = torch.serialization.StorageType(
                ts.__name__
            )
````
- **EN**: This chunk continues `_get_allowed_globals` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_get_allowed_globals`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 209-227 / 第 209-227 行
````python
        else:
            rc[f"{ts.__module__}.{ts.__name__}"] = ts
    # Quantization specific
    for qt in [
        torch.per_tensor_affine,
        torch.per_tensor_symmetric,
        torch.per_channel_affine,
        torch.per_channel_symmetric,
        torch.per_channel_affine_float_qparams,
    ]:
        rc[str(qt)] = qt
    # Rebuild functions
    for f in _tensor_rebuild_functions():
        rc[f"torch._utils.{f.__name__}"] = f

    # Handles Tensor Subclasses, Tensor's with attributes.
    # NOTE: It calls into above rebuild functions for regular Tensor types.
    rc["torch._tensor._rebuild_from_type_v2"] = torch._tensor._rebuild_from_type_v2
    return rc
````
- **EN**: This chunk continues `_get_allowed_globals` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_get_allowed_globals`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 230-249 / 第 230-249 行
````python
def _read_global_instruction(readline: Callable) -> tuple[str, str]:
    module = readline()[:-1].decode("utf-8")
    name = readline()[:-1].decode("utf-8")
    # Patch since torch.save default protocol is 2
    # users will be running this code in python > 3
    if (module, name) in NAME_MAPPING:
        module, name = NAME_MAPPING[(module, name)]
    elif module in IMPORT_MAPPING:
        module = IMPORT_MAPPING[module]
    return module, name


def get_globals_in_pkl(file) -> set[str]:
    globals_in_checkpoint = set()
    read = file.read
    readline = file.readline
    op_to_bytes_to_read = {
        NEWOBJ[0]: 0,
        REDUCE[0]: 0,
        BUILD[0]: 0,
````
- **EN**: This chunk defines `get_globals_in_pkl`, which retrieves runtime state and exposes it through a Python-friendly accessor. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_globals_in_pkl`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 250-269 / 第 250-269 行
````python
        APPEND[0]: 0,
        APPENDS[0]: 0,
        SETITEM[0]: 0,
        SETITEMS[0]: 0,
        MARK[0]: 0,
        TUPLE[0]: 0,
        TUPLE1[0]: 0,
        TUPLE2[0]: 0,
        TUPLE3[0]: 0,
        NONE[0]: 0,
        NEWFALSE[0]: 0,
        NEWTRUE[0]: 0,
        EMPTY_TUPLE[0]: 0,
        EMPTY_LIST[0]: 0,
        EMPTY_DICT[0]: 0,
        EMPTY_SET[0]: 0,
        BINPERSID[0]: 0,
        BININT[0]: 4,
        BININT1[0]: 1,
        BININT2[0]: 2,
````
- **EN**: This chunk continues `get_globals_in_pkl` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `get_globals_in_pkl`，进一步展开其内部控制流或状态更新。

### Lines 270-289 / 第 270-289 行
````python
        BINFLOAT[0]: 8,
        BINGET[0]: 1,
        LONG_BINGET[0]: 4,
        BINPUT[0]: 1,
        LONG_BINPUT[0]: 4,
    }
    while True:
        key = read(1)
        if not key:
            raise EOFError
        if not isinstance(key, bytes_types):
            raise AssertionError(f"Expected bytes, got {type(key).__name__}")
        if key[0] == GLOBAL[0]:
            module, name = _read_global_instruction(readline)
            globals_in_checkpoint.add(f"{module}.{name}")
        elif key[0] in op_to_bytes_to_read:
            bytes_to_read = op_to_bytes_to_read[key[0]]
            if bytes_to_read:
                read(bytes_to_read)
        # ops where bytes to read depends on the data
````
- **EN**: This chunk continues `get_globals_in_pkl` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `get_globals_in_pkl`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 290-304 / 第 290-304 行
````python
        elif key[0] == BINUNICODE[0]:
            strlen = unpack("<I", read(4))[0]
            if strlen > maxsize:
                raise UnpicklingError("String is too long")
            read(strlen)
        elif key[0] in {SHORT_BINSTRING[0], LONG1[0]}:
            strlen = read(1)[0]
            read(strlen)
        # first and last op
        elif key[0] == PROTO[0]:
            read(1)[0]
        elif key[0] == STOP[0]:
            return globals_in_checkpoint
        else:
            raise UnpicklingError(f"Unsupported operand {key[0]}")
````
- **EN**: This chunk continues `get_globals_in_pkl` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_globals_in_pkl`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 307-326 / 第 307-326 行
````python
class Unpickler:
    def __init__(self, file, *, encoding: str = "bytes"):
        self.encoding = encoding
        self.readline = file.readline
        self.read = file.read
        self.memo: dict[int, Any] = {}
        self.proto: int = -1

    def load(self):
        """Read a pickled object representation from the open file.

        Return the reconstituted object hierarchy specified in the file.
        """
        self.metastack = []
        self.stack: list[Any] = []
        self.append = self.stack.append
        read = self.read
        while True:
            key = read(1)
            if not key:
````
- **EN**: It introduces or extends `Unpickler`, which hold the main object-oriented state for this portion of the file. This chunk defines `load`, which serializes or reconstructs state across a Python-visible boundary. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 它引入或扩展了 `Unpickler`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `load`，其作用是在 Python 可见边界上序列化或重建状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 327-346 / 第 327-346 行
````python
                raise EOFError
            if not isinstance(key, bytes_types):
                raise AssertionError(f"Expected bytes, got {type(key).__name__}")
            # Risky operators
            if key[0] == GLOBAL[0]:
                module, name = _read_global_instruction(self.readline)
                full_path = f"{module}.{name}"
                if module in _blocklisted_modules:
                    raise UnpicklingError(
                        f"Trying to load unsupported GLOBAL {full_path} whose module {module} is blocked."
                    )
                if full_path in _get_allowed_globals():
                    self.append(_get_allowed_globals()[full_path])
                elif full_path in _get_user_allowed_globals():
                    self.append(_get_user_allowed_globals()[full_path])
                elif full_path in (
                    [
                        "torch.nested._internal.nested_tensor.NestedTensor",
                        "torch.nested._internal.nested_tensor._rebuild_njt",
                        "torch._dynamo.decorators._DimRange",
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 347-366 / 第 347-366 行
````python
                    ]
                ):
                    raise UnpicklingError(
                        "``torch.nested`` and ``torch._dynamo`` must be imported to load nested jagged tensors (NJTs)"
                    )
                elif full_path in (
                    [
                        "torch.distributed.device_mesh.DeviceMesh",
                        "torch.distributed.tensor._dtensor_spec.DTensorSpec",
                        "torch.distributed.tensor._dtensor_spec.TensorMeta",
                        "torch.distributed.tensor.DTensor",
                        "torch.distributed.tensor.placement_types.Partial",
                        "torch.distributed.tensor.placement_types.Replicate",
                        "torch.distributed.tensor.placement_types.Shard",
                    ]
                ):
                    raise UnpicklingError(
                        "``torch.distributed.tensor`` must be imported to load DTensors"
                    )
                else:
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 367-386 / 第 367-386 行
````python
                    builtins_name = "builtins"
                    if (
                        builtins_name in full_path
                        and builtins_name == full_path[: len(builtins_name)]
                    ):
                        full_path = full_path[len(builtins_name) :]
                        full_path = (
                            full_path[1:]
                            if len(full_path) > 0 and full_path[0] == "."
                            else builtins_name + full_path
                        )
                    raise UnpicklingError(
                        f"Unsupported global: GLOBAL {full_path} was not an allowed global by default. "
                        f"Please use `torch.serialization.add_safe_globals([{full_path}])` or the "
                        f"`torch.serialization.safe_globals([{full_path}])` context manager to allowlist this global "
                        "if you trust this class/function."
                    )
            elif key[0] == NEWOBJ[0]:
                args = self.stack.pop()
                cls = self.stack.pop()
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 387-406 / 第 387-406 行
````python
                if cls is torch.nn.Parameter:
                    self.append(torch.nn.Parameter(*args))
                elif (
                    cls in _get_user_allowed_globals().values()
                    or cls in _get_allowed_globals().values()
                ):
                    result = cls.__new__(cls, *args)
                    if cls in torch._tensor_classes and "sparse" in cls.__module__:
                        _sparse_tensors_to_validate.append(result)
                    self.append(result)
                else:
                    raise UnpicklingError(
                        "Can only create new object for nn.Parameter or classes allowlisted "
                        f"via `add_safe_globals` but got {cls}"
                    )
            elif key[0] == REDUCE[0]:
                args = self.stack.pop()
                func = self.stack[-1]
                if (
                    func not in _get_allowed_globals().values()
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 407-423 / 第 407-423 行
````python
                    and func not in _get_user_allowed_globals().values()
                ):
                    error_msg = (
                        f"Trying to call reduce for unrecognized function {func}"
                    )
                    if hasattr(func, "__self__"):
                        error_msg += f" which belongs to {func.__self__}"
                    raise UnpicklingError(error_msg)
                result = func(*args)
                if func in torch._tensor_classes and "sparse" in func.__module__:
                    _sparse_tensors_to_validate.append(result)
                self.stack[-1] = result
            elif key[0] == BUILD[0]:
                state = self.stack.pop()
                inst = self.stack[-1]
                if type(inst) is torch.Tensor:
                    # Legacy unpickling
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 425-444 / 第 425-444 行
````python
                    inst.set_(*state)
                elif type(inst) is torch.nn.Parameter:
                    inst.__setstate__(state)
                elif type(inst) is OrderedDict:
                    inst.__dict__.update(state)
                elif (
                    type(inst) in _get_user_allowed_globals().values()
                    or type(inst) in _get_allowed_globals().values()
                ):
                    if hasattr(inst, "__setstate__"):
                        inst.__setstate__(state)
                    else:
                        # mimics load_build in pickle
                        # https://github.com/python/cpython/blob/f0c6fccd08904787a39269367f09f263d496114c/Lib/pickle.py#L1854-L1867
                        slotstate = None
                        if isinstance(state, tuple) and len(state) == 2:
                            state, slotstate = state
                        if state:
                            inst.__dict__.update(state)
                        if slotstate:
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 445-464 / 第 445-464 行
````python
                            for k, v in slotstate.items():
                                setattr(inst, k, v)
                else:
                    raise UnpicklingError(
                        "Can only build Tensor, Parameter, OrderedDict or types allowlisted "
                        f"via `add_safe_globals`, but got {type(inst)}"
                    )
            # Stack manipulation
            elif key[0] == APPEND[0]:
                item = self.stack.pop()
                list_obj = self.stack[-1]
                if type(list_obj) is not list:
                    raise UnpicklingError(
                        f"Can only append to lists, but got {type(list_obj)}"
                    )
                list_obj.append(item)
            elif key[0] == APPENDS[0]:
                items = self.pop_mark()
                list_obj = self.stack[-1]
                if type(list_obj) is not list:
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 465-484 / 第 465-484 行
````python
                    raise UnpicklingError(
                        f"Can only extend lists, but got {type(list_obj)}"
                    )
                list_obj.extend(items)
            elif key[0] == SETITEM[0]:
                (v, k) = (self.stack.pop(), self.stack.pop())
                self._check_set_item_target("SETITEM")
                self.stack[-1][k] = v
            elif key[0] == SETITEMS[0]:
                items = self.pop_mark()
                self._check_set_item_target("SETITEMS")
                for i in range(0, len(items), 2):
                    self.stack[-1][items[i]] = items[i + 1]
            elif key[0] == MARK[0]:
                self.metastack.append(self.stack)
                self.stack = []
                self.append = self.stack.append
            elif key[0] == TUPLE[0]:
                items = self.pop_mark()
                self.append(tuple(items))
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 485-504 / 第 485-504 行
````python
            elif key[0] == TUPLE1[0]:
                self.stack[-1] = (self.stack[-1],)
            elif key[0] == TUPLE2[0]:
                self.stack[-2:] = [(self.stack[-2], self.stack[-1])]
            elif key[0] == TUPLE3[0]:
                self.stack[-3:] = [(self.stack[-3], self.stack[-2], self.stack[-1])]
            # Basic types construction
            elif key[0] == NONE[0]:
                self.append(None)
            elif key[0] == NEWFALSE[0]:
                self.append(False)
            elif key[0] == NEWTRUE[0]:
                self.append(True)
            elif key[0] == EMPTY_TUPLE[0]:
                self.append(())
            elif key[0] == EMPTY_LIST[0]:
                self.append([])
            elif key[0] == EMPTY_DICT[0]:
                self.append({})
            elif key[0] == EMPTY_SET[0]:
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 505-524 / 第 505-524 行
````python
                self.append(set())
            elif key[0] == BININT[0]:
                self.append(unpack("<i", read(4))[0])
            elif key[0] == BININT1[0]:
                self.append(self.read(1)[0])
            elif key[0] == BININT2[0]:
                self.append(unpack("<H", read(2))[0])
            elif key[0] == BINFLOAT[0]:
                self.append(unpack(">d", self.read(8))[0])
            elif key[0] == BINUNICODE[0]:
                strlen = unpack("<I", read(4))[0]
                if strlen > maxsize:
                    raise UnpicklingError("String is too long")
                strval = str(read(strlen), "utf-8", "surrogatepass")
                self.append(strval)
            elif key[0] == SHORT_BINSTRING[0]:
                strlen = read(1)[0]
                strdata = read(strlen)
                if self.encoding != "bytes":
                    strdata = strdata.decode(self.encoding, "strict")
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 525-544 / 第 525-544 行
````python
                self.append(strdata)
            elif key[0] == BINPERSID[0]:
                pid = self.stack.pop()
                # Only allow persistent load of storage
                if type(pid) is not tuple and type(pid) is not int:
                    raise UnpicklingError(
                        f"persistent_load id must be tuple or int, but got {type(pid)}"
                    )
                if (
                    type(pid) is tuple
                    and len(pid) > 0
                    and torch.serialization._maybe_decode_ascii(pid[0]) != "storage"
                ):
                    raise UnpicklingError(
                        f"Only persistent_load of storage is allowed, but got {type(pid[0])}"
                    )
                self.append(self.persistent_load(pid))
            elif key[0] in [BINGET[0], LONG_BINGET[0]]:
                idx = (read(1) if key[0] == BINGET[0] else unpack("<I", read(4)))[0]
                self.append(self.memo[idx])
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 545-564 / 第 545-564 行
````python
            elif key[0] in [BINPUT[0], LONG_BINPUT[0]]:
                i = (read(1) if key[0] == BINPUT[0] else unpack("<I", read(4)))[0]
                if i < 0:
                    raise ValueError("negative argument")
                self.memo[i] = self.stack[-1]
            elif key[0] == LONG1[0]:
                n = read(1)[0]
                data = read(n)
                self.append(decode_long(data))
            # First and last deserializer ops
            elif key[0] == PROTO[0]:
                self.proto = read(1)[0]
                if self.proto != 2:
                    warnings.warn(
                        f"Detected pickle protocol {self.proto} in the checkpoint, which was "
                        "not the default pickle protocol used by `torch.load` (2). The weights_only "
                        "Unpickler might not support all instructions implemented by this protocol, "
                        "please file an issue for adding support if you encounter this.",
                        stacklevel=2,
                    )
````
- **EN**: This chunk continues `Unpickler` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `Unpickler`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 565-583 / 第 565-583 行
````python
            elif key[0] == STOP[0]:
                rc = self.stack.pop()
                return rc
            else:
                raise UnpicklingError(f"Unsupported operand {key[0]}")

    # Return a list of items pushed in the stack after last MARK instruction.
    def pop_mark(self):
        items = self.stack
        self.stack = self.metastack.pop()
        self.append = self.stack.append
        return items

    def _check_set_item_target(self, opcode: str):
        if type(self.stack[-1]) not in [dict, OrderedDict, Counter]:
            raise UnpicklingError(
                f"Can only {opcode} for dict, collections.OrderedDict, "
                f"collections.Counter, but got {type(self.stack[-1])}"
            )
````
- **EN**: This chunk defines `_check_set_item_target`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_check_set_item_target`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 585-590 / 第 585-590 行
````python
    def persistent_load(self, pid):
        raise UnpicklingError("unsupported persistent id encountered")


def load(file, *, encoding: str = "ASCII"):
    return Unpickler(file, encoding=encoding).load()
````
- **EN**: This chunk defines `load`, which serializes or reconstructs state across a Python-visible boundary. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `load`，其作用是在 Python 可见边界上序列化或重建状态。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **_weights_only_unpickler**
  - EN: Implements a restricted unpickler used by safer weights-only loading paths.
  - CN: 实现受限的 unpickler，供更安全的 weights-only 加载路径使用。
- **_add_safe_globals**
  - EN: `_add_safe_globals` is one of the main symbols declared or implemented in this file.
  - CN: `_add_safe_globals` 是本文件声明或实现的主要符号之一。
- **_get_safe_globals**
  - EN: `_get_safe_globals` is one of the main symbols declared or implemented in this file.
  - CN: `_get_safe_globals` 是本文件声明或实现的主要符号之一。
- **NumPy compatibility**
  - EN: The implementation mirrors NumPy-style semantics to keep array/tensor behavior familiar.
  - CN: 实现对齐 NumPy 风格语义，使数组/张量行为更易理解。
- **Sparse semantics**
  - EN: The logic preserves sparse layouts and sparse-specific invariants across helpers and wrappers.
  - CN: 逻辑会在辅助函数和包装层中保持稀疏布局及其不变量。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._utils`
- **Standard library / 标准库**: `functools`, `warnings`, `_codecs`, `collections`, `collections.abc`, `pickle`, `struct`, `sys`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `_add_safe_globals`, `_get_safe_globals`, `_clear_safe_globals`, `_remove_safe_globals`, `_safe_globals`, `_get_user_allowed_globals`, `_tensor_rebuild_functions`, `_get_allowed_globals`, `_read_global_instruction`, `get_globals_in_pkl`
