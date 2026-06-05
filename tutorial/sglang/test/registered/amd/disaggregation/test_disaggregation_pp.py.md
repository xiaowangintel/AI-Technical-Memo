# test_disaggregation_pp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/disaggregation/test_disaggregation_pp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on disaggregation pp in SGLang. It drives an evaluation workflow and verifies the resulting quality signals. / 该测试模块用于分析 SGLang 中与 disaggregation pp 相关的实现或行为。 它会驱动评测流程，并验证最终得到的质量信号。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Import dependencies
```python
import os
import time
import unittest
from types import SimpleNamespace

from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.few_shot_gsm8k import run_eval
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    popen_launch_pd_server,
    try_cached_model,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also runs an evaluation workflow to measure model quality and registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会运行评测流程以衡量模型质量，并将该用例注册到 AMD CI 覆盖范围。

### Lines 18-18: Register CI metadata
```python
register_amd_ci(est_time=600, suite="stage-b-test-large-8-gpu-35x-disaggregation-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 21-21: Define class TestDisaggregationPrefillPPAccuracy
```python
class TestDisaggregationPrefillPPAccuracy(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationPrefillPPAccuracy` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationPrefillPPAccuracy` 测试类，并说明它通过继承承担的职责。

### Lines 22-46: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # set up ROCm env
        os.environ["SGLANG_USE_AITER"] = "1"
        rdma_env = os.environ.get("SGLANG_TEST_RDMA_DEVICE")

        if rdma_env:
            cls.rdma_devices = ["--disaggregation-ib-device", rdma_env]
            print(f"Found RDMA devices in env: {rdma_env}")
        else:
            print("SGLANG_TEST_RDMA_DEVICE is not set! Running without RDMA.")
            cls.rdma_devices = []

        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 48-70: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--pp-size",
            "2",
            "--disable-overlap-schedule",
            "--attention-backend",
            "aiter",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationPrefillPPAccuracy` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationPrefillPPAccuracy` 内部调用，从而让场景结构更清晰。

### Lines 72-93: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--base-gpu-id",
            "4",
            "--attention-backend",
            "aiter",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationPrefillPPAccuracy` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationPrefillPPAccuracy` 内部调用，从而让场景结构更清晰。

### Lines 95-110: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host=f"http://{self.base_host}",
            port=int(self.lb_port),
        )
        metrics = run_eval(args)
        print(f"{metrics=}")

        self.assertGreater(metrics["accuracy"], 0.70)
        # Wait a little bit so that the memory check happens.
        time.sleep(5)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 114-114: Define class TestDisaggregationPrefillPPDynamicChunkAccuracy
```python
class TestDisaggregationPrefillPPDynamicChunkAccuracy(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationPrefillPPDynamicChunkAccuracy` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationPrefillPPDynamicChunkAccuracy` 测试类，并说明它通过继承承担的职责。

### Lines 115-139: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # set up ROCm env
        os.environ["SGLANG_USE_AITER"] = "1"
        rdma_env = os.environ.get("SGLANG_TEST_RDMA_DEVICE")

        if rdma_env:
            cls.rdma_devices = ["--disaggregation-ib-device", rdma_env]
            print(f"Found RDMA devices in env: {rdma_env}")
        else:
            print("SGLANG_TEST_RDMA_DEVICE is not set! Running without RDMA.")
            cls.rdma_devices = []

        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 141-164: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--pp-size",
            "2",
            "--disable-overlap-schedule",
            "--enable-dynamic-chunking",
            "--attention-backend",
            "aiter",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationPrefillPPDynamicChunkAccuracy` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationPrefillPPDynamicChunkAccuracy` 内部调用，从而让场景结构更清晰。

### Lines 166-187: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--base-gpu-id",
            "4",
            "--attention-backend",
            "aiter",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationPrefillPPDynamicChunkAccuracy` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationPrefillPPDynamicChunkAccuracy` 内部调用，从而让场景结构更清晰。

### Lines 189-204: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host=f"http://{self.base_host}",
            port=int(self.lb_port),
        )
        metrics = run_eval(args)
        print(f"{metrics=}")

        self.assertGreater(metrics["accuracy"], 0.70)
        # Wait a little bit so that the memory check happens.
        time.sleep(5)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 208-208: Define class TestDisaggregationDecodePPAccuracy
```python
class TestDisaggregationDecodePPAccuracy(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationDecodePPAccuracy` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationDecodePPAccuracy` 测试类，并说明它通过继承承担的职责。

### Lines 209-233: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # set up ROCm env
        os.environ["SGLANG_USE_AITER"] = "1"
        rdma_env = os.environ.get("SGLANG_TEST_RDMA_DEVICE")

        if rdma_env:
            cls.rdma_devices = ["--disaggregation-ib-device", rdma_env]
            print(f"Found RDMA devices in env: {rdma_env}")
        else:
            print("SGLANG_TEST_RDMA_DEVICE is not set! Running without RDMA.")
            cls.rdma_devices = []

        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 235-257: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--pp-size",
            "2",
            "--disable-overlap-schedule",
            "--attention-backend",
            "aiter",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationDecodePPAccuracy` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationDecodePPAccuracy` 内部调用，从而让场景结构更清晰。

### Lines 259-282: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--pp-size",
            "2",
            "--base-gpu-id",
            "4",
            "--attention-backend",
            "aiter",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationDecodePPAccuracy` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationDecodePPAccuracy` 内部调用，从而让场景结构更清晰。

### Lines 284-299: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host=f"http://{self.base_host}",
            port=int(self.lb_port),
        )
        metrics = run_eval(args)
        print(f"{metrics=}")

        self.assertGreater(metrics["accuracy"], 0.70)
        # Wait a little bit so that the memory check happens.
        time.sleep(5)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 302-303: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `time`, `types`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `run_eval`, `SimpleNamespace`, `unittest.main`, `DEFAULT_MODEL_NAME_FOR_TEST`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
