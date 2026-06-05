# run_xe_evt_tests.py — Code Analysis / 代码分析

## Source / 来源
- Path / 路径: `test/python/cutlass/evt/run_xe_evt_tests.py`
- **EN:** Implements a command-line driver that groups EVT tests into named suites, prints summaries, and can export CSV reports.
- **CN:** 实现 EVT 测试的命令行驱动：将测试组织为具名套件、打印摘要，并可导出 CSV 报告。

## Line-by-Line Analysis / 逐行分析

### Lines 32-94
```python
import argparse
import sys
import unittest
import os

from utils.test_report import write_test_results_to_csv, print_test_summary, TestResultWithSuccesses

# Define logs directory at repository root level (sycl-tla/logs/)
SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
REPO_ROOT = os.path.abspath(os.path.join(SCRIPT_DIR, '../../../../'))
LOGS_DIR = os.path.join(REPO_ROOT, 'logs')

# Define test suites - each suite contains a list of test modules
TEST_SUITES = {
    'xe_evt_ci': [
        'evt_compute_xe_sm80_90.TestEVTCompute.test_arith',
        'evt_compute_xe_sm80_90.TestEVTCompute.test_func_call',
        'evt_compute_xe_sm80_90.TestEVTCompute.test_func_call2',
        'evt_compute_xe_sm80_90.TestEVTCompute.test_gelu',
        'evt_compute_xe_sm80_90.TestEVTCompute.test_sigmoid',
        'evt_compute_xe_sm80_90.TestEVTCompute.test_tanh',
        'evt_compute_xe_sm80_90.TestEVTCompute.test_exp',
        'evt_load_xe_sm80_90.TestEVTLoad.test_tensor_load',
        'evt_load_xe_sm80_90.TestEVTLoad.test_row_broadcast',
        'evt_load_xe_sm80_90.TestEVTLoad.test_column_broadcast',
        'evt_load_xe_sm80_90.TestEVTLoad.test_scalar_broadcast',
        'evt_store_xe_sm80_90.TestEVTStore.test_invalid_store',
        'evt_store_xe_sm80_90.TestEVTStore.test_aux_store',
        'evt_store_xe_sm80_90.TestEVTStore.test_col_reduce',
        'evt_store_xe_sm80_90.TestEVTStore.test_row_reduce',
        'evt_store_xe_sm80_90.TestEVTStore.test_scalar_reduce',
        'evt_store_xe_sm80_90.TestEVTStore.test_store_with_multiple_reductions',
        'evt_mixed_xe_sm80_90.TestEVTMixed.test_same_variable_used_multiple_times',
        'evt_mixed_xe_sm80_90.TestEVTMixed.test_no_lca',
        'evt_mixed_xe_sm80_90.TestEVTMixed.test_mixed_dag',
        'evt_mixed_xe_sm80_90.TestEVTMixed.test_mixed_dag_no_batch',
        'evt_layout_xe_sm80_90.TestEVTLayout.test_permute_1',
        'evt_layout_xe_sm80_90.TestEVTLayout.test_reshape',
        'evt_layout_xe_sm80_90.TestEVTLayout.test_reshape2'
    ],
    'evt_compute': [
        'evt_compute_xe_sm80_90.TestEVTCompute',
    ],
    'evt_load': [
        'evt_load_xe_sm80_90.TestEVTLoad',
    ],
    'evt_store': [
        'evt_store_xe_sm80_90.TestEVTStore',
    ],
    'evt_mixed': [
        'evt_mixed_xe_sm80_90.TestEVTMixed',
    ],
    'evt_layout': [
        'evt_layout_xe_sm80_90.TestEVTLayout',
    ],
    'all': [
        'evt_compute_xe_sm80_90.TestEVTCompute',
        'evt_layout_xe_sm80_90.TestEVTLayout',
        'evt_load_xe_sm80_90.TestEVTLoad',
        'evt_store_xe_sm80_90.TestEVTStore',
        'evt_mixed_xe_sm80_90.TestEVTMixed',
    ],
}
```
**EN:** This opening block establishes the module context. It imports the libraries needed for code generation, testing, and tensor/runtime support. The large `TEST_SUITES` dictionary is the central registry mapping suite names to concrete test classes or methods. Path constants compute the repository root and the `logs/` destination for optional reports.
**CN:** 这一开头代码块建立了模块的上下文。 它导入了代码生成、测试以及张量/运行时支持所需的库。 大型 `TEST_SUITES` 字典是核心注册表，用于把套件名映射到具体测试类或测试方法。 这些路径常量会计算仓库根目录以及可选报告输出用的 `logs/` 目标路径。

### Lines 97-103
```python
def list_suites():
    """List all available test suites."""
    print("Available test suites:")
    for suite_name in sorted(TEST_SUITES.keys()):
        print(f"  - {suite_name}")
        for test_module in TEST_SUITES[suite_name]:
            print(f"      {test_module}")
```
**EN:** `list_suites()` is a top-level function. The docstring describes it as: List all available test suites. Key steps in this block are `print('Available test suites:')`. The surrounding control flow iterates over `sorted(TEST_SUITES.keys())` to cover multiple shapes/configurations.
**CN:** `list_suites()` 是一个顶层函数。 文档字符串将其描述为：List all available test suites. 此代码块中的关键步骤是 `print('Available test suites:')`。 外层控制流会遍历 `sorted(TEST_SUITES.keys())`，以覆盖多种形状或配置。

### Lines 106-132
```python
def run_test_suite(suite_name, write_csv=False):
    """Run a specific test suite."""
    if suite_name not in TEST_SUITES:
        print(f"Error: Test suite '{suite_name}' not found.")
        print("\nUse --list to see available test suites.")
        return False, None
    
    # Create logs directory if it doesn't exist
    if write_csv:
        os.makedirs(LOGS_DIR, exist_ok=True)
        print(f"Output will be saved to: {LOGS_DIR}")
    
    test_modules = TEST_SUITES[suite_name]
    loader = unittest.TestLoader()
    
    suite = loader.loadTestsFromNames(test_modules)
    testRunner = unittest.TextTestRunner(verbosity=2, resultclass=TestResultWithSuccesses)
    results = testRunner.run(suite)

    # Always print summary
    print_test_summary(results, suite_name)
    
    # Write results to CSV if requested
    if write_csv:
        write_test_results_to_csv(results, suite_name, output_dir=LOGS_DIR)
    
    return results.wasSuccessful(), results
```
**EN:** `run_test_suite(suite_name, write_csv)` is a top-level function. The docstring describes it as: Run a specific test suite. Key steps in this block are `test_modules = TEST_SUITES[suite_name]; loader = unittest.TestLoader(); suite = loader.loadTestsFromNames(test_modules); testRunner = unittest.TextTestRunner(verbosity=2, resultclass=TestResultWithSuccesses)`. Named suites are resolved dynamically through `unittest.TestLoader.loadTestsFromNames`. Optional CSV export is delegated to the shared reporting utility.
**CN:** `run_test_suite(suite_name, write_csv)` 是一个顶层函数。 文档字符串将其描述为：Run a specific test suite. 此代码块中的关键步骤是 `test_modules = TEST_SUITES[suite_name]; loader = unittest.TestLoader(); suite = loader.loadTestsFromNames(test_modules); testRunner = unittest.TextTestRunner(verbosity=2, resultclass=TestResultWithSuccesses)`。 具名套件是通过 `unittest.TestLoader.loadTestsFromNames` 动态解析的。 可选的 CSV 导出被委托给共享的报告工具函数处理。

### Lines 135-184
```python
def main():
    parser = argparse.ArgumentParser(
        description='Run EVT test suites',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  python run_xe_evt_tests.py -j xe_evt_ci
  python run_xe_evt_tests.py --job evt_layout -o
  python run_xe_evt_tests.py --list
  python run_xe_evt_tests.py -j all --output
        """
    )
    
    parser.add_argument(
        '-j', '--job',
        type=str,
        help='Test suite to run (use --list to see available suites)'
    )
    
    parser.add_argument(
        '--list',
        action='store_true',
        help='List all available test suites'
    )
    
    parser.add_argument(
        '-o', '--output',
        action='store_true',
        help='Write test results to CSV file (format: test_results_<suite>_<timestamp>.csv)'
    )
    
    args = parser.parse_args()
    
    if args.list:
        list_suites()
        return 0
    
    if not args.job:
        parser.print_help()
        print("\nError: Please specify a test suite using -j/--job or use --list to see available suites.")
        return 1
    
    success, results = run_test_suite(args.job, args.output)
    
    if not success:
        print(f"\nTest suite '{args.job}' failed!")
        return 1
    
    print(f"\nTest suite '{args.job}' passed!")
    return 0
```
**EN:** `main()` is a top-level function. Key steps in this block are `parser = argparse.ArgumentParser(description='Run EVT test suites', formatter_class=argparse.RawDescriptionHelpFormatter, epilog='\nExamples:\n python run_xe_evt_tests.py -j xe_evt_ci\n python run_xe_evt_tests.py --job evt_layout -o\n py...`.
**CN:** `main()` 是一个顶层函数。 此代码块中的关键步骤是 `parser = argparse.ArgumentParser(description='Run EVT test suites', formatter_class=argparse.RawDescriptionHelpFormatter, epilog='\nExamples:\n python run_xe_evt_tests.py -j xe_evt_ci\n python run_xe_evt_tests.py --job evt_layout -o\n py...`。

### Lines 187-188
```python
if __name__ == '__main__':
    sys.exit(main())
```
**EN:** This conditional block is guarded by `__name__ == '__main__'`. It acts as the executable entry point and runs `sys.exit(main())` when the file is invoked as a script.
**CN:** 该条件块由 `__name__ == '__main__'` 控制。 当文件以脚本方式运行时，它就是可执行入口，并会执行 `sys.exit(main())`。

## Key Concepts / 关键概念
- **EN:** Named test suite registry via `TEST_SUITES`.
  **CN:** 通过 `TEST_SUITES` 建立具名测试套件注册表。
- **EN:** CLI dispatch with `argparse`.
  **CN:** 使用 `argparse` 进行命令行分发。
- **EN:** Custom result handling and human-readable summaries.
  **CN:** 自定义结果处理与可读性摘要。
- **EN:** Optional CSV logging under the repository `logs/` directory.
  **CN:** 可选地将 CSV 日志输出到仓库 `logs/` 目录。

## Dependencies / 依赖关系
- `import argparse`
  - **EN:** Handles command-line parsing for the standalone runner.
  - **CN:** 为独立运行脚本处理命令行参数解析。
- `import sys`
  - **EN:** Provides standard-library helpers used by the script logic.
  - **CN:** 提供脚本逻辑所需的标准库辅助功能。
- `import unittest`
  - **EN:** Supplies the Python unit-test framework and result/runner abstractions.
  - **CN:** 提供 Python 单元测试框架以及结果/运行器抽象。
- `import os`
  - **EN:** Resolves filesystem paths used for discovery or report output.
  - **CN:** 用于解析测试发现或报告输出所需的文件系统路径。
- `from utils.test_report import write_test_results_to_csv, print_test_summary, TestResultWithSuccesses`
  - **EN:** Supports timestamped CSV report generation.
  - **CN:** 用于生成带时间戳的 CSV 报告。
- `utils.test_report` helpers
  - **EN:** Provide suite summaries and CSV persistence for the CLI runner.
  - **CN:** 为命令行运行器提供套件摘要与 CSV 持久化功能。
