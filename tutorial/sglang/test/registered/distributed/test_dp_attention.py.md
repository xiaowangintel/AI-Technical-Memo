# test_dp_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_dp_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates dp attention behavior in SGLang's distributed area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 分布式 领域中与 dp attention 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: module imports and dependencies / 模块导入与依赖
```python
import unittest

import requests

from sglang.lang.chat_template import get_chat_template_by_model_path
from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.ebnf_constrained_kit import EBNFConstrainedMixin
from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.kits.json_constrained_kit import JSONConstrainedMixin
from sglang.test.kits.radix_cache_server_kit import run_radix_attention_test
from sglang.test.kits.regex_constrained_kit import RegexConstrainedMixin
from sglang.test.test_utils import (
    DEFAULT_IMAGE_URL,
    DEFAULT_MLA_MODEL_NAME_FOR_TEST,
    DEFAULT_MODEL_NAME_FOR_TEST_MLA,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `requests`, `sglang.lang.chat_template`, `sglang.srt.environ`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `requests`, `sglang.lang.chat_template`, `sglang.srt.environ`。

### Lines 24-24: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=420, stage="base-b", runner_config="2-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 27-33: class TestDPAttentionDP2TP2 declaration / 类 TestDPAttentionDP2TP2 声明
```python
class TestDPAttentionDP2TP2(
    CustomTestCase,
    GSM8KMixin,
    JSONConstrainedMixin,
    EBNFConstrainedMixin,
    RegexConstrainedMixin,
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `GSM8KMixin`, `JSONConstrainedMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `GSM8KMixin`, `JSONConstrainedMixin`。

### Lines 34-34: class-level constants and configuration for `TestDPAttentionDP2TP2` / 类级常量与配置
```python
    gsm8k_accuracy_thres = 0.6
```
**EN:** This block defines shared names such as `gsm8k_accuracy_thres`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `gsm8k_accuracy_thres` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 36-59: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST_MLA
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls._env_override = envs.SGLANG_DISABLE_CONSECUTIVE_PREFILL_OVERLAP.override(
            True
        )
        cls._env_override.__enter__()
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--trust-remote-code",
                "--tp",
                "2",
                "--enable-dp-attention",
                "--dp",
                "2",
                "--enable-torch-compile",
                "--torch-compile-max-bs",
                "2",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 61-64: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        cls._env_override.__exit__(None, None, None)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 67-70: class TestDPAttentionMixedChunk declaration / 类 TestDPAttentionMixedChunk 声明
```python
class TestDPAttentionMixedChunk(
    CustomTestCase,
    GSM8KMixin,
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `GSM8KMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `GSM8KMixin`。

### Lines 71-71: class-level constants and configuration for `TestDPAttentionMixedChunk` / 类级常量与配置
```python
    gsm8k_accuracy_thres = 0.6
```
**EN:** This block defines shared names such as `gsm8k_accuracy_thres`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `gsm8k_accuracy_thres` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 73-92: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MLA_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--trust-remote-code",
                "--tp",
                "2",
                "--enable-dp-attention",
                "--dp",
                "2",
                "--enable-mixed-chunk",
                "--chunked-prefill-size",
                "256",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 94-96: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 99-104: class TestDPRetract declaration / 类 TestDPRetract 声明
```python
class TestDPRetract(
    CustomTestCase,
    JSONConstrainedMixin,
    EBNFConstrainedMixin,
    RegexConstrainedMixin,
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `JSONConstrainedMixin`, `EBNFConstrainedMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `JSONConstrainedMixin`, `EBNFConstrainedMixin`。

### Lines 105-127: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MLA_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--trust-remote-code",
                "--tp",
                "2",
                "--enable-dp-attention",
                "--dp",
                "2",
                "--max-total-tokens",
                "4500",
                "--max-running-requests",
                "128",
                "--chunked-prefill-size",
                "256",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 129-131: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 133-136: test case radix attention / 测试用例 radix attention
```python
    def test_radix_attention(self):
        with envs.SGLANG_TEST_RETRACT.override(True):
            run_radix_attention_test(self.base_url)
            self.assertIsNone(self.process.poll())
```
**EN:** This test exercises `test_radix_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_radix_attention`。

### Lines 139-139: class TestDPAttentionDP2TP2VLM declaration / 类 TestDPAttentionDP2TP2VLM 声明
```python
class TestDPAttentionDP2TP2VLM(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 140-157: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "moonshotai/Kimi-VL-A3B-Instruct"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.image_url = DEFAULT_IMAGE_URL
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--trust-remote-code",
                "--tp",
                "2",
                "--enable-dp-attention",
                "--dp",
                "2",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 159-161: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 163-181: test case vlm generate / 测试用例 vlm generate
```python
    def test_vlm_generate(self):
        chat_template = get_chat_template_by_model_path(self.model)
        prompt = f"{chat_template.image_token}What is in this image?"
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": prompt,
                "image_data": [self.image_url],
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 16,
                },
            },
        )
        response.raise_for_status()
        response_json = response.json()
        print(response_json)
        self.assertIn("output_ids", response_json)
        self.assertGreater(len(response_json["output_ids"]), 0)
```
**EN:** This test exercises `test_vlm_generate` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_vlm_generate`。

### Lines 184-185: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestDPAttentionDP2TP2`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDPAttentionMixedChunk`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDPRetract`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDPAttentionDP2TP2VLM`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDPAttentionDP2TP2.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDPAttentionDP2TP2.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestDPAttentionMixedChunk.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDPAttentionMixedChunk.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestDPRetract.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDPRetract.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestDPRetract.test_radix_attention`: This test exercises `test_radix_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_radix_attention`。
- `TestDPAttentionDP2TP2VLM.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.lang.chat_template`, `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.ebnf_constrained_kit`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.kits.json_constrained_kit`, `sglang.test.kits.radix_cache_server_kit`, `sglang.test.kits.regex_constrained_kit`, `sglang.test.test_utils`

- **Total lines / 总行数**: 185
