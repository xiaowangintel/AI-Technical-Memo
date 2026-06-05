# test_qwen35_hicache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/4-gpu-models/test_qwen35_hicache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 4 gpu models qwen35 hicache in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 4 gpu models qwen35 hicache 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies
```python
import shutil
import tempfile
import time
import unittest
from types import SimpleNamespace

import requests
import zmq
from msgspec.msgpack import Decoder

from sglang.srt.disaggregation.kv_events import BlockStored, KVEventBatch
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also releases spawned processes after the checks finish and registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会在检查完成后释放已启动的进程，并将该用例注册到 CUDA CI 覆盖范围。

### Lines 17-23: Import dependencies
```python
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and runs an evaluation workflow to measure model quality.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并运行评测流程以衡量模型质量。

### Lines 25-25: Register CI metadata
```python
register_cuda_ci(est_time=540, stage="extra-b", runner_config="4-gpu-h100")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 27-28: Define module constants
```python
QWEN35_27B_MODEL = "Qwen/Qwen3.5-27B"
ACC_THRESHOLDS = {QWEN35_27B_MODEL: {"gsm8k": 0.8}}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 31-31: Define class TestQwen35WithHiCache
```python
class TestQwen35WithHiCache(CustomTestCase):
```
**EN:** This declaration introduces the `TestQwen35WithHiCache` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen35WithHiCache` 测试类，并说明它通过继承承担的职责。

### Lines 32-82: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = QWEN35_27B_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.storage_dir = tempfile.mkdtemp(prefix="qwen35-hicache-")
        env = {
            "SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR": cls.storage_dir,
        }
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            env=env,
            other_args=[
                "--tp-size",
                "4",
                "--max-mamba-cache-size",
                "500",
                "--max-total-tokens",
                "120000",
                "--chunked-prefill-size",
                "2048",
                "--mamba-scheduler-strategy",
                "extra_buffer",
                "--mamba-track-interval",
                "128",
                "--mamba-ssm-dtype",
                "bfloat16",
                "--max-running-requests",
                "128",
                "--reasoning-parser",
                "qwen3",
                "--model-loader-extra-config",
                '{"enable_multithread_load": true,"num_threads": 64}',
                "--hicache-mem-layout",
                "page_first_direct",
                "--enable-hierarchical-cache",
                "--hicache-ratio",
                "2",
                "--hicache-size",
                "0",
                "--hicache-write-policy",
                "write_through",
                "--hicache-storage-backend",
                "file",
                "--hicache-storage-prefetch-policy",
                "wait_complete",
                "--kv-events-config",
                '{"publisher": "zmq", "topic": "kv-events"}',
            ],
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 84-87: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        shutil.rmtree(cls.storage_dir, ignore_errors=True)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 89-105: Define helper: run gsm8k
```python
    def _run_gsm8k(self):
        args = SimpleNamespace(
            model=self.model,
            eval_name="gsm8k",
            num_shots=5,
            num_examples=100,
            max_tokens=16000,
            num_threads=50,
            repeat=1,
            temperature=0.6,
            top_p=0.95,
            top_k=20,
            base_url=self.base_url,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        return run_eval(args)
```
**EN:** This helper function encapsulates reusable logic inside `TestQwen35WithHiCache` so the scenario stays organized. It also runs an evaluation workflow to measure model quality and packages launch or evaluation arguments into a lightweight namespace.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestQwen35WithHiCache` 内部调用，从而让场景结构更清晰。 其中还会运行评测流程以衡量模型质量，并把启动或评测参数封装到轻量级命名空间中。

### Lines 107-132: Run test: gsm8k
```python
    def test_gsm8k(self):
        first_metrics = self._run_gsm8k()
        print(f"first_metrics={first_metrics}")
        self.assertGreaterEqual(
            first_metrics["score"], ACC_THRESHOLDS[self.model]["gsm8k"]
        )

        print(f"flush cache")
        res = requests.post(
            f"{self.base_url}/flush_cache",
            params={"timeout": 30},
            timeout=40,
        )
        res.raise_for_status()

        second_metrics = self._run_gsm8k()
        print(f"second_metrics={second_metrics}")
        self.assertGreaterEqual(
            second_metrics["score"], ACC_THRESHOLDS[self.model]["gsm8k"]
        )
        self.assertLessEqual(
            abs(second_metrics["score"] - first_metrics["score"]),
            0.05,
            f"HiCache prefetch accuracy drift too large: "
            f"first={first_metrics['score']}, second={second_metrics['score']}",
        )
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 134-168: Run test: kv events smoke
```python
    def test_kv_events_smoke(self):
        decoder = Decoder(type=KVEventBatch)
        context = zmq.Context()
        sub = context.socket(zmq.SUB)
        sub.connect("tcp://localhost:5557")
        sub.setsockopt_string(zmq.SUBSCRIBE, "kv-events")

        try:
            time.sleep(1.0)
            res = requests.post(
                f"{self.base_url}/generate",
                json={
                    "text": "HiCache KV event compatibility check. " * 64,
                    "sampling_params": {"temperature": 0, "max_new_tokens": 1},
                },
                timeout=120,
            )
            res.raise_for_status()

            events = []
            deadline = time.time() + 10
            while time.time() < deadline and not any(
                isinstance(event, BlockStored) for event in events
            ):
                if sub.poll(timeout=100):
                    _, _, payload = sub.recv_multipart()
                    events.extend(decoder.decode(payload).events)

            self.assertTrue(
                any(isinstance(event, BlockStored) for event in events),
                "Expected at least one BlockStored event from Qwen3.5 HiCache server",
            )
        finally:
            sub.close()
            context.term()
```
**EN:** This test method exercises kv events smoke and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 kv events smoke 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 171-172: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.disaggregation.kv_events`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `msgspec.msgpack`, `requests`, `shutil`, `tempfile`, `time`, `types`, `unittest`, `zmq`
- Notable symbols / 关键符号: `register_cuda_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
