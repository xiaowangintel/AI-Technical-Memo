# run_all_tests.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `test/python/pycute/run_all_tests.py`
- **EN:** This utility script discovers every `test_*.py` file in the PyCuTe test directory, configures logging from the command line, and fails the process if any discovered unit test fails.
- **CN:** 该工具脚本会发现 PyCuTe 测试目录中的所有 `test_*.py` 文件，根据命令行参数配置日志级别，并在任一单元测试失败时让进程返回失败。

## Line-by-Line Analysis / 逐行分析

### Lines 33-40

```python
"""
Utility script for discovering and running all PyCuTe tests
"""

import argparse
import logging
import pathlib
import unittest
```

**EN:** The docstring states that the file is a discovery-and-runner utility, and the imports reflect that role: argument parsing, logging, filesystem path handling, and unittest execution.

**CN:** 文档字符串明确指出该文件是一个“发现并执行测试”的工具，导入项也完全围绕这一职责展开：参数解析、日志、路径处理以及 unittest 执行。

### Lines 43-57

```python
def numeric_log_level(log_level: str) -> int:
  """
  Converts the string identifier of the log level into the numeric identifier used
  in setting the log level

  :param x: string representation of log level (e.g., 'INFO', 'DEBUG')
  :type x: str

  :return: numeric representation of log level
  :rtype: int
  """
  numeric_level = getattr(logging, log_level.upper(), None)
  if not isinstance(numeric_level, int):
    raise ValueError(f"Invalid log level: {log_level}")
  return numeric_level
```

**EN:** `numeric_log_level()` converts a string such as `info` or `DEBUG` into the integer constant used by `logging`. It relies on `getattr(logging, log_level.upper(), None)` and explicitly raises `ValueError` when the lookup does not yield an integer.

**CN:** `numeric_log_level()` 把诸如 `info` 或 `DEBUG` 之类的字符串转换成 `logging` 使用的整数常量。其实现依赖 `getattr(logging, log_level.upper(), None)`，并在查找结果不是整数时显式抛出 `ValueError`。

### Lines 60-75

```python
if __name__ == "__main__":
  parser = argparse.ArgumentParser()
  parser.add_argument("--log-level", default='info', type=numeric_log_level, required=False,
                      help='Logging level to be used by the generator script')
  args = parser.parse_args()

  # Set the logging level based on the user-provided `--log-level` command-line option
  logging.basicConfig(level=args.log_level)

  loader = unittest.TestLoader()
  script_dir = str(pathlib.Path(__file__).parent.resolve()) + '/'
  tests = loader.discover(script_dir, "test_*.py")
  test_runner = unittest.runner.TextTestRunner()
  results = test_runner.run(tests)
  if not results.wasSuccessful():
    raise Exception("Test cases failed")
```

**EN:** The main block builds an `ArgumentParser`, uses `numeric_log_level` as the `type=` converter for `--log-level`, initializes the global logging system, discovers tests in the current script directory, runs them with a text runner, and raises a generic exception if the suite is not successful.

**CN:** 主执行块先创建 `ArgumentParser`，把 `numeric_log_level` 作为 `--log-level` 的 `type=` 转换器，再初始化全局日志系统，发现当前脚本目录下的测试，用文本运行器执行，并在测试结果不成功时抛出通用异常。

## Key Concepts / 关键概念
- **EN:** Argparse type conversion is used to validate and normalize the log-level option early.
  **CN:** 这里利用 argparse 的类型转换机制，尽早完成日志级别参数的校验与归一化。
- **EN:** `unittest.TestLoader().discover()` makes the runner directory-driven instead of relying on a hard-coded file list.
  **CN:** 通过 `unittest.TestLoader().discover()`，运行器以目录发现的方式工作，而不是依赖手工维护的文件清单。
- **EN:** A failing test run becomes a failing process through an explicit post-run exception.
  **CN:** 通过在测试结束后显式抛出异常，把“测试失败”进一步转换成“进程失败”。

## Dependencies / 依赖关系
- **EN:** Standard library only: `argparse`, `logging`, `pathlib`, `unittest`.
  **CN:** 仅依赖标准库：`argparse`、`logging`、`pathlib`、`unittest`。
- **EN:** Runtime dependency on sibling files matching `test_*.py` in the same directory.
  **CN:** 运行时依赖于同目录下所有匹配 `test_*.py` 的兄弟测试文件。
