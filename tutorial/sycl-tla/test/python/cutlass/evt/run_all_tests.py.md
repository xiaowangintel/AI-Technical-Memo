# run_all_tests.py — Code Analysis / 代码分析

## Source / 来源
- Path / 路径: `test/python/cutlass/evt/run_all_tests.py`
- **EN:** Provides a minimal unittest discovery entry point that runs every `evt_*.py` module in the directory.
- **CN:** 提供一个最小化的 unittest 发现入口，用于运行当前目录中全部 `evt_*.py` 模块。

## Line-by-Line Analysis / 逐行分析

### Lines 33-34
```python
import pathlib
import unittest
```
**EN:** This opening block establishes the module context. It imports the libraries needed for code generation, testing, and tensor/runtime support.
**CN:** 这一开头代码块建立了模块的上下文。 它导入了代码生成、测试以及张量/运行时支持所需的库。

### Lines 37-44
```python
if __name__ == '__main__':
    loader = unittest.TestLoader()
    script_dir = str(pathlib.Path(__file__).parent.resolve()) + '/'
    tests = loader.discover(script_dir, 'evt_*.py')
    testRunner = unittest.runner.TextTestRunner()
    results = testRunner.run(tests)
    if not results.wasSuccessful():
        raise Exception('Test cases failed')
```
**EN:** This conditional block is guarded by `__name__ == '__main__'`. It acts as the executable entry point and runs `loader = unittest.TestLoader(); script_dir = str(pathlib.Path(__file__).parent.resolve()) + '/'; tests = loader.discover(script_dir, 'evt_*.py')` when the file is invoked as a script.
**CN:** 该条件块由 `__name__ == '__main__'` 控制。 当文件以脚本方式运行时，它就是可执行入口，并会执行 `loader = unittest.TestLoader(); script_dir = str(pathlib.Path(__file__).parent.resolve()) + '/'; tests = loader.discover(script_dir, 'evt_*.py')`。

## Key Concepts / 关键概念
- **EN:** Directory-based unittest discovery.
  **CN:** 基于目录的 unittest 自动发现。
- **EN:** Pattern filtering with `evt_*.py`.
  **CN:** 通过 `evt_*.py` 进行模式过滤。
- **EN:** Fail-fast script behavior by raising on unsuccessful runs.
  **CN:** 通过失败即抛异常的方式实现脚本级失败传播。
- **EN:** Simple executable entry point.
  **CN:** 简洁的可执行入口。

## Dependencies / 依赖关系
- `import pathlib`
  - **EN:** Resolves filesystem paths used for discovery or report output.
  - **CN:** 用于解析测试发现或报告输出所需的文件系统路径。
- `import unittest`
  - **EN:** Supplies the Python unit-test framework and result/runner abstractions.
  - **CN:** 提供 Python 单元测试框架以及结果/运行器抽象。
