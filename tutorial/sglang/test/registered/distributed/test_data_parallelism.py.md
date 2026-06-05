# test_data_parallelism.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_data_parallelism.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on distributed data parallelism in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 distributed data parallelism 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Import dependencies
```python
import time
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 17-18: Register CI metadata
```python
register_cuda_ci(est_time=91, stage="base-b", runner_config="2-gpu-large")
register_amd_ci(est_time=73, suite="stage-b-test-2-gpu-large-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 21-21: Define class TestDataParallelism
```python
class TestDataParallelism(CustomTestCase, GSM8KMixin):
```
**EN:** This declaration introduces the `TestDataParallelism` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDataParallelism` 测试类，并说明它通过继承承担的职责。

### Lines 22-22: Declare TestDataParallelism configuration
```python
    gsm8k_accuracy_thres = 0.7
```
**EN:** This block defines class-level settings that are shared across the `TestDataParallelism` test methods.
**CN:** 该代码块定义了 `TestDataParallelism` 各测试方法共享的类级配置。

### Lines 24-33: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--dp", 2],
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 35-37: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 39-57: Run test: update weight
```python
    def test_update_weight(self):
        response = requests.post(
            self.base_url + "/update_weights_from_disk",
            json={"model_path": DEFAULT_MODEL_NAME_FOR_TEST},
        )

        # check if the response is 200
        assert response.status_code == 200

        # pause a few seconds then send again
        time.sleep(1)

        response = requests.post(
            self.base_url + "/update_weights_from_disk",
            json={"model_path": DEFAULT_MODEL_NAME_FOR_TEST},
        )

        # check if the response is 200
        assert response.status_code == 200
```
**EN:** This test method exercises update weight and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 update weight 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 59-67: Run test: get memory pool size
```python
    def test_get_memory_pool_size(self):
        # use `server_info` instead since `get_memory_pool_size` is merged into `server_info`
        response = requests.get(self.base_url + "/server_info")
        assert response.status_code == 200

        time.sleep(1)

        response = requests.get(self.base_url + "/server_info")
        assert response.status_code == 200
```
**EN:** This test method exercises get memory pool size and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 get memory pool size 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 70-71: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `requests`, `time`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.get`, `requests.post`, `unittest.main`, `DEFAULT_MODEL_NAME_FOR_TEST`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
