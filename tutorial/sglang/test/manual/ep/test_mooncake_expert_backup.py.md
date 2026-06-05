# test_mooncake_expert_backup.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/ep/test_mooncake_expert_backup.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `mooncake expert backup` scenario in `test/manual/ep`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/ep` 中的 `mooncake expert backup` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and shared helpers / 导入与共享辅助项
```python
import time
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.server_fixtures.disaggregation_fixture import get_rdma_devices_args
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST_MLA,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    CustomTestCase,
    popen_launch_pd_server,
)

ib_devices = get_rdma_devices_args()
```
**EN:** This range imports `time`, `unittest`, `types` and `requests`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `get_rdma_devices_args`.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 20-23: Class definition for TestBackup / 类定义
```python
class TestBackup(CustomTestCase):
    extra_args = []

    @classmethod
```
**EN:** This range declares `TestBackup`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 24-44: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST_MLA
        cls.base_port = 20000
        cls.base_url = f"http://127.0.0.1:{cls.base_port}"
        cls.num_processes = 2
        # TODO (stage 100): in the future, implement a specified multiprocess launcher
        cls.processes = [
            popen_launch_pd_server(
                cls.model,
                f"http://127.0.0.1:{cls.base_port + i}",
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--trust-remote-code",
                    "--tp",
                    "4",
                    "--enable-dp-attention",
                    "--dp",
                    "4",
                    "--elastic-ep-backend",
                    "mooncake",
                    "--mooncake-ib-device",
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `TODO` and `popen_launch_pd_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 45-48: Scenario logic / 场景逻辑
```python
                    ib_devices,
                    "--moe-a2a-backend",
                    "mooncake",
                    "--deepep-mode",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 49-69: Scenario logic / 场景逻辑
```python
                    "low_latency",
                    "--moe-dense-tp-size",
                    "1",
                    "--enable-dp-lm-head",
                    "--enable-two-batch-overlap",
                    "--disable-custom-all-reduce",
                    "--enable-elastic-expert-backup",
                    "--enable-eplb",
                    "--eplb-rebalance-num-iterations",
                    "50",
                    "--chunked-prefill-size",
                    "512",
                    "--cuda-graph-max-bs",
                    "128",
                    "--max-running-requests",
                    "512",
                    "--mem-fraction-static",
                    "0.5",
                    "--dist-init-addr",
                    "127.0.0.1:5000",
                    "--nnodes",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 70-73: Scenario logic / 场景逻辑
```python
                    f"{cls.num_processes}",
                    "--node-rank",
                    f"{i}",
                    "--base-gpu-id",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 74-94: Request and response handling / 请求与响应处理
```python
                    f"{i * 2}",
                ],
            )
            for i in range(cls.num_processes)
        ]

        server_ready = [False] * cls.num_processes
        start_time = time.perf_counter()
        with requests.Session() as session:
            while (
                time.perf_counter() - start_time < DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH
                and not all(server_ready)
            ):
                for i, process in enumerate(cls.processes):
                    return_code = process.poll()
                    if return_code is not None:
                        # Server failed to start (non-zero exit code) or crashed
                        raise Exception(
                            f"Server process exited with code {return_code}. "
                            "Check server logs for errors."
                        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `perf_counter`, `Session`, `poll` and `start`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 95-98: Scenario logic / 场景逻辑
```python

                    try:
                        headers = {
                            "Content-Type": "application/json; charset=utf-8",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 99-119: Process control logic / 进程控制逻辑
```python
                        }
                        response = session.get(
                            f"http://127.0.0.1:{cls.base_port + i}/health_generate",
                            headers=headers,
                        )
                        if response.status_code == 200:
                            server_ready[i] = True
                    except requests.RequestException:
                        pass

                    return_code = process.poll()
                    if return_code is not None:
                        raise Exception(
                            f"Server unexpectedly exits ({return_code=}). Usually there will be error logs describing the cause far above this line."
                        )

                    time.sleep(10)
        if not all(server_ready):
            for process in cls.processes:
                kill_process_tree(process.pid)
            raise TimeoutError("Server failed to start within the timeout period.")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get`, `poll`, `Exception` and `exits`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 120-121: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 122-124: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        for process in cls.processes:
            kill_process_tree(process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 126-139: Test routines around test_gsm8k / 测试例程
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
        print(metrics)

        self.assertGreater(metrics["score"], 0.60)
```
**EN:** This range defines concrete test routine(s) `test_gsm8k`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `SimpleNamespace`, `run_eval` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 140-143: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `time`, `types`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
