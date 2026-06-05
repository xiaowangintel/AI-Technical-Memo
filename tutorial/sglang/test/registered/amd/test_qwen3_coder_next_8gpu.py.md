# test_qwen3_coder_next_8gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/test_qwen3_coder_next_8gpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on amd qwen3 coder next 8gpu in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 amd qwen3 coder next 8gpu 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Document the module
```python
"""MI35x Qwen3-Coder-Next Functionality Test (8-GPU)

Tests Qwen3-Coder-Next model with basic configuration
on MI35x. Covers GSM8K accuracy and BS=1 decode speed.

Server args match run_qwen3-coder-next_spec.sh.

Registry: stage-c-test-large-8-gpu-amd-mi35x-qwen3-coder-next suite
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 11-26: Import dependencies
```python
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.few_shot_gsm8k import run_eval as run_eval_few_shot_gsm8k
from sglang.test.send_one import BenchArgs, send_one_prompt
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 28-28: Register CI metadata
```python
register_amd_ci(est_time=3600, suite="stage-c-test-large-8-gpu-amd-mi35x")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 30-44: Define module constants
```python
QWEN3_CODER_NEXT_MODEL_PATH = "Qwen/Qwen3-Coder-Next"
SERVER_LAUNCH_TIMEOUT = 1800

COMMON_ARGS = [
    "--tp",
    "8",
    "--attention-backend",
    "aiter",
    "--chunked-prefill-size",
    "131072",
    "--disable-radix-cache",
    "--mem-fraction-static",
    "0.8",
    "--trust-remote-code",
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 47-47: Define class TestQwen3CoderNext
```python
class TestQwen3CoderNext(CustomTestCase):
```
**EN:** This declaration introduces the `TestQwen3CoderNext` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen3CoderNext` 测试类，并说明它通过继承承担的职责。

### Lines 48-61: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = QWEN3_CODER_NEXT_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = COMMON_ARGS + [
            "--kv-cache-dtype",
            "fp8_e4m3",
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=SERVER_LAUNCH_TIMEOUT,
            other_args=other_args,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 63-65: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 67-87: Run test: a gsm8k
```python
    def test_a_gsm8k(self):
        """GSM8K few-shot accuracy (runs first to warm up server)."""
        requests.get(self.base_url + "/flush_cache")

        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            parallel=128,
            max_new_tokens=512,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_gsm8k (qwen3-coder-next)\n" f'{metrics["accuracy"]=:.3f}\n'
            )
            self.assertGreater(metrics["accuracy"], 0.90)
```
**EN:** This test method exercises a gsm8k and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 a gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 89-99: Run test: bs 1 speed
```python
    def test_bs_1_speed(self):
        """Batch-size 1 decode speed."""
        args = BenchArgs(port=int(self.base_url.split(":")[-1]), max_new_tokens=2048)
        _, speed = send_one_prompt(args)

        print(f"{speed=:.2f}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_bs_1_speed (qwen3-coder-next)\n" f"{speed=:.2f} token/s\n"
            )
```
**EN:** This test method exercises bs 1 speed and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bs 1 speed 场景，并验证观测到的行为是否符合预期契约。

### Lines 103-104: Define class TestQwen3CoderNextMTP
```python
@unittest.skip("MTP perf not ready yet — Triton extend_attention fp8 kv cache TODO")
class TestQwen3CoderNextMTP(CustomTestCase):
```
**EN:** This declaration introduces the `TestQwen3CoderNextMTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen3CoderNextMTP` 测试类，并说明它通过继承承担的职责。

### Lines 105-127: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = QWEN3_CODER_NEXT_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        # TODO: Support MTP with fp8 kv cache on gfx950.
        # Note: no --kv-cache-dtype fp8_e4m3 because Triton extend_attention
        # used by MTP does not support fp8 kv cache on gfx950.
        other_args = COMMON_ARGS + [
            "--speculative-algorithm",
            "EAGLE",
            "--speculative-num-steps",
            "3",
            "--speculative-eagle-topk",
            "1",
            "--speculative-num-draft-tokens",
            "4",
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=SERVER_LAUNCH_TIMEOUT,
            other_args=other_args,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 129-131: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 133-162: Run test: a gsm8k
```python
    def test_a_gsm8k(self):
        """GSM8K few-shot accuracy with MTP (runs first to warm up server)."""
        requests.get(self.base_url + "/flush_cache")

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

        server_info = requests.get(self.base_url + "/server_info")
        avg_spec_accept_length = server_info.json()["internal_states"][0][
            "avg_spec_accept_length"
        ]
        print(f"{avg_spec_accept_length=}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_gsm8k (qwen3-coder-next mtp)\n"
                f'{metrics["accuracy"]=:.3f}\n'
                f"{avg_spec_accept_length=:.2f}\n"
            )
            self.assertGreater(metrics["accuracy"], 0.90)
            self.assertGreater(avg_spec_accept_length, 2.0)
```
**EN:** This test method exercises a gsm8k and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 a gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 164-176: Run test: bs 1 speed
```python
    def test_bs_1_speed(self):
        """Batch-size 1 decode speed with MTP."""
        args = BenchArgs(port=int(self.base_url.split(":")[-1]), max_new_tokens=2048)
        acc_length, speed = send_one_prompt(args)

        print(f"{acc_length=:.2f} {speed=:.2f}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_bs_1_speed (qwen3-coder-next mtp)\n"
                f"{acc_length=:.2f}\n"
                f"{speed=:.2f} token/s\n"
            )
```
**EN:** This test method exercises bs 1 speed and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bs 1 speed 场景，并验证观测到的行为是否符合预期契约。

### Lines 181-184: Expose unittest entrypoint
```python
if __name__ == "__main__":
    import unittest

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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.send_one`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `requests`, `types`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `requests.get`, `unittest.main`
