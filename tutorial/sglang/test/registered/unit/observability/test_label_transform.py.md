# test_label_transform.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/observability/test_label_transform.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates label transform behavior in SGLang's unit / observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / observability 领域中与 label transform 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for label_transform — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-3: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 5-5: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=6, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 7-12: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.observability.label_transform import (
    UNKNOWN_PRIORITY_VALUE,
    transform_priority,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.observability.label_transform`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.observability.label_transform`。

### Lines 15-15: class TestTransformPriority declaration / 类 TestTransformPriority 声明
```python
class TestTransformPriority(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 16-16: supporting statements / 辅助语句
```python
    """Test cases for transform_priority."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 18-20: test case none returns unknown / 测试用例 none returns unknown
```python
    def test_none_returns_unknown(self):
        """None priority returns UNKNOWN."""
        self.assertEqual(transform_priority(None), UNKNOWN_PRIORITY_VALUE)
```
**EN:** None priority returns UNKNOWN. This test exercises `test_none_returns_unknown` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** None priority returns UNKNOWN. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_returns_unknown`。

### Lines 22-24: test case negative returns low / 测试用例 negative returns low
```python
    def test_negative_returns_low(self):
        """Priority below minimum returns LOW."""
        self.assertEqual(transform_priority(-1), "LOW")
```
**EN:** Priority below minimum returns LOW. This test exercises `test_negative_returns_low` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Priority below minimum returns LOW. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_returns_low`。

### Lines 26-29: test case above max returns high / 测试用例 above max returns high
```python
    def test_above_max_returns_high(self):
        """Priority at or above max returns HIGH."""
        self.assertEqual(transform_priority(31), "HIGH")
        self.assertEqual(transform_priority(100), "HIGH")
```
**EN:** Priority at or above max returns HIGH. This test exercises `test_above_max_returns_high` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Priority at or above max returns HIGH. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_above_max_returns_high`。

### Lines 31-35: test case in range returns string / 测试用例 in range returns string
```python
    def test_in_range_returns_string(self):
        """Priority in valid range [0, 31) returns its string representation."""
        self.assertEqual(transform_priority(0), "0")
        self.assertEqual(transform_priority(15), "15")
        self.assertEqual(transform_priority(30), "30")
```
**EN:** Priority in valid range [0, 31) returns its string representation. This test exercises `test_in_range_returns_string` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Priority in valid range [0, 31) returns its string representation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_in_range_returns_string`。

### Lines 38-39: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestTransformPriority`: Test cases for transform_priority. / 用于组织相关测试、夹具或辅助方法。
- `TestTransformPriority.test_none_returns_unknown`: None priority returns UNKNOWN. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_returns_unknown`。
- `TestTransformPriority.test_negative_returns_low`: Priority below minimum returns LOW. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_returns_low`。
- `TestTransformPriority.test_above_max_returns_high`: Priority at or above max returns HIGH. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_above_max_returns_high`。
- `TestTransformPriority.test_in_range_returns_string`: Priority in valid range [0, 31) returns its string representation. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_in_range_returns_string`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.observability.label_transform`

- **Total lines / 总行数**: 39
