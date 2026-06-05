# test_mmmu_eval_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/bench/test_mmmu_eval_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mmmu eval utils behavior in SGLang's unit / bench area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / bench 领域中与 mmmu eval utils 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: module imports and dependencies / 模块导入与依赖
```python
import importlib.util
import re
import sys
import types
import unittest
from pathlib import Path
```
**EN:** This block imports the modules needed by the rest of the file, including `importlib.util`, `re`, `sys`, `types`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `importlib.util`, `re`, `sys`, `types`。

### Lines 8-18: CI registration and metadata / CI 注册与元数据
```python
try:
    from sglang.test.ci.ci_register import register_cpu_ci
    from sglang.test.test_utils import CustomTestCase
except ModuleNotFoundError:
    CustomTestCase = unittest.TestCase

    def register_cpu_ci(*args, **kwargs):
        pass


register_cpu_ci(est_time=5, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-48: function load mmmu eval utils / 函数 load mmmu eval utils
```python
def _load_mmmu_eval_utils():
    repo_root = Path(__file__).resolve().parents[4]
    module_path = repo_root / "benchmark" / "mmmu" / "eval_utils.py"
    module_name = "_test_mmmu_eval_utils"

    stub_modules = {
        "data_utils": _build_data_utils_stub(),
        "datasets": _build_datasets_stub(),
        "numpy": _build_numpy_stub(),
        "torch": types.ModuleType("torch"),
        "tqdm": _build_tqdm_stub(),
    }
    previous_modules = {name: sys.modules.get(name) for name in stub_modules}
    sys.modules.update(stub_modules)

    spec = importlib.util.spec_from_file_location(module_name, module_path)
    module = importlib.util.module_from_spec(spec)
    sys.modules[module_name] = module
    try:
        spec.loader.exec_module(module)
    finally:
        for name, previous_module in previous_modules.items():
            if previous_module is None:
                sys.modules.pop(name, None)
            else:
                sys.modules[name] = previous_module

    return module
```
**EN:** This block implements `_load_mmmu_eval_utils` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_load_mmmu_eval_utils`，承担模块行为中的一个聚焦逻辑片段。

### Lines 51-63: function build data utils stub / 函数 build data utils stub
```python
def _build_data_utils_stub():
    module = types.ModuleType("data_utils")
    module.CAT_SHORT2LONG = {}
    module.DOMAIN_CAT2SUB_CAT = {}

    def _unused(*args, **kwargs):
        raise AssertionError("Unexpected data_utils call in MMMU parser unit test")

    module.construct_prompt = _unused
    module.load_yaml = _unused
    module.process_single_sample = _unused
    module.save_json = _unused
    return module
```
**EN:** This block implements `_build_data_utils_stub` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_data_utils_stub`，承担模块行为中的一个聚焦逻辑片段。

### Lines 66-74: function build datasets stub / 函数 build datasets stub
```python
def _build_datasets_stub():
    module = types.ModuleType("datasets")

    def _unused(*args, **kwargs):
        raise AssertionError("Unexpected datasets call in MMMU parser unit test")

    module.concatenate_datasets = _unused
    module.load_dataset = _unused
    return module
```
**EN:** This block implements `_build_datasets_stub` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_datasets_stub`，承担模块行为中的一个聚焦逻辑片段。

### Lines 77-80: function build numpy stub / 函数 build numpy stub
```python
def _build_numpy_stub():
    module = types.ModuleType("numpy")
    module.argmax = lambda values: max(range(len(values)), key=values.__getitem__)
    return module
```
**EN:** This block implements `_build_numpy_stub` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_numpy_stub`，承担模块行为中的一个聚焦逻辑片段。

### Lines 83-86: function build tqdm stub / 函数 build tqdm stub
```python
def _build_tqdm_stub():
    module = types.ModuleType("tqdm")
    module.tqdm = lambda iterable=None, *args, **kwargs: iterable
    return module
```
**EN:** This block implements `_build_tqdm_stub` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_tqdm_stub`，承担模块行为中的一个聚焦逻辑片段。

### Lines 89-89: class TestMMMUEvalUtils declaration / 类 TestMMMUEvalUtils 声明
```python
class TestMMMUEvalUtils(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 90-92: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.eval_utils = _load_mmmu_eval_utils()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 94-100: test case default response answer regex captures multiline response / 测试用例 default response answer regex captures multiline response
```python
    def test_default_response_answer_regex_captures_multiline_response(self):
        response = "Based on the diagram, compare the labeled points.\nAnswer: B"

        answer = re.search(self.eval_utils.EvalArgs.response_answer_regex, response)

        self.assertIsNotNone(answer)
        self.assertEqual(answer.group(1), response)
```
**EN:** This test exercises `test_default_response_answer_regex_captures_multiline_response` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_response_answer_regex_captures_multiline_response`。

### Lines 102-120: test case default regex extraction preserves multiline answer for processing / 测试用例 default regex extraction preserves multiline answer for processing
```python
    def test_default_regex_extraction_preserves_multiline_answer_for_processing(self):
        response = "Based on the diagram, compare the labeled points.\nAnswer: B"
        sample = self._multiple_choice_sample(response)
        answer = re.search(self.eval_utils.EvalArgs.response_answer_regex, response)
        answer_dict = {}
        out_samples = {}
        previous_random_choice = self.eval_utils.random.choice
        self.eval_utils.random.choice = lambda choices: "A"
        try:
            self.eval_utils.process_result(
                answer.group(1).strip() if answer else response,
                sample,
                answer_dict,
                out_samples,
            )
        finally:
            self.eval_utils.random.choice = previous_random_choice

        self.assertEqual(out_samples["sample-1"]["pred_ans"], "B")
```
**EN:** This test exercises `test_default_regex_extraction_preserves_multiline_answer_for_processing` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_regex_extraction_preserves_multiline_answer_for_processing`。

### Lines 122-138: test case parse multi choice prefers explicit answer marker after copied options / 测试用例 parse multi choice prefers explicit answer marker after copied options
```python
    def test_parse_multi_choice_prefers_explicit_answer_marker_after_copied_options(
        self,
    ):
        response = (
            "The options are:\n"
            "(A) red\n"
            "(B) blue\n"
            "(C) green\n"
            "(D) yellow\n"
            "Answer: B"
        )

        pred_ans = self.eval_utils.parse_multi_choice_response(
            response, ["A", "B", "C", "D"], self._index_to_answer()
        )

        self.assertEqual(pred_ans, "B")
```
**EN:** This test exercises `test_parse_multi_choice_prefers_explicit_answer_marker_after_copied_options` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_prefers_explicit_answer_marker_after_copied_options`。

### Lines 140-157: test case parse multi choice prefers final standalone letter after copied options / 测试用例 parse multi choice prefers final standalone letter after copied options
```python
    def test_parse_multi_choice_prefers_final_standalone_letter_after_copied_options(
        self,
    ):
        response = (
            "The options are:\n"
            "(A) red\n"
            "(B) blue\n"
            "(C) green\n"
            "(D) yellow\n"
            "The diagram rules out the other labels.\n"
            "**B**"
        )

        pred_ans = self.eval_utils.parse_multi_choice_response(
            response, ["A", "B", "C", "D"], self._index_to_answer()
        )

        self.assertEqual(pred_ans, "B")
```
**EN:** This test exercises `test_parse_multi_choice_prefers_final_standalone_letter_after_copied_options` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_prefers_final_standalone_letter_after_copied_options`。

### Lines 159-166: test case parse multi choice prefers latest explicit answer / 测试用例 parse multi choice prefers latest explicit answer
```python
    def test_parse_multi_choice_prefers_latest_explicit_answer(self):
        response = "Initial thought: Answer: A\nAfter checking the image again:\n**B**"

        pred_ans = self.eval_utils.parse_multi_choice_response(
            response, ["A", "B", "C", "D"], self._index_to_answer()
        )

        self.assertEqual(pred_ans, "B")
```
**EN:** This test exercises `test_parse_multi_choice_prefers_latest_explicit_answer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_prefers_latest_explicit_answer`。

### Lines 168-175: test case parse multi choice extracts boxed answer / 测试用例 parse multi choice extracts boxed answer
```python
    def test_parse_multi_choice_extracts_boxed_answer(self):
        response = "After computing the integral the result lines up with \\boxed{C}."

        pred_ans = self.eval_utils.parse_multi_choice_response(
            response, ["A", "B", "C", "D"], self._index_to_answer()
        )

        self.assertEqual(pred_ans, "C")
```
**EN:** This test exercises `test_parse_multi_choice_extracts_boxed_answer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_extracts_boxed_answer`。

### Lines 177-184: test case parse multi choice extracts the answer is / 测试用例 parse multi choice extracts the answer is
```python
    def test_parse_multi_choice_extracts_the_answer_is(self):
        response = "Reasoning about the diagram, the answer is D."

        pred_ans = self.eval_utils.parse_multi_choice_response(
            response, ["A", "B", "C", "D"], self._index_to_answer()
        )

        self.assertEqual(pred_ans, "D")
```
**EN:** This test exercises `test_parse_multi_choice_extracts_the_answer_is` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_extracts_the_answer_is`。

### Lines 186-193: test case parse multi choice extracts final answer / 测试用例 parse multi choice extracts final answer
```python
    def test_parse_multi_choice_extracts_final_answer(self):
        response = "Working through the steps...\nFinal answer: A"

        pred_ans = self.eval_utils.parse_multi_choice_response(
            response, ["A", "B", "C", "D"], self._index_to_answer()
        )

        self.assertEqual(pred_ans, "A")
```
**EN:** This test exercises `test_parse_multi_choice_extracts_final_answer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_extracts_final_answer`。

### Lines 195-205: test case parse multi choice extracts correct answer phrase / 测试用例 parse multi choice extracts correct answer phrase
```python
    def test_parse_multi_choice_extracts_correct_answer_phrase(self):
        response = (
            "(A) is wrong because the proportions do not match.\n"
            "The correct answer is B."
        )

        pred_ans = self.eval_utils.parse_multi_choice_response(
            response, ["A", "B", "C", "D"], self._index_to_answer()
        )

        self.assertEqual(pred_ans, "B")
```
**EN:** This test exercises `test_parse_multi_choice_extracts_correct_answer_phrase` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_extracts_correct_answer_phrase`。

### Lines 207-227: test case parse multi choice ignores parenthetical option mentions / 测试用例 parse multi choice ignores parenthetical option mentions
```python
    def test_parse_multi_choice_ignores_parenthetical_option_mentions(self):
        # Thinking-style outputs discuss/reject several options inside
        # ``<think>...</think>`` using parenthetical mentions like ``(A)``,
        # ``(B)``.  Those mentions must NOT match any explicit-commit
        # pattern, otherwise the latest-match heuristic would pick up a
        # rejected option from the thinking text.  Only the explicit
        # ``Answer: D`` after ``</think>`` should win.
        response = (
            "<think>\n"
            "Option (A) seems plausible but the diagram shows otherwise.\n"
            "Maybe (B) given the labels — actually no, (B) is contradicted "
            "by the second figure. Let me reconsider; the data points to D.\n"
            "</think>\n"
            "Answer: D"
        )

        pred_ans = self.eval_utils.parse_multi_choice_response(
            response, ["A", "B", "C", "D"], self._index_to_answer()
        )

        self.assertEqual(pred_ans, "D")
```
**EN:** This test exercises `test_parse_multi_choice_ignores_parenthetical_option_mentions` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_ignores_parenthetical_option_mentions`。

### Lines 229-237: method multiple choice sample / 方法 multiple choice sample
```python
    def _multiple_choice_sample(self, response):
        return {
            "id": "sample-1",
            "question_type": "multiple-choice",
            "all_choices": ["A", "B", "C", "D"],
            "index2ans": self._index_to_answer(),
            "answer": "B",
            "original_response": response,
        }
```
**EN:** This block implements `_multiple_choice_sample` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_multiple_choice_sample`，承担模块行为中的一个聚焦逻辑片段。

### Lines 239-240: method index to answer / 方法 index to answer
```python
    def _index_to_answer(self):
        return {"A": "red", "B": "blue", "C": "green", "D": "yellow"}
```
**EN:** This block implements `_index_to_answer` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_index_to_answer`，承担模块行为中的一个聚焦逻辑片段。

### Lines 243-244: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_load_mmmu_eval_utils`: This block implements `_load_mmmu_eval_utils` and captures one focused piece of the module's behavior. / 该代码块实现 `_load_mmmu_eval_utils`，承担模块行为中的一个聚焦逻辑片段。
- `_build_data_utils_stub`: This block implements `_build_data_utils_stub` and captures one focused piece of the module's behavior. / 该代码块实现 `_build_data_utils_stub`，承担模块行为中的一个聚焦逻辑片段。
- `_build_datasets_stub`: This block implements `_build_datasets_stub` and captures one focused piece of the module's behavior. / 该代码块实现 `_build_datasets_stub`，承担模块行为中的一个聚焦逻辑片段。
- `_build_numpy_stub`: This block implements `_build_numpy_stub` and captures one focused piece of the module's behavior. / 该代码块实现 `_build_numpy_stub`，承担模块行为中的一个聚焦逻辑片段。
- `_build_tqdm_stub`: This block implements `_build_tqdm_stub` and captures one focused piece of the module's behavior. / 该代码块实现 `_build_tqdm_stub`，承担模块行为中的一个聚焦逻辑片段。
- `TestMMMUEvalUtils`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMMMUEvalUtils.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestMMMUEvalUtils.test_default_response_answer_regex_captures_multiline_response`: This test exercises `test_default_response_answer_regex_captures_multiline_response` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_response_answer_regex_captures_multiline_response`。
- `TestMMMUEvalUtils.test_default_regex_extraction_preserves_multiline_answer_for_processing`: This test exercises `test_default_regex_extraction_preserves_multiline_answer_for_processing` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_regex_extraction_preserves_multiline_answer_for_processing`。
- `TestMMMUEvalUtils.test_parse_multi_choice_prefers_explicit_answer_marker_after_copied_options`: This test exercises `test_parse_multi_choice_prefers_explicit_answer_marker_after_copied_options` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_prefers_explicit_answer_marker_after_copied_options`。
- `TestMMMUEvalUtils.test_parse_multi_choice_prefers_final_standalone_letter_after_copied_options`: This test exercises `test_parse_multi_choice_prefers_final_standalone_letter_after_copied_options` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_prefers_final_standalone_letter_after_copied_options`。
- `TestMMMUEvalUtils.test_parse_multi_choice_prefers_latest_explicit_answer`: This test exercises `test_parse_multi_choice_prefers_latest_explicit_answer` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_multi_choice_prefers_latest_explicit_answer`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `importlib.util`, `re`, `sys`, `types`, `unittest`, `pathlib`

- **Total lines / 总行数**: 244
