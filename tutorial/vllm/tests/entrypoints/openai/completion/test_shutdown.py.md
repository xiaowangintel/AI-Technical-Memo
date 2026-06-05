# test_shutdown.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/completion/test_shutdown.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers text completion behavior and OpenAI-compatible serving. The file defines 7 test(s), 0 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖文本补全行为与OpenAI 兼容服务。它定义了 7 个测试、0 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L19)
```python
import asyncio
import signal
import subprocess
import sys
import time
from dataclasses import dataclass, field

import httpx
import openai
import psutil
import pytest

from tests.utils import RemoteOpenAIServer
from vllm.platforms import current_platform
from vllm.utils.network_utils import get_open_port
```
**EN:** Imports standard-library modules such as `asyncio`, `dataclasses.dataclass`, `dataclasses.field`, third-party packages like `httpx`, `openai`, `psutil`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.platforms.current_platform`, `vllm.utils.network_utils.get_open_port`.
**CN:** 导入标准库模块（如 `asyncio`、`dataclasses.dataclass`、`dataclasses.field`）、第三方包（如 `httpx`、`openai`、`psutil`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.platforms.current_platform`、`vllm.utils.network_utils.get_open_port`）。

### Module setup / 模块级配置: MODEL_NAME, _IS_ROCM, _SERVER_STARTUP_TIMEOUT (L21-L28)
```python
MODEL_NAME = "hmellor/tiny-random-LlamaForCausalLM"

# GPU initialization might take take longer
_IS_ROCM = current_platform.is_rocm()
_SERVER_STARTUP_TIMEOUT = 120
_PROCESS_EXIT_TIMEOUT = 15
_SHUTDOWN_DETECTION_TIMEOUT = 10
_CHILD_CLEANUP_TIMEOUT = 10
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `_IS_ROCM`, `_SERVER_STARTUP_TIMEOUT`, `_PROCESS_EXIT_TIMEOUT`, `_SHUTDOWN_DETECTION_TIMEOUT`, `_CHILD_CLEANUP_TIMEOUT`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`_IS_ROCM`、`_SERVER_STARTUP_TIMEOUT`、`_PROCESS_EXIT_TIMEOUT`、`_SHUTDOWN_DETECTION_TIMEOUT`、`_CHILD_CLEANUP_TIMEOUT`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _get_child_pids (L31-L36)
```python
def _get_child_pids(parent_pid: int) -> list[int]:
    try:
        parent = psutil.Process(parent_pid)
        return [c.pid for c in parent.children(recursive=True)]
    except psutil.NoSuchProcess:
        return []
```
**EN:** This helper encapsulates reusable logic in `_get_child_pids`. Key inputs are `parent_pid`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_child_pids` 中。 关键输入包括 `parent_pid`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _assert_children_cleaned_up (L39-L64)
```python
async def _assert_children_cleaned_up(
    child_pids: list[int],
    timeout: float = _CHILD_CLEANUP_TIMEOUT,
):
    """Wait for child processes to exit and fail if any remain."""
    if not child_pids:
        return

    deadline = time.time() + timeout
    while time.time() < deadline:
        still_alive = []
        for pid in child_pids:
            try:
                p = psutil.Process(pid)
                if p.is_running() and p.status() != psutil.STATUS_ZOMBIE:
                    still_alive.append(pid)
            except psutil.NoSuchProcess:
                pass
        if not still_alive:
            return
        await asyncio.sleep(0.5)

    pytest.fail(
        f"Child processes {still_alive} still alive after {timeout}s. "
        f"Process cleanup may not be working correctly."
    )
```
**EN:** This async helper encapsulates reusable logic in `_assert_children_cleaned_up`. Key inputs are `child_pids`, `timeout`.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `_assert_children_cleaned_up` 中。 关键输入包括 `child_pids`、`timeout`。

### Class / 类: ShutdownState (L67-L75)
```python
@dataclass
class ShutdownState:
    got_503: bool = False
    got_500: bool = False
    requests_after_sigterm: int = 0
    aborted_requests: int = 0
    connection_errors: int = 0
    stop_requesting: bool = False
    errors: list[str] = field(default_factory=list)
```
**EN:** This class groups related scenarios in `ShutdownState`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `ShutdownState` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Helper / 辅助函数: _concurrent_request_loop (L78-L121)
```python
async def _concurrent_request_loop(
    client: openai.AsyncOpenAI,
    state: ShutdownState,
    sigterm_sent: asyncio.Event | None = None,
    concurrency: int = 10,
):
    """Run multiple concurrent requests to keep the server busy."""

    async def single_request():
        while not state.stop_requesting:
            try:
                response = await client.completions.create(
                    model=MODEL_NAME,
                    prompt="Write a story: ",
                    max_tokens=200,
                )
                if sigterm_sent is not None and sigterm_sent.is_set():
                    state.requests_after_sigterm += 1
# ... 18 lines omitted for brevity ...

    tasks = [asyncio.create_task(single_request()) for _ in range(concurrency)]
    try:
        await asyncio.gather(*tasks, return_exceptions=True)
    finally:
        for t in tasks:
            if not t.done():
                t.cancel()
```
**EN:** This async helper encapsulates reusable logic in `_concurrent_request_loop`. Key inputs are `client`, `state`, `sigterm_sent`, `concurrency`. The body fans out concurrent work via `asyncio.gather`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `_concurrent_request_loop` 中。 关键输入包括 `client`、`state`、`sigterm_sent`、`concurrency`。 函数体通过 `asyncio.gather` 并发展开工作负载。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_shutdown_on_engine_failure (L124-L205)
```python
@pytest.mark.asyncio
async def test_shutdown_on_engine_failure():
    """Verify that API returns connection error when server process is killed.

    Starts a vLLM server, kills it to simulate a crash, then verifies that
    subsequent API calls fail appropriately.
    """

    port = get_open_port()

    proc = subprocess.Popen(
        [
            # dtype, max-len etc set so that this can run in CI
            sys.executable,
            "-m",
            "vllm.entrypoints.openai.api_server",
            "--model",
            MODEL_NAME,
# ... 56 lines omitted for brevity ...
    # Verify API calls now fail
    with pytest.raises((openai.APIConnectionError, openai.APIStatusError)):
        await client.completions.create(
            model=MODEL_NAME, prompt="This should fail", max_tokens=1
        )

    return_code = proc.wait(timeout=_PROCESS_EXIT_TIMEOUT)
    assert return_code is not None
```
**EN:** This async test validates `test_shutdown_on_engine_failure`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `return_code is not None`.
**CN:** 这个异步测试验证 `test_shutdown_on_engine_failure`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `return_code is not None`。

### Test / 测试: test_wait_timeout_completes_requests (L208-L263)
```python
@pytest.mark.asyncio
async def test_wait_timeout_completes_requests():
    """Verify wait timeout: new requests rejected, in-flight requests complete."""
    server_args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "256",
        "--enforce-eager",
        "--gpu-memory-utilization",
        "0.05",
        "--max-num-seqs",
        "4",
        "--shutdown-timeout",
        "30",
    ]

    with RemoteOpenAIServer(MODEL_NAME, server_args) as remote_server:
# ... 30 lines omitted for brevity ...
        )
        # server must stop accepting new requests (503, 500, or connection close)
        assert state.got_503 or state.got_500 or state.connection_errors > 0, (
            f"Server should stop accepting requests. "
            f"completed: {state.requests_after_sigterm}, errors: {state.errors}"
        )

        await _assert_children_cleaned_up(child_pids)
```
**EN:** This async test validates `test_wait_timeout_completes_requests`. Relevant pytest markers include `asyncio`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The body fans out concurrent work via `asyncio.gather`. It drives client-facing request creation through the API surface under test. The main assertion is `state.requests_after_sigterm > 0` and `state.got_503 or state.got_500 or state.connection_errors > 0`.
**CN:** 这个异步测试验证 `test_wait_timeout_completes_requests`。 相关的 pytest 标记包括 `asyncio`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 函数体通过 `asyncio.gather` 并发展开工作负载。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `state.requests_after_sigterm > 0` and `state.got_503 or state.got_500 or state.connection_errors > 0`。

### Test / 测试: test_abort_timeout_exits_quickly (L266-L320)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("wait_for_engine_idle", [0.0, 2.0])
async def test_abort_timeout_exits_quickly(wait_for_engine_idle: float):
    server_args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "256",
        "--enforce-eager",
        "--gpu-memory-utilization",
        "0.05",
        "--max-num-seqs",
        "4",
        "--shutdown-timeout",
        "0",
    ]

    with RemoteOpenAIServer(MODEL_NAME, server_args) as remote_server:
# ... 29 lines omitted for brevity ...

        exit_time = time.time() - start_time
        assert exit_time < max_exit_time, (
            f"Default shutdown took too long: {exit_time:.1f}s"
        )
        assert proc.returncode in (0, -15, None), f"Unexpected: {proc.returncode}"

        await _assert_children_cleaned_up(child_pids)
```
**EN:** This async test validates `test_abort_timeout_exits_quickly`. It uses parameterization over `wait_for_engine_idle`. Relevant pytest markers include `asyncio`. Key inputs are `wait_for_engine_idle`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. It drives client-facing request creation through the API surface under test. The main assertion is `exit_time < max_exit_time` and `proc.returncode in (0, -15, None)`.
**CN:** 这个异步测试验证 `test_abort_timeout_exits_quickly`。 它通过参数化组合 `wait_for_engine_idle`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `wait_for_engine_idle`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `exit_time < max_exit_time` and `proc.returncode in (0, -15, None)`。

### Test / 测试: test_wait_timeout_with_short_duration (L323-L380)
```python
@pytest.mark.asyncio
async def test_wait_timeout_with_short_duration():
    """Verify server exits cleanly with a short wait timeout."""
    wait_timeout = 3
    server_args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "256",
        "--enforce-eager",
        "--gpu-memory-utilization",
        "0.05",
        "--max-num-seqs",
        "4",
        "--shutdown-timeout",
        str(wait_timeout),
    ]

# ... 32 lines omitted for brevity ...
            pytest.fail(f"Process did not exit within {max_wait}s after SIGTERM")

        assert exit_time < wait_timeout + 10, (
            f"Took too long to exit ({exit_time:.1f}s), expected <{wait_timeout + 10}s"
        )
        assert proc.returncode in (0, -15, None), f"Unexpected: {proc.returncode}"

        await _assert_children_cleaned_up(child_pids)
```
**EN:** This async test validates `test_wait_timeout_with_short_duration`. Relevant pytest markers include `asyncio`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The body fans out concurrent work via `asyncio.gather`. It drives client-facing request creation through the API surface under test. The main assertion is `exit_time < wait_timeout + 10` and `proc.returncode in (0, -15, None)`.
**CN:** 这个异步测试验证 `test_wait_timeout_with_short_duration`。 相关的 pytest 标记包括 `asyncio`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 函数体通过 `asyncio.gather` 并发展开工作负载。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `exit_time < wait_timeout + 10` and `proc.returncode in (0, -15, None)`。

### Test / 测试: test_abort_timeout_fails_inflight_requests (L383-L452)
```python
@pytest.mark.asyncio
async def test_abort_timeout_fails_inflight_requests():
    """Verify abort timeout (0) immediately aborts in-flight requests."""
    server_args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "256",
        "--enforce-eager",
        "--gpu-memory-utilization",
        "0.05",
        "--max-num-seqs",
        "4",
        "--shutdown-timeout",
        "0",
    ]

    with RemoteOpenAIServer(MODEL_NAME, server_args) as remote_server:
# ... 44 lines omitted for brevity ...
            if proc.poll() is not None:
                break
            time.sleep(0.1)

        exit_time = time.time() - start_time
        assert exit_time < 10, f"Abort timeout shutdown took too long: {exit_time:.1f}s"

        await _assert_children_cleaned_up(child_pids)
```
**EN:** This async test validates `test_abort_timeout_fails_inflight_requests`. Relevant pytest markers include `asyncio`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The body fans out concurrent work via `asyncio.gather`. It drives client-facing request creation through the API surface under test. The main assertion is `state.aborted_requests > 0 or state.connection_errors > 0 or state.got_500 or state.got_503` and `exit_time < 10`.
**CN:** 这个异步测试验证 `test_abort_timeout_fails_inflight_requests`。 相关的 pytest 标记包括 `asyncio`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 函数体通过 `asyncio.gather` 并发展开工作负载。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `state.aborted_requests > 0 or state.connection_errors > 0 or state.got_500 or state.got_503` and `exit_time < 10`。

### Test / 测试: test_request_rejection_during_shutdown (L455-L501)
```python
@pytest.mark.asyncio
async def test_request_rejection_during_shutdown():
    """Verify new requests are rejected with error during shutdown."""
    server_args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "256",
        "--enforce-eager",
        "--gpu-memory-utilization",
        "0.05",
        "--max-num-seqs",
        "4",
        "--shutdown-timeout",
        "30",
    ]

    with RemoteOpenAIServer(MODEL_NAME, server_args) as remote_server:
# ... 21 lines omitted for brevity ...
            await asyncio.sleep(0.1)

        assert rejected_count > 0, (
            f"Expected requests to be rejected during shutdown, "
            f"but {rejected_count} were rejected out of 10"
        )

        await _assert_children_cleaned_up(child_pids)
```
**EN:** This async test validates `test_request_rejection_during_shutdown`. Relevant pytest markers include `asyncio`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. It drives client-facing request creation through the API surface under test. The main assertion is `rejected_count > 0`.
**CN:** 这个异步测试验证 `test_request_rejection_during_shutdown`。 相关的 pytest 标记包括 `asyncio`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `rejected_count > 0`。

### Test / 测试: test_multi_api_server_shutdown (L504-L566)
```python
@pytest.mark.asyncio
async def test_multi_api_server_shutdown():
    """Verify shutdown works with multiple API servers."""
    server_args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "256",
        "--enforce-eager",
        "--gpu-memory-utilization",
        "0.05",
        "--max-num-seqs",
        "4",
        "--shutdown-timeout",
        "30",
        "--api-server-count",
        "2",
    ]
# ... 37 lines omitted for brevity ...
            time.sleep(0.1)

        if proc.poll() is None:
            proc.kill()
            proc.wait(timeout=5)
            pytest.fail("Process did not exit after SIGTERM")

        await _assert_children_cleaned_up(child_pids)
```
**EN:** This async test validates `test_multi_api_server_shutdown`. Relevant pytest markers include `asyncio`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The body fans out concurrent work via `asyncio.gather`. It drives client-facing request creation through the API surface under test. The main assertion is `len(child_pids) >= 2`.
**CN:** 这个异步测试验证 `test_multi_api_server_shutdown`。 相关的 pytest 标记包括 `asyncio`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 函数体通过 `asyncio.gather` 并发展开工作负载。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(child_pids) >= 2`。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `dataclasses.dataclass`, `dataclasses.field`, `signal`, `subprocess`, `sys`, `time`
- **Third-party / 第三方**: `httpx`, `openai`, `psutil`, `pytest`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.platforms.current_platform`, `vllm.utils.network_utils.get_open_port`
