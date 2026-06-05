# test_penalty.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/sampling/test_penalty.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates penalty behavior in SGLang's sampling area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 sampling 领域中与 penalty 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: module imports and dependencies / 模块导入与依赖
```python
import json
import random
import re
import unittest
from concurrent.futures import ThreadPoolExecutor

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `random`, `re`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `random`, `re`, `unittest`。

### Lines 19-20: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=53, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=82, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 23-23: class TestPenalty declaration / 类 TestPenalty 声明
```python
class TestPenalty(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 24-32: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 34-36: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 38-63: helper routine run decode / 辅助流程 run decode
```python
    def run_decode(self, sampling_params):
        """Helper method for basic decode tests."""
        return_logprob = True
        top_logprobs_num = 5
        return_text = True
        n = 1

        response = requests.post(
            self.base_url + "/generate",
            json={
                # prompt that is supposed to generate < 32 tokens
                "text": "<|start_header_id|>user<|end_header_id|>\n\nWhat is the answer for 1 + 1 = ?<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n",
                "sampling_params": {
                    "max_new_tokens": 48,
                    "n": n,
                    **sampling_params,
                },
                "return_logprob": return_logprob,
                "top_logprobs_num": top_logprobs_num,
                "return_text_in_logprobs": return_text,
                "logprob_start_len": 0,
            },
        )
        self.assertEqual(response.status_code, 200)
        print(json.dumps(response.json()))
        print("=" * 100)
```
**EN:** Helper method for basic decode tests. This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Helper method for basic decode tests. 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 65-87: helper routine run generate with prompt / 辅助流程 run generate with prompt
```python
    def run_generate_with_prompt(
        self, prompt, sampling_params, max_tokens=100, seed=None
    ):
        """Helper method to generate text with a specific prompt and parameters."""
        sampling_params = sampling_params.copy()
        sampling_params.setdefault("temperature", 0.05)
        sampling_params.setdefault("top_p", 1.0)
        if seed is not None:
            sampling_params["seed"] = seed

        response = requests.post(
            self.base_url + "/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": prompt}],
                "max_tokens": max_tokens,
                **sampling_params,
            },
        )
        self.assertEqual(response.status_code, 200)
        result = response.json()
        content = result["choices"][0]["message"]["content"]
        return content
```
**EN:** Helper method to generate text with a specific prompt and parameters. This helper encapsulates `run_generate_with_prompt` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Helper method to generate text with a specific prompt and parameters. 该辅助函数封装了 `run_generate_with_prompt`，以便周围测试复用准备、执行或校验逻辑。

### Lines 89-97: method get vocab diversity / 方法 get vocab diversity
```python
    def _get_vocab_diversity(self, text):
        """Calculate vocabulary diversity as unique_words / total_words.

        Higher values mean more diverse (less repetitive) text.
        """
        words = re.findall(r"\b\w+\b", text.lower())
        if not words:
            return 1.0
        return len(set(words)) / len(words)
```
**EN:** Calculate vocabulary diversity as unique_words / total_words. This block implements `_get_vocab_diversity` and captures one focused piece of the module's behavior.
**CN:** Calculate vocabulary diversity as unique_words / total_words. 该代码块实现 `_get_vocab_diversity`，承担模块行为中的一个聚焦逻辑片段。

### Lines 99-155: method test penalty effect / 方法 test penalty effect
```python
    def _test_penalty_effect(
        self,
        prompt,
        baseline_params,
        penalty_params,
        expected_reduction=True,
        max_tokens=150,
    ):
        """Generic test for penalty effects using vocabulary diversity.

        Measures unique_words/total_words ratio instead of counting a specific
        word, because penalties affect ALL token probabilities — the model may
        avoid some repeated tokens while using others more.
        """
        # Use higher temperature so penalties can actually affect token selection.
        # The default temperature (0.05) is near-greedy, making penalty adjustments
        # to logits ineffective since the top token still dominates.
        baseline_params = baseline_params.copy()
        penalty_params = penalty_params.copy()
        baseline_params.setdefault("temperature", 0.8)
        penalty_params.setdefault("temperature", 0.8)

        # Run multiple iterations to get more reliable results
        # Use fixed seeds for deterministic behavior
        base_seed = 42
        baseline_diversities = []
        penalty_diversities = []

        for i in range(5):
            seed = base_seed + i
            baseline_output = self.run_generate_with_prompt(
                prompt, baseline_params, max_tokens, seed=seed
            )
            penalty_output = self.run_generate_with_prompt(
                prompt, penalty_params, max_tokens, seed=seed
            )

            baseline_diversities.append(self._get_vocab_diversity(baseline_output))
            penalty_diversities.append(self._get_vocab_diversity(penalty_output))

        avg_baseline = sum(baseline_diversities) / len(baseline_diversities)
        avg_penalty = sum(penalty_diversities) / len(penalty_diversities)

        if expected_reduction:
            # Penalty should increase vocabulary diversity (less repetition)
            self.assertGreater(
                avg_penalty,
                avg_baseline,
                f"Penalty should increase vocab diversity: {avg_baseline:.3f} → {avg_penalty:.3f}",
            )
        else:
            # Negative penalty should decrease diversity (more repetition)
            self.assertLess(
                avg_penalty,
                avg_baseline,
                f"Negative penalty should decrease vocab diversity: {avg_baseline:.3f} → {avg_penalty:.3f}",
            )
```
**EN:** Generic test for penalty effects using vocabulary diversity. This block implements `_test_penalty_effect` and captures one focused piece of the module's behavior.
**CN:** Generic test for penalty effects using vocabulary diversity. 该代码块实现 `_test_penalty_effect`，承担模块行为中的一个聚焦逻辑片段。

### Lines 157-158: test case default values / 测试用例 default values
```python
    def test_default_values(self):
        self.run_decode({})
```
**EN:** This test exercises `test_default_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_values`。

### Lines 160-161: test case frequency penalty / 测试用例 frequency penalty
```python
    def test_frequency_penalty(self):
        self.run_decode({"frequency_penalty": 2})
```
**EN:** This test exercises `test_frequency_penalty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_frequency_penalty`。

### Lines 163-164: test case min new tokens / 测试用例 min new tokens
```python
    def test_min_new_tokens(self):
        self.run_decode({"min_new_tokens": 16})
```
**EN:** This test exercises `test_min_new_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_min_new_tokens`。

### Lines 166-167: test case presence penalty / 测试用例 presence penalty
```python
    def test_presence_penalty(self):
        self.run_decode({"presence_penalty": 2})
```
**EN:** This test exercises `test_presence_penalty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_presence_penalty`。

### Lines 169-188: test case penalty mixed / 测试用例 penalty mixed
```python
    def test_penalty_mixed(self):
        args = [
            {},
            {},
            {},
            {"frequency_penalty": 2},
            {"presence_penalty": 1},
            {"min_new_tokens": 16},
            {"frequency_penalty": 0.2},
            {"presence_penalty": 0.4},
            {"min_new_tokens": 8},
            {"frequency_penalty": 0.4, "presence_penalty": 0.8},
            {"frequency_penalty": 0.4, "min_new_tokens": 12},
            {"presence_penalty": 0.8, "min_new_tokens": 12},
            {"presence_penalty": -0.3, "frequency_penalty": 1.3, "min_new_tokens": 32},
            {"presence_penalty": 0.3, "frequency_penalty": -1.3, "min_new_tokens": 32},
        ]
        random.shuffle(args * 5)
        with ThreadPoolExecutor(8) as executor:
            list(executor.map(self.run_decode, args))
```
**EN:** This test exercises `test_penalty_mixed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_penalty_mixed`。

### Lines 190-195: test case frequency penalty reduces word repetition / 测试用例 frequency penalty reduces word repetition
```python
    def test_frequency_penalty_reduces_word_repetition(self):
        """Test that frequency penalty increases vocabulary diversity."""
        prompt = "Write exactly 10 very small sentences, each containing the word 'data'. Use the word 'data' as much as possible."
        baseline_params = {"frequency_penalty": 0.0, "repetition_penalty": 1.0}
        penalty_params = {"frequency_penalty": 1.99, "repetition_penalty": 1.0}
        self._test_penalty_effect(prompt, baseline_params, penalty_params)
```
**EN:** Test that frequency penalty increases vocabulary diversity. This test exercises `test_frequency_penalty_reduces_word_repetition` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that frequency penalty increases vocabulary diversity. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_frequency_penalty_reduces_word_repetition`。

### Lines 197-202: test case presence penalty reduces topic repetition / 测试用例 presence penalty reduces topic repetition
```python
    def test_presence_penalty_reduces_topic_repetition(self):
        """Test that presence penalty increases vocabulary diversity."""
        prompt = "Write the word 'machine learning' exactly 20 times in a row, separated by spaces."
        baseline_params = {"presence_penalty": 0.0, "repetition_penalty": 1.0}
        penalty_params = {"presence_penalty": 1.99, "repetition_penalty": 1.0}
        self._test_penalty_effect(prompt, baseline_params, penalty_params)
```
**EN:** Test that presence penalty increases vocabulary diversity. This test exercises `test_presence_penalty_reduces_topic_repetition` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that presence penalty increases vocabulary diversity. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_presence_penalty_reduces_topic_repetition`。

### Lines 204-217: test case combined penalties reduce repetition / 测试用例 combined penalties reduce repetition
```python
    def test_combined_penalties_reduce_repetition(self):
        """Test that combined penalties increase vocabulary diversity."""
        prompt = "Write exactly 10 short sentences, each containing the word 'data'. Use the word 'data' as much as possible."
        baseline_params = {
            "frequency_penalty": 0.0,
            "presence_penalty": 0.0,
            "repetition_penalty": 1.0,
        }
        penalty_params = {
            "frequency_penalty": 1.99,
            "presence_penalty": 1.99,
            "repetition_penalty": 1.99,
        }
        self._test_penalty_effect(prompt, baseline_params, penalty_params)
```
**EN:** Test that combined penalties increase vocabulary diversity. This test exercises `test_combined_penalties_reduce_repetition` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that combined penalties increase vocabulary diversity. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_combined_penalties_reduce_repetition`。

### Lines 219-237: test case penalty edge cases negative penalty values / 测试用例 penalty edge cases negative penalty values
```python
    def test_penalty_edge_cases_negative_penalty_values(self):
        """Test that negative penalties decrease vocabulary diversity."""
        prompt = "Write the word 'test' exactly 15 times in a row, separated by spaces."
        baseline_params = {
            "frequency_penalty": 0.0,
            "presence_penalty": 0.0,
            "repetition_penalty": 1.0,
        }
        negative_penalty_params = {
            "frequency_penalty": -0.5,
            "presence_penalty": -0.25,
            "repetition_penalty": 1.0,
        }
        self._test_penalty_effect(
            prompt,
            baseline_params,
            negative_penalty_params,
            expected_reduction=False,
        )
```
**EN:** Test that negative penalties decrease vocabulary diversity. This test exercises `test_penalty_edge_cases_negative_penalty_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that negative penalties decrease vocabulary diversity. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_penalty_edge_cases_negative_penalty_values`。

### Lines 239-258: test case penalty edge cases extreme penalty values / 测试用例 penalty edge cases extreme penalty values
```python
    def test_penalty_edge_cases_extreme_penalty_values(self):
        """Test that extreme penalties strongly increase vocabulary diversity."""
        prompt = (
            "Write the word 'extreme' exactly 20 times in a row, separated by spaces."
        )
        baseline_params = {
            "frequency_penalty": 0.0,
            "presence_penalty": 0.0,
            "repetition_penalty": 1.0,
        }
        extreme_penalty_params = {
            "frequency_penalty": 2.0,
            "presence_penalty": 2.0,
            "repetition_penalty": 2.0,
        }
        self._test_penalty_effect(
            prompt,
            baseline_params,
            extreme_penalty_params,
        )
```
**EN:** Test that extreme penalties strongly increase vocabulary diversity. This test exercises `test_penalty_edge_cases_extreme_penalty_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that extreme penalties strongly increase vocabulary diversity. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_penalty_edge_cases_extreme_penalty_values`。

### Lines 261-262: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=3)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPenalty`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPenalty.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestPenalty.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestPenalty.run_decode`: Helper method for basic decode tests. / 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestPenalty.run_generate_with_prompt`: Helper method to generate text with a specific prompt and parameters. / 该辅助函数封装了 `run_generate_with_prompt`，以便周围测试复用准备、执行或校验逻辑。
- `TestPenalty._get_vocab_diversity`: Calculate vocabulary diversity as unique_words / total_words. / 该代码块实现 `_get_vocab_diversity`，承担模块行为中的一个聚焦逻辑片段。
- `TestPenalty._test_penalty_effect`: Generic test for penalty effects using vocabulary diversity. / 该代码块实现 `_test_penalty_effect`，承担模块行为中的一个聚焦逻辑片段。
- `TestPenalty.test_default_values`: This test exercises `test_default_values` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_values`。
- `TestPenalty.test_frequency_penalty`: This test exercises `test_frequency_penalty` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_frequency_penalty`。
- `TestPenalty.test_min_new_tokens`: This test exercises `test_min_new_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_min_new_tokens`。
- `TestPenalty.test_presence_penalty`: This test exercises `test_presence_penalty` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_presence_penalty`。
- `TestPenalty.test_penalty_mixed`: This test exercises `test_penalty_mixed` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_penalty_mixed`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `random`, `re`, `unittest`, `concurrent.futures`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 262
