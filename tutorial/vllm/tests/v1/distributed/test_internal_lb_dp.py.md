# test_internal_lb_dp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/distributed/test_internal_lb_dp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `internal lb dp` behavior and regressions in the v1 stack. / 验证 v1 栈中 `internal lb dp` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-17)
```python
import asyncio
import os
import threading
import time
import traceback
from typing import cast

import openai  # use the official client for correctness check
import pytest
import pytest_asyncio
import requests

from tests.utils import ROCM_ENV_OVERRIDES, RemoteOpenAIServer
from tests.v1.utils import check_request_balancing
from vllm.platforms import current_platform
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `openai, pytest, pytest_asyncio, requests`. vLLM modules under test include `vllm.platforms`. Local helpers come from `tests.utils, tests.v1.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `openai, pytest, pytest_asyncio, requests`。 被测试的 vLLM 模块包括 `vllm.platforms`。 本地测试辅助逻辑来自 `tests.utils, tests.v1.utils`。

### Module state / 模块级状态 (lines 19-27)
```python
MODEL_NAME = "ibm-research/PowerMoE-3b"

# Number of data parallel ranks for multi-node internal LB testing
DP_SIZE = int(os.getenv("DP_SIZE", "2"))
# Default tensor parallel size to use
TP_SIZE = int(os.getenv("TP_SIZE", "1"))

# Number of nodes to simulate
NUM_NODES = 2
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `MODEL_NAME, DP_SIZE, TP_SIZE, NUM_NODES`. Shared setup calls include `int, os.getenv`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`MODEL_NAME, DP_SIZE, TP_SIZE, NUM_NODES`。 共享初始化调用包括 `int, os.getenv`。

### _make_completion_request (lines 30-74)
```python
async def _make_completion_request(
    client: openai.AsyncOpenAI,
    model_name: str,
) -> openai.types.Completion:
    """Make a single completion request and validate the response.

    Uses temperature=1.0 to ensure diverse outputs across concurrent
    requests for realistic load balancer testing.
    """
    completion = await client.completions.create(
        model=model_name,
        prompt="Hello, my name is",
        max_tokens=5,
        temperature=1.0,
    )
    assert completion.id is not None, (
        f"Expected non-None completion id. usage={completion.usage!r}"
    # ... excerpt omitted for brevity ...
    assert completion.choices is not None and len(completion.choices) == 1, (
    assert choice.finish_reason in ("length", "stop"), (
        assert len(choice.text) >= 1, (
    assert completion.usage.prompt_tokens > 0, (
        f"Expected positive prompt_tokens, got {completion.usage.prompt_tokens}"
    assert completion.usage.total_tokens > 0, (
        f"Expected positive total_tokens, got {completion.usage.total_tokens}"
    return completion
```
**EN:** Helper function `_make_completion_request` encapsulates reusable logic for `completion request`. Inputs: `client, model_name`. Key calls include `completions.create, len`. It includes 6 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_make_completion_request` 封装了与 `completion request` 相关的可复用逻辑。 输入参数：`client, model_name`。 关键调用包括 `completions.create, len`。 其中包含 6 个内部断言，用于保护前置假设。

### _run_request_bursts (lines 77-105)
```python
async def _run_request_bursts(
    client: openai.AsyncOpenAI,
    model_name: str,
    num_requests: int = 200,
    num_bursts: int = 2,
):
    """Send multiple bursts of completion requests and validate all succeed."""
    for burst in range(num_bursts):
        all_tasks = []
        for _ in range(num_requests):
            all_tasks.append(
                asyncio.create_task(_make_completion_request(client, model_name))
            )
            await asyncio.sleep(0.01)

        results = await asyncio.gather(*all_tasks, return_exceptions=True)
        assert len(results) == num_requests, (
            f"Burst {burst}: expected {num_requests} results, got {len(results)}"
        )

        for result in results:
            if isinstance(result, BaseException):
                raise result

        assert all(completion is not None for completion in results), (
            f"Burst {burst}: some completions were None"
        )

        await asyncio.sleep(0.5)
```
**EN:** Helper function `_run_request_bursts` encapsulates reusable logic for `run request bursts`. Inputs: `client, model_name, num_requests, num_bursts`. Key calls include `range, all, all_tasks.append, asyncio.gather, len, isinstance`. It includes 2 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_run_request_bursts` 封装了与 `run request bursts` 相关的可复用逻辑。 输入参数：`client, model_name, num_requests, num_bursts`。 关键调用包括 `range, all, all_tasks.append, asyncio.gather, len, isinstance`。 其中包含 2 个内部断言，用于保护前置假设。

### MultinodeInternalLBServerManager (lines 108-237)
```python
class MultinodeInternalLBServerManager:
    """Manages multi-node data parallel vLLM server instances for internal
    load balancer testing using --headless mode."""

    def __init__(
        self,
        model_name: str,
        dp_size: int,
        api_server_count: int,
        base_server_args: list,
        dp_per_node: int = 1,
        tp_size: int = TP_SIZE,
    ):
        self.model_name = model_name
        self.dp_size = dp_size
        self.dp_per_node = dp_per_node
        self.tp_size = tp_size
        self.api_server_count = api_server_count
    # ... excerpt omitted for brevity ...
        return cast(list[tuple[RemoteOpenAIServer, list[str]]], self.servers)
        self.servers.clear()
        try:
            RemoteOpenAIServer.shutdown_many(servers)
        except Exception as e:
            print(f"Error stopping servers: {e}")
            traceback.print_exc()
```
**EN:** Class `MultinodeInternalLBServerManager` groups 0 test method(s) and 3 helper/fixture method(s).
**CN:** 类 `MultinodeInternalLBServerManager` 组织了 0 个测试方法，以及 3 个辅助或 fixture 方法。

### APIOnlyServerManager (lines 240-379)
```python
class APIOnlyServerManager:
    """Manages API-only server (Node 0) and headless engines server (Node 1)
    for testing separated API server and engine configuration."""

    def __init__(
        self,
        model_name: str,
        dp_size: int,
        api_server_count: int,
        base_server_args: list,
        tp_size: int = TP_SIZE,
    ):
        self.model_name = model_name
        self.dp_size = dp_size
        self.tp_size = tp_size
        self.api_server_count = api_server_count
        self.base_server_args = base_server_args
        self.servers: list[tuple[RemoteOpenAIServer, list[str]] | None] = [None] * 2
    # ... excerpt omitted for brevity ...
        return cast(list[tuple[RemoteOpenAIServer, list[str]]], self.servers)
        self.servers.clear()
        try:
            RemoteOpenAIServer.shutdown_many(servers)
        except Exception as e:
            print(f"Error stopping servers: {e}")
            traceback.print_exc()
```
**EN:** Class `APIOnlyServerManager` groups 0 test method(s) and 3 helper/fixture method(s).
**CN:** 类 `APIOnlyServerManager` 组织了 0 个测试方法，以及 3 个辅助或 fixture 方法。

### default_server_args (lines 383-393)
```python
def default_server_args():
    return [
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "128",
        "--enforce-eager",
    ]
```
**EN:** Fixture/helper `default_server_args` prepares reusable state for downstream tests. Key calls include `pytest.fixture`.
**CN:** `default_server_args` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `pytest.fixture`。

### server_manager (lines 397-409)
```python
def server_manager(request, default_server_args):
    api_server_count = request.param
    server_manager = MultinodeInternalLBServerManager(
        MODEL_NAME,
        DP_SIZE,
        api_server_count,
        default_server_args,
        DP_SIZE // NUM_NODES,
        TP_SIZE,
    )

    with server_manager:
        yield server_manager
```
**EN:** Fixture/helper `server_manager` prepares reusable state for downstream tests. Inputs: `request, default_server_args`. Key calls include `pytest.fixture, MultinodeInternalLBServerManager`.
**CN:** `server_manager` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`request, default_server_args`。 关键调用包括 `pytest.fixture, MultinodeInternalLBServerManager`。

### servers (lines 413-414)
```python
def servers(server_manager):
    return server_manager.servers
```
**EN:** Fixture/helper `servers` prepares reusable state for downstream tests. Inputs: `server_manager`.
**CN:** `servers` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`server_manager`。

### api_only_servers (lines 418-424)
```python
def api_only_servers(request, default_server_args):
    """Fixture for API-only server + headless engines configuration."""
    api_server_count = request.param
    with APIOnlyServerManager(
        MODEL_NAME, DP_SIZE, api_server_count, default_server_args, TP_SIZE
    ) as server_list:
        yield server_list
```
**EN:** Fixture/helper `api_only_servers` prepares reusable state for downstream tests. Inputs: `request, default_server_args`. Key calls include `pytest.fixture, APIOnlyServerManager`.
**CN:** `api_only_servers` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`request, default_server_args`。 关键调用包括 `pytest.fixture, APIOnlyServerManager`。

### client (lines 428-433)
```python
async def client(servers: list[tuple[RemoteOpenAIServer, list[str]]]):
    # For internal LB, we only connect to the head node (rank 0)
    # which provides the single API endpoint
    head_server = servers[0][0]
    async with head_server.get_async_client() as client:
        yield client
```
**EN:** Fixture/helper `client` prepares reusable state for downstream tests. Inputs: `servers`. Key calls include `head_server.get_async_client`.
**CN:** `client` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`servers`。 关键调用包括 `head_server.get_async_client`。

### api_only_client (lines 437-442)
```python
async def api_only_client(api_only_servers: list[tuple[RemoteOpenAIServer, list[str]]]):
    """Client fixture for API-only server configuration."""
    # Connect to the API-only server (first server in the list)
    api_server = api_only_servers[0][0]
    async with api_server.get_async_client() as client:
        yield client
```
**EN:** Fixture/helper `api_only_client` prepares reusable state for downstream tests. Inputs: `api_only_servers`. Key calls include `api_server.get_async_client`.
**CN:** `api_only_client` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`api_only_servers`。 关键调用包括 `api_server.get_async_client`。

### _get_parallel_config (lines 445-450)
```python
def _get_parallel_config(server: RemoteOpenAIServer):
    response = requests.get(server.url_for("server_info?config_format=json"))
    response.raise_for_status()

    vllm_config = response.json()["vllm_config"]
    return vllm_config["parallel_config"]
```
**EN:** Helper function `_get_parallel_config` encapsulates reusable logic for `parallel config`. Inputs: `server`. Key calls include `requests.get, response.raise_for_status, server.url_for, response.json`.
**CN:** 辅助函数 `_get_parallel_config` 封装了与 `parallel config` 相关的可复用逻辑。 输入参数：`server`。 关键调用包括 `requests.get, response.raise_for_status, server.url_for, response.json`。

### test_multinode_dp_server_info (lines 453-466)
```python
def test_multinode_dp_server_info(server_manager):
    head_server = server_manager.servers[0][0]
    api_server_count = server_manager.api_server_count

    # Each request will hit one of the API servers
    # `n_reqs` is set so that there is a good chance each server
    # receives at least one request
    n_reqs = 2 * api_server_count * api_server_count
    parallel_configs = [_get_parallel_config(head_server) for _ in range(n_reqs)]
    api_process_counts = [c["_api_process_count"] for c in parallel_configs]
    api_process_ranks = [c["_api_process_rank"] for c in parallel_configs]

    assert all(c == api_server_count for c in api_process_counts), api_process_counts
    assert all(0 <= r < api_server_count for r in api_process_ranks), api_process_ranks
```
**EN:** Test case covering `multinode dp server info`. Inputs/fixtures: `server_manager`. It exercises `all, _get_parallel_config, range`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `multinode dp server info` 的测试用例。 输入或 fixture：`server_manager`。 该测试会调用 `all, _get_parallel_config, range`。 代码主体包含 2 个显式断言。

### test_multinode_dp_completion (lines 474-502)
```python
async def test_multinode_dp_completion(
    client: openai.AsyncOpenAI,
    servers: list[tuple[RemoteOpenAIServer, list[str]]],
    model_name: str,
) -> None:
    # Test single request
    result = await _make_completion_request(client, model_name)
    assert result is not None
    print("Multi-node internal LB handled single completion request successfully")

    await asyncio.sleep(0.5)

    # Send multiple bursts - internal LB should distribute across DP ranks
    await _run_request_bursts(client, model_name)

    _, server_args = servers[0]
    api_server_count = (
        server_args.count("--api-server-count")
        and server_args[server_args.index("--api-server-count") + 1]
        or 1
    )
    print(
        f"Successfully completed multi-node internal LB test with "
        f"{len(servers)} DP ranks (API server count: {api_server_count})"
    )

    # Check request balancing via Prometheus metrics
    head_server = servers[0][0]
    check_request_balancing(head_server, DP_SIZE)
```
**EN:** Parameterized test covering `multinode dp completion`. Parameter axes: `model_name`. Inputs/fixtures: `client, servers, model_name`. It exercises `mark.parametrize, print, check_request_balancing, _make_completion_request, asyncio.sleep, _run_request_bursts`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `multinode dp completion` 的测试用例。 参数维度：`model_name`。 输入或 fixture：`client, servers, model_name`。 该测试会调用 `mark.parametrize, print, check_request_balancing, _make_completion_request, asyncio.sleep, _run_request_bursts`。 代码主体包含 1 个显式断言。

### test_multinode_dp_completion_streaming (lines 510-596)
```python
async def test_multinode_dp_completion_streaming(
    client: openai.AsyncOpenAI,
    servers: list[tuple[RemoteOpenAIServer, list[str]]],
    model_name: str,
) -> None:
    prompt = "What is an LLM?"

    async def make_streaming_request():
        # Perform a non-streaming request to get the expected full output
        single_completion = await client.completions.create(
            model=model_name,
            prompt=prompt,
            max_tokens=5,
            temperature=0.0,
        )
        single_output = single_completion.choices[0].text
        # Perform the streaming request
    # ... excerpt omitted for brevity ...
        assert finish_reason_count == 1, "Finish reason should appear exactly once."
        assert last_chunk is not None, "Stream should have yielded at least one chunk."
        assert last_chunk.choices[0].finish_reason == "length", (
        assert "".join(chunks) == single_output, (
        return True  # Indicate success for this request
    assert result is not None
        f"{len(servers)} DP ranks (API server count: {api_server_count})"
    )
    # Check request balancing via Prometheus metrics
    head_server = servers[0][0]
    check_request_balancing(head_server, DP_SIZE)
```
**EN:** Parameterized test covering `multinode dp completion streaming`. Parameter axes: `model_name`. Inputs/fixtures: `client, servers, model_name`. It exercises `mark.parametrize, print, range, all, check_request_balancing, make_streaming_request`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `multinode dp completion streaming` 的测试用例。 参数维度：`model_name`。 输入或 fixture：`client, servers, model_name`。 该测试会调用 `mark.parametrize, print, range, all, check_request_balancing, make_streaming_request`。 代码主体包含 9 个显式断言。

### test_api_only_multinode_dp_completion (lines 604-634)
```python
async def test_api_only_multinode_dp_completion(
    api_only_client: openai.AsyncOpenAI,
    api_only_servers: list[tuple[RemoteOpenAIServer, list[str]]],
    model_name: str,
) -> None:
    """Test API-only server with all engines on separate headless server."""

    # Test single request
    result = await _make_completion_request(api_only_client, model_name)
    assert result is not None
    print("API-only server handled single completion request successfully")

    await asyncio.sleep(0.5)

    # Send multiple bursts - should be distributed across engines on
    # headless server
    await _run_request_bursts(api_only_client, model_name)

    api_server, api_server_args = api_only_servers[0]
    api_server_count = (
        api_server_args.count("--api-server-count")
        and api_server_args[api_server_args.index("--api-server-count") + 1]
        or 1
    )
    print(
        f"Successfully completed API-only multi-node test with {DP_SIZE} "
        f"engines on headless server (API server count: {api_server_count})"
    )

    # Check request balancing via Prometheus metrics
    check_request_balancing(api_server, DP_SIZE)
```
**EN:** Parameterized test covering `api only multinode dp completion`. Parameter axes: `model_name`. Inputs/fixtures: `api_only_client, api_only_servers, model_name`. It exercises `mark.parametrize, print, check_request_balancing, _make_completion_request, asyncio.sleep, _run_request_bursts`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `api only multinode dp completion` 的测试用例。 参数维度：`model_name`。 输入或 fixture：`api_only_client, api_only_servers, model_name`。 该测试会调用 `mark.parametrize, print, check_request_balancing, _make_completion_request, asyncio.sleep, _run_request_bursts`。 代码主体包含 1 个显式断言。

### test_api_only_multinode_dp_completion_streaming (lines 642-729)
```python
async def test_api_only_multinode_dp_completion_streaming(
    api_only_client: openai.AsyncOpenAI,
    api_only_servers: list[tuple[RemoteOpenAIServer, list[str]]],
    model_name: str,
) -> None:
    """Test API-only server streaming with all engines on separate
    headless server."""
    prompt = "What is an LLM?"

    async def make_streaming_request():
        # Perform a non-streaming request to get the expected full output
        single_completion = await api_only_client.completions.create(
            model=model_name,
            prompt=prompt,
            max_tokens=5,
            temperature=0.0,
        )
        single_output = single_completion.choices[0].text
    # ... excerpt omitted for brevity ...
        assert finish_reason_count == 1, "Finish reason should appear exactly once."
        assert last_chunk is not None, "Stream should have yielded at least one chunk."
        assert last_chunk.choices[0].finish_reason == "length", (
        assert "".join(chunks) == single_output, (
        return True  # Indicate success for this request
    assert result is not None
        f"engines on headless server (API server count: {api_server_count})"
    )
    # Check request balancing via Prometheus metrics
    api_server = api_only_servers[0][0]
    check_request_balancing(api_server, DP_SIZE)
```
**EN:** Parameterized test covering `api only multinode dp completion streaming`. Parameter axes: `model_name`. Inputs/fixtures: `api_only_client, api_only_servers, model_name`. It exercises `mark.parametrize, print, range, all, check_request_balancing, make_streaming_request`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `api only multinode dp completion streaming` 的测试用例。 参数维度：`model_name`。 输入或 fixture：`api_only_client, api_only_servers, model_name`。 该测试会调用 `mark.parametrize, print, range, all, check_request_balancing, make_streaming_request`。 代码主体包含 9 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围

## Dependencies / 依赖关系
- **EN:** External libraries: `openai, pytest, pytest_asyncio, requests`.
- **CN:** 外部库：`openai, pytest, pytest_asyncio, requests`。
- **EN:** vLLM modules under test: `vllm.platforms`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms`。
- **EN:** Local test helpers: `tests.utils, tests.v1.utils`.
- **CN:** 本地测试辅助模块：`tests.utils, tests.v1.utils`。
- **EN:** Standard-library support: `asyncio, os, threading, time, traceback, typing`.
- **CN:** 标准库支持：`asyncio, os, threading, time, traceback, typing`。
