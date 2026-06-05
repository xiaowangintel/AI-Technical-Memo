# package.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/package/package.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module packages compiled artifacts and related metadata. It exposes functions such as `compile_so`, `package_aoti`, and `load_package`.
- **用途（中文）**: 该模块打包已编译产物及其相关元数据。同时提供 `compile_so`、`package_aoti`、`load_package` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
import io
import json
import logging
import os
import tempfile
from typing import IO

import torch
from torch._inductor import config
from torch._inductor.cpp_builder import BuildOptionsBase, CppBuilder
from torch.export.pt2_archive._package import (
    AOTI_FILES,
    AOTICompiledModel,
    load_pt2,
````
- **EN**: Imports dependencies such as `io`, `json`, `logging`, `os`, `tempfile`, `typing`, and `...+4` for the logic in this range.
- **CN**: 这里导入了 `io`、`json`、`logging`、`os`、`tempfile`、`typing`、`另有4项` 等依赖，为后续逻辑提供基础能力。

### Lines 15-28 / 第 15-28 行
````python
    package_pt2,
)
from torch.types import FileLike


log = logging.getLogger(__name__)


def compile_so(aoti_dir: str, aoti_files: list[str], so_path: str) -> str:
    def get_aoti_file_with_suffix(suffix: str) -> str:
        for file in aoti_files:
            if file.endswith(suffix):
                return file
        raise RuntimeError(f"Unable to find file with suffix {suffix}")
````
- **EN**: Imports dependencies such as `torch.types` for the logic in this range. Introduces function `compile_so`, function `get_aoti_file_with_suffix`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch.types` 等依赖，为后续逻辑提供基础能力。这里定义了函数`compile_so`、函数`get_aoti_file_with_suffix`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 29-42 / 第 29-42 行
````python

    # Compile all the files into a .so
    cpp_file = os.path.join(aoti_dir, get_aoti_file_with_suffix(".cpp"))
    consts_o = os.path.join(aoti_dir, get_aoti_file_with_suffix(".o"))

    file_name = os.path.splitext(cpp_file)[0]

    # Parse compile flags and build the .o file
    with open(file_name + "_compile_flags.json") as f:
        compile_flags = json.load(f)

    compile_options = BuildOptionsBase(
        **compile_flags, use_relative_path=config.is_fbcode()
    )
````
- **EN**: Serializes or deserializes JSON data as part of persistence or interchange. Touches the filesystem to load, validate, or store compiler artifacts. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `cpp_file`, `consts_o`, `file_name`, `compile_flags`, and `compile_options`.
- **CN**: 通过 JSON 的序列化或反序列化来完成持久化或数据交换。这一段会访问文件系统，用于加载、校验或保存编译产物。包含分支、循环或上下文管理等控制流。初始化或更新了 `cpp_file`、`consts_o`、`file_name`、`compile_flags`、`compile_options` 等值。

### Lines 43-56 / 第 43-56 行
````python
    object_builder = CppBuilder(
        name=file_name,
        sources=cpp_file,
        BuildOption=compile_options,
    )
    output_o = object_builder.get_target_file_path()
    object_builder.build()

    # Parse linker flags and build the .so file
    with open(file_name + "_linker_flags.json") as f:
        linker_flags = json.load(f)

    linker_options = BuildOptionsBase(
        **linker_flags, use_relative_path=config.is_fbcode()
````
- **EN**: Serializes or deserializes JSON data as part of persistence or interchange. Touches the filesystem to load, validate, or store compiler artifacts. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `object_builder`, `name`, `sources`, `BuildOption`, `output_o`, `linker_flags`, and `...+1`.
- **CN**: 通过 JSON 的序列化或反序列化来完成持久化或数据交换。这一段会访问文件系统，用于加载、校验或保存编译产物。包含分支、循环或上下文管理等控制流。初始化或更新了 `object_builder`、`name`、`sources`、`BuildOption`、`output_o`、`linker_flags`、`另有1项` 等值。

### Lines 57-70 / 第 57-70 行
````python
    )
    so_builder = CppBuilder(
        name=os.path.split(so_path)[-1],
        sources=[output_o, consts_o],
        BuildOption=linker_options,
        output_dir=so_path,
    )
    output_so = so_builder.get_target_file_path()
    so_builder.build()

    # mmapped weights
    serialized_weights_filename = file_name + "_serialized_weights.bin"
    if serialized_weights_filename in aoti_files:
        with open(serialized_weights_filename, "rb") as f_weights:
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `so_builder`, `name`, `sources`, `BuildOption`, `output_dir`, `output_so`, and `...+1`. This range continues the implementation of function `compile_so`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。包含分支、循环或上下文管理等控制流。初始化或更新了 `so_builder`、`name`、`sources`、`BuildOption`、`output_dir`、`output_so`、`另有1项` 等值。这一段延续了函数`compile_so` 的具体实现。

### Lines 71-84 / 第 71-84 行
````python
            serialized_weights = f_weights.read()

        with open(output_so, "a+b") as f_so:
            so_size = f_so.tell()
            # Page align the weights
            f_so.write(b" " * (16384 - so_size % 16384))
            f_so.write(serialized_weights)

    return output_so


def package_aoti(
    archive_file: FileLike,
    aoti_files: AOTI_FILES,
````
- **EN**: Introduces function `package_aoti`. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`package_aoti`。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 85-98 / 第 85-98 行
````python
) -> FileLike:
    """
    Saves the AOTInductor generated files to the PT2Archive format.

    Args:
        archive_file: The file name to save the package to.
        aoti_files: This can either be a singular path to a directory containing
        the AOTInductor files, or a dictionary mapping the model name to the
        path to its AOTInductor generated files.
    """

    return package_pt2(
        archive_file,
        aoti_files=aoti_files,
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `Args`, `archive_file`, and `aoti_files`. This range continues the implementation of function `package_aoti`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `Args`、`archive_file`、`aoti_files` 等值。这一段延续了函数`package_aoti` 的具体实现。

### Lines 99-112 / 第 99-112 行
````python
    )


def load_package(
    path: FileLike,
    model_name: str = "model",
    run_single_threaded: bool = False,
    num_runners: int = 1,
    device_index: int = -1,
) -> AOTICompiledModel:
    try:
        pt2_contents = load_pt2(
            path,
            run_single_threaded=run_single_threaded,
````
- **EN**: Introduces function `load_package`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `path`, `model_name`, `run_single_threaded`, `num_runners`, `device_index`, `try`, and `...+1`.
- **CN**: 这里定义了函数`load_package`。包含分支、循环或上下文管理等控制流。初始化或更新了 `path`、`model_name`、`run_single_threaded`、`num_runners`、`device_index`、`try`、`另有1项` 等值。

### Lines 113-126 / 第 113-126 行
````python
            num_runners=num_runners,
            device_index=device_index,
        )
        if model_name not in pt2_contents.aoti_runners:
            raise RuntimeError(f"Model {model_name} not found in package")
        return pt2_contents.aoti_runners[model_name]
    except RuntimeError:
        log.warning("Loading outdated pt2 file. Please regenerate your package.")

    if isinstance(path, (io.IOBase, IO)):
        with tempfile.NamedTemporaryFile(suffix=".pt2") as f:
            # TODO(angelayi): We shouldn't need to do this -- miniz should
            # handle reading the buffer. This is just a temporary workaround
            path.seek(0)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `num_runners`, and `device_index`. This range continues the implementation of function `load_package`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `num_runners`、`device_index` 等值。这一段延续了函数`load_package` 的具体实现。

### Lines 127-138 / 第 127-138 行
````python
            f.write(path.read())
            log.debug("Writing buffer to tmp file located at %s.", f.name)
            loader = torch._C._aoti.AOTIModelPackageLoader(
                f.name, model_name, run_single_threaded, num_runners, device_index
            )
            return AOTICompiledModel(loader)

    path = os.fspath(path)  # AOTIModelPackageLoader expects (str, str)
    loader = torch._C._aoti.AOTIModelPackageLoader(
        path, model_name, run_single_threaded, num_runners, device_index
    )
    return AOTICompiledModel(loader)
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `loader`, and `path`. This range continues the implementation of function `load_package`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `loader`、`path` 等值。这一段延续了函数`load_package` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Packages compiled artifacts and related metadata  
  **CN**: 打包已编译产物及其相关元数据
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Primary functions: `compile_so`, `package_aoti`, and `load_package`  
  **CN**: 主要函数：`compile_so`、`package_aoti`、`load_package`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `io`, `json`, `logging`, `os`, `tempfile`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor`, `torch._inductor.cpp_builder`, `torch.export.pt2_archive._package`, `torch.types`
