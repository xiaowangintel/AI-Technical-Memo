# test_disaggregation_xpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/disaggregation/test_disaggregation_xpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on disaggregation xpu in SGLang. It interacts with externally visible endpoints and verifies the returned behavior. / 该测试模块用于分析 SGLang 中与 disaggregation xpu 相关的实现或行为。 它会与对外暴露的端点交互，并验证返回结果是否正确。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Document the module
```python
"""
Disaggregation integration test for the NIXL transfer backend on Intel XPU.

Launches a prefill server, a decode server, and a load-balancer using the
NIXL KV-transfer backend, then verifies that basic text completion works
end-to-end.  This exercises the np.uint64 pointer-arithmetic fix in
python/sglang/srt/disaggregation/nixl/conn.py, which is required on
Intel XPU where device addresses have bit 63 set (e.g. 0xffff81ab54e01000)
and would overflow np.int64.

Usage:
    python3 -m pytest test/registered/disaggregation/test_disaggregation_xpu.py -v
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 15-25: Import dependencies
```python
import subprocess
import unittest

import requests
import torch

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import DEFAULT_SMALL_MODEL_NAME_FOR_TEST_QWEN
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 27-32: Register CI metadata
```python
register_cuda_ci(
    est_time=300,
    stage="base-a",
    runner_config="1-gpu-small",
    disabled="Intel XPU only — not available in standard CUDA CI",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 34-34: Define module constants
```python
_XPU_AVAILABLE = torch.xpu.is_available()
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 37-40: Define class TestDisaggregationNixlBasic
```python
@unittest.skipUnless(
    _XPU_AVAILABLE, "Intel XPU not available (torch.xpu.is_available() returned False)"
)
class TestDisaggregationNixlBasic(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationNixlBasic` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationNixlBasic` 测试类，并说明它通过继承承担的职责。

### Lines 41-41: Document the class `TestDisaggregationNixlBasic`
```python
    """Smoke-test the NIXL disaggregation backend with a small completion."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDisaggregationNixlBasic`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDisaggregationNixlBasic`的设计意图。

### Lines 43-57: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST_QWEN
        # Force the NIXL backend and XPU device.
        cls.transfer_backend = ["--disaggregation-transfer-backend", "nixl"]
        cls.rdma_devices = []
        cls.extra_prefill_args = ["--device", "xpu"]
        cls.extra_decode_args = ["--device", "xpu"]
        subprocess.check_call(
            ["pip", "install", "sglang-router"],
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL,
        )
        cls.launch_all()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 59-75: Run test: completion returns text
```python
    def test_completion_returns_text(self):
        """A simple completion must succeed and return non-empty generated text."""
        response = requests.post(
            self.lb_url + "/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {"temperature": 0, "max_new_tokens": 16},
            },
        )
        self.assertEqual(response.status_code, 200, response.text)
        data = response.json()
        self.assertIn("text", data, f"Unexpected response shape: {data}")
        self.assertGreater(
            len(data["text"]),
            0,
            "Generated text should not be empty",
        )
```
**EN:** This test method exercises completion returns text and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 completion returns text 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 77-89: Run test: completion correct output
```python
    def test_completion_correct_output(self):
        """Disaggregated NIXL output must produce the expected token for a deterministic prompt."""
        response = requests.post(
            self.lb_url + "/generate",
            json={
                "text": "1 + 1 =",
                "sampling_params": {"temperature": 0, "max_new_tokens": 4},
            },
        )
        self.assertEqual(response.status_code, 200, response.text)
        generated = response.json()["text"]
        # The model should produce "2" somewhere in the first few tokens.
        self.assertIn("2", generated, f"Expected '2' in output, got: {generated!r}")
```
**EN:** This test method exercises completion correct output and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 completion correct output 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 92-93: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `requests`, `subprocess`, `torch`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `requests.post`, `unittest.main`
