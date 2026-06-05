# test_hicache_storage_file_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/hicache/test_hicache_storage_file_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates hicache storage file backend behavior in SGLang's hicache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 hicache 领域中与 hicache storage file backend 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: supporting statements / 辅助语句
```python
"""
E2E tests for HiCache Storage functionality.
Usage:
    python3 -m pytest test/registered/hicache/test_hicache_storage_file_backend.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 7-32: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import random
import tempfile
import time
import unittest
from types import SimpleNamespace
from typing import Dict
from urllib.parse import urlparse

import requests

from sglang.benchmark.utils import get_tokenizer
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MLA_MODEL_NAME_FOR_TEST,
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
)
from sglang.utils import wait_for_http_ready
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `random`, `tempfile`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `random`, `tempfile`。

### Lines 34-35: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=148, stage="base-b", runner_config="2-gpu-large")
register_amd_ci(est_time=526, suite="stage-b-test-2-gpu-large-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 38-38: class HiCacheStorageBaseMixin declaration / 类 HiCacheStorageBaseMixin 声明
```python
class HiCacheStorageBaseMixin:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 39-39: supporting statements / 辅助语句
```python
    """Base mixin class with common setup and utilities"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 41-60: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        """Set up test environment and launch server once for all tests"""
        cls.temp_dir = tempfile.mkdtemp()
        cls.model = cls._get_model_name()
        cls.base_url = DEFAULT_URL_FOR_TEST

        parsed_url = urlparse(cls.base_url)
        cls.base_host = parsed_url.hostname
        cls.base_port = str(parsed_url.port)

        # Prepare tokenizer for prompt generation
        cls.tokenizer = get_tokenizer(cls.model)

        # Launch server with HiCache enabled and cache report
        cls.process = cls._launch_server_with_hicache()
        cls._wait_for_server_ready(process=cls.process)

        print(f"Test server launched successfully at {cls.base_url}")
        print(f"Cache directory: {cls.temp_dir}")
```
**EN:** Set up test environment and launch server once for all tests This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test environment and launch server once for all tests 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 62-71: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        """Clean up test environment"""
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)

        import shutil

        if hasattr(cls, "temp_dir"):
            shutil.rmtree(cls.temp_dir, ignore_errors=True)
```
**EN:** Clean up test environment This routine releases resources and restores state after the related tests finish.
**CN:** Clean up test environment 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 73-76: method get model name / 方法 get model name
```python
    @classmethod
    def _get_model_name(cls):
        """Get model name for the test configuration - override in subclasses"""
        return DEFAULT_MODEL_NAME_FOR_TEST
```
**EN:** Get model name for the test configuration - override in subclasses This block implements `_get_model_name` and captures one focused piece of the module's behavior.
**CN:** Get model name for the test configuration - override in subclasses 该代码块实现 `_get_model_name`，承担模块行为中的一个聚焦逻辑片段。

### Lines 78-93: method get base server args / 方法 get base server args
```python
    @classmethod
    def _get_base_server_args(cls):
        """Get base server arguments - can be extended in subclasses"""
        extra_config = {
            "hicache_storage_pass_prefix_keys": True,
        }
        return {
            "--enable-hierarchical-cache": True,
            "--mem-fraction-static": 0.6,
            "--hicache-ratio": 1.2,
            "--page-size": 64,
            "--enable-cache-report": True,
            "--hicache-storage-prefetch-policy": "wait_complete",
            "--hicache-storage-backend": "file",
            "--hicache-storage-backend-extra-config": json.dumps(extra_config),
        }
```
**EN:** Get base server arguments - can be extended in subclasses This block implements `_get_base_server_args` and captures one focused piece of the module's behavior.
**CN:** Get base server arguments - can be extended in subclasses 该代码块实现 `_get_base_server_args`，承担模块行为中的一个聚焦逻辑片段。

### Lines 95-98: method get additional server args and env / 方法 get additional server args and env
```python
    @classmethod
    def _get_additional_server_args_and_env(cls):
        """Get additional server arguments specific to configuration - override in subclasses"""
        return {}, {"SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR": cls.temp_dir}
```
**EN:** Get additional server arguments specific to configuration - override in subclasses This block implements `_get_additional_server_args_and_env` and captures one focused piece of the module's behavior.
**CN:** Get additional server arguments specific to configuration - override in subclasses 该代码块实现 `_get_additional_server_args_and_env`，承担模块行为中的一个聚焦逻辑片段。

### Lines 100-131: method launch server with hicache / 方法 launch server with hicache
```python
    @classmethod
    def _launch_server_with_hicache(cls):
        """Launch server with HiCache enabled"""

        additional_server_args, env_vars = cls._get_additional_server_args_and_env()
        env_vars["SGLANG_ENABLE_DETERMINISTIC_INFERENCE"] = "1"
        server_args = cls._get_base_server_args()
        if additional_server_args:
            server_args.update(additional_server_args)

        final_server_args = []
        for k, v in server_args.items():
            if isinstance(v, bool):
                final_server_args.append(str(k))
            else:
                final_server_args.append(str(k))
                final_server_args.append(str(v))

        print(f"final_server_args: {final_server_args}")

        env_vars = {
            **os.environ,
            **env_vars,
        }

        return popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=final_server_args,
            env=env_vars,
        )
```
**EN:** Launch server with HiCache enabled This block implements `_launch_server_with_hicache` and captures one focused piece of the module's behavior.
**CN:** Launch server with HiCache enabled 该代码块实现 `_launch_server_with_hicache`，承担模块行为中的一个聚焦逻辑片段。

### Lines 133-141: method wait for server ready / 方法 wait for server ready
```python
    @classmethod
    def _wait_for_server_ready(cls, timeout: int = 60, process=None) -> bool:
        """Wait for server to be ready"""
        wait_for_http_ready(
            url=f"{cls.base_url}/health",
            timeout=timeout,
            process=process,
        )
        return True
```
**EN:** Wait for server to be ready This block implements `_wait_for_server_ready` and captures one focused piece of the module's behavior.
**CN:** Wait for server to be ready 该代码块实现 `_wait_for_server_ready`，承担模块行为中的一个聚焦逻辑片段。

### Lines 143-165: method send request / 方法 send request
```python
    def send_request(
        self, prompt: str, max_tokens: int = 100, temperature: float = 0.0
    ) -> Dict:
        """Send a generate request and return response"""
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "text": prompt,
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
**EN:** Send a generate request and return response This block implements `send_request` and captures one focused piece of the module's behavior.
**CN:** Send a generate request and return response 该代码块实现 `send_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 167-170: helper routine get cached tokens / 辅助流程 get cached tokens
```python
    def get_cached_tokens(self, response_json: Dict) -> int:
        """Extract cached tokens count from /generate response"""
        meta = response_json.get("meta_info", {})
        return int(meta.get("cached_tokens", 0))
```
**EN:** Extract cached tokens count from /generate response This helper encapsulates `get_cached_tokens` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Extract cached tokens count from /generate response 该辅助函数封装了 `get_cached_tokens`，以便周围测试复用准备、执行或校验逻辑。

### Lines 172-179: method flush cache / 方法 flush cache
```python
    def flush_cache(self):
        """Flush device cache to force remote storage access."""
        res = requests.post(
            f"{self.base_url}/flush_cache",
            params={"timeout": 30},
            timeout=40,
        )
        res.raise_for_status()
```
**EN:** Flush device cache to force remote storage access. This block implements `flush_cache` and captures one focused piece of the module's behavior.
**CN:** Flush device cache to force remote storage access. 该代码块实现 `flush_cache`，承担模块行为中的一个聚焦逻辑片段。

### Lines 181-185: method gen prompt / 方法 gen prompt
```python
    def gen_prompt(self, token_num: int) -> str:
        """Generate a random prompt of specified token length using tokenizer vocabulary."""
        all_available_tokens = list(self.tokenizer.get_vocab().values())
        selected_tokens = random.choices(all_available_tokens, k=token_num)
        return self.tokenizer.decode(selected_tokens)
```
**EN:** Generate a random prompt of specified token length using tokenizer vocabulary. This block implements `gen_prompt` and captures one focused piece of the module's behavior.
**CN:** Generate a random prompt of specified token length using tokenizer vocabulary. 该代码块实现 `gen_prompt`，承担模块行为中的一个聚焦逻辑片段。

### Lines 187-193: method trigger offloading and flush / 方法 trigger offloading and flush
```python
    def trigger_offloading_and_flush(self):
        """Helper method to trigger offloading and flush cache"""
        # Trigger offloading
        self.send_request(self.gen_prompt(1), max_tokens=150)

        # Flush device cache to force remote storage access
        self.flush_cache()
```
**EN:** Helper method to trigger offloading and flush cache This block implements `trigger_offloading_and_flush` and captures one focused piece of the module's behavior.
**CN:** Helper method to trigger offloading and flush cache 该代码块实现 `trigger_offloading_and_flush`，承担模块行为中的一个聚焦逻辑片段。

### Lines 195-225: test case basic backup and prefetch / 测试用例 basic backup and prefetch
```python
    def test_basic_backup_and_prefetch(self):
        """Test storage and retrieval of large context through remote cache"""
        print("\n=== Testing Large Context Cache Storage & Retrieval ===")

        # Generate substantial context that will be cached
        base_prompt = self.gen_prompt(768)

        # First request - populate cache
        print("Step 1: Populating cache with large context...")
        response1 = self.send_request(base_prompt, max_tokens=150)
        self.assertIsNotNone(response1)

        # Flush device cache to force remote storage access
        self.trigger_offloading_and_flush()

        # Second request with extended prompt - should hit remote cache
        print("Step 2: Testing cache hit from remote storage...")

        start_time = time.time()
        response2 = self.send_request(base_prompt, max_tokens=150)
        retrieval_time = time.time() - start_time

        cached_tokens = self.get_cached_tokens(response2)
        print(
            f"Remote cache retrieval time: {retrieval_time:.3f}s, cached_tokens={cached_tokens}"
        )

        # Assert cached tokens indicate a remote hit
        self.assertGreater(
            cached_tokens, 700, "Expected significant cached tokens for remote hit"
        )
```
**EN:** Test storage and retrieval of large context through remote cache This test exercises `test_basic_backup_and_prefetch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test storage and retrieval of large context through remote cache 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_backup_and_prefetch`。

### Lines 228-229: class TestHiCacheStoragePageFirstLayout declaration / 类 TestHiCacheStoragePageFirstLayout 声明
```python
@unittest.skipIf(is_in_ci(), "To reduce the CI execution time.")
class TestHiCacheStoragePageFirstLayout(HiCacheStorageBaseMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `HiCacheStorageBaseMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `HiCacheStorageBaseMixin`, `CustomTestCase`。

### Lines 230-230: supporting statements / 辅助语句
```python
    """Page first layout tests for HiCache Storage functionality"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 232-236: method get additional server args and env / 方法 get additional server args and env
```python
    @classmethod
    def _get_additional_server_args_and_env(cls):
        """Get additional server arguments specific to configuration - override in subclasses"""
        server_args = {"--hicache-mem-layout": "page_first"}
        return server_args, {}
```
**EN:** Get additional server arguments specific to configuration - override in subclasses This block implements `_get_additional_server_args_and_env` and captures one focused piece of the module's behavior.
**CN:** Get additional server arguments specific to configuration - override in subclasses 该代码块实现 `_get_additional_server_args_and_env`，承担模块行为中的一个聚焦逻辑片段。

### Lines 239-240: class TestHiCacheStorageMLA declaration / 类 TestHiCacheStorageMLA 声明
```python
@unittest.skipIf(is_in_ci(), "To reduce the CI execution time.")
class TestHiCacheStorageMLA(HiCacheStorageBaseMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `HiCacheStorageBaseMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `HiCacheStorageBaseMixin`, `CustomTestCase`。

### Lines 241-241: supporting statements / 辅助语句
```python
    """MLA Model tests for HiCache Storage functionality"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 243-246: method get model name / 方法 get model name
```python
    @classmethod
    def _get_model_name(cls):
        """Use MLA model for testing"""
        return DEFAULT_MLA_MODEL_NAME_FOR_TEST
```
**EN:** Use MLA model for testing This block implements `_get_model_name` and captures one focused piece of the module's behavior.
**CN:** Use MLA model for testing 该代码块实现 `_get_model_name`，承担模块行为中的一个聚焦逻辑片段。

### Lines 248-252: method get additional server args and env / 方法 get additional server args and env
```python
    @classmethod
    def _get_additional_server_args_and_env(cls):
        """Get additional server arguments specific to configuration - override in subclasses"""
        server_args = {"--tp-size": 2}
        return server_args, {}
```
**EN:** Get additional server arguments specific to configuration - override in subclasses This block implements `_get_additional_server_args_and_env` and captures one focused piece of the module's behavior.
**CN:** Get additional server arguments specific to configuration - override in subclasses 该代码块实现 `_get_additional_server_args_and_env`，承担模块行为中的一个聚焦逻辑片段。

### Lines 255-255: class TestHiCacheStoragePageFirstDirectIO declaration / 类 TestHiCacheStoragePageFirstDirectIO 声明
```python
class TestHiCacheStoragePageFirstDirectIO(HiCacheStorageBaseMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `HiCacheStorageBaseMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `HiCacheStorageBaseMixin`, `CustomTestCase`。

### Lines 256-256: supporting statements / 辅助语句
```python
    """Page first direct tests for HiCache Storage functionality"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 258-266: method get additional server args and env / 方法 get additional server args and env
```python
    @classmethod
    def _get_additional_server_args_and_env(cls):
        """Get additional server arguments specific to configuration - override in subclasses"""
        server_args = {
            "--hicache-mem-layout": "page_first_direct",
            "--hicache-io-backend": "direct",
            "--tp-size": 2,
        }
        return server_args, {}
```
**EN:** Get additional server arguments specific to configuration - override in subclasses This block implements `_get_additional_server_args_and_env` and captures one focused piece of the module's behavior.
**CN:** Get additional server arguments specific to configuration - override in subclasses 该代码块实现 `_get_additional_server_args_and_env`，承担模块行为中的一个聚焦逻辑片段。

### Lines 269-269: class TestHiCacheStorageAccuracy declaration / 类 TestHiCacheStorageAccuracy 声明
```python
class TestHiCacheStorageAccuracy(HiCacheStorageBaseMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `HiCacheStorageBaseMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `HiCacheStorageBaseMixin`, `CustomTestCase`。

### Lines 270-270: supporting statements / 辅助语句
```python
    """Accuracy tests for HiCache Storage functionality"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 272-280: method get additional server args and env / 方法 get additional server args and env
```python
    @classmethod
    def _get_additional_server_args_and_env(cls):
        """Get additional server arguments specific to configuration - override in subclasses"""
        server_args = {
            "--tp-size": 2,
            "--hicache-ratio": 1.5,
        }

        return server_args, {}
```
**EN:** Get additional server arguments specific to configuration - override in subclasses This block implements `_get_additional_server_args_and_env` and captures one focused piece of the module's behavior.
**CN:** Get additional server arguments specific to configuration - override in subclasses 该代码块实现 `_get_additional_server_args_and_env`，承担模块行为中的一个聚焦逻辑片段。

### Lines 282-284: test case eval accuracy / 测试用例 eval accuracy
```python
    def test_eval_accuracy(self):
        """Test eval accuracy with cache persistence across cache flushes"""
        run_eval_accuracy_test(self)
```
**EN:** Test eval accuracy with cache persistence across cache flushes This test exercises `test_eval_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test eval accuracy with cache persistence across cache flushes 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_eval_accuracy`。

### Lines 287-330: helper routine run eval accuracy test / 辅助流程 run eval accuracy test
```python
def run_eval_accuracy_test(test_instance, accuracy_threshold: float = 0.03):
    """Generic eval accuracy test with configurable accuracy threshold

    Args:
        test_instance: The test class instance that provides base_host, base_port, flush_cache, and assert methods
    """
    print("\n=== Testing Eval Accuracy with Cache Persistence ===")

    # First evaluation - populate cache
    print("Phase 1: Running initial GSM8K evaluation to populate cache...")
    args_initial = SimpleNamespace(
        base_url=f"http://{test_instance.base_host}:{test_instance.base_port}",
        eval_name="gsm8k",
        api="completion",
        max_tokens=512,
        num_examples=200,
        num_threads=64,
    )
    metrics_initial = run_eval(args_initial)

    # Flush cache to force remote storage access
    print("Phase 2: Flushing device cache...")
    test_instance.flush_cache()

    # Second evaluation - should use remote cache
    print("Phase 3: Running second GSM8K evaluation using remote cache...")
    metrics_cached = run_eval(args_initial)

    # Verify accuracy consistency
    accuracy_diff = abs(metrics_initial["score"] - metrics_cached["score"])
    print(f"Accuracy difference: {accuracy_diff:.4f}")

    # Assertions
    test_instance.assertGreater(
        metrics_initial["score"], 0.6, "Initial accuracy should be reasonable"
    )
    test_instance.assertGreater(
        metrics_cached["score"], 0.6, "Cached accuracy should be reasonable"
    )
    test_instance.assertLess(
        accuracy_diff,
        accuracy_threshold,
        "Accuracy should be consistent between cache states",
    )
```
**EN:** Generic eval accuracy test with configurable accuracy threshold Args: test_instance: The test class instance that provides base_host, base_port, flush_cache, and assert methods This helper encapsulates `run_eval_accuracy_test` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Generic eval accuracy test with configurable accuracy threshold Args: test_instance: The test class instance that provides base_host, base_port, flush_cache, and assert methods 该辅助函数封装了 `run_eval_accuracy_test`，以便周围测试复用准备、执行或校验逻辑。

### Lines 333-334: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `HiCacheStorageBaseMixin`: Base mixin class with common setup and utilities / 用于组织相关测试、夹具或辅助方法。
- `TestHiCacheStoragePageFirstLayout`: Page first layout tests for HiCache Storage functionality / 用于组织相关测试、夹具或辅助方法。
- `TestHiCacheStorageMLA`: MLA Model tests for HiCache Storage functionality / 用于组织相关测试、夹具或辅助方法。
- `TestHiCacheStoragePageFirstDirectIO`: Page first direct tests for HiCache Storage functionality / 用于组织相关测试、夹具或辅助方法。
- `TestHiCacheStorageAccuracy`: Accuracy tests for HiCache Storage functionality / 用于组织相关测试、夹具或辅助方法。
- `run_eval_accuracy_test`: Generic eval accuracy test with configurable accuracy threshold Args: test_instance: The test class instance that provides base_host, base_port, flush_cache, and assert methods / 该辅助函数封装了 `run_eval_accuracy_test`，以便周围测试复用准备、执行或校验逻辑。
- `HiCacheStorageBaseMixin.setUpClass`: Set up test environment and launch server once for all tests / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `HiCacheStorageBaseMixin.tearDownClass`: Clean up test environment / 该流程会在相关测试结束后释放资源并恢复状态。
- `HiCacheStorageBaseMixin._get_model_name`: Get model name for the test configuration - override in subclasses / 该代码块实现 `_get_model_name`，承担模块行为中的一个聚焦逻辑片段。
- `HiCacheStorageBaseMixin._get_base_server_args`: Get base server arguments - can be extended in subclasses / 该代码块实现 `_get_base_server_args`，承担模块行为中的一个聚焦逻辑片段。
- `HiCacheStorageBaseMixin._get_additional_server_args_and_env`: Get additional server arguments specific to configuration - override in subclasses / 该代码块实现 `_get_additional_server_args_and_env`，承担模块行为中的一个聚焦逻辑片段。
- `HiCacheStorageBaseMixin._launch_server_with_hicache`: Launch server with HiCache enabled / 该代码块实现 `_launch_server_with_hicache`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `random`, `tempfile`, `time`, `unittest`, `types`, `typing`, `urllib.parse`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.benchmark.utils`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`, `sglang.utils`

- **Total lines / 总行数**: 334
