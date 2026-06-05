# server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/sweep/server.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements server support for the `sweep` portion of vLLM. / 为 vLLM 的 `sweep` 子目录实现与 server 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-11)
```python
import contextlib

import os

import signal

import subprocess

import time

from types import TracebackType

import requests

from typing_extensions import Self
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Class `ServerProcess` (lines 14-142)
```python
class ServerProcess:
    VLLM_RESET_CACHE_ENDPOINTS = [
        "/reset_prefix_cache",
        "/reset_mm_cache",
        "/reset_encoder_cache",
    ]

    def __init__(
        self,
        server_cmd: list[str],
        after_bench_cmd: list[str],
        *,
        show_stdout: bool,
    ) -> None:
        super().__init__()

        self.server_cmd = server_cmd
        self.after_bench_cmd = after_bench_cmd
        self.show_stdout = show_stdout

    def __enter__(self) -> Self:
        self.start()
        return self
    # ... omitted for brevity ...
                "Please specify a custom command via `--after-bench-cmd`."
            )
```
**EN:** Class `ServerProcess` is a structured building block in this module. Key methods include `__init__`, `__enter__`, `__exit__`, `start`, `stop`, `run_subcommand`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `ServerProcess` 是该模块中的结构化构件。 关键方法包括 `__init__`, `__enter__`, `__exit__`, `start`, `stop`, `run_subcommand`，它们共同定义初始化、校验、变换或访问模式。

### Method `ServerProcess.__init__` (lines 21-32)
```python
    def __init__(
        self,
        server_cmd: list[str],
        after_bench_cmd: list[str],
        *,
        show_stdout: bool,
    ) -> None:
        super().__init__()

        self.server_cmd = server_cmd
        self.after_bench_cmd = after_bench_cmd
        self.show_stdout = show_stdout
```
**EN:** Method `ServerProcess.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super` show the concrete execution path.
**CN:** Method `ServerProcess.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ServerProcess.__exit__` (lines 38-44)
```python
    def __exit__(
        self,
        exc_type: type[BaseException] | None,
        exc_value: BaseException | None,
        exc_traceback: TracebackType | None,
    ) -> None:
        self.stop()
```
**EN:** Method `ServerProcess.__exit__` provides a reusable helper around the module's main workflow. Key calls such as `self.stop` show the concrete execution path.
**CN:** Method `ServerProcess.__exit__` 为模块主流程提供可复用的辅助逻辑。 像 `self.stop` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ServerProcess.start` (lines 46-54)
```python
    def start(self):
        # Create new process for clean termination
        self._server_process = subprocess.Popen(
            self.server_cmd,
            start_new_session=True,
            stdout=None if self.show_stdout else subprocess.DEVNULL,
            # Need `VLLM_SERVER_DEV_MODE=1` for `_reset_caches`
            env=os.environ | {"VLLM_SERVER_DEV_MODE": "1"},
        )
```
**EN:** Method `ServerProcess.start` provides a reusable helper around the module's main workflow. Key calls such as `subprocess.Popen` show the concrete execution path.
**CN:** Method `ServerProcess.start` 为模块主流程提供可复用的辅助逻辑。 像 `subprocess.Popen` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ServerProcess.stop` (lines 56-63)
```python
    def stop(self):
        server_process = self._server_process

        if server_process.poll() is None:
            # In case only some processes have been terminated
            with contextlib.suppress(ProcessLookupError):
                # We need to kill both API Server and Engine processes
                os.killpg(os.getpgid(server_process.pid), signal.SIGKILL)
```
**EN:** Method `ServerProcess.stop` provides a reusable helper around the module's main workflow. Key calls such as `server_process.poll`, `contextlib.suppress`, `os.killpg`, `os.getpgid` show the concrete execution path.
**CN:** Method `ServerProcess.stop` 为模块主流程提供可复用的辅助逻辑。 像 `server_process.poll`, `contextlib.suppress`, `os.killpg`, `os.getpgid` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ServerProcess.wait_until_ready` (lines 106-119)
```python
    def wait_until_ready(self, timeout: int) -> None:
        start_time = time.monotonic()
        while not self.is_server_ready():
            # Check if server process has crashed
            if self._server_process.poll() is not None:
                returncode = self._server_process.returncode
                raise RuntimeError(
                    f"Server process crashed with return code {returncode}"
                )
            if time.monotonic() - start_time > timeout:
                raise TimeoutError(
                    f"Server failed to become ready within {timeout} seconds."
                )
            time.sleep(1)
```
**EN:** Method `ServerProcess.wait_until_ready` handles loading or retrieval of external/internal data. Key calls such as `time.monotonic`, `self.is_server_ready`, `self._server_process.poll`, `RuntimeError`, `TimeoutError` show the concrete execution path.
**CN:** Method `ServerProcess.wait_until_ready` 负责加载或获取外部/内部数据。 像 `time.monotonic`, `self.is_server_ready`, `self._server_process.poll`, `RuntimeError`, `TimeoutError` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ServerProcess.reset_caches` (lines 121-142)
```python
    def reset_caches(self) -> None:
        server_cmd = self.server_cmd

        # Use `.endswith()` to match `/bin/...`
        if server_cmd[0].endswith("vllm"):
            server_address = self._get_vllm_server_address()
            print(f"Resetting caches at {server_address}")

            for endpoint in self.VLLM_RESET_CACHE_ENDPOINTS:
                res = requests.post(server_address + endpoint)
                res.raise_for_status()
        elif server_cmd[0].endswith("infinity_emb"):
            if "--vector-disk-cache" in server_cmd:
                raise NotImplementedError(
                    "Infinity server uses caching but does not expose a method "
                    "to reset the cache"
                )
        else:
            raise NotImplementedError(
                f"No implementation of `reset_caches` for `{server_cmd[0]}` server. "
                "Please specify a custom command via `--after-bench-cmd`."
            )
```
**EN:** Method `ServerProcess.reset_caches` manages cache lookup, reuse, or lifecycle decisions. Key calls such as `server_cmd[0].endswith`, `self._get_vllm_server_address`, `print`, `requests.post`, `res.raise_for_status` show the concrete execution path.
**CN:** Method `ServerProcess.reset_caches` 负责管理缓存查询、复用或生命周期决策。 像 `server_cmd[0].endswith`, `self._get_vllm_server_address`, `print`, `requests.post`, `res.raise_for_status` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import contextlib`, `import os`, `import signal`, `import subprocess`, `import time`, `from types import TracebackType`
- **Third-party / 第三方**: `import requests`, `from typing_extensions import Self`
