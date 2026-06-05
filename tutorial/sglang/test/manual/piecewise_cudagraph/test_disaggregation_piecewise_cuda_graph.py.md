# test_disaggregation_piecewise_cuda_graph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/piecewise_cudagraph/test_disaggregation_piecewise_cuda_graph.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `disaggregation piecewise cuda graph` scenario in `test/manual/piecewise_cudagraph`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/piecewise_cudagraph` 中的 `disaggregation piecewise cuda graph` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and shared helpers / 导入与共享辅助项
```python
import unittest
from types import SimpleNamespace

from sglang.test.run_eval import run_eval
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    popen_launch_pd_server,
)
```
**EN:** This range imports `unittest`, `types`, `sglang.test.run_eval` and `sglang.test.server_fixtures.disaggregation_fixture`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 15-18: Class definition for TestDisaggregationPiecewiseCudaGraph / 类定义
```python
class TestDisaggregationPiecewiseCudaGraph(PDDisaggregationServerBase):
    """Test piecewise CUDA graph support in disaggregation prefill server"""

    @classmethod
```
**EN:** This range declares `TestDisaggregationPiecewiseCudaGraph`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 19-31: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        super().setUpClass()
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST

        # Start servers
        cls.start_prefill()
        cls.start_decode()

        # Wait for both to be ready
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `start_prefill`, `start_decode`, `wait_server_ready` and `launch_lb`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 32-33: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 34-39: Helper routines around start_prefill / 辅助例程
```python
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--tp",
```
**EN:** This range implements helper routine(s) `start_prefill` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 40-49: Process control logic / 进程控制逻辑
```python
            "1",
            "--enforce-piecewise-cuda-graph",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `popen_launch_pd_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 50-51: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 52-64: Helper routines around start_decode / 辅助例程
```python
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--tp",
            "1",
            "--base-gpu-id",
            "1",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
```
**EN:** This range implements helper routine(s) `start_decode` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `popen_launch_pd_server`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 65-68: Scenario logic / 场景逻辑
```python
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 70-84: Test routines around test_gsm8k_accuracy / 测试例程
```python
    def test_gsm8k_accuracy(self):
        """Verify that piecewise cuda graph works correctly in prefill server"""
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
        print(f"GSM8K accuracy with piecewise cuda graph: {metrics['score']:.3f}")

        self.assertGreater(metrics["score"], 0.62)
```
**EN:** This range defines concrete test routine(s) `test_gsm8k_accuracy`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `SimpleNamespace`, `run_eval` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 85-88: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Environment-aware configuration / 环境感知配置

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `types`, `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.run_eval`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
