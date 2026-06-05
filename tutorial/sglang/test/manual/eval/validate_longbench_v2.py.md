# validate_longbench_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/eval/validate_longbench_v2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module supports the `validate longbench v2` workflow in `test/manual/eval`. It organizes scenario-specific logic used by SGLang's manual validation suite. / 该 Python 模块支撑 `test/manual/eval` 中的 `validate longbench v2` 流程。它组织了 SGLang 手动验证套件所需的场景化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Scenario logic / 场景逻辑
```python
#!/usr/bin/env python3
"""
Validation script for LongBench-v2 implementation.
This script validates our implementation against official LongBench-v2 format and benchmarks.
"""

import json
import os
import tempfile
from typing import Any, Dict, List

from sglang.test.simple_eval_longbench_v2 import (
    LongBenchV2Eval,
    extract_longbench_v2_answer,
    format_longbench_v2_question,
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 19-43: Helper routines around create_sample_official_data / 辅助例程
```python
def create_sample_official_data() -> List[Dict[str, Any]]:
    """Create sample data in official LongBench-v2 format for validation."""
    return [
        {
            "_id": "test_001",
            "domain": "science",
            "sub_domain": "physics",
            "difficulty": "hard",
            "length": "medium",
            "question": "What is the fundamental force responsible for holding atomic nuclei together?",
            "choice_A": "Electromagnetic force",
            "choice_B": "Strong nuclear force",
            "choice_C": "Weak nuclear force",
            "choice_D": "Gravitational force",
            "answer": "B",
            "context": "Nuclear physics studies the components and behavior of atomic nuclei. "
            * 100,
        },
        {
            "_id": "test_002",
            "domain": "literature",
            "sub_domain": "analysis",
            "difficulty": "hard",
            "length": "long",
            "question": "What literary technique is primarily used in the given passage?",
```
**EN:** This range implements helper routine(s) `create_sample_official_data` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 44-67: Scenario logic / 场景逻辑
```python
            "choice_A": "Metaphor",
            "choice_B": "Alliteration",
            "choice_C": "Symbolism",
            "choice_D": "Irony",
            "answer": "C",
            "context": "Literary analysis involves examining various techniques authors use to convey meaning. "
            * 150,
        },
        {
            "_id": "test_003",
            "domain": "code",
            "sub_domain": "algorithms",
            "difficulty": "easy",
            "length": "short",
            "question": "What is the time complexity of binary search?",
            "choice_A": "O(n)",
            "choice_B": "O(log n)",
            "choice_C": "O(n²)",
            "choice_D": "O(1)",
            "answer": "B",
            "context": "Binary search is a fundamental algorithm in computer science. "
            * 50,
        },
    ]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `O`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 70-89: Helper routines around create_alternative_format_data / 辅助例程
```python
def create_alternative_format_data() -> List[Dict[str, Any]]:
    """Create sample data in alternative format (choices as list) for validation."""
    return [
        {
            "_id": "alt_001",
            "question": "What is 2 + 2?",
            "choices": ["3", "4", "5", "6"],
            "answer": "B",
            "category": "single_document_qa",
            "context": "Basic arithmetic operations. " * 30,
        },
        {
            "_id": "alt_002",
            "question": "What color is the sky?",
            "choices": ["Red", "Blue", "Green", "Yellow"],
            "answer": "B",
            "category": "multi_document_qa",
            "context": "Color perception and atmospheric science. " * 40,
        },
    ]
```
**EN:** This range implements helper routine(s) `create_alternative_format_data` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `format`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 92-94: Class definition for MockSampler / 类定义
```python
class MockSampler:
    """Mock sampler for testing that returns predictable responses."""
```
**EN:** This range declares `MockSampler`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 95-97: Helper routines around __init__ / 辅助例程
```python
    def __init__(self, responses: Dict[str, str]):
        self.responses = responses
        self.call_count = 0
```
**EN:** This range implements helper routine(s) `__init__` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 99-100: Helper routines around _pack_message / 辅助例程
```python
    def _pack_message(self, content: str, role: str) -> Dict[str, str]:
        return {"content": content, "role": role}
```
**EN:** This range implements helper routine(s) `_pack_message` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 102-116: Helper routines around __call__ / 辅助例程
```python
    def __call__(self, messages: List[Dict[str, str]]) -> str:
        """Return a mock response based on the question content."""
        prompt = messages[0]["content"]
        self.call_count += 1

        if "atomic nuclei" in prompt:
            return "The correct answer is (B)"
        if "literary technique" in prompt:
            return "The correct answer is (C)"
        if "binary search" in prompt:
            return "The correct answer is (B)"
        if "2 + 2" in prompt:
            return "The correct answer is (B)"
        if "color is the sky" in prompt:
            return "The correct answer is (B)"
```
**EN:** This range implements helper routine(s) `__call__` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `is`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 117-119: Scenario logic / 场景逻辑
```python
        if "Complex reasoning question" in prompt:
            return "The correct answer is (B)"
        return "The correct answer is (A)"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `is`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 122-146: Test routines around test_format_compatibility / 测试例程
```python
def test_format_compatibility() -> None:
    """Test that our implementation handles official LongBench-v2 format correctly."""
    print("Testing official format compatibility...")

    official_sample = {
        "context": "Test context",
        "question": "Test question?",
        "choice_A": "Option A",
        "choice_B": "Option B",
        "choice_C": "Option C",
        "choice_D": "Option D",
        "answer": "A",
    }

    formatted = format_longbench_v2_question(official_sample)
    assert "Test context" in formatted
    assert "Test question?" in formatted
    assert "(A) Option A" in formatted
    assert "(B) Option B" in formatted
    assert "The correct answer is" in formatted
    print("✓ Official format compatibility verified")

    alt_sample = {
        "context": "Test context",
        "question": "Test question?",
```
**EN:** This range defines concrete test routine(s) `test_format_compatibility`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `format_longbench_v2_question`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 147-154: Assertions and result checks / 断言与结果检查
```python
        "choices": ["Option A", "Option B", "Option C", "Option D"],
        "answer": "A",
    }

    formatted_alt = format_longbench_v2_question(alt_sample)
    assert "Test context" in formatted_alt
    assert "(A) Option A" in formatted_alt
    print("✓ Alternative format compatibility verified")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `format_longbench_v2_question`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 157-176: Test routines around test_answer_extraction / 测试例程
```python
def test_answer_extraction() -> None:
    """Test answer extraction with various response formats."""
    print("Testing answer extraction...")

    test_cases = [
        ("The correct answer is (B)", "B"),
        ("The correct answer is C", "C"),
        ("After analysis, The correct answer is (D)", "D"),
        ("*The correct answer is (A)*", "A"),
        ("I think the answer is B", "B"),
        ("No clear answer here", None),
    ]

    for response, expected in test_cases:
        result = extract_longbench_v2_answer(response)
        assert (
            result == expected
        ), f"Failed for '{response}': got {result}, expected {expected}"

    print("✓ Answer extraction verified")
```
**EN:** This range defines concrete test routine(s) `test_answer_extraction`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `is`, `extract_longbench_v2_answer` and `assert`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 179-201: Test routines around test_evaluation_pipeline / 测试例程
```python
def test_evaluation_pipeline() -> None:
    """Test the complete evaluation pipeline with mock data."""
    print("Testing evaluation pipeline...")

    official_data = create_sample_official_data()

    with tempfile.NamedTemporaryFile(mode="w", suffix=".json", delete=False) as f:
        json.dump(official_data, f)
        temp_file = f.name

    try:
        eval_obj = LongBenchV2Eval(data_source=temp_file, num_examples=3, num_threads=1)
        mock_sampler = MockSampler({})
        result = eval_obj(mock_sampler)

        assert result.score > 0, "Expected positive score"
        assert len(result.convos) == 3, "Expected 3 evaluated conversations"
        assert "chars" in result.metrics, "Expected chars metric"

        print(f"✓ Evaluation pipeline verified (score: {result.score:.3f})")

    finally:
        os.unlink(temp_file)
```
**EN:** This range defines concrete test routine(s) `test_evaluation_pipeline`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. Assertions in this block enforce the intended outcome. Representative call sites include `create_sample_official_data`, `NamedTemporaryFile`, `dump` and `LongBenchV2Eval`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 204-227: Test routines around test_category_filtering / 测试例程
```python
def test_category_filtering() -> None:
    """Test category-based filtering functionality."""
    print("Testing category filtering...")

    alt_data = create_alternative_format_data()

    with tempfile.NamedTemporaryFile(mode="w", suffix=".json", delete=False) as f:
        json.dump(alt_data, f)
        temp_file = f.name

    try:
        eval_obj = LongBenchV2Eval(
            data_source=temp_file,
            categories=["single_document_qa"],
            num_threads=1,
        )

        assert len(eval_obj.examples) == 1, "Expected 1 example after filtering"
        assert eval_obj.examples[0]["category"] == "single_document_qa"

        print("✓ Category filtering verified")

    finally:
        os.unlink(temp_file)
```
**EN:** This range defines concrete test routine(s) `test_category_filtering`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `create_alternative_format_data`, `NamedTemporaryFile`, `dump` and `LongBenchV2Eval`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 230-254: Helper routines around run_accuracy_benchmark / 辅助例程
```python
def run_accuracy_benchmark() -> None:
    """Run a small accuracy benchmark to compare with expected performance."""
    print("Running accuracy benchmark...")

    benchmark_data = [
        {
            "_id": "bench_001",
            "question": "Complex reasoning question",
            "choice_A": "Incorrect option 1",
            "choice_B": "Correct answer",
            "choice_C": "Incorrect option 2",
            "choice_D": "Incorrect option 3",
            "answer": "B",
            "context": "This requires careful analysis. " * 200,
        }
    ] * 10

    with tempfile.NamedTemporaryFile(mode="w", suffix=".json", delete=False) as f:
        json.dump(benchmark_data, f)
        temp_file = f.name

    try:
        eval_obj = LongBenchV2Eval(data_source=temp_file, num_threads=1)
        perfect_sampler = MockSampler({})
        result = eval_obj(perfect_sampler)
```
**EN:** This range implements helper routine(s) `run_accuracy_benchmark` so setup, transformation, or validation logic can be reused cleanly. The code also ties this block to evaluation or benchmark execution. Representative call sites include `NamedTemporaryFile`, `dump`, `LongBenchV2Eval` and `MockSampler`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 255-265: Assertions and result checks / 断言与结果检查
```python

        print(f"✓ Benchmark completed - Perfect sampler accuracy: {result.score:.3f}")
        print(f"  Total examples: {len(result.convos)}")
        print(f"  Average response length: {result.metrics.get('chars', 0):.1f} chars")

        assert (
            result.score == 1.0
        ), f"Perfect sampler should get 100% accuracy, got {result.score:.3f}"

    finally:
        os.unlink(temp_file)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `assert` and `unlink`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 268-292: Helper routines around generate_comparison_report / 辅助例程
```python
def generate_comparison_report() -> None:
    """Generate a comparison report with official benchmarks."""
    print("\n" + "=" * 60)
    print("LONGBENCH-V2 IMPLEMENTATION VALIDATION REPORT")
    print("=" * 60)

    print("\n📊 OFFICIAL BENCHMARK RESULTS (for comparison):")
    print("  • Human Experts: 53.7% accuracy (15-min constraint)")
    print("  • Best Direct Model: 50.1% accuracy")
    print("  • o1-preview (with CoT): 57.7% accuracy")
    print("  • Dataset: 503 questions, 8k-2M word contexts")

    print("\n✅ IMPLEMENTATION VALIDATION:")
    print("  • Format compatibility: VERIFIED")
    print("  • Answer extraction: VERIFIED")
    print("  • Evaluation pipeline: VERIFIED")
    print("  • Category filtering: VERIFIED")
    print("  • Perfect sampler benchmark: VERIFIED (100% accuracy)")

    print("\n🔍 TECHNICAL VERIFICATION:")
    print("  • Handles official choice_A/B/C/D format: ✓")
    print("  • Handles alternative choices list format: ✓")
    print("  • Official answer extraction patterns: ✓")
    print("  • Context length filtering: ✓")
    print("  • HuggingFace dataset integration: ✓")
```
**EN:** This range implements helper routine(s) `generate_comparison_report` so setup, transformation, or validation logic can be reused cleanly. The code also ties this block to evaluation or benchmark execution. Representative call sites include `RESULTS`, `accuracy`, `preview` and `VERIFIED`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 293-312: Scenario logic / 场景逻辑
```python
    print("  • SGLang evaluation framework compliance: ✓")

    print("\n📈 EXPECTED PERFORMANCE RANGE:")
    print("  • Small models (7B): 35-45% accuracy")
    print("  • Medium models (13-30B): 45-55% accuracy")
    print("  • Large models (70B+): 55-65% accuracy")
    print(
        "  • Note: Actual results depend on model capabilities and context length handling"
    )

    print("\n✨ IMPLEMENTATION HIGHLIGHTS:")
    print("  • Follows official LongBench-v2 evaluation methodology")
    print("  • Compatible with SGLang's existing evaluation patterns")
    print("  • Supports multiple data sources (HF, JSON, CSV)")
    print("  • Robust error handling and fallback mechanisms")
    print("  • Comprehensive filtering and configuration options")

    print("\n" + "=" * 60)
    print("VALIDATION COMPLETE - IMPLEMENTATION READY FOR USE")
    print("=" * 60)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `models` and `sources`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 315-333: Helper routines around main / 辅助例程
```python
def main() -> None:
    """Run all validation tests."""
    print("🔍 Starting LongBench-v2 Implementation Validation...\n")

    try:
        test_format_compatibility()
        test_answer_extraction()
        test_evaluation_pipeline()
        test_category_filtering()
        run_accuracy_benchmark()

        generate_comparison_report()

        print("\n🎉 All validation tests passed successfully!")
        print("The LongBench-v2 implementation is working correctly and ready for use.")

    except Exception as exc:  # pragma: no cover - debug helper
        print(f"\n❌ Validation failed: {exc}")
        raise
```
**EN:** This range implements helper routine(s) `main` so setup, transformation, or validation logic can be reused cleanly. The code also ties this block to evaluation or benchmark execution. Representative call sites include `test_format_compatibility`, `test_answer_extraction`, `test_evaluation_pipeline` and `test_category_filtering`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 334-337: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Accuracy evaluation / 精度评测
- Performance benchmarking / 性能基准测试
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `os`, `tempfile`, `typing`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.simple_eval_longbench_v2`
