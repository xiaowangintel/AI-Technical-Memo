# test_openai_server_hidden_states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/openai_server/features/test_openai_server_hidden_states.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates openai server hidden states behavior in SGLang's openai server / features area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 openai server / features 领域中与 openai server hidden states 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: module imports and dependencies / 模块导入与依赖
```python
import unittest
from abc import ABC

import openai

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TARGET_MODEL_EAGLE,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `abc`, `openai`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `abc`, `openai`, `sglang.srt.utils`。

### Lines 19-24: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=222, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(
    est_time=186,
    suite="stage-b-test-1-gpu-small-amd",
    disabled="see https://github.com/sgl-project/sglang/issues/11127",
)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 27-28: class BaseTestOpenAIServerWithHiddenStates declaration / 类 BaseTestOpenAIServerWithHiddenStates 声明
```python
class BaseTestOpenAIServerWithHiddenStates(ABC):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ABC`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ABC`。

### Lines 29-33: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.return_hidden_states = [False, True]
        cls.use_list_input = [True, False]
        cls.parallel_sample_nums = [1, 2]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 35-43: test case completion / 测试用例 completion
```python
    def test_completion(self):
        for return_hidden_states in self.return_hidden_states:
            for use_list_input in self.use_list_input:
                for parallel_sample_num in self.parallel_sample_nums:
                    self.run_completion(
                        use_list_input,
                        parallel_sample_num,
                        return_hidden_states,
                    )
```
**EN:** This test exercises `test_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion`。

### Lines 45-54: test case completion stream / 测试用例 completion stream
```python
    def test_completion_stream(self):
        # parallel sampling and list input are not supported in streaming mode
        for return_hidden_states in self.return_hidden_states:
            for use_list_input in self.use_list_input:
                for parallel_sample_num in self.parallel_sample_nums:
                    self.run_completion_stream(
                        use_list_input,
                        parallel_sample_num,
                        return_hidden_states,
                    )
```
**EN:** This test exercises `test_completion_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion_stream`。

### Lines 56-63: test case chat completion / 测试用例 chat completion
```python
    def test_chat_completion(self):
        for return_hidden_states in self.return_hidden_states:
            for (
                parallel_sample_num
            ) in (
                self.parallel_sample_nums
            ):  # parallel sample num 2 breaks in the adapter with a 400 for EAGLE
                self.run_chat_completion(parallel_sample_num, return_hidden_states)
```
**EN:** This test exercises `test_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion`。

### Lines 65-74: test case chat completion stream / 测试用例 chat completion stream
```python
    def test_chat_completion_stream(self):
        for return_hidden_states in self.return_hidden_states:
            for (
                parallel_sample_num
            ) in (
                self.parallel_sample_nums
            ):  # parallel sample num > 1 breaks in the adapter with a 400 for EAGLE
                self.run_chat_completion_stream(
                    parallel_sample_num, return_hidden_states
                )
```
**EN:** This test exercises `test_chat_completion_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_stream`。

### Lines 76-105: helper routine run completion / 辅助流程 run completion
```python
    def run_completion(
        self,
        use_list_input,
        parallel_sample_num,
        return_hidden_states,
    ):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        prompt = "The capital of France is"
        prompt_input = prompt

        if use_list_input:
            prompt_arg = [prompt_input, prompt_input]
            num_choices = len(prompt_arg)
        else:
            prompt_arg = prompt_input
            num_choices = 1

        response = client.completions.create(
            model=self.model,
            prompt=prompt_arg,
            temperature=0,
            max_tokens=32,
            n=parallel_sample_num,
            extra_body=dict(return_hidden_states=return_hidden_states),
        )

        for choice in response.choices:
            assert hasattr(choice, "hidden_states") == return_hidden_states
            if return_hidden_states:
                assert choice.hidden_states is not None, "hidden_states was None"
```
**EN:** This helper encapsulates `run_completion` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_completion`，以便周围测试复用准备、执行或校验逻辑。

### Lines 107-153: helper routine run completion stream / 辅助流程 run completion stream
```python
    def run_completion_stream(
        self,
        use_list_input,
        parallel_sample_num,
        return_hidden_states,
    ):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        prompt = "The capital of France is"
        prompt_input = prompt
        num_prompt_tokens = len(self.tokenizer.encode(prompt))

        if use_list_input:
            prompt_arg = [prompt_input, prompt_input]
            num_choices = len(prompt_arg)
            num_prompt_tokens *= 2
        else:
            prompt_arg = prompt_input
            num_choices = 1

        generator = client.completions.create(
            model=self.model,
            prompt=prompt_arg,
            temperature=0,
            max_tokens=32,
            stream=True,
            stream_options={"include_usage": True},
            n=parallel_sample_num,
            extra_body=dict(return_hidden_states=return_hidden_states),
        )

        hidden_states_list = []
        for response in generator:
            usage = response.usage
            for choice in response.choices:
                if hasattr(choice, "hidden_states"):
                    assert return_hidden_states
                    assert choice.hidden_states is not None
                    hidden_states_list.append(choice.hidden_states)

        if return_hidden_states:
            assert (
                len(hidden_states_list) == parallel_sample_num * num_choices
            ), f"Expected {parallel_sample_num * num_choices} hidden states, got {len(hidden_states_list)}"
        else:
            assert (
                hidden_states_list == []
            ), "hidden_states were returned and should not have been"
```
**EN:** This helper encapsulates `run_completion_stream` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_completion_stream`，以便周围测试复用准备、执行或校验逻辑。

### Lines 155-174: helper routine run chat completion / 辅助流程 run chat completion
```python
    def run_chat_completion(self, parallel_sample_num, return_hidden_states):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        response = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {
                    "role": "user",
                    "content": "What is the capital of France? Answer in a few words.",
                },
            ],
            temperature=0,
            n=parallel_sample_num,
            extra_body=dict(return_hidden_states=return_hidden_states),
        )

        for choice in response.choices:
            assert hasattr(choice, "hidden_states") == return_hidden_states
            if return_hidden_states:
                assert choice.hidden_states is not None, "hidden_states was None"
```
**EN:** This helper encapsulates `run_chat_completion` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_chat_completion`，以便周围测试复用准备、执行或校验逻辑。

### Lines 176-210: helper routine run chat completion stream / 辅助流程 run chat completion stream
```python
    def run_chat_completion_stream(
        self, parallel_sample_num=1, return_hidden_states=False
    ):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        generator = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {"role": "user", "content": "What is the capital of France?"},
            ],
            temperature=0,
            stream=True,
            stream_options={"include_usage": True},
            n=parallel_sample_num,
            extra_body=dict(return_hidden_states=return_hidden_states),
        )

        is_firsts = {}
        hidden_states_list = []

        for response in generator:
            for choice in response.choices:
                if hasattr(choice.delta, "hidden_states"):
                    assert return_hidden_states
                    assert choice.delta.hidden_states is not None
                    hidden_states_list.append(choice.delta.hidden_states)

        if return_hidden_states:
            assert (
                len(hidden_states_list) == parallel_sample_num
            ), f"Expected {parallel_sample_num} hidden states, got {len(hidden_states_list)}"
        else:
            assert (
                hidden_states_list == []
            ), "hidden_states were returned and should not have been"
```
**EN:** This helper encapsulates `run_chat_completion_stream` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_chat_completion_stream`，以便周围测试复用准备、执行或校验逻辑。

### Lines 213-215: class TestOpenAIServerWithHiddenStatesEnabled declaration / 类 TestOpenAIServerWithHiddenStatesEnabled 声明
```python
class TestOpenAIServerWithHiddenStatesEnabled(
    CustomTestCase, BaseTestOpenAIServerWithHiddenStates
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `BaseTestOpenAIServerWithHiddenStates`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `BaseTestOpenAIServerWithHiddenStates`。

### Lines 216-232: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=["--enable-return-hidden-states"],
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
        cls.return_hidden_states = [False, True]
        cls.use_list_input = [True, False]
        cls.parallel_sample_nums = [1, 2]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 234-236: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 239-241: class TestOpenAIServerWithHiddenStatesEnabledAndCUDAGraphDisabled declaration / 类 TestOpenAIServerWithHiddenStatesEnabledAndCUDAGraphDisabled 声明
```python
class TestOpenAIServerWithHiddenStatesEnabledAndCUDAGraphDisabled(
    CustomTestCase, BaseTestOpenAIServerWithHiddenStates
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `BaseTestOpenAIServerWithHiddenStates`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `BaseTestOpenAIServerWithHiddenStates`。

### Lines 242-258: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=["--enable-return-hidden-states", "--disable-cuda-graph"],
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
        cls.return_hidden_states = [False, True]
        cls.use_list_input = [True, False]
        cls.parallel_sample_nums = [1]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 260-262: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 265-267: class TestOpenAIServerWithEAGLEAndHiddenStatesEnabled declaration / 类 TestOpenAIServerWithEAGLEAndHiddenStatesEnabled 声明
```python
class TestOpenAIServerWithEAGLEAndHiddenStatesEnabled(
    CustomTestCase, BaseTestOpenAIServerWithHiddenStates
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `BaseTestOpenAIServerWithHiddenStates`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `BaseTestOpenAIServerWithHiddenStates`。

### Lines 268-303: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_TARGET_MODEL_EAGLE
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.speculative_draft_model = DEFAULT_DRAFT_MODEL_EAGLE
        cls.speculative_algorithm = "EAGLE"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--speculative-algorithm",
                "EAGLE",
                "--speculative-draft-model-path",
                DEFAULT_DRAFT_MODEL_EAGLE,
                "--speculative-num-steps",
                5,
                "--speculative-eagle-topk",
                8,
                "--speculative-num-draft-tokens",
                64,
                "--mem-fraction-static",
                0.7,
                "--chunked-prefill-size",
                128,
                "--max-running-requests",
                8,
                "--enable-return-hidden-states",
            ],
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(DEFAULT_TARGET_MODEL_EAGLE)
        cls.return_hidden_states = [False, True]
        cls.use_list_input = [True, False]
        cls.parallel_sample_nums = [1]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 305-307: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 310-312: class TestOpenAIServerWithEAGLE3AndHiddenStatesEnabled declaration / 类 TestOpenAIServerWithEAGLE3AndHiddenStatesEnabled 声明
```python
class TestOpenAIServerWithEAGLE3AndHiddenStatesEnabled(
    CustomTestCase, BaseTestOpenAIServerWithHiddenStates
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `BaseTestOpenAIServerWithHiddenStates`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `BaseTestOpenAIServerWithHiddenStates`。

### Lines 313-350: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "meta-llama/Llama-3.1-8B-Instruct"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.speculative_algorithm = "EAGLE3"
        cls.speculative_draft_model = "jamesliu1/sglang-EAGLE3-Llama-3.1-Instruct-8B"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--speculative-algorithm",
                cls.speculative_algorithm,
                "--speculative-draft-model-path",
                cls.speculative_draft_model,
                "--speculative-num-steps",
                5,
                "--speculative-eagle-topk",
                16,
                "--speculative-num-draft-tokens",
                64,
                "--mem-fraction-static",
                0.7,
                "--chunked-prefill-size",
                128,
                "--max-running-requests",
                8,
                "--dtype",
                "float16",
                "--enable-return-hidden-states",
            ],
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(cls.model)
        cls.return_hidden_states = [False, True]
        cls.use_list_input = [True, False]
        cls.parallel_sample_nums = [1]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 352-354: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 357-358: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `BaseTestOpenAIServerWithHiddenStates`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIServerWithHiddenStatesEnabled`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIServerWithHiddenStatesEnabledAndCUDAGraphDisabled`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIServerWithEAGLEAndHiddenStatesEnabled`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIServerWithEAGLE3AndHiddenStatesEnabled`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `BaseTestOpenAIServerWithHiddenStates.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `BaseTestOpenAIServerWithHiddenStates.test_completion`: This test exercises `test_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion`。
- `BaseTestOpenAIServerWithHiddenStates.test_completion_stream`: This test exercises `test_completion_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion_stream`。
- `BaseTestOpenAIServerWithHiddenStates.test_chat_completion`: This test exercises `test_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion`。
- `BaseTestOpenAIServerWithHiddenStates.test_chat_completion_stream`: This test exercises `test_chat_completion_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_stream`。
- `BaseTestOpenAIServerWithHiddenStates.run_completion`: This helper encapsulates `run_completion` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_completion`，以便周围测试复用准备、执行或校验逻辑。
- `BaseTestOpenAIServerWithHiddenStates.run_completion_stream`: This helper encapsulates `run_completion_stream` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_completion_stream`，以便周围测试复用准备、执行或校验逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `abc`
- **Third-party modules / 第三方模块**: `openai`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 358
