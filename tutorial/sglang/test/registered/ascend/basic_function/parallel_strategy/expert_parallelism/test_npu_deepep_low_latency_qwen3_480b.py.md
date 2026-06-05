# test_npu_deepep_low_latency_qwen3_480b.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/parallel_strategy/expert_parallelism/test_npu_deepep_low_latency_qwen3_480b.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on expert parallelism npu deepep low latency qwen3 480b in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 expert parallelism npu deepep low latency qwen3 480b 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Import dependencies
```python
import os
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ascend.test_ascend_utils import (
    QWEN3_CODER_480B_A35B_INSTRUCT_W8A8_QUAROT_WEIGHTS_PATH,
)
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.few_shot_gsm8k import run_eval as run_gsm8k
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

### Lines 19-19: Register CI metadata
```python
register_npu_ci(est_time=200, suite="nightly-16-npu-a3", nightly=False)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 22-22: Define class TestDeepEpQwen
```python
class TestDeepEpQwen(CustomTestCase):
```
**EN:** This declaration introduces the `TestDeepEpQwen` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDeepEpQwen` 测试类，并说明它通过继承承担的职责。

### Lines 23-30: Document the class `TestDeepEpQwen`
```python
    """
    Testcase:Test the Qwen3-Coder-480B-A35B-Instruct-w8a8-QuaRot model with DeepEP's low_latency mode enabled,
    and verify that there is no drop in accuracy compared to when DeepEP is not enabled.

    [Test Category] Expert Parallelism
    [Test Target] --moe-a2a-backend, --deepep-mode
    [Test Suggestions] Mixing deployment + low_latency mode is not recommended.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDeepEpQwen`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDeepEpQwen`的设计意图。

### Lines 32-87: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = QWEN3_CODER_480B_A35B_INSTRUCT_W8A8_QUAROT_WEIGHTS_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--trust-remote-code",
                "--nnodes",
                "1",
                "--node-rank",
                "0",
                "--attention-backend",
                "ascend",
                "--device",
                "npu",
                "--quantization",
                "modelslim",
                "--max-running-requests",
                96,
                "--context-length",
                8192,
                "--dtype",
                "bfloat16",
                "--chunked-prefill-size",
                1024,
                "--max-prefill-tokens",
                458880,
                "--disable-radix-cache",
                "--moe-a2a-backend",
                "deepep",
                "--deepep-mode",
                "low_latency",
                "--tp-size",
                16,
                "--dp-size",
                4,
                "--enable-dp-attention",
                "--enable-dp-lm-head",
                "--mem-fraction-static",
                0.7,
                "--cuda-graph-bs",
                16,
                20,
                24,
            ],
            env={
                "PYTORCH_NPU_ALLOC_CONF": "expandable_segments:True",
                "SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT": "600",
                "HCCL_BUFFSIZE": "2100",
                "HCCL_OP_EXPANSION_MODE": "AIV",
                **os.environ,
            },
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 89-91: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 93-103: Run test: mmlu
```python
    def test_mmlu(self):
        expect_score = 0.61
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="mmlu",
            num_examples=8,
            num_threads=32,
        )
        metrics = run_eval(args)
        self.assertGreater(metrics["score"], expect_score)
```
**EN:** This test method exercises mmlu and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 mmlu 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 105-121: Run test: gsm8k
```python
    def test_gsm8k(self):
        expect_accuracy = 0.91
        args = SimpleNamespace(
            num_shots=8,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_gsm8k(args)
        self.assertGreaterEqual(
            metrics["accuracy"],
            expect_accuracy,
            f'Accuracy of {self.model} is {str(metrics["accuracy"])}, is lower than {expect_accuracy}',
        )
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions and packages launch or evaluation arguments into a lightweight namespace.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期，并把启动或评测参数封装到轻量级命名空间中。

### Lines 124-125: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.run_eval`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `types`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
