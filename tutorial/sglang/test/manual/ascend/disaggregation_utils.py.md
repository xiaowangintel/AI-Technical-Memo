# disaggregation_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/ascend/disaggregation_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This helper module supports manual tests under `test/manual/ascend`. It centralizes reusable setup, cleanup, or environment-handling logic shared by multiple scenarios. / 该辅助模块服务于 `test/manual/ascend` 下的手动测试。它集中封装多个场景共享的初始化、清理或环境处理逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and shared helpers / 导入与共享辅助项
```python
import logging
import os
import time
import warnings
from urllib.parse import urlparse

import requests

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_with_error_check,
)

logger = logging.getLogger(__name__)
```
**EN:** This range imports `logging`, `os`, `time` and `warnings`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `getLogger`.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 21-22: Class definition for TestDisaggregationBase / 类定义
```python
class TestDisaggregationBase(CustomTestCase):
    @classmethod
```
**EN:** This range declares `TestDisaggregationBase`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 23-45: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        parsed_url = urlparse(DEFAULT_URL_FOR_TEST)
        cls.base_host = parsed_url.hostname
        base_port = str(parsed_url.port)
        cls.lb_port = base_port
        cls.prefill_port = f"{int(base_port) + 100}"
        cls.decode_port = f"{int(base_port) + 200}"
        cls.prefill_url = f"http://{cls.base_host}:{cls.prefill_port}"
        cls.decode_url = f"http://{cls.base_host}:{cls.decode_port}"
        cls.lb_url = f"http://{cls.base_host}:{cls.lb_port}"
        print(f"{cls.base_host=} {cls.lb_port=} {cls.prefill_port=} {cls.decode_port=}")
        cls.process_lb, cls.process_decode, cls.process_prefill = None, None, None

        # config transfer backend and rdma devices
        cls.transfer_backend = [
            "--disaggregation-transfer-backend",
            envs.SGLANG_TEST_PD_DISAGG_BACKEND.get(),
        ]
        cls.rdma_devices = [
            "--disaggregation-ib-device",
            envs.SGLANG_TEST_PD_DISAGG_DEVICES.get(),
        ]
        if cls.rdma_devices[1] is None:
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `urlparse` and `get`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 46-47: Scenario logic / 场景逻辑
```python
            cls.rdma_devices = []
            msg = "No RDMA devices specified for disaggregation test, using default settings."
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 48-48: Scenario logic / 场景逻辑
```python
            warnings.warn(msg)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `warn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 49-50: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 51-69: Helper routines around launch_lb / 辅助例程
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
        cls.process_lb = popen_with_error_check(lb_command)
        cls.wait_server_ready(cls.lb_url + "/health")
```
**EN:** This range implements helper routine(s) `launch_lb` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `join`, `popen_with_error_check` and `wait_server_ready`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 71-71: Scenario logic / 场景逻辑
```python
    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 72-85: Helper routines around wait_server_ready / 辅助例程
```python
    def wait_server_ready(cls, url, timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH):
        start_time = time.perf_counter()
        while True:
            try:
                response = requests.get(url)
                if response.status_code == 200:
                    print(f"Server {url} is ready")
                    return
            except Exception:
                pass

            if time.perf_counter() - start_time > timeout:
                raise RuntimeError(f"Server {url} failed to start in {timeout}s")
            time.sleep(1)
```
**EN:** This range implements helper routine(s) `wait_server_ready` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `perf_counter`, `get`, `RuntimeError` and `sleep`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 86-87: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 88-95: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        for process in [cls.process_lb, cls.process_decode, cls.process_prefill]:
            if process:
                try:
                    kill_process_tree(process.pid)
                except Exception as e:
                    print(f"Error killing process {process.pid}: {e}")
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 96-97: Scenario logic / 场景逻辑
```python
        # wait for 5 seconds
        time.sleep(5)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `sleep`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 100-124: Helper routines around get_rdma_devices_args / 辅助例程
```python
def get_rdma_devices_args():
    def _parse_list_env(var_name: str):
        val = os.getenv(var_name)
        if not val:
            return None
        items = [x.strip() for x in val.split(",") if x.strip()]
        return items or None

    def _pick_default_pair(rdma_all_devices):
        return [rdma_all_devices[0], rdma_all_devices[len(rdma_all_devices) // 2]]

    rdma_all_devices = _parse_list_env("SGLANG_CI_RDMA_ALL_DEVICES") or [
        f"mlx5_roce{i}" for i in range(8)
    ]
    logger.info("Resolved rdma_all_devices=%s", rdma_all_devices)

    n_rdma = len(rdma_all_devices)

    # 1. Get visible GPU indices
    cuda_visible_devices = os.getenv("CUDA_VISIBLE_DEVICES")
    if not cuda_visible_devices:
        warnings.warn("CUDA_VISIBLE_DEVICES is not set. Using default RDMA devices.")
        return ",".join(_pick_default_pair(rdma_all_devices))

    try:
```
**EN:** This range implements helper routine(s) `get_rdma_devices_args`, `_parse_list_env` and `_pick_default_pair` so setup, transformation, or validation logic can be reused cleanly. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `getenv`, `strip`, `split` and `info`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 125-149: Scenario logic / 场景逻辑
```python
        # Convert to list of integers (handling possible spaces and empty strings)
        gpu_indices = [
            int(idx.strip()) for idx in cuda_visible_devices.split(",") if idx.strip()
        ]
        if not gpu_indices or len(gpu_indices) > 4:
            return ",".join(_pick_default_pair(rdma_all_devices))
    except ValueError:
        warnings.warn(f"Invalid CUDA_VISIBLE_DEVICES format: {cuda_visible_devices}")
        return ",".join(_pick_default_pair(rdma_all_devices))

    # 2. Calculate base RDMA index group (each group of 4 GPUs uses consecutive devices)
    base_rdma_group = (min(gpu_indices) // 4) * 4
    for gpu_idx in gpu_indices:
        if not (base_rdma_group <= gpu_idx < base_rdma_group + 4):
            warnings.warn(
                f"GPU index {gpu_idx} is outside expected group "
                f"{base_rdma_group}-{base_rdma_group+3}"
            )

    # 3. Generate RDMA device names
    rdma_devices = []
    for gpu_idx in gpu_indices:
        nic_index = gpu_idx // (8 // n_rdma)
        rdma_devices.append(rdma_all_devices[nic_index])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `integers`, `strip`, `split` and `join`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 150-153: Scenario logic / 场景逻辑
```python
    if not rdma_devices:
        return ",".join(_pick_default_pair(rdma_all_devices))

    return ",".join(rdma_devices)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `join` and `_pick_default_pair`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Environment-aware configuration / 环境感知配置
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `logging`, `os`, `time`, `urllib.parse`, `warnings`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.test_utils`
