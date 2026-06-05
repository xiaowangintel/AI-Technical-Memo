# test_moriep_small.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/test_moriep_small.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on amd moriep small in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 amd moriep small 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Import dependencies
```python
import os
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.server_args import ZMQ_TCP_PORT_DELTA
from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.network import is_port_available
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.few_shot_gsm8k import run_eval as run_eval_few_shot_gsm8k
from sglang.test.test_utils import (
    DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST,
    DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST_NEXTN,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 22-42: Define helper: wait all ports release
```python
def wait_all_ports_release(base_url, timeout_s=60):
    """Wait until all derived ports are fully released."""
    import time

    port = int(base_url.split(":")[-1])

    # See https://github.com/sgl-project/sglang/blob/495ef8ec64b6b937e59cd530ad3150172061a008/python/sglang/srt/server_args.py#L6958-L6969
    offsets = [
        0,  # no offset
        ZMQ_TCP_PORT_DELTA,  # dist_init_port
        ZMQ_TCP_PORT_DELTA + 1,  # detokenizer_port
        ZMQ_TCP_PORT_DELTA + 2,  # rpc_port
        ZMQ_TCP_PORT_DELTA + 3,  # metrics_port
        ZMQ_TCP_PORT_DELTA + 4,  # scheduler_input_port
    ]
    for _ in range(timeout_s):
        if all(is_port_available(port + off) for off in offsets):
            return
        time.sleep(1)
    # Best-effort: log but don't raise so tearDown doesn't break the next class.
    print(f"Warning: some ports still occupied after {timeout_s}s")
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also waits for asynchronous state transitions before rechecking results.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会在再次检查结果前等待异步状态转换完成。

### Lines 45-45: Register CI metadata
```python
register_amd_ci(est_time=5400, suite="stage-c-test-large-8-gpu-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 47-90: Define module constants
```python
common_args = [
    "--tp-size",
    "8",
    "--ep-size",
    "8",
    "--dp-size",
    "8",
    "--enable-dp-attention",
    "--moe-a2a-backend",
    "mori",
    "--trust-remote-code",
    "--load-balance-method",
    "round_robin",
    "--moe-dense-tp-size",
    "1",
    "--enable-dp-lm-head",
    "--mem-fraction-static",
    "0.7",  # relax for mi300x
    "--chunked-prefill-size",
    "1024",
    "--max-running-requests",
    "128",
    "--context-length",
    "4096",
    "--max-total-tokens",
    "32768",
    "--attention-backend",
    "aiter",
    "--cuda-graph-max-bs",
    "32",
]

mtp_args = [
    "--speculative-algo",
    "EAGLE",
    "--speculative-num-steps",
    "3",
    "--speculative-eagle-topk",
    "1",
    "--speculative-num-draft-tokens",
    "4",
    "--speculative-draft-model-path",
    DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST_NEXTN,
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 93-93: Define class TestPureDP
```python
class TestPureDP(CustomTestCase):
```
**EN:** This declaration introduces the `TestPureDP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPureDP` 测试类，并说明它通过继承承担的职责。

### Lines 95-113: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = common_args

        env = dict(os.environ)
        env["SGLANG_USE_AITER"] = "1"
        env["SGLANG_MORI_DISPATCH_DTYPE"] = "bf16"
        env["SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK"] = "128"
        env["MORI_SHMEM_MODE"] = "ISOLATION"  # avoid out of symmetric heap memory

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 5,
            other_args=other_args,
            env=env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 115-118: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        wait_all_ports_release(cls.base_url)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 120-135: Run test: gsm8k
```python
    def test_gsm8k(
        self,
    ):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")

        self.assertGreaterEqual(metrics["accuracy"], 0.90)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 138-138: Define class TestMTP
```python
class TestMTP(CustomTestCase):
```
**EN:** This declaration introduces the `TestMTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMTP` 测试类，并说明它通过继承承担的职责。

### Lines 140-158: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = common_args + mtp_args

        env = dict(os.environ)
        env["SGLANG_USE_AITER"] = "1"
        env["SGLANG_MORI_DISPATCH_DTYPE"] = "bf16"
        env["SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK"] = "128"
        env["MORI_SHMEM_MODE"] = "ISOLATION"  # avoid out of symmetric heap memory

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 5,
            other_args=other_args,
            env=env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 160-163: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        wait_all_ports_release(cls.base_url)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 165-186: Run test: gsm8k
```python
    def test_gsm8k(
        self,
    ):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")
        self.assertGreaterEqual(metrics["accuracy"], 0.90)

        server_info = requests.get(self.base_url + "/server_info")
        avg_spec_accept_length = server_info.json()["internal_states"][0][
            "avg_spec_accept_length"
        ]
        print(f"{avg_spec_accept_length=}")
        self.assertGreaterEqual(avg_spec_accept_length, 2.8)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 189-189: Define class TestNormal
```python
class TestNormal(CustomTestCase):
```
**EN:** This declaration introduces the `TestNormal` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNormal` 测试类，并说明它通过继承承担的职责。

### Lines 191-212: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = common_args + [
            "--deepep-mode",
            "normal",
        ]

        env = dict(os.environ)
        env["SGLANG_USE_AITER"] = "1"
        env["SGLANG_MORI_DISPATCH_DTYPE"] = "bf16"
        env["SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK"] = "128"
        env["MORI_SHMEM_MODE"] = "ISOLATION"  # avoid out of symmetric heap memory

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 5,
            other_args=other_args,
            env=env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 214-217: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        wait_all_ports_release(cls.base_url)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 219-234: Run test: gsm8k
```python
    def test_gsm8k(
        self,
    ):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")

        self.assertGreaterEqual(metrics["accuracy"], 0.90)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 237-237: Define class TestLowLatency
```python
class TestLowLatency(CustomTestCase):
```
**EN:** This declaration introduces the `TestLowLatency` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLowLatency` 测试类，并说明它通过继承承担的职责。

### Lines 239-262: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = common_args + [
            "--deepep-mode",
            "low_latency",
        ]

        env = dict(os.environ)
        env["SGLANG_USE_AITER"] = "1"
        env["SGLANG_MORI_DISPATCH_DTYPE"] = "bf16"
        env["SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK"] = "128"
        env["MORI_SHMEM_MODE"] = "ISOLATION"  # avoid out of symmetric heap memory
        # FIXME(billishyahao): enable p2p due to no rdma devices on CI machine
        # env["MORI_DISABLE_P2P"] = "1"

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 5,
            other_args=other_args,
            env=env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 264-267: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        wait_all_ports_release(cls.base_url)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 269-284: Run test: gsm8k
```python
    def test_gsm8k(
        self,
    ):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")

        self.assertGreaterEqual(metrics["accuracy"], 0.90)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 287-287: Define class TestTBOwithNormal
```python
class TestTBOwithNormal(CustomTestCase):
```
**EN:** This declaration introduces the `TestTBOwithNormal` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTBOwithNormal` 测试类，并说明它通过继承承担的职责。

### Lines 289-311: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = common_args + [
            "--deepep-mode",
            "normal",
            "--enable-two-batch-overlap",
        ]

        env = dict(os.environ)
        env["SGLANG_USE_AITER"] = "1"
        env["SGLANG_MORI_DISPATCH_DTYPE"] = "bf16"
        env["SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK"] = "128"
        env["MORI_SHMEM_MODE"] = "ISOLATION"  # avoid out of symmetric heap memory

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 5,
            other_args=other_args,
            env=env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 313-316: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        wait_all_ports_release(cls.base_url)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 318-333: Run test: gsm8k
```python
    def test_gsm8k(
        self,
    ):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")

        self.assertGreaterEqual(metrics["accuracy"], 0.90)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 336-336: Define class TestTBOwithLowLatency
```python
class TestTBOwithLowLatency(CustomTestCase):
```
**EN:** This declaration introduces the `TestTBOwithLowLatency` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTBOwithLowLatency` 测试类，并说明它通过继承承担的职责。

### Lines 338-362: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = common_args + [
            "--deepep-mode",
            "low_latency",
            "--enable-two-batch-overlap",
        ]

        env = dict(os.environ)
        env["SGLANG_USE_AITER"] = "1"
        env["SGLANG_MORI_DISPATCH_DTYPE"] = "bf16"
        env["SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK"] = "128"
        env["MORI_SHMEM_MODE"] = "ISOLATION"  # avoid out of symmetric heap memory
        # FIXME(billishyahao): enable p2p due to no rdma devices on CI machine
        # env["MORI_DISABLE_P2P"] = "1"

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 5,
            other_args=other_args,
            env=env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 364-367: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        wait_all_ports_release(cls.base_url)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 369-384: Run test: gsm8k
```python
    def test_gsm8k(
        self,
    ):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")

        self.assertGreaterEqual(metrics["accuracy"], 0.90)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 387-387: Define class TestMTPwithTBONormal
```python
class TestMTPwithTBONormal(CustomTestCase):
```
**EN:** This declaration introduces the `TestMTPwithTBONormal` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMTPwithTBONormal` 测试类，并说明它通过继承承担的职责。

### Lines 389-416: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = (
            common_args
            + mtp_args
            + [
                "--deepep-mode",
                "normal",
                "--enable-two-batch-overlap",
            ]
        )

        env = dict(os.environ)
        env["SGLANG_USE_AITER"] = "1"
        env["SGLANG_MORI_DISPATCH_DTYPE"] = "bf16"
        env["SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK"] = "128"
        env["MORI_SHMEM_MODE"] = "ISOLATION"  # avoid out of symmetric heap memory
        env["MORI_ENABLE_SDMA"] = "true"

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 5,
            other_args=other_args,
            env=env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 418-421: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        wait_all_ports_release(cls.base_url)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 423-444: Run test: gsm8k
```python
    def test_gsm8k(
        self,
    ):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")
        self.assertGreaterEqual(metrics["accuracy"], 0.90)

        server_info = requests.get(self.base_url + "/server_info")
        avg_spec_accept_length = server_info.json()["internal_states"][0][
            "avg_spec_accept_length"
        ]
        print(f"{avg_spec_accept_length=}")
        self.assertGreaterEqual(avg_spec_accept_length, 2.8)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 447-447: Define class TestMTPwithTBOLowLatency
```python
class TestMTPwithTBOLowLatency(CustomTestCase):
```
**EN:** This declaration introduces the `TestMTPwithTBOLowLatency` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMTPwithTBOLowLatency` 测试类，并说明它通过继承承担的职责。

### Lines 449-479: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_DEEPEP_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = (
            common_args
            + mtp_args
            + [
                "--deepep-mode",
                "low_latency",
                "--enable-two-batch-overlap",
            ]
        )

        env = dict(os.environ)
        env["SGLANG_USE_AITER"] = "1"
        env["SGLANG_MORI_DISPATCH_DTYPE"] = "bf16"
        env["SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK"] = "128"
        env["SGLANG_ENABLE_SPEC_V2"] = "false"
        env["MORI_SHMEM_MODE"] = "ISOLATION"  # avoid out of symmetric heap memory
        # FIXME(billishyahao): enable p2p due to no rdma devices on CI machine
        # env["MORI_DISABLE_P2P"] = "1"
        env["MORI_ENABLE_SDMA"] = "true"

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 5,
            other_args=other_args,
            env=env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 481-484: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        wait_all_ports_release(cls.base_url)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 486-507: Run test: gsm8k
```python
    def test_gsm8k(
        self,
    ):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")
        self.assertGreaterEqual(metrics["accuracy"], 0.90)

        server_info = requests.get(self.base_url + "/server_info")
        avg_spec_accept_length = server_info.json()["internal_states"][0][
            "avg_spec_accept_length"
        ]
        print(f"{avg_spec_accept_length=}")
        self.assertGreaterEqual(avg_spec_accept_length, 2.8)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 510-511: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.network`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `requests`, `time`, `types`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `requests.get`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
