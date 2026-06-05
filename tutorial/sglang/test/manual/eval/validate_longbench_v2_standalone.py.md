# validate_longbench_v2_standalone.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/eval/validate_longbench_v2_standalone.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module supports the `validate longbench v2 standalone` workflow in `test/manual/eval`. It organizes scenario-specific logic used by SGLang's manual validation suite. / 该 Python 模块支撑 `test/manual/eval` 中的 `validate longbench v2 standalone` 流程。它组织了 SGLang 手动验证套件所需的场景化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and shared helpers / 导入与共享辅助项
```python
#!/usr/bin/env python3
"""
Standalone validation script for LongBench-v2 implementation.
Tests core functionality without requiring full SGLang dependencies.
"""

import json
import os
import re
import tempfile
from typing import Any, Dict, List, Optional

ANSWER_PATTERN_MULTICHOICE = r"(?i)(?:the\s+)?(?:correct\s+)?(?:answer\s+)?(?:is\s+)?(?:\(?\s*)?([A-D])(?:\s*\)?)"
```
**EN:** This range imports `json`, `os`, `re` and `tempfile`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 16-40: Helper routines around format_longbench_v2_question / 辅助例程
```python
def format_longbench_v2_question(row: Dict[str, Any]) -> str:
    """Format a LongBench-v2 question using the official template."""
    context = row.get("context", "")
    question = row.get("question", "")

    if "choices" in row:
        choices = row["choices"]
        choice_A = choices[0] if len(choices) > 0 else ""
        choice_B = choices[1] if len(choices) > 1 else ""
        choice_C = choices[2] if len(choices) > 2 else ""
        choice_D = choices[3] if len(choices) > 3 else ""
    else:
        choice_A = row.get("choice_A", row.get("A", ""))
        choice_B = row.get("choice_B", row.get("B", ""))
        choice_C = row.get("choice_C", row.get("C", ""))
        choice_D = row.get("choice_D", row.get("D", ""))

    prompt = f"""{context.strip()}

What is the correct answer to this question: {question.strip()}
Choices:
(A) {choice_A.strip()}
(B) {choice_B.strip()}
(C) {choice_C.strip()}
(D) {choice_D.strip()}
```
**EN:** This range implements helper routine(s) `format_longbench_v2_question` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get` and `strip`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 41-44: Scenario logic / 场景逻辑
```python

The correct answer is"""

    return prompt
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 47-63: Helper routines around extract_longbench_v2_answer / 辅助例程
```python
def extract_longbench_v2_answer(response: str) -> Optional[str]:
    """Extract answer from model response using official LongBench-v2 method."""
    response = response.replace("*", "")

    match = re.search(r"The correct answer is \(([A-D])\)", response, re.IGNORECASE)
    if match:
        return match.group(1).upper()

    match = re.search(r"The correct answer is ([A-D])", response, re.IGNORECASE)
    if match:
        return match.group(1).upper()

    match = re.search(ANSWER_PATTERN_MULTICHOICE, response)
    if match:
        return match.group(1).upper()

    return None
```
**EN:** This range implements helper routine(s) `extract_longbench_v2_answer` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `replace`, `search`, `group` and `upper`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 66-90: Helper routines around create_official_format_samples / 辅助例程
```python
def create_official_format_samples() -> List[Dict[str, Any]]:
    """Create test samples in official LongBench-v2 format."""
    return [
        {
            "_id": "official_001",
            "domain": "science",
            "sub_domain": "physics",
            "difficulty": "hard",
            "length": "medium",
            "question": "What force holds atomic nuclei together?",
            "choice_A": "Electromagnetic force",
            "choice_B": "Strong nuclear force",
            "choice_C": "Weak nuclear force",
            "choice_D": "Gravitational force",
            "answer": "B",
            "context": "Nuclear physics studies atomic nuclei behavior." * 50,
        },
        {
            "_id": "official_002",
            "domain": "literature",
            "sub_domain": "analysis",
            "difficulty": "hard",
            "length": "long",
            "question": "What literary device is primarily demonstrated?",
            "choice_A": "Metaphor",
```
**EN:** This range implements helper routine(s) `create_official_format_samples` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 91-98: Scenario logic / 场景逻辑
```python
            "choice_B": "Alliteration",
            "choice_C": "Symbolism",
            "choice_D": "Irony",
            "answer": "C",
            "context": "The recurring image of the white whale represents much more than a literal creature."
            * 80,
        },
    ]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 101-113: Helper routines around create_alternative_format_samples / 辅助例程
```python
def create_alternative_format_samples() -> List[Dict[str, Any]]:
    """Create test samples in alternative format."""
    return [
        {
            "_id": "alt_001",
            "question": "What is 2 + 2?",
            "choices": ["3", "4", "5", "6"],
            "answer": "B",
            "category": "single_document_qa",
            "context": "Basic arithmetic: Addition is a fundamental mathematical operation."
            * 30,
        }
    ]
```
**EN:** This range implements helper routine(s) `create_alternative_format_samples` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 116-134: Test routines around test_format_compatibility / 测试例程
```python
def test_format_compatibility() -> None:
    """Test format compatibility with both official and alternative formats."""
    print("Testing format compatibility...")

    official_sample = create_official_format_samples()[0]
    formatted = format_longbench_v2_question(official_sample)

    assert "Nuclear physics studies" in formatted
    assert "(A) Electromagnetic force" in formatted
    assert "(B) Strong nuclear force" in formatted
    assert "The correct answer is" in formatted
    print("✓ Official format (choice_A/B/C/D) working correctly")

    alt_sample = create_alternative_format_samples()[0]
    formatted_alt = format_longbench_v2_question(alt_sample)

    assert "What is 2 + 2?" in formatted_alt
    assert "(B) 4" in formatted_alt
    print("✓ Alternative format (choices list) working correctly")
```
**EN:** This range defines concrete test routine(s) `test_format_compatibility`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `create_official_format_samples`, `format_longbench_v2_question`, `format` and `create_alternative_format_samples`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 137-158: Test routines around test_answer_extraction / 测试例程
```python
def test_answer_extraction() -> None:
    """Test answer extraction patterns."""
    print("Testing answer extraction...")

    test_cases = [
        ("The correct answer is (B)", "B"),
        ("The correct answer is C", "C"),
        ("After analysis, The correct answer is (D)", "D"),
        ("*The correct answer is (A)*", "A"),
        ("I believe the answer is B", "B"),
        ("Looking at this, A seems correct", "A"),
        ("The answer should be (C)", "C"),
        ("No clear pattern here", None),
    ]

    for response, expected in test_cases:
        result = extract_longbench_v2_answer(response)
        assert (
            result == expected
        ), f"Failed for '{response}': got {result}, expected {expected}"

    print("✓ Answer extraction patterns working correctly")
```
**EN:** This range defines concrete test routine(s) `test_answer_extraction`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `is`, `be`, `extract_longbench_v2_answer` and `assert`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 161-182: Test routines around test_data_loading_simulation / 测试例程
```python
def test_data_loading_simulation() -> None:
    """Simulate data loading and processing."""
    print("Testing data loading simulation...")

    test_data = create_official_format_samples() + create_alternative_format_samples()

    with tempfile.NamedTemporaryFile(mode="w", suffix=".json", delete=False) as f:
        json.dump(test_data, f)
        temp_file = f.name

    try:
        with open(temp_file, "r", encoding="utf-8") as fh:
            loaded_data = json.load(fh)

        assert len(loaded_data) == 3
        assert loaded_data[0]["_id"] == "official_001"
        assert "choices" in loaded_data[2]

        print("✓ JSON data loading working correctly")

    finally:
        os.unlink(temp_file)
```
**EN:** This range defines concrete test routine(s) `test_data_loading_simulation`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `create_official_format_samples`, `create_alternative_format_samples`, `NamedTemporaryFile` and `dump`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 185-208: Helper routines around run_accuracy_simulation / 辅助例程
```python
def run_accuracy_simulation() -> None:
    """Simulate accuracy testing with perfect responses."""
    print("Running accuracy simulation...")

    samples = create_official_format_samples()
    correct_responses = {
        "official_001": "The correct answer is (B)",
        "official_002": "The correct answer is (C)",
    }

    total_score = 0
    for sample in samples:
        formatted = format_longbench_v2_question(sample)
        response = correct_responses[sample["_id"]]
        extracted = extract_longbench_v2_answer(response)
        expected = sample["answer"]
        score = 1.0 if extracted == expected else 0.0
        total_score += score
        print(f"  Question {sample['_id']}: {extracted} == {expected} -> {score}")

    accuracy = total_score / len(samples)
    print(f"✓ Simulation accuracy: {accuracy:.3f} (expected: 1.0)")

    assert accuracy == 1.0, "Perfect simulation should achieve 100% accuracy"
```
**EN:** This range implements helper routine(s) `run_accuracy_simulation` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `create_official_format_samples`, `is`, `format_longbench_v2_question` and `extract_longbench_v2_answer`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 211-235: Helper routines around generate_validation_report / 辅助例程
```python
def generate_validation_report() -> None:
    """Generate comprehensive validation report."""
    print("\n" + "=" * 70)
    print("LONGBENCH-V2 IMPLEMENTATION VALIDATION REPORT")
    print("=" * 70)

    print("\n📚 OFFICIAL LONGBENCH-V2 BENCHMARK:")
    print("  • Dataset: 503 multiple-choice questions")
    print("  • Context length: 8k to 2M words (majority < 128k)")
    print("  • Categories: 6 major task categories")
    print("  • Human expert accuracy: 53.7%")
    print("  • Best direct model: 50.1% accuracy")
    print("  • o1-preview (with CoT): 57.7% accuracy")

    print("\n✅ IMPLEMENTATION VERIFICATION:")
    print("  • Official format compatibility: VERIFIED")
    print("  • Alternative format support: VERIFIED")
    print("  • Answer extraction patterns: VERIFIED")
    print("  • Data loading mechanisms: VERIFIED")
    print("  • Accuracy calculation: VERIFIED")

    print("\n🔧 TECHNICAL COMPLIANCE:")
    print("  • Official question template: ✓")
    print("  • Multiple answer extraction patterns: ✓")
    print("  • HuggingFace dataset integration: ✓")
```
**EN:** This range implements helper routine(s) `generate_validation_report` so setup, transformation, or validation logic can be reused cleanly. The code also ties this block to evaluation or benchmark execution. Representative call sites include `words` and `preview`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 236-260: Scenario logic / 场景逻辑
```python
    print("  • CSV/JSON file support: ✓")
    print("  • Category-based filtering: ✓")
    print("  • Context length filtering: ✓")

    print("\n📊 EXPECTED PERFORMANCE BENCHMARKS:")
    print("  Model Category          | Expected Accuracy")
    print("  ----------------------- | ----------------")
    print("  Small models (7B)       | 35-45%")
    print("  Medium models (13-30B)  | 45-55%")
    print("  Large models (70B+)     | 55-65%")
    print("  Human experts           | 53.7%")
    print("  Advanced reasoning      | 57.7%")

    print("\n🏗️ IMPLEMENTATION FEATURES:")
    print("  • Multiple data source support (HuggingFace, JSON, CSV)")
    print("  • Robust answer extraction with fallback patterns")
    print("  • Category-based evaluation filtering")
    print("  • Context length range filtering")
    print("  • SGLang evaluation framework integration")
    print("  • Comprehensive error handling")

    print("\n📋 FORMAT COMPATIBILITY:")
    print("  • Official format: choice_A, choice_B, choice_C, choice_D")
    print('  • Alternative format: choices = ["A", "B", "C", "D"]')
    print('  • Answer format: "A", "B", "C", or "D"')
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `models` and `support`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 261-280: Scenario logic / 场景逻辑
```python
    print("  • Context field: Long-form text content")

    print("\n🚀 USAGE EXAMPLES:")
    print("  # Command line usage:")
    print("  python -m sglang.test.run_eval --eval-name longbench_v2 --port 30000")
    print("  ")
    print("  # Python API usage:")
    print("  from sglang.test.simple_eval_longbench_v2 import LongBenchV2Eval")
    print("  eval_obj = LongBenchV2Eval(data_source='THUDM/LongBench-v2')")
    print("  result = eval_obj(sampler)")

    print("\n🎯 ACCURACY COMPARISON GUIDANCE:")
    print("  • Run evaluation on a subset for validation")
    print("  • Compare results within expected performance ranges")
    print("  • Verify answer extraction matches official pattern")
    print("  • Confirm handling of long-context inputs")

    print("\n" + "=" * 70)
    print("VALIDATION STATUS: ✅ PASSED - IMPLEMENTATION READY FOR PRODUCTION")
    print("=" * 70)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `LongBenchV2Eval` and `eval_obj`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 283-301: Helper routines around main / 辅助例程
```python
def main() -> bool:
    """Run complete validation suite."""
    print("🔍 LongBench-v2 Implementation Validation Starting...\n")

    try:
        test_format_compatibility()
        test_answer_extraction()
        test_data_loading_simulation()
        run_accuracy_simulation()

        generate_validation_report()

        print("\n🎉 All validation tests completed successfully!")
        print("Implementation is ready for accuracy comparison testing.")
        return True

    except Exception as exc:  # pragma: no cover - debug helper
        print(f"\n❌ Validation failed: {exc}")
        raise
```
**EN:** This range implements helper routine(s) `main` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `test_format_compatibility`, `test_answer_extraction`, `test_data_loading_simulation` and `run_accuracy_simulation`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 302-306: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    success = main()
    raise SystemExit(0 if success else 1)
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main` and `SystemExit`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Accuracy evaluation / 精度评测
- Performance benchmarking / 性能基准测试
- Multimodal inputs / 多模态输入
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `os`, `re`, `tempfile`, `typing`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: None / 无
