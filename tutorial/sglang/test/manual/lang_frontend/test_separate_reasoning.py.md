# test_separate_reasoning.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/lang_frontend/test_separate_reasoning.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `separate reasoning` scenario in `test/manual/lang_frontend`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/lang_frontend` 中的 `separate reasoning` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and shared helpers / 导入与共享辅助项
```python
"""
Tests for the separate_reasoning functionality in sglang.

Usage:
python3 -m unittest test/lang/test_separate_reasoning.py
"""

import unittest

from sglang import gen, separate_reasoning
from sglang.lang.ir import SglExprList, SglSeparateReasoning
from sglang.test.test_utils import CustomTestCase
```
**EN:** This range imports `unittest`, `sglang`, `sglang.lang.ir` and `sglang.test.test_utils`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 15-15: Class definition for TestSeparateReasoning / 类定义
```python
class TestSeparateReasoning(CustomTestCase):
```
**EN:** This range declares `TestSeparateReasoning`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 16-32: Test routines around test_separate_reasoning_creation / 测试例程
```python
    def test_separate_reasoning_creation(self):
        """Test that SglSeparateReasoning objects are created correctly."""
        # Test with valid model type and gen expression
        test_gen = gen("test")
        expr = separate_reasoning(test_gen, model_type="deepseek-r1")
        self.assertIsInstance(expr, SglExprList)
        self.assertEqual(len(expr.expr_list), 2)
        self.assertEqual(expr.expr_list[0], test_gen)
        reasoning_expr = expr.expr_list[1]
        self.assertIsInstance(reasoning_expr, SglSeparateReasoning)
        self.assertEqual(reasoning_expr.model_type, "deepseek-r1")
        self.assertEqual(reasoning_expr.name, "test_reasoning_content")

        # Test with another valid model type
        expr = separate_reasoning(test_gen, model_type="qwen3")
        self.assertIsInstance(expr, SglExprList)
        self.assertEqual(expr.expr_list[1].model_type, "qwen3")
```
**EN:** This range defines concrete test routine(s) `test_separate_reasoning_creation`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `gen`, `separate_reasoning`, `assertIsInstance` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 34-39: Test routines around test_separate_reasoning_name_processing / 测试例程
```python
    def test_separate_reasoning_name_processing(self):
        """Test that separate_reasoning correctly processes names."""
        test_gen = gen("test_var")
        expr = separate_reasoning(test_gen, model_type="deepseek-r1")
        reasoning_expr = expr.expr_list[1]
        self.assertEqual(reasoning_expr.name, "test_var_reasoning_content")
```
**EN:** This range defines concrete test routine(s) `test_separate_reasoning_name_processing`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `gen`, `separate_reasoning` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 40-45: Assertions and result checks / 断言与结果检查
```python

        # Test the process_name_for_reasoning method
        self.assertEqual(
            reasoning_expr.process_name_for_reasoning("another_var"),
            "another_var_reasoning_content",
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual` and `process_name_for_reasoning`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 47-55: Test routines around test_separate_reasoning_repr / 测试例程
```python
    def test_separate_reasoning_repr(self):
        """Test the string representation of SglSeparateReasoning."""
        test_gen = gen("test_var")
        expr = separate_reasoning(test_gen, model_type="deepseek-r1")
        reasoning_expr = expr.expr_list[1]
        self.assertEqual(
            repr(reasoning_expr),
            "SeparateReasoning(model_type=deepseek-r1, name=test_var_reasoning_content)",
        )
```
**EN:** This range defines concrete test routine(s) `test_separate_reasoning_repr`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `gen`, `separate_reasoning`, `assertEqual` and `repr`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 57-63: Test routines around test_separate_reasoning_with_invalid_model_type / 测试例程
```python
    def test_separate_reasoning_with_invalid_model_type(self):
        """Test that separate_reasoning accepts any model type during creation."""
        # Create with invalid model type
        test_gen = gen("test")
        expr = separate_reasoning(test_gen, model_type="invalid-model")
        self.assertIsInstance(expr, SglExprList)
        self.assertEqual(expr.expr_list[1].model_type, "invalid-model")
```
**EN:** This range defines concrete test routine(s) `test_separate_reasoning_with_invalid_model_type`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `gen`, `separate_reasoning`, `assertIsInstance` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 64-68: Script entry point / 脚本入口
```python
        # The actual validation happens in the ReasoningParser constructor


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang`, `sglang.lang.ir`, `sglang.test.test_utils`
