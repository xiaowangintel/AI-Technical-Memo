# test_report.py — Code Analysis / 代码分析

## Source / 来源
- Path / 路径: `test/python/cutlass/evt/utils/test_report.py`
- **EN:** Adds reporting helpers around unittest results, including success tracking, console summaries, and CSV export.
- **CN:** 围绕 unittest 结果提供报告工具，包括成功用例跟踪、控制台摘要和 CSV 导出。

## Line-by-Line Analysis / 逐行分析

### Lines 32-35
```python
import csv
import datetime
import os
import unittest
```
**EN:** This opening block establishes the module context. It imports the libraries needed for code generation, testing, and tensor/runtime support.
**CN:** 这一开头代码块建立了模块的上下文。 它导入了代码生成、测试以及张量/运行时支持所需的库。

### Lines 38-40
```python
class TestResultWithSuccesses(unittest.TextTestResult):
    """Custom TestResult that tracks successful tests."""
    
```
**EN:** `TestResultWithSuccesses` is the main class introduced in this block. It inherits from `unittest.TextTestResult`, so the class reuses shared helpers or unittest behavior.
**CN:** `TestResultWithSuccesses` 是此处引入的主要类。 它继承自 `unittest.TextTestResult`，因此会复用共享辅助逻辑或 unittest 行为。

### Lines 41-43
```python
    def __init__(self, stream, descriptions, verbosity):
        super().__init__(stream, descriptions, verbosity)
        self.successes = []
```
**EN:** `__init__(self, stream, descriptions, verbosity)` is a method inside `TestResultWithSuccesses`. Key steps in this block are `super().__init__(stream, descriptions, verbosity); self.successes = []`.
**CN:** `__init__(self, stream, descriptions, verbosity)` 是一个位于 `TestResultWithSuccesses` 内部的方法。 此代码块中的关键步骤是 `super().__init__(stream, descriptions, verbosity); self.successes = []`。

### Lines 45-47
```python
    def addSuccess(self, test):
        super().addSuccess(test)
        self.successes.append(test)
```
**EN:** `addSuccess(self, test)` is a method inside `TestResultWithSuccesses`. Key steps in this block are `super().addSuccess(test); self.successes.append(test)`.
**CN:** `addSuccess(self, test)` 是一个位于 `TestResultWithSuccesses` 内部的方法。 此代码块中的关键步骤是 `super().addSuccess(test); self.successes.append(test)`。

### Lines 50-80
```python
def print_test_summary(test_results, suite_name):
    """
    Print test results summary.
    
    Args:
        test_results: unittest.TestResult object containing test results
        suite_name: Name of the test suite that was run
    """
    # Check if test_results has successes attribute (custom result class)
    if hasattr(test_results, 'successes'):
        num_passed = len(test_results.successes)
    else:
        # Calculate successful tests (total - failures - errors - skipped)
        num_passed = (test_results.testsRun - len(test_results.failures) - 
                      len(test_results.errors) - len(test_results.skipped))
    
    num_failures = len(test_results.failures)
    num_errors = len(test_results.errors)
    num_skipped = len(test_results.skipped)
    total = test_results.testsRun
    
    print(f"\n{'='*70}")
    print(f"Test Report Summary")
    print(f"{'='*70}")
    print(f"Suite: {suite_name}")
    print(f"Total tests run: {total}")
    print(f"Passed: {num_passed}")
    print(f"Failed: {num_failures}")
    print(f"Errors: {num_errors}")
    print(f"Skipped: {num_skipped}")
    print(f"{'='*70}\n")
```
**EN:** `print_test_summary(test_results, suite_name)` is a top-level function. The docstring describes it as: Print test results summary. Key steps in this block are `num_failures = len(test_results.failures); num_errors = len(test_results.errors); num_skipped = len(test_results.skipped); total = test_results.testsRun`.
**CN:** `print_test_summary(test_results, suite_name)` 是一个顶层函数。 文档字符串将其描述为：Print test results summary. 此代码块中的关键步骤是 `num_failures = len(test_results.failures); num_errors = len(test_results.errors); num_skipped = len(test_results.skipped); total = test_results.testsRun`。

### Lines 83-146
```python
def write_test_results_to_csv(results, suite_name, output_dir='.'):
    """Write test results to CSV file in the specified directory."""
    timestamp = datetime.datetime.now().strftime('%Y%m%d_%H%M%S')
    filename = f'test_results_{suite_name}_{timestamp}.csv'
    filepath = os.path.join(output_dir, filename)
    
    # Ensure output directory exists
    os.makedirs(output_dir, exist_ok=True)
    
    # Collect all test results
    all_tests = []
    
    # Check if test_results has successes attribute (custom result class)
    if hasattr(results, 'successes'):
        # Add successful tests
        for test in results.successes:
            test_name = str(test)
            all_tests.append({
                'test_name': test_name,
                'status': 'PASS',
                'message': ''
            })
    
    # Add failed tests
    for test, traceback in results.failures:
        test_name = str(test)
        # Extract just the error message (first line of traceback)
        message = traceback.split('\n')[-2] if traceback else 'Failed'
        all_tests.append({
            'test_name': test_name,
            'status': 'FAIL',
            'message': message
        })
    
    # Add error tests
    for test, traceback in results.errors:
        test_name = str(test)
        # Extract just the error message (first line of traceback)
        message = traceback.split('\n')[-2] if traceback else 'Error'
        all_tests.append({
            'test_name': test_name,
            'status': 'ERROR',
            'message': message
        })
    
    # Add skipped tests
    for test, reason in results.skipped:
        test_name = str(test)
        all_tests.append({
            'test_name': test_name,
            'status': 'SKIP',
            'message': reason
        })
    
    # Write to CSV
    with open(filepath, 'w', newline='') as csvfile:
        fieldnames = ['test_name', 'status', 'message']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        
        writer.writeheader()
        for test_result in all_tests:
            writer.writerow(test_result)
    
    print(f"\nTest results written to: {filepath}")
```
**EN:** `write_test_results_to_csv(results, suite_name, output_dir)` is a top-level function. The docstring describes it as: Write test results to CSV file in the specified directory. Key steps in this block are `timestamp = datetime.datetime.now().strftime('%Y%m%d_%H%M%S'); filename = f'test_results_{suite_name}_{timestamp}.csv'; filepath = os.path.join(output_dir, filename); os.makedirs(output_dir, exist_ok=True)`. The surrounding control flow iterates over `results.failures` to cover multiple shapes/configurations.
**CN:** `write_test_results_to_csv(results, suite_name, output_dir)` 是一个顶层函数。 文档字符串将其描述为：Write test results to CSV file in the specified directory. 此代码块中的关键步骤是 `timestamp = datetime.datetime.now().strftime('%Y%m%d_%H%M%S'); filename = f'test_results_{suite_name}_{timestamp}.csv'; filepath = os.path.join(output_dir, filename); os.makedirs(output_dir, exist_ok=True)`。 外层控制流会遍历 `results.failures`，以覆盖多种形状或配置。

## Key Concepts / 关键概念
- **EN:** Extending `unittest.TextTestResult` with explicit success tracking.
  **CN:** 通过扩展 `unittest.TextTestResult` 显式跟踪成功用例。
- **EN:** Aggregated pass/fail/error/skip counts.
  **CN:** 汇总通过/失败/错误/跳过计数。
- **EN:** Timestamped CSV export of per-test outcomes.
  **CN:** 按时间戳导出逐测试结果的 CSV。
- **EN:** Reusable reporting helpers for the EVT runner script.
  **CN:** 供 EVT 运行脚本复用的报告工具。

## Dependencies / 依赖关系
- `import csv`
  - **EN:** Supports timestamped CSV report generation.
  - **CN:** 用于生成带时间戳的 CSV 报告。
- `import datetime`
  - **EN:** Supports timestamped CSV report generation.
  - **CN:** 用于生成带时间戳的 CSV 报告。
- `import os`
  - **EN:** Resolves filesystem paths used for discovery or report output.
  - **CN:** 用于解析测试发现或报告输出所需的文件系统路径。
- `import unittest`
  - **EN:** Supplies the Python unit-test framework and result/runner abstractions.
  - **CN:** 提供 Python 单元测试框架以及结果/运行器抽象。
- `utils.test_report` helpers
  - **EN:** Provide suite summaries and CSV persistence for the CLI runner.
  - **CN:** 为命令行运行器提供套件摘要与 CSV 持久化功能。
