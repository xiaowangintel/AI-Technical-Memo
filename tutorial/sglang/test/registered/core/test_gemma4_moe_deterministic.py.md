# test_gemma4_moe_deterministic.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/core/test_gemma4_moe_deterministic.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on core gemma4 moe deterministic in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 core gemma4 moe deterministic 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Document the module
```python
"""Regression test for issue #24394.

`--enable-deterministic-inference` with `--attention-backend triton` on a
hybrid `SWAKVPool` model (Gemma4 family) used to crash with
`CUDA error: an illegal memory access` inside `_fwd_kernel_unified`: the
unified extend kernel read the new tokens at `out_cache_loc` (full-pool
index space) while `SWAKVPool.set_kv_buffer` had written them at the
SWA-translated indices. With diverse prompts the OOB never materialises;
the repro is same-prompt × high-concurrency, which is what this test fires.
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 12-24: Import dependencies
```python
import concurrent.futures
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 26-26: Register CI metadata
```python
register_cuda_ci(est_time=107, stage="base-b", runner_config="2-gpu-large")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 29-38: Define module constants
```python
PROMPT = (
    "Question: Janet's ducks lay 16 eggs per day. She eats three for breakfast "
    "every morning and bakes muffins for her friends every day with four. She "
    "sells the remainder at the farmers' market daily for $2 per fresh duck "
    "egg. How much in dollars does she make every day at the farmers' market?\n"
    "Answer:"
)
NUM_REQUESTS = 180
CONCURRENCY = 128
MAX_TOKENS = 256
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 41-41: Define class TestGemma4MoeDeterministic
```python
class TestGemma4MoeDeterministic(CustomTestCase):
```
**EN:** This declaration introduces the `TestGemma4MoeDeterministic` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGemma4MoeDeterministic` 测试类，并说明它通过继承承担的职责。

### Lines 42-70: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "google/gemma-4-26B-A4B-it"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--tp-size",
                "2",
                "--attention-backend",
                "triton",
                "--enable-deterministic-inference",
                "--dtype",
                "bfloat16",
                "--mem-fraction-static",
                "0.55",
                "--max-running-requests",
                "16",
                "--context-length",
                "2048",
                "--max-total-tokens",
                "32768",
                "--skip-server-warmup",
                "--random-seed",
                "0",
            ],
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 72-75: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 77-93: Define helper: fire one
```python
    def _fire_one(self):
        try:
            r = requests.post(
                self.base_url + "/v1/completions",
                json={
                    "model": self.model,
                    "prompt": PROMPT,
                    "max_tokens": MAX_TOKENS,
                    "temperature": 0.0,
                    "top_k": 1,
                },
                timeout=300,
            )
            r.raise_for_status()
            return True, ""
        except Exception as e:
            return False, repr(e)
```
**EN:** This helper function encapsulates reusable logic inside `TestGemma4MoeDeterministic` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGemma4MoeDeterministic` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 95-119: Run test: no ima under concurrent load
```python
    def test_no_ima_under_concurrent_load(self):
        try:
            requests.get(self.base_url + "/flush_cache", timeout=30)
        except Exception:
            pass

        n_ok = n_fail = 0
        first_fail = ""
        with concurrent.futures.ThreadPoolExecutor(max_workers=CONCURRENCY) as ex:
            futs = [ex.submit(self._fire_one) for _ in range(NUM_REQUESTS)]
            for f in concurrent.futures.as_completed(futs):
                ok, msg = f.result()
                if ok:
                    n_ok += 1
                else:
                    if n_fail == 0:
                        first_fail = msg
                    n_fail += 1

        print(f"n_ok={n_ok} n_fail={n_fail} first_fail={first_fail!r}")
        self.assertEqual(
            n_fail,
            0,
            f"{n_fail}/{NUM_REQUESTS} requests failed; first error: {first_fail}",
        )
```
**EN:** This test method exercises no ima under concurrent load and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 no ima under concurrent load 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 122-123: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `concurrent.futures`, `requests`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.get`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
