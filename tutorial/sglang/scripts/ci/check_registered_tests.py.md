# check_registered_tests.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/check_registered_tests.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `check_registered_tests` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `check_registered_tests` 流程，主要负责CI 编排、测试执行。它属于 `ci` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-8: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Pre-commit hook: validate that all Python test files under test/registered/
contain a CI registry call (register_cuda_ci, register_amd_ci, etc.).

Reuses ut_parse_one_file() from ci_register.py (AST-based parsing)
to match the same logic used by run_suite.py's collect_tests().
"""
```
**EN:** Pre-commit hook: validate that all Python test files under test/registered/ contain a CI registry call (register_cuda_ci, register_amd_ci, etc.).
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 10-13: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import glob
import importlib.util
import os
import sys
```
**EN:** This block loads glob, importlib.util, os, sys. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 glob, importlib.util, os, sys。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 16-52: Defines the `main` routine / 定义 `main` 例程
```python
def main() -> int:
    # Import ci_register directly to avoid pulling in all of sglang
    spec = importlib.util.spec_from_file_location(
        "ci_register",
        os.path.join("python", "sglang", "test", "ci", "ci_register.py"),
    )
    ci_register = importlib.util.module_from_spec(spec)
    spec.loader.exec_module(ci_register)

    # Same filter as run_suite.py: skip conftest.py, __init__.py, and utils.py
    files = sorted(
        f
        for f in glob.glob("test/registered/**/*.py", recursive=True)
        if os.path.basename(f) not in ("conftest.py", "__init__.py", "utils.py")
    )
    if not files:
        return 0

    errors = []
    for f in files:
        try:
            registries, _has_main_entry = ci_register.ut_parse_one_file(f)
            if len(registries) == 0:
                errors.append(f)
        except Exception:
            # Skip files that can't be parsed (syntax errors, etc.)
            pass

    if errors:
        print("ERROR: Files in test/registered/ missing CI registry call:")
        print("  Move manual-only tests to test/manual/.\n")
        for f in errors:
            print(f"  {f}")
        print()
        return 1

    return 0
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会输出状态信息。

### Lines 55-56: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    sys.exit(main())
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It controls process exit status.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会控制进程退出状态。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Process control** / 进程控制

## Dependencies / 依赖关系
- **Standard library / 标准库**: `glob`, `importlib`, `os`, `sys`
- **External commands inferred from code / 从代码推断的外部命令**: `python`
