# test_nvidia_nemotron_3_super_nvfp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/4-gpu-models/test_nvidia_nemotron_3_super_nvfp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 4 gpu models nvidia nemotron 3 super nvfp4 in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 4 gpu models nvidia nemotron 3 super nvfp4 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Import dependencies
```python
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 14-14: Register CI metadata
```python
register_cuda_ci(est_time=540, suite="nightly-4-gpu-b200", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 16-44: Define module constants
```python
NEMOTRON_3_SUPER_NVFP4_MODEL = "nvidia/NVIDIA-Nemotron-3-Super-120B-A12B-NVFP4"

NEMOTRON_3_SUPER_NVFP4_ARGS = [
    "--tp-size",
    "4",
    "--trust-remote-code",
    "--reasoning-parser",
    "nemotron_3",
    "--tool-call-parser",
    "qwen3_coder",
    "--disable-radix-cache",
    "--model-loader-extra-config",
    '{"enable_multithread_load": true, "num_threads": 17}',
]

MTP_ARGS = [
    "--speculative-algorithm",
    "EAGLE",
    "--speculative-num-steps",
    "3",
    "--speculative-eagle-topk",
    "1",
    "--speculative-num-draft-tokens",
    "4",
    "--max-running-requests",
    "200",
    "--mem-fraction-static",
    "0.75",
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 47-64: Define helper: run gsm8k
```python
def _run_gsm8k(test_case):
    args = SimpleNamespace(
        model=test_case.model,
        eval_name="gsm8k",
        num_shots=5,
        num_examples=200,
        max_tokens=16000,
        num_threads=200,
        repeat=1,
        temperature=1.0,
        top_p=0.95,
        base_url=test_case.base_url,
        host="http://127.0.0.1",
        port=int(test_case.base_url.split(":")[-1]),
    )
    metrics = run_eval(args)
    print(f"{metrics=}")
    test_case.assertGreaterEqual(metrics["score"], 0.96)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 67-67: Define class TestNvidiaNemotron3SuperNVFP4
```python
class TestNvidiaNemotron3SuperNVFP4(CustomTestCase):
```
**EN:** This declaration introduces the `TestNvidiaNemotron3SuperNVFP4` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNvidiaNemotron3SuperNVFP4` 测试类，并说明它通过继承承担的职责。

### Lines 68-77: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = NEMOTRON_3_SUPER_NVFP4_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=NEMOTRON_3_SUPER_NVFP4_ARGS,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 79-81: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 83-84: Run test: gsm8k
```python
    def test_gsm8k(self):
        _run_gsm8k(self)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。

### Lines 87-87: Define class TestNvidiaNemotron3SuperNVFP4MTP
```python
class TestNvidiaNemotron3SuperNVFP4MTP(CustomTestCase):
```
**EN:** This declaration introduces the `TestNvidiaNemotron3SuperNVFP4MTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNvidiaNemotron3SuperNVFP4MTP` 测试类，并说明它通过继承承担的职责。

### Lines 88-97: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = NEMOTRON_3_SUPER_NVFP4_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=NEMOTRON_3_SUPER_NVFP4_ARGS + MTP_ARGS,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 99-101: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 103-104: Run test: gsm8k
```python
    def test_gsm8k(self):
        _run_gsm8k(self)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。

### Lines 107-108: Expose unittest entrypoint
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
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
