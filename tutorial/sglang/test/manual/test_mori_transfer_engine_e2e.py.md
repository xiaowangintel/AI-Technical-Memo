# test_mori_transfer_engine_e2e.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_mori_transfer_engine_e2e.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `mori transfer engine e2e` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `mori transfer engine e2e` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and shared helpers / 导入与共享辅助项
```python
import os
import subprocess
import unittest

import requests

from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    popen_launch_pd_server,
)
```
**EN:** This range imports `os`, `subprocess`, `unittest` and `requests`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 17-27: Class definition for TestMoriTransferEngineE2E / 类定义
```python
class TestMoriTransferEngineE2E(PDDisaggregationServerBase):
    """
    Run:
        SGLANG_MORI_MANUAL_E2E=1 python3 test/manual/test_mori_transfer_engine_e2e.py

    Optional:
    - SGLANG_MORI_E2E_TEST_MODEL: override model (defaults to a small test model)
    - SGLANG_TEST_PD_DISAGG_DEVICES: RDMA devices string, e.g. "mlx5_roce0,mlx5_roce4"
    """

    @classmethod
```
**EN:** This range declares `TestMoriTransferEngineE2E`, which organizes the scenario as a reusable test-oriented class. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `model`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 28-41: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if os.environ.get("SGLANG_MORI_MANUAL_E2E", "") not in ("1", "true", "True"):
            raise unittest.SkipTest(
                "Set SGLANG_MORI_MANUAL_E2E=1 to run this manual MORI E2E test."
            )

        try:
            import torch

            if not torch.cuda.is_available():
                raise unittest.SkipTest("torch.cuda is not available.")
        except Exception as e:
            raise unittest.SkipTest(f"torch is not available/usable: {e}")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `get`, `in`, `SkipTest` and `is_available`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 42-52: Request and response handling / 请求与响应处理
```python
        # Force the disaggregation fixture to use MORI backend in local/manual runs.
        os.environ["SGLANG_TEST_PD_DISAGG_BACKEND"] = "mori"

        super().setUpClass()

        cls.model = os.environ.get(
            "SGLANG_MORI_E2E_TEST_MODEL", DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        )

        cls.start_prefill()
        cls.start_decode()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `setUpClass`, `get`, `start_prefill` and `start_decode`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 53-65: Scenario logic / 场景逻辑
```python

        cls.wait_server_ready(
            cls.prefill_url + "/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            process=cls.process_prefill,
        )
        cls.wait_server_ready(
            cls.decode_url + "/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            process=cls.process_decode,
        )

        cls.launch_lb()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `wait_server_ready` and `launch_lb`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 67-67: Scenario logic / 场景逻辑
```python
    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 68-70: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        os.environ.pop("SGLANG_TEST_PD_DISAGG_BACKEND", None)
        super().tearDownClass()
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `pop`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 71-72: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 73-91: Helper routines around launch_lb / 辅助例程
```python
    def launch_lb(cls):
        lb_command = [
            "python3",
            "-m",
            "sglang_router.launch_router",
            "--pd-disaggregation",
            "--mini-lb",
            "--prefill",
            cls.prefill_url,
            "--decode",
            cls.decode_url,
            "--host",
            cls.base_host,
            "--port",
            cls.lb_port,
        ]
        print("Starting load balancer:", " ".join(lb_command))
        cls.process_lb = subprocess.Popen(lb_command, stdout=None, stderr=None)
        cls.wait_server_ready(
```
**EN:** This range implements helper routine(s) `launch_lb` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `join`, `Popen` and `wait_server_ready`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 92-95: Scenario logic / 场景逻辑
```python
            cls.lb_url + "/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            process=cls.process_lb,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 96-97: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 98-112: Helper routines around start_prefill / 辅助例程
```python
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--tp",
            "1",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This range implements helper routine(s) `start_prefill` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `popen_launch_pd_server`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 113-114: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 115-116: Helper routines around start_decode / 辅助例程
```python
    def start_decode(cls):
        decode_args = [
```
**EN:** This range implements helper routine(s) `start_decode` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 117-131: Process control logic / 进程控制逻辑
```python
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
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `popen_launch_pd_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 133-141: Test routines around test_generate_basic / 测试例程
```python
    def test_generate_basic(self):
        resp = requests.post(
            self.lb_url + "/generate",
            json={
                "text": "Hello",
                "sampling_params": {"temperature": 0, "max_new_tokens": 8},
            },
            timeout=120,
        )
```
**EN:** This range defines concrete test routine(s) `test_generate_basic`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `post`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 142-146: Assertions and result checks / 断言与结果检查
```python
        self.assertEqual(resp.status_code, 200, resp.text)
        out = resp.json()
        self.assertIn("text", out)
        self.assertIsInstance(out["text"], str)
        self.assertGreater(len(out["text"]), 0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`, `json`, `assertIn` and `assertIsInstance`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 149-161: Class definition for TestMoriTransferEngineTPMismatchE2E / 类定义
```python
class TestMoriTransferEngineTPMismatchE2E(PDDisaggregationServerBase):
    """Manual MORI PD-disaggregation E2E with TP mismatch.

    Scenario:
    - prefill: tp=2 (GPU 0-1)
    - decode:  tp=4 (GPU 2-5)

    Manual-only and requires >= 6 visible GPUs.
    """

    _PORT_DELTA = 10

    @classmethod
```
**EN:** This range declares `TestMoriTransferEngineTPMismatchE2E`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 162-173: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if os.environ.get("SGLANG_MORI_MANUAL_E2E", "") not in ("1", "true", "True"):
            raise unittest.SkipTest(
                "Set SGLANG_MORI_MANUAL_E2E=1 to run this manual MORI E2E test."
            )

        try:
            import torch

            if not torch.cuda.is_available():
                raise unittest.SkipTest("torch.cuda is not available.")
            if torch.cuda.device_count() < 6:
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `get`, `in`, `SkipTest` and `is_available`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 174-186: Environment-driven configuration / 环境驱动配置
```python
                raise unittest.SkipTest(
                    "TP-mismatch test requires >= 6 visible GPUs (prefill tp=2 + decode tp=4)."
                )
        except Exception as e:
            raise unittest.SkipTest(f"torch is not available/usable: {e}")

        os.environ["SGLANG_TEST_PD_DISAGG_BACKEND"] = "mori"
        super().setUpClass()

        # Shift ports to avoid clashing with TestMoriTransferEngineE2E.
        cls.lb_port = str(int(cls.lb_port) + cls._PORT_DELTA)
        cls.prefill_port = str(int(cls.prefill_port) + cls._PORT_DELTA)
        cls.decode_port = str(int(cls.decode_port) + cls._PORT_DELTA)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `SkipTest`, `GPUs` and `setUpClass`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 187-198: Request and response handling / 请求与响应处理
```python
        cls.prefill_url = f"http://{cls.base_host}:{cls.prefill_port}"
        cls.decode_url = f"http://{cls.base_host}:{cls.decode_port}"
        cls.lb_url = f"http://{cls.base_host}:{cls.lb_port}"

        cls.model = os.environ.get(
            "SGLANG_MORI_E2E_TEST_MODEL", DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        )

        cls.start_prefill()
        cls.start_decode()

        cls.wait_server_ready(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `get`, `start_prefill`, `start_decode` and `wait_server_ready`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 199-208: Scenario logic / 场景逻辑
```python
            cls.prefill_url + "/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            process=cls.process_prefill,
        )
        cls.wait_server_ready(
            cls.decode_url + "/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            process=cls.process_decode,
        )
        cls.launch_lb()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `wait_server_ready` and `launch_lb`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 209-210: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 211-213: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        os.environ.pop("SGLANG_TEST_PD_DISAGG_BACKEND", None)
        super().tearDownClass()
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `pop`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 214-215: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 216-223: Helper routines around launch_lb / 辅助例程
```python
    def launch_lb(cls):
        lb_command = [
            "python3",
            "-m",
            "sglang_router.launch_router",
            "--pd-disaggregation",
            "--mini-lb",
            "--prefill",
```
**EN:** This range implements helper routine(s) `launch_lb` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 224-238: Process control logic / 进程控制逻辑
```python
            cls.prefill_url,
            "--decode",
            cls.decode_url,
            "--host",
            cls.base_host,
            "--port",
            cls.lb_port,
        ]
        print("Starting load balancer:", " ".join(lb_command))
        cls.process_lb = subprocess.Popen(lb_command, stdout=None, stderr=None)
        cls.wait_server_ready(
            cls.lb_url + "/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            process=cls.process_lb,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `join`, `Popen` and `wait_server_ready`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 239-240: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 241-248: Helper routines around start_prefill / 辅助例程
```python
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--tp",
            "2",
        ]
```
**EN:** This range implements helper routine(s) `start_prefill` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 249-255: Process control logic / 进程控制逻辑
```python
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

### Lines 256-257: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 258-273: Helper routines around start_decode / 辅助例程
```python
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--tp",
            "4",
            "--base-gpu-id",
            "2",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
```
**EN:** This range implements helper routine(s) `start_decode` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `popen_launch_pd_server`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 274-274: Scenario logic / 场景逻辑
```python
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 276-289: Test routines around test_generate_with_tp_mismatch / 测试例程
```python
    def test_generate_with_tp_mismatch(self):
        resp = requests.post(
            self.lb_url + "/generate",
            json={
                "text": "Hello",
                "sampling_params": {"temperature": 0, "max_new_tokens": 8},
            },
            timeout=120,
        )
        self.assertEqual(resp.status_code, 200, resp.text)
        out = resp.json()
        self.assertIn("text", out)
        self.assertIsInstance(out["text"], str)
        self.assertGreater(len(out["text"]), 0)
```
**EN:** This range defines concrete test routine(s) `test_generate_with_tp_mismatch`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `post`, `assertEqual`, `json` and `assertIn`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 290-293: Script entry point / 脚本入口
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
- **Standard Library / 标准库**: `os`, `subprocess`, `unittest`
- **Third-party / 第三方库**: `requests`, `torch`
- **Project Modules / 项目模块**: `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
