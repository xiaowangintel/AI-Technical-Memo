# test_local_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/attention/test_local_attn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `local attn` scenario in `test/manual/attention`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/attention` 中的 `local attn` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and shared helpers / 导入与共享辅助项
```python
import os
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import get_device_sm, kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST_LOCAL_ATTENTION,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)


# Local attention with FA3 (requires SM 90+ / H100, tp=4)
@unittest.skipIf(get_device_sm() < 90, "Test requires CUDA SM 90 or higher")
```
**EN:** This range imports `os`, `unittest`, `types` and `requests`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `FA3`, `skipIf` and `get_device_sm`.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 20-25: Class definition for TestFlashAttention3LocalAttn / 类定义
```python
class TestFlashAttention3LocalAttn(CustomTestCase):
    model = DEFAULT_MODEL_NAME_FOR_TEST_LOCAL_ATTENTION
    base_url = DEFAULT_URL_FOR_TEST
    accuracy_threshold = 0.90

    @classmethod
```
**EN:** This range declares `TestFlashAttention3LocalAttn`, which organizes the scenario as a reusable test-oriented class. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 26-36: Helper routines around get_server_args / 辅助例程
```python
    def get_server_args(cls):
        return [
            "--cuda-graph-max-bs",
            "2",
            "--attention-backend",
            "fa3",
            "--tp",
            "4",
            "--context-length",
            "1000000",
        ]
```
**EN:** This range implements helper routine(s) `get_server_args` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 37-38: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 39-44: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=cls.get_server_args(),
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server` and `get_server_args`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 45-46: Environment-driven configuration / 环境驱动配置
```python
            env=os.environ,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 47-48: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 49-50: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 52-69: Test routines around test_gsm8k / 测试例程
```python
    def test_gsm8k(self):
        requests.get(self.base_url + "/flush_cache")

        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=100,
            num_threads=128,
            num_shots=4,
        )
        metrics = run_eval(args)
        print(f"{metrics=}")

        # Use the appropriate metric key based on the test class
        metric_key = "score"
```
**EN:** This range defines concrete test routine(s) `test_gsm8k`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get`, `SimpleNamespace` and `run_eval`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 70-70: Assertions and result checks / 断言与结果检查
```python
        self.assertGreater(metrics[metric_key], self.accuracy_threshold)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 71-74: Script entry point / 脚本入口
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
- **Standard Library / 标准库**: `os`, `types`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.test_utils`
