# test_npu_piecewise_graph_prefill.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/optimization_debug/test_npu_piecewise_graph_prefill.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on optimization debug npu piecewise graph prefill in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 optimization debug npu piecewise graph prefill 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies
```python
import subprocess
import unittest

from sglang.test.ascend.gsm8k_ascend_mixin import GSM8KAscendMixin
from sglang.test.ascend.test_ascend_utils import (
    QWEN2_5_7B_INSTRUCT_WEIGHTS_PATH,
    write_results_to_github_step_summary,
)
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import (
    CustomTestCase,
    run_bench_one_batch,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 15-16: Register CI metadata
```python
register_npu_ci(est_time=400, suite="stage-b-test-1-npu-a2", nightly=False)
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 19-19: Define module constants
```python
TOKENS_TO_CAPTURE = [i for i in range(128, 4096, 128)]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 22-22: Define class TestPiecewiseGraphPrefillCorrectness
```python
class TestPiecewiseGraphPrefillCorrectness(GSM8KAscendMixin, CustomTestCase):
```
**EN:** This declaration introduces the `TestPiecewiseGraphPrefillCorrectness` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPiecewiseGraphPrefillCorrectness` 测试类，并说明它通过继承承担的职责。

### Lines 23-37: Declare TestPiecewiseGraphPrefillCorrectness configuration
```python
    model = QWEN2_5_7B_INSTRUCT_WEIGHTS_PATH
    other_args = [
        "--trust-remote-code",
        "--mem-fraction-static",
        0.8,
        "--attention-backend",
        "ascend",
        "--cuda-graph-bs",
        128,
        "--enforce-piecewise-cuda-graph",
        "--piecewise-cuda-graph-tokens",
        *TOKENS_TO_CAPTURE,
    ]
    accuracy = 0.84
    num_questions = 1319
```
**EN:** This block defines class-level settings that are shared across the `TestPiecewiseGraphPrefillCorrectness` test methods.
**CN:** 该代码块定义了 `TestPiecewiseGraphPrefillCorrectness` 各测试方法共享的类级配置。

### Lines 40-40: Define class TestPiecewiseGraphPrefillBenchmark
```python
class TestPiecewiseGraphPrefillBenchmark(CustomTestCase):
```
**EN:** This declaration introduces the `TestPiecewiseGraphPrefillBenchmark` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPiecewiseGraphPrefillBenchmark` 测试类，并说明它通过继承承担的职责。

### Lines 41-52: Declare TestPiecewiseGraphPrefillBenchmark configuration
```python
    model = QWEN2_5_7B_INSTRUCT_WEIGHTS_PATH
    other_args = [
        "--trust-remote-code",
        "--mem-fraction-static",
        0.8,
        "--attention-backend",
        "ascend",
        "--enforce-piecewise-cuda-graph",
        "--piecewise-cuda-graph-tokens",
    ] + TOKENS_TO_CAPTURE

    latency = 0.045
```
**EN:** This block defines class-level settings that are shared across the `TestPiecewiseGraphPrefillBenchmark` test methods.
**CN:** 该代码块定义了 `TestPiecewiseGraphPrefillBenchmark` 各测试方法共享的类级配置。

### Lines 54-73: Run test: latency
```python
    def test_latency(self):
        print(f"##=== Testing prefill latency: {self.model} ===##")
        model_metrics = {
            "server": subprocess.list2cmdline(map(str, self.other_args)),
            "client": "bench_one_batch",
            "latency_threshold": self.latency,
        }
        try:
            prefill_latency, _, _ = run_bench_one_batch(
                self.model,
                other_args=self.other_args,
            )
            model_metrics["latency"] = float(prefill_latency)
            self.assertLess(prefill_latency, self.latency)
        except Exception as e:
            model_metrics["error"] = e
            print(f"Error testing {self.model}: {e}")
            self.fail(f"Test failed for {self.model}: {e}")
        finally:
            write_results_to_github_step_summary({self.model: model_metrics})
```
**EN:** This test method exercises latency and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 latency 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

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
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ascend.gsm8k_ascend_mixin`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `subprocess`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
