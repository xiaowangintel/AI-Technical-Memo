# test_llada2_mini.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/dllm/test_llada2_mini.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates llada2 mini behavior in SGLang's dllm area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 dllm 领域中与 llada2 mini 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 3-4: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=139, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=330, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 6-20: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.send_one import BenchArgs, send_one_prompt
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_amd_ci,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `sglang.srt.utils`, `sglang.test.run_eval`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `sglang.srt.utils`, `sglang.test.run_eval`。

### Lines 23-23: class TestLLaDA2Mini declaration / 类 TestLLaDA2Mini 声明
```python
class TestLLaDA2Mini(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 24-53: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "inclusionAI/LLaDA2.0-mini"
        cls.base_url = DEFAULT_URL_FOR_TEST

        other_args = [
            "--trust-remote-code",
            "--tp-size",
            "1",
            "--mem-fraction-static",
            "0.9",
            "--max-running-requests",
            "4",
            "--attention-backend",
            "flashinfer",
            "--dllm-algorithm",
            "LowConfidence",
            "--cuda-graph-bs",
            "1",
            "2",
            "3",
            "4",
        ]

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 55-57: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 59-76: test case gsm8k / 测试用例 gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=200,
            num_threads=128,
        )
        metrics = run_eval(args)
        print(f"{metrics=}")

        self.assertGreater(metrics["score"], 0.88)
        if is_in_amd_ci():
            self.assertGreater(metrics["output_throughput"], 80)
        else:
            self.assertGreater(metrics["output_throughput"], 350)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 78-92: test case bs 1 speed / 测试用例 bs 1 speed
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
            if is_in_amd_ci():
                self.assertGreater(speed, 10)
            else:
                self.assertGreater(speed, 250)
```
**EN:** This test exercises `test_bs_1_speed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bs_1_speed`。

### Lines 95-96: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestLLaDA2Mini`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLLaDA2Mini.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLLaDA2Mini.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestLLaDA2Mini.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `TestLLaDA2Mini.test_bs_1_speed`: This test exercises `test_bs_1_speed` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bs_1_speed`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.send_one`, `sglang.test.test_utils`

- **Total lines / 总行数**: 96
