# lit.cfg.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/lit.cfg.py`
- **Purpose / 目的:** **EN:** This python configuration file exercises lit test-suite configuration in Triton's test infrastructure coverage. **CN:** 该Python 配置文件用于覆盖 Triton 测试基础设施中的lit 测试套件配置相关场景。
- **Validation / 验证内容:** **EN:** The file documents and validates lit test-suite configuration in the surrounding test infrastructure. **CN:** 该文件在周边测试基础设施中记录并验证 lit 测试套件配置。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```python
# -*- Python -*-
# ruff: noqa: F821
```
**EN:** This comment block explains the intent of the nearby lit configuration settings.
**CN:** 这一注释块说明了附近 lit 配置项的用途。

### Line 4
```python
import os
```
**EN:** This block imports lit/LLVM helper modules used to build the Triton test configuration.
**CN:** 这一块导入了构建 Triton 测试配置所需的 lit/LLVM 辅助模块。

### Lines 6-9
```python
import lit.formats
import lit.util
from lit.llvm import llvm_config
from lit.llvm.subst import ToolSubst
```
**EN:** This block imports lit/LLVM helper modules used to build the Triton test configuration.
**CN:** 这一块导入了构建 Triton 测试配置所需的 lit/LLVM 辅助模块。

### Line 11
```python
# Configuration file for the 'lit' test runner
```
**EN:** This comment block explains the intent of the nearby lit configuration settings.
**CN:** 这一注释块说明了附近 lit 配置项的用途。

### Lines 13-15
```python
# (config is an instance of TestingConfig created when discovering tests)
# name: The name of this test suite
config.name = 'TRITON'
```
**EN:** This block configures lit settings such as `name`. These assignments decide how Triton's test suite is discovered and executed.
**CN:** 这一块配置了 lit 选项，例如 `name`。这些赋值决定了 Triton 测试集如何被发现和执行。

### Line 17
```python
config.test_format = lit.formats.ShTest(not llvm_config.use_lit_shell)
```
**EN:** This block configures lit settings such as `test_format`, `use_lit_shell`. These assignments decide how Triton's test suite is discovered and executed.
**CN:** 这一块配置了 lit 选项，例如 `test_format`, `use_lit_shell`。这些赋值决定了 Triton 测试集如何被发现和执行。

### Lines 19-20
```python
# suffixes: A list of file extensions to treat as test files.
config.suffixes = ['.mlir', '.ll']
```
**EN:** This block configures lit settings such as `suffixes`. These assignments decide how Triton's test suite is discovered and executed.
**CN:** 这一块配置了 lit 选项，例如 `suffixes`。这些赋值决定了 Triton 测试集如何被发现和执行。

### Lines 22-23
```python
# test_source_root: The root path where tests are located.
config.test_source_root = os.path.dirname(__file__)
```
**EN:** This block configures lit settings such as `test_source_root`. These assignments decide how Triton's test suite is discovered and executed.
**CN:** 这一块配置了 lit 选项，例如 `test_source_root`。这些赋值决定了 Triton 测试集如何被发现和执行。

### Lines 25-29
```python
# test_exec_root: The root path where tests should be run.
config.test_exec_root = os.path.join(config.triton_obj_root, 'test')
config.substitutions.append(('%PATH%', config.environment['PATH']))
config.substitutions.append(("%shlibdir", config.llvm_shlib_dir))
config.substitutions.append(("%shlibext", config.llvm_shlib_ext))
```
**EN:** This block configures lit settings such as `test_exec_root`, `triton_obj_root`, `substitutions`, `environment`, `llvm_shlib_dir`, `llvm_shlib_ext`. These assignments decide how Triton's test suite is discovered and executed.
**CN:** 这一块配置了 lit 选项，例如 `test_exec_root`, `triton_obj_root`, `substitutions`, `environment`, `llvm_shlib_dir`, `llvm_shlib_ext`。这些赋值决定了 Triton 测试集如何被发现和执行。

### Line 31
```python
llvm_config.with_system_environment(['HOME', 'INCLUDE', 'LIB', 'TMP', 'TEMP'])
```
**EN:** This block propagates environment variables and search paths that the lit test harness needs at runtime.
**CN:** 这一块传播 lit 测试框架在运行时需要的环境变量和搜索路径。

### Line 33
```python
# llvm_config.use_default_substitutions()
```
**EN:** This comment block explains the intent of the nearby lit configuration settings.
**CN:** 这一注释块说明了附近 lit 配置项的用途。

### Lines 35-38
```python
# excludes: A list of directories to exclude from the testsuite. The 'Inputs'
# subdirectories contain auxiliary inputs for various tests in their parent
# directories.
config.excludes = ['Inputs', 'Examples', 'CMakeLists.txt', 'README.txt', 'LICENSE.txt']
```
**EN:** This block configures lit settings such as `excludes`. These assignments decide how Triton's test suite is discovered and executed.
**CN:** 这一块配置了 lit 选项，例如 `excludes`。这些赋值决定了 Triton 测试集如何被发现和执行。

### Lines 40-41
```python
# test_source_root: The root path where tests are located.
config.test_source_root = os.path.dirname(__file__)
```
**EN:** This block configures lit settings such as `test_source_root`. These assignments decide how Triton's test suite is discovered and executed.
**CN:** 这一块配置了 lit 选项，例如 `test_source_root`。这些赋值决定了 Triton 测试集如何被发现和执行。

### Lines 43-46
```python
# test_exec_root: The root path where tests should be run.
config.test_exec_root = os.path.join(config.triton_obj_root, 'test')
config.triton_tools_dir = os.path.join(config.triton_obj_root, 'bin')
config.filecheck_dir = os.path.join(config.triton_obj_root, 'bin', 'FileCheck')
```
**EN:** This block configures lit settings such as `test_exec_root`, `triton_obj_root`, `triton_tools_dir`, `filecheck_dir`. These assignments decide how Triton's test suite is discovered and executed.
**CN:** 这一块配置了 lit 选项，例如 `test_exec_root`, `triton_obj_root`, `triton_tools_dir`, `filecheck_dir`。这些赋值决定了 Triton 测试集如何被发现和执行。

### Lines 48-51
```python
# FileCheck -enable-var-scope is enabled by default in MLIR test
# This option avoids to accidentally reuse variable across -LABEL match,
# it can be explicitly opted-in by prefixing the variable name with $
config.environment["FILECHECK_OPTS"] = "--enable-var-scope"
```
**EN:** This block propagates environment variables and search paths that the lit test harness needs at runtime.
**CN:** 这一块传播 lit 测试框架在运行时需要的环境变量和搜索路径。

### Line 53
```python
tool_dirs = [config.triton_tools_dir, config.llvm_tools_dir, config.filecheck_dir]
```
**EN:** This block configures lit settings such as `triton_tools_dir`, `llvm_tools_dir`, `filecheck_dir`. These assignments decide how Triton's test suite is discovered and executed.
**CN:** 这一块配置了 lit 选项，例如 `triton_tools_dir`, `llvm_tools_dir`, `filecheck_dir`。这些赋值决定了 Triton 测试集如何被发现和执行。

### Lines 55-64
```python
# Tweak the PATH to include the tools dir.
for d in tool_dirs:
    llvm_config.with_environment('PATH', d, append_path=True)
tools = [
    'triton-opt',
    'triton-llvm-opt',
    'mlir-translate',
    'llc',
    ToolSubst('%PYTHON', config.python_executable, unresolved='ignore'),
]
```
**EN:** This block defines tool substitutions so test scripts can call `triton-opt`, LLVM tools, and Python in a portable way.
**CN:** 这一块定义了工具替换规则，使测试脚本可以以可移植方式调用 `triton-opt`、LLVM 工具和 Python。

### Lines 66-68
```python
# Static libraries are not built if TRITON_EXT_ENABLED is ON.
if config.triton_ext_enabled:
    config.available_features.add("triton-ext-enabled")
```
**EN:** This block configures lit settings such as `triton_ext_enabled`, `available_features`. These assignments decide how Triton's test suite is discovered and executed.
**CN:** 这一块配置了 lit 选项，例如 `triton_ext_enabled`, `available_features`。这些赋值决定了 Triton 测试集如何被发现和执行。

### Line 70
```python
llvm_config.add_tool_substitutions(tools, tool_dirs)
```
**EN:** This block defines tool substitutions so test scripts can call `triton-opt`, LLVM tools, and Python in a portable way.
**CN:** 这一块定义了工具替换规则，使测试脚本可以以可移植方式调用 `triton-opt`、LLVM 工具和 Python。

### Lines 72-75
```python
# TODO: what's this?
llvm_config.with_environment('PYTHONPATH', [
    os.path.join(config.mlir_binary_dir, 'python_packages', 'triton'),
], append_path=True)
```
**EN:** This block propagates environment variables and search paths that the lit test harness needs at runtime.
**CN:** 这一块传播 lit 测试框架在运行时需要的环境变量和搜索路径。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's test infrastructure coverage and focuses on lit test-suite configuration.  
  **CN:** 该文件属于 Triton 的 测试基础设施 测试覆盖，关注点是 lit 测试套件配置。
- **EN:** The file configures lit discovery, environment variables, and tool substitution for the test suite.  
  **CN:** 该文件负责配置 lit 的测试发现、环境变量以及工具替换规则。

## Dependencies / 依赖关系

- **EN:** Python modules used here: `os`, `lit.formats`, `lit.util`, `lit.llvm`, `lit.llvm.subst`.  
  **CN:** 这里使用的 Python 模块：`os`、`lit.formats`、`lit.util`、`lit.llvm`、`lit.llvm.subst`。
- **EN:** The configuration also depends on LLVM-lit variables injected by the build (`config.*`, tool directories, environment).  
  **CN:** 该配置还依赖构建系统注入的 LLVM-lit 变量（`config.*`、工具目录、环境变量）。
