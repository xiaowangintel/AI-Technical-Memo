# test_deepseek_v32_mtp_eval_mi35x.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/accuracy/mi35x/test_deepseek_v32_mtp_eval_mi35x.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on deepseek v32 mtp eval mi35x in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 deepseek v32 mtp eval mi35x 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Document the module
```python
"""MI35x DeepSeek-V3.2 TP+MTP GSM8K Accuracy Evaluation Test (8-GPU)

Tests DeepSeek-V3.2 with TP=8 + MTP (EAGLE speculative decoding) using few-shot
completion benchmark on MI35x.

Registry: nightly-amd-accuracy-8-gpu-mi35x-deepseek-v32-mtp suite
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 9-9: Import dependencies
```python
import os
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 12-13: Implement expr logic
```python
os.environ.setdefault("HF_HOME", "/data2/models/huggingface")
os.environ.setdefault("HF_HUB_CACHE", "/data2/models/huggingface/hub")
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 15-30: Import dependencies
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

### Lines 33-37: Register CI metadata
```python
register_amd_ci(
    est_time=5400,
    suite="nightly-amd-accuracy-8-gpu-mi35x-deepseek-v32-mtp",
    nightly=True,
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 39-39: Define module constants
```python
DEEPSEEK_V32_MODEL_PATH = "deepseek-ai/DeepSeek-V3.2"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 42-43: Define module constants
```python
GSM8K_ACCURACY_THRESHOLD = 0.94
AVG_SPEC_ACCEPT_LENGTH_THRESHOLD = 2.7
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 46-46: Define class TestDeepseekV32TPMTP
```python
class TestDeepseekV32TPMTP(CustomTestCase):
```
**EN:** This declaration introduces the `TestDeepseekV32TPMTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDeepseekV32TPMTP` 测试类，并说明它通过继承承担的职责。

### Lines 47-51: Document the class `TestDeepseekV32TPMTP`
```python
    """Test DeepSeek V3.2 with TP=8 + MTP (EAGLE speculative decoding).

    This test runs GSM8K evaluation and measures both accuracy and
    speculative decoding acceptance length on MI35x.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDeepseekV32TPMTP`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDeepseekV32TPMTP`的设计意图。

### Lines 53-86: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEEPSEEK_V32_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        # Use same args as perf test (which passes successfully)
        other_args = [
            "--trust-remote-code",
            "--tp",
            "8",
            "--nsa-prefill-backend",
            "tilelang",
            "--nsa-decode-backend",
            "tilelang",
            "--speculative-algorithm",
            "EAGLE",
            "--speculative-num-steps",
            "3",
            "--speculative-eagle-topk",
            "1",
            "--speculative-num-draft-tokens",
            "4",
            "--mem-fraction-static",
            "0.7",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true}',
            "--watchdog-timeout",
            "1200",
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=5400,
            other_args=other_args,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 88-90: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 92-124: Run test: a gsm8k
```python
    def test_a_gsm8k(self):
        """GSM8K evaluation for TP+MTP configuration.

        Named with 'a' prefix to run first (alphabetically) to warm up the server.
        """
        requests.get(self.base_url + "/flush_cache")

        args = SimpleNamespace(
            num_shots=20,
            data_path=None,
            num_questions=200,
            parallel=64,
            max_new_tokens=512,
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
                f"### test_gsm8k (deepseek-v32 TP+MTP MI35x)\n"
                f'{metrics["accuracy"]=:.3f}\n'
                f"{avg_spec_accept_length=:.2f}\n"
            )
            self.assertGreater(metrics["accuracy"], GSM8K_ACCURACY_THRESHOLD)
            self.assertGreater(avg_spec_accept_length, AVG_SPEC_ACCEPT_LENGTH_THRESHOLD)
```
**EN:** This test method exercises a gsm8k and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 a gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 126-140: Run test: bs 1 speed
```python
    def test_bs_1_speed(self):
        """Single batch speed test for TP+MTP configuration."""
        args = BenchArgs(port=int(self.base_url.split(":")[-1]), max_new_tokens=2048)
        acc_length, speed = send_one_prompt(args)

        print(f"{acc_length=:.2f} {speed=:.2f}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_bs_1_speed (deepseek-v32 TP+MTP MI35x)\n"
                f"{acc_length=:.2f}\n"
                f"{speed=:.2f} token/s\n"
            )
            self.assertGreater(acc_length, AVG_SPEC_ACCEPT_LENGTH_THRESHOLD)
            self.assertGreater(speed, 55)  # Lowered from 60 for AMD MI35x
```
**EN:** This test method exercises bs 1 speed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bs 1 speed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 143-144: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.send_one`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `requests`, `types`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `requests.get`, `unittest.main`
