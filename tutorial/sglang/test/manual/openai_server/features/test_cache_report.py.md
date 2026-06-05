# test_cache_report.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/openai_server/features/test_cache_report.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `cache report` scenario in `test/manual/openai_server/features`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual/openai_server/features` 中的 `cache report` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and shared helpers / 导入与共享辅助项
```python
import unittest

import openai
import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This range imports `unittest`, `openai`, `requests` and `sglang.srt.utils`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 15-16: Class definition for TestCacheReport / 类定义
```python
class TestCacheReport(CustomTestCase):
    @classmethod
```
**EN:** This range declares `TestCacheReport`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 17-39: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.min_cached = 5
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=300,
            other_args=[
                "--chunked-prefill-size=40",
                "--attention-backend=triton",
                "--enable-cache-report",
            ],
        )
        cls.client = openai.Client(api_key="EMPTY", base_url=f"{cls.base_url}/v1")
        cls.aclient = openai.AsyncClient(api_key="EMPTY", base_url=f"{cls.base_url}/v1")

        usage = cls.run_openai(cls, "1").usage
        # we can assume that our request is of size 1, plus the total template size
        # ideally we would like to know the begin size / end size of the template to be more precise
        total_template_size = usage.prompt_tokens - 1
        print(f"template size: {total_template_size}")
        usage2 = cls.run_openai(cls, "2").usage
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`, `Client`, `AsyncClient` and `run_openai`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 40-41: Assertions and result checks / 断言与结果检查
```python
        assert usage2.prompt_tokens_details.cached_tokens <= total_template_size
        cls.min_cached = max(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。

### Lines 42-44: Scenario logic / 场景逻辑
```python
            usage2.prompt_tokens_details.cached_tokens,
            total_template_size - usage2.prompt_tokens_details.cached_tokens,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 45-46: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 47-48: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 50-64: Helper routines around run_decode / 辅助例程
```python
    def run_decode(self, return_logprob=False, top_logprobs_num=0, n=1):
        response = requests.post(
            self.base_url + "/generate",
            # we use an uncommon start to minimise the chance that the cache is hit by chance
            json={
                "text": "_ The capital of France is",
                "sampling_params": {
                    "temperature": 0 if n == 1 else 0.5,
                    "max_new_tokens": 128,
                    "n": n,
                    "stop_token_ids": [119690],
                },
                "stream": False,
                "return_logprob": return_logprob,
                "top_logprobs_num": top_logprobs_num,
```
**EN:** This range implements helper routine(s) `run_decode` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `post`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 65-68: Scenario logic / 场景逻辑
```python
                "logprob_start_len": 0,
            },
        )
        return response
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 70-80: Helper routines around run_openai / 辅助例程
```python
    def run_openai(self, message):
        response = self.client.chat.completions.create(
            model=self.model,
            messages=[
                # {"role": "system", "content": "You are a helpful AI assistant"},
                {"role": "user", "content": message},
            ],
            temperature=0,
            max_tokens=100,
        )
        return response
```
**EN:** This range implements helper routine(s) `run_openai` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `create`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 82-89: Helper routines around run_openai_async / 辅助例程
```python
    async def run_openai_async(self, message):
        response = await self.aclient.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "user", "content": message},
            ],
            temperature=0,
            max_tokens=100,
```
**EN:** This range implements helper routine(s) `run_openai_async` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `create`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 90-91: Scenario logic / 场景逻辑
```python
        )
        return response
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 93-106: Helper routines around cache_report_openai / 辅助例程
```python
    def cache_report_openai(self, message):
        response = self.run_openai(message)
        print(
            f"openai first request cached_tokens: {int(response.usage.prompt_tokens_details.cached_tokens)}"
        )
        first_cached_tokens = int(response.usage.prompt_tokens_details.cached_tokens)
        # assert int(response.usage.cached_tokens) == 0
        assert first_cached_tokens <= self.min_cached
        response = self.run_openai(message)
        cached_tokens = int(response.usage.prompt_tokens_details.cached_tokens)
        print(f"openai second request cached_tokens: {cached_tokens}")
        assert cached_tokens > 0
        assert cached_tokens == int(response.usage.prompt_tokens) - 1
        return first_cached_tokens
```
**EN:** This range implements helper routine(s) `cache_report_openai` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `run_openai`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 108-112: Helper routines around cache_report_openai_async / 辅助例程
```python
    async def cache_report_openai_async(self, message):
        response = await self.run_openai_async(message)
        cached_tokens = int(response.usage.prompt_tokens_details.cached_tokens)
        prompt_tokens = int(response.usage.prompt_tokens)
        return cached_tokens, prompt_tokens
```
**EN:** This range implements helper routine(s) `cache_report_openai_async` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `run_openai_async`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 114-114: Test routines around test_generate / 测试例程
```python
    def test_generate(self):
```
**EN:** This range defines concrete test routine(s) `test_generate`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 115-131: Assertions and result checks / 断言与结果检查
```python
        print("=" * 100)
        response = self.run_decode()
        # print(response.json())
        cached_tokens = int(response.json()["meta_info"]["cached_tokens"])
        print(f"sglang first request cached_tokens: {cached_tokens}")
        print(
            f"sglang first request prompt_tokens: {int(response.json()['meta_info']['prompt_tokens'])}"
        )
        # can't assure to be 0: depends on the initialisation request / if a template is used with the model
        assert cached_tokens < self.min_cached
        response = self.run_decode()
        cached_tokens = int(response.json()["meta_info"]["cached_tokens"])
        print(f"sglang second request cached_tokens: {cached_tokens}")
        print(
            f"sglang second request prompt_tokens: {int(response.json()['meta_info']['prompt_tokens'])}"
        )
        assert cached_tokens == int(response.json()["meta_info"]["prompt_tokens"]) - 1
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `run_decode` and `json`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 133-139: Test routines around test_cache_split_prefill_openai / 测试例程
```python
    def test_cache_split_prefill_openai(self):
        print("=" * 100)
        self.cache_report_openai(
            "€ This is a very long and unique text that should not be already cached, the twist is"
            " that it should be longer than the chunked-prefill-size, so it should be split among"
            " several prefill requests. Still, it shouldn't be cached"
        )
```
**EN:** This range defines concrete test routine(s) `test_cache_split_prefill_openai`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `cache_report_openai`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 141-164: Test routines around test_cache_report_openai / 测试例程
```python
    def test_cache_report_openai(self):
        print("=" * 100)
        # warm up the cache, for the template
        self.run_openai("Introduce the capital of France.")

        first_cached_tokens_1 = self.run_openai(
            "How many sparrow do you need to lift a coconut?"
        ).usage.prompt_tokens_details.cached_tokens

        usage_2 = self.run_openai("* sing something about cats").usage
        first_cached_tokens_2 = usage_2.prompt_tokens_details.cached_tokens
        # first request may not have 0 cached tokens, but if they only have the template in common they
        # should be the same once the cache is warmed up
        assert first_cached_tokens_1 == first_cached_tokens_2

        resp = self.run_openai("* sing something about cats and dogs")
        print(resp.usage)

        resp = self.run_openai("* sing something about cats, please")
        print(resp.usage)
        assert (
            resp.usage.prompt_tokens_details.cached_tokens
            >= usage_2.prompt_tokens - self.min_cached
        )
```
**EN:** This range defines concrete test routine(s) `test_cache_report_openai`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `run_openai` and `assert`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 165-189: Request and response handling / 请求与响应处理
```python

    # TODO: flaky test
    # def test_cache_report_openai_async(self):
    #     print("=" * 100)

    #     async def run_test():
    #         task0 = asyncio.create_task(
    #             self.cache_report_openai_async(
    #                 "first request, to start the inference and let the next two request be started in the same batch"
    #             )
    #         )
    #         await asyncio.sleep(1)  # to force the first request to be started first
    #         task1 = asyncio.create_task(
    #             self.cache_report_openai_async(
    #                 "> can the same batch parallel request use the cache?"
    #             )
    #         )
    #         task2 = asyncio.create_task(
    #             self.cache_report_openai_async(
    #                 "> can the same batch parallel request use the cache?"
    #             )
    #         )
    #         result0, result1, result2 = await asyncio.gather(task0, task1, task2)

    #         cached_tokens0, prompt_tokens0 = result0
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `test_cache_report_openai_async`, `run_test`, `create_task` and `cache_report_openai_async`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 190-210: Assertions and result checks / 断言与结果检查
```python
    #         cached_tokens1, prompt_tokens1 = result1
    #         cached_tokens2, prompt_tokens2 = result2

    #         print(
    #             f"Async request 0 - Cached tokens: {cached_tokens0}, Prompt tokens: {prompt_tokens0}"
    #         )
    #         print(
    #             f"Async request 1 - Cached tokens: {cached_tokens1}, Prompt tokens: {prompt_tokens1}"
    #         )
    #         print(
    #             f"Async request 2 - Cached tokens: {cached_tokens2}, Prompt tokens: {prompt_tokens2}"
    #         )

    #         # Assert that no requests used the cache (because first is alone, and the next two are in the same batch)
    #         # If a new optimisation limiting starting request with same prefix at the same time was added
    #         # to maximise the cache hit, this would not be true
    #         assert cached_tokens1 == cached_tokens2 == cached_tokens0

    #     asyncio.run(run_test())

    @staticmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `cache`, `run` and `run_test`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 211-214: Helper routines around _get_cached_tokens / 辅助例程
```python
    def _get_cached_tokens(response) -> int:
        """Extract cached_tokens from response, returning 0 if prompt_tokens_details is None."""
        details = response.usage.prompt_tokens_details
        if details is None:
```
**EN:** This range implements helper routine(s) `_get_cached_tokens` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 215-216: Scenario logic / 场景逻辑
```python
            return 0
        return int(details.cached_tokens)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 218-239: Test routines around test_cache_salt_effectiveness / 测试例程
```python
    def test_cache_salt_effectiveness(self):
        print("=" * 100)
        print("Testing cache_salt effectiveness")

        # Use a unique message to avoid interference with other tests
        test_message = "What is the capital of Japan?"

        # First request with cache_salt "salt1"
        response1 = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": test_message}],
            temperature=0,
            max_tokens=10,
            extra_body={"cache_salt": "salt1"},
        )
        cached_tokens_1_first = self._get_cached_tokens(response1)
        prompt_tokens_1 = int(response1.usage.prompt_tokens)
        print(
            f"First request with salt1 - cached_tokens: {cached_tokens_1_first}, prompt_tokens: {prompt_tokens_1}"
        )

        # Second request with same cache_salt "salt1" - should get cache hit
```
**EN:** This range defines concrete test routine(s) `test_cache_salt_effectiveness`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `create` and `_get_cached_tokens`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 240-242: Request and response handling / 请求与响应处理
```python
        response2 = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": test_message}],
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `create`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 243-264: Assertions and result checks / 断言与结果检查
```python
            temperature=0,
            max_tokens=10,
            extra_body={"cache_salt": "salt1"},
        )
        cached_tokens_1_second = self._get_cached_tokens(response2)
        print(
            f"Second request with salt1 - cached_tokens: {cached_tokens_1_second}, prompt_tokens: {prompt_tokens_1}"
        )

        # Verify cache hit for same salt
        assert (
            cached_tokens_1_second > cached_tokens_1_first
        ), "Should have cache hit with same cache_salt"
        assert (
            cached_tokens_1_second == prompt_tokens_1 - 1
        ), "Should cache all prompt tokens except the last one"

        # Third request with different cache_salt "salt2" - should not get cache hit
        response3 = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": test_message}],
            temperature=0,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `_get_cached_tokens`, `assert` and `create`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 265-267: Scenario logic / 场景逻辑
```python
            max_tokens=10,
            extra_body={"cache_salt": "salt2"},
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 268-289: Assertions and result checks / 断言与结果检查
```python
        cached_tokens_2_first = self._get_cached_tokens(response3)
        print(f"First request with salt2 - cached_tokens: {cached_tokens_2_first}")

        # Verify no cache hit for different salt (should be similar to first request with salt1)
        assert (
            cached_tokens_2_first <= cached_tokens_1_first + self.min_cached
        ), "Different cache_salt should not share cache"

        # Fourth request with same cache_salt "salt2" - should now get cache hit
        response4 = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": test_message}],
            temperature=0,
            max_tokens=10,
            extra_body={"cache_salt": "salt2"},
        )
        cached_tokens_2_second = self._get_cached_tokens(response4)
        print(f"Second request with salt2 - cached_tokens: {cached_tokens_2_second}")

        # Verify cache hit for salt2
        assert (
            cached_tokens_2_second > cached_tokens_2_first
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `_get_cached_tokens`, `salt`, `assert` and `create`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 290-290: Scenario logic / 场景逻辑
```python
        ), "Should have cache hit with same cache_salt for salt2"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 291-294: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Streaming responses / 流式响应
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: `openai`, `requests`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.test_utils`
