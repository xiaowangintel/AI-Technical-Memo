# test_dflash.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/dflash/test_dflash.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates dflash behavior in SGLang's spec / dflash area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 / dflash 领域中与 dflash 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: module imports and dependencies / 模块导入与依赖
```python
import os
import unittest

import openai

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.kits.matched_stop_kit import MatchedStopMixin
from sglang.test.kits.radix_cache_server_kit import gen_radix_tree
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_DFLASH,
    DEFAULT_TARGET_MODEL_DFLASH,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `unittest`, `openai`, `sglang.srt.environ`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `unittest`, `openai`, `sglang.srt.environ`。

### Lines 21-21: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=302, stage="base-b", runner_config="1-gpu-small")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-24: class TestDFlashServerBase declaration / 类 TestDFlashServerBase 声明
```python
class TestDFlashServerBase(CustomTestCase, MatchedStopMixin, GSM8KMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `MatchedStopMixin`, `GSM8KMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `MatchedStopMixin`, `GSM8KMixin`。

### Lines 25-32: class-level constants and configuration for `TestDFlashServerBase` / 类级常量与配置
```python
    max_running_requests = 64
    attention_backend = "flashinfer"
    page_size = 1
    other_launch_args = []
    model = DEFAULT_TARGET_MODEL_DFLASH
    draft_model = DEFAULT_DRAFT_MODEL_DFLASH
    gsm8k_accuracy_thres = 0.75
    gsm8k_accept_length_thres = 2.8
```
**EN:** This block defines shared names such as `max_running_requests`, `attention_backend`, `page_size`, `other_launch_args`, `model`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `max_running_requests`, `attention_backend`, `page_size`, `other_launch_args`, `model` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 34-71: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        launch_args = [
            "--trust-remote-code",
            "--attention-backend",
            cls.attention_backend,
            "--speculative-algorithm",
            "DFLASH",
            "--speculative-draft-model-path",
            cls.draft_model,
            "--page-size",
            str(cls.page_size),
            "--max-running-requests",
            str(cls.max_running_requests),
            "--cuda-graph-bs",
            *[str(i) for i in range(1, cls.max_running_requests + 1)],
        ]
        launch_args.extend(cls.other_launch_args)
        old_value = os.environ.get("SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN")
        os.environ["SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN"] = "1"
        try:
            with (
                envs.SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_BUSY.override(1),
                envs.SGLANG_SPEC_NAN_DETECTION.override(True),
                envs.SGLANG_SPEC_OOB_DETECTION.override(True),
            ):
                cls.process = popen_launch_server(
                    cls.model,
                    cls.base_url,
                    timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                    other_args=launch_args,
                )
        finally:
            if old_value is None:
                del os.environ["SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN"]
            else:
                os.environ["SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN"] = old_value
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 73-75: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 77-89: test case early stop / 测试用例 early stop
```python
    def test_early_stop(self):
        client = openai.Client(base_url=self.base_url + "/v1", api_key="EMPTY")
        for i in range(8):
            max_tokens = (i % 3) + 1
            response = client.completions.create(
                model=self.model,
                prompt=f"There are {i} apples on the table. How to divide them equally?",
                max_tokens=max_tokens,
                temperature=0,
            )
            text = response.choices[0].text
            print(f"early_stop: max_tokens={max_tokens}, text={text!r}")
        assert self.process.poll() is None
```
**EN:** This test exercises `test_early_stop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_early_stop`。

### Lines 91-103: test case eos handling / 测试用例 eos handling
```python
    def test_eos_handling(self):
        client = openai.Client(base_url=self.base_url + "/v1", api_key="EMPTY")
        response = client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": "Today is a sunny day and I like"}],
            max_tokens=256,
            temperature=0.1,
        )
        text = response.choices[0].message.content
        print(f"eos_handling: text={text!r}")
        self.assertNotIn("<|eot_id|>", text)
        self.assertNotIn("<|end_of_text|>", text)
        assert self.process.poll() is None
```
**EN:** This test exercises `test_eos_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_eos_handling`。

### Lines 105-119: test case greedy determinism / 测试用例 greedy determinism
```python
    def test_greedy_determinism(self):
        client = openai.Client(base_url=self.base_url + "/v1", api_key="EMPTY")
        prompt = "The capital of France is"
        outputs = []
        for _ in range(2):
            response = client.completions.create(
                model=self.model,
                prompt=prompt,
                max_tokens=32,
                temperature=0,
            )
            outputs.append(response.choices[0].text)
        print(f"determinism: {outputs=}")
        self.assertEqual(outputs[0], outputs[1])
        assert self.process.poll() is None
```
**EN:** This test exercises `test_greedy_determinism` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_greedy_determinism`。

### Lines 122-122: class TestDFlashServerPage256 declaration / 类 TestDFlashServerPage256 声明
```python
class TestDFlashServerPage256(TestDFlashServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestDFlashServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestDFlashServerBase`。

### Lines 123-123: class-level constants and configuration for `TestDFlashServerPage256` / 类级常量与配置
```python
    page_size = 256
```
**EN:** This block defines shared names such as `page_size`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `page_size` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 125-138: test case radix attention / 测试用例 radix attention
```python
    def test_radix_attention(self):
        import requests

        nodes = gen_radix_tree(num_nodes=50)
        data = {
            "input_ids": [node["input_ids"] for node in nodes],
            "sampling_params": [
                {"max_new_tokens": node["decode_len"], "temperature": 0}
                for node in nodes
            ],
        }
        res = requests.post(self.base_url + "/generate", json=data)
        assert res.status_code == 200
        assert self.process.poll() is None
```
**EN:** This test exercises `test_radix_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_radix_attention`。

### Lines 141-141: class TestDFlashServerChunkedPrefill declaration / 类 TestDFlashServerChunkedPrefill 声明
```python
class TestDFlashServerChunkedPrefill(TestDFlashServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestDFlashServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestDFlashServerBase`。

### Lines 142-142: class-level constants and configuration for `TestDFlashServerChunkedPrefill` / 类级常量与配置
```python
    other_launch_args = ["--chunked-prefill-size", "4"]
```
**EN:** This block defines shared names such as `other_launch_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `other_launch_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 145-145: class TestDFlashServerNoCudaGraph declaration / 类 TestDFlashServerNoCudaGraph 声明
```python
class TestDFlashServerNoCudaGraph(TestDFlashServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestDFlashServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestDFlashServerBase`。

### Lines 146-146: class-level constants and configuration for `TestDFlashServerNoCudaGraph` / 类级常量与配置
```python
    other_launch_args = ["--disable-cuda-graph"]
```
**EN:** This block defines shared names such as `other_launch_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `other_launch_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 149-150: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestDFlashServerBase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDFlashServerPage256`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDFlashServerChunkedPrefill`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDFlashServerNoCudaGraph`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDFlashServerBase.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDFlashServerBase.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestDFlashServerBase.test_early_stop`: This test exercises `test_early_stop` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_early_stop`。
- `TestDFlashServerBase.test_eos_handling`: This test exercises `test_eos_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_eos_handling`。
- `TestDFlashServerBase.test_greedy_determinism`: This test exercises `test_greedy_determinism` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_greedy_determinism`。
- `TestDFlashServerPage256.test_radix_attention`: This test exercises `test_radix_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_radix_attention`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `unittest`
- **Third-party modules / 第三方模块**: `openai`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.kits.matched_stop_kit`, `sglang.test.kits.radix_cache_server_kit`, `sglang.test.test_utils`

- **Total lines / 总行数**: 150
