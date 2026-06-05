# compiler.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/compiler.py`
- **EN:** Defines classes `CompilationOptions`, `ArtifactManager` and functions `compile_with_nvcc`, `convertToBinaryData`, `CDLLBin` for the cutlass_cppgen package.
- **CN:** 为 cutlass_cppgen 包定义类 `CompilationOptions`, `ArtifactManager`和函数 `compile_with_nvcc`, `convertToBinaryData`, `CDLLBin`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32 — File header

```python
#################################################################################################
#
# Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: BSD-3-Clause
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation
# and/or other materials provided with the distribution.
#
# 3. Neither the name of the copyright holder nor the names of its
# contributors may be used to endorse or promote products derived from
# this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
#################################################################################################
```
**EN:** Contains the file header, license notice, and opening comments for the module.
**CN:** 包含文件头、许可证声明以及模块开头的注释。

### Line 33 — Import `ctypes`

```python
import ctypes
```
**EN:** Imports `ctypes` so later code can use these APIs at module scope.
**CN:** 导入 `ctypes`，供后续代码在模块级使用这些 API。

### Line 34 — Import `json`

```python
import json
```
**EN:** Imports `json` so later code can use these APIs at module scope.
**CN:** 导入 `json`，供后续代码在模块级使用这些 API。

### Line 35 — Import `pathlib`

```python
import pathlib
```
**EN:** Imports `pathlib` so later code can use these APIs at module scope.
**CN:** 导入 `pathlib`，供后续代码在模块级使用这些 API。

### Line 36 — Import `os`

```python
import os
```
**EN:** Imports `os` so later code can use these APIs at module scope.
**CN:** 导入 `os`，供后续代码在模块级使用这些 API。

### Line 37 — Import `sqlite3`

```python
import sqlite3
```
**EN:** Imports `sqlite3` so later code can use these APIs at module scope.
**CN:** 导入 `sqlite3`，供后续代码在模块级使用这些 API。

### Line 38 — Import `subprocess`

```python
import subprocess
```
**EN:** Imports `subprocess` so later code can use these APIs at module scope.
**CN:** 导入 `subprocess`，供后续代码在模块级使用这些 API。

### Line 39 — Import `tempfile`

```python
import tempfile
```
**EN:** Imports `tempfile` so later code can use these APIs at module scope.
**CN:** 导入 `tempfile`，供后续代码在模块级使用这些 API。

### Line 40 — From `functools` import

```python
from functools import lru_cache
```
**EN:** Imports `lru_cache` from `functools` to reuse shared definitions at module scope.
**CN:** 从 `functools` 导入 `lru_cache`，以便后续代码在模块级复用共享定义。

### Line 42 — From `cutlass_cppgen.utils.lazy_import` import

```python
from cutlass_cppgen.utils.lazy_import import lazy_import
```
**EN:** Imports `lazy_import` from `cutlass_cppgen.utils.lazy_import` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.utils.lazy_import` 导入 `lazy_import`，以便后续代码在模块级复用共享定义。

### Line 43 — Assign `cuda`

```python
cuda = lazy_import("cuda.cuda")
```
**EN:** Assigns `cuda` from `lazy_import('cuda.cuda')`, establishing state at module scope.
**CN:** 将 `cuda` 赋值为 `lazy_import('cuda.cuda')`，用于在模块级建立状态。

### Line 44 — Assign `cudart`

```python
cudart = lazy_import("cuda.cudart")
```
**EN:** Assigns `cudart` from `lazy_import('cuda.cudart')`, establishing state at module scope.
**CN:** 将 `cudart` 赋值为 `lazy_import('cuda.cudart')`，用于在模块级建立状态。

### Line 45 — Assign `nvrtc`

```python
nvrtc = lazy_import("cuda.nvrtc")
```
**EN:** Assigns `nvrtc` from `lazy_import('cuda.nvrtc')`, establishing state at module scope.
**CN:** 将 `nvrtc` 赋值为 `lazy_import('cuda.nvrtc')`，用于在模块级建立状态。

### Line 46 — Assign `dpctl`

```python
dpctl = lazy_import("dpctl")
```
**EN:** Assigns `dpctl` from `lazy_import('dpctl')`, establishing state at module scope.
**CN:** 将 `dpctl` 赋值为 `lazy_import('dpctl')`，用于在模块级建立状态。

### Line 47 — From `cutlass_library` import

```python
from cutlass_library import SubstituteTemplate
```
**EN:** Imports `SubstituteTemplate` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `SubstituteTemplate`，以便后续代码在模块级复用共享定义。

### Line 50 — Import `cutlass_cppgen`

```python
import cutlass_cppgen
```
**EN:** Imports `cutlass_cppgen` so later code can use these APIs at module scope.
**CN:** 导入 `cutlass_cppgen`，供后续代码在模块级使用这些 API。

### Line 51 — From `cutlass_cppgen` import

```python
from cutlass_cppgen import CACHE_FILE, CUTLASS_PATH, cuda_install_path, logger
```
**EN:** Imports `CACHE_FILE, CUTLASS_PATH, cuda_install_path, logger` from `cutlass_cppgen` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen` 导入 `CACHE_FILE, CUTLASS_PATH, cuda_install_path, logger`，以便后续代码在模块级复用共享定义。

### Line 52 — From `cutlass_cppgen.backend.gemm_operation` import

```python
from cutlass_cppgen.backend.gemm_operation import GemmOperationUniversal
```
**EN:** Imports `GemmOperationUniversal` from `cutlass_cppgen.backend.gemm_operation` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.gemm_operation` 导入 `GemmOperationUniversal`，以便后续代码在模块级复用共享定义。

### Line 53 — From `cutlass_cppgen.backend.library` import

```python
from cutlass_cppgen.backend.library import ApiVersion
```
**EN:** Imports `ApiVersion` from `cutlass_cppgen.backend.library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.library` 导入 `ApiVersion`，以便后续代码在模块级复用共享定义。

### Line 54 — From `cutlass_cppgen.backend.utils.device` import

```python
from cutlass_cppgen.backend.utils.device import device_cc
```
**EN:** Imports `device_cc` from `cutlass_cppgen.backend.utils.device` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.utils.device` 导入 `device_cc`，以便后续代码在模块级复用共享定义。

### Line 55 — From `cutlass_library.arch_constants` import

```python
from cutlass_library.arch_constants import ( INTEL_XE12, INTEL_XE20)
```
**EN:** Imports `INTEL_XE12, INTEL_XE20` from `cutlass_library.arch_constants` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library.arch_constants` 导入 `INTEL_XE12, INTEL_XE20`，以便后续代码在模块级复用共享定义。

### Lines 57-58 — Assign `IncludeTemplate`

```python
IncludeTemplate = r"""#include "${include}"
"""
```
**EN:** Assigns `IncludeTemplate` from `'#include "${include}"\n'`, establishing state at module scope.
**CN:** 将 `IncludeTemplate` 赋值为 `'#include "${include}"\n'`，用于在模块级建立状态。

### Line 61 — Function `compile_with_nvcc`

```python
def compile_with_nvcc(cmd, source, error_file):
```
**EN:** Defines function `compile_with_nvcc` with parameters `cmd, source, error_file`.
**CN:** 定义函数 `compile_with_nvcc`，参数为 `cmd, source, error_file`。

#### Line 62 — Assign `succeed`

```python
    succeed = True
```
**EN:** Assigns `succeed` from `True`, establishing state in function `compile_with_nvcc`.
**CN:** 将 `succeed` 赋值为 `True`，用于在函数 `compile_with_nvcc` 中建立状态。

#### Lines 63-73 — Exception handling

```python
    try:
        subprocess.check_output(cmd, stderr=subprocess.STDOUT)
    except subprocess.CalledProcessError as e:
        error_message = e.output.decode()
        with open(error_file, "w") as error_out:
            error_log = "Compilation error for the following kernel: \n"
            error_log += source
            error_log += "\nError Message:\n"
            error_log += error_message
            error_out.write(error_log)
        succeed = False
```
**EN:** Wraps the enclosed logic with exception handling and optional cleanup paths.
**CN:** 使用异常处理包装该逻辑，并提供回退或清理路径。

#### Lines 74-78 — Conditional `not succeed`

```python
    if not succeed:
        # Print the error log to stdout if log level is set to warning or higher
        # verbosity. Otherwise, simply point to the error log file.
        logger.warning(error_log)
        raise Exception(f"Invalid Kernel. See '{error_file}' for details.")
```
**EN:** Checks `not succeed` and selects the matching branch in function `compile_with_nvcc`.
**CN:** 检查 `not succeed`，并在函数 `compile_with_nvcc` 中选择匹配的分支。

### Lines 81-84 — Class `CompilationOptions`

```python
class CompilationOptions:
    """
    Compilation options.
    """
```
**EN:** Declares class `CompilationOptions` deriving from `object`. Purpose: Compilation options.
**CN:** 声明类 `CompilationOptions`，其基类为 `object`。 其用途：Compilation options.

#### Line 86 — Function `__init__`

```python
    def __init__(self, flags, arch, include_paths=[], for_sycl=False, device_flag=None):
```
**EN:** Defines function `__init__` with parameters `self, flags, arch, include_paths, for_sycl, device_flag`.
**CN:** 定义函数 `__init__`，参数为 `self, flags, arch, include_paths, for_sycl, device_flag`。

##### Line 87 — Assign `self.includes`

```python
        self.includes = []
```
**EN:** Assigns `self.includes` from `[]`, establishing state in function `__init__`.
**CN:** 将 `self.includes` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 88 — Assign `self.include_paths`

```python
        self.include_paths = include_paths
```
**EN:** Assigns `self.include_paths` from `include_paths`, establishing state in function `__init__`.
**CN:** 将 `self.include_paths` 赋值为 `include_paths`，用于在函数 `__init__` 中建立状态。

##### Line 89 — Assign `self.flags`

```python
        self.flags = flags
```
**EN:** Assigns `self.flags` from `flags`, establishing state in function `__init__`.
**CN:** 将 `self.flags` 赋值为 `flags`，用于在函数 `__init__` 中建立状态。

##### Line 90 — Assign `self.arch`

```python
        self.arch = arch
```
**EN:** Assigns `self.arch` from `arch`, establishing state in function `__init__`.
**CN:** 将 `self.arch` 赋值为 `arch`，用于在函数 `__init__` 中建立状态。

##### Line 91 — Assign `self.for_sycl`

```python
        self.for_sycl = for_sycl
```
**EN:** Assigns `self.for_sycl` from `for_sycl`, establishing state in function `__init__`.
**CN:** 将 `self.for_sycl` 赋值为 `for_sycl`，用于在函数 `__init__` 中建立状态。

##### Line 92 — Assign `self.device_flag`

```python
        self.device_flag = device_flag
```
**EN:** Assigns `self.device_flag` from `device_flag`, establishing state in function `__init__`.
**CN:** 将 `self.device_flag` 赋值为 `device_flag`，用于在函数 `__init__` 中建立状态。

#### Line 94 — Function `_encode`

```python
    def _encode(self):
```
**EN:** Defines function `_encode` with parameters `self`.
**CN:** 定义函数 `_encode`，参数为 `self`。

##### Line 95 — Assign `opts`

```python
        opts = []
```
**EN:** Assigns `opts` from `[]`, establishing state in function `_encode`.
**CN:** 将 `opts` 赋值为 `[]`，用于在函数 `_encode` 中建立状态。

##### Lines 96-97 — Loop over `self.flags`

```python
        for flag in self.flags:
            opts.append(flag)
```
**EN:** Iterates `flag` over `self.flags` to repeat a processing step.
**CN:** 让 `flag` 遍历 `self.flags`，从而重复执行处理步骤。

##### Lines 99-103 — Loop over `self.include_paths`

```python
        for incl in self.include_paths:
            if self.for_sycl:
                opts.append(f"-I{incl}")
            else:
                opts.append(f"--include-path={incl}")
```
**EN:** Iterates `incl` over `self.include_paths` to repeat a processing step.
**CN:** 让 `incl` 遍历 `self.include_paths`，从而重复执行处理步骤。

##### Lines 105-110 — Conditional `self.for_sycl`

```python
        if self.for_sycl:
            arch_flag = f"-fsycl-targets={self.arch}"
        else:
            arch_flag = f"-arch=sm_{self.arch}"
            if self.arch in [90, 100, 101, 103, 120, 121] and int(cutlass_cppgen.nvcc_version().split('.')[0]) >= 12:
               arch_flag += "a"
```
**EN:** Checks `self.for_sycl` and selects the matching branch in function `_encode`.
**CN:** 检查 `self.for_sycl`，并在函数 `_encode` 中选择匹配的分支。

##### Line 112 — Call `opts.append`

```python
        opts.append(arch_flag)
```
**EN:** Calls `opts.append` for side effects or initialization work in function `_encode`.
**CN:** 调用 `opts.append` 执行副作用或初始化逻辑；该语句位于在函数 `_encode` 中。

##### Lines 113-116 — Conditional `self.device_flag`

```python
        if self.device_flag:
            if self.for_sycl:
                opts.append("-Xs")
            opts.append(self.device_flag)
```
**EN:** Checks `self.device_flag` and selects the matching branch in function `_encode`.
**CN:** 检查 `self.device_flag`，并在函数 `_encode` 中选择匹配的分支。

##### Line 118 — Return

```python
        return opts
```
**EN:** Returns `opts` to the caller.
**CN:** 向调用方返回 `opts`。

#### Line 120 — Function `get_str`

```python
    def get_str(self):
```
**EN:** Defines function `get_str` with parameters `self`.
**CN:** 定义函数 `get_str`，参数为 `self`。

##### Line 121 — Assign `opts`

```python
        opts = self._encode()
```
**EN:** Assigns `opts` from `self._encode()`, establishing state in function `get_str`.
**CN:** 将 `opts` 赋值为 `self._encode()`，用于在函数 `get_str` 中建立状态。

##### Line 123 — Return

```python
        return " ".join(opts)
```
**EN:** Returns `' '.join(opts)` to the caller.
**CN:** 向调用方返回 `' '.join(opts)`。

#### Line 125 — Function `get`

```python
    def get(self):
```
**EN:** Defines function `get` with parameters `self`.
**CN:** 定义函数 `get`，参数为 `self`。

##### Line 126 — Assign `options`

```python
        options = self._encode()
```
**EN:** Assigns `options` from `self._encode()`, establishing state in function `get`.
**CN:** 将 `options` 赋值为 `self._encode()`，用于在函数 `get` 中建立状态。

##### Line 127 — Return

```python
        return [bytes(str.encode(s)) for s in options]
```
**EN:** Returns `[bytes(str.encode(s)) for s in options]` to the caller.
**CN:** 向调用方返回 `[bytes(str.encode(s)) for s in options]`。

### Line 130 — Function `convertToBinaryData`

```python
def convertToBinaryData(filename):
```
**EN:** Defines function `convertToBinaryData` with parameters `filename`.
**CN:** 定义函数 `convertToBinaryData`，参数为 `filename`。

#### Lines 131-132 — Context manager

```python
    with open(filename, "rb") as file:
        blobData = file.read()
```
**EN:** Uses context manager(s) `open(filename, 'rb')` to manage resources safely.
**CN:** 使用上下文管理器 `open(filename, 'rb')` 安全地管理资源。

#### Line 133 — Return

```python
    return blobData
```
**EN:** Returns `blobData` to the caller.
**CN:** 向调用方返回 `blobData`。

### Line 136 — Function `CDLLBin`

```python
def CDLLBin(host_binary):
```
**EN:** Defines function `CDLLBin` with parameters `host_binary`.
**CN:** 定义函数 `CDLLBin`，参数为 `host_binary`。

#### Line 137 — Assign `tempfile.tempdir`

```python
    tempfile.tempdir = "./"
```
**EN:** Assigns `tempfile.tempdir` from `'./'`, establishing state in function `CDLLBin`.
**CN:** 将 `tempfile.tempdir` 赋值为 `'./'`，用于在函数 `CDLLBin` 中建立状态。

#### Lines 138-139 — Assign `temp_so`

```python
    temp_so = tempfile.NamedTemporaryFile(
        prefix="host_func", suffix=".so", delete=True)
```
**EN:** Assigns `temp_so` from `tempfile.NamedTemporaryFile(prefix='host_func', suffix='.so', delete=True)`, establishing state in function `CDLLBin`.
**CN:** 将 `temp_so` 赋值为 `tempfile.NamedTemporaryFile(prefix='host_func', suffix='.so', delete=True)`，用于在函数 `CDLLBin` 中建立状态。

#### Lines 140-141 — Context manager

```python
    with open(temp_so.name, "wb") as file:
        file.write(host_binary)
```
**EN:** Uses context manager(s) `open(temp_so.name, 'wb')` to manage resources safely.
**CN:** 使用上下文管理器 `open(temp_so.name, 'wb')` 安全地管理资源。

#### Line 142 — Assign `host_lib`

```python
    host_lib = ctypes.CDLL(temp_so.name)
```
**EN:** Assigns `host_lib` from `ctypes.CDLL(temp_so.name)`, establishing state in function `CDLLBin`.
**CN:** 将 `host_lib` 赋值为 `ctypes.CDLL(temp_so.name)`，用于在函数 `CDLLBin` 中建立状态。

#### Line 143 — Return

```python
    return host_lib
```
**EN:** Returns `host_lib` to the caller.
**CN:** 向调用方返回 `host_lib`。

### Lines 146-149 — Class `ArtifactManager`

```python
class ArtifactManager:
    """
    Artifact manager
    """
```
**EN:** Declares class `ArtifactManager` deriving from `object`. Purpose: Artifact manager
**CN:** 声明类 `ArtifactManager`，其基类为 `object`。 其用途：Artifact manager

#### Line 151 — Function `__init__`

```python
    def __init__(self) -> None:
```
**EN:** Defines function `__init__` with parameters `self`.
**CN:** 定义函数 `__init__`，参数为 `self`。

##### Line 152 — Assign `connection`

```python
        connection = sqlite3.connect(CACHE_FILE)
```
**EN:** Assigns `connection` from `sqlite3.connect(CACHE_FILE)`, establishing state in function `__init__`.
**CN:** 将 `connection` 赋值为 `sqlite3.connect(CACHE_FILE)`，用于在函数 `__init__` 中建立状态。

##### Line 153 — Assign `cursor`

```python
        cursor = connection.cursor()
```
**EN:** Assigns `cursor` from `connection.cursor()`, establishing state in function `__init__`.
**CN:** 将 `cursor` 赋值为 `connection.cursor()`，用于在函数 `__init__` 中建立状态。

##### Line 154 — Comment or spacing block

```python
        # Create the table if it does not already exist
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 155-161 — Assign `sqlite_create_table_query`

```python
        sqlite_create_table_query = """
        CREATE TABLE IF NOT EXISTS compiled_operations(op_key TEXT NOT NULL UNIQUE,
                                                        cubin BLOB NOT NULL,
                                                        hostbin BLOB NOT NULL,
                                                        op_name TEXT NOT NULL,
                                                        op_attrs TEXT NOT NULL)
        """
```
**EN:** Assigns `sqlite_create_table_query` from `'\n CREATE TABLE IF NOT EXISTS compiled_operations(op_key TEXT NOT NULL UNIQUE,\n cubin BLOB NOT NULL,\n hostbin BLOB NOT NULL,\n op_name TEXT NOT NULL,\n op...`, establishing state in function `__init__`.
**CN:** 将 `sqlite_create_table_query` 赋值为 `'\n CREATE TABLE IF NOT EXISTS compiled_operations(op_key TEXT NOT NULL UNIQUE,\n cubin BLOB NOT NULL,\n hostbin BLOB NOT NULL,\n op_name TEXT NOT NULL,\n op...`，用于在函数 `__init__` 中建立状态。

##### Line 162 — Call `cursor.execute`

```python
        cursor.execute(sqlite_create_table_query)
```
**EN:** Calls `cursor.execute` for side effects or initialization work in function `__init__`.
**CN:** 调用 `cursor.execute` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 163 — Call `connection.commit`

```python
        connection.commit()
```
**EN:** Calls `connection.commit` for side effects or initialization work in function `__init__`.
**CN:** 调用 `connection.commit` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 164 — Call `cursor.close`

```python
        cursor.close()
```
**EN:** Calls `cursor.close` for side effects or initialization work in function `__init__`.
**CN:** 调用 `cursor.close` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 166 — Assign `self._nvrtc_compile_options`

```python
        self._nvrtc_compile_options = ["-std=c++17", "-default-device"]
```
**EN:** Assigns `self._nvrtc_compile_options` from `['-std=c++17', '-default-device']`, establishing state in function `__init__`.
**CN:** 将 `self._nvrtc_compile_options` 赋值为 `['-std=c++17', '-default-device']`，用于在函数 `__init__` 中建立状态。

##### Lines 167-171 — Assign `self._nvcc_compile_options`

```python
        self._nvcc_compile_options = [
            "-std=c++17",
            "--expt-relaxed-constexpr",
            "-Xcudafe --diag_suppress=esa_on_defaulted_function_ignored",
        ]
```
**EN:** Assigns `self._nvcc_compile_options` from `['-std=c++17', '--expt-relaxed-constexpr', '-Xcudafe --diag_suppress=esa_on_defaulted_function_ignored']`, establishing state in function `__init__`.
**CN:** 将 `self._nvcc_compile_options` 赋值为 `['-std=c++17', '--expt-relaxed-constexpr', '-Xcudafe --diag_suppress=esa_on_defaulted_function_ignored']`，用于在函数 `__init__` 中建立状态。

##### Line 172 — Comment or spacing block

```python
        # TODO(Codeplay): remove CUTLASS_SYCL_BUILTIN_ENABLE when the spirv functions are available for PVC
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 173-182 — Assign `self._dpcpp_compile_options`

```python
        self._dpcpp_compile_options = ["-fsycl", "-std=c++17",
                                       "-DCUTLASS_ENABLE_SYCL",
                                       "-fsycl-rtc-mode",
                                       "-DSYCL_INTEL_TARGET",
                                       "-DCUTLASS_SYCL_BUILTIN_ENABLE",
                                       "-shared", "-fPIC",
                                       "-fno-sycl-dead-args-optimization",
                                       "-Xspirv-translator", "-spirv-ext=+SPV_INTEL_split_barrier",
                                       "-fno-sycl-instrument-device-code",
                                       "-fsycl-range-rounding=disable"]
```
**EN:** Assigns `self._dpcpp_compile_options` from `['-fsycl', '-std=c++17', '-DCUTLASS_ENABLE_SYCL', '-fsycl-rtc-mode', '-DSYCL_INTEL_TARGET', '-DCUTLASS_SYCL_BUILTIN_ENABLE', '-shared', '-fPIC', '-fno-sycl-d...`, establishing state in function `__init__`.
**CN:** 将 `self._dpcpp_compile_options` 赋值为 `['-fsycl', '-std=c++17', '-DCUTLASS_ENABLE_SYCL', '-fsycl-rtc-mode', '-DSYCL_INTEL_TARGET', '-DCUTLASS_SYCL_BUILTIN_ENABLE', '-shared', '-fPIC', '-fno-sycl-d...`，用于在函数 `__init__` 中建立状态。

##### Lines 183-193 — Exception handling

```python
        try:
            cc = device_cc()
            if cc in [INTEL_XE12, INTEL_XE20]:  
                self.dpcpp()
            else:
                self.nvcc()
        except (ImportError, AttributeError, RuntimeError):
            if os.environ.get('CUTLASS_USE_SYCL', '0') == '1':
                self.dpcpp()
            else:
                self.nvcc()
```
**EN:** Wraps the enclosed logic with exception handling and optional cleanup paths.
**CN:** 使用异常处理包装该逻辑，并提供回退或清理路径。

##### Line 194 — Assign `self.compiled_cache_device`

```python
        self.compiled_cache_device = {}
```
**EN:** Assigns `self.compiled_cache_device` from `{}`, establishing state in function `__init__`.
**CN:** 将 `self.compiled_cache_device` 赋值为 `{}`，用于在函数 `__init__` 中建立状态。

##### Line 195 — Assign `self.compiled_cache_host`

```python
        self.compiled_cache_host = {}
```
**EN:** Assigns `self.compiled_cache_host` from `{}`, establishing state in function `__init__`.
**CN:** 将 `self.compiled_cache_host` 赋值为 `{}`，用于在函数 `__init__` 中建立状态。

#### Line 197 — Function `nvrtc`

```python
    def nvrtc(self):
```
**EN:** Defines function `nvrtc` with parameters `self`.
**CN:** 定义函数 `nvrtc`，参数为 `self`。

##### Line 198 — Assign `self.backend`

```python
        self.backend = "nvrtc"
```
**EN:** Assigns `self.backend` from `'nvrtc'`, establishing state in function `nvrtc`.
**CN:** 将 `self.backend` 赋值为 `'nvrtc'`，用于在函数 `nvrtc` 中建立状态。

##### Line 199 — Assign `self.default_compile_options`

```python
        self.default_compile_options = self._nvrtc_compile_options
```
**EN:** Assigns `self.default_compile_options` from `self._nvrtc_compile_options`, establishing state in function `nvrtc`.
**CN:** 将 `self.default_compile_options` 赋值为 `self._nvrtc_compile_options`，用于在函数 `nvrtc` 中建立状态。

#### Line 201 — Function `nvcc`

```python
    def nvcc(self):
```
**EN:** Defines function `nvcc` with parameters `self`.
**CN:** 定义函数 `nvcc`，参数为 `self`。

##### Line 202 — Assign `self.backend`

```python
        self.backend = "nvcc"
```
**EN:** Assigns `self.backend` from `'nvcc'`, establishing state in function `nvcc`.
**CN:** 将 `self.backend` 赋值为 `'nvcc'`，用于在函数 `nvcc` 中建立状态。

##### Line 203 — Assign `self.default_compile_options`

```python
        self.default_compile_options = self._nvcc_compile_options
```
**EN:** Assigns `self.default_compile_options` from `self._nvcc_compile_options`, establishing state in function `nvcc`.
**CN:** 将 `self.default_compile_options` 赋值为 `self._nvcc_compile_options`，用于在函数 `nvcc` 中建立状态。

#### Line 205 — Function `dpcpp`

```python
    def dpcpp(self):
```
**EN:** Defines function `dpcpp` with parameters `self`.
**CN:** 定义函数 `dpcpp`，参数为 `self`。

##### Line 206 — Assign `self.backend`

```python
        self.backend = "dpcpp"
```
**EN:** Assigns `self.backend` from `'dpcpp'`, establishing state in function `dpcpp`.
**CN:** 将 `self.backend` 赋值为 `'dpcpp'`，用于在函数 `dpcpp` 中建立状态。

##### Line 207 — Assign `self.default_compile_options`

```python
        self.default_compile_options = self._dpcpp_compile_options
```
**EN:** Assigns `self.default_compile_options` from `self._dpcpp_compile_options`, establishing state in function `dpcpp`.
**CN:** 将 `self.default_compile_options` 赋值为 `self._dpcpp_compile_options`，用于在函数 `dpcpp` 中建立状态。

#### Line 209 — Function `_is_sycl`

```python
    def _is_sycl(self):
```
**EN:** Defines function `_is_sycl` with parameters `self`.
**CN:** 定义函数 `_is_sycl`，参数为 `self`。

##### Line 210 — Return

```python
        return self.backend == "dpcpp"
```
**EN:** Returns `self.backend == 'dpcpp'` to the caller.
**CN:** 向调用方返回 `self.backend == 'dpcpp'`。

#### Line 212 — Function `insert_operation`

```python
    def insert_operation(self, op_key, cubin, hostfile, op_name, op_attrs):
```
**EN:** Defines function `insert_operation` with parameters `self, op_key, cubin, hostfile, op_name, op_attrs`.
**CN:** 定义函数 `insert_operation`，参数为 `self, op_key, cubin, hostfile, op_name, op_attrs`。

##### Line 213 — Assign `connection`

```python
        connection = sqlite3.connect(CACHE_FILE)
```
**EN:** Assigns `connection` from `sqlite3.connect(CACHE_FILE)`, establishing state in function `insert_operation`.
**CN:** 将 `connection` 赋值为 `sqlite3.connect(CACHE_FILE)`，用于在函数 `insert_operation` 中建立状态。

##### Line 214 — Assign `cursor`

```python
        cursor = connection.cursor()
```
**EN:** Assigns `cursor` from `connection.cursor()`, establishing state in function `insert_operation`.
**CN:** 将 `cursor` 赋值为 `connection.cursor()`，用于在函数 `insert_operation` 中建立状态。

##### Line 215 — Assign `sqlite_insert_blob_query`

```python
        sqlite_insert_blob_query = """ INSERT OR IGNORE INTO compiled_operations (op_key, cubin, hostbin, op_name, op_attrs) VALUES (?, ?, ?, ?, ?)"""
```
**EN:** Assigns `sqlite_insert_blob_query` from `' INSERT OR IGNORE INTO compiled_operations (op_key, cubin, hostbin, op_name, op_attrs) VALUES (?, ?, ?, ?, ?)'`, establishing state in function `insert_operation`.
**CN:** 将 `sqlite_insert_blob_query` 赋值为 `' INSERT OR IGNORE INTO compiled_operations (op_key, cubin, hostbin, op_name, op_attrs) VALUES (?, ?, ?, ?, ?)'`，用于在函数 `insert_operation` 中建立状态。

##### Line 217 — Assign `hostbin`

```python
        hostbin = convertToBinaryData(hostfile)
```
**EN:** Assigns `hostbin` from `convertToBinaryData(hostfile)`, establishing state in function `insert_operation`.
**CN:** 将 `hostbin` 赋值为 `convertToBinaryData(hostfile)`，用于在函数 `insert_operation` 中建立状态。

##### Line 219 — Assign `data_tuple`

```python
        data_tuple = (op_key, cubin, hostbin, op_name, json.dumps(op_attrs))
```
**EN:** Assigns `data_tuple` from `(op_key, cubin, hostbin, op_name, json.dumps(op_attrs))`, establishing state in function `insert_operation`.
**CN:** 将 `data_tuple` 赋值为 `(op_key, cubin, hostbin, op_name, json.dumps(op_attrs))`，用于在函数 `insert_operation` 中建立状态。

##### Line 221 — Call `cursor.execute`

```python
        cursor.execute(sqlite_insert_blob_query, data_tuple)
```
**EN:** Calls `cursor.execute` for side effects or initialization work in function `insert_operation`.
**CN:** 调用 `cursor.execute` 执行副作用或初始化逻辑；该语句位于在函数 `insert_operation` 中。

##### Line 222 — Call `connection.commit`

```python
        connection.commit()
```
**EN:** Calls `connection.commit` for side effects or initialization work in function `insert_operation`.
**CN:** 调用 `connection.commit` 执行副作用或初始化逻辑；该语句位于在函数 `insert_operation` 中。

##### Line 223 — Call `cursor.close`

```python
        cursor.close()
```
**EN:** Calls `cursor.close` for side effects or initialization work in function `insert_operation`.
**CN:** 调用 `cursor.close` 执行副作用或初始化逻辑；该语句位于在函数 `insert_operation` 中。

#### Line 225 — Function `load_operation`

```python
    def load_operation(self, op_key, extra_funcs):
```
**EN:** Defines function `load_operation` with parameters `self, op_key, extra_funcs`.
**CN:** 定义函数 `load_operation`，参数为 `self, op_key, extra_funcs`。

##### Line 226 — Assign `connection`

```python
        connection = sqlite3.connect(CACHE_FILE)
```
**EN:** Assigns `connection` from `sqlite3.connect(CACHE_FILE)`, establishing state in function `load_operation`.
**CN:** 将 `connection` 赋值为 `sqlite3.connect(CACHE_FILE)`，用于在函数 `load_operation` 中建立状态。

##### Line 227 — Assign `cursor`

```python
        cursor = connection.cursor()
```
**EN:** Assigns `cursor` from `connection.cursor()`, establishing state in function `load_operation`.
**CN:** 将 `cursor` 赋值为 `connection.cursor()`，用于在函数 `load_operation` 中建立状态。

##### Line 228 — Assign `sqlite_fetch_blob_query`

```python
        sqlite_fetch_blob_query = """SELECT * from compiled_operations where op_key = ?"""
```
**EN:** Assigns `sqlite_fetch_blob_query` from `'SELECT * from compiled_operations where op_key = ?'`, establishing state in function `load_operation`.
**CN:** 将 `sqlite_fetch_blob_query` 赋值为 `'SELECT * from compiled_operations where op_key = ?'`，用于在函数 `load_operation` 中建立状态。

##### Line 229 — Call `cursor.execute`

```python
        cursor.execute(sqlite_fetch_blob_query, (op_key,))
```
**EN:** Calls `cursor.execute` for side effects or initialization work in function `load_operation`.
**CN:** 调用 `cursor.execute` 执行副作用或初始化逻辑；该语句位于在函数 `load_operation` 中。

##### Line 230 — Assign `record`

```python
        record = cursor.fetchall()
```
**EN:** Assigns `record` from `cursor.fetchall()`, establishing state in function `load_operation`.
**CN:** 将 `record` 赋值为 `cursor.fetchall()`，用于在函数 `load_operation` 中建立状态。

##### Lines 231-232 — Conditional `len(record) == 0`

```python
        if len(record) == 0:
            return False
```
**EN:** Checks `len(record) == 0` and selects the matching branch in function `load_operation`.
**CN:** 检查 `len(record) == 0`，并在函数 `load_operation` 中选择匹配的分支。

##### Lines 233-273 — Loop over `record`

```python
        for row in record:
            key, cubin_image, host_binary, operation_name, op_attr = row
            op_attr = json.loads(op_attr)
            if self._is_sycl():
                q = dpctl.SyclQueue(cutlass_cppgen.sycl_device())
                module = dpctl.program.create_program_from_spirv(
                    q, cubin_image)
                kernel = module.get_sycl_kernel(f"__sycl_kernel_{operation_name}")
            else:
                err, module = cuda.cuModuleLoadData(cubin_image)
                if err != cuda.CUresult.CUDA_SUCCESS:
                    raise RuntimeError("Cuda Error: {}".format(err))
                err, kernel = cuda.cuModuleGetFunction(
                    module, bytes(str.encode(operation_name)))

            self.compiled_cache_device[key] = kernel

            compiled_host_fns = {}
            host_lib = CDLLBin(host_binary)

            func_name = operation_name + "_get_params"
            func = getattr(host_lib, func_name)
            func.restype = ctypes.POINTER(ctypes.c_char * op_attr[0])
            compiled_host_fns["get_args"] = func

            func_name = operation_name + "_shared_memory_size"
            func = getattr(host_lib, func_name)
            compiled_host_fns["shared_memory_capacity"] = func()

            for attr in op_attr:
                if isinstance(attr, str):
                    func_name = operation_name + "_" + attr
                    func = getattr(host_lib, func_name)

                    # Set the return type of the function
                    if attr in extra_funcs and extra_funcs[attr] != None:
                        func.restype = extra_funcs[attr]

                    compiled_host_fns[attr] = func

            self.compiled_cache_host[key] = compiled_host_fns
```
**EN:** Iterates `row` over `record` to repeat a processing step.
**CN:** 让 `row` 遍历 `record`，从而重复执行处理步骤。

##### Line 274 — Return

```python
        return True
```
**EN:** Returns `True` to the caller.
**CN:** 向调用方返回 `True`。

#### Lines 276-279 — Function `emit_compile_`

```python
    def emit_compile_(self, operation_list, compilation_options, host_compilation_options):
        """
        Compile a list of kernels and store them into database
        """
```
**EN:** Defines function `emit_compile_` with parameters `self, operation_list, compilation_options, host_compilation_options`. Purpose: Compile a list of kernels and store them into database
**CN:** 定义函数 `emit_compile_`，参数为 `self, operation_list, compilation_options, host_compilation_options`。 其用途：Compile a list of kernels and store them into database

##### Line 280 — Assign `source_buffer_device`

```python
        source_buffer_device = ""
```
**EN:** Assigns `source_buffer_device` from `''`, establishing state in function `emit_compile_`.
**CN:** 将 `source_buffer_device` 赋值为 `''`，用于在函数 `emit_compile_` 中建立状态。

##### Line 281 — Assign `source_buffer_host`

```python
        source_buffer_host = ""
```
**EN:** Assigns `source_buffer_host` from `''`, establishing state in function `emit_compile_`.
**CN:** 将 `source_buffer_host` 赋值为 `''`，用于在函数 `emit_compile_` 中建立状态。

##### Line 282 — Comment or spacing block

```python
        # 1. include
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 283 — Assign `includes`

```python
        includes = []
```
**EN:** Assigns `includes` from `[]`, establishing state in function `emit_compile_`.
**CN:** 将 `includes` 赋值为 `[]`，用于在函数 `emit_compile_` 中建立状态。

##### Lines 284-287 — Loop over `operation_list`

```python
        for operation in operation_list:
            for incl in operation.emitter.includes:
                if incl not in includes:
                    includes.append(incl)
```
**EN:** Iterates `operation` over `operation_list` to repeat a processing step.
**CN:** 让 `operation` 遍历 `operation_list`，从而重复执行处理步骤。

##### Line 289 — Assign `includes_host`

```python
        includes_host = includes
```
**EN:** Assigns `includes_host` from `includes`, establishing state in function `emit_compile_`.
**CN:** 将 `includes_host` 赋值为 `includes`，用于在函数 `emit_compile_` 中建立状态。

##### Lines 290-294 — Conditional `self._is_sycl()`

```python
        if self._is_sycl():
            includes_host.extend(["stddef.h"])
        else:
            includes_host.extend(
                ["builtin_types.h", "device_launch_parameters.h", "cstddef"])
```
**EN:** Checks `self._is_sycl()` and selects the matching branch in function `emit_compile_`.
**CN:** 检查 `self._is_sycl()`，并在函数 `emit_compile_` 中选择匹配的分支。

##### Lines 296-300 — Loop over `includes`

```python
        for incl in includes:
            source_buffer_device += SubstituteTemplate(
                IncludeTemplate,
                {"include": incl},
            )
```
**EN:** Iterates `incl` over `includes` to repeat a processing step.
**CN:** 让 `incl` 遍历 `includes`，从而重复执行处理步骤。

##### Lines 302-306 — Loop over `includes_host`

```python
        for incl in includes_host:
            source_buffer_host += SubstituteTemplate(
                IncludeTemplate,
                {"include": incl},
            )
```
**EN:** Iterates `incl` over `includes_host` to repeat a processing step.
**CN:** 让 `incl` 遍历 `includes_host`，从而重复执行处理步骤。

##### Line 308 — Comment or spacing block

```python
        # 2. Operations
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 309-321 — Loop over `operation_list`

```python
        for operation in operation_list:
            source_buffer_device += operation.emit()
            source_buffer_host += operation.emit()
            values = {
                "operation_name": operation.name(),
                "operation_suffix": operation.emitter.operation_suffix,
            }
            source_buffer_device += SubstituteTemplate(
                operation.KernelTemplate,
                values,
            )
            source_buffer_host += SubstituteTemplate(
                operation.HostTemplate, values)
```
**EN:** Iterates `operation` over `operation_list` to repeat a processing step.
**CN:** 让 `operation` 遍历 `operation_list`，从而重复执行处理步骤。

##### Line 323 — Comment or spacing block

```python
        # 3. compile
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 324-432 — Conditional `self.backend == 'nvrtc'`

```python
        if self.backend == "nvrtc":
            err, program = nvrtc.nvrtcCreateProgram(
                str.encode(source_buffer_device),
                bytes(str.encode("module.cu")),
                0, [], [])

            if err != nvrtc.nvrtcResult.NVRTC_SUCCESS:
                raise RuntimeError("NVRTC Error: {}".format(err))

            # Compile program
            options = compilation_options.get()

            err, = nvrtc.nvrtcCompileProgram(program, len(options), options)
            if err != nvrtc.nvrtcResult.NVRTC_SUCCESS:
                error_string = "NVRTC Error: {}\n".format(err)

                # Get log from compilation
                err, logSize = nvrtc.nvrtcGetProgramLogSize(program)
                if err != nvrtc.nvrtcResult.NVRTC_SUCCESS:
                    raise RuntimeError("NVRTC Error: {}".format(err))

                log = b" " * logSize
                err, = nvrtc.nvrtcGetProgramLog(program, log)
                if err != nvrtc.nvrtcResult.NVRTC_SUCCESS:
                    raise RuntimeError("NVRTC Error: {}".format(err))

                raise RuntimeError(
                    error_string + log.decode() + source_buffer_device)

            # Get data from compilation
            err, dataSize = nvrtc.nvrtcGetCUBINSize(program)
            if err != nvrtc.nvrtcResult.NVRTC_SUCCESS:
                raise RuntimeError("NVRTC Error: {}".format(err))

            cubin_image = b" " * dataSize
            (err,) = nvrtc.nvrtcGetCUBIN(program, cubin_image)
            if err != nvrtc.nvrtcResult.NVRTC_SUCCESS:
                raise RuntimeError("NVRTC Error: {}".format(err))

        elif self.backend == "dpcpp":
            # Emit code to file
            tempfile.tempdir = "./"
            with (
                tempfile.NamedTemporaryFile(
                    prefix="kernel_", suffix=".cpp", delete=True) as temp_cpp,
                tempfile.TemporaryDirectory(
                    prefix="kernel_", suffix="_dpcpp") as temp_dump_dir,
                tempfile.NamedTemporaryFile(
                    prefix="kernel_", suffix=".o", delete=True) as ignore_out
            ):
                with open(temp_cpp.name, "w") as file:
                    file.write(source_buffer_device)

                # Compile with DPC++
                cmd = ["icpx"]
                cmd.extend(compilation_options._encode())
                cmd.extend([temp_cpp.name, "-o", ignore_out.name,
                            f"-fsycl-dump-device-code={temp_dump_dir}"])
                compile_with_nvcc(cmd, source_buffer_device,
                                  "./cutlass_python_compilation_device_error.txt")

                # Find SPIR-V device code in temporary directory
                spv_files = list(pathlib.Path(
                    temp_dump_dir).glob("*.spv"))

                # When specifying a specific subgroup size, DPC++ currently
                # generates multiple SPIR-V files. We create a program from each of
                # them to find the one containing the kernel with the correct
                # subgroup size.
                q = dpctl.SyclQueue(cutlass_cppgen.sycl_device())
                op_name = f"__sycl_kernel_{operation_list[0].name()}"
                cubin_image = None
                for f in spv_files:
                    with open(f, "rb") as spirv_file:
                        spirv_image = spirv_file.read()
                        program = dpctl.program.create_program_from_spirv(
                            q, spirv_image)
                        if not program.has_sycl_kernel(op_name):
                            continue
                        spirv_kernel = program.get_sycl_kernel(op_name)
                        if spirv_kernel.max_sub_group_size == 16:
                            cubin_image = spirv_image
                            break

        else:  # with nvcc backend
            # emit code
            tempfile.tempdir = "./"
            temp_cu = tempfile.NamedTemporaryFile(
                prefix="kernel", suffix=".cu", delete=True)
            temp_cubin = tempfile.NamedTemporaryFile(
                prefix="kernel", suffix=".cubin", delete=True)
            with open(temp_cu.name, "w") as file:
                file.write(source_buffer_device)

            # compile with nvcc
            cmd_template = "${cuda_install_path}/bin/nvcc ${options} -cubin ${srcfile} -o ${tarfile}"
            values = {
                "cuda_install_path": cuda_install_path(),
                "options": compilation_options.get_str(),
                "srcfile": temp_cu.name,
                "tarfile": temp_cubin.name,
            }
            cmd = SubstituteTemplate(cmd_template, values)
            compile_with_nvcc(cmd.split(" "), source_buffer_device,
                              "./cutlass_python_compilation_device_error.txt")

            # load the cubin image
            with open(temp_cubin.name, "rb") as file:
                cubin_image = file.read()
```
**EN:** Checks `self.backend == 'nvrtc'` and selects the matching branch in function `emit_compile_`.
**CN:** 检查 `self.backend == 'nvrtc'`，并在函数 `emit_compile_` 中选择匹配的分支。

##### Line 434 — Assign `tempfile.tempdir`

```python
        tempfile.tempdir = "./"
```
**EN:** Assigns `tempfile.tempdir` from `'./'`, establishing state in function `emit_compile_`.
**CN:** 将 `tempfile.tempdir` 赋值为 `'./'`，用于在函数 `emit_compile_` 中建立状态。

##### Line 435 — Assign `host_suffix`

```python
        host_suffix = ".cpp" if self._is_sycl() else ".cu"
```
**EN:** Assigns `host_suffix` from `'.cpp' if self._is_sycl() else '.cu'`, establishing state in function `emit_compile_`.
**CN:** 将 `host_suffix` 赋值为 `'.cpp' if self._is_sycl() else '.cu'`，用于在函数 `emit_compile_` 中建立状态。

##### Lines 436-437 — Assign `temp_src`

```python
        temp_src = tempfile.NamedTemporaryFile(
            prefix="host_src", suffix=host_suffix, delete=True)
```
**EN:** Assigns `temp_src` from `tempfile.NamedTemporaryFile(prefix='host_src', suffix=host_suffix, delete=True)`, establishing state in function `emit_compile_`.
**CN:** 将 `temp_src` 赋值为 `tempfile.NamedTemporaryFile(prefix='host_src', suffix=host_suffix, delete=True)`，用于在函数 `emit_compile_` 中建立状态。

##### Line 439 — Comment or spacing block

```python
        # Write the host source
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 440-441 — Context manager

```python
        with open(temp_src.name, "w") as outfile:
            outfile.write(source_buffer_host)
```
**EN:** Uses context manager(s) `open(temp_src.name, 'w')` to manage resources safely.
**CN:** 使用上下文管理器 `open(temp_src.name, 'w')` 安全地管理资源。

##### Lines 443-444 — Assign `temp_dst`

```python
        temp_dst = tempfile.NamedTemporaryFile(
            prefix="host_func", suffix=".so", delete=True)
```
**EN:** Assigns `temp_dst` from `tempfile.NamedTemporaryFile(prefix='host_func', suffix='.so', delete=True)`, establishing state in function `emit_compile_`.
**CN:** 将 `temp_dst` 赋值为 `tempfile.NamedTemporaryFile(prefix='host_func', suffix='.so', delete=True)`，用于在函数 `emit_compile_` 中建立状态。

##### Line 446 — Comment or spacing block

```python
        # Set up host compilation arguments
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 447 — Assign `cmd`

```python
        cmd = []
```
**EN:** Assigns `cmd` from `[]`, establishing state in function `emit_compile_`.
**CN:** 将 `cmd` 赋值为 `[]`，用于在函数 `emit_compile_` 中建立状态。

##### Lines 448-460 — Conditional `not self._is_sycl()`

```python
        if not self._is_sycl():
            cmd.append(f"{cuda_install_path()}/bin/nvcc")
            cmd.extend(["-x", "cu", "-Xcompiler=-fpermissive",
                        "-Xcompiler=-w", "-Xcompiler=-fPIC"])
            cmd.extend(host_compilation_options.get_str().split(" "))
            cmd.extend(["-shared", "-o", temp_dst.name,
                        temp_src.name, "-lcudart", "-lcuda"])
        else:
            cmd.append("icpx")
            # Clang does not support "-fpermissive"
            cmd.extend(["-fsycl", "-w", "-fPIC"])
            cmd.extend(host_compilation_options._encode())
            cmd.extend(["-shared", "-o", temp_dst.name, temp_src.name])
```
**EN:** Checks `not self._is_sycl()` and selects the matching branch in function `emit_compile_`.
**CN:** 检查 `not self._is_sycl()`，并在函数 `emit_compile_` 中选择匹配的分支。

##### Line 462 — Comment or spacing block

```python
        # Comile and load the library
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 463-464 — Call `compile_with_nvcc`

```python
        compile_with_nvcc(cmd, source_buffer_host,
                          error_file="./cutlass_python_compilation_host_error.txt")
```
**EN:** Calls `compile_with_nvcc` for side effects or initialization work in function `emit_compile_`.
**CN:** 调用 `compile_with_nvcc` 执行副作用或初始化逻辑；该语句位于在函数 `emit_compile_` 中。

##### Line 465 — Assign `host_lib`

```python
        host_lib = ctypes.CDLL(temp_dst.name)
```
**EN:** Assigns `host_lib` from `ctypes.CDLL(temp_dst.name)`, establishing state in function `emit_compile_`.
**CN:** 将 `host_lib` 赋值为 `ctypes.CDLL(temp_dst.name)`，用于在函数 `emit_compile_` 中建立状态。

##### Line 467 — Return

```python
        return cubin_image, host_lib, temp_dst
```
**EN:** Returns `(cubin_image, host_lib, temp_dst)` to the caller.
**CN:** 向调用方返回 `(cubin_image, host_lib, temp_dst)`。

#### Lines 469-472 — Function `add_module`

```python
    def add_module(self, operations, compile_options=None, bypass_cache=False):
        """
        Insert a new compiled device module
        """
```
**EN:** Defines function `add_module` with parameters `self, operations, compile_options, bypass_cache`. Purpose: Insert a new compiled device module
**CN:** 定义函数 `add_module`，参数为 `self, operations, compile_options, bypass_cache`。 其用途：Insert a new compiled device module

##### Lines 473-477 — Assign `include_paths`

```python
        include_paths = [
            CUTLASS_PATH + "/include",
            CUTLASS_PATH + "/tools/util/include",
            CUTLASS_PATH + "/python/cutlass/cpp/include",
        ]
```
**EN:** Assigns `include_paths` from `[CUTLASS_PATH + '/include', CUTLASS_PATH + '/tools/util/include', CUTLASS_PATH + '/python/cutlass/cpp/include']`, establishing state in function `add_module`.
**CN:** 将 `include_paths` 赋值为 `[CUTLASS_PATH + '/include', CUTLASS_PATH + '/tools/util/include', CUTLASS_PATH + '/python/cutlass/cpp/include']`，用于在函数 `add_module` 中建立状态。

##### Line 479 — Assign `device_flag`

```python
        device_flag = None
```
**EN:** Assigns `device_flag` from `None`, establishing state in function `add_module`.
**CN:** 将 `device_flag` 赋值为 `None`，用于在函数 `add_module` 中建立状态。

##### Lines 480-502 — Conditional `not self._is_sycl()`

```python
        if not self._is_sycl():
            include_paths.append(cuda_install_path() + "/include")

            cutlass_cppgen.initialize_cuda_context()
            arch = device_cc()
            host_compile_options = CompilationOptions(
                self._nvcc_compile_options, arch, include_paths, False)
        else:
            cutlass_cppgen.initialize_sycl_context()
            cc = device_cc()
            if cc == 12:
                arch = "intel_gpu_pvc"
            elif cc == 20:
                # Multi target case i.e G21, G31
                arch = "spir64_gen"
                device_flag = "-device bmg-g21,bmg-g31"
            else:
                arch = "intel_gpu_bmg_g21"

            flags = ["-std=c++17", "-DCUTLASS_ENABLE_SYCL", "-DSYCL_INTEL_TARGET"]
            host_compile_options = CompilationOptions(
                flags,
                arch, include_paths, True, device_flag)
```
**EN:** Checks `not self._is_sycl()` and selects the matching branch in function `add_module`.
**CN:** 检查 `not self._is_sycl()`，并在函数 `add_module` 中选择匹配的分支。

##### Lines 504-506 — Conditional `compile_options is None`

```python
        if compile_options is None:
            compile_options = CompilationOptions(
                self.default_compile_options, arch, include_paths, self._is_sycl(), device_flag)
```
**EN:** Checks `compile_options is None` and selects the matching branch in function `add_module`.
**CN:** 检查 `compile_options is None`，并在函数 `add_module` 中选择匹配的分支。

##### Line 507 — Comment or spacing block

```python
        # save the cubin
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 508 — Assign `operation_key`

```python
        operation_key = []
```
**EN:** Assigns `operation_key` from `[]`, establishing state in function `add_module`.
**CN:** 将 `operation_key` 赋值为 `[]`，用于在函数 `add_module` 中建立状态。

##### Line 509 — Assign `operation_list`

```python
        operation_list = []
```
**EN:** Assigns `operation_list` from `[]`, establishing state in function `add_module`.
**CN:** 将 `operation_list` 赋值为 `[]`，用于在函数 `add_module` 中建立状态。

##### Lines 510-530 — Loop over `operations`

```python
        for operation in operations:
            # step 1: get kernel string as key
            key = operation.rt_module.emit() + operation.procedural_name() + self.backend
            # step 1: check if the operation is in cache
            compiled_kernel = self.compiled_cache_device.get(key)

            if compiled_kernel is None and not bypass_cache:
                hit = self.load_operation(key, getattr( operation.rt_module, "extra_funcs", {}))
                if hit:
                    compiled_kernel = self.compiled_cache_device.get(key)
                    assert compiled_kernel is not None
            if compiled_kernel is not None:
                operation.rt_module.kernel = compiled_kernel
                compiled_host_fns = self.compiled_cache_host.get(key)
                assert compiled_host_fns is not None
                for key in compiled_host_fns.keys():
                    setattr(operation.rt_module, key, compiled_host_fns[key])
                operation.rt_module.initialize()
            else:
                operation_list.append(operation.rt_module)
                operation_key.append(key)
```
**EN:** Iterates `operation` over `operations` to repeat a processing step.
**CN:** 让 `operation` 遍历 `operations`，从而重复执行处理步骤。

##### Lines 532-604 — Conditional `len(operation_list) > 0`

```python
        if len(operation_list) > 0:
            cubin_image, host_lib, host_file = self.emit_compile_(
                operation_list, compile_options, host_compile_options)

            if self._is_sycl():
                if cubin_image is None:
                    raise RuntimeError("SYCL compilation failed, see debug log")
                q = dpctl.SyclQueue(cutlass_cppgen.sycl_device())
                program = dpctl.program.create_program_from_spirv(
                    q, cubin_image)
            else:
                err, module = cuda.cuModuleLoadData(cubin_image)
                if err != cuda.CUresult.CUDA_SUCCESS:
                    raise RuntimeError("Cuda Error: {}".format(err))

            operation_name = []
            operation_attr = []
            for operation, key in zip(operation_list, operation_key):
                # get device kernels
                if self._is_sycl():
                    # Free function kernels always have a name prefix.
                    fnName = f"__sycl_kernel_{operation.name()}"
                    operation.kernel = program.get_sycl_kernel(fnName)
                else:
                    err, operation.kernel = cuda.cuModuleGetFunction(
                        module,
                        bytes(str.encode(operation.name()))
                    )
                operation_name.append(operation.name())
                self.compiled_cache_device[key] = operation.kernel
                # get host functions
                compiled_host_fns = {}
                op_attr = []

                # get param size
                func_name = operation.name() + "_get_param_size"
                func = getattr(host_lib, func_name)
                param_size = func()

                func_name = operation.name() + "_get_params"
                func = getattr(host_lib, func_name)
                func.argtype = operation.argtype
                func.restype = ctypes.POINTER(ctypes.c_char * param_size)
                setattr(operation, "get_args", func)
                compiled_host_fns["get_args"] = func

                # set shared memory size
                func_name = operation.name() + "_shared_memory_size"
                func = getattr(host_lib, func_name)
                setattr(operation, "shared_memory_capacity", func())
                compiled_host_fns["shared_memory_capacity"] = func()
                # set the maximum dynamic shared size
                operation.initialize()

                # get extra functions
                op_attr.append(param_size)

                if hasattr(operation, "extra_funcs"):
                    for suffix, ret_type in operation.extra_funcs.items():
                        func_name = operation.name() + "_" + suffix
                        func = getattr(host_lib, func_name)
                        if ret_type is not None:
                            func.restype = ret_type
                        setattr(operation, suffix, func)
                        compiled_host_fns[suffix] = func
                        op_attr.append(suffix)

                operation_attr.append(op_attr)
                self.compiled_cache_host[key] = compiled_host_fns

            for (key, operation_name, operation_attr,) in zip(operation_key, operation_name, operation_attr):
                self.insert_operation(
                    key, cubin_image, host_file.name, operation_name, operation_attr)
```
**EN:** Checks `len(operation_list) > 0` and selects the matching branch in function `add_module`.
**CN:** 检查 `len(operation_list) > 0`，并在函数 `add_module` 中选择匹配的分支。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `CompilationOptions`, `ArtifactManager`.
- **CN:** 顶层类：`CompilationOptions`, `ArtifactManager`。
- **EN:** Top-level functions: `compile_with_nvcc`, `convertToBinaryData`, `CDLLBin`.
- **CN:** 顶层函数：`compile_with_nvcc`, `convertToBinaryData`, `CDLLBin`。
- **EN:** This module organizes backend utilities, data structures, or runtime helpers used by cutlass_cppgen.
- **CN:** 该模块组织 cutlass_cppgen 所需的后端工具、数据结构或运行时辅助逻辑。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen`, `cutlass_cppgen.backend.gemm_operation`, `cutlass_cppgen.backend.library`, `cutlass_cppgen.backend.utils.device`, `cutlass_cppgen.utils.lazy_import`, `cutlass_library`, `cutlass_library.arch_constants`
- **Standard & third-party / 标准库与第三方:** `ctypes`, `functools`, `json`, `os`, `pathlib`, `sqlite3`, `subprocess`, `tempfile`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
