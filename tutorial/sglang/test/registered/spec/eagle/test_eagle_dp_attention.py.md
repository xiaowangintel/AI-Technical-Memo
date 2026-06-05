# test_eagle_dp_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/eagle/test_eagle_dp_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates eagle dp attention behavior in SGLang's spec / eagle area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 / eagle 领域中与 eagle dp attention 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.environ import envs
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.send_one import BenchArgs, send_one_prompt
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE_DP_ATTN,
    DEFAULT_TARGET_MODEL_EAGLE_DP_ATTN,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_amd_ci,
    is_in_ci,
    kill_process_tree,
    popen_launch_server,
    write_github_step_summary,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `requests`, `sglang.srt.environ`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `requests`, `sglang.srt.environ`。

### Lines 22-26: supporting source context / 辅助源码上下文
```python

# EAGLE3 with DP attention (tp=2, dp=2, requires 4 GPUs).
# Per-commit EAGLE + DP-attn coverage on CUDA is provided by
# test_eagle_infer_beta_dp_attention.py (B200 4-gpu), so this H100 variant
# is gated to extra-b only.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 27-28: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=99, stage="extra-b", runner_config="4-gpu-h100")
register_amd_ci(est_time=200, suite="stage-c-test-4-gpu-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 31-31: class TestEAGLE3EngineDPAttention declaration / 类 TestEAGLE3EngineDPAttention 声明
```python
class TestEAGLE3EngineDPAttention(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 32-72: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_TARGET_MODEL_EAGLE_DP_ATTN
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--speculative-algorithm",
            "EAGLE3",
            "--speculative-num-steps",
            "6",
            "--speculative-eagle-topk",
            "10",
            "--speculative-num-draft-tokens",
            "32",
            "--speculative-draft-model-path",
            DEFAULT_DRAFT_MODEL_EAGLE_DP_ATTN,
            "--tp-size",
            "2",
            "--dp-size",
            "2",
            "--enable-dp-attention",
            "--enable-dp-lm-head",
            "--moe-dense-tp-size",
            "1",
            "--attention-backend",
            "triton" if is_in_amd_ci() else "fa3",
            "--mem-fraction-static",
            "0.75",
            "--cuda-graph-max-bs",
            "64",
        ]
        with (
            envs.SGLANG_SPEC_NAN_DETECTION.override(True),
            envs.SGLANG_SPEC_OOB_DETECTION.override(True),
        ):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=other_args,
            )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 74-76: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 78-124: test case a gsm8k / 测试用例 a gsm8k
```python
    def test_a_gsm8k(self):
        """Test GSM8K evaluation - append 'a' to run first alphabetically"""
        requests.get(self.base_url + "/flush_cache")

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

        server_info = requests.get(self.base_url + "/server_info")
        server_data = server_info.json()

        # Try to get avg_spec_accept_length
        avg_spec_accept_length = None
        if "internal_states" in server_data and len(server_data["internal_states"]) > 0:
            internal_state = server_data["internal_states"][0]
            if "avg_spec_accept_length" in internal_state:
                avg_spec_accept_length = internal_state["avg_spec_accept_length"]
            elif "spec_accept_length" in internal_state:
                avg_spec_accept_length = internal_state["spec_accept_length"]

        print(f"{avg_spec_accept_length=}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_gsm8k (EAGLE3 DP Attention)\n"
                f'{metrics["score"]=:.3f}\n'
                f"{avg_spec_accept_length=:.2f}\n"
            )
            if is_in_amd_ci():
                # AMD triton backend produces slightly lower accuracy than FA3 on NVIDIA
                self.assertGreater(metrics["score"], 0.88)
            else:
                self.assertGreater(metrics["score"], 0.91)
            if avg_spec_accept_length is not None:
                if is_in_amd_ci():
                    # AMD triton backend produces slightly lower accept length than FA3 on NVIDIA
                    self.assertGreater(avg_spec_accept_length, 2.0)
                else:
                    self.assertGreater(avg_spec_accept_length, 2.5)
```
**EN:** Test GSM8K evaluation - append 'a' to run first alphabetically This test exercises `test_a_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test GSM8K evaluation - append 'a' to run first alphabetically 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_a_gsm8k`。

### Lines 126-146: test case bs 1 speed / 测试用例 bs 1 speed
```python
    def test_bs_1_speed(self):
        """Test batch size 1 speed with EAGLE3 DP Attention"""
        args = BenchArgs(port=int(self.base_url.split(":")[-1]), max_new_tokens=2048)
        acc_length, speed = send_one_prompt(args)

        print(f"{acc_length=:.2f} {speed=:.2f}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_bs_1_speed (EAGLE3 DP Attention)\n"
                f"{acc_length=:.2f}\n"
                f"{speed=:.2f} token/s\n"
            )
            if is_in_amd_ci():
                self.assertGreater(acc_length, 2.0)
            else:
                self.assertGreater(acc_length, 2.3)
            if is_in_amd_ci():
                self.assertGreater(speed, 10)
            else:
                self.assertGreater(speed, 40)
```
**EN:** Test batch size 1 speed with EAGLE3 DP Attention This test exercises `test_bs_1_speed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test batch size 1 speed with EAGLE3 DP Attention 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bs_1_speed`。

### Lines 149-150: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEAGLE3EngineDPAttention`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLE3EngineDPAttention.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestEAGLE3EngineDPAttention.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestEAGLE3EngineDPAttention.test_a_gsm8k`: Test GSM8K evaluation - append 'a' to run first alphabetically / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_a_gsm8k`。
- `TestEAGLE3EngineDPAttention.test_bs_1_speed`: Test batch size 1 speed with EAGLE3 DP Attention / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bs_1_speed`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.send_one`, `sglang.test.test_utils`

- **Total lines / 总行数**: 150
