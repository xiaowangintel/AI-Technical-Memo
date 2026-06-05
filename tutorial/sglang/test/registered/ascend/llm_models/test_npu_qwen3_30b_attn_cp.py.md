# test_npu_qwen3_30b_attn_cp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/llm_models/test_npu_qwen3_30b_attn_cp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on llm models npu qwen3 30b attn cp in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 llm models npu qwen3 30b attn cp 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import os
import unittest
from types import SimpleNamespace

from sglang.test.ascend.test_ascend_utils import QWEN3_30B_A3B_WEIGHTS_PATH
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.few_shot_gsm8k import run_eval as run_eval_few_shot_gsm8k
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    kill_process_tree,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 16-16: Register CI metadata
```python
register_npu_ci(est_time=500, suite="nightly-4-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 18-24: Define module constants
```python
QWEN3_30B_MODEL = QWEN3_30B_A3B_WEIGHTS_PATH
GSM8K_MIN_ACCURACY = 0.92
GSM8K_NUM_QUESTIONS = 100

_NPU_ENV_VARS = {
    "ASCEND_USE_FIA": "1",
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 27-27: Define class TestQwen330BAttnCP
```python
class TestQwen330BAttnCP(CustomTestCase):
```
**EN:** This declaration introduces the `TestQwen330BAttnCP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen330BAttnCP` 测试类，并说明它通过继承承担的职责。

### Lines 28-38: Document the class `TestQwen330BAttnCP`
```python
    """GSM8K accuracy test for Qwen3-30B-A3B mixed deployment on 4 NPUs.

    The test uses:
    - TP = 4
    - MOE_DP = 2
    - ATTN_CP = 2
    - prefill context parallel enabled

    This is the mixed/co-located deployment variant and reuses the Ascend
    environment variables from the PD GSM8K test.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestQwen330BAttnCP`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestQwen330BAttnCP`的设计意图。

### Lines 40-68: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = QWEN3_30B_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.npu_env = {**os.environ, **_NPU_ENV_VARS}
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--trust-remote-code",
                "--mem-fraction-static",
                "0.7",
                "--max-running-requests",
                "32",
                "--attention-backend",
                "ascend",
                "--tp-size",
                "4",
                "--moe-dp-size",
                "2",
                "--attn-cp-size",
                "2",
                "--cuda-graph-max-bs",
                "32",
                "--enable-prefill-context-parallel",
            ],
            env=cls.npu_env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 70-73: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process is not None:
            kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 75-91: Run test: gsm8k accuracy
```python
    def test_gsm8k_accuracy(self):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=GSM8K_NUM_QUESTIONS,
            max_new_tokens=512,
            parallel=32,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(
            "GSM8K accuracy "
            f"(mixed TP=4 MOE_DP=2 ATTN_CP=2, {GSM8K_NUM_QUESTIONS} samples): "
            f"{metrics['accuracy']:.3f}"
        )
        self.assertGreaterEqual(metrics["accuracy"], GSM8K_MIN_ACCURACY)
```
**EN:** This test method exercises gsm8k accuracy and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k accuracy 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 94-95: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `types`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
