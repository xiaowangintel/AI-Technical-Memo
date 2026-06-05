# test_deepseek_v3_fp4_cutedsl_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/backends/test_deepseek_v3_fp4_cutedsl_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on backends deepseek v3 fp4 cutedsl moe in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 backends deepseek v3 fp4 cutedsl moe 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Document the module
```python
"""Backend tests for CuteDSL MoE (FusedMoE + moe_runner, moe_a2a=none).

Exercises the CuteDSL moe_runner path with ModelOpt FP4 by launching a
server with --moe-runner-backend flashinfer_cutedsl.

Two configurations are tested:
  - EP=1, TP=4: each GPU holds all experts with TP-sharded intermediate dim
  - EP=4, TP=4: each GPU holds 1/4 of experts at full intermediate width,
    partial results combined via all-reduce (no A2A dispatch)

Requires 4 GPUs. Run from repo root with:
  python -m pytest test/registered/backends/test_deepseek_v3_fp4_cutedsl_moe.py -v -s
Or via the nightly suite:
  python test/run_suite.py --hw cuda --suite nightly-4-gpu-b200 --nightly
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 17-29: Import dependencies
```python
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
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

### Lines 31-31: Register CI metadata
```python
register_cuda_ci(est_time=900, suite="nightly-4-gpu-b200", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 33-35: Define module constants
```python
FULL_DEEPSEEK_V3_FP4_MODEL_PATH = "nvidia/DeepSeek-V3-0324-FP4"
SERVER_LAUNCH_TIMEOUT = 1000
GSM8K_ACCURACY_THRESHOLD = 0.935
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 38-38: Define class TestDeepseekV3FP4CuteDSLMoE
```python
class TestDeepseekV3FP4CuteDSLMoE(CustomTestCase):
```
**EN:** This declaration introduces the `TestDeepseekV3FP4CuteDSLMoE` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDeepseekV3FP4CuteDSLMoE` 测试类，并说明它通过继承承担的职责。

### Lines 39-39: Document the class `TestDeepseekV3FP4CuteDSLMoE`
```python
    """CuteDSL standard moe_runner path: flashinfer_cutedsl + modelopt_fp4, EP=1."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDeepseekV3FP4CuteDSLMoE`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDeepseekV3FP4CuteDSLMoE`的设计意图。

### Lines 41-66: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = FULL_DEEPSEEK_V3_FP4_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--tp",
            "4",
            "--ep",
            "1",
            "--mem-fraction-static",
            "0.75",
            "--attention-backend",
            "trtllm_mla",
            "--moe-runner-backend",
            "flashinfer_cutedsl",
            "--quantization",
            "modelopt_fp4",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true}',
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

### Lines 68-70: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 72-90: Run test: a gsm8k
```python
    def test_a_gsm8k(
        self,
    ):  # Append an "a" to make this test run first (alphabetically) to warm up the server
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
        if is_in_ci():
            write_github_step_summary(
                f"### test_gsm8k (deepseek-v3-fp4-cutedsl-moe)\n"
                f'{metrics["accuracy"]=:.3f}\n'
            )
        self.assertGreater(metrics["accuracy"], GSM8K_ACCURACY_THRESHOLD)
```
**EN:** This test method exercises a gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 a gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 93-93: Define class TestDeepseekV3FP4CuteDSLMoEEP4
```python
class TestDeepseekV3FP4CuteDSLMoEEP4(CustomTestCase):
```
**EN:** This declaration introduces the `TestDeepseekV3FP4CuteDSLMoEEP4` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDeepseekV3FP4CuteDSLMoEEP4` 测试类，并说明它通过继承承担的职责。

### Lines 94-94: Document the class `TestDeepseekV3FP4CuteDSLMoEEP4`
```python
    """CuteDSL standard moe_runner path: flashinfer_cutedsl + modelopt_fp4, EP=TP=4."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDeepseekV3FP4CuteDSLMoEEP4`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDeepseekV3FP4CuteDSLMoEEP4`的设计意图。

### Lines 96-123: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = FULL_DEEPSEEK_V3_FP4_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--tp",
            "4",
            "--ep",
            "4",
            "--mem-fraction-static",
            "0.75",
            "--attention-backend",
            "trtllm_mla",
            "--moe-runner-backend",
            "flashinfer_cutedsl",
            "--moe-a2a-backend",
            "none",
            "--quantization",
            "modelopt_fp4",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true}',
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

### Lines 125-127: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 129-145: Run test: a gsm8k
```python
    def test_a_gsm8k(self):
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
        if is_in_ci():
            write_github_step_summary(
                f"### test_gsm8k (deepseek-v3-fp4-cutedsl-moe-ep4)\n"
                f'{metrics["accuracy"]=:.3f}\n'
            )
        self.assertGreater(metrics["accuracy"], GSM8K_ACCURACY_THRESHOLD)
```
**EN:** This test method exercises a gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 a gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 148-149: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`
