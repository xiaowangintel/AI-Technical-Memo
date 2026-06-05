# test_metrics_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/observability/test_metrics_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates metrics utils behavior in SGLang's unit / observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / observability 领域中与 metrics utils 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.observability.utils import (
    generate_buckets,
    two_sides_exponential_buckets,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.observability.utils`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.observability.utils`, `sglang.test.ci.ci_register`。

### Lines 9-9: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=6, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 12-12: class TestMetricsUtils declaration / 类 TestMetricsUtils 声明
```python
class TestMetricsUtils(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 13-13: supporting statements / 辅助语句
```python
    """Test cases for metrics utility functions."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 15-32: test case two sides exponential buckets basic / 测试用例 two sides exponential buckets basic
```python
    def test_two_sides_exponential_buckets_basic(self):
        """Test basic functionality of two_sides_exponential_buckets."""
        # Test with simple parameters
        count = 5
        buckets = two_sides_exponential_buckets(middle=10.0, base=2.0, count=count)

        # Should contain the middle value
        self.assertIn(10.0, buckets)

        # Should be sorted
        self.assertEqual(buckets, sorted(buckets))

        # Should have unique values (no duplicates)
        self.assertEqual(len(buckets), len(set(buckets)))

        # Should have reasonable number of buckets (not exactly count due to ceiling and deduplication)
        self.assertGreaterEqual(len(buckets), 3)
        self.assertLessEqual(len(buckets), count + 2)
```
**EN:** Test basic functionality of two_sides_exponential_buckets. This test exercises `test_two_sides_exponential_buckets_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic functionality of two_sides_exponential_buckets. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_two_sides_exponential_buckets_basic`。

### Lines 34-38: test case two sides exponential buckets specific values / 测试用例 two sides exponential buckets specific values
```python
    def test_two_sides_exponential_buckets_specific_values(self):
        """Test specific values for two_sides_exponential_buckets."""
        buckets = two_sides_exponential_buckets(middle=100.0, base=2.0, count=4)
        expected_values = [96.0, 98.0, 100.0, 102.0, 104.0]
        self.assertEqual(buckets, expected_values)
```
**EN:** Test specific values for two_sides_exponential_buckets. This test exercises `test_two_sides_exponential_buckets_specific_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test specific values for two_sides_exponential_buckets. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_two_sides_exponential_buckets_specific_values`。

### Lines 40-49: test case two sides exponential buckets negative values / 测试用例 two sides exponential buckets negative values
```python
    def test_two_sides_exponential_buckets_negative_values(self):
        """Test two_sides_exponential_buckets with values that could go negative."""
        buckets = two_sides_exponential_buckets(middle=5.0, base=3.0, count=4)

        # Should not contain negative values (max(0, middle - distance))
        for bucket in buckets:
            self.assertGreaterEqual(bucket, 0.0)

        # Should contain the middle value
        self.assertIn(5.0, buckets)
```
**EN:** Test two_sides_exponential_buckets with values that could go negative. This test exercises `test_two_sides_exponential_buckets_negative_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test two_sides_exponential_buckets with values that could go negative. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_two_sides_exponential_buckets_negative_values`。

### Lines 51-61: test case two sides exponential buckets edge cases / 测试用例 two sides exponential buckets edge cases
```python
    def test_two_sides_exponential_buckets_edge_cases(self):
        """Test edge cases for two_sides_exponential_buckets."""
        # Count = 1
        buckets = two_sides_exponential_buckets(middle=10.0, base=2.0, count=1)
        self.assertIn(10.0, buckets)

        # Very small middle value
        buckets = two_sides_exponential_buckets(middle=0.1, base=2.0, count=2)
        self.assertIn(0.1, buckets)
        for bucket in buckets:
            self.assertGreaterEqual(bucket, 0.0)
```
**EN:** Test edge cases for two_sides_exponential_buckets. This test exercises `test_two_sides_exponential_buckets_edge_cases` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test edge cases for two_sides_exponential_buckets. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_two_sides_exponential_buckets_edge_cases`。

### Lines 63-77: test case generate buckets default / 测试用例 generate buckets default
```python
    def test_generate_buckets_default(self):
        """Test generate_buckets with default rule."""
        default_buckets = [1.0, 5.0, 10.0, 50.0, 100.0]

        # Test with "default" rule
        result = generate_buckets(["default"], default_buckets)
        self.assertEqual(result, default_buckets)

        # Test with None (should default to "default")
        result = generate_buckets(None, default_buckets)
        self.assertEqual(result, default_buckets)

        # Test with empty (should default to "default")
        result = generate_buckets(None, default_buckets)
        self.assertEqual(result, default_buckets)
```
**EN:** Test generate_buckets with default rule. This test exercises `test_generate_buckets_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test generate_buckets with default rule. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_buckets_default`。

### Lines 79-88: test case generate buckets tse / 测试用例 generate buckets tse
```python
    def test_generate_buckets_tse(self):
        """Test generate_buckets with tse (two sides exponential) rule."""
        default_buckets = [1.0, 5.0, 10.0]

        # Test with "tse" rule
        result = generate_buckets(["tse", "10", "2.0", "4"], default_buckets)

        # Should return the same as calling two_sides_exponential_buckets directly
        expected = two_sides_exponential_buckets(10.0, 2.0, 4)
        self.assertEqual(result, expected)
```
**EN:** Test generate_buckets with tse (two sides exponential) rule. This test exercises `test_generate_buckets_tse` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test generate_buckets with tse (two sides exponential) rule. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_buckets_tse`。

### Lines 90-99: test case generate buckets custom / 测试用例 generate buckets custom
```python
    def test_generate_buckets_custom(self):
        """Test generate_buckets with custom rule."""
        default_buckets = [1.0, 5.0, 10.0]

        # Test with "custom" rule
        result = generate_buckets(
            ["custom", "1.5", "3.2", "7.8", "15.6"], default_buckets
        )
        expected = [1.5, 3.2, 7.8, 15.6]
        self.assertEqual(result, expected)
```
**EN:** Test generate_buckets with custom rule. This test exercises `test_generate_buckets_custom` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test generate_buckets with custom rule. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_buckets_custom`。

### Lines 101-108: test case generate buckets custom with integers / 测试用例 generate buckets custom with integers
```python
    def test_generate_buckets_custom_with_integers(self):
        """Test generate_buckets with custom rule using integer strings."""
        default_buckets = [1.0, 5.0, 10.0]

        # Test with integer strings
        result = generate_buckets(["custom", "1", "5", "10", "50"], default_buckets)
        expected = [1.0, 5.0, 10.0, 50.0]
        self.assertEqual(result, expected)
```
**EN:** Test generate_buckets with custom rule using integer strings. This test exercises `test_generate_buckets_custom_with_integers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test generate_buckets with custom rule using integer strings. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_buckets_custom_with_integers`。

### Lines 110-128: test case generate buckets preserves order and type / 测试用例 generate buckets preserves order and type
```python
    def test_generate_buckets_preserves_order_and_type(self):
        """Test that generate_buckets preserves order and returns floats."""
        default_buckets = [1, 5, 10, 50, 100]  # integers

        # Test default rule
        result = generate_buckets(["default"], default_buckets)
        self.assertEqual(result, default_buckets)
        self.assertIsInstance(result, list)

        # Test custom rule with proper float conversion
        result = generate_buckets(
            ["custom", "100", "50", "10", "5", "1"], default_buckets
        )
        expected = [1.0, 5.0, 10.0, 50.0, 100.0]
        self.assertEqual(result, expected)

        # All values should be floats
        for value in result:
            self.assertIsInstance(value, float)
```
**EN:** Test that generate_buckets preserves order and returns floats. This test exercises `test_generate_buckets_preserves_order_and_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that generate_buckets preserves order and returns floats. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_buckets_preserves_order_and_type`。

### Lines 130-139: test case integration tse through generate buckets / 测试用例 integration tse through generate buckets
```python
    def test_integration_tse_through_generate_buckets(self):
        """Test integration of TSE buckets through generate_buckets function."""
        default_buckets = [1.0, 10.0, 100.0]

        # Generate buckets using both methods
        direct_result = two_sides_exponential_buckets(50.0, 1.5, 6)
        indirect_result = generate_buckets(["tse", "50.0", "1.5", "6"], default_buckets)

        # Results should be identical
        self.assertEqual(direct_result, indirect_result)
```
**EN:** Test integration of TSE buckets through generate_buckets function. This test exercises `test_integration_tse_through_generate_buckets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test integration of TSE buckets through generate_buckets function. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_integration_tse_through_generate_buckets`。

### Lines 142-143: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestMetricsUtils`: Test cases for metrics utility functions. / 用于组织相关测试、夹具或辅助方法。
- `TestMetricsUtils.test_two_sides_exponential_buckets_basic`: Test basic functionality of two_sides_exponential_buckets. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_two_sides_exponential_buckets_basic`。
- `TestMetricsUtils.test_two_sides_exponential_buckets_specific_values`: Test specific values for two_sides_exponential_buckets. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_two_sides_exponential_buckets_specific_values`。
- `TestMetricsUtils.test_two_sides_exponential_buckets_negative_values`: Test two_sides_exponential_buckets with values that could go negative. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_two_sides_exponential_buckets_negative_values`。
- `TestMetricsUtils.test_two_sides_exponential_buckets_edge_cases`: Test edge cases for two_sides_exponential_buckets. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_two_sides_exponential_buckets_edge_cases`。
- `TestMetricsUtils.test_generate_buckets_default`: Test generate_buckets with default rule. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_buckets_default`。
- `TestMetricsUtils.test_generate_buckets_tse`: Test generate_buckets with tse (two sides exponential) rule. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_buckets_tse`。
- `TestMetricsUtils.test_generate_buckets_custom`: Test generate_buckets with custom rule. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_buckets_custom`。
- `TestMetricsUtils.test_generate_buckets_custom_with_integers`: Test generate_buckets with custom rule using integer strings. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_buckets_custom_with_integers`。
- `TestMetricsUtils.test_generate_buckets_preserves_order_and_type`: Test that generate_buckets preserves order and returns floats. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_buckets_preserves_order_and_type`。
- `TestMetricsUtils.test_integration_tse_through_generate_buckets`: Test integration of TSE buckets through generate_buckets function. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_integration_tse_through_generate_buckets`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.observability.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 143
