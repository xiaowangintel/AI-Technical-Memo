# _utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/cuda/_utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-facing Python wrappers for streams, memory, random state, and runtime management.
- **Purpose (CN)**: 实现面向 CUDA 的 Python 包装层，用于流、内存、随机状态以及运行时管理。
## Line-by-Line Analysis / 逐行分析

### Lines 1-19 / 第 1-19 行
````python
import ctypes
import sys
from typing import Any

import torch


try:
    from cuda.bindings import (  # pyrefly: ignore[missing-import]
        runtime as _cuda_bindings_runtime,
    )

    _HAS_CUDA_BINDINGS = True
except ImportError:
    _cuda_bindings_runtime = None  # type: ignore[assignment]
    _HAS_CUDA_BINDINGS = False

# The _get_device_index has been moved to torch.utils._get_device_index
from torch._utils import _get_device_index as _torch_get_device_index
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._utils; standard-library helpers such as ctypes, sys, typing; other helper packages such as cuda.bindings. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._utils；标准库辅助模块，如 ctypes、sys、typing；其他辅助包，如 cuda.bindings。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。

### Lines 22-35 / 第 22-35 行
````python
def _get_hip_runtime_library() -> ctypes.CDLL:
    # If ROCm python packages are available, query the OS-independent absolute
    # path to the library provided by those packages, including any version suffix.
    # See https://github.com/ROCm/TheRock/blob/main/docs/packaging/python_packaging.md#dynamic-library-resolution
    try:
        # pyrefly: ignore [import-error, missing-import]
        import rocm_sdk

        lib = ctypes.CDLL(str(rocm_sdk.find_libraries("amdhip64")[0]))
    except (ImportError, IndexError):
        if sys.platform == "win32":
            lib = ctypes.CDLL(f"amdhip64_{torch.version.hip[0]}.dll")
        else:  # Unix-based systems
            lib = ctypes.CDLL("libamdhip64.so")
````
- **EN**: This block assembles module dependencies, pulling in other helper packages such as rocm_sdk. This chunk defines `_get_hip_runtime_library`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了其他辅助包，如 rocm_sdk。 这一段定义了 `_get_hip_runtime_library`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 37-49 / 第 37-49 行
````python
    lib.cuGetErrorString = lib.hipGetErrorString  # type: ignore[attr-defined]
    lib.cuModuleLoadData = lib.hipModuleLoadData  # type: ignore[attr-defined]
    lib.cuModuleGetFunction = lib.hipModuleGetFunction  # type: ignore[attr-defined]
    lib.cuLaunchKernel = lib.hipModuleLaunchKernel  # type: ignore[attr-defined]
    lib.cuFuncSetAttribute = lib.hipFuncSetAttribute  # type: ignore[attr-defined]
    return lib


def _get_cuda_library() -> ctypes.CDLL:
    if sys.platform == "win32":
        return ctypes.CDLL("nvcuda.dll")
    else:  # Unix-based systems
        return ctypes.CDLL("libcuda.so.1")
````
- **EN**: This chunk defines `_get_cuda_library`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_cuda_library`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 52-70 / 第 52-70 行
````python
# Load GPU driver runtime
def _get_gpu_runtime_library() -> ctypes.CDLL:
    if torch.version.hip:
        return _get_hip_runtime_library()
    else:
        return _get_cuda_library()


# Helper: check CUDA errors
def _check_cuda(result: int) -> None:
    if result == 0:
        return
    err_str = ctypes.c_char_p()
    libcuda = _get_gpu_runtime_library()  # Get reference to CUDA library
    libcuda.cuGetErrorString(result, ctypes.byref(err_str))
    error_message = (
        err_str.value.decode() if err_str.value is not None else "Unknown CUDA error"
    )
    raise RuntimeError(f"CUDA error: {error_message}")
````
- **EN**: This chunk defines `_check_cuda`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_check_cuda`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 73-92 / 第 73-92 行
````python
def _check_cuda_bindings(result: Any) -> Any:
    """Check a cuda.bindings (cuda-python) call result for errors.

    All cuda.bindings runtime calls return ``(error, *outputs)``.  This
    helper unpacks the tuple, raises on non-success, and returns the
    outputs (``None`` for zero outputs, scalar for one, tuple otherwise).
    """
    if not _HAS_CUDA_BINDINGS:
        raise RuntimeError("cuda.bindings is not available")
    err, *out = result
    if (
        err
        != _cuda_bindings_runtime.cudaError_t.cudaSuccess  # pyrefly: ignore[missing-attribute]
    ):
        _, err_str = (
            _cuda_bindings_runtime.cudaGetErrorString(  # pyrefly: ignore[missing-attribute]
                err
            )
        )
        if isinstance(err_str, bytes):
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_check_cuda_bindings`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_check_cuda_bindings`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 93-105 / 第 93-105 行
````python
            err_str = err_str.decode()
        raise RuntimeError(f"CUDA error: {err} ({err_str})")
    if len(out) == 0:
        return None
    if len(out) == 1:
        return out[0]
    return out


def _get_hiprtc_library() -> ctypes.CDLL:
    try:
        # pyrefly: ignore [import-error, missing-import]
        import rocm_sdk
````
- **EN**: This block assembles module dependencies, pulling in other helper packages such as rocm_sdk. This chunk defines `_get_hiprtc_library`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了其他辅助包，如 rocm_sdk。 这一段定义了 `_get_hiprtc_library`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 107-126 / 第 107-126 行
````python
        lib = ctypes.CDLL(str(rocm_sdk.find_libraries("hiprtc")[0]))
    except (ImportError, IndexError):
        if sys.platform == "win32":
            version_str = "".join(
                ["0", torch.version.hip[0], "0", torch.version.hip[2]]
            )
            lib = ctypes.CDLL(f"hiprtc{version_str}.dll")
        else:
            lib = ctypes.CDLL("libhiprtc.so")

    # Provide aliases for HIP RTC functions to match NVRTC API
    lib.nvrtcGetErrorString = lib.hiprtcGetErrorString  # type: ignore[attr-defined]
    lib.nvrtcCreateProgram = lib.hiprtcCreateProgram  # type: ignore[attr-defined]
    lib.nvrtcDestroyProgram = lib.hiprtcDestroyProgram  # type: ignore[attr-defined]
    lib.nvrtcCompileProgram = lib.hiprtcCompileProgram  # type: ignore[attr-defined]
    lib.nvrtcGetCUBINSize = lib.hiprtcGetCodeSize  # type: ignore[attr-defined]
    lib.nvrtcGetCUBIN = lib.hiprtcGetCode  # type: ignore[attr-defined]
    lib.nvrtcGetProgramLogSize = lib.hiprtcGetProgramLogSize  # type: ignore[attr-defined]
    lib.nvrtcGetProgramLog = lib.hiprtcGetProgramLog  # type: ignore[attr-defined]
    lib.nvrtcAddNameExpression = lib.hiprtcAddNameExpression  # type: ignore[attr-defined]
````
- **EN**: This chunk continues `_get_hiprtc_library` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_get_hiprtc_library`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 127-146 / 第 127-146 行
````python
    lib.nvrtcGetLoweredName = lib.hiprtcGetLoweredName  # type: ignore[attr-defined]
    return lib


def _get_nvrtc_library() -> ctypes.CDLL:
    major_version = int(torch.version.cuda.split(".")[0])  # type: ignore[union-attr]
    if sys.platform == "win32":
        nvrtc_libs = [
            f"nvrtc64_{major_version}0_0.dll",
        ]
    else:
        nvrtc_libs = [
            f"libnvrtc.so.{major_version}",
            "libnvrtc.so",  # Fallback to unversioned
        ]
    for lib_name in nvrtc_libs:
        try:
            return ctypes.CDLL(lib_name)
        except OSError:
            continue
````
- **EN**: This chunk defines `_get_nvrtc_library`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_nvrtc_library`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 147-161 / 第 147-161 行
````python
    raise OSError("Could not find any NVRTC library")


def _get_gpu_rtc_library() -> ctypes.CDLL:
    # Since PyTorch already loads the GPU RTC library, we can use the system library
    # which should be compatible with PyTorch's version
    if torch.version.hip:
        return _get_hiprtc_library()
    else:
        return _get_nvrtc_library()


def _get_gpu_rtc_compatible_flags() -> list[str]:
    """
    Get HIPCC/NVCC flags that are compatible with NVRTC compilation.
````
- **EN**: This chunk defines `_get_gpu_rtc_compatible_flags`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_gpu_rtc_compatible_flags`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 163-180 / 第 163-180 行
````python
    Returns:
        List of HIPCC/NVCC flags that can be safely used with NVRTC.
    """
    from torch.utils.cpp_extension import COMMON_HIPCC_FLAGS, COMMON_NVCC_FLAGS

    nvrtc_unsupported_flags = {
        "--expt-relaxed-constexpr",
    }

    # Filter out unsupported flags
    compatible_flags = [
        flag for flag in COMMON_NVCC_FLAGS if flag not in nvrtc_unsupported_flags
    ]

    if torch.version.hip:
        compatible_flags.extend(COMMON_HIPCC_FLAGS)

    return compatible_flags
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.utils.cpp_extension. This chunk continues `_get_gpu_rtc_compatible_flags` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.utils.cpp_extension。 这一段延续了 `_get_gpu_rtc_compatible_flags`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 183-201 / 第 183-201 行
````python
def _nvrtc_compile(
    kernel_source: str,
    kernel_name: str,
    compute_capability: str | None = None,
    cuda_include_dirs: list | None = None,
    nvcc_options: list | None = None,
    auto_pch: bool = False,
) -> tuple[bytes, str]:
    """
    Compiles a CUDA kernel using NVRTC and returns the PTX code.

    Args:
        kernel_source (str): The CUDA kernel source code as a string
        kernel_name (str): The name of the kernel function to compile
        compute_capability (str, None): The compute capability to target (e.g., "86").
                                           If None, will detect from current device.
        cuda_include_dirs (list, None): List of directories containing CUDA headers
        nvcc_options (list, None): Additional options to pass to NVRTC
        auto_pch (bool): Enable automatic precompiled headers (CUDA 12.8+)
````
- **EN**: This chunk defines `_nvrtc_compile`, which coordinates graph-oriented state needed for tracing, capture, or compilation.
- **CN**: 这一段定义了 `_nvrtc_compile`，其作用是协调 tracing、捕获或编译所需的图相关状态。

### Lines 203-222 / 第 203-222 行
````python
    Returns:
        Tuple[bytes, str]: The compiled PTX code and mangled kernel name
    """
    # Ensure CUDA is initialized
    import torch.cuda

    # Load NVRTC library
    libnvrtc = _get_gpu_rtc_library()

    # NVRTC constants
    NVRTC_SUCCESS = 0

    # Helper: check NVRTC errors
    def check_nvrtc(result: int) -> None:
        if result != NVRTC_SUCCESS:
            err_str = ctypes.c_char_p()
            libnvrtc.nvrtcGetErrorString(result, ctypes.byref(err_str))
            error_message = (
                err_str.value.decode()
                if err_str.value is not None
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.cuda. This chunk defines `check_nvrtc`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.cuda。 这一段定义了 `check_nvrtc`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 223-236 / 第 223-236 行
````python
                else "Unknown CUDA error"
            )
            raise RuntimeError(f"CUDA error: {error_message}")

    # Convert source to bytes
    source_bytes = kernel_source.encode("utf-8")

    # Get compute capability if not provided
    if compute_capability is None:
        props = torch.cuda.get_device_properties(torch.cuda.current_device())
        if torch.version.hip:
            compute_capability = f"{props.gcnArchName}"
        else:
            compute_capability = f"{props.major}{props.minor}"
````
- **EN**: This chunk continues `check_nvrtc` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `check_nvrtc`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 238-255 / 第 238-255 行
````python
    # Prepare compilation options
    options = []
    if torch.version.hip:
        options.append(f"--offload-arch={compute_capability}".encode())
    else:
        options.append(f"--gpu-architecture=sm_{compute_capability}".encode())

    # Auto-detect and add CUDA include paths
    from torch.utils.cpp_extension import include_paths

    cuda_include_paths = include_paths("cuda")
    for cuda_path in cuda_include_paths:
        options.append(f"-I{cuda_path}".encode())

    # Add custom include directories
    if cuda_include_dirs:
        for directory in cuda_include_dirs:
            options.append(f"-I{directory}".encode())
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.utils.cpp_extension. This chunk continues `check_nvrtc` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.utils.cpp_extension。 这一段延续了 `check_nvrtc`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 257-275 / 第 257-275 行
````python
    # Enable automatic precompiled headers (CUDA 12.8+)
    if auto_pch:
        if str(torch.version.cuda) < "12.8":
            raise AssertionError(f"PCH requires CUDA 12.8+, got {torch.version.cuda}")
        if nvcc_options is None:
            nvcc_options = []
        nvcc_options.append("--pch")

    # Add custom NVCC options
    if nvcc_options:
        for option in nvcc_options:
            options.append(option.encode("utf-8"))

    nvrtc_compatible_flags = _get_gpu_rtc_compatible_flags()
    options.extend([flag.encode("utf-8") for flag in nvrtc_compatible_flags])

    # Convert options to C array
    num_options = len(options)
    options_array = (ctypes.c_char_p * num_options)(*options)
````
- **EN**: This chunk continues `check_nvrtc` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `check_nvrtc`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 277-295 / 第 277-295 行
````python
    # Create program
    prog = ctypes.c_void_p()
    check_nvrtc(
        libnvrtc.nvrtcCreateProgram(
            ctypes.byref(prog),
            source_bytes,
            f"{kernel_name}.cu".encode(),
            0,
            None,
            None,
        )
    )

    # Add kernel name, which can be a template expression
    c_kernel_name = kernel_name.encode("utf-8")
    check_nvrtc(libnvrtc.nvrtcAddNameExpression(prog, c_kernel_name))

    # Compile program
    res = libnvrtc.nvrtcCompileProgram(prog, num_options, options_array)
````
- **EN**: This chunk continues `check_nvrtc` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `check_nvrtc`，进一步展开其内部控制流或状态更新。

### Lines 297-310 / 第 297-310 行
````python
    # Handle compilation errors
    if res != NVRTC_SUCCESS:
        # Get log
        log_size = ctypes.c_size_t()
        libnvrtc.nvrtcGetProgramLogSize(prog, ctypes.byref(log_size))
        log = ctypes.create_string_buffer(log_size.value)
        libnvrtc.nvrtcGetProgramLog(prog, log)
        raise RuntimeError(f"Kernel compilation failed:\n{log.value.decode()}")

    # Get binary
    binary_size = ctypes.c_size_t()
    check_nvrtc(libnvrtc.nvrtcGetCUBINSize(prog, ctypes.byref(binary_size)))
    binary = ctypes.create_string_buffer(binary_size.value)
    check_nvrtc(libnvrtc.nvrtcGetCUBIN(prog, binary))
````
- **EN**: This chunk continues `check_nvrtc` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `check_nvrtc`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 312-326 / 第 312-326 行
````python
    # Get mangled name
    c_mangled_name = ctypes.c_char_p()
    check_nvrtc(
        libnvrtc.nvrtcGetLoweredName(prog, c_kernel_name, ctypes.byref(c_mangled_name))
    )
    if c_mangled_name.value is not None:
        mangled_name = c_mangled_name.value.decode()  # make a copy
    else:
        mangled_name = ""

    libnvrtc.nvrtcDestroyProgram(ctypes.byref(prog))

    # For some reason, ".value" causes the string to be truncated,
    # likely due to the presence of '\0' in the string. So we use .raw instead.
    return binary.raw, mangled_name
````
- **EN**: This chunk continues `check_nvrtc` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `check_nvrtc`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 329-342 / 第 329-342 行
````python
class _CudaModule:
    def __init__(self, module: ctypes.c_void_p) -> None:
        self._module = module
        self._kernels: dict[str, _CudaKernel] = {}

    def __getattr__(self, name: str) -> "_CudaKernel":
        if name in self._kernels:
            return self._kernels[name]

        # Import the CUDA library inside the method
        # pyrefly: ignore [missing-module-attribute]
        from torch.cuda._utils import _get_gpu_runtime_library

        libcuda = _get_gpu_runtime_library()
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.cuda._utils. It introduces or extends `_CudaModule`, which hold the main object-oriented state for this portion of the file. This chunk defines `__getattr__`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.cuda._utils。 它引入或扩展了 `_CudaModule`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__getattr__`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 344-362 / 第 344-362 行
````python
        func = ctypes.c_void_p()
        try:
            _check_cuda(
                libcuda.cuModuleGetFunction(
                    ctypes.byref(func), self._module, name.encode("utf-8")
                )
            )
            kernel = _CudaKernel(func, self._module)
            self._kernels[name] = kernel
            return kernel

        except RuntimeError as err:
            raise AttributeError(f"No kernel named '{name}' in this module") from err


class _CudaKernel:
    """
    Represents a compiled CUDA kernel that can be called with PyTorch tensors.
    """
````
- **EN**: It introduces or extends `_CudaKernel`, which hold the main object-oriented state for this portion of the file. This chunk continues `_CudaKernel` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `_CudaKernel`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `_CudaKernel`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 364-378 / 第 364-378 行
````python
    def __init__(self, func: ctypes.c_void_p, module: ctypes.c_void_p) -> None:
        self.func = func
        self.module = module
        self._max_shared_mem_bytes = 0

    def __call__(
        self,
        grid: tuple[int, int, int] = (1, 1, 1),
        block: tuple[int, int, int] = (1, 1, 1),
        args: list | None = None,
        shared_mem: int = 0,
        stream: Any | None = None,
    ) -> None:
        """
        Call the compiled CUDA kernel
````
- **EN**: This chunk defines `__call__`, which implements a focused helper used by the surrounding module.
- **CN**: 这一段定义了 `__call__`，其作用是实现周边模块使用的关键辅助逻辑。

### Lines 380-397 / 第 380-397 行
````python
        Args:
            grid (tuple): Grid dimensions (grid_x, grid_y, grid_z)
            block (tuple): Block dimensions (block_x, block_y, block_z)
            args (list): List of arguments to pass to the kernel.
                         PyTorch tensor arguments will be automatically converted to pointers.
            shared_mem (int): Shared memory size in bytes
            stream (torch.cuda.Stream): CUDA stream to use. If None, uses current stream.
        """
        import torch

        libcuda = torch.cuda._utils._get_gpu_runtime_library()

        if not args:
            args = []

        # Process arguments and convert tensors to pointers
        processed_args: list[ctypes.c_void_p] = []
        c_args = []
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch. This chunk continues `__call__` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch。 这一段延续了 `__call__`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 399-418 / 第 399-418 行
````python
        for arg in args:
            if isinstance(arg, torch.Tensor):
                if not arg.is_cuda and not (arg.is_cpu and arg.is_pinned()):
                    raise ValueError(
                        "All tensor arguments must be CUDA tensors or pinned CPU tensors"
                    )
                # Get pointer to tensor data
                ptr = ctypes.c_void_p(arg.data_ptr())
                processed_args.append(ptr)
                c_args.append(ctypes.byref(ptr))
            elif isinstance(arg, int):
                # Convert integers to C int
                c_int = ctypes.c_int(arg)
                # Store the C int for reference keeping, not in processed_args
                c_args.append(ctypes.byref(c_int))
            elif isinstance(arg, float):
                # Python floats are doubles - use double by default
                c_double = ctypes.c_double(arg)
                # Store the C double for reference keeping, not in processed_args
                c_args.append(ctypes.byref(c_double))
````
- **EN**: This chunk continues `__call__` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `__call__`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 419-432 / 第 419-432 行
````python
            else:
                raise TypeError(f"Unsupported argument type: {type(arg)}")

        # Convert to array of void pointers
        c_args_array = (ctypes.c_void_p * len(c_args))()
        for i, arg in enumerate(c_args):
            c_args_array[i] = ctypes.cast(arg, ctypes.c_void_p)

        # Get the stream
        if stream is None:
            # Defer import to avoid circular imports
            import torch.cuda

            stream = torch.cuda.current_stream()
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.cuda. This chunk continues `__call__` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.cuda。 这一段延续了 `__call__`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 434-448 / 第 434-448 行
````python
        # Check if kernel requires large shared memory but hasn't been configured
        if shared_mem >= 48 * 1024 and (
            self._max_shared_mem_bytes == 0 or shared_mem > self._max_shared_mem_bytes
        ):
            configured_msg = (
                "not configured"
                if self._max_shared_mem_bytes == 0
                else f"only {self._max_shared_mem_bytes} bytes configured"
            )
            raise RuntimeError(
                f"Kernel requires {shared_mem} bytes of shared memory (>= 48KB), "
                f"but {configured_msg}. "
                "Call kernel.set_shared_memory_config(shared_mem) after compilation "
                "and before launching the kernel."
            )
````
- **EN**: This chunk continues `__call__` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `__call__`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 450-464 / 第 450-464 行
````python
        _check_cuda(
            libcuda.cuLaunchKernel(
                self.func,
                grid[0],
                grid[1],
                grid[2],
                block[0],
                block[1],
                block[2],
                shared_mem,
                stream._as_parameter_,
                c_args_array,
                None,
            )
        )
````
- **EN**: This chunk continues `__call__` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `__call__`，进一步展开其内部控制流或状态更新。

### Lines 466-485 / 第 466-485 行
````python
    def set_shared_memory_config(self, shared_mem_bytes: int) -> None:
        if shared_mem_bytes < 48 * 1024:
            # No configuration needed for <= 48KB, just update the value
            self._max_shared_mem_bytes = shared_mem_bytes
            return

        libcuda = _get_gpu_runtime_library()

        # Get device properties to validate against limits
        device_props = torch.cuda.get_device_properties()
        # HIP doesn't have shared_memory_per_block_optin in device properties, so we hard-code it here
        if torch.version.hip:
            # navi, CDNA1-CDNA3 allows a max of 64KB shared memory
            # CDNA4 allows a max of 160KB shared memory
            max_shared_mem = (
                65536 if device_props.gcnArchName != "gfx950" else 160 * 1024
            )
        else:
            max_shared_mem = getattr(
                device_props, "shared_memory_per_block_optin", 49152
````
- **EN**: This chunk defines `set_shared_memory_config`, which mutates configuration or backend state that affects later execution. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `set_shared_memory_config`，其作用是修改会影响后续执行的配置或后端状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 486-504 / 第 486-504 行
````python
            )

        if shared_mem_bytes > max_shared_mem:
            raise RuntimeError(
                f"Requested shared memory ({shared_mem_bytes} bytes) exceeds "
                f"device limit ({max_shared_mem} bytes). "
                "Consider reducing block size or shared memory usage."
            )

        # Set the function attribute once
        # https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__TYPES.html
        cudaFuncAttributeMaxDynamicSharedMemorySize = 8
        _check_cuda(
            libcuda.cuFuncSetAttribute(
                self.func,
                cudaFuncAttributeMaxDynamicSharedMemorySize,
                shared_mem_bytes,
            )
        )
````
- **EN**: This chunk continues `set_shared_memory_config` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `set_shared_memory_config`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 506-518 / 第 506-518 行
````python
        self._max_shared_mem_bytes = shared_mem_bytes


def _cuda_load_module(
    ptx: str | bytes, kernel_names: list[str] | None = None
) -> _CudaModule | dict[str, "_CudaKernel"]:
    """
    Loads a CUDA module from PTX code and returns a module object that can access kernels.

    Args:
        ptx (bytes or str): The PTX code to load
        kernel_names (list, optional): List of kernel names to extract from the module.
                                      If None, will return a module object with __getattr__.
````
- **EN**: This chunk defines `_cuda_load_module`, which serializes or reconstructs state across a Python-visible boundary. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_cuda_load_module`，其作用是在 Python 可见边界上序列化或重建状态。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 520-532 / 第 520-532 行
````python
    Returns:
        object: If kernel_names is None, returns a module object with __getattr__ to access kernels.
               If kernel_names is provided, returns a dict mapping kernel names to _CudaKernel objects.
    """
    # Ensure CUDA is initialized
    import torch.cuda

    # Load CUDA driver library
    libcuda = _get_gpu_runtime_library()

    # Convert PTX to bytes if it's a string
    if isinstance(ptx, str):
        ptx = ptx.encode("utf-8")
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.cuda. This chunk continues `_cuda_load_module` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.cuda。 这一段延续了 `_cuda_load_module`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 534-553 / 第 534-553 行
````python
    # Load PTX module
    module = ctypes.c_void_p()
    # Get the current stream without directly importing torch.cuda at module level
    stream = torch.cuda.current_stream()
    with stream:
        _check_cuda(libcuda.cuModuleLoadData(ctypes.byref(module), ptx))

    if not kernel_names:
        return _CudaModule(module)

    # Return specific kernels
    kernels = {}
    for name in kernel_names:
        func = ctypes.c_void_p()
        _check_cuda(
            libcuda.cuModuleGetFunction(
                ctypes.byref(func), module, name.encode("utf-8")
            )
        )
        kernels[name] = _CudaKernel(func, module)
````
- **EN**: This chunk continues `_cuda_load_module` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_cuda_load_module`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 554-568 / 第 554-568 行
````python
    return kernels


def _get_device_index(
    device: Any, optional: bool = False, allow_cpu: bool = False
) -> int:
    r"""Get the device index from :attr:`device`, which can be a torch.device object, a Python integer, or ``None``.

    If :attr:`device` is a torch.device object, returns the device index if it
    is a CUDA device. Note that for a CUDA device without a specified index,
    i.e., ``torch.device('cuda')``, this will return the current default CUDA
    device if :attr:`optional` is ``True``. If :attr:`allow_cpu` is ``True``,
    CPU devices will be accepted and ``-1`` will be returned in this case.

    If :attr:`device` is a Python integer, it is returned as is.
````
- **EN**: This chunk defines `_get_device_index`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_device_index`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 570-586 / 第 570-586 行
````python
    If :attr:`device` is ``None``, this will return the current default CUDA
    device if :attr:`optional` is ``True``.
    """
    if isinstance(device, int):
        return device
    if isinstance(device, str):
        device = torch.device(device)
    if isinstance(device, torch.device):
        if allow_cpu:
            if device.type not in ["cuda", "cpu"]:
                raise ValueError(f"Expected a cuda or cpu device, but got: {device}")
        elif device.type != "cuda":
            raise ValueError(f"Expected a cuda device, but got: {device}")
    if not torch.jit.is_scripting():
        if isinstance(device, torch.cuda.device):
            return device.idx
    return _torch_get_device_index(device, optional, allow_cpu)
````
- **EN**: This chunk continues `_get_device_index` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_get_device_index`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **CUDA runtime**
  - EN: Wraps streams, devices, memory state, and CUDA-specific coordination points.
  - CN: 封装流、设备、内存状态以及 CUDA 专用协同点。
- **_get_hip_runtime_library**
  - EN: `_get_hip_runtime_library` is one of the main symbols declared or implemented in this file.
  - CN: `_get_hip_runtime_library` 是本文件声明或实现的主要符号之一。
- **_get_cuda_library**
  - EN: `_get_cuda_library` is one of the main symbols declared or implemented in this file.
  - CN: `_get_cuda_library` 是本文件声明或实现的主要符号之一。
- **Import/export packaging**
  - EN: The code bundles Python/Torch resources into packages or reconstructs packaged content.
  - CN: 代码把 Python/Torch 资源打包成 package，或重建打包内容。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._utils`, `torch.utils.cpp_extension`, `torch.cuda`, `torch.cuda._utils`
- **Standard library / 标准库**: `ctypes`, `sys`, `typing`
- **Other helper packages / 其他辅助包**: `cuda.bindings`, `rocm_sdk`
- **Primary symbols in this file / 本文件核心符号**: `_get_hip_runtime_library`, `_get_cuda_library`, `_get_gpu_runtime_library`, `_check_cuda`, `_check_cuda_bindings`, `_get_hiprtc_library`, `_get_nvrtc_library`, `_get_gpu_rtc_library`, `_get_gpu_rtc_compatible_flags`, `_nvrtc_compile`
