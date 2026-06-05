# test_deepseek_v3_cutedsl_4gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/4-gpu-models/test_deepseek_v3_cutedsl_4gpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 4 gpu models deepseek v3 cutedsl 4gpu in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 4 gpu models deepseek v3 cutedsl 4gpu 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Import dependencies
```python
import os
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_DEEPSEEK_NVFP4_MODEL_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
    try_cached_model,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 17-17: Register CI metadata
```python
register_cuda_ci(est_time=1800, stage="base-c", runner_config="4-gpu-gb200")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 20-20: Define class TestDeepseekR1Nvfp4CuteDSLDeepEP
```python
class TestDeepseekR1Nvfp4CuteDSLDeepEP(CustomTestCase):
```
**EN:** This declaration introduces the `TestDeepseekR1Nvfp4CuteDSLDeepEP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDeepseekR1Nvfp4CuteDSLDeepEP` 测试类，并说明它通过继承承担的职责。

### Lines 21-68: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = try_cached_model(DEFAULT_DEEPSEEK_NVFP4_MODEL_FOR_TEST)
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--disable-radix-cache",
            "--mem-fraction-static",
            "0.8",
            "--max-prefill-tokens",
            "16384",
            "--max-running-requests",
            "256",
            "--chunked-prefill-size",
            "1024",
            "--tp",
            "4",
            "--dp",
            "4",
            "--ep",
            "4",
            "--moe-dense-tp-size",
            "1",
            "--enable-dp-attention",
            "--quantization",
            "modelopt_fp4",
            "--attention-backend",
            "trtllm_mla",
            "--moe-runner-backend",
            "flashinfer_cutedsl",
            "--moe-a2a-backend",
            "deepep",
            "--deepep-mode",
            "low_latency",
            "--deepep-dispatcher-output-dtype",
            "bf16",
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
            env={
                **os.environ,
                "SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "256",
                "SGLANG_MOE_NVFP4_DISPATCH": "0",
            },
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 70-72: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 74-87: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=512,
            num_threads=512,
        )
        metrics = run_eval(args)
        print(f"Eval accuracy of GSM8K: {metrics=}")

        self.assertGreater(metrics["score"], 0.92)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 90-90: Define class TestDummyWithSBO
```python
class TestDummyWithSBO(CustomTestCase):
```
**EN:** This declaration introduces the `TestDummyWithSBO` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDummyWithSBO` 测试类，并说明它通过继承承担的职责。

### Lines 91-145: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = try_cached_model(DEFAULT_DEEPSEEK_NVFP4_MODEL_FOR_TEST)
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--disable-radix-cache",
            "--mem-fraction-static",
            "0.05",
            "--max-prefill-tokens",
            "16384",
            "--max-running-requests",
            "256",
            "--chunked-prefill-size",
            "1024",
            "--cuda-graph-bs",
            "64",
            "--tp",
            "4",
            "--dp",
            "4",
            "--ep",
            "4",
            "--moe-dense-tp-size",
            "1",
            "--enable-dp-attention",
            "--quantization",
            "modelopt_fp4",
            "--attention-backend",
            "trtllm_mla",
            "--moe-runner-backend",
            "flashinfer_cutedsl",
            "--moe-a2a-backend",
            "deepep",
            "--deepep-mode",
            "low_latency",
            "--deepep-dispatcher-output-dtype",
            "bf16",
            "--json-model-override-args",
            '{"num_hidden_layers": 1, "first_k_dense_replace": 0, "n_routed_experts": 24}',
            "--enable-single-batch-overlap",
            "--load-format",
            "dummy",
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
            env={
                **os.environ,
                "SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "256",
                "SGLANG_MOE_NVFP4_DISPATCH": "0",
            },
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 147-149: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 151-163: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=512,
            num_threads=512,
            num_shots=0,
        )
        metrics = run_eval(args)
        print(f"Eval accuracy of GSM8K: {metrics=}")
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and packages launch or evaluation arguments into a lightweight namespace.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并把启动或评测参数封装到轻量级命名空间中。

### Lines 166-167: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `os`, `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
