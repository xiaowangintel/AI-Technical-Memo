# test_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/constrained/test_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates utils behavior in SGLang's unit / constrained area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / constrained 领域中与 utils 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: supporting statements / 辅助语句
```python
"""
Unit tests for sglang.srt.constrained.utils.

Test Coverage:
- is_legacy_structural_tag: legacy format detection, new format detection,
  missing fields, edge cases with assertion errors.

Usage:
    python -m pytest test_utils.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 12-15: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.constrained.utils import is_legacy_structural_tag
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.constrained.utils`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.constrained.utils`, `sglang.test.ci.ci_register`。

### Lines 17-17: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(1.0, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 20-20: class TestIsLegacyStructuralTag declaration / 类 TestIsLegacyStructuralTag 声明
```python
class TestIsLegacyStructuralTag(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 21-21: supporting statements / 辅助语句
```python
    """Test is_legacy_structural_tag function."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 23-28: test case legacy format returns true / 测试用例 legacy format returns true
```python
    def test_legacy_format_returns_true(self):
        obj = {
            "structures": [{"begin": "<tool>", "end": "</tool>"}],
            "triggers": ["<tool>"],
        }
        self.assertTrue(is_legacy_structural_tag(obj))
```
**EN:** This test exercises `test_legacy_format_returns_true` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_legacy_format_returns_true`。

### Lines 30-32: test case legacy format empty lists / 测试用例 legacy format empty lists
```python
    def test_legacy_format_empty_lists(self):
        obj = {"structures": [], "triggers": []}
        self.assertTrue(is_legacy_structural_tag(obj))
```
**EN:** This test exercises `test_legacy_format_empty_lists` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_legacy_format_empty_lists`。

### Lines 34-36: test case new format returns false / 测试用例 new format returns false
```python
    def test_new_format_returns_false(self):
        obj = {"format": {"type": "json_schema", "schema": {}}}
        self.assertFalse(is_legacy_structural_tag(obj))
```
**EN:** This test exercises `test_new_format_returns_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_new_format_returns_false`。

### Lines 38-40: test case new format empty format / 测试用例 new format empty format
```python
    def test_new_format_empty_format(self):
        obj = {"format": {}}
        self.assertFalse(is_legacy_structural_tag(obj))
```
**EN:** This test exercises `test_new_format_empty_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_new_format_empty_format`。

### Lines 42-46: test case legacy missing triggers raises / 测试用例 legacy missing triggers raises
```python
    def test_legacy_missing_triggers_raises(self):
        """Legacy format requires both 'structures' and 'triggers'."""
        obj = {"structures": [{"begin": "<tool>", "end": "</tool>"}]}
        with self.assertRaises(AssertionError):
            is_legacy_structural_tag(obj)
```
**EN:** Legacy format requires both 'structures' and 'triggers'. This test exercises `test_legacy_missing_triggers_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Legacy format requires both 'structures' and 'triggers'. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_legacy_missing_triggers_raises`。

### Lines 48-52: test case new format missing format raises / 测试用例 new format missing format raises
```python
    def test_new_format_missing_format_raises(self):
        """New format (no 'structures') requires 'format' key."""
        obj = {"other_key": "value"}
        with self.assertRaises(AssertionError):
            is_legacy_structural_tag(obj)
```
**EN:** New format (no 'structures') requires 'format' key. This test exercises `test_new_format_missing_format_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** New format (no 'structures') requires 'format' key. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_new_format_missing_format_raises`。

### Lines 54-56: test case empty dict raises / 测试用例 empty dict raises
```python
    def test_empty_dict_raises(self):
        with self.assertRaises(AssertionError):
            is_legacy_structural_tag({})
```
**EN:** This test exercises `test_empty_dict_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_dict_raises`。

### Lines 58-61: test case structures none uses new format path / 测试用例 structures none uses new format path
```python
    def test_structures_none_uses_new_format_path(self):
        """Explicitly None 'structures' should fall to new format check."""
        obj = {"structures": None, "format": {"type": "json_schema"}}
        self.assertFalse(is_legacy_structural_tag(obj))
```
**EN:** Explicitly None 'structures' should fall to new format check. This test exercises `test_structures_none_uses_new_format_path` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Explicitly None 'structures' should fall to new format check. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_structures_none_uses_new_format_path`。

### Lines 63-70: test case both keys present legacy wins / 测试用例 both keys present legacy wins
```python
    def test_both_keys_present_legacy_wins(self):
        """When both 'structures' and 'format' present, 'structures' takes priority."""
        obj = {
            "structures": [{"begin": "<tool>"}],
            "triggers": ["<tool>"],
            "format": {"type": "json_schema"},
        }
        self.assertTrue(is_legacy_structural_tag(obj))
```
**EN:** When both 'structures' and 'format' present, 'structures' takes priority. This test exercises `test_both_keys_present_legacy_wins` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When both 'structures' and 'format' present, 'structures' takes priority. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_both_keys_present_legacy_wins`。

### Lines 73-74: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestIsLegacyStructuralTag`: Test is_legacy_structural_tag function. / 用于组织相关测试、夹具或辅助方法。
- `TestIsLegacyStructuralTag.test_legacy_format_returns_true`: This test exercises `test_legacy_format_returns_true` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_legacy_format_returns_true`。
- `TestIsLegacyStructuralTag.test_legacy_format_empty_lists`: This test exercises `test_legacy_format_empty_lists` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_legacy_format_empty_lists`。
- `TestIsLegacyStructuralTag.test_new_format_returns_false`: This test exercises `test_new_format_returns_false` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_new_format_returns_false`。
- `TestIsLegacyStructuralTag.test_new_format_empty_format`: This test exercises `test_new_format_empty_format` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_new_format_empty_format`。
- `TestIsLegacyStructuralTag.test_legacy_missing_triggers_raises`: Legacy format requires both 'structures' and 'triggers'. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_legacy_missing_triggers_raises`。
- `TestIsLegacyStructuralTag.test_new_format_missing_format_raises`: New format (no 'structures') requires 'format' key. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_new_format_missing_format_raises`。
- `TestIsLegacyStructuralTag.test_empty_dict_raises`: This test exercises `test_empty_dict_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_dict_raises`。
- `TestIsLegacyStructuralTag.test_structures_none_uses_new_format_path`: Explicitly None 'structures' should fall to new format check. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_structures_none_uses_new_format_path`。
- `TestIsLegacyStructuralTag.test_both_keys_present_legacy_wins`: When both 'structures' and 'format' present, 'structures' takes priority. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_both_keys_present_legacy_wins`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.constrained.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 74
