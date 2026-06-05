# test_hybrid_lb_dp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/distributed/test_hybrid_lb_dp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `hybrid lb dp` behavior and regressions in the v1 stack. / 验证 v1 栈中 `hybrid lb dp` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-16)
```python
import asyncio
import os
import threading
import time
from contextlib import AsyncExitStack

import openai  # use the official client for correctness check
import pytest
import pytest_asyncio
import requests

from tests.utils import RemoteOpenAIServer
from tests.v1.utils import check_request_balancing
from vllm.platforms import current_platform
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `openai, pytest, pytest_asyncio, requests`. vLLM modules under test include `vllm.platforms`. Local helpers come from `tests.utils, tests.v1.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `openai, pytest, pytest_asyncio, requests`。 被测试的 vLLM 模块包括 `vllm.platforms`。 本地测试辅助逻辑来自 `tests.utils, tests.v1.utils`。

### Module state / 模块级状态 (lines 18-27)
```python
MODEL_NAME = "ibm-research/PowerMoE-3b"

# Number of data parallel ranks for hybrid LB testing (4 total)
DP_SIZE = int(os.getenv("DP_SIZE", "4"))
# Default tensor parallel size to use
TP_SIZE = int(os.getenv("TP_SIZE", "1"))

# Number of nodes (2 nodes, each with 2 DP ranks)
NUM_NODES = 2
DP_SIZE_LOCAL = DP_SIZE // NUM_NODES  # 2 ranks per node
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `MODEL_NAME, DP_SIZE, TP_SIZE, NUM_NODES, DP_SIZE_LOCAL`. Shared setup calls include `int, os.getenv`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`MODEL_NAME, DP_SIZE, TP_SIZE, NUM_NODES, DP_SIZE_LOCAL`。 共享初始化调用包括 `int, os.getenv`。

### HybridLBServerManager (lines 30-142)
```python
class HybridLBServerManager:
    """Manages hybrid data parallel vLLM server instances where each node
    runs a single logical API server that balances requests only to the
    DP engines running on that same node."""

    def __init__(
        self,
        model_name: str,
        dp_size: int,
        api_server_count: int,
        base_server_args: list,
        dp_size_local: int = DP_SIZE_LOCAL,
        tp_size: int = TP_SIZE,
    ):
        self.model_name = model_name
        self.dp_size = dp_size
        self.dp_size_local = dp_size_local
        self.tp_size = tp_size
    # ... excerpt omitted for brevity ...
        return self.servers
        servers = [s for s, _ in self.servers]
        self.servers.clear()
        try:
            RemoteOpenAIServer.shutdown_many(servers)
        except Exception as e:
            print(f"Error stopping servers: {e}")
```
**EN:** Class `HybridLBServerManager` groups 0 test method(s) and 3 helper/fixture method(s).
**CN:** 类 `HybridLBServerManager` 组织了 0 个测试方法，以及 3 个辅助或 fixture 方法。

### default_server_args (lines 146-156)
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

### server_manager (lines 160-172)
```python
def server_manager(request, default_server_args):
    api_server_count = request.param
    server_manager = HybridLBServerManager(
        MODEL_NAME,
        DP_SIZE,
        api_server_count,
        default_server_args,
        DP_SIZE_LOCAL,
        TP_SIZE,
    )

    with server_manager:
        yield server_manager
```
**EN:** Fixture/helper `server_manager` prepares reusable state for downstream tests. Inputs: `request, default_server_args`. Key calls include `pytest.fixture, HybridLBServerManager`.
**CN:** `server_manager` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`request, default_server_args`。 关键调用包括 `pytest.fixture, HybridLBServerManager`。

### servers (lines 176-177)
```python
def servers(server_manager):
    return server_manager.servers
```
**EN:** Fixture/helper `servers` prepares reusable state for downstream tests. Inputs: `server_manager`.
**CN:** `servers` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`server_manager`。

### clients (lines 181-187)
```python
async def clients(servers: list[tuple[RemoteOpenAIServer, list[str]]]):
    # Create a client for each node (each node has its own API endpoint)
    async with AsyncExitStack() as stack:
        yield [
            await stack.enter_async_context(server.get_async_client())
            for server, _ in servers
        ]
```
**EN:** Fixture/helper `clients` prepares reusable state for downstream tests. Inputs: `servers`. Key calls include `AsyncExitStack, stack.enter_async_context, server.get_async_client`.
**CN:** `clients` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`servers`。 关键调用包括 `AsyncExitStack, stack.enter_async_context, server.get_async_client`。

### _get_parallel_config (lines 190-195)
```python
def _get_parallel_config(server: RemoteOpenAIServer):
    response = requests.get(server.url_for("server_info?config_format=json"))
    response.raise_for_status()

    vllm_config = response.json()["vllm_config"]
    return vllm_config["parallel_config"]
```
**EN:** Helper function `_get_parallel_config` encapsulates reusable logic for `parallel config`. Inputs: `server`. Key calls include `requests.get, response.raise_for_status, server.url_for, response.json`.
**CN:** 辅助函数 `_get_parallel_config` 封装了与 `parallel config` 相关的可复用逻辑。 输入参数：`server`。 关键调用包括 `requests.get, response.raise_for_status, server.url_for, response.json`。

### test_hybrid_dp_server_info (lines 198-218)
```python
def test_hybrid_dp_server_info(server_manager):
    servers = server_manager.servers
    api_server_count = server_manager.api_server_count

    for i, (server, _) in enumerate(servers):
        print(f"Testing {i=}")

        # Each request will hit one of the API servers
        # `n_reqs` is set so that there is a good chance each server
        # receives at least one request
        n_reqs = 2 * api_server_count * api_server_count
        parallel_configs = [_get_parallel_config(server) for _ in range(n_reqs)]
        api_process_counts = [c["_api_process_count"] for c in parallel_configs]
        api_process_ranks = [c["_api_process_rank"] for c in parallel_configs]

        assert all(c == api_server_count for c in api_process_counts), (
            api_process_counts
        )
        assert all(0 <= r < api_server_count for r in api_process_ranks), (
            api_process_ranks
        )
```
**EN:** Test case covering `hybrid dp server info`. Inputs/fixtures: `server_manager`. It exercises `enumerate, print, all, _get_parallel_config, range`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `hybrid dp server info` 的测试用例。 输入或 fixture：`server_manager`。 该测试会调用 `enumerate, print, all, _get_parallel_config, range`。 代码主体包含 2 个显式断言。

### test_hybrid_lb_completion (lines 226-301)
```python
async def test_hybrid_lb_completion(
    clients: list[openai.AsyncOpenAI],
    servers: list[tuple[RemoteOpenAIServer, list[str]]],
    model_name: str,
) -> None:
    async def make_request(client: openai.AsyncOpenAI):
        completion = await client.completions.create(
            model=model_name, prompt="Hello, my name is", max_tokens=5, temperature=1.0
        )

        assert completion.id is not None
        assert completion.choices is not None and len(completion.choices) == 1
        choice = completion.choices[0]
        # The exact number of tokens can vary slightly with temperature=1.0,
        # so we check for a reasonable minimum length.
        assert len(choice.text) >= 1
        # Finish reason might not always be 'length' if the model finishes early
    # ... excerpt omitted for brevity ...
        assert choice.finish_reason in ("length", "stop")
        assert completion.usage.completion_tokens > 0
        assert completion.usage.prompt_tokens > 0
        assert completion.usage.total_tokens > 0
        return completion
        assert result is not None
    )
    # Check request balancing within each node
    for i, (server, _) in enumerate(servers):
        print(f"Checking request balancing for node {i}")
        check_request_balancing(server, DP_SIZE_LOCAL)
```
**EN:** Parameterized test covering `hybrid lb completion`. Parameter axes: `model_name`. Inputs/fixtures: `clients, servers, model_name`. It exercises `mark.parametrize, enumerate, range, all, print, asyncio.sleep`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `hybrid lb completion` 的测试用例。 参数维度：`model_name`。 输入或 fixture：`clients, servers, model_name`。 该测试会调用 `mark.parametrize, enumerate, range, all, print, asyncio.sleep`。 代码主体包含 12 个显式断言。

### test_hybrid_lb_completion_streaming (lines 309-399)
```python
async def test_hybrid_lb_completion_streaming(
    clients: list[openai.AsyncOpenAI],
    servers: list[tuple[RemoteOpenAIServer, list[str]]],
    model_name: str,
) -> None:
    prompt = "What is an LLM?"

    async def make_streaming_request(client: openai.AsyncOpenAI):
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
    )
    # Check request balancing within each node
    for i, (server, _) in enumerate(servers):
        print(f"Checking streaming request balancing for node {i}")
        check_request_balancing(server, DP_SIZE_LOCAL)
```
**EN:** Parameterized test covering `hybrid lb completion streaming`. Parameter axes: `model_name`. Inputs/fixtures: `clients, servers, model_name`. It exercises `mark.parametrize, enumerate, range, all, print, asyncio.sleep`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `hybrid lb completion streaming` 的测试用例。 参数维度：`model_name`。 输入或 fixture：`clients, servers, model_name`。 该测试会调用 `mark.parametrize, enumerate, range, all, print, asyncio.sleep`。 代码主体包含 9 个显式断言。

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
- **EN:** Standard-library support: `asyncio, os, threading, time, contextlib`.
- **CN:** 标准库支持：`asyncio, os, threading, time, contextlib`。
