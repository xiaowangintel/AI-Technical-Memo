# test_npu_llada2_mini.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/dllm/test_npu_llada2_mini.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on dllm npu llada2 mini in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 dllm npu llada2 mini 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Import dependencies
```python
import os
import unittest

from sglang.test.ascend.gsm8k_ascend_mixin import GSM8KAscendMixin
from sglang.test.ascend.test_ascend_utils import LLaDA2_0_MINI_WEIGHTS_PATH
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.send_one import BenchArgs, send_one_prompt
from sglang.test.test_utils import (
    CustomTestCase,
    is_in_ci,
    write_github_step_summary,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 14-15: Register CI metadata
```python
register_npu_ci(est_time=400, suite="stage-b-test-4-npu-a3", nightly=False)
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 18-18: Define class TestLLaDA2Mini
```python
class TestLLaDA2Mini(GSM8KAscendMixin, CustomTestCase):
```
**EN:** This declaration introduces the `TestLLaDA2Mini` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLLaDA2Mini` 测试类，并说明它通过继承承担的职责。

### Lines 19-38: Declare TestLLaDA2Mini configuration
```python
    model = LLaDA2_0_MINI_WEIGHTS_PATH

    other_args = [
        "--trust-remote-code",
        "--disable-radix-cache",
        "--mem-fraction-static",
        "0.9",
        "--max-running-requests",
        "1",
        "--attention-backend",
        "ascend",
        "--dllm-algorithm",
        "LowConfidence",  # TODO: Add dLLM configurations
    ]
    env = {
        **os.environ,
        "SGLANG_NPU_DISABLE_ACL_FORMAT_WEIGHT": "1",  # Need to avoid OOM issue
    }
    accuracy = 0.88
    output_throughput = 70
```
**EN:** This block defines class-level settings that are shared across the `TestLLaDA2Mini` test methods.
**CN:** 该代码块定义了 `TestLLaDA2Mini` 各测试方法共享的类级配置。

### Lines 40-51: Run test: bs 1 speed
```python
    def test_bs_1_speed(self):
        args = BenchArgs(port=int(self.base_url.split(":")[-1]), max_new_tokens=2048)
        acc_length, speed = send_one_prompt(args)

        print(f"{speed=:.2f}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_bs_1_speed (llada2-mini) with tp1\n"
                f"{speed=:.2f} token/s\n"
            )
            self.assertGreater(speed, 130)
```
**EN:** This test method exercises bs 1 speed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bs 1 speed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 54-55: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ascend.gsm8k_ascend_mixin`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.send_one`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
