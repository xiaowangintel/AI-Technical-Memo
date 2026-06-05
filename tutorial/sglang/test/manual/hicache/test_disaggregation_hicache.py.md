# test_disaggregation_hicache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/hicache/test_disaggregation_hicache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `disaggregation hicache` scenario in `test/manual/hicache`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/hicache` 中的 `disaggregation hicache` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and shared helpers / 导入与共享辅助项
```python
import os
import random
import tempfile
import unittest
from typing import Dict

import requests

from sglang.benchmark.utils import get_tokenizer
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    popen_launch_pd_server,
)
```
**EN:** This range imports `os`, `random`, `tempfile` and `unittest`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 20-23: Class definition for DisaggregationHiCacheBase / 类定义
```python
class DisaggregationHiCacheBase(PDDisaggregationServerBase):
    """Base class for disaggregation with HiCache tests"""

    @classmethod
```
**EN:** This range declares `DisaggregationHiCacheBase`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 24-38: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        super(DisaggregationHiCacheBase, cls).setUpClass()

        cls.model = DEFAULT_MODEL_NAME_FOR_TEST

        cls.tokenizer = get_tokenizer(cls.model)
        cls.temp_dir = tempfile.mkdtemp()
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `get_tokenizer`, `mkdtemp`, `start_prefill` and `start_decode`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 39-40: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 41-44: Helper routines around start_prefill / 辅助例程
```python
    def start_prefill(cls):
        # Prefill with HiCache enabled
        prefill_args = [
            "--trust-remote-code",
```
**EN:** This range implements helper routine(s) `start_prefill` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 45-65: Scenario logic / 场景逻辑
```python
            "--disaggregation-mode",
            "prefill",
            "--tp-size",
            "1",
            "--page-size",
            "64",
            "--enable-hierarchical-cache",
            "--hicache-ratio",
            "1.2",
            "--hicache-size",
            "0",
            "--hicache-write-policy",
            "write_through",
            "--hicache-storage-backend",
            "file",
            "--hicache-storage-prefetch-policy",
            "wait_complete",
            "--mem-fraction-static",
            "0.8",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 66-69: Environment-driven configuration / 环境驱动配置
```python
        env = {
            **os.environ,
            "SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR": cls.temp_dir,
        }
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 70-76: Process control logic / 进程控制逻辑
```python
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
            env=env,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `popen_launch_pd_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 77-78: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 79-80: Helper routines around start_decode / 辅助例程
```python
    def start_decode(cls):
        pass
```
**EN:** This range implements helper routine(s) `start_decode` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 82-85: Helper routines around gen_prompt / 辅助例程
```python
    def gen_prompt(self, token_num: int) -> str:
        all_available_tokens = list(self.tokenizer.get_vocab().values())
        selected_tokens = random.choices(all_available_tokens, k=token_num)
        return self.tokenizer.decode(selected_tokens)
```
**EN:** This range implements helper routine(s) `gen_prompt` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `get_vocab`, `values`, `choices` and `decode`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 87-94: Helper routines around send_request / 辅助例程
```python
    def send_request(
        self, prompt: str, max_tokens: int = 100, temperature: float = 0.0
    ) -> Dict:
        """Send a generate request and return response"""
        response = requests.post(
            f"{self.lb_url}/generate",
            json={
                "text": prompt,
```
**EN:** This range implements helper routine(s) `send_request` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `post`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 95-109: Assertions and result checks / 断言与结果检查
```python
                "sampling_params": {
                    "temperature": temperature,
                    "max_new_tokens": max_tokens,
                    "ignore_eos": True,
                },
            },
            timeout=60,
        )

        self.assertEqual(
            response.status_code,
            200,
            f"Request failed: {response.status_code} - {response.text}",
        )
        return response.json()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual` and `json`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 111-119: Helper routines around trigger_offloading_and_flush / 辅助例程
```python
    def trigger_offloading_and_flush(self):
        """Helper method to trigger offloading and flush cache"""
        # Trigger offloading
        self.send_request(self.gen_prompt(1), max_tokens=150)

        # Flush device cache to force remote storage access.
        res = requests.post(
            f"{self.prefill_url}/flush_cache",
            params={"timeout": 30},
```
**EN:** This range implements helper routine(s) `trigger_offloading_and_flush` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `send_request`, `gen_prompt` and `post`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 120-122: Scenario logic / 场景逻辑
```python
            timeout=40,
        )
        res.raise_for_status()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `raise_for_status`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 125-128: Class definition for TestDisaggregationPrefillWithHiCache / 类定义
```python
class TestDisaggregationPrefillWithHiCache(DisaggregationHiCacheBase):
    """Test disaggregation with HiCache enabled only on Prefill side"""

    @classmethod
```
**EN:** This range declares `TestDisaggregationPrefillWithHiCache`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 129-149: Helper routines around start_decode / 辅助例程
```python
    def start_decode(cls):
        # Decode without HiCache offload
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--tp-size",
            "1",
            "--page-size",
            "64",
            "--mem-fraction-static",
            "0.8",
            "--base-gpu-id",
            "1",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        env = {
            **os.environ,
            "SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR": cls.temp_dir,
        }
        cls.process_decode = popen_launch_pd_server(
```
**EN:** This range implements helper routine(s) `start_decode` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `popen_launch_pd_server`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 150-153: Scenario logic / 场景逻辑
```python
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 154-155: Scenario logic / 场景逻辑
```python
            env=env,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 157-172: Test routines around test_prefill_cache_hit / 测试例程
```python
    def test_prefill_cache_hit(self):
        """Test that prefill cache works with repeated queries"""

        repeated_prompt = self.gen_prompt(800)

        # First request - should miss cache
        self.send_request(repeated_prompt, max_tokens=100)

        # Flush cache
        self.trigger_offloading_and_flush()

        # Second request - should hit cache (faster)
        response2 = self.send_request(repeated_prompt, max_tokens=100)

        # Assert cached tokens cnt
        self.assertGreater(response2["meta_info"]["cached_tokens"], 700)
```
**EN:** This range defines concrete test routine(s) `test_prefill_cache_hit`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `gen_prompt`, `send_request`, `trigger_offloading_and_flush` and `cache`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 175-178: Class definition for TestDisaggregationDecodeWithHiCache / 类定义
```python
class TestDisaggregationDecodeWithHiCache(DisaggregationHiCacheBase):
    """Test disaggregation with HiCache enabled on both Prefill and Decode sides"""

    @classmethod
```
**EN:** This range declares `TestDisaggregationDecodeWithHiCache`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 179-199: Helper routines around start_decode / 辅助例程
```python
    def start_decode(cls):
        # Decode with HiCache offload enabled
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--tp-size",
            "1",
            "--page-size",
            "64",
            "--mem-fraction-static",
            "0.8",
            "--base-gpu-id",
            "1",
            "--disaggregation-decode-enable-offload-kvcache",
            "--hicache-ratio",
            "1.2",
            "--hicache-size",
            "0",
            "--hicache-storage-backend",
            "file",
```
**EN:** This range implements helper routine(s) `start_decode` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 200-203: Scenario logic / 场景逻辑
```python
            "--hicache-storage-prefetch-policy",
            "wait_complete",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 204-214: Process control logic / 进程控制逻辑
```python
        env = {
            **os.environ,
            "SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR": cls.temp_dir,
        }
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
            env=env,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `popen_launch_pd_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 216-224: Test routines around test_multi_turn_conversation_cache / 测试例程
```python
    def test_multi_turn_conversation_cache(self):
        """Test multi-turn conversation scenario with cache hit improvement"""

        print("=== Multi-turn Conversation Cache Test ===")

        # Turn 1
        initial_prompt = self.gen_prompt(300)

        response1 = self.send_request(initial_prompt, max_tokens=200, temperature=0.1)
```
**EN:** This range defines concrete test routine(s) `test_multi_turn_conversation_cache`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `gen_prompt` and `send_request`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 225-240: Scenario logic / 场景逻辑
```python
        current_context = initial_prompt + response1["text"]

        # Turns 2-4: Continue generation based on previous context
        previous_cached_tokens = 0

        for turn in range(2, 5):
            print(f"\nTurn {turn}: Continuing from previous context")

            response = self.send_request(
                current_context, max_tokens=200, temperature=0.1
            )
            cached_tokens = response["meta_info"]["cached_tokens"]

            print(f"Turn {turn} cached tokens: {cached_tokens}")
            print(f"Improvement: {cached_tokens - previous_cached_tokens} tokens")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `send_request`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 241-249: Assertions and result checks / 断言与结果检查
```python
            # Assert cache improvement
            self.assertGreater(
                cached_tokens,
                previous_cached_tokens,
                f"Turn {turn} should have more cached tokens than turn {turn-1}",
            )

            # Update context and cached tokens for next iteration
            current_context += response["text"]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 250-253: Scenario logic / 场景逻辑
```python
            previous_cached_tokens = cached_tokens

            # Flush prefill cache
            self.trigger_offloading_and_flush()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `trigger_offloading_and_flush`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 254-257: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Environment-aware configuration / 环境感知配置

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `os`, `random`, `tempfile`, `typing`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.benchmark.utils`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
