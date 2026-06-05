# run_all_tests.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/python/cutlass/gemm/run_all_tests.py`
- **Purpose (EN):** Provides a tiny CLI entry point that discovers every `gemm_*.py` test module in the directory and raises an exception if any suite fails.
- **用途 (CN):** 提供一个简洁的命令行入口：发现目录下所有 `gemm_*.py` 测试模块，并在任意套件失败时抛出异常。

## Line-by-Line Analysis / 逐行分析

### Lines 33-40 / 第 33-40 行

```python
import pathlib
import unittest


if __name__ == '__main__':
    loader = unittest.TestLoader()
    script_dir = str(pathlib.Path(__file__).parent.resolve()) + '/'
    tests = loader.discover(script_dir, 'gemm_*.py')
```

**EN:** The file imports only `pathlib` and `unittest`, which matches its narrow role as a test launcher.
Inside the `__main__` guard, it creates a `TestLoader`, resolves the directory containing the script, and discovers all test modules whose filenames match `gemm_*.py`.

**CN:** 该文件只导入了 `pathlib` 和 `unittest`，与其“纯测试启动器”的职责完全一致。
在 `__main__` 分支中，它先创建 `TestLoader`，再解析脚本所在目录，并发现所有文件名匹配 `gemm_*.py` 的测试模块。

### Lines 41-44 / 第 41-44 行

```python
    testRunner = unittest.runner.TextTestRunner()
    results = testRunner.run(tests)
    if not results.wasSuccessful():
        raise Exception('Test cases failed')
```

**EN:** A plain `TextTestRunner` executes the discovered suite.
Rather than relying only on process exit status from unittest, the file explicitly inspects `results.wasSuccessful()` and raises `Exception('Test cases failed')` on failure, making non-success easy to detect from external automation.

**CN:** 发现到的测试集合由标准的 `TextTestRunner` 执行。
文件没有只依赖 unittest 自身的退出状态，而是显式检查 `results.wasSuccessful()`；如果失败，就抛出 `Exception('Test cases failed')`，这样外部自动化工具更容易判断执行是否成功。

## Key Concepts / 关键概念

- **Pattern-based discovery / 基于模式的发现:** All GEMM suites are picked up by filename convention rather than a manually maintained list. / 所有 GEMM 测试都通过文件名约定自动发现，而不是手工维护清单。
- **Single entry point / 单一入口:** The script is meant to be run directly as a one-shot test aggregator. / 该脚本设计成可直接执行的一次性测试聚合入口。
- **Explicit failure signaling / 显式失败信号:** Raising an exception after `wasSuccessful()` gives calling scripts a simple success/failure contract. / 在 `wasSuccessful()` 之后主动抛异常，为调用脚本提供了简单明确的成功/失败契约。

## Dependencies / 依赖关系

- **`pathlib` / `pathlib`:** Resolves the script directory in a platform-friendly way. / 以跨平台方式解析脚本所在目录。
- **`unittest` / `unittest`:** Provides the loader, runner, and result object for discovery/execution. / 提供测试发现与执行所需的 loader、runner 和结果对象。
