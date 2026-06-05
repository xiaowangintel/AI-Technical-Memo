# test_disaggregation_decode_radix_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_disaggregation_decode_radix_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on distributed disaggregation decode radix cache in SGLang. It interacts with externally visible endpoints and verifies the returned behavior. / 该测试模块用于分析 SGLang 中与 distributed disaggregation decode radix cache 相关的实现或行为。 它会与对外暴露的端点交互，并验证返回结果是否正确。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Import dependencies
```python
import time
import unittest
from types import SimpleNamespace

import requests

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.cache_hit_kit import run_multiturn_cache_hit_test
from sglang.test.run_eval import run_eval
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    is_in_ci,
    try_cached_model,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also runs an evaluation workflow to measure model quality and registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会运行评测流程以衡量模型质量，并将该用例注册到 CUDA CI 覆盖范围。

### Lines 19-19: Register CI metadata
```python
register_cuda_ci(est_time=300, stage="base-c", runner_config="8-gpu-h20")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 22-27: Define helper: has nixl
```python
def _has_nixl():
    try:
        import nixl._api  # noqa: F401
    except ImportError:
        return False
    return True
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 30-35: Define helper: has mooncake
```python
def _has_mooncake():
    try:
        import mooncake.engine  # noqa: F401
    except ImportError:
        return False
    return True
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 38-38: Define class DisaggregationDecodeRadixCacheTestMixin
```python
class DisaggregationDecodeRadixCacheTestMixin:
```
**EN:** This declaration introduces the `DisaggregationDecodeRadixCacheTestMixin` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `DisaggregationDecodeRadixCacheTestMixin` 测试类，并说明它通过继承承担的职责。

### Lines 39-40: Declare DisaggregationDecodeRadixCacheTestMixin configuration
```python
    extra_decode_args = ["--disaggregation-decode-enable-radix-cache"]
    transfer_backend_name = None
```
**EN:** This block defines class-level settings that are shared across the `DisaggregationDecodeRadixCacheTestMixin` test methods.
**CN:** 该代码块定义了 `DisaggregationDecodeRadixCacheTestMixin` 各测试方法共享的类级配置。

### Lines 42-50: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)
        cls.transfer_backend = [
            "--disaggregation-transfer-backend",
            cls.transfer_backend_name,
        ]
        cls.launch_all()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 52-59: Define helper: assert process healthy
```python
    def _assert_process_healthy(self, name, process, url):
        self.assertIsNotNone(process, f"{name} process was not started")
        self.assertIsNone(
            process.poll(),
            f"{name} exited unexpectedly with code {process.returncode}",
        )
        response = requests.get(f"{url}/health", timeout=10)
        response.raise_for_status()
```
**EN:** This helper function encapsulates reusable logic inside `DisaggregationDecodeRadixCacheTestMixin` so the scenario stays organized. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `DisaggregationDecodeRadixCacheTestMixin` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 61-89: Run test: decode radix cache hits and workers stay alive
```python
    def test_decode_radix_cache_hits_and_workers_stay_alive(self):
        decode_info = requests.get(f"{self.decode_url}/server_info", timeout=10).json()
        self.assertFalse(
            decode_info.get("disable_radix_cache", True),
            "decode server did not enable radix cache",
        )

        result = run_multiturn_cache_hit_test(
            base_url=self.base_url,
            model_path=self.model,
            num_clients=4,
            num_rounds=3,
            request_length=384,
            output_length=64,
            max_parallel=4,
        )
        self.assertGreater(
            result["overall"]["total_cached_tokens"],
            0,
            "expected decode radix cache to reuse at least some tokens",
        )

        # Give the schedulers a short idle window so any post-request leak/crash
        # paths have a chance to surface before the liveness checks below.
        time.sleep(5)

        self._assert_process_healthy("load balancer", self.process_lb, self.lb_url)
        self._assert_process_healthy("prefill", self.process_prefill, self.prefill_url)
        self._assert_process_healthy("decode", self.process_decode, self.decode_url)
```
**EN:** This test method exercises decode radix cache hits and workers stay alive and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 decode radix cache hits and workers stay alive 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 91-120: Run test: gsm8k accuracy two passes
```python
    def test_gsm8k_accuracy_two_passes(self):
        """Run GSM8K twice to verify decode radix cache does not degrade accuracy."""
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=500,
            num_threads=100,
            num_shots=6,
        )

        metrics_first = run_eval(args)
        print(f"First run metrics: {metrics_first}")

        metrics_second = run_eval(args)
        print(f"Second run metrics: {metrics_second}")

        self.assertGreater(metrics_first["score"], 0.80)
        self.assertGreater(metrics_second["score"], 0.80)

        accuracy_drop = metrics_first["score"] - metrics_second["score"]
        self.assertLessEqual(
            accuracy_drop,
            0.03,
            f"Second run accuracy dropped by {accuracy_drop:.4f} "
            f"(first={metrics_first['score']:.4f}, second={metrics_second['score']:.4f}), "
            f"exceeds 3% threshold",
        )
```
**EN:** This test method exercises gsm8k accuracy two passes and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k accuracy two passes 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 123-124: Define class TestDisaggregationDecodeRadixCacheNixl
```python
@unittest.skip("Temporarily disabled until nixl backend is stable.")
class TestDisaggregationDecodeRadixCacheNixl(
```
**EN:** This declaration introduces the `TestDisaggregationDecodeRadixCacheNixl` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationDecodeRadixCacheNixl` 测试类，并说明它通过继承承担的职责。

### Lines 127-127: Declare TestDisaggregationDecodeRadixCacheNixl configuration
```python
    transfer_backend_name = "nixl"
```
**EN:** This block defines class-level settings that are shared across the `TestDisaggregationDecodeRadixCacheNixl` test methods.
**CN:** 该代码块定义了 `TestDisaggregationDecodeRadixCacheNixl` 各测试方法共享的类级配置。

### Lines 130-134: Define class TestDisaggregationDecodeRadixCacheMooncake
```python
@unittest.skipUnless(
    is_in_ci() or _has_mooncake(),
    "Mooncake is required for decode radix cache disaggregation coverage.",
)
class TestDisaggregationDecodeRadixCacheMooncake(
```
**EN:** This declaration introduces the `TestDisaggregationDecodeRadixCacheMooncake` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationDecodeRadixCacheMooncake` 测试类，并说明它通过继承承担的职责。

### Lines 137-137: Declare TestDisaggregationDecodeRadixCacheMooncake configuration
```python
    transfer_backend_name = "mooncake"
```
**EN:** This block defines class-level settings that are shared across the `TestDisaggregationDecodeRadixCacheMooncake` test methods.
**CN:** 该代码块定义了 `TestDisaggregationDecodeRadixCacheMooncake` 各测试方法共享的类级配置。

### Lines 140-141: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.kits.cache_hit_kit`, `sglang.test.run_eval`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `mooncake.engine`, `nixl._api`, `requests`, `time`, `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `run_eval`, `SimpleNamespace`, `requests.get`, `unittest.main`, `DEFAULT_MODEL_NAME_FOR_TEST`
