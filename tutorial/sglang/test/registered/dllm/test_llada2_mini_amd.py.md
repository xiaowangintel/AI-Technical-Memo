# test_llada2_mini_amd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/dllm/test_llada2_mini_amd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates llada2 mini amd behavior in SGLang's dllm area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 dllm 领域中与 llada2 mini amd 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: supporting statements / 辅助语句
```python
"""
Test LLaDA2 (Diffusion Language Model) on AMD GPUs.

This test verifies that DLLM works on AMD with triton attention backend.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 7-21: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.run_eval import run_eval
from sglang.test.send_one import BenchArgs, send_one_prompt
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `sglang.srt.utils`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `sglang.srt.utils`, `sglang.test.ci.ci_register`。

### Lines 23-23: CI registration and metadata / CI 注册与元数据
```python
register_amd_ci(est_time=1000, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_amd_ci.
**CN:** 该代码块通过 register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 26-26: class TestLLaDA2MiniAMD declaration / 类 TestLLaDA2MiniAMD 声明
```python
class TestLLaDA2MiniAMD(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 27-49: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "inclusionAI/LLaDA2.0-mini"
        cls.base_url = DEFAULT_URL_FOR_TEST

        other_args = [
            "--trust-remote-code",
            "--mem-fraction-static",
            "0.9",
            "--max-running-requests",
            "1",
            "--attention-backend",
            "triton",  # Use triton for AMD instead of flashinfer
            "--dllm-algorithm",
            "LowConfidence",
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

### Lines 51-53: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 55-69: test case gsm8k / 测试用例 gsm8k
```python
    def test_gsm8k(self):
        """Test GSM8K accuracy with DLLM on AMD."""
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            num_examples=200,
            num_threads=128,
        )
        metrics = run_eval(args)
        print(f"{metrics=}")

        # Relaxed thresholds for AMD - may need adjustment
        self.assertGreater(metrics["score"], 0.80)
        self.assertGreater(metrics["output_throughput"], 50)
```
**EN:** Test GSM8K accuracy with DLLM on AMD. This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test GSM8K accuracy with DLLM on AMD. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 71-84: test case bs 1 speed / 测试用例 bs 1 speed
```python
    def test_bs_1_speed(self):
        """Test single batch inference speed."""
        args = BenchArgs(port=int(self.base_url.split(":")[-1]), max_new_tokens=2048)
        acc_length, speed = send_one_prompt(args)

        print(f"{speed=:.2f}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_bs_1_speed (llada2-mini AMD) with tp1\n"
                f"{speed=:.2f} token/s\n"
            )
            # Relaxed threshold for AMD
            self.assertGreater(speed, 10)
```
**EN:** Test single batch inference speed. This test exercises `test_bs_1_speed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test single batch inference speed. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bs_1_speed`。

### Lines 87-88: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestLLaDA2MiniAMD`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLLaDA2MiniAMD.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLLaDA2MiniAMD.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestLLaDA2MiniAMD.test_gsm8k`: Test GSM8K accuracy with DLLM on AMD. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `TestLLaDA2MiniAMD.test_bs_1_speed`: Test single batch inference speed. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bs_1_speed`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.send_one`, `sglang.test.test_utils`

- **Total lines / 总行数**: 88
