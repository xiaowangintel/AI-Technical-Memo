# test_disaggregation_hybrid_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_disaggregation_hybrid_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates disaggregation hybrid attention behavior in SGLang's distributed area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 分布式 领域中与 disaggregation hybrid attention 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    is_in_ci,
    popen_launch_pd_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`。

### Lines 15-15: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=310, stage="extra-b", runner_config="8-gpu-h200")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 18-19: class TestDisaggregationHybridAttentionMamba declaration / 类 TestDisaggregationHybridAttentionMamba 声明
```python
@unittest.skipIf(is_in_ci(), "Temporarily disable the flaky test.")
class TestDisaggregationHybridAttentionMamba(PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PDDisaggregationServerBase`。

### Lines 20-33: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = "Qwen/Qwen3-Next-80B-A3B-Instruct"

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

### Lines 35-52: helper routine start prefill / 辅助流程 start prefill
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

### Lines 54-73: helper routine start decode / 辅助流程 start decode
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

### Lines 75-88: test case gsm8k / 测试用例 gsm8k
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

        self.assertGreater(metrics["score"], 0.93)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 91-91: class TestDisaggregationHybridAttentionMambaExtraBuffer declaration / 类 TestDisaggregationHybridAttentionMambaExtraBuffer 声明
```python
class TestDisaggregationHybridAttentionMambaExtraBuffer(PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PDDisaggregationServerBase`。

### Lines 92-105: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = "Qwen/Qwen3-Next-80B-A3B-Instruct"

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

### Lines 107-126: helper routine start prefill / 辅助流程 start prefill
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
            "--mamba-scheduler-strategy",
            "extra_buffer",
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

### Lines 128-149: helper routine start decode / 辅助流程 start decode
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
            "--mamba-scheduler-strategy",
            "extra_buffer",
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

### Lines 151-165: test case gsm8k / 测试用例 gsm8k
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

        # TODO: Fix PD disaggregation accuracy issue (https://github.com/sgl-project/sglang/issues/21744) and increase the threshold back to 0.93.
        self.assertGreater(metrics["score"], 0.90)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 168-168: class TestDisaggregationHybridAttentionMambaDPDecode declaration / 类 TestDisaggregationHybridAttentionMambaDPDecode 声明
```python
class TestDisaggregationHybridAttentionMambaDPDecode(PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PDDisaggregationServerBase`。

### Lines 169-169: supporting statements / 辅助语句
```python
    """Test with prefill tp=2 and decode tp=2/dp=2 with dp-attention enabled."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 171-184: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = "Qwen/Qwen3-Next-80B-A3B-Instruct"

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

### Lines 186-203: helper routine start prefill / 辅助流程 start prefill
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

### Lines 205-228: helper routine start decode / 辅助流程 start decode
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
            "--dp",
            "2",
            "--enable-dp-attention",
            "--enable-dp-lm-head",
            "--base-gpu-id",
            "2",
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

### Lines 230-244: test case gsm8k / 测试用例 gsm8k
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

        # TODO: Fix PD disaggregation accuracy issue (https://github.com/sgl-project/sglang/issues/21744) and increase the threshold back to 0.93.
        self.assertGreater(metrics["score"], 0.90)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 247-248: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestDisaggregationHybridAttentionMamba`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDisaggregationHybridAttentionMambaExtraBuffer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDisaggregationHybridAttentionMambaDPDecode`: Test with prefill tp=2 and decode tp=2/dp=2 with dp-attention enabled. / 用于组织相关测试、夹具或辅助方法。
- `TestDisaggregationHybridAttentionMamba.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDisaggregationHybridAttentionMamba.start_prefill`: This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。
- `TestDisaggregationHybridAttentionMamba.start_decode`: This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestDisaggregationHybridAttentionMamba.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `TestDisaggregationHybridAttentionMambaExtraBuffer.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDisaggregationHybridAttentionMambaExtraBuffer.start_prefill`: This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。
- `TestDisaggregationHybridAttentionMambaExtraBuffer.start_decode`: This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestDisaggregationHybridAttentionMambaExtraBuffer.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `TestDisaggregationHybridAttentionMambaDPDecode.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`

- **Total lines / 总行数**: 248
