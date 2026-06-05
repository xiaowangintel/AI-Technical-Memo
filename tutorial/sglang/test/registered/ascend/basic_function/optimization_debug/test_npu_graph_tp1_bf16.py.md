# test_npu_graph_tp1_bf16.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/optimization_debug/test_npu_graph_tp1_bf16.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on optimization debug npu graph tp1 bf16 in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 optimization debug npu graph tp1 bf16 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies
```python
import unittest
from types import SimpleNamespace
from urllib.parse import urlparse

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.few_shot_gsm8k import run_eval as run_eval_few_shot_gsm8k
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 15-16: Register CI metadata
```python
register_npu_ci(est_time=400, suite="stage-b-test-1-npu-a2", nightly=False)
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 18-24: Define module constants
```python
TEST_MODEL_MATRIX = {
    "/root/.cache/modelscope/hub/models/Qwen/Qwen2.5-7B-Instruct": {
        "accuracy": 0.85,
        "latency": 150,
        "output_throughput": 30,
    },
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 27-27: Define class TestAscendGraphTp1Bf16
```python
class TestAscendGraphTp1Bf16(CustomTestCase):
```
**EN:** This declaration introduces the `TestAscendGraphTp1Bf16` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestAscendGraphTp1Bf16` 测试类，并说明它通过继承承担的职责。

### Lines 29-40: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.models = TEST_MODEL_MATRIX.keys()
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.url = urlparse(DEFAULT_URL_FOR_TEST)
        cls.common_args = [
            "--trust-remote-code",
            "--mem-fraction-static",
            0.8,
            "--attention-backend",
            "ascend",
        ]
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 42-73: Run test: a gsm8k
```python
    def test_a_gsm8k(self):
        for model in self.models:
            with self.subTest(model=model):
                print(f"##=== Testing accuracy: {model} ===##")

                process = popen_launch_server(
                    model,
                    self.base_url,
                    timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                    other_args=[
                        *self.common_args,
                    ],
                )

                try:
                    args = SimpleNamespace(
                        num_shots=5,
                        data_path=None,
                        num_questions=1319,
                        max_new_tokens=512,
                        parallel=128,
                        host=f"http://{self.url.hostname}",
                        port=int(self.url.port),
                    )

                    metrics = run_eval_few_shot_gsm8k(args)
                    self.assertGreaterEqual(
                        metrics["accuracy"],
                        TEST_MODEL_MATRIX[model]["accuracy"],
                    )
                finally:
                    kill_process_tree(process.pid)
```
**EN:** This test method exercises a gsm8k and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 a gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 76-77: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `types`, `unittest`, `urllib.parse`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
