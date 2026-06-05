# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cutlass/utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CUTLASSArgs`, and `CUTLASSCompileSourceCapturingContext`. It exposes functions such as `move_cutlass_compiled_cache`, `_rename_cutlass_import`, `try_import_cutlass`, `_normalize_xpu_arch`, `_normalize_cuda_arch`, `cutlass_arch`, and `...+9`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CUTLASSArgs`、`CUTLASSCompileSourceCapturingContext` 等类。同时提供 `move_cutlass_compiled_cache`、`_rename_cutlass_import`、`try_import_cutlass`、`_normalize_xpu_arch`、`_normalize_cuda_arch`、`cutlass_arch`、`另有9项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import atexit
import functools
import logging
import os
import shutil
import sys
import time
from dataclasses import dataclass
from pathlib import Path
from typing import Any
from typing_extensions import TypeIs

import sympy

import torch
from torch._inductor.runtime.runtime_utils import dynamo_timed
from torch._inductor.utils import clear_on_fresh_cache
from torch.utils._ordered_set import OrderedSet

````
- **EN**: Imports dependencies such as `atexit`, `functools`, `logging`, `os`, `shutil`, `sys`, and `...+10` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `atexit`、`functools`、`logging`、`os`、`shutil`、`sys`、`另有10项` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。

### Lines 21-40 / 第 21-40 行
````python
from ... import config
from ...ir import Layout
from ...runtime.runtime_utils import cache_dir
from ...virtualized import V
from ..cuda.cuda_env import get_cuda_arch, get_cuda_version
from ..xpu.xpu_env import get_xpu_arch, get_xpu_version


log = logging.getLogger(__name__)

CUTLASS_OPERATION_KIND: str = "gemm"
ACCUMULATOR_DTYPES: OrderedSet[torch.dtype] = OrderedSet([torch.float, torch.int32])
XW_DTYPES: OrderedSet[torch.dtype] = OrderedSet(
    [torch.half, torch.bfloat16, torch.float8_e4m3fn, torch.int8, torch.float8_e5m2]
)


@atexit.register
def move_cutlass_compiled_cache() -> None:
    """Move CUTLASS compiled cache file to the cache directory if it exists."""
````
- **EN**: Imports dependencies such as `...`, `...ir`, `...runtime.runtime_utils`, `...virtualized`, `..cuda.cuda_env`, and `..xpu.xpu_env` for the logic in this range. Introduces function `move_cutlass_compiled_cache`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `...`、`...ir`、`...runtime.runtime_utils`、`...virtualized`、`..cuda.cuda_env`、`..xpu.xpu_env` 等依赖，为后续逻辑提供基础能力。这里定义了函数`move_cutlass_compiled_cache`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 41-60 / 第 41-60 行
````python
    if try_import_cutlass.cache_info().currsize == 0:
        return

    try:
        import cutlass_cppgen  # type: ignore[import-not-found]
    except ImportError:
        return

    # Check if the CACHE_FILE attribute exists in cutlass_cppgen and if the file exists
    if not hasattr(cutlass_cppgen, "CACHE_FILE") or not os.path.exists(
        cutlass_cppgen.CACHE_FILE
    ):
        return

    try:
        filename = os.path.basename(cutlass_cppgen.CACHE_FILE)
        shutil.move(cutlass_cppgen.CACHE_FILE, os.path.join(cache_dir(), filename))
        log.debug("Moved CUTLASS compiled cache file to %s", cache_dir())
    except OSError:
        log.warning("Failed to move CUTLASS compiled cache file", exc_info=True)
````
- **EN**: Imports dependencies such as `cutlass_cppgen  # type: ignore[import-not-found]` for the logic in this range. Touches the filesystem to load, validate, or store compiler artifacts. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `cutlass_cppgen  # type: ignore[import-not-found]` 等依赖，为后续逻辑提供基础能力。这一段会访问文件系统，用于加载、校验或保存编译产物。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 61-80 / 第 61-80 行
````python


def _rename_cutlass_import(content: str, cutlass_modules: list[str]) -> str:
    for cutlass_module in cutlass_modules:
        content = content.replace(
            f"from {cutlass_module} import ",
            f"from cutlass_library.{cutlass_module} import ",
        )
    return content


@functools.cache
def try_import_cutlass() -> bool:
    """
    We want to support three ways of passing in CUTLASS:
    1. fbcode, handled by the internal build system.
    2. User specifies cutlass_dir. The default is ../third_party/cutlass/,
       which is the directory when developers build from source.
    """
    if config.is_fbcode():
````
- **EN**: Introduces function `_rename_cutlass_import`, function `try_import_cutlass`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_rename_cutlass_import`、函数`try_import_cutlass`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 81-100 / 第 81-100 行
````python
        try:
            import cutlass_cppgen  # type: ignore[import-not-found]
            import cutlass_library  # type: ignore[import-not-found]
        except ImportError as e:
            log.warning(
                "Failed to import CUTLASS packages in fbcode: %s, ignoring the CUTLASS backend.",
                e,
            )
            return False

        return True

    # Copy CUTLASS python scripts to a temp dir and add the temp dir to Python search path.
    # This is a temporary hack to avoid CUTLASS module naming conflicts.
    # TODO(ipiszy): remove this hack when CUTLASS solves Python scripts packaging structure issues.

    # TODO(mlazos): epilogue visitor tree currently lives in python/cutlass,
    # but will be moved to python/cutlass_library in the future (later 2025)
    def path_join(path0, path1):
        return os.path.abspath(os.path.join(path0, path1))
````
- **EN**: Imports dependencies such as `cutlass_cppgen  # type: ignore[import-not-found]`, and `cutlass_library  # type: ignore[import-not-found]` for the logic in this range. Introduces function `path_join`. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `cutlass_cppgen  # type: ignore[import-not-found]`、`cutlass_library  # type: ignore[import-not-found]` 等依赖，为后续逻辑提供基础能力。这里定义了函数`path_join`。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 101-120 / 第 101-120 行
````python

    # contains both cutlass and cutlass_library
    # we need cutlass for eVT
    cutlass_dir = (
        config.xpu.cutlass_dir
        if torch.xpu._is_compiled()
        else config.cutlass.cutlass_dir
    )
    cutlass_python_path = path_join(cutlass_dir, "python")
    torch_root = os.path.abspath(os.path.dirname(torch.__file__))
    mock_src_path = os.path.join(
        torch_root,
        "_inductor",
        "codegen",
        "cutlass",
        "lib_extensions",
        "cutlass_mock_imports",
    )

    cutlass_library_src_path = path_join(cutlass_python_path, "cutlass_library")
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `cutlass_dir`, `cutlass_python_path`, `torch_root`, `mock_src_path`, and `cutlass_library_src_path`. This range continues the implementation of function `try_import_cutlass`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。包含分支、循环或上下文管理等控制流。初始化或更新了 `cutlass_dir`、`cutlass_python_path`、`torch_root`、`mock_src_path`、`cutlass_library_src_path` 等值。这一段延续了函数`try_import_cutlass` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
    cutlass_cppgen_src_path = path_join(cutlass_python_path, "cutlass_cppgen")
    pycute_src_path = path_join(cutlass_python_path, "pycute")

    tmp_cutlass_full_path = os.path.abspath(os.path.join(cache_dir(), "torch_cutlass"))

    dst_link_library = path_join(tmp_cutlass_full_path, "cutlass_library")
    dst_link_cutlass_cppgen = path_join(tmp_cutlass_full_path, "cutlass_cppgen")
    dst_link_pycute = path_join(tmp_cutlass_full_path, "pycute")

    # mock modules to import cutlass
    mock_modules = ["cuda", "scipy", "pydot"]

    if os.path.isdir(cutlass_python_path):
        if tmp_cutlass_full_path not in sys.path:

            def link_and_append(dst_link, src_path, parent_dir):
                if os.path.lexists(dst_link):
                    assert os.path.islink(dst_link), (
                        f"{dst_link} is not a symlink. Try to remove {dst_link} manually and try again."
                    )
````
- **EN**: Introduces function `link_and_append`. Touches the filesystem to load, validate, or store compiler artifacts. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`link_and_append`。这一段会访问文件系统，用于加载、校验或保存编译产物。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 141-160 / 第 141-160 行
````python
                    assert os.path.realpath(os.readlink(dst_link)) == os.path.realpath(
                        src_path,
                    ), f"Symlink at {dst_link} does not point to {src_path}"
                else:
                    os.makedirs(parent_dir, exist_ok=True)
                    os.symlink(src_path, dst_link)

                if parent_dir not in sys.path:
                    sys.path.append(parent_dir)

            link_and_append(
                dst_link_library, cutlass_library_src_path, tmp_cutlass_full_path
            )
            link_and_append(
                dst_link_cutlass_cppgen, cutlass_cppgen_src_path, tmp_cutlass_full_path
            )
            link_and_append(dst_link_pycute, pycute_src_path, tmp_cutlass_full_path)

            for module in mock_modules:
                link_and_append(
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 161-180 / 第 161-180 行
````python
                    path_join(tmp_cutlass_full_path, module),  # dst_link
                    path_join(mock_src_path, module),  # src_path
                    tmp_cutlass_full_path,  # parent
                )

        try:
            import cutlass_cppgen  # type: ignore[import-not-found]  # noqa: F401
            import cutlass_library.generator
            import cutlass_library.library
            import cutlass_library.manifest  # noqa: F401
            import pycute  # type: ignore[import-not-found]  # noqa: F401

            return True
        except ImportError as e:
            log.debug(
                "Failed to import CUTLASS packages: %s, ignoring the CUTLASS backend.",
                e,
            )
    else:
        log.debug(
````
- **EN**: Imports dependencies such as `cutlass_cppgen  # type: ignore[import-not-found]  # noqa: F401`, `cutlass_library.generator`, `cutlass_library.library`, `cutlass_library.manifest  # noqa: F401`, and `pycute  # type: ignore[import-not-found]  # noqa: F401` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`, and `else`.
- **CN**: 这里导入了 `cutlass_cppgen  # type: ignore[import-not-found]  # noqa: F401`、`cutlass_library.generator`、`cutlass_library.library`、`cutlass_library.manifest  # noqa: F401`、`pycute  # type: ignore[import-not-found]  # noqa: F401` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `try`、`else` 等值。

### Lines 181-200 / 第 181-200 行
````python
            "Failed to import CUTLASS packages: CUTLASS repo does not exist: %s",
            cutlass_python_path,
        )
    return False


def _normalize_xpu_arch(arch: str) -> str:
    if arch.startswith("Xe"):
        return arch[2:]
    if 12 <= int(arch) and int(arch) <= 50:
        return arch
    else:
        raise NotImplementedError(f"Unsupported xpu arch: {arch}")


def _normalize_cuda_arch(arch: str) -> str:
    arch_num = arch
    if isinstance(arch, str):
        digits = "".join(ch for ch in arch if ch.isdigit())
        if not digits:
````
- **EN**: Introduces function `_normalize_xpu_arch`, function `_normalize_cuda_arch`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `arch_num`, and `digits`.
- **CN**: 这里定义了函数`_normalize_xpu_arch`、函数`_normalize_cuda_arch`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`arch_num`、`digits` 等值。

### Lines 201-220 / 第 201-220 行
````python
            raise ValueError(f"Unrecognized cuda arch: {arch}")
        arch_num = int(digits)
    else:
        arch_num = int(arch)

    if arch_num > 103:
        log.warning("Detected CUDA architecture > 103: %s. Please file an issue.", arch)
        return str(arch_num)
    if arch_num >= 103:
        return "103"
    if arch_num >= 100:
        return "100"
    if arch_num >= 90:
        return "90"
    if arch_num >= 80:
        return "80"
    if arch_num >= 75:
        return "75"
    if arch_num >= 70:
        return "70"
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `arch_num`, and `else`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `arch_num`、`else` 等值。

### Lines 221-240 / 第 221-240 行
````python
    raise NotImplementedError(f"Unsupported cuda arch: {arch}")


@functools.lru_cache(8)
def cutlass_arch(device_type: str) -> str:
    if device_type == "xpu":
        arch = get_xpu_arch()
        return _normalize_xpu_arch(arch)
    else:
        arch = get_cuda_arch()
        return _normalize_cuda_arch(arch)


@functools.lru_cache(1)
def toolkit_version(device_type: str) -> str:
    if device_type == "xpu":
        return get_xpu_version()
    else:
        return get_cuda_version()

````
- **EN**: Introduces function `cutlass_arch`, function `toolkit_version`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`cutlass_arch`、函数`toolkit_version`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 241-260 / 第 241-260 行
````python

@dataclass
class CUTLASSArgs:
    """
    CUTLASS args used to initialize a CUTLASS Manifest.
    """

    architectures: str | None = None
    toolkit_version: str | None = None
    instantiation_level: str | None = None
    operations: str | None = None

    build_dir = ""
    curr_build_dir = ""
    generator_target = ""
    kernels = "all"
    ignore_kernels = ""
    exclude_kernels = ""
    # TODO: these three look dead?
    kernel_filter_file: None = None
````
- **EN**: Introduces class `CUTLASSArgs`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Initializes or updates values such as `architectures`, `toolkit_version`, `instantiation_level`, `operations`, `build_dir`, `curr_build_dir`, and `...+5`.
- **CN**: 这里定义了类`CUTLASSArgs`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。初始化或更新了 `architectures`、`toolkit_version`、`instantiation_level`、`operations`、`build_dir`、`curr_build_dir`、`另有5项` 等值。

### Lines 261-280 / 第 261-280 行
````python
    selected_kernel_list: None = None
    interface_dir: None = None
    filter_by_cc = True
    disable_full_archs_compilation = False
    device_type: str = "cuda"

    def __post_init__(self):
        if self.architectures is None or self.toolkit_version is None:
            raise RuntimeError(
                f"{self.architectures=} or {self.toolkit_version=} is None!"
            )


@clear_on_fresh_cache
@functools.cache
def _gen_ops_cached(arch: str, version: str, device_type: str) -> dict[Any, Any]:
    # Note: Cache needs to be specific for cuda architecture and version

    # Import cutlass python scripts.
    assert try_import_cutlass()
````
- **EN**: Introduces function `__post_init__`, function `_gen_ops_cached`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`__post_init__`、函数`_gen_ops_cached`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 281-300 / 第 281-300 行
````python
    import cutlass_library.generator as cutlass_generator
    import cutlass_library.manifest as cutlass_manifest

    if arch is None or version is None:
        log.error(
            "Cannot detect cuda arch %s or version %s. "
            "Will discard all cutlass ops. "
            "Please consider setting _inductor.cuda.arch and _inductor.cuda.version configs.",
            arch,
            version,
        )
        return {}

    gen_arch = (
        "100" if arch == "103" else arch
    )  # CUTLASS SM103 generator only covers NVFB4; fallback to SM100 set
    instantiation_level: str = config.cutlass.cutlass_instantiation_level
    args = CUTLASSArgs(
        architectures=gen_arch,
        toolkit_version=version,
````
- **EN**: Imports dependencies such as `cutlass_library.generator`, and `cutlass_library.manifest` for the logic in this range. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `cutlass_library.generator`、`cutlass_library.manifest` 等依赖，为后续逻辑提供基础能力。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 301-320 / 第 301-320 行
````python
        instantiation_level=instantiation_level,
        operations=CUTLASS_OPERATION_KIND,
        device_type=device_type,
    )
    manifest = cutlass_manifest.Manifest(args)

    start_time = time.time()
    if device_type == "xpu":
        if hasattr(cutlass_generator, "GenerateIntelXe"):
            cutlass_generator.GenerateIntelXe(
                manifest, args.toolkit_version, arch=int(arch)
            )
        else:
            raise NotImplementedError(
                "Arch " + arch + " is not supported by current cutlass lib."
            )

    elif arch == "100":
        if hasattr(cutlass_generator, "GenerateSM100"):
            cutlass_generator.GenerateSM100(manifest, args.toolkit_version)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `instantiation_level`, `operations`, `device_type`, `manifest`, `start_time`, and `else`. This range continues the implementation of function `_gen_ops_cached`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `instantiation_level`、`operations`、`device_type`、`manifest`、`start_time`、`else` 等值。这一段延续了函数`_gen_ops_cached` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
        cutlass_generator.GenerateSM90(manifest, args.toolkit_version)
    else:
        try:
            func = getattr(cutlass_generator, "GenerateSM" + gen_arch)
            func(manifest, args.toolkit_version)
        except AttributeError as e:
            raise NotImplementedError(
                "Arch " + gen_arch + " is not supported by current cutlass lib."
            ) from e

    log.info(
        "CUTLASS library generated a dict of %d operation kinds in %.2f seconds",
        len(manifest.operations),
        time.time() - start_time,
    )
    return manifest.operations


def gen_ops(device_type: str) -> dict[Any, Any]:
    """
````
- **EN**: Introduces function `gen_ops`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `try`, and `func`.
- **CN**: 这里定义了函数`gen_ops`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`try`、`func` 等值。

### Lines 341-360 / 第 341-360 行
````python
    Generates all supported CUTLASS operations.
    """
    with dynamo_timed("cutlass_utils.gen_ops"):
        arch = cutlass_arch(device_type)
        version = toolkit_version(device_type)
        return _gen_ops_cached(arch, version, device_type)


from ..cpp_utils import DTYPE_TO_CPP


if torch.xpu._is_compiled():
    DTYPE_TO_CUTLASS_TYPE = {
        **DTYPE_TO_CPP,
        torch.float16: "uint16_t",
        torch.bfloat16: "uint16_t",
        torch.float8_e4m3fn: "uint8_t",
        torch.float8_e5m2: "uint8_t",
    }
else:
````
- **EN**: Imports dependencies such as `..cpp_utils` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `..cpp_utils` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 361-380 / 第 361-380 行
````python
    DTYPE_TO_CUTLASS_TYPE = {
        **DTYPE_TO_CPP,
        torch.float16: "__half",
        torch.bfloat16: "__nv_bfloat16",
        torch.float8_e4m3fn: "__nv_fp8_e4m3",
        torch.float8_e5m2: "__nv_fp8_e5m2",
    }


@functools.lru_cache(32)
def torch_dtype_to_cutlass_type(
    torch_dtype: torch.dtype,
) -> "cutlass_library.library.DataType":  # type: ignore[name-defined] # noqa: F821
    # Import cutlass python scripts.
    assert try_import_cutlass()
    import cutlass_library  # type: ignore[import]

    if torch_dtype == torch.float:
        return cutlass_library.library.DataType.f32
    elif torch_dtype == torch.half:
````
- **EN**: Imports dependencies such as `cutlass_library  # type: ignore[import]` for the logic in this range. Introduces function `torch_dtype_to_cutlass_type`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `cutlass_library  # type: ignore[import]` 等依赖，为后续逻辑提供基础能力。这里定义了函数`torch_dtype_to_cutlass_type`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 381-400 / 第 381-400 行
````python
        return cutlass_library.library.DataType.f16
    elif torch_dtype == torch.bfloat16:
        return cutlass_library.library.DataType.bf16
    elif torch_dtype == torch.float8_e4m3fn:
        return cutlass_library.library.DataType.e4m3
    elif torch_dtype == torch.float8_e5m2:
        return cutlass_library.library.DataType.e5m2
    else:
        raise NotImplementedError(f"Unsupported data type: {torch_dtype=}")


@functools.lru_cache(32)
def dtype_match(
    torch_dtype: torch.dtype | None,
    cutlass_dtype: "cutlass_library.library.DataType",  # type: ignore[name-defined]  # noqa: F821
) -> bool:
    # Import cutlass python scripts.
    assert try_import_cutlass()
    import cutlass_library

````
- **EN**: Imports dependencies such as `cutlass_library` for the logic in this range. Introduces function `dtype_match`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `cutlass_library` 等依赖，为后续逻辑提供基础能力。这里定义了函数`dtype_match`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 401-420 / 第 401-420 行
````python
    if torch_dtype == torch.float:
        return (
            cutlass_dtype == cutlass_library.library.DataType.f32
            or cutlass_dtype == cutlass_library.library.DataType.tf32
        )
    elif torch_dtype == torch.half:
        return cutlass_dtype == cutlass_library.library.DataType.f16
    elif torch_dtype == torch.bfloat16:
        return cutlass_dtype == cutlass_library.library.DataType.bf16
    elif torch_dtype == torch.int8:
        return cutlass_dtype == cutlass_library.library.DataType.s8
    elif torch_dtype == torch.uint8:
        return cutlass_dtype == cutlass_library.library.DataType.u8
    elif torch_dtype == torch.int32:
        return cutlass_dtype == cutlass_library.library.DataType.s32
    elif torch_dtype == torch.float8_e4m3fn:
        return cutlass_dtype == cutlass_library.library.DataType.e4m3
    elif torch_dtype == torch.float8_e5m2:
        return cutlass_dtype == cutlass_library.library.DataType.e5m2
    else:
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `cutlass_dtype`, and `else`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `cutlass_dtype`、`else` 等值。

### Lines 421-440 / 第 421-440 行
````python
        return False


def get_accumulator_dtype(
    input_torch_dtypes: list[torch.dtype],
) -> torch.dtype | None:
    """
    Given a pair of input torch dtypes, returns the inferred accumulator torch dtype.
    """

    assert OrderedSet(input_torch_dtypes) <= XW_DTYPES, (
        f"{input_torch_dtypes=} is not supported"
    )

    if len(input_torch_dtypes) != 2:
        return None

    if OrderedSet(input_torch_dtypes) == OrderedSet(
        [torch.float8_e5m2, torch.float8_e4m3fn]
    ):
````
- **EN**: Introduces function `get_accumulator_dtype`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_accumulator_dtype`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 441-460 / 第 441-460 行
````python
        return torch.float

    torch_dtype = None
    if input_torch_dtypes[0] == input_torch_dtypes[1]:
        torch_dtype = input_torch_dtypes[0]
    else:
        size0 = torch.tensor([], dtype=input_torch_dtypes[0]).element_size()
        size1 = torch.tensor([], dtype=input_torch_dtypes[1]).element_size()
        if size0 > size1:
            dtype0, dtype1 = input_torch_dtypes
        else:
            dtype1, dtype0 = input_torch_dtypes
        if dtype0 in [torch.half, torch.bfloat16] and dtype1 in [
            torch.int8,
            torch.uint8,
        ]:
            torch_dtype = dtype0

    if torch_dtype in (
        torch.float16,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `torch_dtype`, `else`, `size0`, and `size1`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `torch_dtype`、`else`、`size0`、`size1` 等值。

### Lines 461-480 / 第 461-480 行
````python
        torch.bfloat16,
        torch.float,
        torch.float8_e4m3fn,
        torch.float8_e5m2,
    ):
        accumulator_dtype = torch.float
    elif torch_dtype == torch.int8:
        accumulator_dtype = torch.int32
    else:
        raise NotImplementedError(f"Unsupported data types: {input_torch_dtypes=}")

    assert accumulator_dtype in ACCUMULATOR_DTYPES, (
        f"{accumulator_dtype=} is not supported"
    )
    return accumulator_dtype


@functools.lru_cache(32)
def get_alignments(torch_dtype: torch.dtype) -> list[int]:
    """
````
- **EN**: Introduces function `get_alignments`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`get_alignments`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 481-500 / 第 481-500 行
````python
    Returns all possible valid CUTLASS alignments in terms of the number of elements for a given dtype.
    CUTLASS gemm / conv SM80 APIs support 16 bytes max alignment, and 2 bytes min alignment.
    """

    if torch_dtype in (torch.half, torch.bfloat16):
        return [8, 4, 2, 1]
    elif torch_dtype == torch.float:
        return [4, 2, 1]
    elif torch_dtype in (
        torch.uint8,
        torch.int8,
        torch.float8_e4m3fn,
        torch.float8_e5m2,
    ):
        return [16, 8, 4, 2]
    elif torch_dtype == torch.int32:
        return [4, 2, 1]
    else:
        raise NotImplementedError(f"unsupported {torch_dtype=} for alignments")

````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 501-520 / 第 501-520 行
````python

def get_max_alignment(inductor_layout: Layout) -> int:
    """
    Returns the max alignment (in terms of number of elements) for a given Inductor Layout.
    """

    dtype = inductor_layout.dtype
    size = inductor_layout.size
    offset = inductor_layout.offset

    def is_static_int(number: object) -> TypeIs[int | sympy.Integer]:
        return isinstance(number, (int | sympy.Integer))

    def a_factor_of(x, alignment):
        if is_static_int(x) and is_static_int(alignment):
            return x % alignment == 0
        rem = sympy.Mod(x, alignment)
        return V.graph.sizevars.evaluate_expr(sympy.Eq(rem, 0))

    try:
````
- **EN**: Introduces function `get_max_alignment`, function `is_static_int`, function `a_factor_of`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_max_alignment`、函数`is_static_int`、函数`a_factor_of`。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 521-540 / 第 521-540 行
````python
        contiguous_dim = inductor_layout.stride.index(1)
    except ValueError:
        # No dim with stride 1 found, return 1
        return 1
    alignments = get_alignments(dtype)
    for alignment in alignments:
        if not a_factor_of(size[contiguous_dim], alignment) or not a_factor_of(
            offset, alignment
        ):
            continue
        if all(
            (dim == contiguous_dim)
            or a_factor_of(inductor_layout.stride[dim], alignment)
            for dim in range(len(size))
        ):
            return alignment
    return 1


class CUTLASSCompileSourceCapturingContext:
````
- **EN**: Introduces class `CUTLASSCompileSourceCapturingContext`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`CUTLASSCompileSourceCapturingContext`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 541-560 / 第 541-560 行
````python
    # Helper class for Benchmarking and Testing CUTLASS Kernels in isolation.
    # Can be used to capture the sourcecode passed to CUDACodeCache.compile

    def __init__(self, device_type: str):
        self.sources = []
        self._compile_patch = None
        self.device_type = device_type

    def __enter__(self, *args, **kwargs):
        import unittest.mock as mock

        import torch._inductor.codecache

        codecache_cls = (
            torch._inductor.codecache.XPUCodeCache
            if self.device_type == "xpu"
            else torch._inductor.codecache.CUDACodeCache
        )
        _compile_method_orig = codecache_cls.compile

````
- **EN**: Imports dependencies such as `unittest.mock`, and `torch._inductor.codecache` for the logic in this range. Introduces function `__init__`, function `__enter__`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `unittest.mock`、`torch._inductor.codecache` 等依赖，为后续逻辑提供基础能力。这里定义了函数`__init__`、函数`__enter__`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。

### Lines 561-580 / 第 561-580 行
````python
        def my_compile(source_code, dst_file_ext, extra_args: list[str] | None = None):
            self.sources.append(source_code)
            return _compile_method_orig(source_code, dst_file_ext)

        # pyrefly: ignore [bad-assignment]
        self._compile_patch = mock.patch(
            f"torch._inductor.codecache.{codecache_cls.__name__}.compile", my_compile
        )
        self._compile_patch.__enter__(*args, **kwargs)  # type: ignore[union-attr]
        return self

    def __exit__(self, *args, **kwargs):
        self._compile_patch.__exit__(*args, **kwargs)  # type: ignore[union-attr]


def cutlass_standalone_runner_compile_command(
    device_type: str, srcpath: Path, exepath: Path
):
    # returns command string to compile a (captured) CUDA GEMM Kernel source to a standalone executable that's ready to run
    # Passes the correct preprocessor define to nvcc to ensure the standalone runner is enabled.
````
- **EN**: Introduces function `my_compile`, function `__exit__`, function `cutlass_standalone_runner_compile_command`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`my_compile`、函数`__exit__`、函数`cutlass_standalone_runner_compile_command`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 581-593 / 第 581-593 行
````python

    extra_args = ["-DGENERATE_STANDALONE_RUNNER=1"]
    if device_type != "xpu":
        extra_args.append("-DCUTLASS_DEBUG_TRACE_LEVEL=1")
    cutlass_compile_command = (
        torch._inductor.codegen.xpu.compile_utils.xpu_compile_command
        if device_type == "xpu"
        else torch._inductor.codegen.cuda.compile_utils.cuda_compile_command
    )
    compile_command = cutlass_compile_command(
        [str(srcpath)], str(exepath), "exe", extra_args=extra_args
    )
    return compile_command
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `extra_args`, `cutlass_compile_command`, and `compile_command`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `extra_args`、`cutlass_compile_command`、`compile_command` 等值。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `atexit`, `functools`, `logging`, `os`, `shutil`, `sys`, `time`, `dataclasses`, `pathlib`, `typing`, `unittest.mock`
- **Third-party / 第三方**: `typing_extensions`, `sympy`, `cutlass_library.generator`, `cutlass_library.manifest`, `cutlass_library`, `cutlass_cppgen`, `cutlass_library.library`, `pycute`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.runtime.runtime_utils`, `torch._inductor.utils`, `torch.utils._ordered_set`, `...`, `...ir`, `...runtime.runtime_utils`, `...virtualized`, `..cuda.cuda_env`, `..xpu.xpu_env`, `..cpp_utils`, `torch._inductor.codecache`
