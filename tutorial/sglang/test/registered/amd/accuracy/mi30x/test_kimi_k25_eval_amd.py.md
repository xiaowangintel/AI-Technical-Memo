# test_kimi_k25_eval_amd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/accuracy/mi30x/test_kimi_k25_eval_amd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on mi30x kimi k25 eval amd in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 mi30x kimi k25 eval amd 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Document the module
```python
"""AMD Kimi-K2.5 GSM8K Completion Evaluation Test (8-GPU)

Tests moonshotai/Kimi-K2.5 with GSM8K few-shot benchmark on MI325.

Registry: nightly-amd-accuracy-8-gpu-kimi-k25 suite
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 8-23: Import dependencies
```python
import os
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.few_shot_gsm8k import run_eval as run_eval_few_shot_gsm8k
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

### Lines 26-28: Register CI metadata
```python
register_amd_ci(
    est_time=3600, suite="nightly-amd-accuracy-8-gpu-kimi-k25", nightly=True
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 30-33: Define module constants
```python
KIMI_K25_MODEL_PATH = "moonshotai/Kimi-K2.5"
SERVER_LAUNCH_TIMEOUT = 3600
ACCURACY_THRESHOLD = 0.92
TP_SIZE = 8
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 36-36: Define class TestKimiK25EvalAMD
```python
class TestKimiK25EvalAMD(CustomTestCase):
```
**EN:** This declaration introduces the `TestKimiK25EvalAMD` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestKimiK25EvalAMD` 测试类，并说明它通过继承承担的职责。

### Lines 37-37: Document the class `TestKimiK25EvalAMD`
```python
    """Kimi-K2.5 GSM8K Completion Evaluation Test for AMD MI325."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestKimiK25EvalAMD`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestKimiK25EvalAMD`的设计意图。

### Lines 39-63: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = KIMI_K25_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--tp",
            str(TP_SIZE),
            "--decode-attention-backend",
            "triton",
            "--prefill-attention-backend",
            "aiter",
            "--trust-remote-code",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true}',
        ]
        env = os.environ.copy()
        env["SGLANG_USE_AITER"] = "1"
        env["SGLANG_ROCM_FUSED_DECODE_MLA"] = "0"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=SERVER_LAUNCH_TIMEOUT,
            other_args=other_args,
            env=env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 65-67: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 69-100: Run test: kimi k25 gsm8k accuracy
```python
    def test_kimi_k25_gsm8k_accuracy(self):
        """Test Kimi-K2.5 with GSM8K few-shot completion benchmark."""
        requests.get(self.base_url + "/flush_cache")

        args = SimpleNamespace(
            num_shots=8,
            data_path=None,
            num_questions=1319,
            parallel=1319,
            max_new_tokens=512,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        acc = metrics["accuracy"]

        passed = acc >= ACCURACY_THRESHOLD
        status = "✅ PASS" if passed else "❌ FAIL"
        print(f"  accuracy={acc:.3f} threshold={ACCURACY_THRESHOLD} {status}")

        if is_in_ci():
            summary = "### Kimi-K2.5 Model (MI325)\n\n"
            summary += "| Model | TP | Accuracy | Threshold | Status |\n"
            summary += "| ----- | -- | -------- | --------- | ------ |\n"
            summary += f"| {KIMI_K25_MODEL_PATH} | {TP_SIZE} | {acc:.3f} | {ACCURACY_THRESHOLD} | {status} |\n"
            write_github_step_summary(summary)

        self.assertGreaterEqual(
            acc,
            ACCURACY_THRESHOLD,
            f"Kimi-K2.5 accuracy {acc:.3f} below threshold {ACCURACY_THRESHOLD}",
        )
```
**EN:** This test method exercises kimi k25 gsm8k accuracy and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 kimi k25 gsm8k accuracy 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 103-104: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `requests`, `types`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `requests.get`, `unittest.main`
