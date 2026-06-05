# __init__.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/backends/cuda/__init__.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Initializes the package namespace, re-exports symbols, and wires lazy imports or feature flags.
- **Purpose (CN)**: 初始化包命名空间，重新导出符号，并连接懒加载导入或特性开关。
## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
import contextlib
from typing import Any
from typing_extensions import deprecated

import torch


__all__ = [
    "is_built",
    "cuFFTPlanCacheAttrContextProp",
    "cuFFTPlanCache",
    "cuFFTPlanCacheManager",
    "cuBLASModule",
    "preferred_linalg_library",
    "preferred_blas_library",
    "cublas_workspace_size",
    "cublaslt_workspace_size",
    "blas_workspace_size",
    "preferred_rocm_fa_library",
    "is_ck_sdpa_available",
    "cufft_plan_cache",
    "matmul",
    "SDPAParams",
    "enable_cudnn_sdp",
    "cudnn_sdp_enabled",
    "enable_flash_sdp",
    "flash_sdp_enabled",
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch; standard-library helpers such as contextlib, typing; other helper packages such as typing_extensions. `__all__` defines the public symbols that this module chooses to export.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch；标准库辅助模块，如 contextlib、typing；其他辅助包，如 typing_extensions。 `__all__` 定义了本模块选择导出的公共符号。

### Lines 29-50 / 第 29-50 行
````python
    "enable_mem_efficient_sdp",
    "mem_efficient_sdp_enabled",
    "math_sdp_enabled",
    "enable_math_sdp",
    "allow_fp16_bf16_reduction_math_sdp",
    "fp16_bf16_reduction_math_sdp_allowed",
    "is_flash_attention_available",
    "can_use_flash_attention",
    "can_use_efficient_attention",
    "can_use_cudnn_attention",
    "sdp_kernel",
]


def is_built():
    r"""
    Return whether PyTorch is built with CUDA support.

    Note that this doesn't necessarily mean CUDA is available; just that if this PyTorch
    binary were run on a machine with working CUDA drivers and devices, we would be able to use it.
    """
    return torch._C._has_cuda
````
- **EN**: This chunk defines `is_built`, which checks a capability or invariant before later code relies on it. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_built`，其作用是检查某项能力或不变量，供后续逻辑依赖。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 53-78 / 第 53-78 行
````python
class cuFFTPlanCacheAttrContextProp:
    # Like regular ContextProp, but uses the `.device_index` attribute from the
    # calling object as the first argument to the getter and setter.
    def __init__(self, getter, setter):
        self.getter = getter
        self.setter = setter

    def __get__(self, obj, objtype):
        return self.getter(obj.device_index)

    def __set__(self, obj, val):
        if isinstance(self.setter, str):
            raise RuntimeError(self.setter)
        self.setter(obj.device_index, val)


class cuFFTPlanCache:
    r"""
    Represent a specific plan cache for a specific `device_index`.

    The attributes `size` and `max_size`, and method `clear`, can fetch and/ or
    change properties of the C++ cuFFT plan cache.
    """

    def __init__(self, device_index):
        self.device_index = device_index
````
- **EN**: It introduces or extends `cuFFTPlanCacheAttrContextProp`, `cuFFTPlanCache`, which hold the main object-oriented state for this portion of the file. This chunk defines `__set__`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `cuFFTPlanCacheAttrContextProp`、`cuFFTPlanCache`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__set__`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 80-103 / 第 80-103 行
````python
    size = cuFFTPlanCacheAttrContextProp(
        torch._cufft_get_plan_cache_size,
        ".size is a read-only property showing the number of plans currently in the "
        "cache. To change the cache capacity, set cufft_plan_cache.max_size.",
    )

    max_size = cuFFTPlanCacheAttrContextProp(
        torch._cufft_get_plan_cache_max_size, torch._cufft_set_plan_cache_max_size
    )

    def clear(self):
        return torch._cufft_clear_plan_cache(self.device_index)


class cuFFTPlanCacheManager:
    r"""
    Represent all cuFFT plan caches, return the cuFFTPlanCache for a given device when indexed.

    Finally, this object, when used directly as a `cuFFTPlanCache` object (e.g.,
    setting the `.max_size`) attribute, the current device's cuFFT plan cache is
    used.
    """

    __initialized = False
````
- **EN**: It introduces or extends `cuFFTPlanCacheManager`, which hold the main object-oriented state for this portion of the file. This chunk defines `clear`, which drops cached state so a later execution phase can rebuild it cleanly. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `cuFFTPlanCacheManager`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `clear`，其作用是清除缓存状态，以便后续执行阶段干净地重建。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 105-129 / 第 105-129 行
````python
    def __init__(self):
        self.caches = []
        self.__initialized = True

    def __getitem__(self, device):
        index = torch.cuda._utils._get_device_index(device)
        if index < 0 or index >= torch.cuda.device_count():
            raise RuntimeError(
                f"cufft_plan_cache: expected 0 <= device index < {torch.cuda.device_count()}, but got "
                f"device with index {index}"
            )
        if len(self.caches) == 0:
            self.caches.extend(
                cuFFTPlanCache(index) for index in range(torch.cuda.device_count())
            )
        return self.caches[index]

    def __getattr__(self, name):
        return getattr(self[torch.cuda.current_device()], name)

    def __setattr__(self, name, value):
        if self.__initialized:
            return setattr(self[torch.cuda.current_device()], name, value)
        else:
            return super().__setattr__(name, value)
````
- **EN**: This chunk defines `__setattr__`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__setattr__`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 132-156 / 第 132-156 行
````python
class cuBLASModule:
    @staticmethod
    def _parse_reduction_setting(value: Any, attr_name: str) -> tuple[bool, bool]:
        def _ensure_bool(obj: Any, which: str) -> bool:
            if isinstance(obj, bool):
                return obj
            raise TypeError(
                f"{attr_name} expects a bool for {which}, but got {type(obj)!r}"
            )

        if isinstance(value, bool):
            return value, True
        if isinstance(value, (list, tuple)):
            if not value:
                raise TypeError(f"{attr_name} expects at least one boolean argument")
            if len(value) > 2:
                raise TypeError(f"{attr_name} expects at most two boolean arguments")
            allow_reduced_precision = _ensure_bool(value[0], "allow_reduced_precision")
            if len(value) == 1:
                return allow_reduced_precision, True
            allow_splitk = _ensure_bool(value[1], "allow_splitk")
            return allow_reduced_precision, allow_splitk
        raise TypeError(
            f"{attr_name} expects a bool or a tuple/list of bools, but got {type(value)!r}"
        )
````
- **EN**: It introduces or extends `cuBLASModule`, which hold the main object-oriented state for this portion of the file. This chunk defines `_ensure_bool`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `cuBLASModule`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `_ensure_bool`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 158-185 / 第 158-185 行
````python
    def __getattr__(self, name):
        if name == "allow_tf32":
            return torch._C._get_cublas_allow_tf32()
        elif name == "allow_fp16_reduced_precision_reduction":
            allow_reduced_precision, _ = (
                torch._C._get_cublas_allow_fp16_reduced_precision_reduction()
            )
            return allow_reduced_precision
        elif name == "allow_fp16_reduced_precision_reduction_split_k":
            _, allow_splitk = (
                torch._C._get_cublas_allow_fp16_reduced_precision_reduction()
            )
            return allow_splitk
        elif name == "allow_bf16_reduced_precision_reduction":
            allow_reduced_precision, _ = (
                torch._C._get_cublas_allow_bf16_reduced_precision_reduction()
            )
            return allow_reduced_precision
        elif name == "allow_bf16_reduced_precision_reduction_split_k":
            _, allow_splitk = (
                torch._C._get_cublas_allow_bf16_reduced_precision_reduction()
            )
            return allow_splitk
        elif name == "allow_fp16_accumulation":
            return torch._C._get_cublas_allow_fp16_accumulation()
        elif name == "fp32_precision":
            return torch._C._get_fp32_precision_getter("cuda", "matmul")
        raise AttributeError("Unknown attribute " + name)
````
- **EN**: This chunk defines `__getattr__`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__getattr__`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 187-210 / 第 187-210 行
````python
    def __setattr__(self, name, value):
        if name == "allow_tf32":
            return torch._C._set_cublas_allow_tf32(value)
        elif name == "allow_fp16_reduced_precision_reduction":
            allow_reduced_precision, allow_splitk = self._parse_reduction_setting(
                value, "allow_fp16_reduced_precision_reduction"
            )
            return torch._C._set_cublas_allow_fp16_reduced_precision_reduction(
                allow_reduced_precision,
                allow_splitk,
            )
        elif name == "allow_bf16_reduced_precision_reduction":
            allow_reduced_precision, allow_splitk = self._parse_reduction_setting(
                value, "allow_bf16_reduced_precision_reduction"
            )
            return torch._C._set_cublas_allow_bf16_reduced_precision_reduction(
                allow_reduced_precision,
                allow_splitk,
            )
        elif name == "allow_fp16_accumulation":
            return torch._C._set_cublas_allow_fp16_accumulation(value)
        elif name == "fp32_precision":
            return torch._C._set_fp32_precision_setter("cuda", "matmul", value)
        raise AttributeError("Unknown attribute " + name)
````
- **EN**: This chunk defines `__setattr__`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__setattr__`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 213-231 / 第 213-231 行
````python
_LinalgBackends = {
    "default": torch._C._LinalgBackend.Default,
    "cusolver": torch._C._LinalgBackend.Cusolver,
    "magma": torch._C._LinalgBackend.Magma,
}
_LinalgBackends_str = ", ".join(_LinalgBackends.keys())


def preferred_linalg_library(
    backend: None | str | torch._C._LinalgBackend = None,
) -> torch._C._LinalgBackend:
    r"""
    Override the heuristic PyTorch uses to choose between cuSOLVER and MAGMA for CUDA linear algebra operations.

    .. warning:: This flag is experimental and subject to change.

    When PyTorch runs a CUDA linear algebra operation it often uses the cuSOLVER or MAGMA libraries,
    and if both are available it decides which to use with a heuristic.
    This flag (a :class:`str`) allows overriding those heuristics.
````
- **EN**: This chunk defines `preferred_linalg_library`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `preferred_linalg_library`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 233-260 / 第 233-260 行
````python
    * If `"cusolver"` is set then cuSOLVER will be used wherever possible.
    * If `"magma"` is set then MAGMA will be used wherever possible.
    * If `"default"` (the default) is set then heuristics will be used to pick between
      cuSOLVER and MAGMA if both are available.
    * When no input is given, this function returns the currently preferred library.
    * User may use the environment variable TORCH_LINALG_PREFER_CUSOLVER=1 to set the preferred library to cuSOLVER
      globally.
      This flag only sets the initial value of the preferred library and the preferred library
      may still be overridden by this function call later in your script.

    Note: When a library is preferred other libraries may still be used if the preferred library
    doesn't implement the operation(s) called.
    This flag may achieve better performance if PyTorch's heuristic library selection is incorrect
    for your application's inputs.

    Currently supported linalg operators:

    * :func:`torch.linalg.inv`
    * :func:`torch.linalg.inv_ex`
    * :func:`torch.linalg.cholesky`
    * :func:`torch.linalg.cholesky_ex`
    * :func:`torch.cholesky_solve`
    * :func:`torch.cholesky_inverse`
    * :func:`torch.linalg.lu_factor`
    * :func:`torch.linalg.lu`
    * :func:`torch.linalg.lu_solve`
    * :func:`torch.linalg.qr`
    * :func:`torch.linalg.eigh`
````
- **EN**: This chunk continues `preferred_linalg_library` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `preferred_linalg_library`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 261-278 / 第 261-278 行
````python
    * :func:`torch.linalg.eighvals`
    * :func:`torch.linalg.svd`
    * :func:`torch.linalg.svdvals`
    """
    if backend is None:
        pass
    elif isinstance(backend, str):
        if backend not in _LinalgBackends:
            raise RuntimeError(
                f"Unknown input value. Choose from: {_LinalgBackends_str}."
            )
        torch._C._set_linalg_preferred_backend(_LinalgBackends[backend])
    elif isinstance(backend, torch._C._LinalgBackend):
        torch._C._set_linalg_preferred_backend(backend)
    else:
        raise RuntimeError("Unknown input value type.")

    return torch._C._get_linalg_preferred_backend()
````
- **EN**: This chunk continues `preferred_linalg_library` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `preferred_linalg_library`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 281-302 / 第 281-302 行
````python
_BlasBackends = {
    "default": torch._C._BlasBackend.Default,
    "cublas": torch._C._BlasBackend.Cublas,
    "hipblas": torch._C._BlasBackend.Cublas,  # alias
    "cublaslt": torch._C._BlasBackend.Cublaslt,
    "hipblaslt": torch._C._BlasBackend.Cublaslt,  # alias
    "ck": torch._C._BlasBackend.Ck,
}
_BlasBackends_str = ", ".join(_BlasBackends.keys())


def preferred_blas_library(
    backend: None | str | torch._C._BlasBackend = None,
) -> torch._C._BlasBackend:
    r"""
    Override the library PyTorch uses for BLAS operations. Choose between cuBLAS, cuBLASLt, and CK [ROCm-only].

    .. warning:: This flag is experimental and subject to change.

    When PyTorch runs a CUDA BLAS operation it defaults to cuBLAS even if both cuBLAS and cuBLASLt are available.
    For PyTorch built for ROCm, hipBLAS, hipBLASLt, and CK may offer different performance.
    This flag (a :class:`str`) allows overriding which BLAS library to use.
````
- **EN**: This chunk defines `preferred_blas_library`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `preferred_blas_library`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 304-331 / 第 304-331 行
````python
    * If `"cublas"` is set then cuBLAS will be used wherever possible.
    * If `"cublaslt"` is set then cuBLASLt will be used wherever possible.
    * If `"ck"` is set then CK will be used wherever possible.
    * If `"default"` (the default) is set then heuristics will be used to pick between the other options.
    * When no input is given, this function returns the currently preferred library.
    * User may use the environment variable TORCH_BLAS_PREFER_CUBLASLT=1 to set the preferred library to cuBLASLt
      globally.
      This flag only sets the initial value of the preferred library and the preferred library
      may still be overridden by this function call later in your script.

    Note: When a library is preferred other libraries may still be used if the preferred library
    doesn't implement the operation(s) called.
    This flag may achieve better performance if PyTorch's library selection is incorrect
    for your application's inputs.

    """
    if backend is None:
        pass
    elif isinstance(backend, str):
        if backend not in _BlasBackends:
            raise RuntimeError(
                f"Unknown input value. Choose from: {_BlasBackends_str}."
            )
        torch._C._set_blas_preferred_backend(_BlasBackends[backend])
    elif isinstance(backend, torch._C._BlasBackend):
        torch._C._set_blas_preferred_backend(backend)
    else:
        raise RuntimeError("Unknown input value type.")
````
- **EN**: This chunk continues `preferred_blas_library` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `preferred_blas_library`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 333-356 / 第 333-356 行
````python
    return torch._C._get_blas_preferred_backend()


def cublas_workspace_size(size: None | int = None) -> int:
    r"""Query or set the cuBLAS workspace size in bytes.

    When called with no arguments, returns the current workspace size.
    When called with a size argument, sets the workspace size and returns the new value.
    Setting the workspace size will take precedence over the CUBLAS_WORKSPACE_CONFIG environment variable.
    Changes take effect lazily: only handles used after the change get new workspaces.

    Args:
        size (int, optional): workspace size in bytes. Must be non-negative.

    Returns:
        int: the current (or newly set) workspace size in bytes.
    """
    if size is not None:
        torch._C._cuda_setCublasWorkspaceSize(size)
    return torch._C._cuda_getCublasWorkspaceSize()


def cublaslt_workspace_size(size: None | int = None) -> int:
    r"""Query or set the cuBLASLt workspace size in bytes.
````
- **EN**: This chunk defines `cublaslt_workspace_size`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `cublaslt_workspace_size`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 358-381 / 第 358-381 行
````python
    When called with no arguments, returns the current workspace size.
    When called with a size argument, sets the workspace size and returns the new value.
    Setting the workspace size will take precedence over the CUBLASLT_WORKSPACE_SIZE environment variable.
    Changes take effect lazily: only handles used after the change get new workspaces.

    Args:
        size (int, optional): workspace size in bytes. Must be non-negative.

    Returns:
        int: the current (or newly set) workspace size in bytes.
    """
    if size is not None:
        torch._C._cuda_setCublasLtWorkspaceSize(size)
    return torch._C._cuda_getCublasLtWorkspaceSize()


def blas_workspace_size(
    size: None | int = None,
    backend: None | str | torch._C._BlasBackend = None,
) -> int:
    r"""Query or set the BLAS workspace size for a given backend.

    Convenience wrapper that dispatches to :func:`cublas_workspace_size` or
    :func:`cublaslt_workspace_size` depending on the backend.
````
- **EN**: This chunk defines `blas_workspace_size`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `blas_workspace_size`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 383-407 / 第 383-407 行
````python
    When *backend* is ``None`` the current :func:`preferred_blas_library` is
    used.  ``Default`` is resolved to the platform's default backend (cuBLAS
    on NVIDIA, potentially hipBLASLt on supported ROCm architectures).

    .. note::

       When ``TORCH_CUBLASLT_UNIFIED_WORKSPACE`` is enabled (the default on
       open-source CUDA builds), the cuBLASLt workspace is capped at the
       cuBLAS workspace size and physically reuses the same allocation.
       Setting a large cuBLASLt workspace via this function will therefore
       *not* increase memory beyond the cuBLAS workspace size.

    .. note::

        Setting the workspace size for the cublas backend will take precedence
        over the CUBLAS_WORKSPACE_CONFIG environment variable, and setting the
        workspace size for the cublaslt backend will take precedence over the
        CUBLASLT_WORKSPACE_SIZE environment variable.

    Args:
        size (int, optional): workspace size in bytes.  Must be non-negative.
            When omitted the current size is returned without modification.
        backend (str | torch._C._BlasBackend, optional): which backend's
            workspace to query/set.  Accepts the same strings as
            :func:`preferred_blas_library` (e.g. ``"cublas"``, ``"cublaslt"``).
````
- **EN**: This chunk continues `blas_workspace_size` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `blas_workspace_size`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 409-432 / 第 409-432 行
````python
    Returns:
        int: the current (or newly set) workspace size in bytes.

    Raises:
        RuntimeError: if the resolved backend is CK (no workspace concept).
    """
    if backend is None:
        resolved = preferred_blas_library()
    elif isinstance(backend, str):
        if backend not in _BlasBackends:
            raise RuntimeError(
                f"Unknown backend string. Choose from: {_BlasBackends_str}."
            )
        resolved = _BlasBackends[backend]
    elif isinstance(backend, torch._C._BlasBackend):
        resolved = backend
    else:
        raise RuntimeError("Unknown backend type.")

    if resolved == torch._C._BlasBackend.Default:
        resolved = torch._C._get_blas_default_backend()

    if resolved == torch._C._BlasBackend.Ck:
        raise RuntimeError("CK backend does not use a workspace.")
````
- **EN**: This chunk continues `blas_workspace_size` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `blas_workspace_size`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 434-460 / 第 434-460 行
````python
    if resolved == torch._C._BlasBackend.Cublaslt:
        return cublaslt_workspace_size(size)
    return cublas_workspace_size(size)


_ROCmFABackends = {
    "default": torch._C._ROCmFABackend.Default,
    "aotriton": torch._C._ROCmFABackend.AOTriton,
    "ck": torch._C._ROCmFABackend.Ck,
}
_ROCmFABackends_str = ", ".join(_ROCmFABackends.keys())


from torch._C import _SDPAParams as SDPAParams, _SDPBackend as SDPBackend


def preferred_rocm_fa_library(
    backend: None | str | torch._C._ROCmFABackend = None,
) -> torch._C._ROCmFABackend:
    r"""
    [ROCm-only]
    Override the backend PyTorch uses in ROCm environments for Flash Attention. Choose between AOTriton and CK

    .. warning:: This flag is experimental and subject to change.

    When Flash Attention is enabled and desired, PyTorch defaults to using AOTriton as the backend.
    This flag (a :class:`str`) allows users to override this backend to use composable_kernel
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._C. This chunk defines `preferred_rocm_fa_library`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._C。 这一段定义了 `preferred_rocm_fa_library`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 462-487 / 第 462-487 行
````python
    * If `"default"` is set then the default backend will be used wherever possible. Currently AOTriton.
    * If `"aotriton"` is set then AOTriton will be used wherever possible.
    * If `"ck"` is set then CK will be used wherever possible.
    * When no input is given, this function returns the currently preferred library.
    * User may use the environment variable TORCH_ROCM_FA_PREFER_CK=1 to set the preferred library to CK
      globally.

    Note: When a library is preferred other libraries may still be used if the preferred library
    doesn't implement the operation(s) called.
    This flag may achieve better performance if PyTorch's library selection is incorrect
    for your application's inputs.
    """
    if backend is None:
        pass
    elif isinstance(backend, str):
        if backend not in _ROCmFABackends:
            raise RuntimeError(
                f"Unknown input value. Choose from: {_ROCmFABackends_str}."
            )
        torch._C._set_rocm_fa_preferred_backend(_ROCmFABackends[backend])
    elif isinstance(backend, torch._C._ROCmFABackend):
        torch._C._set_rocm_fa_preferred_backend(backend)
    else:
        raise ValueError(f"Unknown input value. Choose from: {_ROCmFABackends_str}.")

    return torch._C._get_rocm_fa_preferred_backend()
````
- **EN**: This chunk continues `preferred_rocm_fa_library` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `preferred_rocm_fa_library`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 490-512 / 第 490-512 行
````python
# Set the __module__ attribute
SDPAParams.__module__ = "torch.backends.cuda"
SDPAParams.__name__ = "SDPAParams"


def is_ck_sdpa_available() -> bool:
    r"""
    .. warning:: This flag is beta and subject to change.

    Returns whether composable_kernel may be used as the backend for
    scaled-dot-product-attention.
    """
    # pyrefly: ignore [missing-attribute]
    return torch._C._is_ck_sdpa_available()


def flash_sdp_enabled():
    r"""
    .. warning:: This flag is beta and subject to change.

    Returns whether flash scaled dot product attention is enabled or not.
    """
    return torch._C._get_flash_sdp_enabled()
````
- **EN**: This chunk defines `flash_sdp_enabled`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `flash_sdp_enabled`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 515-539 / 第 515-539 行
````python
def enable_flash_sdp(enabled: bool):
    r"""
    .. warning:: This flag is beta and subject to change.

    Enables or disables flash scaled dot product attention.
    """
    torch._C._set_sdp_use_flash(enabled)


def mem_efficient_sdp_enabled():
    r"""
    .. warning:: This flag is beta and subject to change.

    Returns whether memory efficient scaled dot product attention is enabled or not.
    """
    return torch._C._get_mem_efficient_sdp_enabled()


def enable_mem_efficient_sdp(enabled: bool):
    r"""
    .. warning:: This flag is beta and subject to change.

    Enables or disables memory efficient scaled dot product attention.
    """
    torch._C._set_sdp_use_mem_efficient(enabled)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `enable_mem_efficient_sdp`, which mutates configuration or backend state that affects later execution. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `enable_mem_efficient_sdp`，其作用是修改会影响后续执行的配置或后端状态。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 542-566 / 第 542-566 行
````python
def math_sdp_enabled():
    r"""
    .. warning:: This flag is beta and subject to change.

    Returns whether math scaled dot product attention is enabled or not.
    """
    return torch._C._get_math_sdp_enabled()


def enable_math_sdp(enabled: bool):
    r"""
    .. warning:: This flag is beta and subject to change.

    Enables or disables math scaled dot product attention.
    """
    torch._C._set_sdp_use_math(enabled)


def allow_fp16_bf16_reduction_math_sdp(enabled: bool):
    r"""
    .. warning:: This flag is beta and subject to change.

    Enables or disables fp16/bf16 reduction in math scaled dot product attention.
    """
    torch._C._set_math_sdp_allow_fp16_bf16_reduction(enabled)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `allow_fp16_bf16_reduction_math_sdp`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `allow_fp16_bf16_reduction_math_sdp`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 569-592 / 第 569-592 行
````python
def fp16_bf16_reduction_math_sdp_allowed():
    r"""
    .. warning:: This flag is beta and subject to change.

    Returns whether fp16/bf16 reduction in math scaled dot product attention is enabled or not.
    """
    return torch._C._get_math_sdp_allow_fp16_bf16_reduction()


def is_flash_attention_available() -> bool:
    r"""Check if PyTorch was built with FlashAttention for scaled_dot_product_attention.

    Returns:
        True if FlashAttention is built and available; otherwise, False.

    Note:
        This function is dependent on a CUDA-enabled build of PyTorch. It will return False
        in non-CUDA environments.
    """
    return torch._C._is_flash_attention_available()


def can_use_flash_attention(params: SDPAParams, debug: bool = False) -> bool:
    r"""Check if FlashAttention can be utilized in scaled_dot_product_attention.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `can_use_flash_attention`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `can_use_flash_attention`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 594-619 / 第 594-619 行
````python
    Args:
        params: An instance of SDPAParams containing the tensors for query,
                key, value, an optional attention mask, dropout rate, and
                a flag indicating if the attention is causal.
        debug: Whether to logging.warn debug information as to why FlashAttention could not be run.
            Defaults to False.

    Returns:
        True if FlashAttention can be used with the given parameters; otherwise, False.

    Note:
        This function is dependent on a CUDA-enabled build of PyTorch. It will return False
        in non-CUDA environments.
    """
    return torch._C._can_use_flash_attention(params, debug)


def can_use_efficient_attention(params: SDPAParams, debug: bool = False) -> bool:
    r"""Check if efficient_attention can be utilized in scaled_dot_product_attention.

    Args:
        params: An instance of SDPAParams containing the tensors for query,
                key, value, an optional attention mask, dropout rate, and
                a flag indicating if the attention is causal.
        debug: Whether to logging.warn with information as to why efficient_attention could not be run.
            Defaults to False.
````
- **EN**: This chunk defines `can_use_efficient_attention`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `can_use_efficient_attention`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 621-642 / 第 621-642 行
````python
    Returns:
        True if efficient_attention can be used with the given parameters; otherwise, False.

    Note:
        This function is dependent on a CUDA-enabled build of PyTorch. It will return False
        in non-CUDA environments.
    """
    return torch._C._can_use_mem_efficient_attention(params, debug)


def can_use_cudnn_attention(params: SDPAParams, debug: bool = False) -> bool:
    r"""Check if cudnn_attention can be utilized in scaled_dot_product_attention.

    Args:
        params: An instance of SDPAParams containing the tensors for query,
                key, value, an optional attention mask, dropout rate, and
                a flag indicating if the attention is causal.
        debug: Whether to logging.warn with information as to why cuDNN attention could not be run.
            Defaults to False.

    Returns:
        True if cuDNN can be used with the given parameters; otherwise, False.
````
- **EN**: This chunk defines `can_use_cudnn_attention`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `can_use_cudnn_attention`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 644-666 / 第 644-666 行
````python
    Note:
        This function is dependent on a CUDA-enabled build of PyTorch. It will return False
        in non-CUDA environments.
    """
    return torch._C._can_use_cudnn_attention(params, debug)


def cudnn_sdp_enabled():
    r"""
    .. warning:: This flag is beta and subject to change.

    Returns whether cuDNN scaled dot product attention is enabled or not.
    """
    return torch._C._get_cudnn_sdp_enabled()


def enable_cudnn_sdp(enabled: bool):
    r"""
    .. warning:: This flag is beta and subject to change.

    Enables or disables cuDNN scaled dot product attention.
    """
    torch._C._set_sdp_use_cudnn(enabled)
````
- **EN**: This chunk defines `enable_cudnn_sdp`, which mutates configuration or backend state that affects later execution. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `enable_cudnn_sdp`，其作用是修改会影响后续执行的配置或后端状态。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 669-691 / 第 669-691 行
````python
@contextlib.contextmanager
@deprecated(
    (
        "`torch.backends.cuda.sdp_kernel()` is deprecated. "
        "In the future, this context manager will be removed. "
        "Please see `torch.nn.attention.sdpa_kernel()` for the new context manager, "
        "with updated signature."
    ),
    category=FutureWarning,
)
def sdp_kernel(
    enable_flash: bool = True,
    enable_math: bool = True,
    enable_mem_efficient: bool = True,
    enable_cudnn: bool = True,
):
    r"""
    .. warning:: This flag is beta and subject to change.

    This context manager can be used to temporarily enable or disable any of the three backends for scaled dot product attention.
    Upon exiting the context manager, the previous state of the flags will be restored.
    """
    from torch.nn.attention import sdpa_kernel
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.nn.attention. This chunk defines `sdp_kernel`, which implements a focused helper used by the surrounding module. Decorators such as `contextlib.contextmanager`, `deprecated` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.nn.attention。 这一段定义了 `sdp_kernel`，其作用是实现周边模块使用的关键辅助逻辑。 像 `contextlib.contextmanager`、`deprecated` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 693-711 / 第 693-711 行
````python
    backend_list = []
    if enable_flash:
        backend_list.append(SDPBackend.FLASH_ATTENTION)
    if enable_mem_efficient:
        backend_list.append(SDPBackend.EFFICIENT_ATTENTION)
    if enable_math:
        backend_list.append(SDPBackend.MATH)
    if enable_cudnn:
        backend_list.append(SDPBackend.CUDNN_ATTENTION)

    with sdpa_kernel(backend_list) as context:
        try:
            yield context
        finally:
            pass


cufft_plan_cache = cuFFTPlanCacheManager()
matmul = cuBLASModule()
````
- **EN**: This chunk continues `sdp_kernel` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `sdp_kernel`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Backend configuration**
  - EN: Exposes optional-runtime toggles and capability checks that alter low-level execution behavior.
  - CN: 暴露可选运行时开关与能力检查，以改变底层执行行为。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **is_built**
  - EN: `is_built` is one of the main symbols declared or implemented in this file.
  - CN: `is_built` 是本文件声明或实现的主要符号之一。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
- **Package wiring**
  - EN: The file shapes the public namespace by re-exporting symbols and applying package-level initialization.
  - CN: 该文件通过重新导出符号并执行包级初始化来塑造公共命名空间。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._C`, `torch.nn.attention`
- **Standard library / 标准库**: `contextlib`, `typing`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `is_built`, `cuFFTPlanCacheAttrContextProp`, `cuFFTPlanCache`, `cuFFTPlanCacheManager`, `cuBLASModule`, `preferred_linalg_library`, `preferred_blas_library`, `cublas_workspace_size`, `cublaslt_workspace_size`
