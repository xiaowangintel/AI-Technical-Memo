# test_crash_dump.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/test_crash_dump.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on debug utils crash dump in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 debug utils crash dump 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Import dependencies
```python
import glob
import os
import pickle
import tempfile
import time
import unittest

import requests

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 20-21: Register CI metadata
```python
register_cuda_ci(est_time=40, suite="nightly-1-gpu", nightly=True)
register_amd_ci(est_time=40, suite="nightly-amd-1-gpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 24-24: Define class TestCrashDump
```python
class TestCrashDump(CustomTestCase):
```
**EN:** This declaration introduces the `TestCrashDump` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCrashDump` 测试类，并说明它通过继承承担的职责。

### Lines 25-27: Declare TestCrashDump configuration
```python
    crash_dump_folder = None
    MAX_NEW_TOKENS = 4
    NUM_REQUESTS_BEFORE_CRASH = 5
```
**EN:** This block defines class-level settings that are shared across the `TestCrashDump` test methods.
**CN:** 该代码块定义了 `TestCrashDump` 各测试方法共享的类级配置。

### Lines 29-45: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.crash_dump_folder = tempfile.mkdtemp(prefix="crash_dump_test_")

        with envs.SGLANG_TEST_CRASH_AFTER_STREAM_OUTPUTS.override(
            cls.NUM_REQUESTS_BEFORE_CRASH * cls.MAX_NEW_TOKENS + 10
        ):
            cls.process = popen_launch_server(
                "Qwen/Qwen3-0.6B",
                DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--crash-dump-folder",
                    cls.crash_dump_folder,
                    "--skip-server-warmup",
                ],
            )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 47-49: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 51-100: Run test: crash dump generated (part 1)
```python
    def test_crash_dump_generated(self):
        """Test that crash dump file is generated after server crash."""
        # Send multiple requests to trigger the crash
        for i in range(self.NUM_REQUESTS_BEFORE_CRASH * 2):
            try:
                response = requests.post(
                    DEFAULT_URL_FOR_TEST + "/generate",
                    json={
                        "text": f"Hello, this is request {i}.",
                        "sampling_params": {
                            "max_new_tokens": self.MAX_NEW_TOKENS,
                            "temperature": 0,
                        },
                    },
                    timeout=30,
                )
            except requests.exceptions.RequestException:
                # Connection error expected after crash
                pass

        # Wait for crash dump to be written
        time.sleep(5)

        # Find the crash dump file
        dump_pattern = os.path.join(self.crash_dump_folder, "*", "crash_dump_*.pkl")
        dump_files = glob.glob(dump_pattern)

        # Check that a dump file was created
        self.assertTrue(
            len(dump_files) > 0,
            f"No crash dump file found in {self.crash_dump_folder}. "
            f"Pattern: {dump_pattern}",
        )

        # Read the dump file and verify contents
        dump_file = dump_files[0]
        with open(dump_file, "rb") as f:
            dump_data = pickle.load(f)

        # Verify the dump structure
        self.assertIn("server_args", dump_data)
        self.assertIn("requests", dump_data)

        # Check that there are more than 5 requests in the dump
        requests_list = dump_data["requests"]
        self.assertGreater(
            len(requests_list),
            self.NUM_REQUESTS_BEFORE_CRASH,
            f"Expected more than {self.NUM_REQUESTS_BEFORE_CRASH} requests in dump, but got {len(requests_list)}",
        )
```
**EN:** This test method exercises crash dump generated and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 crash dump generated 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 102-113: Run test: crash dump generated (part 2)
```python
        # Verify each request tuple has the expected structure (obj, out, created_time, finish_time)
        for i, req_tuple in enumerate(requests_list):
            self.assertIsInstance(
                req_tuple,
                tuple,
                f"Request {i} should be a tuple, got {type(req_tuple)}",
            )
            self.assertGreaterEqual(
                len(req_tuple),
                4,
                f"Request {i} tuple should have at least 4 elements",
            )
```
**EN:** This test method exercises crash dump generated and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 crash dump generated 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 116-117: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `glob`, `os`, `pickle`, `requests`, `tempfile`, `time`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
