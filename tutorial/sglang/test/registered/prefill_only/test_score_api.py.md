# test_score_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/prefill_only/test_score_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates score api behavior in SGLang's prefill only area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 prefill only 领域中与 score api 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: supporting statements / 辅助语句
```python
"""HTTP layer tests for the /v1/score endpoint.

Two test classes, each with its own server instance:

  TestCausalLMScoringHTTP    — basic endpoint: schema defaults, response
                               structure, error rejection (no MIS)
  TestCausalLMMISScoringHTTP — MIS mode: validates --enable-mis CLI flag
                               wiring and per-item output shape

Engine-level correctness (numerical accuracy, batching, edge cases) lives in
test_score_engine.py.  These tests focus on the HTTP integration seam:
Pydantic schema defaults, FastAPI routing, and server argument wiring.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 15-28: module imports and dependencies / 模块导入与依赖
```python
import os
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `unittest`, `requests`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `unittest`, `requests`, `sglang.srt.utils`。

### Lines 30-32: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=71, stage="base-b", runner_config="1-gpu-small")

_MODEL = os.environ.get("TEST_MODEL_NAME", DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, get.
**CN:** 该代码块通过 register_cuda_ci, get 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 33-39: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Basic scoring (no MIS delimiter)
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 40-40: class TestCausalLMScoringHTTP declaration / 类 TestCausalLMScoringHTTP 声明
```python
class TestCausalLMScoringHTTP(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 41-46: supporting statements / 辅助语句
```python
    """Validates /v1/score HTTP integration — schema, defaults, and error handling.

    Starts a plain CausalLM server (no --enable-mis) to test
    the HTTP layer in isolation: response envelope shape, the apply_softmax
    default (False), and Pydantic validation errors on malformed input.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 48-56: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = _MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 58-61: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 63-64: method post / 方法 post
```python
    def _post(self, payload):
        return requests.post(self.base_url + "/v1/score", json=payload)
```
**EN:** This block implements `_post` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_post`，承担模块行为中的一个聚焦逻辑片段。

### Lines 66-81: test case response envelope / 测试用例 response envelope
```python
    def test_response_envelope(self):
        """Response JSON contains scores, model, and object='scoring'."""
        resp = self._post(
            {
                "query": "The capital of France is",
                "items": ["Paris", "Berlin"],
                "label_token_ids": [1, 2],
                "apply_softmax": True,
                "model": self.model,
            }
        )
        self.assertEqual(resp.status_code, 200)
        body = resp.json()
        self.assertIn("scores", body)
        self.assertIn("model", body)
        self.assertEqual(body["object"], "scoring")
```
**EN:** Response JSON contains scores, model, and object='scoring'. This test exercises `test_response_envelope` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Response JSON contains scores, model, and object='scoring'. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_envelope`。

### Lines 83-98: test case apply softmax false by default / 测试用例 apply softmax false by default
```python
    def test_apply_softmax_false_by_default(self):
        """Without apply_softmax=True, raw log-probs are returned (do not sum to 1)."""
        resp = self._post(
            {
                "query": "The capital of France is",
                "items": ["Paris"],
                "label_token_ids": [1, 2],
                # apply_softmax intentionally omitted — default is False
                "model": self.model,
            }
        )
        self.assertEqual(resp.status_code, 200)
        scores = resp.json()["scores"]
        self.assertEqual(len(scores), 1)
        # Raw log-probs over a vocabulary subset do not sum to 1
        self.assertNotAlmostEqual(sum(scores[0]), 1.0, places=3)
```
**EN:** Without apply_softmax=True, raw log-probs are returned (do not sum to 1). This test exercises `test_apply_softmax_false_by_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Without apply_softmax=True, raw log-probs are returned (do not sum to 1). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_apply_softmax_false_by_default`。

### Lines 100-115: test case apply softmax true normalizes / 测试用例 apply softmax true normalizes
```python
    def test_apply_softmax_true_normalizes(self):
        """With apply_softmax=True, scores form a valid probability distribution."""
        resp = self._post(
            {
                "query": "The capital of France is",
                "items": ["Paris", "Berlin", "Rome"],
                "label_token_ids": [1, 2, 3],
                "apply_softmax": True,
                "model": self.model,
            }
        )
        self.assertEqual(resp.status_code, 200)
        for row in resp.json()["scores"]:
            self.assertAlmostEqual(sum(row), 1.0, places=6)
            for v in row:
                self.assertGreaterEqual(v, 0.0)
```
**EN:** With apply_softmax=True, scores form a valid probability distribution. This test exercises `test_apply_softmax_true_normalizes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** With apply_softmax=True, scores form a valid probability distribution. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_apply_softmax_true_normalizes`。

### Lines 117-137: test case schema rejection / 测试用例 schema rejection
```python
    def test_schema_rejection(self):
        """Malformed payloads must be rejected with HTTP 4xx."""
        bad_payloads = [
            # label_token_ids must be List[int]
            {
                "query": "Q",
                "items": ["X"],
                "label_token_ids": "bad",
                "model": self.model,
            },
            # items must be str / List[str] / List[List[int]], not int
            {
                "query": "Q",
                "items": 42,
                "label_token_ids": [1, 2],
                "model": self.model,
            },
        ]
        for payload in bad_payloads:
            with self.subTest(payload=list(payload.keys())):
                self.assertGreaterEqual(self._post(payload).status_code, 400)
```
**EN:** Malformed payloads must be rejected with HTTP 4xx. This test exercises `test_schema_rejection` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Malformed payloads must be rejected with HTTP 4xx. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_schema_rejection`。

### Lines 138-144: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# MIS scoring (with --enable-mis)
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 145-145: class TestCausalLMMISScoringHTTP declaration / 类 TestCausalLMMISScoringHTTP 声明
```python
class TestCausalLMMISScoringHTTP(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 146-151: supporting statements / 辅助语句
```python
    """Validates /v1/score with --enable-mis.

    Confirms that the CLI flag is correctly wired into ServerArgs and that the
    endpoint returns one probability vector per item when items are
    delimiter-packed into a single forward pass.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 153-169: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = _MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--disable-radix-cache",
                "--chunked-prefill-size",
                "-1",
                "--enable-mis",
                "--attention-backend",
                "flashinfer",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 171-174: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 176-188: method score / 方法 score
```python
    def _score(self, query, items, label_token_ids):
        resp = requests.post(
            self.base_url + "/v1/score",
            json={
                "query": query,
                "items": items,
                "label_token_ids": label_token_ids,
                "apply_softmax": True,
                "model": self.model,
            },
        )
        self.assertEqual(resp.status_code, 200)
        return resp.json()
```
**EN:** This block implements `_score` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_score`，承担模块行为中的一个聚焦逻辑片段。

### Lines 190-200: test case one probability vector per item / 测试用例 one probability vector per item
```python
    def test_one_probability_vector_per_item(self):
        """Each item yields one softmax-normalised score vector."""
        items = ["Sacramento", "San Jose", "San Francisco"]
        label_token_ids = [9454, 2753]
        scores = self._score("Is each the capital?", items, label_token_ids)["scores"]
        self.assertEqual(len(scores), len(items))
        for i, row in enumerate(scores):
            self.assertEqual(len(row), len(label_token_ids))
            self.assertAlmostEqual(sum(row), 1.0, places=6, msg=f"Item {i}")
            for v in row:
                self.assertGreaterEqual(v, 0.0)
```
**EN:** Each item yields one softmax-normalised score vector. This test exercises `test_one_probability_vector_per_item` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Each item yields one softmax-normalised score vector. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_one_probability_vector_per_item`。

### Lines 202-204: test case empty items returns empty scores / 测试用例 empty items returns empty scores
```python
    def test_empty_items_returns_empty_scores(self):
        result = self._score("Test query", [], [1, 2])
        self.assertEqual(len(result["scores"]), 0)
```
**EN:** This test exercises `test_empty_items_returns_empty_scores` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_items_returns_empty_scores`。

### Lines 206-220: test case varying item counts / 测试用例 varying item counts
```python
    def test_varying_item_counts(self):
        """1, 2, 4, and 6 items all return the correct number of score vectors."""
        label_token_ids = [1, 2, 3, 4, 5]
        for items in (
            ["Single item"],
            ["Item 1", "Item 2"],
            ["A", "B", "C", "D"],
            ["X", "Y", "Z", "W", "V", "U"],
        ):
            with self.subTest(n=len(items)):
                scores = self._score("Rate each:", items, label_token_ids)["scores"]
                self.assertEqual(len(scores), len(items))
                for row in scores:
                    self.assertEqual(len(row), len(label_token_ids))
                    self.assertAlmostEqual(sum(row), 1.0, places=6)
```
**EN:** 1, 2, 4, and 6 items all return the correct number of score vectors. This test exercises `test_varying_item_counts` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 1, 2, 4, and 6 items all return the correct number of score vectors. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_varying_item_counts`。

### Lines 222-230: test case deterministic / 测试用例 deterministic
```python
    def test_deterministic(self):
        """Back-to-back identical requests return identical scores."""
        items, label_token_ids = ["Option A", "Option B", "Option C"], [1, 2, 3]
        s1 = self._score("Choose:", items, label_token_ids)["scores"]
        s2 = self._score("Choose:", items, label_token_ids)["scores"]
        self.assertEqual(len(s1), len(s2))
        for r1, r2 in zip(s1, s2):
            for v1, v2 in zip(r1, r2):
                self.assertAlmostEqual(v1, v2, places=6)
```
**EN:** Back-to-back identical requests return identical scores. This test exercises `test_deterministic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Back-to-back identical requests return identical scores. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deterministic`。

### Lines 233-234: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=3)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestCausalLMScoringHTTP`: Validates /v1/score HTTP integration — schema, defaults, and error handling. / 用于组织相关测试、夹具或辅助方法。
- `TestCausalLMMISScoringHTTP`: Validates /v1/score with --enable-mis. / 用于组织相关测试、夹具或辅助方法。
- `TestCausalLMScoringHTTP.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestCausalLMScoringHTTP.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestCausalLMScoringHTTP._post`: This block implements `_post` and captures one focused piece of the module's behavior. / 该代码块实现 `_post`，承担模块行为中的一个聚焦逻辑片段。
- `TestCausalLMScoringHTTP.test_response_envelope`: Response JSON contains scores, model, and object='scoring'. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_envelope`。
- `TestCausalLMScoringHTTP.test_apply_softmax_false_by_default`: Without apply_softmax=True, raw log-probs are returned (do not sum to 1). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_apply_softmax_false_by_default`。
- `TestCausalLMScoringHTTP.test_apply_softmax_true_normalizes`: With apply_softmax=True, scores form a valid probability distribution. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_apply_softmax_true_normalizes`。
- `TestCausalLMScoringHTTP.test_schema_rejection`: Malformed payloads must be rejected with HTTP 4xx. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_schema_rejection`。
- `TestCausalLMMISScoringHTTP.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestCausalLMMISScoringHTTP.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestCausalLMMISScoringHTTP._score`: This block implements `_score` and captures one focused piece of the module's behavior. / 该代码块实现 `_score`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `unittest`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 234
