# test_npu_hierarchical_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/HiCache/test_npu_hierarchical_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on Hi Cache npu hierarchical cache in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 Hi Cache npu hierarchical cache 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies
```python
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ascend.test_ascend_utils import QWEN3_8B_WEIGHTS_PATH
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 15-15: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 18-18: Define class TestNPUHierarchicalCache
```python
class TestNPUHierarchicalCache(CustomTestCase):
```
**EN:** This declaration introduces the `TestNPUHierarchicalCache` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNPUHierarchicalCache` 测试类，并说明它通过继承承担的职责。

### Lines 19-27: Document the class `TestNPUHierarchicalCache`
```python
    """Testcase: HierarchicalCache Test on Ascend NPU.
    Cover scenarios:
    1. Long identical texts: cache can be reused
    2. Short identical texts: cache cannot be reused (page size limit)
    3. Different long texts: cache cannot be reused (prefix mismatch)

    [Test Category] HiCache
    [Test Target] --enable-hierarchical-cache
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNPUHierarchicalCache`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNPUHierarchicalCache`的设计意图。

### Lines 29-50: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = QWEN3_8B_WEIGHTS_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.prefill_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--attention-backend",
            "ascend",
            "--disable-cuda-graph",
            "--mem-fraction-static",
            0.8,
            "--tp-size",
            1,
            "--enable-hierarchical-cache",
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
        cls.base_url += "/v1"
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 52-54: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 56-78: Run test: hierarchical cache reused long identical
```python
    def test_hierarchical_cache_reused_long_identical(self):
        """Long identical texts should reuse HierarchicalCache"""
        # Ultra-long repeated prompt (meets page size requirement)
        long_text = "What is The capital of France?" * 36
        for i in range(2):
            response = requests.post(
                f"{DEFAULT_URL_FOR_TEST}/generate",
                json={
                    "text": long_text,
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": 10,
                    },
                },
            )
            self.assertEqual(response.status_code, 200)
            cached_tokens = int(response.json()["meta_info"]["cached_tokens"])
            if i == 0:
                # First request: no cache
                self.assertEqual(cached_tokens, 0)
            else:
                # Second request: cache reused
                self.assertGreater(cached_tokens, 0)
```
**EN:** This test method exercises hierarchical cache reused long identical and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 hierarchical cache reused long identical 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 80-98: Run test: hierarchical cache not reused short identical
```python
    def test_hierarchical_cache_not_reused_short_identical(self):
        """Short identical texts should NOT reuse HierarchicalCache (page size limit)"""
        # Short text prompt (does not meet page size requirement)
        short_text = "who am i?"
        for i in range(2):
            response = requests.post(
                f"{DEFAULT_URL_FOR_TEST}/generate",
                json={
                    "text": short_text,
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": 10,
                    },
                },
            )
            self.assertEqual(response.status_code, 200)
            # No cache reuse for both requests
            cached_tokens = int(response.json()["meta_info"]["cached_tokens"])
            self.assertEqual(cached_tokens, 0)
```
**EN:** This test method exercises hierarchical cache not reused short identical and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 hierarchical cache not reused short identical 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 100-123: Run test: hierarchical cache not reused different long
```python
    def test_hierarchical_cache_not_reused_different_long(self):
        """Different long texts should NOT reuse HierarchicalCache (text uniqueness)"""
        # Two different long text prompts (both meet the page size requirement)
        texts = [
            "Marie ordered one chicken meal that costs $12, 5 packs of milk that costs $3 each, 4 apples that cost $1.50 each, and some boxes of pizza. Marie paid a total of $50. How many boxes of pizza did Marie order if each box costs $8.50?"
            * 8,
            "Mishka bought 3 pairs of shorts, 3 pairs of pants, and 3 pairs of shoes. One pair of shorts costs $16.50. One pair of pants costs $22.50 and one pair of shoes costs $42. How many dollars did Mishka spend on all the clothing items?"
            * 8,
        ]
        for text in texts:
            response = requests.post(
                f"{DEFAULT_URL_FOR_TEST}/generate",
                json={
                    "text": text,
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": 10,
                    },
                },
            )
            self.assertEqual(response.status_code, 200)
            # No cache reuse for different text requests
            cached_tokens = int(response.json()["meta_info"]["cached_tokens"])
            self.assertEqual(cached_tokens, 0)
```
**EN:** This test method exercises hierarchical cache not reused different long and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 hierarchical cache not reused different long 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 126-127: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `requests`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
