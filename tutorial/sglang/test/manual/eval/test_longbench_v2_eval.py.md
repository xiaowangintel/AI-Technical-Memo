# test_longbench_v2_eval.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/eval/test_longbench_v2_eval.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `longbench v2 eval` scenario in `test/manual/eval`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/eval` 中的 `longbench v2 eval` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Scenario logic / 场景逻辑
```python
"""
Test cases for LongBench-v2 evaluation utility.
"""

import json
import os
import tempfile

from sglang.test.simple_eval_longbench_v2 import (
    LongBenchV2Eval,
    extract_longbench_v2_answer,
    format_longbench_v2_question,
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 16-40: Test routines around test_format_longbench_v2_question / 测试例程
```python
def test_format_longbench_v2_question():
    """Test the official LongBench-v2 question formatting."""
    sample_row = {
        "context": "This is a sample context about environmental issues.",
        "question": "What is the main theme?",
        "A": "Technology",
        "B": "Environment",
        "C": "Economics",
        "D": "Politics",
        "answer": "B",
    }

    formatted = format_longbench_v2_question(sample_row)

    # Verify official template structure
    assert "This is a sample context about environmental issues." in formatted
    assert (
        "What is the correct answer to this question: What is the main theme?"
        in formatted
    )
    assert "(A) Technology" in formatted
    assert "(B) Environment" in formatted
    assert "(C) Economics" in formatted
    assert "(D) Politics" in formatted
    assert "The correct answer is" in formatted
```
**EN:** This range defines concrete test routine(s) `test_format_longbench_v2_question`. The logic drives the target scenario and encodes the expected acceptance criteria. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Assertions in this block enforce the intended outcome. Representative call sites include `format_longbench_v2_question` and `assert`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 41-41: Scenario logic / 场景逻辑
```python
    print("✓ Question formatting works correctly")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 44-67: Test routines around test_extract_longbench_v2_answer / 测试例程
```python
def test_extract_longbench_v2_answer():
    """Test the official LongBench-v2 answer extraction."""

    # Test official format: "The correct answer is (A)"
    response1 = "After analyzing the context, The correct answer is (B)."
    assert extract_longbench_v2_answer(response1) == "B"

    # Test alternative format: "The correct answer is A"
    response2 = "Based on the evidence, The correct answer is C."
    assert extract_longbench_v2_answer(response2) == "C"

    # Test with asterisks
    response3 = "*The correct answer is (D)*"
    assert extract_longbench_v2_answer(response3) == "D"

    # Test fallback to standard pattern
    response4 = "I think the answer is A."
    assert extract_longbench_v2_answer(response4) == "A"

    # Test no answer
    response5 = "I'm not sure about this."
    assert extract_longbench_v2_answer(response5) is None

    print("✓ Answer extraction works correctly")
```
**EN:** This range defines concrete test routine(s) `test_extract_longbench_v2_answer`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `is` and `extract_longbench_v2_answer`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 70-94: Test routines around test_longbench_v2_eval_initialization / 测试例程
```python
def test_longbench_v2_eval_initialization():
    """Test LongBench-v2 evaluation class initialization."""

    # Create a temporary JSON file with sample data
    sample_data = [
        {
            "_id": "test_001",
            "domain": "single_document_qa",
            "question": "What is X?",
            "choice_A": "Option A1",
            "choice_B": "Option B1",
            "choice_C": "Option C1",
            "choice_D": "Option D1",
            "answer": "A",
            "context": "Context 1",
        },
        {
            "_id": "test_002",
            "domain": "multi_document_qa",
            "question": "What is Y?",
            "A": "Option A2",
            "B": "Option B2",
            "C": "Option C2",
            "D": "Option D2",
            "answer": "B",
```
**EN:** This range defines concrete test routine(s) `test_longbench_v2_eval_initialization`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 95-116: Assertions and result checks / 断言与结果检查
```python
            "context": "Context 2",
        },
    ]

    with tempfile.NamedTemporaryFile(mode="w", suffix=".json", delete=False) as f:
        json.dump(sample_data, f)
        temp_file = f.name

    try:
        # Test initialization with new data_source parameter
        eval_instance = LongBenchV2Eval(data_source=temp_file, num_examples=1)
        assert len(eval_instance.examples) == 1
        first_example = eval_instance.examples[0]
        assert first_example.get("category") in {
            "single_document_qa",
            "multi_document_qa",
        }
        assert first_example.get("A") in {"Option A1", "Option A2"}
        print("✓ Evaluation class initialization works correctly")

    finally:
        os.unlink(temp_file)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `NamedTemporaryFile`, `dump`, `LongBenchV2Eval` and `get`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 119-143: Test routines around test_category_filtering / 测试例程
```python
def test_category_filtering():
    """Ensure category filtering keeps only requested domains."""

    sample_data = [
        {
            "_id": "test_001",
            "domain": "single_document_qa",
            "question": "What is X?",
            "choice_A": "Option A1",
            "choice_B": "Option B1",
            "choice_C": "Option C1",
            "choice_D": "Option D1",
            "answer": "A",
            "context": "Context 1",
        },
        {
            "_id": "test_002",
            "domain": "multi_document_qa",
            "question": "What is Y?",
            "choice_A": "Option A2",
            "choice_B": "Option B2",
            "choice_C": "Option C2",
            "choice_D": "Option D2",
            "answer": "B",
            "context": "Context 2",
```
**EN:** This range defines concrete test routine(s) `test_category_filtering`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 144-160: Assertions and result checks / 断言与结果检查
```python
        },
    ]

    with tempfile.NamedTemporaryFile(mode="w", suffix=".json", delete=False) as f:
        json.dump(sample_data, f)
        temp_file = f.name

    try:
        eval_instance = LongBenchV2Eval(
            data_source=temp_file,
            categories=["multi_document_qa"],
        )
        assert len(eval_instance.examples) == 1
        assert eval_instance.examples[0]["category"] == "multi_document_qa"
        print("✓ Category filtering works correctly")
    finally:
        os.unlink(temp_file)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `NamedTemporaryFile`, `dump`, `LongBenchV2Eval` and `unlink`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 163-187: Test routines around test_difficulty_metrics / 测试例程
```python
def test_difficulty_metrics():
    """Validate that difficulty-specific metrics are recorded."""

    sample_data = [
        {
            "_id": "easy_001",
            "domain": "single_document_qa",
            "difficulty": "easy",
            "question": "Easy question?",
            "choice_A": "Correct",
            "choice_B": "Wrong",
            "choice_C": "Wrong",
            "choice_D": "Wrong",
            "answer": "A",
            "context": "Easy context",
        },
        {
            "_id": "hard_001",
            "domain": "single_document_qa",
            "difficulty": "hard",
            "question": "Hard question?",
            "choice_A": "Wrong",
            "choice_B": "Correct",
            "choice_C": "Wrong",
            "choice_D": "Wrong",
```
**EN:** This range defines concrete test routine(s) `test_difficulty_metrics`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 188-212: Class definition for FixedSampler / 类定义
```python
            "answer": "B",
            "context": "Hard context",
        },
    ]

    with tempfile.NamedTemporaryFile(mode="w", suffix=".json", delete=False) as f:
        json.dump(sample_data, f)
        temp_file = f.name

    class FixedSampler:  # noqa: D401 - simple helper
        """Mock sampler returning the correct answer based on question text."""

        def _pack_message(self, content: str, role: str):
            return {"content": content, "role": role}

        def __call__(self, messages):
            prompt = messages[0]["content"]
            if "Easy question" in prompt:
                return "The correct answer is (A)"
            return "The correct answer is (B)"

    try:
        eval_instance = LongBenchV2Eval(data_source=temp_file, num_threads=1)
        result = eval_instance(FixedSampler())
```
**EN:** This range declares `FixedSampler`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `NamedTemporaryFile`, `dump`, `is` and `LongBenchV2Eval`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 213-217: Assertions and result checks / 断言与结果检查
```python
        assert result.metrics.get("difficulty_easy") == 1.0
        assert result.metrics.get("difficulty_hard") == 1.0
        print("✓ Difficulty metrics recorded correctly")
    finally:
        os.unlink(temp_file)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get` and `unlink`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 220-234: Helper routines around main / 辅助例程
```python
def main():
    """Run all tests."""
    print("Testing simplified LongBench-v2 evaluation utility...\n")

    test_format_longbench_v2_question()
    test_extract_longbench_v2_answer()
    test_longbench_v2_eval_initialization()
    test_category_filtering()
    test_difficulty_metrics()

    print("\n" + "=" * 50)
    print("✅ ALL TESTS PASSED!")
    print("The simplified implementation follows SGLang patterns")
    print("while maintaining LongBench-v2 compatibility.")
    print("=" * 50)
```
**EN:** This range implements helper routine(s) `main` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `test_format_longbench_v2_question`, `test_extract_longbench_v2_answer`, `test_longbench_v2_eval_initialization` and `test_category_filtering`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 235-238: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Accuracy evaluation / 精度评测
- Environment-aware configuration / 环境感知配置
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `os`, `tempfile`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.simple_eval_longbench_v2`
