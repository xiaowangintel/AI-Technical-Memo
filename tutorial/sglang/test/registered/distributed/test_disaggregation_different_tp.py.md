# test_disaggregation_different_tp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_disaggregation_different_tp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on distributed disaggregation different tp in SGLang. It drives an evaluation workflow and verifies the resulting quality signals. / 该测试模块用于分析 SGLang 中与 distributed disaggregation different tp 相关的实现或行为。 它会驱动评测流程，并验证最终得到的质量信号。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Import dependencies
```python
import os
import unittest
from types import SimpleNamespace

from sglang.srt.environ import envs
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_MODEL_NAME_FOR_TEST_MLA,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    popen_launch_pd_server,
    try_cached_model,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also runs an evaluation workflow to measure model quality and registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会运行评测流程以衡量模型质量，并将该用例注册到 CUDA CI 覆盖范围。

### Lines 19-19: Register CI metadata
```python
register_cuda_ci(est_time=375, stage="base-c", runner_config="8-gpu-h20")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 22-22: Define class TestDisaggregationMooncakePrefillLargerTP
```python
class TestDisaggregationMooncakePrefillLargerTP(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationMooncakePrefillLargerTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationMooncakePrefillLargerTP` 测试类，并说明它通过继承承担的职责。

### Lines 23-39: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Temporarily disable JIT DeepGEMM
        envs.SGLANG_ENABLE_JIT_DEEPGEMM.set(False)

        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST_MLA)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 41-60: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "4",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakePrefillLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakePrefillLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 62-83: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "2",
            "--base-gpu-id",
            "4",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakePrefillLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakePrefillLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 85-98: Run test: gsm8k
```python
    def test_gsm8k(self):
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
        print(f"Evaluation metrics: {metrics}")

        self.assertGreater(metrics["score"], 0.60)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 101-101: Define class TestDisaggregationMooncakeDecodeLargerTP
```python
class TestDisaggregationMooncakeDecodeLargerTP(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationMooncakeDecodeLargerTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationMooncakeDecodeLargerTP` 测试类，并说明它通过继承承担的职责。

### Lines 102-118: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Temporarily disable JIT DeepGEMM
        envs.SGLANG_ENABLE_JIT_DEEPGEMM.set(False)

        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST_MLA)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 120-139: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "2",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakeDecodeLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakeDecodeLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 141-162: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "4",
            "--base-gpu-id",
            "4",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakeDecodeLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakeDecodeLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 164-177: Run test: gsm8k
```python
    def test_gsm8k(self):
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
        print(f"Evaluation metrics: {metrics}")

        self.assertGreater(metrics["score"], 0.60)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 180-180: Define class TestDisaggregationMooncakeMHAPrefillLargerTP
```python
class TestDisaggregationMooncakeMHAPrefillLargerTP(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationMooncakeMHAPrefillLargerTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationMooncakeMHAPrefillLargerTP` 测试类，并说明它通过继承承担的职责。

### Lines 181-197: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Temporarily disable JIT DeepGEMM
        envs.SGLANG_ENABLE_JIT_DEEPGEMM.set(False)

        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 199-218: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "4",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakeMHAPrefillLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakeMHAPrefillLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 220-241: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "2",
            "--base-gpu-id",
            "4",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakeMHAPrefillLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakeMHAPrefillLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 243-256: Run test: gsm8k
```python
    def test_gsm8k(self):
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
        print(f"Evaluation metrics: {metrics}")

        self.assertGreater(metrics["score"], 0.60)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 259-259: Define class TestDisaggregationMooncakeMHADecodeLargerTP
```python
class TestDisaggregationMooncakeMHADecodeLargerTP(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationMooncakeMHADecodeLargerTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationMooncakeMHADecodeLargerTP` 测试类，并说明它通过继承承担的职责。

### Lines 260-276: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Temporarily disable JIT DeepGEMM
        envs.SGLANG_ENABLE_JIT_DEEPGEMM.set(False)

        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 278-297: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "2",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakeMHADecodeLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakeMHADecodeLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 299-320: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "4",
            "--base-gpu-id",
            "4",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakeMHADecodeLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakeMHADecodeLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 322-335: Run test: gsm8k
```python
    def test_gsm8k(self):
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
        print(f"Evaluation metrics: {metrics}")

        self.assertGreater(metrics["score"], 0.60)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 338-342: Define module constants
```python
STAGING_ENV = {
    "SGLANG_DISAGG_STAGING_BUFFER": "1",
    "SGLANG_DISAGG_STAGING_BUFFER_SIZE_MB": "64",
    "SGLANG_DISAGG_STAGING_POOL_SIZE_MB": "1024",
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 345-345: Define class TestDisaggregationStagingPrefillLargerTP
```python
class TestDisaggregationStagingPrefillLargerTP(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationStagingPrefillLargerTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationStagingPrefillLargerTP` 测试类，并说明它通过继承承担的职责。

### Lines 346-346: Document the class `TestDisaggregationStagingPrefillLargerTP`
```python
    """Prefill TP=4 -> Decode TP=2 with staging buffer enabled (MHA model)."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDisaggregationStagingPrefillLargerTP`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDisaggregationStagingPrefillLargerTP`的设计意图。

### Lines 348-361: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        envs.SGLANG_ENABLE_JIT_DEEPGEMM.set(False)

        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)

        cls.start_prefill()
        cls.start_decode()

        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 363-384: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "4",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        env = {**os.environ, **STAGING_ENV}
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
            env=env,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationStagingPrefillLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationStagingPrefillLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 386-409: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "2",
            "--base-gpu-id",
            "4",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        env = {**os.environ, **STAGING_ENV}
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
            env=env,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationStagingPrefillLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationStagingPrefillLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 411-423: Run test: gsm8k
```python
    def test_gsm8k(self):
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
        print(f"[Staging PrefillLargerTP] Evaluation metrics: {metrics}")
        self.assertGreater(metrics["score"], 0.60)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 426-426: Define class TestDisaggregationStagingDecodeLargerTP
```python
class TestDisaggregationStagingDecodeLargerTP(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationStagingDecodeLargerTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationStagingDecodeLargerTP` 测试类，并说明它通过继承承担的职责。

### Lines 427-427: Document the class `TestDisaggregationStagingDecodeLargerTP`
```python
    """Prefill TP=2 -> Decode TP=4 with staging buffer enabled (MHA model)."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDisaggregationStagingDecodeLargerTP`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDisaggregationStagingDecodeLargerTP`的设计意图。

### Lines 429-442: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        envs.SGLANG_ENABLE_JIT_DEEPGEMM.set(False)

        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)

        cls.start_prefill()
        cls.start_decode()

        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 444-465: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "2",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        env = {**os.environ, **STAGING_ENV}
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
            env=env,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationStagingDecodeLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationStagingDecodeLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 467-490: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "4",
            "--base-gpu-id",
            "4",
            "--enable-metrics",
            "--enable-request-time-stats-logging",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        env = {**os.environ, **STAGING_ENV}
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
            env=env,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationStagingDecodeLargerTP` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationStagingDecodeLargerTP` 内部调用，从而让场景结构更清晰。

### Lines 492-504: Run test: gsm8k
```python
    def test_gsm8k(self):
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
        print(f"[Staging DecodeLargerTP] Evaluation metrics: {metrics}")
        self.assertGreater(metrics["score"], 0.60)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 507-508: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.environ`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `run_eval`, `SimpleNamespace`, `unittest.main`, `DEFAULT_MODEL_NAME_FOR_TEST`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
