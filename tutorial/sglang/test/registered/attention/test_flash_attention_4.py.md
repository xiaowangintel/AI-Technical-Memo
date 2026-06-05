# test_flash_attention_4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/attention/test_flash_attention_4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on flash attention 4 in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 flash attention 4 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies
```python
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import get_device_sm, kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 16-16: Register CI metadata
```python
register_cuda_ci(est_time=260, stage="base-b", runner_config="4-gpu-b200")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 19-20: Define class TestFlashAttention4
```python
@unittest.skipIf(get_device_sm() < 100, "Test requires CUDA SM 100 or higher")
class TestFlashAttention4(unittest.TestCase):
```
**EN:** This declaration introduces the `TestFlashAttention4` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFlashAttention4` 测试类，并说明它通过继承承担的职责。

### Lines 21-35: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "Qwen/Qwen3-8B"
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--attention-backend",
            "fa4",
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 37-39: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 41-54: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=1319,
            num_threads=200,
        )
        metrics = run_eval(args)
        print(metrics)

        self.assertGreater(metrics["score"], 0.89)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 57-58: Define class TestFlashAttention4SpeculativeDecodeTopk
```python
@unittest.skipIf(get_device_sm() < 100, "Test requires CUDA SM 100 or higher")
class TestFlashAttention4SpeculativeDecodeTopk(unittest.TestCase):
```
**EN:** This declaration introduces the `TestFlashAttention4SpeculativeDecodeTopk` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFlashAttention4SpeculativeDecodeTopk` 测试类，并说明它通过继承承担的职责。

### Lines 59-63: Document the class `TestFlashAttention4SpeculativeDecodeTopk`
```python
    """Test FlashAttention4 with EAGLE3 speculative decoding (topk > 1).

    Verifies that FA4 + EAGLE3 topk > 1 produces correct outputs and
    achieves meaningful speculative acceptance length.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFlashAttention4SpeculativeDecodeTopk`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFlashAttention4SpeculativeDecodeTopk`的设计意图。

### Lines 65-89: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "Qwen/Qwen3-30B-A3B-Instruct-2507"
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--attention-backend",
            "fa4",
            "--speculative-algorithm",
            "EAGLE3",
            "--speculative-draft-model-path",
            "lmsys/SGLang-EAGLE3-Qwen3-30B-A3B-Instruct-2507-SpecForge-Nex",
            "--speculative-num-steps",
            "5",
            "--speculative-eagle-topk",
            "4",
            "--speculative-num-draft-tokens",
            "8",
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 91-93: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 95-114: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=1319,
            num_threads=200,
        )
        metrics = run_eval(args)
        print(metrics)
        self.assertGreater(metrics["score"], 0.89)

        server_info = requests.get(self.base_url + "/server_info").json()
        avg_spec_accept_length = server_info["internal_states"][0][
            "avg_spec_accept_length"
        ]
        print(f"{avg_spec_accept_length=}")
        self.assertGreater(avg_spec_accept_length, 1.5)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 117-118: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `requests`, `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `SimpleNamespace`, `requests.get`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
