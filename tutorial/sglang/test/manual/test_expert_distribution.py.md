# test_expert_distribution.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_expert_distribution.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `expert distribution` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `expert distribution` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and shared helpers / 导入与共享辅助项
```python
import tempfile
import unittest
from pathlib import Path

import requests
import torch

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This range imports `tempfile`, `unittest`, `pathlib` and `requests`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 18-18: Class definition for TestExpertDistribution / 类定义
```python
class TestExpertDistribution(CustomTestCase):
```
**EN:** This range declares `TestExpertDistribution`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 19-29: Test routines around test_expert_distribution_record / 测试例程
```python
    def test_expert_distribution_record(self):
        # TODO: Add tests for DeepEP gatherer (currently our CI cannot run that)
        for info in [
            dict(model_path="deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct"),
            dict(model_path="Qwen/Qwen1.5-MoE-A2.7B"),
            dict(model_path="Qwen/Qwen1.5-MoE-A2.7B", tp_size=2),
            dict(model_path="Qwen/Qwen1.5-MoE-A2.7B", mode="per_pass"),
            dict(model_path="Qwen/Qwen1.5-MoE-A2.7B", mode="per_token"),
        ]:
            with self.subTest(info=info):
                self._execute_core(**info)
```
**EN:** This range defines concrete test routine(s) `test_expert_distribution_record`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `gatherer`, `subTest` and `_execute_core`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 31-42: Helper routines around _execute_core / 辅助例程
```python
    def _execute_core(self, model_path: str, mode: str = "stat", tp_size: int = 1):
        """Test expert distribution record endpoints"""
        with tempfile.TemporaryDirectory() as tmp_dir:
            envs.SGLANG_EXPERT_DISTRIBUTION_RECORDER_DIR.set(tmp_dir)

            process = popen_launch_server(
                model_path,
                DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--trust-remote-code",
                    "--tp-size",
```
**EN:** This range implements helper routine(s) `_execute_core` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `TemporaryDirectory` and `popen_launch_server`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 43-55: Request and response handling / 请求与响应处理
```python
                    str(tp_size),
                    "--expert-distribution-recorder-mode",
                    mode,
                    "--disable-cuda-graph",
                    "--disable-overlap-schedule",
                ],
            )

            try:
                # Start recording
                response = requests.post(
                    f"{DEFAULT_URL_FOR_TEST}/start_expert_distribution_record"
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `post`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 56-67: Assertions and result checks / 断言与结果检查
```python
                self.assertEqual(response.status_code, 200)

                # Make some requests to generate expert distribution data
                response = requests.post(
                    f"{DEFAULT_URL_FOR_TEST}/generate",
                    json={
                        "text": "The capital of France is",
                        "sampling_params": {
                            "temperature": 0,
                            "max_new_tokens": 32,
                        },
                    },
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual` and `post`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 68-80: Assertions and result checks / 断言与结果检查
```python
                )
                self.assertEqual(response.status_code, 200)

                # Stop recording
                response = requests.post(
                    f"{DEFAULT_URL_FOR_TEST}/stop_expert_distribution_record"
                )
                self.assertEqual(response.status_code, 200)

                # Dump the recorded data
                response = requests.post(
                    f"{DEFAULT_URL_FOR_TEST}/dump_expert_distribution_record"
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual` and `post`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 81-92: Assertions and result checks / 断言与结果检查
```python
                self.assertEqual(response.status_code, 200)

                # Check data rows
                data = torch.load(
                    list(Path(tmp_dir).glob("*.pt"))[0], weights_only=True
                )
                print(f"{data=}")

                if mode in ["per_pass", "per_token"]:
                    self.assertGreater(len(data), 0, "Should contain data rows")
                else:
                    logical_count = data["logical_count"]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`, `load`, `Path` and `glob`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 93-97: Assertions and result checks / 断言与结果检查
```python
                    print(f"{logical_count.sum()=} {logical_count=}")
                    self.assertTrue(logical_count.sum() > 0)

            finally:
                kill_process_tree(process.pid)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `kill_process_tree`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 98-101: Script entry point / 脚本入口
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
- Environment-aware configuration / 环境感知配置
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `pathlib`, `tempfile`, `unittest`
- **Third-party / 第三方库**: `requests`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.test_utils`
