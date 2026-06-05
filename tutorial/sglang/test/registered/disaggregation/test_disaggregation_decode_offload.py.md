# test_disaggregation_decode_offload.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/disaggregation/test_disaggregation_decode_offload.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on disaggregation decode offload in SGLang. It drives an evaluation workflow and verifies the resulting quality signals. / 该测试模块用于分析 SGLang 中与 disaggregation decode offload 相关的实现或行为。 它会驱动评测流程，并验证最终得到的质量信号。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Import dependencies
```python
import os
import shutil
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
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
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also releases spawned processes after the checks finish and runs an evaluation workflow to measure model quality.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会在检查完成后释放已启动的进程，并运行评测流程以衡量模型质量。

### Lines 20-25: Register CI metadata
```python
register_cuda_ci(
    est_time=600,
    stage="base-b",
    runner_config="2-gpu-large",
    disabled="Temporarily disable the flaky test.",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 28-28: Define class TestDisaggregationDecodeOffload
```python
class TestDisaggregationDecodeOffload(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationDecodeOffload` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationDecodeOffload` 测试类，并说明它通过继承承担的职责。

### Lines 29-32: Document the class `TestDisaggregationDecodeOffload`
```python
    """
    Test class for verifying KV cache offloading on the decode side in a
    prefill-decode disaggregation setup.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDisaggregationDecodeOffload`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDisaggregationDecodeOffload`的设计意图。

### Lines 34-58: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        # Set environment variable to make offloading more frequent for testing purposes
        cls.old_stride = os.environ.get("SGLANG_HICACHE_DECODE_OFFLOAD_STRIDE")
        cls.hicache_dir = "/tmp/hicache_test"
        os.environ["SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR"] = cls.hicache_dir
        os.environ["SGLANG_HICACHE_DECODE_OFFLOAD_STRIDE"] = "16"

        # Ensure a clean cache directory
        if os.path.exists(cls.hicache_dir):
            shutil.rmtree(cls.hicache_dir)
        os.makedirs(cls.hicache_dir, exist_ok=True)

        super().setUpClass()
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST

        # Non-blocking start of prefill and decode servers
        cls.start_prefill()
        cls.start_decode()

        # Wait for both servers to be ready before proceeding
        cls.wait_server_ready(cls.prefill_url + "/health")
        cls.wait_server_ready(cls.decode_url + "/health")

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 60-73: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        # Restore the original environment variable state
        super().tearDownClass()
        if cls.old_stride is not None:
            os.environ["SGLANG_HICACHE_DECODE_OFFLOAD_STRIDE"] = cls.old_stride
        else:
            os.environ.pop("SGLANG_HICACHE_DECODE_OFFLOAD_STRIDE", None)

        os.environ.pop("SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR", None)

        # Clean up the cache directory
        if os.path.exists(cls.hicache_dir):
            shutil.rmtree(cls.hicache_dir)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。

### Lines 75-99: Define helper: start prefill
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
            "1",
            "--page-size",
            "16",
            "--enable-hierarchical-cache",
            "--hicache-storage-backend",
            "file",
            "--hicache-ratio",
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
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationDecodeOffload` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationDecodeOffload` 内部调用，从而让场景结构更清晰。

### Lines 101-129: Define helper: start decode
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
            "1",
            "--base-gpu-id",
            "1",
            "--disaggregation-decode-enable-offload-kvcache",
            "--num-reserved-decode-tokens",
            "128",
            "--hicache-ratio",
            "2",
            "--page-size",
            "16",
            "--hicache-storage-backend",
            "file",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationDecodeOffload` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationDecodeOffload` 内部调用，从而让场景结构更清晰。

### Lines 131-174: Run test: mmlu double eval
```python
    def test_mmlu_double_eval(self):
        """
        Run two rounds of MMLU evaluation:
        1. First round: Decode node offloads KV cache back to disk (HiCache).
        2. Restart All Nodes to clear memory cache.
        3. Second round: Prefill node loads KV cache from disk (HiCache).
        Verify that both rounds produce consistent scores.
        """
        args = SimpleNamespace(
            base_url=f"http://{self.base_host}:{self.lb_port}",
            model=self.model,
            eval_name="mmlu",
            num_examples=64,
            num_threads=32,
        )

        metrics1 = run_eval(args)

        # Ensure all offloads are committed to disk
        import time

        time.sleep(10)

        kill_process_tree(self.process_prefill.pid)
        kill_process_tree(self.process_decode.pid)
        kill_process_tree(self.process_lb.pid)
        self.process_prefill.wait()
        self.process_decode.wait()
        self.process_lb.wait()

        self.start_prefill()
        self.start_decode()
        self.launch_lb()
        self.wait_server_ready(self.prefill_url + "/health")
        self.wait_server_ready(self.decode_url + "/health")

        metrics2 = run_eval(args)

        # Assert score is above a minimum threshold for both rounds
        self.assertGreater(metrics1["score"], 0.65)
        self.assertGreater(metrics2["score"], 0.65)

        # Score should be consistent: round 2 should be >= round 1, or at least within a 0.05 margin if slightly lower
        self.assertGreaterEqual(metrics2["score"], metrics1["score"] - 0.05)
```
**EN:** This test method exercises mmlu double eval and verifies that the observed behavior matches the expected contract. It also releases spawned processes after the checks finish and runs an evaluation workflow to measure model quality.
**CN:** 该测试方法会执行 mmlu double eval 场景，并验证观测到的行为是否符合预期契约。 其中还会在检查完成后释放已启动的进程，并运行评测流程以衡量模型质量。

### Lines 177-178: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `shutil`, `time`, `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `kill_process_tree`, `run_eval`, `SimpleNamespace`, `unittest.main`, `DEFAULT_MODEL_NAME_FOR_TEST`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
