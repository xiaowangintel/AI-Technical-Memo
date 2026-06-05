# test_disaggregation_aarch64.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_disaggregation_aarch64.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on distributed disaggregation aarch64 in SGLang. It drives an evaluation workflow and verifies the resulting quality signals. / 该测试模块用于分析 SGLang 中与 distributed disaggregation aarch64 相关的实现或行为。 它会驱动评测流程，并验证最终得到的质量信号。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import os
import unittest
from types import SimpleNamespace

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    popen_launch_pd_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also runs an evaluation workflow to measure model quality and registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会运行评测流程以衡量模型质量，并将该用例注册到 CUDA CI 覆盖范围。

### Lines 16-16: Register CI metadata
```python
register_cuda_ci(est_time=300, stage="base-c", runner_config="4-gpu-gb200")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 19-19: Define class TestDisaggregationMooncakeAARCH64Accuracy
```python
class TestDisaggregationMooncakeAARCH64Accuracy(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationMooncakeAARCH64Accuracy` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationMooncakeAARCH64Accuracy` 测试类，并说明它通过继承承担的职责。

### Lines 20-35: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        os.environ["SGLANG_MOONCAKE_CUSTOM_MEM_POOL"] = "true"
        os.environ["MC_FORCE_MNNVL"] = "true"
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST

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

### Lines 37-41: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        os.environ.pop("SGLANG_MOONCAKE_CUSTOM_MEM_POOL")
        os.environ.pop("MC_FORCE_MNNVL")
        super().tearDownClass()
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。

### Lines 43-60: Define helper: start prefill
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
            "2",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakeAARCH64Accuracy` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakeAARCH64Accuracy` 内部调用，从而让场景结构更清晰。

### Lines 62-81: Define helper: start decode
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
            "2",
            "--base-gpu-id",
            "2",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakeAARCH64Accuracy` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakeAARCH64Accuracy` 内部调用，从而让场景结构更清晰。

### Lines 83-96: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=200,
            num_threads=128,
        )
        metrics = run_eval(args)
        print(f"Evaluation metrics: {metrics}")

        self.assertGreater(metrics["score"], 0.62)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 99-100: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `run_eval`, `SimpleNamespace`, `unittest.main`, `DEFAULT_MODEL_NAME_FOR_TEST`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
