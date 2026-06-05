# test_pp_with_hicache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/hicache/test_pp_with_hicache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `pp with hicache` scenario in `test/manual/hicache`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/hicache` 中的 `pp with hicache` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and shared helpers / 导入与共享辅助项
```python
"""
Usage:
python3 -m unittest test_pp_with_hicache.TestPPWithHiCache.test_eval_accuracy
"""

import os
import subprocess
import time
import unittest
from types import SimpleNamespace
from urllib.parse import urlparse

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    find_available_port,
    popen_launch_server,
)
```
**EN:** This range imports `os`, `subprocess`, `time` and `unittest`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 25-26: Class definition for TestPPWithHiCache / 类定义
```python
class TestPPWithHiCache(unittest.TestCase):
    @classmethod
```
**EN:** This range declares `TestPPWithHiCache`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 27-49: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.base_url = f"http://127.0.0.1:{find_available_port(23337)}"
        parsed_url = urlparse(cls.base_url)
        cls.base_host = parsed_url.hostname
        cls.base_port = str(parsed_url.port)
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST

        cls._start_mooncake_services()

        server_args_dict = {
            "--enable-hierarchical-cache": True,
            "--mem-fraction-static": 0.6,
            "--hicache-ratio": 1.2,
            "--page-size": 64,
            "--enable-cache-report": True,
            "--hicache-storage-prefetch-policy": "wait_complete",
            "--hicache-storage-backend": "mooncake",
            "--tp-size": 2,
            "--pp-size": 2,
            "--chunked-prefill-size": 256,
            "--hicache-mem-layout": "page_first",
        }
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `find_available_port`, `urlparse` and `_start_mooncake_services`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 50-51: Scenario logic / 场景逻辑
```python
        final_server_args = []
        for key, value in server_args_dict.items():
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `items`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 52-68: Process control logic / 进程控制逻辑
```python
            final_server_args.append(str(key))
            if value is not True:
                final_server_args.append(str(value))

        env_vars = {**os.environ, **cls._mooncake_env()}

        try:
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=final_server_args,
                env=env_vars,
            )
        except Exception:
            cls._stop_mooncake_services()
            raise
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `append`, `_mooncake_env`, `popen_launch_server` and `_stop_mooncake_services`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 69-70: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 71-74: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        if hasattr(cls, "process"):
            kill_process_tree(cls.process.pid)
        cls._stop_mooncake_services()
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `hasattr`, `kill_process_tree` and `_stop_mooncake_services`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 75-76: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 77-99: Helper routines around _start_mooncake_services / 辅助例程
```python
    def _start_mooncake_services(cls):
        try:
            import mooncake.http_metadata_server  # type: ignore  # noqa: F401
        except Exception as exc:  # pragma: no cover - environment dependent
            raise unittest.SkipTest(
                f"Mooncake metadata server module unavailable: {exc}"
            ) from exc

        cls._mooncake_master_port = find_available_port(50051)
        cls._mooncake_metadata_port = find_available_port(8080)

        try:
            cls._mooncake_metadata_process = subprocess.Popen(
                [
                    "python3",
                    "-m",
                    "mooncake.http_metadata_server",
                    "--port",
                    str(cls._mooncake_metadata_port),
                ],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                preexec_fn=os.setsid,
```
**EN:** This range implements helper routine(s) `_start_mooncake_services` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `SkipTest`, `find_available_port` and `Popen`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 100-101: Process control logic / 进程控制逻辑
```python
            )
        except (FileNotFoundError, subprocess.SubprocessError) as exc:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `except`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 102-120: Process control logic / 进程控制逻辑
```python
            cls._stop_mooncake_services()
            raise unittest.SkipTest(
                f"Could not start Mooncake metadata service: {exc}"
            ) from exc

        try:
            cls._mooncake_master_process = subprocess.Popen(
                ["mooncake_master", "--port", str(cls._mooncake_master_port)],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                preexec_fn=os.setsid,
            )
        except (FileNotFoundError, subprocess.SubprocessError) as exc:
            cls._stop_mooncake_services()
            raise unittest.SkipTest(f"Could not start mooncake_master: {exc}") from exc

        if not cls._wait_for_mooncake_ready():
            cls._stop_mooncake_services()
            raise unittest.SkipTest("Mooncake services did not become ready in time")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `_stop_mooncake_services`, `SkipTest`, `Popen` and `except`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 121-122: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 123-124: Helper routines around _stop_mooncake_services / 辅助例程
```python
    def _stop_mooncake_services(cls):
        for attr in ("_mooncake_metadata_process", "_mooncake_master_process"):
```
**EN:** This range implements helper routine(s) `_stop_mooncake_services` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `in`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 125-133: Scenario logic / 场景逻辑
```python
            proc = getattr(cls, attr, None)
            if proc:
                try:
                    os.killpg(os.getpgid(proc.pid), 9)
                    proc.wait(timeout=5)
                except Exception:
                    pass
        cls._mooncake_metadata_process = None
        cls._mooncake_master_process = None
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `getattr`, `killpg`, `getpgid` and `wait`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 134-135: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 136-145: Helper routines around _mooncake_env / 辅助例程
```python
    def _mooncake_env(cls):
        return {
            "MOONCAKE_MASTER": f"127.0.0.1:{cls._mooncake_master_port}",
            "MOONCAKE_PROTOCOL": "tcp",
            "MC_MS_AUTO_DISC": "0",
            "MOONCAKE_DEVICE": "",
            "MOONCAKE_TE_META_DATA_SERVER": f"http://127.0.0.1:{cls._mooncake_metadata_port}/metadata",
            "MOONCAKE_GLOBAL_SEGMENT_SIZE": "4294967296",
            "SGLANG_ENABLE_DETERMINISTIC_INFERENCE": "1",
        }
```
**EN:** This range implements helper routine(s) `_mooncake_env` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 146-147: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 148-149: Helper routines around _wait_for_mooncake_ready / 辅助例程
```python
    def _wait_for_mooncake_ready(cls, timeout: int = 30) -> bool:
        start_time = time.time()
```
**EN:** This range implements helper routine(s) `_wait_for_mooncake_ready` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `time`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 150-172: Request and response handling / 请求与响应处理
```python
        while time.time() - start_time < timeout:
            metadata_ready = False
            master_ready = False

            if (
                getattr(cls, "_mooncake_metadata_process", None)
                and cls._mooncake_metadata_process.poll() is None
            ):
                try:
                    resp = requests.get(
                        f"http://127.0.0.1:{cls._mooncake_metadata_port}/metadata",
                        timeout=2,
                    )
                    print(resp)
                    metadata_ready = True
                except requests.RequestException:
                    metadata_ready = False

            if (
                getattr(cls, "_mooncake_master_process", None)
                and cls._mooncake_master_process.poll() is None
            ):
                if time.time() - start_time > 3:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `time`, `getattr`, `poll` and `get`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 173-174: Scenario logic / 场景逻辑
```python
                    master_ready = True
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 175-180: Scenario logic / 场景逻辑
```python
            if metadata_ready and master_ready:
                return True

            time.sleep(1.5)

        return False
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `sleep`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 182-188: Helper routines around flush_cache / 辅助例程
```python
    def flush_cache(self):
        res = requests.post(
            f"{self.base_url}/flush_cache",
            params={"timeout": 30},
            timeout=40,
        )
        res.raise_for_status()
```
**EN:** This range implements helper routine(s) `flush_cache` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `post` and `raise_for_status`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 190-199: Test routines around test_eval_accuracy / 测试例程
```python
    def test_eval_accuracy(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=40,
            num_threads=24,
        )
```
**EN:** This range defines concrete test routine(s) `test_eval_accuracy`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Representative call sites include `SimpleNamespace`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 200-210: Assertions and result checks / 断言与结果检查
```python

        metrics_initial = run_eval(args)
        self.assertGreater(metrics_initial["score"], 0.6)

        self.flush_cache()

        metrics_cached = run_eval(args)
        self.assertGreater(metrics_cached["score"], 0.6)

        accuracy_diff = abs(metrics_initial["score"] - metrics_cached["score"])
        self.assertLess(accuracy_diff, 0.05)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Assertions in this block enforce the intended outcome. Representative call sites include `run_eval`, `assertGreater`, `flush_cache` and `abs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 211-214: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Environment-aware configuration / 环境感知配置
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `os`, `subprocess`, `time`, `types`, `unittest`, `urllib.parse`
- **Third-party / 第三方库**: `mooncake.http_metadata_server`, `requests`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.test_utils`
