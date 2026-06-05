# test_continuous_usage_stats.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/openai_server/features/test_continuous_usage_stats.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `continuous usage stats` scenario in `test/manual/openai_server/features`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual/openai_server/features` 中的 `continuous usage stats` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and shared helpers / 导入与共享辅助项
```python
import asyncio
import unittest

import openai

from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This range imports `asyncio`, `unittest`, `openai` and `sglang.srt.utils`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 15-16: Class definition for TestContinuousUsageStats / 类定义
```python
class TestContinuousUsageStats(CustomTestCase):
    @classmethod
```
**EN:** This range declares `TestContinuousUsageStats`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 17-22: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(cls.model, cls.base_url, timeout=300)
        cls.client = openai.Client(api_key="EMPTY", base_url=f"{cls.base_url}/v1")
        cls.aclient = openai.AsyncClient(api_key="EMPTY", base_url=f"{cls.base_url}/v1")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`, `Client` and `AsyncClient`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 23-24: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 25-26: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 28-39: Test routines around test_continuous_usage_stats_enabled / 测试例程
```python
    def test_continuous_usage_stats_enabled(self):
        stream = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": "What is machine learning?"}],
            stream=True,
            max_tokens=30,
            temperature=0,
            stream_options={"include_usage": True, "continuous_usage_stats": True},
        )

        chunks_with_usage = 0
        chunks_with_content = 0
```
**EN:** This range defines concrete test routine(s) `test_continuous_usage_stats_enabled`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 40-52: Assertions and result checks / 断言与结果检查
```python
        last_usage = None

        for chunk in stream:
            has_content = len(chunk.choices) > 0 and chunk.choices[0].delta.content
            if chunk.usage:
                chunks_with_usage += 1
                last_usage = chunk.usage
            if has_content:
                chunks_with_content += 1

        assert chunks_with_content > 0
        assert chunks_with_usage >= chunks_with_content
        assert last_usage.prompt_tokens > 0
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。

### Lines 53-57: Assertions and result checks / 断言与结果检查
```python
        assert last_usage.completion_tokens > 0
        assert (
            last_usage.total_tokens
            == last_usage.prompt_tokens + last_usage.completion_tokens
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 59-64: Test routines around test_continuous_usage_stats_async / 测试例程
```python
    async def test_continuous_usage_stats_async(self):
        stream = await self.aclient.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": "What is deep learning?"}],
            stream=True,
            max_tokens=30,
```
**EN:** This range defines concrete test routine(s) `test_continuous_usage_stats_async`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 65-80: Assertions and result checks / 断言与结果检查
```python
            temperature=0,
            stream_options={"include_usage": True, "continuous_usage_stats": True},
        )

        chunks_with_usage = 0
        chunks_with_content = 0

        async for chunk in stream:
            has_content = len(chunk.choices) > 0 and chunk.choices[0].delta.content
            if chunk.usage:
                chunks_with_usage += 1
            if has_content:
                chunks_with_content += 1

        assert chunks_with_content > 0
        assert chunks_with_usage >= chunks_with_content
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。

### Lines 82-89: Test routines around test_continuous_usage_stats_disabled / 测试例程
```python
    def test_continuous_usage_stats_disabled(self):
        stream = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": "What is AI?"}],
            stream=True,
            max_tokens=30,
            temperature=0,
            stream_options={"include_usage": True, "continuous_usage_stats": False},
```
**EN:** This range defines concrete test routine(s) `test_continuous_usage_stats_disabled`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 90-98: Assertions and result checks / 断言与结果检查
```python
        )

        usage_chunks = []
        for chunk in stream:
            if chunk.usage:
                usage_chunks.append(chunk)

        assert len(usage_chunks) == 1
        assert len(usage_chunks[0].choices) == 0
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `append`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 100-101: Test routines around test_async_runner / 测试例程
```python
    def test_async_runner(self):
        asyncio.run(self.test_continuous_usage_stats_async())
```
**EN:** This range defines concrete test routine(s) `test_async_runner`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `run` and `test_continuous_usage_stats_async`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 102-105: Script entry point / 脚本入口
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
- **Standard Library / 标准库**: `asyncio`, `unittest`
- **Third-party / 第三方库**: `openai`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.test_utils`
