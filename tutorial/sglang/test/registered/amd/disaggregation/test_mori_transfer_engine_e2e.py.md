# test_mori_transfer_engine_e2e.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/disaggregation/test_mori_transfer_engine_e2e.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on disaggregation mori transfer engine e2e in SGLang. It interacts with externally visible endpoints and verifies the returned behavior. / 该测试模块用于分析 SGLang 中与 disaggregation mori transfer engine e2e 相关的实现或行为。 它会与对外暴露的端点交互，并验证返回结果是否正确。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Import dependencies
```python
import os
import unittest

import requests

from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    popen_launch_pd_server,
    try_cached_model,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 17-17: Register CI metadata
```python
register_amd_ci(est_time=300, suite="stage-b-test-large-8-gpu-35x-disaggregation-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 20-20: Define class MoriTransferEngineBase
```python
class MoriTransferEngineBase(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `MoriTransferEngineBase` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `MoriTransferEngineBase` 测试类，并说明它通过继承承担的职责。

### Lines 21-25: Declare MoriTransferEngineBase configuration
```python
    port_delta = 0
    prefill_tp = 1
    decode_tp = 1
    decode_base_gpu_id = 1
    required_gpus = 2
```
**EN:** This block defines class-level settings that are shared across the `MoriTransferEngineBase` test methods.
**CN:** 该代码块定义了 `MoriTransferEngineBase` 各测试方法共享的类级配置。

### Lines 27-78: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        try:
            import torch

            if not torch.cuda.is_available():
                raise unittest.SkipTest("torch.cuda is not available.")
            if torch.cuda.device_count() < cls.required_gpus:
                raise unittest.SkipTest(
                    f"MORI PD smoke test requires >= {cls.required_gpus} visible GPUs."
                )
        except Exception as e:
            raise unittest.SkipTest(f"torch is not available/usable: {e}")

        super().setUpClass()

        cls._old_use_aiter = os.environ.get("SGLANG_USE_AITER")
        os.environ["SGLANG_USE_AITER"] = "1"

        # The shared fixture defaults to Mooncake in CI; pin Mori explicitly here.
        cls.transfer_backend = ["--disaggregation-transfer-backend", "mori"]

        rdma_env = os.environ.get("SGLANG_TEST_RDMA_DEVICE")
        if rdma_env:
            cls.rdma_devices = ["--disaggregation-ib-device", rdma_env]
            print(f"Found RDMA devices in env: {rdma_env}")
        else:
            print("SGLANG_TEST_RDMA_DEVICE is not set! Running without RDMA.")
            cls.rdma_devices = []

        cls._shift_ports()
        cls.model = try_cached_model(
            os.environ.get(
                "SGLANG_MORI_E2E_TEST_MODEL",
                DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
            )
        )

        cls.start_prefill()
        cls.start_decode()

        cls.wait_server_ready(
            cls.prefill_url + "/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            process=cls.process_prefill,
        )
        cls.wait_server_ready(
            cls.decode_url + "/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            process=cls.process_decode,
        )
        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 80-86: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        if getattr(cls, "_old_use_aiter", None) is None:
            os.environ.pop("SGLANG_USE_AITER", None)
        else:
            os.environ["SGLANG_USE_AITER"] = cls._old_use_aiter
        super().tearDownClass()
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。

### Lines 88-100: Define helper: shift ports
```python
    @classmethod
    def _shift_ports(cls):
        if cls.port_delta == 0:
            return

        cls.lb_port = str(int(cls.lb_port) + cls.port_delta)
        cls.prefill_port = str(int(cls.prefill_port) + cls.port_delta)
        cls.decode_port = str(int(cls.decode_port) + cls.port_delta)
        cls.bootstrap_port = str(int(cls.bootstrap_port) + cls.port_delta)
        cls.prefill_url = f"http://{cls.base_host}:{cls.prefill_port}"
        cls.decode_url = f"http://{cls.base_host}:{cls.decode_port}"
        cls.lb_url = f"http://{cls.base_host}:{cls.lb_port}"
        cls.base_url = cls.lb_url
```
**EN:** This helper function encapsulates reusable logic inside `MoriTransferEngineBase` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `MoriTransferEngineBase` 内部调用，从而让场景结构更清晰。

### Lines 102-121: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            str(cls.prefill_tp),
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
**EN:** This helper function encapsulates reusable logic inside `MoriTransferEngineBase` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `MoriTransferEngineBase` 内部调用，从而让场景结构更清晰。

### Lines 123-144: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            str(cls.decode_tp),
            "--base-gpu-id",
            str(cls.decode_base_gpu_id),
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
**EN:** This helper function encapsulates reusable logic inside `MoriTransferEngineBase` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `MoriTransferEngineBase` 内部调用，从而让场景结构更清晰。

### Lines 146-159: Define helper: assert generate smoke
```python
    def _assert_generate_smoke(self):
        resp = requests.post(
            self.lb_url + "/generate",
            json={
                "text": "Hello",
                "sampling_params": {"temperature": 0, "max_new_tokens": 8},
            },
            timeout=120,
        )
        self.assertEqual(resp.status_code, 200, resp.text)
        out = resp.json()
        self.assertIn("text", out)
        self.assertIsInstance(out["text"], str)
        self.assertGreater(len(out["text"]), 0)
```
**EN:** This helper function encapsulates reusable logic inside `MoriTransferEngineBase` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `MoriTransferEngineBase` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 162-162: Define class TestMoriTransferEngineE2E
```python
class TestMoriTransferEngineE2E(MoriTransferEngineBase):
```
**EN:** This declaration introduces the `TestMoriTransferEngineE2E` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMoriTransferEngineE2E` 测试类，并说明它通过继承承担的职责。

### Lines 163-164: Run test: generate smoke
```python
    def test_generate_smoke(self):
        self._assert_generate_smoke()
```
**EN:** This test method exercises generate smoke and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 generate smoke 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 167-167: Define class TestMoriTransferEngineTPMismatchE2E
```python
class TestMoriTransferEngineTPMismatchE2E(MoriTransferEngineBase):
```
**EN:** This declaration introduces the `TestMoriTransferEngineTPMismatchE2E` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMoriTransferEngineTPMismatchE2E` 测试类，并说明它通过继承承担的职责。

### Lines 168-172: Declare TestMoriTransferEngineTPMismatchE2E configuration
```python
    port_delta = 10
    prefill_tp = 2
    decode_tp = 4
    decode_base_gpu_id = 2
    required_gpus = 6
```
**EN:** This block defines class-level settings that are shared across the `TestMoriTransferEngineTPMismatchE2E` test methods.
**CN:** 该代码块定义了 `TestMoriTransferEngineTPMismatchE2E` 各测试方法共享的类级配置。

### Lines 174-175: Run test: generate smoke tp mismatch
```python
    def test_generate_smoke_tp_mismatch(self):
        self._assert_generate_smoke()
```
**EN:** This test method exercises generate smoke tp mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 generate smoke tp mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 178-179: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `requests`, `torch`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
