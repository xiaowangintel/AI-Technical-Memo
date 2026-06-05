# test_triton_sliding_window.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/attention/test_triton_sliding_window.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on attention triton sliding window in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 attention triton sliding window 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Import dependencies
```python
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_amd_ci,
    is_in_ci,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 19-20: Register CI metadata
```python
register_cuda_ci(est_time=93, stage="extra-a", runner_config="1-gpu-large")
register_amd_ci(est_time=200, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 23-23: Define class TestSlidingWindowAttentionTriton
```python
class TestSlidingWindowAttentionTriton(CustomTestCase):
```
**EN:** This declaration introduces the `TestSlidingWindowAttentionTriton` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSlidingWindowAttentionTriton` 测试类，并说明它通过继承承担的职责。

### Lines 24-24: Document the class `TestSlidingWindowAttentionTriton`
```python
    """Test sliding window attention functionality with triton backend."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestSlidingWindowAttentionTriton`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestSlidingWindowAttentionTriton`的设计意图。

### Lines 26-49: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        """Set up the test server with Gemma3 model and triton backend."""
        # Gemma3 model supports sliding window attention
        cls.model = "google/gemma-3-4b-it"
        cls.base_url = DEFAULT_URL_FOR_TEST

        cls.common_args = [
            "--trust-remote-code",
            "--attention-backend",
            "triton",
            "--context-length",
            "8192",
            "--random-seed",
            "42",
        ]

        cls.short_context_prompt = "The capital of France is"

        # Test prompt longer than window size
        cls.long_context_prompt = """
        Once upon a time, there was a mountain. In the mountain, there was a temple. In the temple, there was an old monk telling a story. The story was:
        """ * 100
        cls.long_context_prompt += "\nNow, summarize the story in one sentence:"
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 51-66: Define helper: test mmlu
```python
    def _test_mmlu(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="mmlu",
            num_examples=200,
            num_threads=32,
        )

        metrics = run_eval(args)
        print(f"MMLU metrics with sliding window: {metrics}")

        if is_in_amd_ci():
            self.assertGreaterEqual(metrics["score"], 0.55)
        else:
            self.assertGreaterEqual(metrics["score"], 0.60)
```
**EN:** This helper function encapsulates reusable logic inside `TestSlidingWindowAttentionTriton` so the scenario stays organized. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestSlidingWindowAttentionTriton` 内部调用，从而让场景结构更清晰。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 68-83: Define helper: test short context generation
```python
    def _test_short_context_generation(self):
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": self.short_context_prompt,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 256,
                },
            },
        )

        self.assertEqual(response.status_code, 200)
        result = response.json()
        self.assertIn("paris", result["text"].lower())
        print(f"Short context generation result: {result['text']}")
```
**EN:** This helper function encapsulates reusable logic inside `TestSlidingWindowAttentionTriton` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestSlidingWindowAttentionTriton` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 85-100: Define helper: test long context generation
```python
    def _test_long_context_generation(self):
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": self.long_context_prompt,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 256,
                },
            },
        )

        self.assertEqual(response.status_code, 200)
        result = response.json()
        self.assertGreater(len(result["text"].strip()), 0)
        print(f"Long context generation result: {result['text'][:100]}...")
```
**EN:** This helper function encapsulates reusable logic inside `TestSlidingWindowAttentionTriton` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestSlidingWindowAttentionTriton` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 102-116: Run test: no cuda graph
```python
    @unittest.skipIf(is_in_ci(), "To reduce the CI execution time.")
    def test_no_cuda_graph(self):
        self.no_cuda_graph_process = popen_launch_server(
            self.model,
            self.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=self.common_args + ["--disable-cuda-graph"],
        )

        try:
            self._test_short_context_generation()
            self._test_long_context_generation()
            self._test_mmlu()
        finally:
            kill_process_tree(self.no_cuda_graph_process.pid)
```
**EN:** This test method exercises no cuda graph and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 no cuda graph 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 118-131: Run test: cuda graph
```python
    def test_cuda_graph(self):
        self.cuda_graph_process = popen_launch_server(
            self.model,
            self.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=self.common_args,
        )

        try:
            self._test_short_context_generation()
            self._test_long_context_generation()
            self._test_mmlu()
        finally:
            kill_process_tree(self.cuda_graph_process.pid)
```
**EN:** This test method exercises cuda graph and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 cuda graph 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 134-135: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `requests`, `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
