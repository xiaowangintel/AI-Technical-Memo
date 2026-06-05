# test_disaggregation_pp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_disaggregation_pp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates disaggregation pp behavior in SGLang's distributed area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 分布式 领域中与 disaggregation pp 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module imports and dependencies / 模块导入与依赖
```python
import time
import unittest
from types import SimpleNamespace

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    popen_launch_pd_server,
    try_cached_model,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `time`, `unittest`, `types`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `time`, `unittest`, `types`, `sglang.test.ci.ci_register`。

### Lines 17-17: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=216, stage="base-c", runner_config="8-gpu-h20")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 20-20: class TestDisaggregationPrefillPPAccuracy declaration / 类 TestDisaggregationPrefillPPAccuracy 声明
```python
class TestDisaggregationPrefillPPAccuracy(PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PDDisaggregationServerBase`。

### Lines 21-34: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 36-56: helper routine start prefill / 辅助流程 start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--pp-size",
            "2",
            "--disable-overlap-schedule",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。

### Lines 58-77: helper routine start decode / 辅助流程 start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--base-gpu-id",
            "4",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 79-94: test case gsm8k / 测试用例 gsm8k
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
        print(f"{metrics=}")

        self.assertGreater(metrics["score"], 0.24)
        # Wait a little bit so that the memory check happens.
        time.sleep(5)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 97-97: class TestDisaggregationPrefillPPDynamicChunkAccuracy declaration / 类 TestDisaggregationPrefillPPDynamicChunkAccuracy 声明
```python
class TestDisaggregationPrefillPPDynamicChunkAccuracy(PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PDDisaggregationServerBase`。

### Lines 98-111: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 113-134: helper routine start prefill / 辅助流程 start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--pp-size",
            "2",
            "--disable-overlap-schedule",
            "--enable-dynamic-chunking",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。

### Lines 136-155: helper routine start decode / 辅助流程 start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--base-gpu-id",
            "4",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 157-172: test case gsm8k / 测试用例 gsm8k
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
        print(f"{metrics=}")

        self.assertGreater(metrics["score"], 0.24)
        # Wait a little bit so that the memory check happens.
        time.sleep(5)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 175-175: class TestDisaggregationDecodePPAccuracy declaration / 类 TestDisaggregationDecodePPAccuracy 声明
```python
class TestDisaggregationDecodePPAccuracy(PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PDDisaggregationServerBase`。

### Lines 176-189: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 191-211: helper routine start prefill / 辅助流程 start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--pp-size",
            "2",
            "--disable-overlap-schedule",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。

### Lines 213-234: helper routine start decode / 辅助流程 start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp-size",
            "2",
            "--pp-size",
            "2",
            "--base-gpu-id",
            "4",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 236-251: test case gsm8k / 测试用例 gsm8k
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
        print(f"{metrics=}")

        self.assertGreater(metrics["score"], 0.24)
        # Wait a little bit so that the memory check happens.
        time.sleep(5)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 254-255: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestDisaggregationPrefillPPAccuracy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDisaggregationPrefillPPDynamicChunkAccuracy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDisaggregationDecodePPAccuracy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDisaggregationPrefillPPAccuracy.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDisaggregationPrefillPPAccuracy.start_prefill`: This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。
- `TestDisaggregationPrefillPPAccuracy.start_decode`: This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestDisaggregationPrefillPPAccuracy.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `TestDisaggregationPrefillPPDynamicChunkAccuracy.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDisaggregationPrefillPPDynamicChunkAccuracy.start_prefill`: This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。
- `TestDisaggregationPrefillPPDynamicChunkAccuracy.start_decode`: This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestDisaggregationPrefillPPDynamicChunkAccuracy.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `TestDisaggregationDecodePPAccuracy.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `time`, `unittest`, `types`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`

- **Total lines / 总行数**: 255
