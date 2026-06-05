# test_sagemaker_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_sagemaker_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `sagemaker server` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `sagemaker server` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Process control logic / 进程控制逻辑
```python
"""
python3 -m unittest test_sagemaker_server.TestSageMakerServer.test_chat_completion
"""

import json
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 21-22: Class definition for TestSageMakerServer / 类定义
```python
class TestSageMakerServer(CustomTestCase):
    @classmethod
```
**EN:** This range declares `TestSageMakerServer`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 23-33: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
        )
        cls.tokenizer = get_tokenizer(DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server` and `get_tokenizer`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 34-35: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 36-37: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 39-45: Helper routines around run_chat_completion / 辅助例程
```python
    def run_chat_completion(self, logprobs, parallel_sample_num):
        data = {
            "model": self.model,
            "messages": [
                {"role": "system", "content": "You are a helpful AI assistant"},
                {
                    "role": "user",
```
**EN:** This range implements helper routine(s) `run_chat_completion` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 46-63: Assertions and result checks / 断言与结果检查
```python
                    "content": "What is the capital of France? Answer in a few words.",
                },
            ],
            "temperature": 0,
            "logprobs": logprobs is not None and logprobs > 0,
            "top_logprobs": logprobs,
            "n": parallel_sample_num,
        }

        headers = {"Authorization": f"Bearer {self.api_key}"}

        response = requests.post(
            f"{self.base_url}/invocations", json=data, headers=headers
        ).json()

        if logprobs:
            assert isinstance(
                response["choices"][0]["logprobs"]["content"][0]["top_logprobs"][0][
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `post` and `json`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 64-70: Scenario logic / 场景逻辑
```python
                    "token"
                ],
                str,
            )

            ret_num_top_logprobs = len(
                response["choices"][0]["logprobs"]["content"][0]["top_logprobs"]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 71-83: Assertions and result checks / 断言与结果检查
```python
            )
            assert (
                ret_num_top_logprobs == logprobs
            ), f"{ret_num_top_logprobs} vs {logprobs}"

        assert len(response["choices"]) == parallel_sample_num
        assert response["choices"][0]["message"]["role"] == "assistant"
        assert isinstance(response["choices"][0]["message"]["content"], str)
        assert response["id"]
        assert response["created"]
        assert response["usage"]["prompt_tokens"] > 0
        assert response["usage"]["completion_tokens"] > 0
        assert response["usage"]["total_tokens"] > 0
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 85-95: Helper routines around run_chat_completion_stream / 辅助例程
```python
    def run_chat_completion_stream(self, logprobs, parallel_sample_num=1):
        data = {
            "model": self.model,
            "messages": [
                {"role": "system", "content": "You are a helpful AI assistant"},
                {
                    "role": "user",
                    "content": "What is the capital of France? Answer in a few words.",
                },
            ],
            "temperature": 0,
```
**EN:** This range implements helper routine(s) `run_chat_completion_stream` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 96-109: Request and response handling / 请求与响应处理
```python
            "logprobs": logprobs is not None and logprobs > 0,
            "top_logprobs": logprobs,
            "stream": True,
            "stream_options": {"include_usage": True},
            "n": parallel_sample_num,
        }

        headers = {"Authorization": f"Bearer {self.api_key}"}

        response = requests.post(
            f"{self.base_url}/invocations", json=data, stream=True, headers=headers
        )

        is_firsts = {}
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `post`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 110-120: Assertions and result checks / 断言与结果检查
```python
        for line in response.iter_lines():
            line = line.decode("utf-8").replace("data: ", "")
            if len(line) < 1 or line == "[DONE]":
                continue
            print(f"value: {line}")
            line = json.loads(line)
            usage = line.get("usage")
            if usage is not None:
                assert usage["prompt_tokens"] > 0
                assert usage["completion_tokens"] > 0
                assert usage["total_tokens"] > 0
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `iter_lines`, `decode`, `replace` and `loads`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 121-134: Assertions and result checks / 断言与结果检查
```python
                continue

            index = line.get("choices")[0].get("index")
            data = line.get("choices")[0].get("delta")

            if is_firsts.get(index, True):
                assert data["role"] == "assistant"
                is_firsts[index] = False
                continue

            # Skip chunks that are just empty placeholders, usually at stream end/stop
            if data.get("content") is None:
                continue
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 135-145: Assertions and result checks / 断言与结果检查
```python
            if logprobs:
                assert line.get("choices")[0].get("logprobs")
                assert isinstance(
                    line.get("choices")[0]
                    .get("logprobs")
                    .get("content")[0]
                    .get("top_logprobs")[0]
                    .get("token"),
                    str,
                )
                assert isinstance(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 146-159: Assertions and result checks / 断言与结果检查
```python
                    line.get("choices")[0]
                    .get("logprobs")
                    .get("content")[0]
                    .get("top_logprobs"),
                    list,
                )
                ret_num_top_logprobs = len(
                    line.get("choices")[0]
                    .get("logprobs")
                    .get("content")[0]
                    .get("top_logprobs")
                )
                assert (
                    ret_num_top_logprobs == logprobs
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get` and `assert`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 160-169: Assertions and result checks / 断言与结果检查
```python
                ), f"{ret_num_top_logprobs} vs {logprobs}"

            assert isinstance(data["content"], str)
            assert line["id"]
            assert line["created"]

        for index in [i for i in range(parallel_sample_num)]:
            assert not is_firsts.get(
                index, True
            ), f"index {index} is not found in the response"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 171-174: Test routines around test_chat_completion / 测试例程
```python
    def test_chat_completion(self):
        for logprobs in [None, 5]:
            for parallel_sample_num in [1, 2]:
                self.run_chat_completion(logprobs, parallel_sample_num)
```
**EN:** This range defines concrete test routine(s) `test_chat_completion`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `run_chat_completion`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 176-179: Test routines around test_chat_completion_stream / 测试例程
```python
    def test_chat_completion_stream(self):
        for logprobs in [None, 5]:
            for parallel_sample_num in [1, 2]:
                self.run_chat_completion_stream(logprobs, parallel_sample_num)
```
**EN:** This range defines concrete test routine(s) `test_chat_completion_stream`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `run_chat_completion_stream`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 180-183: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Streaming responses / 流式响应
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.test_utils`
