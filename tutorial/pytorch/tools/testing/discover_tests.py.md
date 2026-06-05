# discover_tests.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/discover_tests.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides shared testing infrastructure, test-selection helpers, or execution utilities for PyTorch CI.
- **Purpose (CN)**: 提供 PyTorch CI 使用的共享测试基础设施、测试选择辅助逻辑或执行工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```python
from __future__ import annotations

import glob
import os
import sys
from pathlib import Path
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, glob, os, and 2 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、glob、os 等共 5 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 8-13
```python

CPP_TEST_PREFIX = "cpp"
CPP_TEST_PATH = "build/bin"
CPP_TESTS_DIR = os.path.abspath(os.getenv("CPP_TESTS_DIR", default=CPP_TEST_PATH))
REPO_ROOT = Path(__file__).resolve().parents[2]
```
- **EN**: Configuration constants such as CPP_TEST_PREFIX, CPP_TEST_PATH, CPP_TESTS_DIR, and 1 more centralize defaults so later functions share the same policy knobs. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: CPP_TEST_PREFIX、CPP_TEST_PATH、CPP_TESTS_DIR 等共 4 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 14-19
```python

def parse_test_module(test: str) -> str:
    return test.split(".", maxsplit=1)[0]


def discover_tests(
```
- **EN**: This chunk defines `discover_tests`, which implements a focused step inside the testing infrastructure pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `discover_tests`，其作用是实现测试基础设施流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 20-31
```python
    base_dir: Path = REPO_ROOT / "test",
    cpp_tests_dir: str | Path | None = None,
    blocklisted_patterns: list[str] | None = None,
    blocklisted_tests: list[str] | None = None,
    extra_tests: list[str] | None = None,
) -> list[str]:
    """
    Searches for all python files starting with test_ excluding one specified by patterns.
    If cpp_tests_dir is provided, also scan for all C++ tests under that directory. They
    are usually found in build/bin
    """
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `discover_tests` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `discover_tests`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 32-39
```python
    def skip_test_p(name: str) -> bool:
        rc = False
        if blocklisted_patterns is not None:
            rc |= any(name.startswith(pattern) for pattern in blocklisted_patterns)
        if blocklisted_tests is not None:
            rc |= name in blocklisted_tests
        return rc
```
- **EN**: This chunk defines `skip_test_p`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `skip_test_p`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 40-51
```python
    # This supports symlinks, so we can link domain library tests to PyTorch test directory
    all_py_files = [
        Path(p) for p in glob.glob(f"{base_dir}/**/test_*.py", recursive=True)
    ]

    cpp_tests_dir = (
        f"{base_dir.parent}/{CPP_TEST_PATH}" if cpp_tests_dir is None else cpp_tests_dir
    )
    # CPP test files are located under pytorch/build/bin. Unlike Python test, C++ tests
    # are just binaries and could have any name, i.e. basic or atest
    all_cpp_files = [
        Path(p) for p in glob.glob(f"{cpp_tests_dir}/**/*", recursive=True)
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 52-59
```python
    ]

    rc = [str(fname.relative_to(base_dir))[:-3] for fname in all_py_files]
    # Add the cpp prefix for C++ tests so that we can tell them apart
    rc.extend(
        [
            parse_test_module(f"{CPP_TEST_PREFIX}/{fname.relative_to(cpp_tests_dir)}")
            for fname in all_cpp_files
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 60-67
```python
        ]
    )

    # Invert slashes on Windows
    if sys.platform == "win32":
        rc = [name.replace("\\", "/") for name in rc]
    rc = [test for test in rc if not skip_test_p(test)]
    if extra_tests is not None:
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 68-79
```python
        rc += extra_tests
    return sorted(rc)


TESTS = discover_tests(
    cpp_tests_dir=CPP_TESTS_DIR,
    blocklisted_patterns=[
        "ao",
        "custom_backend",
        "custom_operator",
        "fx",  # executed by test_fx.py
        "jit",  # executed by test_jit.py
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement. Configuration constants such as TESTS centralize defaults so later functions share the same policy knobs. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。 TESTS 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 80-91
```python
        "mobile",
        "onnx_caffe2",
        "package",  # executed by test_package.py
        "quantization",  # executed by test_quantization.py
        "autograd",  # executed by test_autograd.py
        "cpp_extensions/open_registration_extension/torch_openreg/tests",  # executed by test_openreg.py
    ],
    blocklisted_tests=[
        "test_bundled_images",
        "test_cpp_extensions_aot",
        "test_determination",
        "test_jit_fuser",
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。

### Lines 92-103
```python
        "test_jit_simple",
        "test_jit_string",
        "test_kernel_launch_checks",
        "test_nnapi",
        "test_static_runtime",
        "test_throughput_benchmark",
        "distributed/bin/test_script",
        "distributed/elastic/multiprocessing/bin/test_script",
        "distributed/launcher/bin/test_script",
        "distributed/launcher/bin/test_script_init_method",
        "distributed/launcher/bin/test_script_is_torchelastic_launched",
        "distributed/launcher/bin/test_script_local_rank",
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。

### Lines 104-115
```python
        "distributed/test_c10d_spawn",
        "distributions/test_transforms",
        "distributions/test_utils",
        "lazy/test_meta_kernel",
        "lazy/test_extract_compiled_graph",
        "test/inductor/test_aot_inductor_utils",
        "inductor/test_aoti_cross_compile_windows",
        "onnx/test_onnxscript_no_runtime",
        "onnx/test_pytorch_onnx_onnxruntime_cuda",
        "onnx/test_models",
        # These are not C++ tests
        f"{CPP_TEST_PREFIX}/CMakeFiles",
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。

### Lines 116-127
```python
        f"{CPP_TEST_PREFIX}/CTestTestfile.cmake",
        f"{CPP_TEST_PREFIX}/Makefile",
        f"{CPP_TEST_PREFIX}/cmake_install.cmake",
        f"{CPP_TEST_PREFIX}/c10_intrusive_ptr_benchmark",
        f"{CPP_TEST_PREFIX}/example_allreduce",
        f"{CPP_TEST_PREFIX}/parallel_benchmark",
        f"{CPP_TEST_PREFIX}/protoc",
        f"{CPP_TEST_PREFIX}/protoc-3.13.0.0",
        f"{CPP_TEST_PREFIX}/torch_shm_manager",
        f"{CPP_TEST_PREFIX}/tutorial_tensorexpr",
    ],
    extra_tests=[
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。

### Lines 128-139
```python
        "test_cpp_extensions_aot_ninja",
        "test_cpp_extensions_aot_no_ninja",
        "distributed/elastic/timer/api_test",
        "distributed/elastic/timer/local_timer_example",
        "distributed/elastic/timer/local_timer_test",
        "distributed/elastic/events/lib_test",
        "distributed/elastic/metrics/api_test",
        "distributed/elastic/utils/logging_test",
        "distributed/elastic/utils/util_test",
        "distributed/elastic/utils/distributed_test",
        "distributed/elastic/multiprocessing/api_test",
        "doctests",
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。

### Lines 140-145
```python
        "test_autoload_enable",
        "test_autoload_disable",
    ],
)
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。

### Lines 146-147
```python
if __name__ == "__main__":
    print(TESTS)
```
- **EN**: This chunk continues `skip_test_p` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `skip_test_p`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **Testing infrastructure**
  - EN: This file belongs to the testing infrastructure layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于测试基础设施层，应结合同一子目录中的相邻脚本一起理解。
- **Autograd generation**
  - EN: The file participates in turning operator metadata into backward formulas, wrappers, or saved-state logic.
  - CN: 该文件参与把算子元数据转换为反向公式、包装层或保存状态逻辑。
- **Build configuration**
  - EN: The file helps discover tools, cache build settings, or translate Python-side configuration into native build arguments.
  - CN: 该文件帮助发现工具、缓存构建设置，或把 Python 侧配置转换为原生构建参数。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **CPP_TEST_PREFIX**
  - EN: `CPP_TEST_PREFIX` is one of the main local symbols exposed or implemented here.
  - CN: `CPP_TEST_PREFIX` 是此处暴露或实现的主要局部符号之一。
- **CPP_TEST_PATH**
  - EN: `CPP_TEST_PATH` is one of the main local symbols exposed or implemented here.
  - CN: `CPP_TEST_PATH` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `glob`, `os`, `sys`, `pathlib`
- **Primary symbols in this file / 本文件核心符号**: `CPP_TEST_PREFIX`, `CPP_TEST_PATH`, `CPP_TESTS_DIR`, `REPO_ROOT`, `parse_test_module`, `discover_tests`, `TESTS`
