# test_flashmla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/mla/test_flashmla.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates flashmla behavior in SGLang's mla area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 mla 领域中与 flashmla 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: supporting statements / 辅助语句
```python
"""
Usage:
python3 test/registered/mla/test_flashmla.py
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 6-21: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace

import requests
import torch

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `requests`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `requests`, `torch`。

### Lines 22-23: supporting source context / 辅助源码上下文
```python

# FlashMLA attention backend tests with MTP speculative decoding
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 24-24: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=160, stage="base-b", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 27-27: class TestFlashMLAMTP declaration / 类 TestFlashMLAMTP 声明
```python
class TestFlashMLAMTP(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 28-63: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "lmsys/sglang-ci-dsv3-test"
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = ["--trust-remote-code"]
        if torch.cuda.is_available() and torch.version.cuda:
            other_args.extend(
                [
                    "--cuda-graph-max-bs",
                    "4",
                    "--disable-radix",
                    "--enable-torch-compile",
                    "--torch-compile-max-bs",
                    "1",
                    "--speculative-algorithm",
                    "EAGLE",
                    "--speculative-draft-model-path",
                    "lmsys/sglang-ci-dsv3-test-NextN",
                    "--speculative-num-steps",
                    "2",
                    "--speculative-eagle-topk",
                    "1",
                    "--speculative-num-draft-tokens",
                    "3",
                    "--attention-backend",
                    "flashmla",
                ]
            )
        # Use longer timeout for DeepGEMM JIT compilation which can take 10-20 minutes
        with envs.SGLANG_ENABLE_SPEC_V2.override(False):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 2,
                other_args=other_args,
            )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 65-67: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 69-91: test case gsm8k / 测试用例 gsm8k
```python
    def test_gsm8k(self):
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
        print(metrics)

        self.assertGreater(metrics["score"], 0.60)

        server_info = requests.get(self.base_url + "/server_info").json()
        avg_spec_accept_length = server_info["internal_states"][0][
            "avg_spec_accept_length"
        ]
        print(f"{avg_spec_accept_length=}")
        self.assertGreater(avg_spec_accept_length, 2.4)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 94-95: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestFlashMLAMTP`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFlashMLAMTP.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestFlashMLAMTP.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestFlashMLAMTP.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`
- **Third-party modules / 第三方模块**: `requests`, `torch`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 95
